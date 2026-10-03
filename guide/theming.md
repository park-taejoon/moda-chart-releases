# 테마

공통 스타일은 `packages/core/src/styles.css`에 있고 어댑터는
`@import "@moda-chart/core/styles.css"`로 재사용한다.

색상은 `--chart-*` CSS 변수로 노출한다 — 새 색상을 추가할 때는 변수로
만들고 이 표에 추가한다.

| 변수                           | 기본값                 | 용도                          |
| ------------------------------ | ---------------------- | ----------------------------- |
| `--chart-bg`                   | `#ffffff`              | 차트 배경                     |
| `--chart-border-color`         | `#e2e8f0`              | 루트·툴팁 경계선              |
| `--chart-text-color`           | `#1e293b`              | 축 라벨·범례 텍스트           |
| `--chart-text-muted`           | `#94a3b8`              | 눈금 라벨                     |
| `--chart-axis-color`           | `#cbd5e1`              | 축선·눈금선                   |
| `--chart-grid-color`           | `#e2e8f0`              | 그리드선                      |
| `--chart-focus-color`          | `#2563eb`              | 포커스 포인트/슬라이스 외곽   |
| `--chart-selection-color`      | `#2563eb`              | 브러시 선택 영역              |
| `--chart-mark-color`           | `#f43f5e`              | 기준선(markLines) 색          |
| `--chart-mark-band-color`      | `rgba(244,63,94,.08)`  | 기준 영역(markBands) 채움     |
| `--chart-up-color`             | `#16a34a`              | waterfall 증가 막대           |
| `--chart-down-color`           | `#dc2626`              | waterfall 감소 막대           |
| `--chart-waterfall-line-color` | `#94a3b8`              | waterfall 연결선              |
| `--chart-heat-from`            | `#dbeafe`              | heatmap 최저값 색             |
| `--chart-heat-to`              | `#1d4ed8`              | heatmap 최고값 색             |
| `--chart-annotation-color`     | `#475569`              | 주석(annotations) 점·글자     |
| `--chart-error-color`          | `#64748b`              | 에러바(errors) 선             |
| `--chart-nav-bg`               | `#f1f5f9`              | 네비게이터 스트립 배경        |
| `--chart-nav-handle`           | `#94a3b8`              | 네비게이터 창 핸들            |
| `--chart-map-from`             | `#dbeafe`              | 지도 최저값 색                |
| `--chart-map-to`               | `#1d4ed8`              | 지도 최고값 색                |
| `--chart-map-empty`            | `#e2e8f0`              | 지도 데이터 없는 지역         |
| `--chart-map-border`           | `#ffffff`              | 지도 지역 경계선              |
| `--chart-node-border`          | `#ffffff`              | 계층/플로우 노드 경계선       |
| `--chart-link-opacity`         | `0.45`                 | sankey/chord 리본 불투명도    |
| `--chart-gauge-track`          | `#e2e8f0`              | 게이지·linear-gauge 배경 트랙 |
| `--chart-gauge-needle`         | `#475569`              | 게이지 바늘/허브              |
| `--chart-menu-bg`              | `#ffffff`              | 컨텍스트 메뉴 배경            |
| `--chart-menu-hover`           | `#f1f5f9`              | 컨텍스트 메뉴 항목 hover      |
| `--chart-org-link-color`       | `#94a3b8`              | organization 커넥터           |
| `--chart-map-line-color`       | `#64748b`              | 지도 연결선(mapLines)         |
| `--chart-map-avatar-ring`      | `#ffffff`              | 아바타 핀 외곽 링             |
| `--chart-map-pulse`            | `#db2777`              | 내 위치 펄스 링               |
| `--chart-map-cluster-bg`       | `#475569`              | 클러스터 버블 배경            |
| `--chart-map-cluster-color`    | `#ffffff`              | 클러스터 카운트 텍스트        |
| `--chart-draw-color`           | `#7c3aed`              | 그린 주석 선/영역 기본색      |
| `--chart-loading-bg`           | `rgba(255,255,255,.7)` | 로딩 오버레이 배경            |
| `--chart-tooltip-bg`           | `#0f172a`              | 툴팁 배경                     |
| `--chart-tooltip-color`        | `#f8fafc`              | 툴팁 텍스트                   |
| `--chart-palette-1` … `-8`     | 팔레트                 | 시리즈 색 순환                |
| `--chart-animation-duration`   | `300ms`                | 진입 트랜지션 (인라인 제어)   |

## 다크 모드

`.mc-root[data-theme="dark"]` 아래에서 위 변수가 재정의된다.
`theme: "dark"` 옵션 또는 `chart.setTheme("dark")`로 전환한다.
커스텀 테마는 같은 선택자 구조로 변수를 덮어쓰면 된다.

## DOM 클래스

렌더 요소는 `mc-` 접두어를 쓴다 (`.mc-root`/`.mc-svg`/`.mc-plot`/
`.mc-series`/`.mc-point`/`.mc-bar`/`.mc-slice`/`.mc-bubble`/
`.mc-heat-cell`/`.mc-axis`/`.mc-grid`/`.mc-legend-item`/`.mc-tooltip`/
`.mc-selection`/`.mc-crosshair`/`.mc-title`/`.mc-subtitle`/
`.mc-data-label`/`.mc-mark-line`/`.mc-mark-band`/`.mc-waterfall-line`/
`.mc-map-region`/`.mc-map-scale`/`.mc-sunburst-node`/`.mc-treemap-node`/
`.mc-sankey-node`/`.mc-sankey-link`/`.mc-chord-group`/`.mc-chord-ribbon`/
`.mc-nightingale-node`/`.mc-radial-bar`/`.mc-radial-column`/`.mc-gauge-track`/`.mc-gauge-needle`/
`.mc-box`/`.mc-ohlc`/`.mc-lg-*`/`.mc-org-node`/`.mc-org-link`/
`.mc-map-marker`/`.mc-map-line`/`.mc-map-avatar`/`.mc-map-pulse`/
`.mc-map-cluster`/`.mc-map-geo`/`.mc-scrollbar`/`.mc-sb-*`/
`.mc-tick-group`/`.mc-menu`/`.mc-legend-check`/`.mc-sparkline`/
`.mc-draw-*`(그린 주선: `.mc-draw-seg`/`.mc-draw-box`/`.mc-draw-draft`)/
`.mc-legend-prev`/`.mc-legend-next`/`.mc-legend-page`/`.mc-drawing`/
`.mc-picto-*`(픽토그램 아이콘/행 라벨)/
`.mc-pattern`/`.mc-drillup`/`.mc-loading` …).
새 요소도 같은 접두어 + conformance 계약 갱신.
