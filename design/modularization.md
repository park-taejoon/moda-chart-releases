# 코어 모듈화 · 인터페이스화 설계

> 상태: Phase 1·2·3·4·5·6 완료 — `src/build/` 순수 빌더 + 레지스트리,
> `hit-test.ts`, `controllers/` 6개 슬라이스 추출, `interface Chart`
> 명문화, `features/core-host.ts` 피처 계약, `renderer.ts`의
> `ChartRenderer` 경계, `Chart`의 능력 슬라이스 분해(`Chart*Api` 12개),
> `core-impl.ts` 분리 + `lite.ts` 서브패스(빌더·이미터 선택 번들),
> `render/` 이미터 분해 + `svg-impl.ts` 합성부.
> 목적: 유지보수성 + 플러그인/트리셰이킹 + 대체 렌더러 대비

## 배경 — 진단

`packages/core/src/core.ts`(6300줄)는 사실상 세 덩어리가 한 파일이다.

| 구간                                   | 내용                                                       |
| -------------------------------------- | ---------------------------------------------------------- |
| `ChartCore` 공개 API (207–1819줄)      | ~35개 메서드 — 데이터/줌/드릴/그리기/범례/메뉴/포커스/sync |
| `buildSnapshot()` + 헬퍼 (3031–6263줄) | 3200줄 단일 메서드 — 타입별 뷰 계산 전부                   |
| private 상태 (~60개 필드)              | 데이터/뷰포트/인터랙션/지도/그리기/sync 뒤섞임             |

`hitTest`(2411–2941줄, 500줄)도 별도 덩어리다.
어댑터(`packages/{react,vue,vue2,svelte}`)는 이미 얇으므로 리팩터 대상은
이 파일 하나다.

## 목표 아키텍처

두 개의 안정된 경계를 세운다.

### 경계 1 — 스냅샷 생성 (`ViewBuilder`)

상태 → 불변 `ChartSnapshot`. 타입 패밀리별 순수 함수를
**외부 등록 가능한 레지스트리**로 연결한다.

```ts
/** 빌더가 보는 읽기 전용 컨텍스트 — 해소된 시리즈/스케일/팔레트/옵션 */
interface BuildContext { /* ... */ }

/** 차트 타입 하나의 스냅샷 조각을 만드는 순수 함수 */
type ViewBuilder = (ctx: BuildContext) => Partial<ChartSnapshot>;

/** 피처가 자기 타입을 등록하는 진입점 — 하드코딩 switch 대체 */
registerViewBuilder(type: ChartType, b: ViewBuilder): void
```

### 경계 2 — 렌더링 (`ChartRenderer`)

`ChartSnapshot` → 화면. `svg.ts`를 `SvgRenderer` 구현체로 격하시키고
`mount.ts`가 렌더러를 주입받는다. Canvas/WebGL 렌더러가 같은 계약으로
나중에 붙을 수 있다.

```ts
interface ChartRenderer {
  render(snap: ChartSnapshot, host: HTMLElement): void;
  hitTest?(snap: ChartSnapshot, px: number, py: number): HitResult | null;
  dispose(): void;
}
// mountChart(el, opts, renderer = new SvgRenderer())
```

### 피처 계약 (`ChartFeature` + `CoreHost`)

선택 기능(지도/그리기/search/XML 등)이 코어에 접근하는 유일한 통로.
내부 필드는 은닉하고 콜백 주입으로 단방향을 강제한다.

```ts
interface CoreHost {
  options(): ChartCoreOptions;
  series(): ResolvedSeries[];
  requestInvalidate(): void;
  emit<K extends ChartEventType>(type: K, e: ChartEventMap[K]): void;
}
interface ChartFeature {
  install(host: CoreHost): void;
}
```

### 파일 배치 (목표)

```
packages/core/src/
  core/         파사드 + 상태 슬라이스 (Phase 2)
                (data/viewport/interaction/drill/map-state/drawing/sync)
  build/        ✅ Phase 1 완료 — internal.ts(공용 타입/상수),
                context.ts(BuildContext/GeometryResult/판정 헬퍼),
                registry.ts(순서 체인 + registerGeometryBuilder),
                slice/heatmap/wordcloud/pictogram/radar/geo-map/
                hierarchy/flow/polar/gauge/cartesian(폴백) 빌더
  features/     core-host.ts 피처 계약 ✅ (Phase 4)
  render/       ✅ Phase 6 완료 — internal.ts(공용 헬퍼),
                context.ts(RenderContext/GeometryEmitter/sliceMarkup),
                registry.ts(emitGeometry + registerGeometryEmitter),
                defaults.ts(내장 이미터 체인 — 풀 전용),
                cartesian/slice/heatmap/wordcloud/pictogram/radar/
                geo-map/hierarchy/flow/polar/gauge 이미터
  svg-impl.ts   ✅ Phase 6 — svgInnerImpl/svgStringImpl (체인 주입)
  svg.ts        ✅ Phase 6 — 내장 체인 주입 얇은 진입점
  view.ts       ✅ Phase 5/6 — renderIntoView/createSvgRenderer/
                attachChartView (renderer 인자 필수 — 기본값은 mount.ts)
```

## 마이그레이션 단계

각 단계 끝에 `pnpm test && pnpm typecheck` 초록 게이트.
어댑터·conformance·e2e가 공유 계약이라 외부 동작은 바이트 단위로 동일해야 한다.

1. **Phase 1** — `buildSnapshot` 내부 타입별 블록을 순수 함수로 추출 →
   `build/*` + 내부 레지스트리. 스냅샷 동일성을 conformance로 검증
2. **Phase 2** — `hitTest` 분리 → 상태 슬라이스 store화 →
   `ChartCore` 파사드 축소
3. **Phase 3** — `interface Chart` 승격 + 어댑터 타입 의존 전환
   (공개 API 불변을 타입으로 검증)
4. **Phase 4** — 피처 계약 + 확장 심 ✅ (부분)
   - `features/core-host.ts`의 `CoreHost`/`ChartFeature` 계약 ✅
   - `options.features` + `chart.use()` 설치 경로 ✅
   - `registerGeometryBuilder` 공개 — 커스텀 타입 빌더가 내장 빌더
     앞에 끼워지는 플러그인 심 ✅
   - `options.type`을 `ChartType | (string & {})`로 확장해 커스텀
     타입 문자열 허용 ✅
   - `"sideEffects": ["**/*.css"]` 이미 설정 — 트리셰이킹 준비 완료 ✅
   - 4b ✅ (타입 분해) — `Chart`를 능력 슬라이스로 분해:
     `ChartBaseApi`/`ChartDataApi`/`ChartStateApi`/`ChartConfigApi`/
     `ChartLegendApi`/`ChartInteractionApi`/`ChartZoomApi`/
     `ChartDrilldownApi`/`ChartDrawingApi`/`ChartMapApi`/
     `ChartSearchApi`/`ChartExportApi` — `Chart extends`로 재조합해
     공개 API 동일, `implements Chart`가 멤버 전수를 검증.
     소비자는 좁은 계약만 요구할 수 있고 향후 lite 구현의 슬라이스
     선택 경로가 마련됐다. 런타임 비활성화(메서드 제거/무효화)는
     API 의미 변경이 필요해 여전히 별도 설계 대상.
5. **Phase 5** — 렌더러 인터페이스화 ✅
   - `src/renderer.ts`의 `ChartRenderer` 계약 — `renderInto` +
     선택적 `skeletonHtml`/`toMarkup` ✅
   - 기본 구현 `svgRenderer`(mount.ts — 기존 renderInto/svgString) ✅
   - 주입 경로: `MountChartOptions.renderer`,
     `attachChartView(el, chart, renderer)`, 4개 어댑터 `renderer` prop ✅
   - 커스텀 스켈레톤·갱신 호출은 renderer.test.ts로 검증 ✅
   - 주의: `downloadSVG`/`downloadPNG`는 SVG 전용 — 대체 렌더러는
     `toMarkup` 유무로 export 지원을 표현한다
6. **Phase 6** — `svg.ts` 렌더 분해 ✅
   - `render/internal.ts` — 공용 헬퍼(escapeXml/paint/pattern/scaleFill)
   - `render/context.ts` — `RenderContext` + `GeometryEmitter` 계약
     (빌더 체인과 같은 패턴: 미담당 kind는 null → 다음 이미터)
   - `render/<family>.ts` — 패밀리별 이미터 11개 (build/와 1:1 대응)
   - `render/registry.ts` — `emitGeometry` + `registerGeometryEmitter`
     (전역 등록 — DOM 렌더러가 인스턴스를 구별 못하므로 전역 의미)
   - `render/defaults.ts` — 내장 이미터 체인 (풀 진입만 도달)
   - `svg-impl.ts` — 합성부: defs/축/그리드/오버레이/주석/드로잉
   - `svg.ts` — 내장 체인 주입 얇은 셸 (`svgInner`/`svgString` 유지)
   - `view.ts` — `renderIntoView`/`createSvgRenderer` (체인 주입);
     기본값 바인딩은 mount.ts로 이동해 svg.js→defaults 경로 격리
   - `ChartCoreOptions.emitters` + `host.registerGeometryEmitter` ✅
   - lite 의존 그래프 검증: lite.ts → render/defaults·svg·core 미도달

순서가 중요하다 — 1→2로 스냅샷 생성 경계를 먼저 깨끗하게 만들어야
그 위의 `CoreHost`/`ChartRenderer` 인터페이스가 순환 참조 없이 성립한다.

## 지켜야 할 계약

1. **`getSnapshot()` 참조 안정성** — 스냅샷 캐시는 파사드에 유지.
   빌더가 순수 함수면 자연히 지켜진다.
2. **setter 멱등성** — store의 set이 "실제 변경 여부"를 반환해
   파사드가 `invalidate` 스킵 여부를 판단한다.
3. **`mc-*` DOM 계약 불변** — 내부 분리이므로 SVG 출력이 동일해야 함.
   `runChartConformance`가 안전망.
4. **순환 참조 차단** — store/피처 → 코어 역방향 호출은
   `CoreHost`/`invalidate` 콜백 주입으로 단방향 유지.
5. **교차 로직은 컨텍스트로** — `hitTest`처럼 여러 슬라이스를 동시에 보는
   코드는 `HitContext` 같은 읽기 전용 컨텍스트로 전달.
   store끼리 직접 참조시키지 않는다.

## 리스크 상위 3개

- 스냅샷 참조 계약 — 캐시 위치를 파사드에 고정
- 피처 간 순환 의존 — `CoreHost` 콜백 주입 + import 방향 규칙
- `sideEffects: false` 전환 — styles.css, `SYNC_GROUPS` 등록부 등
  "import만 하면 동작"하는 코드 경로 점검
