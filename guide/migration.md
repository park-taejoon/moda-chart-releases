# 마이그레이션 매핑

다른 차트 라이브러리에서 이 프로젝트로 옮기는 사용자를 위한 개념 매핑 표.
새 기능을 추가할 때마다 이 표에도 행을 추가한다.

| 기존 개념                       | 이 프로젝트                                                                                                                              | 비고                                |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| series 배열                     | `series: [{ name, values }]`                                                                                                             | categories와 인덱스 정렬            |
| xAxis categories                | `categories: [...]`                                                                                                                      | string/number/Date                  |
| chart.type                      | `type` / `chart.setType()`                                                                                                               | 시리즈별 `series[].type` 혼합 가능  |
| legend.show / position          | `legend: { position }`                                                                                                                   | 클릭 토글 내장                      |
| tooltip.formatter               | `tooltip: { mode, format }`                                                                                                              | mode: point/shared/none             |
| dataZoom / brush                | `zoom` + `brush` 옵션                                                                                                                    | Shift+드래그가 brush                |
| zoomType: "xy"                  | `zoom: { axes: "xy", select: true }`                                                                                                     | `setZoomWindow({x,y})` 축별 제어    |
| on('click')                     | `events.seriesClick` / `chart.on("seriesClick")`                                                                                         | 이벤트 맵                           |
| color palette                   | `palette` 옵션 / `--chart-palette-*`                                                                                                     | CSS 변수로도 교체 가능              |
| theme / darkMode                | `theme` / `setTheme` / `--chart-*` 변수                                                                                                  | `[data-theme]` 선택자               |
| renderToImage / export          | `chart.toSVGString()` / `.mc-export-svg` 버튼                                                                                            | PNG는 canvas 래스터화               |
| resize / autoResize             | `responsive` (기본 on) — ResizeObserver                                                                                                  | `setSize`로 수동 제어               |
| aria / label                    | `ariaLabel` + 키보드 내비 + `.mc-a11y` 라이브 영역                                                                                       |                                     |
| label / dataLabels              | `dataLabels: true \| { format }`                                                                                                         | `.mc-data-label` 값 텍스트          |
| markLine / 기준선               | `markLines: [{ value, label }]`                                                                                                          | 값 축 기준선                        |
| markBand / plotBand / 경고지역  | `markBands: [{ from, to, label }]`                                                                                                       | 값 축 도메인 구간 `.mc-mark-band`   |
| step / spline 시리즈            | `type: "step"` / `type: "spline"`                                                                                                        | `.mc-line.mc-step`/`.mc-spline`     |
| 100% stacked                    | `type: "percent-stacked-bar"`                                                                                                            | 라벨이 % 표기                       |
| waterfall                       | `type: "waterfall"`                                                                                                                      | `.mc-bar-up`/`-down` + 연결선       |
| bubble                          | `type: "bubble"` + `points[].z`                                                                                                          | z가 반지름                          |
| heatmap                         | `type: "heatmap"` (시리즈=행, 카테고리=열)                                                                                               | `heatmap: {from,to}` 색 스케일      |
| funnel / pyramid                | `type: "funnel"` / `type: "pyramid"`                                                                                                     | 카테고리 범례 공유                  |
| addPoint / addSeries 등 실시간  | `chart.addPoint/removePoint/addSeries/removeSeries`                                                                                      | `shift` 옵션으로 창 유지            |
| drilldown                       | `drilldown: { 라벨: level }` + `chart.drillUp()`                                                                                         | `.mc-drillup` 버튼                  |
| 패턴 채우기 (색각 보조)         | `patterns: true` / `chart.setPatterns()`                                                                                                 | `<pattern class="mc-pattern">`      |
| wordcloud                       | `type: "wordcloud"` (카테고리=단어, 값=크기)                                                                                             | `.mc-word`, 축 없음                 |
| annotations / 인라인 이미지     | `annotations: [{ x, y?, label?, image? }]`                                                                                               | `.mc-annotation`                    |
| allowPointSelect / point select | `allowPointSelect: true` + `pointSelect`/`pointUnselect` 이벤트                                                                          | `.mc-selected`, `selectPoint` API   |
| lang / locale                   | `locale: "ko" \| "en"` / `chart.setLocale()`                                                                                             | 툴바·빈 상태 내장 문구              |
| beforePrint / afterPrint        | `events.beforePrint` / `afterPrint` (window print 연동)                                                                                  | `chart.print()`로 인쇄 호출         |
| radar / radar-area              | `type: "radar"` / `"radar-area"` (values 기반, 카테고리=스포크)                                                                          | `.mc-radar` + `.mc-radar-grid`      |
| range bar / range area          | `type: "range-bar"` / `"range-area"` + `series[].ranges`                                                                                 | `[low,high]` 구간 데이터            |
| candlestick / OHLC              | `type: "candlestick"` + `series[].ohlc`                                                                                                  | `[open,high,low,close]`             |
| navigator / dataZoom 미니맵     | `navigator: true \| { height }`                                                                                                          | `.mc-navigator` 창 드래그가 줌 제어 |
| errorBar / 오차 막대            | `series[].errors: [[lo,hi],…]`                                                                                                           | `.mc-error` (세로 차트)             |
| trendLine / 추세선              | `series[].trend: { type: "linear" \| "movingAverage" }`                                                                                  | `.mc-trendline` 점선                |
| setCategories / 데이터 패치     | `chart.setCategories` / `updateSeries` / `updatePoint`                                                                                   | `pointUpdate` 이벤트                |
| redraw / print                  | `chart.redraw()` / `chart.print()`                                                                                                       | redraw 이벤트 발행                  |
| title.text / subtext            | `title` / `subtitle`                                                                                                                     | `.mc-title`/`.mc-subtitle`          |
| showLoading                     | `setLoading(bool)` / `loading` 옵션                                                                                                      | `.mc-loading` 오버레이              |
| empty 데이터 문구               | `emptyText`                                                                                                                              | `.mc-empty`                         |
| on('legendselectchanged') 등    | `legendToggle`/`legendHover`/`focusChange`/`dataChange`/`typeChange`/`themeChange`/`loadingChange`/`resize`/`chartClick`/`pointDblClick` | 세분화된 이벤트 맵                  |

| map / choropleth 지도 | `type: "map"` + `series[].mapData` + `map: { topology: WORLD_TOPOLOGY }` | TopoJSON/GeoJSON, `.mc-map-region` |
| sunburst / treemap | `type` + `series[].tree` (재귀 `ChartTreeNode`) | value 생략 시 children 합계 |
| sankey | `type: "sankey"` + `series[].links` (`{source,target,value}`) | `sankey: {nodeWidth,nodeGap}` |
| chord | `type: "chord"` + `chord: {matrix, labels}` 옵션 | n×n 유향 행렬 |
| nightingale (polar area) | `type: "nightingale"` + `series[].values` | 카테고리=등분 각도, 반지름=값 |
| radial-bar | `type: "radial-bar"` | 카테고리=링, 각도=값(최대 340°) |
| radial-column | `type: "radial-column"` | 카테고리=각도 방향, 반지름=값 얇은 컬럼 |
| gauge | `type: "gauge"` + `gauge: {min,max,target}` | 값은 첫 시리즈 첫 포인트 |
| OHLC | `type: "ohlc"` + `series[].ohlc` | `[open,high,low,close]` — `.mc-ohlc-*` |
| boxplot | `type: "boxplot"` + `series[].box` | `[min,q1,median,q3,max]` — `.mc-box-*` |
| histogram | `type: "histogram"` + `histogram: {bins\|binWidth}` | 원시 values를 빈 카운트로 |
| cone-funnel | `type: "cone-funnel"` | 값 비례 단계 높이 + 수렴 외곽 |
| organization | `type: "organization"` + `series[].tree` | top-down 트리 `.mc-org-node/link` |
| linear gauge | `type: "linear-gauge"` + `gauge` 옵션 | 수평 트랙 `.mc-lg-*` |
| map markers / lines | `series[].mapPoints` / `mapLines` | 경도·위도 레이어 — `.mc-map-marker/-line` |
| 사용자 지도 핀 | `mapPoints[].image`/`pulse` + `map.cluster`/`panZoom` | 아바타 핀·펄스·클러스터 버블·지도 팬줌 |
| itemStyler / pointColor / pointName | `series[].itemStyle(ctx)` / `points[].color`·`name` | 포인트별 색·이름 오버라이드 |
| tooltip HTML 렌더러 | `tooltip: { render: (ctx) => html }` | `.mc-tooltip` innerHTML (신뢰된 마크업만) |
| context menu | `contextMenu: true \| { items }` | 플롯 우클릭 `.mc-menu`, `menuAction` 이벤트 |
| 차트 동기화 | `sync: "그룹키"` | 같은 키 차트 간 호버·줌 전파 |
| 스크롤바 | `scrollbar: true \| { height }` | 네비게이터 없는 얇은 줌 스트립 `.mc-scrollbar` |
| 범례 체크박스 | `legend: { checkboxes: true }` | `.mc-legend-check` + `legendCheck` 이벤트 |
| 상태 저장/복원 | `chart.getState()` / `setState(state)` | 타입·테마·줌·숨김·선택 왕복 |
| 일괄 갱신 | `chart.batch(fn)` | 내부 변경을 notify 1회로 묶음 |
| drillupAll | `chart.drillUpAll()` | `drillupall` 이벤트, 최상위로 일괄 복귀 |
| grouped category 축 | `categories: [["그룹","리프"],…]` | 두 번째 틱 행 `.mc-tick-group` |
| 워드클라우드 텍스트 마이닝 | `parseText(text, ignore?, max?)` | `{name,value}[]` → wordcloud 입력 |
| sparkline | `mountSparkline(el, opts)` / `sparklineMarkup(opts)` | 축·범례 없는 인라인 미니 차트 |
| 주석 그리기 도구 (AG annotations) | `drawing: true` + `chart.setDrawMode(tool)` | 플롯 드래그로 `.mc-draw-*` 생성, `drawEnd` 이벤트 |
| 범례 페이지네이션 | `legend: { pagination: pageSize }` | `.mc-legend-prev/next` 네비 |
| 터치 핀치 줌 | (zoom 옵션과 함께 자동) | 두 포인터 간격 비율로 줌 |
| IBChart `search` | `search: { url, transform? }` + `chart.search.load()` | `searchEnd`/`searchError` 이벤트 |
| 픽토그램 (ISOTYPE) | `type: "pictogram"` + `pictogram: { unit }` | 카테고리 행에 사람 아이콘, `.mc-picto-*` |
| 인구 피라미드 | `type: "population-pyramid"` + `pyramidPop: { leftSeries? }` | 첫 시리즈 좌측 미러링 대칭 막대 |
| 채널/피보나치/측정 도구 (AG annotations) | `setDrawMode("channel"\|"fibonacci"\|"measure")` | `.mc-draw-band`/`.mc-draw-fib-level` 등 |
| AG `updateDelta` | `chart.updateDelta(partialOptions)` | 키별 재귀 병합 — notify/dataChange 1회 |
| IBChart `etcData` | `series[].etcData` / `points[].etcData` | `pointSelect`·툴팁 컨텍스트·`getEtcData()` |
| 갱신 애니메이션 | `animation: { updates: true }`(기본) | 데이터 변경 후 `.mc-updating` + `.mc-morph` 보간 |
| IBChart XML 데이터 | `chart.loadXml(xml)` / `parseDataXml(xml)` | `<series>`+`<value>` 스키마 → setData |

<!-- 새 기능 행을 여기에 추가 -->
