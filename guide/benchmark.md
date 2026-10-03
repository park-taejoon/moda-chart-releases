# 벤치마크 — moda-chart vs AG Charts vs IBChart

공식 문서 기준 기능 비교. 실사용 측정(성능)은 포함하지 않는다.

- **moda-chart** — 이 저장소. 헤드리스 코어 + 5개 렌더러 어댑터, MIT
- **AG Charts** — ag-grid.com. Community(MIT, 기본 7종) / Enterprise(상용,
  나머지 타입·기능). v14 기준
- **IBChart** — ibsheet.com (IBSoft). 상용, IBSheet/IBTab 제품군의 차트
  컴포넌트. IBChart(H) v7 API 기준

기호: ✅ 기본 지원 / 💰 Enterprise·옵션 등 조건부 / ❌ 없음 / △ 다른 방식으로
부분 대응

## 차트 타입

| 타입                           | moda-chart                                | AG Charts                     | IBChart                  |
| ------------------------------ | ----------------------------------------- | ----------------------------- | ------------------------ |
| line                           | ✅                                        | ✅                            | ✅                       |
| step                           | ✅ `step`                                 | △ line `interpolation: step`  | △ (계단형 소개에 언급)   |
| spline                         | ✅                                        | △ line `interpolation`        | ✅                       |
| area                           | ✅                                        | ✅                            | ✅                       |
| bar / column                   | ✅ `bar`                                  | ✅                            | ✅ bar + column          |
| horizontal bar                 | ✅ `direction`                            | ✅                            | ✅ `inverted`            |
| stacked                        | ✅ `stacked-bar`                          | ✅ `stacked`                  | ✅                       |
| percent-stacked                | ✅                                        | ✅ `normalizedTo: 100`        | △ stack 옵션             |
| range-bar                      | ✅                                        | 💰                            | ✅ `columnrange`         |
| range-area                     | ✅                                        | 💰                            | ✅ `arearange`           |
| waterfall                      | ✅                                        | 💰                            | ✅                       |
| candlestick                    | ✅                                        | 💰                            | ❌                       |
| OHLC                           | ✅ `ohlc`                                 | 💰                            | ❌                       |
| boxplot                        | ✅                                        | 💰                            | ✅                       |
| histogram                      | ✅ `histogram`+`bins`/`binWidth`          | 💰                            | ❌                       |
| errorbar (타입)                | △ `errors` 오버레이                       | 💰 error bars 기능            | ✅ `errorbar`            |
| pie / donut                    | ✅                                        | ✅                            | ✅ (pie `innerSize`)     |
| funnel / pyramid / cone-funnel | ✅                                        | 💰                            | ✅                       |
| radar / radar-area             | ✅                                        | 💰 radar-line/area            | △ `polar` 옵션           |
| scatter                        | ✅                                        | ✅                            | ✅                       |
| bubble                         | ✅                                        | ✅                            | ✅                       |
| heatmap                        | ✅                                        | 💰                            | ✅                       |
| wordcloud                      | ✅                                        | ❌                            | ✅ (+`parseText` 마이닝) |
| map (choropleth)               | ✅                                        | 💰 map-shape                  | ❌                       |
| map marker/line                | ✅ `mapPoints`/`mapLines`                 | 💰                            | ❌                       |
| map 아바타/클러스터/팬줌       | ✅ `image`/`pulse`/`cluster`/`panZoom`    | ❌                            | ❌                       |
| sunburst                       | ✅                                        | 💰                            | ❌                       |
| treemap                        | ✅                                        | 💰                            | ❌                       |
| sankey                         | ✅                                        | 💰                            | ❌                       |
| chord                          | ✅                                        | 💰                            | ❌                       |
| nightingale                    | ✅                                        | 💰                            | ❌                       |
| radial-bar/column              | ✅                                        | 💰                            | ❌                       |
| gauge                          | ✅ + `linear-gauge`                       | 💰 radial + linear gauge      | ✅                       |
| organization                   | ✅                                        | 💰                            | ❌                       |
| pictogram (사람 아이콘)        | ✅ `pictogram`+`unit`                     | ❌                            | ❌                       |
| population pyramid             | ✅ `population-pyramid` (±대칭 수평 막대) | △ 수평 bar+음수 시리즈로 수동 | △ 대칭 bar 구성 가능     |
| 조합(mixed series)             | ✅ `series[].type`                        | ✅                            | ✅ `seriesType`          |
| 타입 수 (합계)                 | 36                                        | ~35 (E 전부)                  | ~18                      |

## 인터랙션

| 기능               | moda-chart                                           | AG Charts                     | IBChart                         |
| ------------------ | ---------------------------------------------------- | ----------------------------- | ------------------------------- |
| 줌 (휠/드래그)     | ✅ x·y·xy, 영역 선택 줌                              | 💰 x/y/xy + 앵커·키 옵션      | ✅ `zoomType` x/y/xy            |
| 팬                 | ✅ 드래그                                            | 💰                            | ✅                              |
| 네비게이터(미니맵) | ✅ 창 드래그/핸들 리사이즈                           | 💰 미니 차트 포함             | ❌                              |
| 스크롤바           | ✅ `scrollbar` (미니맵 없는 스트립)                  | 💰                            | ❌                              |
| 브러시 선택        | ✅ `selectionChange`                                 | 💰 (영역 줌과 공유)           | ✅ `selection` 이벤트           |
| 드릴다운           | ✅ `drilldown` 맵 + `‹ Back` + `drillUpAll()`        | ❌ (click 이벤트로 직접 구현) | ✅ drilldown/drillup/drillupAll |
| 포인트 선택        | ✅ `allowPointSelect`                                | ✅ 노드 클릭 이벤트           | ✅ `allowPointSelect`           |
| 크로스헤어         | △ shared 툴팁 모드의 가이드선                        | 💰 + band highlight           | ✅                              |
| 컨텍스트 메뉴      | ✅ `contextMenu` (내장 reset/export + 커스텀 항목)   | 💰 (우클릭 메뉴·커스텀)       | ❌                              |
| 주석 그리기 도구   | ✅ `drawing` + `setDrawMode` (line/hline/vline/rect) | 💰 추세선/채널/피보나치/측정  | ❌                              |
| 차트 간 동기화     | ✅ `sync` 그룹 (호버/줌 전파)                        | 💰 sync                       | ❌                              |
| 키보드 내비게이션  | ✅ 화살표/Enter/Esc                                  | ✅                            | ✅ 접근성 키보드 이동           |
| 터치 제스처        | ✅ 핀치 줌 (포인터 이벤트)                           | ✅                            | △                               |
| 상태 저장/복원     | ✅ `getState`/`setState`                             | ✅ getState/setState          | △ getOptions/getData            |

## 데이터 · API

| 기능                | moda-chart                                 | AG Charts                                | IBChart                                |
| ------------------- | ------------------------------------------ | ---------------------------------------- | -------------------------------------- |
| 데이터 교체         | `setData`/`setCategories`                  | `update`/`updateDelta`                   | `setOptions`/`setData`                 |
| 시리즈 추가/삭제    | `addSeries`/`removeSeries`                 | ✅                                       | `addSeries`/`removeSeries`/`removeAll` |
| 포인트 추가/삭제    | `addPoint`(`shift`)/`removePoint`          | △ 데이터 재지정                          | `addPoint`/`removePoint`               |
| 부분 갱신           | `updateSeries`/`updatePoint`/`redraw`      | ✅ delta 업데이트                        | ✅ 시리즈 단위                         |
| 델타 병합           | ✅ `updateDelta(partial)` — 키별 재귀 병합 | ✅ `updateDelta`                         | —                                      |
| 일괄 갱신           | ✅ `batch(fn)` — notify 1회로 묶음         | ✅ `updateDelta`                         | ❌                                     |
| 부가 데이터         | ✅ `etcData` (시리즈/포인트)               | ✅ datum 그대로 참조                     | ✅ `etcData`                           |
| 포인트별 색/이름    | ✅ `points[].color`/`name`                 | ✅ 아이템 styler                         | ✅ `pointName`/`pointColor`            |
| 스타일러 콜백       | ✅ `series.itemStyle`                      | ✅ `itemStyler`                          | △ 색상 옵션                            |
| 워드클라우드 텍스트 | ✅ `parseText` 빈도 추출                   | —                                        | ✅ `parseText` 빈도 추출               |
| 시리즈별 타입 혼합  | ✅                                         | ✅                                       | ✅                                     |
| 다중 값 축          | ✅ `yAxis[]` (left/right)                  | ✅ secondary axes                        | ✅                                     |
| 축 스케일           | band/linear/time/log                       | category/grouped/number/log/time/ordinal | ✅                                     |
| 중첩 카테고리 축    | ✅ `["그룹","리프"]` 카테고리 튜플         | ✅ grouped category                      | ❌                                     |
| 기준선/기준영역     | ✅ `markLines`/`markBands`                 | ✅ crossLines                            | ✅ plotLine/plotBand                   |
| 추세선              | ✅ `series.trend` (linear/이동평균)        | 💰 주석 그리기(수동)                     | ❌                                     |
| 에러바              | ✅ `series.errors`                         | 💰                                       | ✅ errorbar 타입                       |
| 외부 데이터 로딩    | ✅ `chart.search.load()`                   | —                                        | ✅ `search` 네임스페이스               |

## 표시 · 테마 · 접근성

| 기능                  | moda-chart                               | AG Charts               | IBChart               |
| --------------------- | ---------------------------------------- | ----------------------- | --------------------- |
| 테마                  | `light`/`dark` + CSS 변수                | ✅ 테마 빌더/팔레트     | ✅ 색상 옵션          |
| 패턴 채우기(색각)     | ✅ `patterns`                            | ✅ pattern fill         | ✅ 패턴 표시          |
| 데이터 라벨           | ✅ + 포맷터                              | ✅                      | ✅                    |
| 툴팁 커스텀           | ✅ `format` + `render` HTML              | ✅ HTML 렌더러          | ✅                    |
| 인라인 주석           | ✅ 텍스트/이미지                         | 💰 (그리기 도구와 통합) | ✅ 이미지/데이터 표시 |
| 범례                  | ✅ 위치·토글·hover·체크박스·페이지네이션 | ✅ + 페이지네이션       | ✅ + 체크박스         |
| 연속값 범례           | △ map 전용 `.mc-map-scale`               | ✅ gradient legend      | ✅                    |
| 로케일                | ✅ ko/en (`setLocale`)                   | ✅ i18n                 | ✅ `IBChart.lang`     |
| 로딩/빈 상태 오버레이 | ✅                                       | ✅ overlays             | ✅                    |
| 애니메이션            | △ 진입 트랜지션만                        | 💰 (업데이트 포함)      | ✅                    |
| 스크린리더            | ✅ aria + 라이브 영역                    | ✅                      | ✅                    |
| 반응형                | ✅ ResizeObserver                        | ✅                      | △ `setSize`           |

## 보내기 · 출력

| 기능            | moda-chart           | AG Charts               | IBChart              |
| --------------- | -------------------- | ----------------------- | -------------------- |
| SVG 문자열      | ✅ `toSVGString`     | ✅ `getImageDataURL` 등 | ✅ `getSVGString`    |
| 이미지 다운로드 | ✅ SVG/PNG           | ✅ (컨텍스트 메뉴)      | ✅ `down2Image`      |
| 인쇄            | ✅ `print` + 이벤트  | ✅                      | ✅ `doPrint`         |
| 인쇄 이벤트     | ✅ before/afterPrint | ❌                      | ✅ before/afterPrint |

## 이벤트 대응표 (IBChart → moda-chart)

| IBChart                            | moda-chart                          |
| ---------------------------------- | ----------------------------------- |
| `click`/`pointClick`               | `chartClick`/`seriesClick`          |
| `pointMouseOver/Out`               | `hoverChange`                       |
| `pointSelect/Unselect`             | `pointSelect`/`pointUnselect`       |
| `pointUpdate`                      | `pointUpdate`                       |
| `seriesClick`/`LegendItemClick`    | `seriesClick`/`legendToggle`        |
| `seriesShow`/`Hide`                | `visibilityChange`                  |
| `selection`                        | `selectionChange`                   |
| `zoom`                             | `zoomChange`                        |
| `drilldown`/`drillup`/`drillupAll` | `drilldown`/`drillup`/`drillupall`  |
| `before/afterPrint`                | `beforePrint`/`afterPrint`          |
| `redraw`                           | `redraw`                            |
| `seriesCheckBoxClick`              | `legendCheck` (`legend.checkboxes`) |
| `seriesAfterAnimate`               | ❌ (진입 트랜지션만)                |
| `searchEnd`                        | `searchEnd` (`chart.search.load`)   |
| — (그리기 완료)                    | `drawEnd`                           |

## 구조적 차이

| 항목       | moda-chart                     | AG Charts              | IBChart                |
| ---------- | ------------------------------ | ---------------------- | ---------------------- |
| 렌더링     | SVG만                          | Canvas 기반 (+SVG?)    | SVG 기반               |
| 구조       | 헤드리스 코어 + 얇은 어댑터    | 모놀리스 + 모듈 시스템 | 모놀리스               |
| 프레임워크 | React/Vue3/Vue2/Svelte/vanilla | React/Vue/Angular/TS   | vanilla (SPA 가능)     |
| 라이선스   | MIT                            | MIT + 상용 Enterprise  | 상용                   |
| 상태 모델  | `getSnapshot()` 불변 스냅샷    | options 객체 + delta   | JSON 데이터 인터페이스 |
| 크기       | 코어 ~수십KB                   | Community 수백KB~      | 상용 번들              |

## 갭 분석 — moda-chart 기준

이번 갭 개발로 아래 항목을 모두 구현했다 (전부 무료 코어 기능):

- **신규 타입 6종** — `boxplot`, `histogram`(`histogram.bins`/`binWidth`),
  `ohlc`, `cone-funnel`, `organization`, `linear-gauge`
- **포인트 스타일** — `points[].color`/`name` + `series.itemStyle` 콜백
  (AG `itemStyler` 해당)
- **툴팁 HTML** — `tooltip.render(ctx) → string` (`.mc-tooltip` innerHTML,
  신뢰된 마크업만)
- **지도 레이어** — `mapPoints` 마커(`.mc-map-marker`) + `mapLines`
  연결선(`.mc-map-line`), 지역과 같은 투영 공유
- **인터랙션** — `contextMenu`(우클릭 `.mc-menu`, `menuAction`),
  `scrollbar`(미니맵 없는 줌 스트립), `sync` 그룹(호버·줌 전파),
  `legend.checkboxes`(`.mc-legend-check`, `legendCheck` 이벤트)
- **상태/일괄** — `getState`/`setState`(줌·숨김·선택 왕복), `batch(fn)`
  (notify 1회), `drillUpAll()`(`drillupall` 이벤트)
- **축** — `[그룹, 리프]` 카테고리 튜플 → 두 번째 틱 행(`.mc-tick-group`)
- **유틸** — `parseText` 워드 빈도 추출(워드클라우드 입력용),
  `mountSparkline`/`sparklineMarkup` 인라인 미니 차트

갭 2라운드로 추가된 항목:

- **주석 그리기 도구** — `drawing` 옵션 + `chart.setDrawMode()`
  (`line`/`hline`/`vline`/`rect`), 플롯 드래그로 데이터 좌표 주석 생성
  (`.mc-draw-*`), `drawEnd` 이벤트, `getState` 왕복, `clearDrawings`
- **범례 페이지네이션** — `legend.pagination`(pageSize) +
  `.mc-legend-prev/next` 네비
- **터치 핀치 줌** — 두 포인터 간격 비율로 중점 앵커 줌
  (`.mc-overlay`의 `touch-action:none`)
- **외부 데이터 로딩** — `search` 옵션 + `chart.search.load()`,
  `searchEnd`/`searchError` 이벤트 (IBChart `search` 해당)

갭 3라운드로 추가된 항목:

- **픽토그램** — `type: "pictogram"`, 카테고리 행마다 사람 아이콘을
  값 비례로 채우는 ISOTYPE 차트 (`.mc-picto-fill`/`.mc-picto-empty`,
  `pictogram.unit`/`maxPerRow`). AG Charts/IBChart 모두 없는 타입
- **인구 피라미드** — `type: "population-pyramid"`, 첫 시리즈를
  중앙축 좌측으로 미러링한 ±대칭 수평 막대 (`pyramidPop.leftSeries`)

갭 4라운드로 추가된 항목:

- **고급 그리기 도구** — `setDrawMode("channel"|"fibonacci"|"measure")`.
  채널은 추세선+평행선+음영 밴드(`.mc-draw-band`, `drawing.channelWidth`),
  피보나치는 되돌림 7레벨(`.mc-draw-fib-level`), 측정은 Δx/Δy 자동 라벨.
  AG Charts에서는 Enterprise 전용 도구
- **`updateDelta`** — `chart.updateDelta(partial)`가 옵션/데이터의
  델타를 키별 재귀 병합해 notify·dataChange를 1회로 묶는다
  (AG Charts `updateDelta` 해당)
- **갱신 애니메이션** — `animation.updates`(기본 true) — 데이터 변경
  후 첫 스냅샷에 `.mc-updating`이 붙어 `mc-update` 전환이 재생된다.
  실제 지오메트리 morph가 아닌 클래스 트리거 전환이다
- **`etcData`** — `series[].etcData`/`points[].etcData`가
  `pointSelect`/툴팁 컨텍스트/`getEtcData()`로 전달된다
  (IBChart `etcData` 해당)

갭 5라운드로 추가된 항목:

- **XML 데이터 인터페이스** — `chart.loadXml(xml)` /
  `parseDataXml(xml)`. `<series>`+`<value>`/속성 목록 스키마를
  해석해 `setData`에 주입 (IBChart XML 데이터 해당)
- **지오메트리 morph 전환** — `renderInto`가 데이터 갱신 시
  재렌더 전후의 `.mc-bar`/`.mc-point`를 키(tag:series:index)로
  매칭해 translate/scale을 보간한다 (`.mc-morph`,
  `animation.updates` 경로). path 계열(선/슬라이스)은 키가 없어
  `.mc-updating` 페이드만 적용

갭 6라운드로 추가된 항목 (도식 사용자 지도 — 어느 경쟁사에도 없는
moda-chart 고유 영역):

- **아바타 핀** — `mapPoints[].image` → 원형 클립 이미지 마커
  (`.mc-map-avatar`), 프로필 사진 핀 용도
- **내 위치 펄스** — `mapPoints[].pulse` → 펄스 링 애니메이션
  (`.mc-map-pulse`)
- **마커 클러스터링** — `map: { cluster: true | { size } }` →
  화면 px 셀 집계 + 카운트 버블(`.mc-map-cluster`), 클릭은
  `clusterSelect` 이벤트로 묶인 포인트 전달
- **지도 팬/줌** — `map: { panZoom: true }` → 지역/라인 그룹
  transform + 마커 좌표 변환(핀 크기 일정), 휠·드래그·핀치,
  `resetZoom`/더블클릭 원복

**여전히 남은 갭**

- IBChart XML 스키마의 포인트별 속성(색상/링크) 확장
- path 계열 morph — 선 경로 d 보간은 현재 미지원

**moda-chart가 앞선 부분**

- 드릴다운 1급 지원 + `drillUpAll` (AG Charts는 수동 구현 필요)
- 워드클라우드 + `parseText` (AG Charts 없음), step 전용 타입
- 5개 프레임워크 동일 DOM 계약 + 컨포먼스 스펙 공유
- 무료(MIT)로 map/sunburst/treemap/sankey/chord/gauge/레이더/
  boxplot/histogram/ohlc/organization/linear-gauge 제공
  — AG Charts에서는 전부 Enterprise
