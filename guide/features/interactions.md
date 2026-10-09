# 인터랙션

## 범례

`legend: true | { position: "top"|"bottom"|"left"|"right"|"none" }`.
`.mc-legend-item` 클릭 → `chart.toggleLegend(id)` → 시리즈 hide/show.
숨겨진 항목은 `.mc-hidden` + `aria-pressed="false"`가 붙는다.
pie/donut에서는 슬라이스 단위(`slice:N`)로 토글된다.

범례 항목에 **hover하면 해당 시리즈만 남기고 나머지가 dim**된다
(`chart.hoverLegend(id)` — `legendHover` 이벤트 발행).

### 범례 체크박스

`legend: { checkboxes: true }` — 각 항목에 `.mc-legend-check`
체크박스가 붙는다 (IBChart seriesCheckBoxClick 해당). 체크박스를
클릭하면 토글과 함께 `legendCheck` `{ id, checked }` 이벤트가 발행된다
— 항목 텍스트 클릭은 기존처럼 `legendToggle`만 발행한다.

### 범례 집계값 · 전체 토글

- `legend: { value: "sum" | "avg" | "latest" | (values) => string }` —
  항목 라벨 옆에 집계값을 `.mc-legend-value`로 표시한다.
  pie/donut 등 슬라이스 범례는 해당 카테고리의 시리즈 값들이 넘어간다.
- `legend: { toggleAll: true }` — 범례 맨 앞에 `.mc-legend-all`
  버튼을 렌더한다. 전부 보이면 클릭 시 전부 숨기고, 하나라도
  숨겨져 있으면 전부 표시한다 (`chart.toggleAllLegend()`).
  발행 이벤트: `legendAllToggle` `{ visible }` + 시리즈별
  `visibilityChange`.

### 커스텀 범례 렌더러

`legend: { render: (ctx) => string }` — 반환 문자열이 `.mc-legend`의
innerHTML을 통째로 교체한다 (tooltip.render 패턴).

```ts
legend: {
  render: (ctx) =>
    ctx.items
      .map(
        (it) =>
          `<button class="mc-legend-item my-pill" data-id="${it.id}">` +
          `${it.label}${it.value != null ? ` (${it.value})` : ""}</button>`,
      )
      .join(""),
},
```

`ctx`에는 페이지네이션 적용 후 항목(`items`), 페이지 정보(`page`),
`allVisible`/`checks`/`toggleAll`, 내장 문구(`labels`)가 들어간다.

내장 인터랙션은 `.mc-legend`에 위임으로 바인딩되므로 커스텀 마크업이
DOM 계약을 지키면 그대로 동작한다:

| 클래스                   | 계약                                        |
| ------------------------ | ------------------------------------------- |
| `.mc-legend-item`        | `data-id` 필수 — 클릭 시 토글, hover 시 dim |
| `.mc-legend-check`       | 항목 안 체크박스 — `legendCheck` 경로       |
| `.mc-legend-all`         | 전체 토글 클릭                              |
| `.mc-legend-prev`/`next` | 페이지네이션 이동                           |

반환값은 escape 없이 삽입되므로 신뢰된 마크업만 반환할 것.
런타임에는 `chart.setLegend(opts)`로 범례 옵션을 통째로 교체한다.

## 툴팁

`tooltip: "point" | "shared" | "none" | { mode, format }`.

- `point` — 가장 가까운 포인트 1개
- `shared` — 같은 x 카테고리의 모든 시리즈 행 + `.mc-crosshair` 가이드
  - 활성 카테고리 배경 밴드 `.mc-crosshair-band`(band 축, 수평 막대는
    가로 스트립). 선형/시간 축은 슬롯이 없어 가이드선만 나온다
- `format(value, ctx)` — 값 표시 문자열 (ctx: seriesId/name/label/percent)
- `render(ctx)` — 툴팁 전체를 HTML 문자열로 렌더한다
  (`.mc-tooltip`의 innerHTML로 삽입). 반환값은 escape되지 않으므로
  신뢰된 마크업만 반환할 것 (AG Charts tooltip.renderer 해당)
- `pin: true` — 클릭으로 툴팁을 고정한다 (`.mc-tooltip.mc-pinned`).
  같은 포인트 재클릭·빈 영역 클릭·Esc로 해제하며, 고정된 툴팁은
  포인터가 벗어나도 닫히지 않는다. API: `chart.unpinTooltip()`.

## 포인트 hover · 클릭

오버레이 위의 pointermove가 `chart.hoverAt(px, py)`로 해석돼
가장 가까운 포인트를 하이라이트하고 툴팁을 연다.
클릭은 `chartClick`(항상, `point: null` 가능) + 히트 시 `seriesClick`,
더블클릭은 `pointDblClick`으로 발행된다.

## 드릴다운

`drilldown: { "라벨": DrilldownLevel }` — 클릭한 포인트/슬라이스의
라벨이 맵에 있으면 하위 레벨로 내려간다.

```ts
new ChartCore({
  type: "bar",
  series: [{ name: "월별", values: [12, 24, 16] }],
  categories: ["1월", "2월", "3월"],
  drilldown: {
    "1월": {
      series: [{ name: "1월 주별", values: [3, 4, 2, 3] }],
      categories: ["1주", "2주", "3주", "4주"],
      title: "1월 주별 매출", // 생략 시 상위 제목 유지
    },
  },
});
```

- 내려가면 툴바에 `‹ Back`(`.mc-drillup`) 버튼이 나타난다 —
  클릭 또는 `chart.drillUp()`으로 한 레벨 복귀.
- API: `chart.drillDown(key)`(성공 시 true) / `chart.drillUp()` /
  `chart.drillUpAll()`(최상위로 일괄 복귀, `drillupall` 이벤트).
- 레벨 교체 시 포커스·선택·줌·숨김 상태는 리셋된다.
- 이벤트: `drilldown` `{ key, depth }`, `drillup` `{ depth }`,
  `drillupall` `{ depth }`.

## 포인트 선택 (allowPointSelect)

`allowPointSelect: true` — 클릭으로 포인트/막대/슬라이스/셀/단어를
선택한다. 선택된 요소는 `.mc-selected` 클래스를 얻는다. 같은
요소를 다시 클릭하면 해제되고, 다른 요소를 선택하면 이전 선택이
먼저 해제된다.

- 이벤트: `pointSelect` `{ seriesId, index, value }`,
  `pointUnselect` `{ seriesId, index }` — 다른 포인트 선택 시 이전
  선택의 unselect가 먼저 발행된다.
- API: `chart.selectPoint(seriesId, index)`(토글),
  `chart.clearSelected()`.
- 스냅샷의 `selected` 필드와 각 뷰의 `selected` 플래그로 현재 선택을
  읽을 수 있다.
- 드릴다운 키에 해당하는 라벨을 클릭하면 드릴다운이 클릭을 소비해
  선택은 일어나지 않는다.

## 인쇄 이벤트

마운트된 차트는 window의 `beforeprint`/`afterprint`를 `beforePrint`/
`afterPrint` 이벤트로 전달한다(`window.print()` 포함). 인쇄 전에
확대 영역을 펼치거나 로딩을 끄는 용도로 쓴다.

## 브러시 선택

`brush: true` — 드래그로 x 범위를 선택한다 (`.mc-selection` rect).
선택 완료 시 `selectionChange`가 `indices`(포함 카테고리 인덱스)와
함께 발행된다. `chart.setSelection(range)` / `clearSelection()`으로
프로그래밍 제어도 가능하다.

## 줌 / 팬

`zoom: true | { wheel, pan, minSpan, axes, select }`.

- 휠 — 커서를 앵커로 x 도메인 확대/축소 (`minSpan` 하한)
- 드래그 — pan (brush와 같이 켜면 **Shift+드래그가 brush**)
- 더블클릭 또는 `.mc-zoom-reset` 버튼 — 리셋
- 창은 항상 전체 도메인 안으로 clamp된다
- 변경마다 `zoomChange` `{ window, yWindow }` 발행 —
  API로는 `setZoomWindow` / `resetZoom`

### x/y 양방향 줌

`axes: "x" | "y" | "xy"`(기본 `"x"`)로 줌이 적용되는 도메인을
고른다. `y`/`xy`면 **값 축 도메인(첫 y축)** 도 휠·팬 대상이 된다 —
IBChart의 zoomType xy에 해당한다.

```ts
zoom: { axes: "xy", select: true }
```

- `select: true` — Shift+드래그가 사각 영역 줌(`.mc-selection` rect에
  y 범위 포함). `brush`와 같이 켜면 brush가 우선한다.
- `setZoomWindow` — `{ x: {min,max}, y: {min,max} }`로 축별 지정,
  `null`은 해당 축 해제, 생략은 유지. `DomainWindow` 하나만 넘기면
  x 창으로 해석된다(기존 호출과 호환).
- 수평(`direction: "horizontal"`) 차트에서도 y 창은 값 도메인을
  가리킨다 — 그쪽 축이 화면상 x 방향으로 렌더된다.
- 드릴다운/타입 전환/데이터 교체 시 두 창 모두 리셋된다.

### 줌 프리셋

`zoom: { presets: ZoomPreset[] }` — 툴바 아래에 범위 버튼 행
(`.mc-presets` > `.mc-preset-btn`)을 렌더한다. 각 프리셋은 `label` +
`window`·`count` 중 하나:

```ts
zoom: {
  presets: [
    { label: "전체" },                       // 리셋
    { label: "최근 4", count: 4 },           // 도메인 끝에서 4개
    { label: "앞쪽", window: { min: 0, max: 2 } },
  ],
}
```

활성 프리셋에 `.mc-active`가 붙는다. API: `chart.applyZoomPreset(i)`.

## 전체화면

툴바의 `.mc-fullscreen` 버튼(항상 렌더)이 루트 요소의 전체화면을
토글한다. 네이티브 Fullscreen API를 지원하면 `requestFullscreen`을
호출하고, 지원하지 않거나 거부되면 `.mc-root.mc-fullscreen-on`
클래스 폴백만 둔다. 브라우저 Esc로 나가도 `fullscreenchange`
리스너가 코어 상태를 동기화한다. API: `chart.setFullscreen(bool)` /
`chart.toggleFullscreen()` — `fullscreenChange` `{ fullscreen }`
이벤트 발행.

## 네비게이터 (미니맵)

`navigator: true | { height }` — 플롯 아래에 전체 x 도메인 개요와
현재 줌 창을 그린다(`.mc-navigator`, AG Charts navigator에 해당).
직교 차트(슬라이스/히트맵/워드클라우드/레이더 제외)에서만 렌더되고,
스크롤이 아니라 **줌 창(`window`)을 제어**한다 — 창 이동/리사이즈는
`setZoomWindow`와 같은 상태를 바꾸므로 `zoomChange`가 발행된다.

- `.mc-nav-window` 드래그 — 창 이동(pan)
- `.mc-nav-handle-l`/`.mc-nav-handle-r` 드래그 — 가장자리 리사이즈
- `.mc-nav-bg`/`.mc-nav-track` 클릭 — 창 중심을 그 위치로 점프
- `.mc-nav-track` — 보이는 시리즈의 전체 범위 미니 라인(줌 무관)

코어 API로는 `navDragStart(px, edge)`/`navDragTo(px)`/`navDragEnd()`/
`navJump(px)`가 있고, `bindChart`가 포인터 이벤트를 이들로 연결한다.

## 컨텍스트 메뉴

`contextMenu: true | { items }` — 플롯 우클릭(contextmenu)으로
`.mc-menu`를 연다 (AG Charts contextMenu 해당). 내장 항목은
`reset-zoom`(줌 리셋)·`export-svg`·`export-png`·`export-csv`이고,
`items: [{ id?, label }]`로 커스텀 항목을 뒤에 붙인다. 커스텀 항목
클릭은 `menuAction` `{ id }` 이벤트로 전달되고 메뉴는 닫힌다.
메뉴 바깥 클릭이나 Esc로도 닫힌다. 코어 API: `chart.openMenu(px, py)` /
`closeMenu()` / `activateMenuItem(id)`.

## 차트 동기화 (sync)

`sync: "그룹키"` — 같은 키를 가진 차트끼리 호버·줌을 공유한다
(AG Charts sync 해당). 한 차트에서 호버/줌하면 그룹의 다른 차트에도
도메인 값 기준으로 전파된다. 동기화로 인한 변경은 다시 전파되지
않는다(피드백 루프 없음).

## 스크롤바

`scrollbar: true | { height }` — 네비게이터(미니 차트) 없이 얇은
줌 스트립만 그린다 (`.mc-scrollbar`, AG Charts scrollbar 해당).
`.mc-sb-window` 드래그로 창 이동, `.mc-sb-handle-l`/`-r`로
리사이즈 — 네비게이터와 같은 `navDragStart/To/End` 경로를 공유한다.
`navigator`가 켜져 있으면 네비게이터가 우선하고 스크롤바는 생략된다.

## 주석 그리기 도구 (drawing)

`drawing: true | { color, channelWidth }` + `chart.setDrawMode(tool)`로
활성화하면 플롯 드래그가 줌/선택 대신 주석을 만든다 (AG Charts
annotations 해당). 도구: `"line"`(추세선) / `"hline"`(수평선) /
`"vline"`(수직선) / `"rect"`(영역) / `"channel"`(평행 채널) /
`"fibonacci"`(피보나치 되돌림) / `"measure"`(구간 측정). 드래그 중
`.mc-draw-draft` 미리보기, 놓으면 확정되고 `drawEnd` 이벤트가
발행된다. 그린 주석은 `.mc-drawings` 안 `.mc-draw-<도구>`
그룹으로 렌더되며 `getState`/`setState`로 왕복된다.

- **channel** — 드래그한 추세선에 수직 오프셋 평행선+음영 밴드
  (`.mc-draw-band`, `.mc-draw-parallel`). 폭은 `drawing.channelWidth`
  또는 `DrawingInput.width` px.
- **fibonacci** — 드래그 구간을 0/23.6/38.2/50/61.8/78.6/100%로 나눈
  수평 되돌림 레벨 (`.mc-draw-fib-level`, `.mc-draw-fib-label`).
- **measure** — 드래그 구간의 Δx/Δy를 자동 라벨로 표시. `label`을
  명시하면 자동 라벨을 덮어쓴다.

```ts
chart.setDrawMode("fibonacci"); // Esc 또는 setDrawMode(null)로 해제
chart.getDrawings(); // DrawingInput[] — 데이터 좌표
chart.clearDrawings(); // 전부 제거
chart.addDrawing({ tool: "channel", x1: 0, y1: 5, x2: 4, y2: 30, width: 20 });
```

## 터치 핀치 줌

두 손가락 포인터가 오버레이 위에 동시에 닿으면 핀치 모드가 된다 —
간격 변화율로 중점 앵커 줌(`.mc-overlay`에 `touch-action: none`).
axes가 `y`/`xy`면 값 축도 함께 줌된다. 진행 중 드래그/그리기는
취소된다.

## 외부 데이터 로딩 (search)

IBChart `search` 네임스페이스 해당 — `search: { url, request?, transform?, auto? }`
옵션 또는 `chart.search.load(args)`로 외부 JSON을 가져와 `setData`에
주입한다. 로딩 중 `loading` 오버레이가 켜지고, 성공 시 `searchEnd`,
실패 시 `searchError` 이벤트가 발행된다. `fetcher`를 주입하면
fetch 대신 커스텀 로더를 쓸 수 있다.

```ts
const chart = new ChartCore({
  search: { url: "/api/chart-data.json", auto: true },
});
// 수동 로드 + 변환
await chart.search.load({
  url: "/api/other.json",
  transform: (raw) => ({ series: normalize(raw) }),
});
```

## 범례 페이지네이션

`legend: { pagination: true | pageSize }` — 항목이 페이지 크기를 넘으면
`.mc-legend-prev`/`.mc-legend-next` 버튼과 `.mc-legend-page` 표시가
생긴다 (AG legend pagination 해당). `chart.setLegendPage(n)`으로
프로그램 이동, `chart.setLegendPagination(size | false)`로 런타임
켜기/끄기도 가능하며, 항목이 줄어들면 페이지가 클램프된다.
페이지 상태는 `getState`/`setState`로 왕복된다.

## 상태 저장/복원 · 일괄 갱신

```ts
const state = chart.getState(); // type·theme·줌 창·숨김·선택 스냅
chart.setState(state);          // 다른 차트/나중에 그대로 복원

chart.batch(() => {             // 내부 변경을 notify 1회로 묶는다
  chart.addPoint("매출", 33);
  chart.updateSeries("비용", { values: [...] });
});
```

`getState`는 데이터 자체가 아닌 인터랙션 상태를 스냅한다 —
시리즈 id 기준으로 매칭되지 않는 항목은 `setState` 시 무시된다.

## 지도 (map)

`type: "map"`은 지역 폴리곤의 내부 포함(ray-casting)으로 hover/클릭을
해석한다 — `mapData`에 조인된 지역만 상호작용 대상이며, 미조인 지역은
`.mc-map-no-data`로 렌더되고 히트되지 않는다. 지역 클릭은
`seriesClick`을 발행하고 `allowPointSelect`가 켜져 있으면
`.mc-selected`로 선택된다. 툴팁 제목은 지역 이름이고 화살표 키로
지역 포커스를 이동할 수 있다. 줌·브러시·네비게이터는 적용되지 않는다.

## 계층 / 플로우 차트

`sunburst`/`treemap`/`sankey`/`chord`는 노드 단위 hitTest로
hover·클릭을 해석한다 — treemap은 리프 rect, sunburst는 극좌표
섹터(중심 거리=링, 각도=섹터), sankey는 노드 rect,
chord는 그룹 아크(링+각도)가 대상이다. sankey 리본과 chord 리본은
히트 대상이 아니다. 노드 클릭은 `seriesClick`을 발행하고
`allowPointSelect`/`selectPoint`로 `.mc-selected` 선택이 된다.
범례는 노드/그룹 단위(`slice:N`)로 토글하면 해당 서브트리·링크·
행/열이 빠지고 재배치된다. 직교 축·줌·브러시·네비게이터는
적용되지 않는다.

## 극좌표 / 게이지 차트

`nightingale`/`radial-bar`/`radial-column`/`gauge`도 극좌표 섹터(중심 거리=링,
각도=섹터)로 hover·클릭을 해석한다. nightingale 범례는 카테고리
단위로 숨긴 카테고리의 섹터가 빠지고, radial-bar/radial-column은 시리즈 단위다.
gauge는 값 아크만 히트 대상이며 범례가 없다. 세 타입 모두
`selectPoint` 선택과 키보드 포커스를 지원하고 줌·브러시·
네비게이터는 적용되지 않는다.

## 키보드 내비게이션

`.mc-root`(tabindex=0)에 포커스 후:

| 키          | 동작                               |
| ----------- | ---------------------------------- |
| ←/→         | 같은 시리즈의 이전/다음 포인트     |
| ↑/↓         | 인접 시리즈로 이동                 |
| Home/End    | 처음/마지막 포인트                 |
| Enter/Space | 포커스 포인트 클릭 (`seriesClick`) |
| Esc         | 포커스/선택/호버 해제              |

포커스된 요소는 `.mc-focused` 클래스 + `.mc-a11y`(aria-live)에
설명이 들어간다.

## 이벤트

`events` 옵션 또는 `chart.on(type, fn)`:

| 타입               | 페이로드                                          |
| ------------------ | ------------------------------------------------- |
| `ready`            | `{ snapshot }`                                    |
| `chartClick`       | `{ point }` — 빈 영역이면 null                    |
| `seriesClick`      | `{ seriesId, index, value }`                      |
| `pointDblClick`    | `{ seriesId, index, value }`                      |
| `legendToggle`     | `{ id, visible }`                                 |
| `legendHover`      | `{ id }` — 해제 시 null, 다른 시리즈 dim          |
| `visibilityChange` | `{ seriesId, visible }`                           |
| `zoomChange`       | `{ window, yWindow }` — null이면 해당 축 리셋     |
| `selectionChange`  | `{ range, indices }`                              |
| `hoverChange`      | `{ point }` — null이면 해제                       |
| `focusChange`      | `{ point }` — 키보드 포커스 이동/해제             |
| `dataChange`       | `{ seriesCount, categoryCount }`                  |
| `typeChange`       | `{ type }`                                        |
| `themeChange`      | `{ theme }`                                       |
| `loadingChange`    | `{ loading }`                                     |
| `drilldown`        | `{ key, depth }` — 하위 레벨 진입                 |
| `drillup`          | `{ depth }` — 상위 레벨 복귀                      |
| `pointSelect`      | `{ seriesId, index, value }` — allowPointSelect   |
| `pointUnselect`    | `{ seriesId, index }` — 선택 해제/교체            |
| `beforePrint`      | `{}` — window beforeprint 전달                    |
| `afterPrint`       | `{}` — window afterprint 전달                     |
| `pointUpdate`      | `{ seriesId, index, value }` — updatePoint 경유   |
| `redraw`           | `{}` — redraw() 호출 시                           |
| `resize`           | `{ width, height }` — setSize/ResizeObserver 경유 |
| `export`           | `{ format }` — svg/png/csv                        |
| `legendAllToggle`  | `{ visible }` — legend.toggleAll 전체 토글        |
| `fullscreenChange` | `{ fullscreen }` — 전체화면 진입/해제             |
| `drillupall`       | `{ depth }` — drillUpAll()로 루트 복귀            |
| `legendCheck`      | `{ id, checked }` — 범례 체크박스 토글            |
| `menuAction`       | `{ id }` — 컨텍스트 메뉴 커스텀 항목 클릭         |
| `drawEnd`          | `{ drawing }` — 주석 그리기 완료                  |
| `searchEnd`        | `{ seriesCount, categoryCount }` — 로드 완료      |
| `searchError`      | `{ message }` — 로드/변환 실패                    |
