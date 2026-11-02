# API 레퍼런스

`Chart` 인터페이스 전체와 공개 헬퍼를 기능별로 정리한다.
옵션의 상세 필드는 각 기능 가이드를 본다:

- 차트 타입·데이터 모델 — [features/chart-types.md](./features/chart-types.md)
- 축·스케일 — [features/axes-scales.md](./features/axes-scales.md)
- 인터랙션·툴팁·범례 — [features/interactions.md](./features/interactions.md)
- 테마·토큰 — [theming.md](./theming.md)
- 플러그인·대체 렌더러·lite 번들 — [plugins.md](./plugins.md)

`Chart`는 `Chart*Api` 능력 슬라이스 13개의 합집합이다
(`types/api.ts`). 모든 메서드는 모든 프레임워크 어댑터에서
동일하게 노출된다.

## 마운트 — vanilla/직접 연결

| 함수                                    | 설명                                                                           |
| --------------------------------------- | ------------------------------------------------------------------------------ |
| `mountChart(host, options)`             | 코어 생성 + 뷰 연결 + 이벤트 바인딩. `{ chart, rootEl, update, destroy }` 반환 |
| `diffOptions(prev, next)`               | 옵션 깊이 비교 — 달라진 키의 부분 패치 반환 (어댑터 prop 동기화용)             |
| `syncOptionPatch(prev, next, applied)`  | prop diff + 제자리 변이 fallback — 같은 참조면 `applied`와 비교                |
| `attachChartView(el, chart, renderer?)` | 이미 만든 `Chart`에 뷰를 붙인다 — 해제 함수 반환                               |
| `mountSparkline(host, opts)`            | 축/범례 없는 인라인 미니 차트. `{ update }` 반환                               |
| `renderInto(root, snap)`                | 스냅샷을 루트 DOM에 반영하는 저수준 렌더                                       |

`MountChartOptions` = `ChartCoreOptions` + `renderer`.
React/Vue/Svelte에서는 `<ChartView>` 컴포넌트가 같은 일을 한다
(`renderer` prop으로 동일하게 주입). 코어 생성부터 맡기는 단일
컴포넌트 `<Chart options>`도 각 어댑터가 제공한다 — 코어 생성·prop
동기화·DOM 연결·dispose가 내장되고 차트 인스턴스는
ref/expose/bind로 노출된다.

옵션 입력 실수(알 수 없는 type·형태 오류)는 생성/`setOptions` 시
`console.warn`으로 알린다 — `devWarnings: false`로 해제.
커스텀 타입 경로(builders/features 지정)는 타입 경고가 나가지 않는다.

### 렌더러 선택

| 함수                          | 설명                                                |
| ----------------------------- | --------------------------------------------------- |
| `svgRenderer`                 | 기본 내장 SVG 렌더러                                |
| `createCanvasRenderer()`      | Canvas2D 렌더러 — 조직도·지도 등 대형 화면에서 유리 |
| `createWebglRenderer()`       | WebGL 렌더러 — 대량 포인트 지오메트리를 GL로 그린다 |
| `createSvgRenderer(emitters)` | 커스텀 이미터 체인으로 SVG 렌더러 생성 (lite 경로)  |

```ts
import { createCanvasRenderer, mountChart } from "@moda-chart/core";
const m = mountChart(el, {
  renderer: createCanvasRenderer(),
  // ...options
});

// 옵션 동기화 — 마운트 옵션과 diff해 달라진 키만 반영한다.
// next에서 빠진 키는 해제되므로 "목표 상태 전체"를 넘긴다.
// 부분 병합이 목적이면 chart.setOptions(patch)를 쓴다.
m.update({ type: "bar", series: nextSeries /* ...나머지 옵션 */ });
```

**WebGL 렌더러 노트** — `createWebglRenderer()`는 두 장의 캔버스를 겹친다:
아래 `canvas.mc-canvas`(WebGL)가 마크밴드·그리드와 직교 지오메트리
(line/step/spline/scatter/bubble/bar/range-bar/heatmap)를 GL 프리미티브로
그리고, 위 `canvas.mc-canvas-2d`(Canvas2D)가 축·라벨·주석과 GL 미지원
패밀리(area/path 채움·candlestick·슬라이스·플로우·맵 등)를 그린다.
WebGL 컨텍스트를 못 얻으면 오버레이가 전체를 그리는 Canvas2D 폴백으로
자동 전환되며 DOM·인터랙션 계약은 동일하다. 라운드 조인/캡과
SVG export(toMarkup)는 미지원 — 내보내기는 코어 `toSVGString` 경로를 쓴다.

## 수명주기 — `ChartBaseApi`

| 메서드              | 설명                                           |
| ------------------- | ---------------------------------------------- |
| `instanceId`        | 인스턴스 고유 id                               |
| `subscribe(fn)`     | 스냅샷 변경 구독 — 반환된 함수로 해제          |
| `on(type, handler)` | 이벤트 구독 — 반환된 함수로 해제               |
| `getSnapshot()`     | 캐시된 `ChartSnapshot` — 변경 전까지 같은 참조 |
| `dispose()`         | sync 그룹에서 제거                             |
| `revive()`          | dispose 복구 — StrictMode effect 재실행 대응   |
| `use(feature)`      | 피처 설치 — [plugins.md](./plugins.md)         |

## 데이터 — `ChartDataApi`

| 메서드                                  | 설명                                              |
| --------------------------------------- | ------------------------------------------------- |
| `setData(series, categories?)`          | 시리즈/카테고리 교체                              |
| `addSeries(input)` / `removeSeries(id)` | 시리즈 추가/제거                                  |
| `addPoint(seriesId, point, {shift, category}?)` | 실시간 포인트 추가 — `{x,y}` 또는 `[x,y]` 튜플. `shift` + 명시 카테고리면 라벨도 함께 밀고 꼬리에 `category` 부착(생략 시 숫자 증분) |
| `removePoint(seriesId, index)`          | 포인트 제거                                       |
| `updateSeries(id, patch)`               | 시리즈 속성 부분 갱신 — 없으면 false              |
| `updatePoint(seriesId, index, value)`   | 포인트 값 갱신 — `pointUpdate` 발행, 없으면 false |
| `setCategories(cats)`                   | 카테고리 배열 교체                                |
| `updateDelta(patch)`                    | 옵션/데이터 부분 병합 — `undefined` 값은 키 해제  |
| `loadXml(xml)`                          | IBChart XML 데이터 로드                           |
| `getEtcData(seriesId, index?)`          | `etcData` 부가 데이터 조회                        |
| `getSeries()`                           | 시리즈 요약 — id 기반 API의 해석된 id 조회        |
| `getCategories()`                       | 현재 카테고리 배열                                |
| `batch(fn)`                             | 여러 변경을 묶어 무효화 1회로                     |
| `redraw()`                              | 강제 재렌더                                       |

## 상태 — `ChartStateApi`

| 메서드            | 설명                                         |
| ----------------- | -------------------------------------------- |
| `getState()`      | 가시성·줌·선택·조직도 접힘 스냅              |
| `setState(state)` | `getState` 스냅 복원 — 없는 시리즈 id는 무시 |

## 설정 — `ChartConfigApi`

| 메서드                                       | 설명                                                       |
| -------------------------------------------- | ---------------------------------------------------------- |
| `getOptions()`                               | 현재 옵션 (읽기 전용)                                      |
| `setOptions(patch)`                          | `updateDelta`의 별칭 — 부분 병합, `undefined` 값은 키 해제 |
| `setType(type)`                              | 차트 타입 전환                                             |
| `setTheme(theme)`                            | `"light"`/`"dark"`/객체 테마                               |
| `setSize(w, h?)`                             | 크기 변경 — `resize` 이벤트                                |
| `setPatterns(bool)`                          | 패턴 채움 토글                                             |
| `setLocale(locale)`                          | 내장 문구 언어                                             |
| `setLoading(bool)`                           | 로딩 오버레이                                              |
| `isResponsive()`                             | 리사이즈 관찰 활성 여부                                    |
| `setFullscreen(bool)` / `toggleFullscreen()` | 전체화면                                                   |

## 범례 — `ChartLegendApi`

| 메서드                                 | 설명                               |
| -------------------------------------- | ---------------------------------- |
| `toggleLegend(id)`                     | 항목 토글 — `"slice:N"`은 슬라이스 |
| `setSeriesVisible(id, visible)`        | 가시성 직접 설정                   |
| `legendCheck(id)`                      | 체크박스 토글 — `legendCheck` 발행 |
| `hoverLegend(id \| null)`              | hover 강조                         |
| `setLegendPage(n)` / `getLegendPage()` | 페이지네이션                       |
| `setLegendPagination(bool \| n)`       | 런타임 켜기/끄기                   |
| `setLegend(bool \| opts)`              | 범례 옵션 전체 교체                |
| `toggleAllLegend()`                    | 전체 표시/숨김                     |

## 인터랙션 — `ChartInteractionApi`

| 메서드                                         | 설명                                                                |
| ---------------------------------------------- | ------------------------------------------------------------------- |
| `hoverAt(px, py)` / `clearHover()`             | 플롯 상대 px 호버                                                   |
| `clickAt(px, py)` / `dblClickAt(px, py)`       | 클릭/더블클릭 — `chartClick`은 항상 발행                            |
| `selectPoint(seriesId, i)` / `clearSelected()` | 포인트 선택                                                         |
| `openMenu(px, py)` / `closeMenu()`             | 컨텍스트 메뉴                                                       |
| `activateMenuItem(id)`                         | 메뉴 항목 활성화 — 커스텀은 `menuAction`                            |
| `moveFocus(dir)` / `clearFocus()`              | 키보드 포인트 내비 — `left/right/up/down` + `first/last`(양끝 점프) |
| `escape()`                                     | 호버·선택·메뉴·드로잉·핀툴팁·전체화면 해제                          |
| `unpinTooltip()`                               | 고정 툴팁 해제                                                      |

## 줌·팬 — `ChartZoomApi`

| 메서드                                     | 설명                                |
| ------------------------------------------ | ----------------------------------- |
| `zoomAt(px, factor, py?)`                  | 앵커 줌 — factor>1 확대             |
| `panByPx(dx, dy?)`                         | px만큼 팬                           |
| `setZoomWindow(w)` / `resetZoom()`         | 줌 창 설정/리셋                     |
| `startDrag` / `dragTo` / `endDrag`         | 드래그 수명주기 (shift로 모드 반전) |
| `setSelection(range)` / `clearSelection()` | 선택 범위                           |
| `navDragStart/To/End` · `navJump(px)`      | 네비게이터 조작                     |
| `navKey(part, key)`                        | 키보드 네비게이터 — 창/핸들 슬라이더 |
| `applyZoomPreset(i)`                       | `zoom.presets` 버튼 적용            |

## 드릴다운 — `ChartDrilldownApi`

`drillDown(key)` · `drillUp()` · `drillUpAll()`.

## 그리기 — `ChartDrawingApi`

`setDrawMode(tool)` · `drawStartAt/MoveTo/EndAt` · `drawCancel()`
· `addDrawing(d)` · `getDrawings()` · `clearDrawings()`.

## 지도 — `ChartMapApi`

`mapZoomAt(px, factor, py?)` — map 타입 팬/줌 변환.

## 조직도 — `ChartOrgApi`

노드 인덱스는 DFS 플랫 인덱스(노드 카드의 `data-index`).
자세한 옵션은 [chart-types.md §조직도](./features/chart-types.md).

| 메서드                                                  | 설명                                             |
| ------------------------------------------------------- | ------------------------------------------------ |
| `toggleOrgNode(i)` / `setOrgNodeCollapsed(i, f)`        | 하위 트리 접기/펼치기 — `orgToggle`              |
| `expandOrgAll()` / `isOrgNodeCollapsed(i)`              | 전체 펼침 / 상태 조회                            |
| `addOrgNode(parent, node)`                              | 노드 추가 — parent `-1`이면 새 루트              |
| `removeOrgNode(i)`                                      | 노드+하위 트리 제거                              |
| `moveOrgNode(i, newParent)`                             | 리페어런팅 — `orgMove`. 사이클/동일 부모는 no-op |
| `beginOrgDragAt` / `updateOrgDrag` / `endOrgDrag`       | `editable` DnD 리페어런팅                        |
| `searchOrg(q)` / `searchOrgNext()` / `clearOrgSearch()` | 검색 — `orgSearch`                               |
| `orgMinimapPan(mx, my)`                                 | 미니맵 좌표로 팬 (뷰 중앙으로)                  |
| `orgMinimapNudge(dx, dy)`                               | 미니맵 viewBox 단위 누진 팬 (키보드 이동)        |

## 외부 데이터 — `ChartSearchApi`

`chart.search.load(args)` — 서버 데이터 로딩
(`searchEnd`/`searchError` 이벤트).

##보내기 — `ChartExportApi`

| 메서드             | 설명                                       |
| ------------------ | ------------------------------------------ |
| `toSVGString()`    | 스냅샷 → 독립 SVG 마크업                   |
| `toCSV()`          | 현재 데이터 → CSV 문자열                   |
| `exportBasename()` | `exporting.filename` 또는 `"chart"`        |
| `print()`          | 브라우저 인쇄 — `beforePrint`/`afterPrint` |

DOM 헬퍼(번들 전용): `downloadSVG(chart)`, `downloadPNG(chart)`
(2배 해상도, async), `downloadCSV(chart)` — 파일명은
`exportBasename()`이 기본값.

## 데이터 헬퍼

| 함수                                 | 설명                                                                             |
| ------------------------------------ | -------------------------------------------------------------------------------- |
| `listToTree(items)`                  | `{id,parentId}` 플랫 배열 → `ChartTreeNode[]`. 고아·중복 id·사이클은 루트로 승격 |
| `normalizeTree(nodes)`               | 트리 입력 정규화 — `TreeDatum`(경로·깊이 부여)                                   |
| `flattenTree(roots)`                 | `TreeDatum[]`을 DFS 플랫 배열로 펼침                                             |
| `resolveTreeInput({tree, treeList})` | `tree` 우선, 없으면 `treeList`를 `listToTree`로 변환                             |

`series[].treeList`만 써도 코어가 자동 변환하므로 `listToTree`를
직접 부를 필요는 보통 없다 — 편집 후 되돌리거나 사전 검증용.

## 이벤트 — `chart.on(type, handler)`

| 이벤트                                                             | 페이로드 요약                                    |
| ------------------------------------------------------------------ | ------------------------------------------------ |
| `ready`                                                            | `{ snapshot }` — 첫 마운트                       |
| `dataChange` / `typeChange` / `themeChange`                        | 변경 요약                                        |
| `seriesClick`                                                      | `{ seriesId, index, value, url? }`               |
| `chartClick`                                                       | `{ point }` — 빈 영역이면 null                   |
| `pointDblClick`                                                    | `{ seriesId, index, value }`                     |
| `pointSelect` / `pointUnselect`                                    | `{ seriesId, index, value, etcData? }`           |
| `pointUpdate` / `redraw`                                           | 값 변경 / 재렌더                                 |
| `hoverChange` / `focusChange`                                      | `{ point }`                                      |
| `zoomChange`                                                       | `{ window, yWindow }`                            |
| `selectionChange`                                                  | `{ range, indices }`                             |
| `legendToggle` / `legendHover` / `legendCheck` / `legendAllToggle` | 범례 상호작용                                    |
| `menuAction`                                                       | `{ id, index? }` — 커스텀 메뉴 항목              |
| `drilldown` / `drillup` / `drillupall`                             | `{ key?, depth }`                                |
| `drawEnd`                                                          | `{ drawing }`                                    |
| `searchEnd` / `searchError`                                        | `search.load` 결과                               |
| `orgToggle`                                                        | `{ index, collapsed, name? }`                    |
| `orgMove`                                                          | `{ index, name?, from, to }` — 이동 후 새 인덱스 |
| `orgSearch`                                                        | `{ query, count }`                               |
| `clusterSelect`                                                    | `{ seriesId, members, points }` — 지도 클러스터  |
| `export`                                                           | `{ format }`                                     |
| `resize` / `loadingChange` / `fullscreenChange`                    | 크기/로딩/전체화면                               |
| `beforePrint` / `afterPrint`                                       | 인쇄 전후                                        |
| `visibilityChange`                                                 | `{ seriesId, visible }`                          |
| `seriesAfterAnimate`                                               | `{ updating }` — 진입/갱신 트랜지션 종료         |
