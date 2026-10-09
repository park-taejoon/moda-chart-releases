# 플러그인 · 대체 렌더러 가이드

코어에는 세 개의 확장 경계가 있다 — 스냅샷 생성(`GeometryBuilder`),
화면 반영(`ChartRenderer`), 그리고 둘을 붙이는 진입점(`ChartFeature`/
`CoreHost`). 내부 구조와 계약은 `docs/design/modularization.md` 참조.

```
데이터/옵션 ──► ChartCore ──► ChartSnapshot ──► ChartRenderer ──► 화면
                  ▲                                ▲
            GeometryBuilder로               renderInto() 구현으로
            지오메트리 조각 생성             DOM/canvas/기타로 반영
                  ▲
            CoreHost를 통해 피처가 등록
```

## 1. 피처 작성 — `ChartFeature` / `CoreHost`

피처는 코어에 접근하는 유일한 통로로 `CoreHost`를 받는다.
내부 필드·컨트롤러는 노출되지 않는다.

```ts
import type { ChartFeature, CoreHost } from "@moda-chart/core";

const myFeature: ChartFeature = {
  install(host: CoreHost) {
    // 필요한 등록/구독을 여기서 한다
  },
};
```

`CoreHost` 진입점:

| 메서드                       | 설명                                       |
| ---------------------------- | ------------------------------------------ |
| `options()`                  | 현재 `ChartCoreOptions` (읽기 전용)        |
| `resolvedSeries()`           | id/색/축이 해소된 시리즈 목록              |
| `getSnapshot()`              | 현재 스냅샷 — 참조 안정성 계약 동일        |
| `requestInvalidate()`        | 피처가 상태를 바꿨을 때 스냅샷 무효화 요청 |
| `emit(type, e)`              | 차트 이벤트 발행                           |
| `registerGeometryBuilder(b)` | 지오메트리 빌더 등록 (§2)                  |

설치 경로는 둘이다:

```ts
// 생성 시 — 이벤트 핸들러 배선 후 install이 호출된다
new ChartCore({ series, features: [myFeature] });

// 사후 설치
chart.use(myFeature);
```

피처는 `host`를 보관해 둘 수 있다 — 이후 `requestInvalidate()`로
갱신을 요청하거나 `emit()`으로 커스텀 이벤트를 발행할 수 있다.

## 2. 커스텀 차트 타입 — `GeometryBuilder`

빌더는 스냅샷의 `geometry` 조각을 만드는 순수 함수다.
담당 타입이 아니면 `null`을 반환해 다음 빌더로 넘긴다.

```ts
import type { GeometryBuilder, ChartFeature } from "@moda-chart/core";

const myBuilder: GeometryBuilder = (ctx) => {
  if ((ctx.type as string) !== "my-chart") return null;
  return {
    geometry: [/* GeometryView[] — plot 좌표계(px)로 계산한 뷰 모델 */],
  };
};

export const myChartFeature: ChartFeature = {
  install: (host) => host.registerGeometryBuilder(myBuilder),
};
```

- **우선순위** — 등록된 빌더는 내장 빌더보다 먼저 호출된다. 내장 타입
  (`"bar"` 등)을 가로채는 빌더로 기본 렌더를 재정의할 수도 있다.
- **커스텀 타입** — `options.type`은 `ChartType` 외 임의 문자열도 받는다
  (`ChartType | (string & {})`). 스냅샷의 `type`·`data-chart-type`
  속성에도 그 문자열이 그대로 들어간다.
- **주의** — 지오메트리 빌더는 스냅샷 조각만 만든다. 새 `GeometryView`
  종류를 화면에 그리려면 렌더러 쪽 대응(`svg.ts` 분기 또는 커스텀
  `ChartRenderer`)도 필요하다.

`BuildContext`의 주요 입력:

| 필드                                                                                     | 설명                           |
| ---------------------------------------------------------------------------------------- | ------------------------------ |
| `type` / `horizontal` / `plot`                                                           | 차트 타입·방향·플롯 사각형(px) |
| `visible` / `categories` / `options`                                                     | 숨김 제외 시리즈·카테고리·옵션 |
| `hidden` / `hiddenSlices` / `focused` / `hover` / `legendHover`                          | 인터랙션 상태                  |
| `axes` / `axisDomains`                                                                   | 해소된 축과 값 도메인          |
| `kind` / `catPos` / `catStep` / `xScale` / `valScale`                                    | 스케일 — 도메인 → px           |
| `mapTf`                                                                                  | 지도 팬/줌 변환 `{k,tx,ty}`    |
| `categoryLabel` / `pointStyle` / `pointsOf` / `histEdges` / `histCounts` / `mapFeatures` | 코어 헬퍼 콜백                 |

부수 캐시(지도 히트·범례 라벨·노드 메타)는 반환값의 `side`로 돌려주면
코어가 적용한다 — 빌더 안에서 외부 상태를 직접 변경하지 않는다.

## 2-1. 지오메트리 이미터 — `GeometryEmitter`

커스텀 빌더가 만든 `GeometryView`를 화면에 그리는 렌더 측 짝이다.
빌더와 같은 체인 규칙: 담당 `kind`가 아니면 `null`을 반환해 다음
이미터로 넘기고, 마크업 문자열을 반환하면 그걸 채택한다.

```ts
import {
  registerGeometryEmitter,
  type GeometryEmitter,
} from "@moda-chart/core";

const myEmitter: GeometryEmitter = (g, ctx) => {
  if (g.kind !== "probe") return null; // 커스텀 kind만 클레임
  return `<g class="mc-series mc-type-probe">…</g>`;
};
registerGeometryEmitter(myEmitter); // 전역 — 모든 체인 앞에 끼워짐
```

- `ctx`(`RenderContext`) — `snap`/`mode`/`fillOf`(패턴 인지 fill)/
  `col`/`colP`/`seriesOf` 색 해석 헬퍼를 제공한다. 색상은 헬퍼를
  써야 `mode.export`(SVG보내기)에서 팔레트 상수로 인라인된다.
- **등록은 전역이다** — DOM 렌더러가 스냅샷만 받아 인스턴스를 구별할
  수 없다. 특정 차트에만 적용하려면 커스텀 `kind`로 클레임 조건을
  좁힌다 (빌더가 고유 kind를 찍어주는 방식).
- `toSVGString`/PNG 보내기에도 같은 체인이 적용된다 — 피처가 커스텀
  타입을보내기 가능하게 하려면 이미터도 함께 등록한다.
- 피처 안에서는 `host.registerGeometryEmitter`로 접근한다.
- 부분 체인을 직접 구성할 때는 `createSvgRenderer(emitters)`로
  렌더러를 만든다 (§4-1 참조).

## 3. 대체 렌더러 — `ChartRenderer`

스냅샷 → 화면 변환은 `ChartRenderer` 뒤에 캡슐화돼 있다. 기본 구현은
`svgRenderer`(SVG/DOM `mc-*` 계약)이며, canvas·WebGL 같은 대체
렌더러는 같은 계약을 구현해 주입한다.

```ts
export interface ChartRenderer {
  renderInto(root: HTMLElement, snap: ChartSnapshot): void;
  skeletonHtml?(): string; // 최초 DOM 골격 — 미지정 시 기본 스켈레톤
  toMarkup?(snap: ChartSnapshot): string; // 독립 마크업 (export용)
}
```

주입 경로:

```ts
// vanilla
mountChart(el, { ...options, renderer: myRenderer });
attachChartView(el, chart, myRenderer);

// 어댑터 — ChartView의 renderer prop
<ChartView chart={chart} renderer={myRenderer} />
```

### 골격 주의 — `bindChart`가 기대하는 요소

이벤트 배선(`bindChart`)은 스켈레톤 안의 `.mc-*` 요소를 질의한다.
`skeletonHtml()`을 재정의하는 렌더러는 아래 요소를 유지해야 인터랙션이
동작한다 (전체 목록은 `mount.ts`의 DOM 계약 주석):

`.mc-viewport` · `.mc-overlay` · `.mc-tooltip` · `.mc-menu` ·
`.mc-legend` · `.mc-zoom-reset` · `.mc-drillup` · `.mc-title` ·
`.mc-subtitle` · `.mc-loading` · `.mc-a11y`

`renderInto`는 `subscribe` 콜백으로 스냅샷이 바뀔 때마다 호출된다.
가장 단순한 시작점은 `svgRenderer`를 감싸는 래퍼다:

```ts
import { svgRenderer, type ChartRenderer } from "@moda-chart/core";

const annotating: ChartRenderer = {
  ...svgRenderer,
  renderInto(root, snap) {
    svgRenderer.renderInto(root, snap);
    root.dataset.customFlag = String(snap.series.length);
  },
};
```

`toMarkup`을 구현하지 않으면 `downloadSVG`/`toSVGString` 계열의
SVG 내보내기는 그 렌더러로 지원되지 않는다(export 경로는 SVG 전용).

## 3-1. 내장 Canvas2D 렌더러 — `createCanvasRenderer`

`createCanvasRenderer()`는 `ChartRenderer` 계약의 Canvas2D 구현이다.
`.mc-svg` 대신 `<canvas class="mc-canvas">`에 스냅샷을 그리고,
툴팁·범례·메뉴·툴바 등 HTML 크롬과 `.mc-overlay` 인터랙션은 SVG
렌더러와 동일한 DOM 계약을 유지한다 (`updateChartChrome` 공유).

```ts
import { createCanvasRenderer } from "@moda-chart/core";

// vanilla
mountChart(el, { ...options, renderer: createCanvasRenderer() });

// 어댑터 — ChartView의 renderer prop
<ChartView chart={chart} renderer={createCanvasRenderer()} />
```

SVG 렌더러와의 차이:

- **지오메트리 DOM 없음** — `.mc-bar`/`.mc-point` 같은 요소가 생기지
  않는다. 히트 테스트는 코어의 스냅샷 좌표 경로라 호버·클릭·드래그·
  줌·키보드 내비는 동일하게 동작한다.
- **네비게이터/스크롤바/조직도 토글** — DOM 요소가 없으므로 스트립
  드래그와 토글 배지 클릭은 좌표 라우팅으로 처리한다
  (view-bindings.ts).
- **morph/enter/exit 트랜지션 미적용** — CSS/DOM 기반 애니메이션이라
  갱신 시 최종 프레임을 즉시 그린다.
- **색상 해석** — `var(--chart-*)` 토큰을 `.mc-root`의 계산 스타일에서
  읽어 실제 색으로 변환한다 (canvas/theme.ts). `theme.vars` 인라인
  주입도 동일하게 반영된다.
- **내보내기** — `toMarkup` 미구현. SVG/PNG/CSV 보내기는 코어의
  `toSVGString` 경로라 렌더러와 무관하게 그대로 동작한다.

렌더러는 인스턴스를 재사용해야 한다(아바타/주석 이미지 캐시 보유) —
React처럼 렌더가 반복되는 환경에서는 `useMemo` 등으로 고정한다.

## 4. 공개 능력 슬라이스 — `Chart*Api`

공개 인터페이스 `Chart`는 능력별 슬라이스의 합집합이다.
소비자는 필요한 능력만 타입으로 요구할 수 있다:

```ts
// 줌만 쓰는 유틸은 좁은 계약으로 받는다
function bindWheel(chart: ChartZoomApi) {
  return (e: WheelEvent) => chart.zoomAt(e.offsetX, e.deltaY < 0 ? 1.1 : 0.9);
}
```

| 슬라이스              | 내용                                            |
| --------------------- | ----------------------------------------------- |
| `ChartBaseApi`        | instanceId·subscribe·on·getSnapshot·dispose·use |
| `ChartDataApi`        | setData·실시간 API·loadXml·updateDelta·batch    |
| `ChartStateApi`       | getState·setState                               |
| `ChartConfigApi`      | setType·setTheme·setSize·setLocale 등 setter    |
| `ChartLegendApi`      | 범례 토글·체크·호버·페이지네이션                |
| `ChartInteractionApi` | 호버·클릭·선택·메뉴·키보드 포커스·escape        |
| `ChartZoomApi`        | 드래그·선택·줌/팬·네비게이터                    |
| `ChartDrilldownApi`   | drillDown/drillUp/drillUpAll                    |
| `ChartDrawingApi`     | 그리기 도구·주석                                |
| `ChartMapApi`         | mapZoomAt                                       |
| `ChartSearchApi`      | search.load                                     |
| `ChartExportApi`      | print·emitPrint·toSVGString                     |

**현재 한계** — 슬라이스는 타입 분해다. `ChartCore`는 항상 전체
`Chart`를 구현하며, 특정 능력을 런타임에 비활성화해 메서드를
제거/무효화하는 기능은 없다 (API 의미가 바뀌므로 별도 설계 대상).

## 4-1. lite 진입점 — 빌더·이미터 선택 번들

`@moda-chart/core/lite`는 내장 지오메트리 빌더·이미터 전체를 번들에서
제외하는 경량 경로다. 필요한 차트 타입의 빌더와 이미터를 골라 넣는다:

```ts
import {
  mountChartLite,
  cartesianGeometry,
  cartesianEmitter,
} from "@moda-chart/core/lite";

mountChartLite(el, {
  series,
  categories,
  builders: [cartesianGeometry], // 스냅샷 지오메트리 생성 체인
  emitters: [cartesianEmitter], // 지오메트리 → SVG 마크업 체인
});
```

- 동작은 같다 — `Chart` 인터페이스·`mc-*` DOM 계약·컨포먼스를 공유한다.
- `builders`·`emitters`는 **체인 대체**다 — 빌더 체인에 없는 타입은
  지오메트리가 만들어지지 않고, 이미터 체인에 없는 kind는 그려지지
  않는다. `cartesianGeometry`는 전 타입 폴백이라 마지막에 넣으면
  나머지 타입도 cartesian으로 렌더된다.
- `emitters`는 `toSVGString`/PNG 보내기와 기본 SVG 렌더러
  (`createSvgRenderer`)에 적용된다. `renderer`를 직접 주입하면
  emitters는 export 경로에만 쓰인다.
- 부분 조합에는 `defaultGeometryBuilders`·`defaultGeometryEmitters`
  (풀 경로 export)를 섞어 쓴다.
- 풀 `ChartCore`도 `options.builders`/`options.emitters`로 체인을
  대체할 수 있다 — 생성 시에만 적용된다.
- 구조: `core-impl.ts`(빌더·이미터 주입) ← `core.ts`(내장 체인 주입) /
  `lite.ts`(options 주입). 뷰 계층(`view.ts`)과 합성(`svg-impl.ts`)은
  내장 체인을 참조하지 않아 lite 경로가 풀 빌더·이미터를 끌어오지
  않는다.

## 4-2. 능력 비활성화 — `options.disabled`

lite가 **코드를 번들에서 빼는** 것이라면 `disabled`는 코드를 둔 채
**동작만 끄는** 옵션이다. `Chart` 타입은 그대로라 비파괴다.

```ts
new ChartCore({
  series,
  disabled: ["zoom", "export"], // 끌 능력 슬라이스
});
```

능력 이름은 `Chart*Api` 슬라이스와 대응한다:

| 값          | 끄는 API                                                            |
| ----------- | ------------------------------------------------------------------- |
| `zoom`      | zoomAt/panByPx/setZoomWindow/resetZoom/startDrag/nav\*/setSelection |
| `drilldown` | drillDown/drillUp/drillUpAll                                        |
| `drawing`   | setDrawMode/draw\*/addDrawing/getDrawings/clearDrawings             |
| `map`       | mapZoomAt + 지도 드래그 팬                                          |
| `search`    | search.load                                                         |
| `export`    | print/emitPrint/toSVGString                                         |

**관대한 실패 의미** — 비활성 능력의 메서드는 조용히 무시되고
반환형은 중립값을 돌린다 (`drillDown`→`false`, `getDrawings`→`[]`,
`toSVGString`→`""`, `search.load`→`false`). throw하지 않는다 —
어댑터의 prop 동기화와 DOM 이벤트 바인딩이 조건 없이 호출하기
때문이다. 기존 옵션 게이트(drilldown 미설정 시 `drillDown`→`false`)와
같은 의미다.

비활성 능력의 뷰 어포던스도 스냅샷에서 생략된다 — `.mc-zoom-reset`,
`.mc-export-svg`/`.mc-export-png`, 네비게이터/스크롤바 스트립,
컨텍스트 메뉴의 `reset-zoom`/`export-*` 내장 항목. 커스텀 메뉴
항목만 남으면 그대로 열리고, 전부 빠지면 메뉴 자체가 열리지 않는다.

## 5. 지켜야 할 계약

- `getSnapshot()` — 변경 전까지 같은 참조 (React `useSyncExternalStore` 등이 의존)
- setter 멱등성 — 동일 값 재호출은 무효화를 발생시키지 않는다
- `mc-*` DOM 계약 — 새 DOM 요소는 같은 접두어 + conformance 갱신
- `sideEffects: ["**/*.css"]` — 모듈은 import 부작용이 없어야
  트리셰이킹이 유지된다 (모듈 최상위에서 상태를 변경하지 않는다)
