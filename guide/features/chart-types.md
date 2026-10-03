# 차트 타입과 데이터 모델

## 데이터 모델

```ts
new ChartCore({
  type: "line",
  categories: ["1월", "2월", "3월"], // x축 — string=band, number=linear, Date=time
  series: [
    { name: "매출", values: [12, 24, 16] },
    { name: "비용", values: [8, 14, 12], color: "#f43f5e" },
  ],
});
```

- `series[].values` — `categories`와 인덱스로 정렬되는 값 배열
- `series[].points` — `{ x, y, z?, color?, name? }` 명시 좌표
  (scatter/시계열). `color`는 포인트 고정색, `name`은 라벨 대체 이름
- `series[].ranges` — `[low, high][]` 구간 데이터 (range-bar/range-area)
- `series[].ohlc` — `[open, high, low, close][]` 봉 데이터
  (candlestick/ohlc)
- `series[].box` — `[min, q1, median, q3, max][]` 5수치 (boxplot)
- `series[].itemStyle` — `(ctx) => { color?, name? }` 콜백으로
  포인트별 색/이름 오버라이드 (AG `itemStyler` 해당)
- `series[].mapPoints`/`mapLines` — map 타입의 경도·위도
  마커/연결선 레이어
- `series[].errors` — `[하한 오차, 상한 오차][]` 에러바 오버레이
- `series[].trend` — `{ type: "linear" | "movingAverage", period? }` 추세선
- `series[].color` — 고정 색. 없으면 `--chart-palette-*` 순환
- `getSeriesId(series, i)` — 시리즈 고유 ID (기본 `name ?? series-N`).
  이벤트·범례·포커스의 식별자가 된다
- `categories` 생략 시 points의 x에서 추출, 없으면 0..N-1 선형 축.
  `[그룹, 리프]` 튜플은 그룹 카테고리 축(두 번째 틱 행)을 만든다

런타임에 `chart.setData(series, categories)`로 데이터 교체 —
동일 참조 재호출은 무시한다(멱등).

## 차트 타입

| `type`                | geometry                             | 비고                                  |
| --------------------- | ------------------------------------ | ------------------------------------- |
| `line`                | `.mc-line`+`.mc-point`               | 포인트 hover/포커스                   |
| `step`                | `.mc-line.mc-step`+`.mc-point`       | 계단형 (middle step)                  |
| `spline`              | `.mc-line.mc-spline`+`.mc-point`     | Catmull-Rom 곡선                      |
| `area`                | `.mc-area`+`.mc-line`                | linePath는 윤곽선                     |
| `bar`                 | `.mc-bar`                            | `direction: "horizontal"`로 수평      |
| `stacked-bar`         | `.mc-bar`                            | 시리즈를 누적                         |
| `percent-stacked-bar` | `.mc-bar`                            | 카테고리별 점유율(합계=100%)          |
| `waterfall`           | `.mc-bar`+`.mc-waterfall-line`       | 누적 델타, `.mc-bar-up`/`-down`       |
| `pie`/`donut`         | `.mc-slice`                          | 첫 시리즈의 카테고리별 비율           |
| `funnel`/`pyramid`    | `.mc-slice`                          | 카테고리별 세로 밴드                  |
| `range-bar`           | `.mc-range-bar`                      | `ranges`의 [low,high] 구간 막대       |
| `range-area`          | `.mc-range-area`                     | 상한선+하한선 사이 밴드               |
| `candlestick`         | `.mc-candle`+`.mc-candle-wick`       | `ohlc` 봉, `.mc-candle-up`/`-down`    |
| `ohlc`                | `.mc-ohlc`(stem/open/close)          | `ohlc` 데이터, 캔들 대신 틱 선분      |
| `boxplot`             | `.mc-box`(body/whisker/cap/median)   | `box` 5수치 요약                      |
| `histogram`           | `.mc-bar`                            | 원시 values → `histogram.bins` 구간화 |
| `cone-funnel`         | `.mc-slice`                          | 값 비례 단계 높이, 수렴 외곽          |
| `organization`        | `.mc-org-node`+`.mc-org-link`        | `tree` 계층, top-down 박스+커넥터     |
| `radar`/`radar-area`  | `.mc-radar`(+`.mc-radar-area`)       | 극좌표 폴리곤, `.mc-radar-grid`       |
| `scatter`             | `.mc-point`                          | points 또는 values(index→x)           |
| `bubble`              | `.mc-bubble`                         | `points[].z`가 반지름                 |
| `heatmap`             | `.mc-heat-cell`                      | 시리즈=행, 카테고리=열                |
| `wordcloud`           | `.mc-word`                           | 카테고리=단어, 값=font 크기           |
| `map`                 | `.mc-map-region`                     | `mapData`+`map.topology` choropleth   |
| `sunburst`            | `.mc-sunburst-node`                  | `tree` 계층, 깊이 링 환형 섹터        |
| `treemap`             | `.mc-treemap-node`                   | `tree` 계층, squarified 리프 rect     |
| `sankey`              | `.mc-sankey-node`+`-link`            | `links` 플로우, 노드 열+유량 리본     |
| `chord`               | `.mc-chord-group`+`-ribbon`          | `chord.matrix` 유향 플로우 원호       |
| `nightingale`         | `.mc-nightingale-node`               | 카테고리=등분 각도, 반지름=값         |
| `radial-bar`          | `.mc-radial-bar`                     | 카테고리=링, 각도=값                  |
| `radial-column`       | `.mc-radial-column`                  | 카테고리=각도 방향, 반지름=값         |
| `gauge`               | `.mc-gauge-track`/`-value`/`-needle` | `gauge:{min,max,target}` 다이얼       |
| `linear-gauge`        | `.mc-lg-track`/`-value`/`-target`    | 수평 바 게이지, `gauge` 옵션 공유     |
| `pictogram`           | `.mc-picto`(fill/empty)+`-label`     | 아이콘 1개=`pictogram.unit` 값        |
| `population-pyramid`  | `.mc-bar` 미러링 (`.mc-horizontal`)  | 첫 시리즈 좌측, 대칭 ±도메인          |

- `series[].type`으로 시리즈별 타입을 섞을 수 있다 (line+bar 혼합).
- `chart.setType(type)`으로 런타임 전환 — 줌 창은 리셋된다.
- bar 계열의 `direction: "horizontal"`은 값 축이 x축이 된다.
- `waterfall`의 값은 **델타**로 해석된다 — 양수는 `.mc-bar-up`,
  음수는 `.mc-bar-down` (각각 `--chart-up-color`/`--chart-down-color`).
- `bubble`은 `series[].points`의 `z`가 크기다 — values 모드면
  `|y|`를 크기로 쓴다.
- `heatmap`은 `series[].values` 행렬을 셀 그리드로 그린다.
  색 스케일은 `heatmap: { from, to }` 옵션 또는
  `--chart-heat-from`/`--chart-heat-to` 변수로 바꾼다.
- `funnel`/`pyramid`는 첫 시리즈의 카테고리 값을 세로 밴드로 그린다 —
  funnel은 다음 단계 폭이 아랫변, pyramid는 뾰족한 꼭대기.
  pie와 마찬가지로 범례는 카테고리 단위(`slice:N`)다.
- `wordcloud`는 카테고리를 단어로, 첫 시리즈 값을 font 크기로 그린다
  (축 없음, 값 큰 단어가 중심). 범례는 카테고리 단위다.
- `range-bar`/`range-area`는 `series[].ranges`의 `[low, high]`를 그린다.
  range-bar는 `direction: "horizontal"`도 지원한다. 값 축 도메인은
  구간의 최저~최고를 포함하도록 자동 확장된다.
- `candlestick`은 `series[].ohlc`의 `[open, high, low, close]`를
  몸통+심지로 그린다. 종가≥시가는 `.mc-candle-up`(`--chart-up-color`),
  하락은 `.mc-candle-down`(`--chart-down-color`).
- `radar`/`radar-area`는 카테고리를 스포크(각도), 값을 반지름으로
  쓰는 극좌표 폴리곤이다. 직교 축 대신 `.mc-radar-grid`의
  동심원/스포크를 그리고 범례는 시리즈 단위다. 줌·브러시·네비게이터는
  적용되지 않는다.
- `map`은 `series[].mapData`(지역 코드→값)를 지형 폴리곤에
  채우는 choropleth다. 지형은 `map: { topology }`에 TopoJSON 또는
  GeoJSON을 넘긴다 — 내장 세계 지도는 `WORLD_TOPOLOGY`를 임포트한다
  (Natural Earth 110m, ~108KB). `code`는 피처의 id/iso 코드/name과
  대소문자 무관하게 조인되고, 조인되지 않은 지역은 `.mc-map-no-data`로
  렌더된다. 축/줌/브러시/네비게이터 없이 지역 hover·선택을 지원하고
  플롯 우하단에 `.mc-map-scale` 값 범례(그라데이션 바)를 그린다.
  `map.projection`은 `"equirectangular"`(기본)/`"mercator"`,
  `map.from`/`map.to`는 색상 스케일, `map.excludeCodes`로 지역 제외
  (기본 남극), `map.featureKey`로 조인 키를 바꿀 수 있다.
- `sunburst`/`treemap`은 `series[].tree`의 재귀 `ChartTreeNode`를
  그린다 — `value` 생략 노드는 `children` 합계로 계산된다.
  sunburst는 깊이=링, 값=각도의 환형 섹터, treemap은 squarified
  리프 rect다. 범례는 최상위 노드 단위(`slice:N`)로 숨기면 그
  서브트리가 빠지고 재배치된다.
- `sankey`는 `series[].links`(`{source, target, value}`)로 노드를
  유도해 열 배치하고 유량 리본을 잇는다. 범례는 노드 단위로,
  숨긴 노드의 링크는 빠진다. `sankey: { nodeWidth, nodeGap }`으로
  노드 두께/간격을 조정한다.
- `chord`는 시리즈 대신 `chord: { matrix, labels }` 옵션의 n×n
  유향 행렬을 그린다 — `matrix[i][j]`는 그룹 i→j 유량, (i,j)/(j,i)는
  같은 리본의 양끝 두께. 범례는 그룹 단위로 숨기면 그 행/열이
  비워진다.
- 계층/플로우 차트는 직교 축·줌·브러시·네비게이터를 렌더하지 않는다.
  노드 클릭 선택(`allowPointSelect`/`selectPoint`), hover/포커스
  dimming, 툴팁은 동일하게 동작한다.
- `nightingale`은 카테고리별 등분 섹터를 값 비례 반지름으로 그린다
  (다중 시리즈는 슬롯 안에서 각도를 나눔). 값 눈금 링과 카테고리
  스포크 라벨은 `.mc-radar-grid`로 렌더된다. 범례는 카테고리
  단위(`slice:N`)다.
- `radial-bar`는 카테고리별 동심 링에 값 비례 각도(12시 기준
  최대 340°)의 막대를 그린다 — 다중 시리즈는 링 안에서 반지름을
  나눈다. 범례는 시리즈 단위다.
- `gauge`는 첫 시리즈 첫 값을 270° 다이얼로 그린다 —
  `gauge: { min, max, target }`으로 범위와 목표 눈금을 준다
  (기본 0–100). `.mc-gauge-track`(배경)/`.mc-gauge-value`(값 아크)/
  `.mc-gauge-needle`(바늘)/`.mc-gauge-label`(중앙 값)/`.mc-gauge-tick`
  (min/max)을 렌더한다. 범례와 축은 없고 `updatePoint`로 값을
  갱신할 수 있다.
- `nightingale`/`radial-bar`/`radial-column`/`gauge`도 축·줌·브러시·네비게이터를
  렌더하지 않는다 — hitTest는 극좌표 섹터(링+각도)로 해석된다.
- `ohlc`는 `ohlc` 데이터를 몸통 없는 틱 선분으로 그린다 —
  `.mc-ohlc-stem`(고저 심지)/`.mc-ohlc-open`(좌 틱)/`.mc-ohlc-close`
  (우 틱), 톤은 `--chart-up-color`/`--chart-down-color`.
- `boxplot`은 `series[].box`의 `[min, q1, median, q3, max]`를
  상자+수염으로 그린다 — `.mc-box-body`(Q1~Q3 채움)/`.mc-box-whisker`/
  `.mc-box-cap`/`.mc-box-median`.
- `histogram`은 원시 `values`를 `histogram: { bins }` 개수(기본
  Sturges) 또는 `binWidth` 고정 폭으로 구간화해 막대를 그린다 —
  x축은 값 도메인(linear)으로 자동 전환된다.
- `cone-funnel`은 단계 높이가 값에 비례하고 외곽이 꼭짓점으로 수렴하는
  퍼널이다 — 슬라이스별 y 밴드가 균등하지 않다. 범례는 카테고리 단위.
- `organization`은 `series[].tree`를 위에서 아래로 펼치는 조직도다 —
  리프는 균등 슬롯, 부모는 자식 중심 위, 부모→자식 엘보 커넥터
  `.mc-org-link`. 축 없이 노드 hover/선택을 지원한다.
- `linear-gauge`는 첫 시리즈 첫 값을 수평 트랙 위 채움으로 그린다 —
  `gauge: { min, max, target }` 옵션을 다이얼 게이지와 공유한다.
- `pictogram`은 카테고리 행마다 사람 아이콘을 값 비례로 채우는
  ISOTYPE 차트다 — 아이콘 1개 = `pictogram.unit` 값(생략 시 행 합계
  최대/`maxPerRow`의 nice 단위로 자동). 채움 아이콘은 시리즈 색
  (`.mc-picto-fill`), 행 최대 대비 잔여는 빈 윤곽(`.mc-picto-empty`)이다.
  축은 렌더하지 않고 행 왼쪽에 카테고리 라벨(`.mc-picto-label`)을 둔다.
  범례는 시리즈 단위이고 아이콘 hover는 포인트 툴팁을 띄운다.
- `population-pyramid`는 첫 시리즈를 중앙축 좌측으로 미러링한 대칭
  수평 막대 차트다 (인구 피라미드). `pyramidPop.leftSeries`(기본 1)개의
  시리즈가 좌측, 나머지가 우측이며 값 축 도메인은 ±대칭, 틱 라벨은
  절대값으로 표시된다. `direction` 옵션과 무관하게 항상 수평이다.
- `map`의 `series[].mapPoints`(`{lon, lat, value?, label?, color?}`)는
  지역과 같은 투영으로 `.mc-map-marker` 원을, `mapLines`
  (`{from, to, label?}`)는 `.mc-map-line` 연결선을 그린다 —
  마커는 hover/선택 대상이고 `value`가 있으면 sqrt 크기 스케일이다.
- 마커의 `image`는 원형 클립 아바타 핀(`.mc-map-avatar` + 링)으로
  렌더하고, `pulse`는 펄스 링(`.mc-map-pulse`)을 추가한다 —
  도식 "주변 사용자" 지도(프로필 핀/내 위치) 용도다.
- `map: { cluster: true | { size } }`는 화면 px 셀(기본 48)에 뭉친
  마커를 카운트 버블(`.mc-map-cluster`)로 묶는다. 버블 클릭은
  `clusterSelect` 이벤트로 `{members, points}`를 전달한다 —
  앱이 목록 카드를 띄우는 데 쓴다.
- `map: { panZoom: true }`는 지도 레이어에 휠 줌·드래그 팬·핀치를
  연다 — 지역/연결선은 그룹 transform으로, 마커/클러스터는 좌표만
  변환돼 핀 크기가 일정하게 유지된다. `resetZoom()`/`dblclick`으로
  원복된다. 축 기반 `zoom` 옵션과 별개다.
- `map.includeCodes`는 지정 피처만 렌더한다 — `["410"]`이면
  WORLD_TOPOLOGY에서 한국만 플롯에 맞춰 한 지역 확대 지도를 만든다.
  시군구·동 단위는 번들 데이터에 없지만 `map.topology`에 임의
  GeoJSON을 주입하면 그 경계로 렌더된다 — 오픈 행정경계
  데이터(국토지리정보원 등)를 넣으면 된다.

```ts
chart.on("clusterSelect", (e) => {
  // e.members: mapPoints 인덱스, e.points: 입력 레코드
  showUserList(e.points.map((p) => p.label));
});
```

- `itemStyle`/`points[].color`·`name`은 line/bar/slice/word/버블 등
  포인트 단위로 동작한다 — `name`은 툴팁·라벨·aria-label을 덮어쓴다.

```ts
import { WORLD_TOPOLOGY } from "@moda-chart/core";

const chart = new ChartCore({
  type: "map",
  map: { topology: WORLD_TOPOLOGY },
  series: [
    {
      name: "국가별 지표",
      mapData: [
        { code: "410", value: 86 }, // ISO numeric id — South Korea
        { code: "Japan", value: 74 }, // properties.name 조인
      ],
    },
  ],
});
```

## 실시간 데이터 API

전체 교체(`setData`) 외에 증분 갱신이 있다 — 변경마다 `dataChange`
이벤트가 발행된다. `categories`를 생략한 차트는 포인트 추가 시
카테고리가 자동 재생성된다.

```ts
chart.addSeries({ name: "이익", values: [3, 6, 2] });
chart.removeSeries("이익"); // 시리즈 id 기준
chart.addPoint("매출", 33); // 끝에 추가
chart.addPoint("매출", 33, { shift: true }); // 앞을 밀어내 창 유지
chart.removePoint("매출", 0); // 인덱스 기준
```

### IBChart식 부분 갱신 메서드

데이터의 부분 갱신/조작은 아래 메서드로도 가능하다
(`pointUpdate`/`redraw` 이벤트를 동반한다):

```ts
chart.setCategories(["Q1", "Q2", "Q3", "Q4"]); // 카테고리 교체
chart.updateSeries("매출", { values: [..], name: "개편" }); // 시리즈 패치
chart.updatePoint("매출", 1, 99); // pointUpdate 이벤트 발행
chart.redraw(); // 스냅샷 무효화 + redraw 이벤트
chart.print(); // window.print() — beforePrint/afterPrint와 연동
```

`updatePoint`는 values/points 모드 시리즈만 지원한다 —
`ranges`/`ohlc` 시리즈는 `updateSeries`로 통째 교체한다.

### 부분 옵션/데이터 병합 — updateDelta (AG Charts 해당)

`chart.updateDelta(partial)`는 patch의 키만 병합해 한 번만 다시
그린다. 중첩 객체 옵션은 재귀 병합되고 `series`/`categories` 등
배열은 통째로 교체된다 — `batch`와 달리 옵션과 데이터를 같은
호출에 섞을 수 있다.

```ts
chart.updateDelta({
  series: [{ name: "개편", values: [4, 3, 2, 1] }], // 데이터 교체
  legend: { position: "top" }, // legend 나머지 키는 유지
});
```

series/categories가 포함되면 데이터 갱신으로 간주해 `dataChange`가
발행되고 `animation.updates`(기본 true)가 켜져 있으면 첫 렌더에
`.mc-updating`이 붙어 `mc-update` 전환이 재생된다. 같은 키의
`.mc-bar`/`.mc-point`는 이전 박스에서 새 박스로 translate/scale이
보간된다(`.mc-morph`) — 키가 없는 path 계열(선/슬라이스)은
morph 없이 `.mc-updating` 페이드만 적용된다.

### 부가 데이터 — etcData (IBChart 해당)

`series[].etcData` / `points[].etcData`에 임의 메타데이터를 실을 수
있다. 차트는 이를 툴팁 컨텍스트(`format`/`render`)와 `pointSelect`
이벤트에 그대로 전달하며 `chart.getEtcData(seriesId, index?)`로도
조회된다 — 코드표/원시 키 등 렌더 외 데이터를 싣는 용도다.

```ts
series: [{ name: "매출", values: […], etcData: { code: "MTD" } }]
chart.on("pointSelect", (e) => console.log(e.etcData)); // { code: "MTD" }
```

### XML 데이터 — loadXml (IBChart XML 인터페이스)

`chart.loadXml(xml)`은 XML 문자열을 `parseDataXml`로 해석해
`setData`에 주입한다. 지원 형식 — `<category>` 목록과
`<series>`(이름/색상 속성 + `<value>` 반복, `values`/`data`
속성이나 `<data>` 쉼표 목록도 허용). 해석 실패 시 false를 반환하고
데이터는 유지된다.

```ts
chart.loadXml(`<chartData>
  <categories><category>A</category><category>B</category></categories>
  <series name="XML"><value>9</value><value>18</value></series>
</chartData>`);

parseDataXml(xml); // { series, categories? } | null — 독립 사용 가능
```

## 에러바 (series.errors)

`series[].errors: [[lo, hi], …]` — 포인트 값의 ±오차를 `.mc-error`
캡 선분으로 그린다(line/bar/scatter 계열, 세로 차트만). 오차 끝이
값 축 도메인에 포함되도록 도메인이 자동 확장된다. 선색은
`--chart-error-color`.

## 추세선 (series.trend)

`series[].trend: { type, period? }` — `.mc-trendline` 점선 오버레이.
`"linear"`는 최소제곱 직선, `"movingAverage"`는 `period`(기본 3)
창의 이동평균 폴리라인이다. 선색은 시리즈 색을 따른다.

## 기준 영역 (markBands)

`markBands: [{ from, to, label?, color?, axis? }]` — 값 축의 도메인
구간을 플롯 배경에 칠한다(`.mc-mark-band` rect + `.mc-mark-label`).
IBSheet의 경고지역(plot band)에 해당한다. 도메인과 겹치지 않는
구간은 생략된다. 기본색은 `--chart-mark-band-color`.

## 인라인 주석 (annotations)

`annotations: [{ x, y?, label?, image?, width?, height? }]` — 데이터
좌표에 텍스트/이미지 주석을 단다(`.mc-annotation`). `x`는 카테고리
라벨(band) 또는 도메인 값(linear/time), `y`는 값 축 값이다. `y`를
생략하면 플롯 상단 근처에 붙는다. 직교 좌표 차트에서만 렌더되며
존재하지 않는 카테고리는 건너뛴다. 글자색은
`--chart-annotation-color`.

```ts
annotations: [
  { x: "4월", y: 30, label: "최고" },
  { x: "2월", image: "/flag.svg", width: 16, height: 16 },
];
```

## 값 라벨 (dataLabels)

`dataLabels: true | { format }` — 바 위·포인트 위·슬라이스 안에
값 텍스트(`.mc-data-label`)를 그린다. pie는 기본값이 백분율이다.
포맷터는 툴팁과 같은 `TooltipFormatContext`를 받는다.

## 기준선 (markLines)

`markLines: [{ value, label?, color?, axis? }]` — 값 축 기준선
(`.mc-mark-line`)을 플롯에 긋는다. 다중 축이면 `axis`로 축 id를 지정.
도메인 밖 값은 생략된다. `color` 미지정 시 `--chart-mark-color`.

## 제목 · 빈 상태 · 로딩

- `title`/`subtitle` — `.mc-title`/`.mc-subtitle`에 렌더
- `emptyText` — `.mc-empty` 문구 (기본 "No data")
- `chart.setLoading(true)` — `.mc-loading` 오버레이 + `aria-busy`

## 로케일 (locale)

`locale: "ko" | "en"`(기본 `en`) — 내장 문구(`.mc-empty`,
`.mc-zoom-reset`, `.mc-drillup` 버튼)가 한국어로 바뀐다.
`chart.setLocale(locale)`로 런타임 전환한다. `emptyText` 등 개별
옵션을 지정하면 로케일 기본값보다 우선한다.

## 접근성 패턴 채우기 (patterns)

`patterns: true` — 색상 외에 무늬로 시리즈를 구분한다
(색각 보조). `<defs>`에 `<pattern class="mc-pattern">`이 생성되고
bar/area/slice/bubble의 채움이 `url(#…)`이 된다.
`chart.setPatterns(bool)`으로 런타임 토글한다.

## 스파크라인

`mountSparkline(el, opts)` / `sparklineMarkup(opts)` — 축·범례 없는
인라인 미니 차트(AG Charts sparklines 해당). `ChartCore` 없이 단독
SVG를 만든다 — `type: "line"|"area"|"bar"`, `values`, `color`,
`width`/`height`, `min`/`max` 도메인, `fillOpacity`.

```ts
import { mountSparkline } from "@moda-chart/core";
mountSparkline(el, { values: [3, 7, 5, 9], type: "area" });
```

## 텍스트 마이닝 (parseText)

`parseText(text, ignore?, max?)` — 텍스트에서 단어 빈도를 추출해
`{ name, value }[]`를 돌려준다 (IBChart `parseText` 해당). 유니코드
문자/숫자 연속을 단어로 보고 길이 1·불용어는 제외, 빈도 내림차순.
wordcloud의 categories/values 입력으로 바로 쓸 수 있다.

## DOM 계약

시리즈 그룹 `.mc-series[data-series-id]` 아래에 타입별 요소
(`.mc-point`/`.mc-bar`/`.mc-slice`/`.mc-bubble`/`.mc-heat-cell`/
`.mc-word` + `.mc-line`/`.mc-area` path). waterfall 연결선은
`.mc-waterfall-line`, 기준 영역은 `.mc-mark-bands` 그룹 아래
`.mc-mark-band` rect, 주석은 `.mc-annotations` 아래 `.mc-annotation`.
range/candle/radar 계열은 `.mc-range-bar`/`.mc-range-area`/
`.mc-candle`(`.mc-candle-wick` 심지)/`.mc-radar`(`.mc-radar-grid`
그리드), 에러바는 `.mc-errors` 아래 `.mc-error`, 추세선은
`.mc-trendlines` 아래 `.mc-trendline`, 네비게이터는 `.mc-navigator`
아래 `.mc-nav-track`/`.mc-nav-window`/`.mc-nav-handle`이다.
지도는 `.mc-type-map` 아래 `.mc-map-region[data-map-code]` 폴리곤
(데이터 없음은 `.mc-map-no-data`)과 `.mc-map-scale` 값 범례를 렌더한다.
계층/플로우는 각각 `.mc-type-sunburst`/`.mc-type-treemap`/
`.mc-type-sankey`/`.mc-type-chord` 아래 `.mc-sunburst-node`,
`.mc-treemap-node`, `.mc-sankey-node`+`.mc-sankey-link`,
`.mc-chord-group`+`.mc-chord-ribbon`으로 렌더된다.
극좌표는 `.mc-type-nightingale`/`.mc-type-radial-bar`/`.mc-type-radial-column`/`.mc-type-gauge`
아래 `.mc-nightingale-node`, `.mc-radial-bar`, `.mc-radial-column`, `.mc-gauge-*`로
렌더된다.
포커스/비활성/선택 상태는 `.mc-focused`/`.mc-dimmed`/`.mc-selected`
클래스로 표시된다.
