# 테마

공통 스타일은 `packages/core/src/styles.css`에 있고 어댑터는
`@import "@moda-chart/core/styles.css"`로 재사용한다.

색상은 `--chart-*` CSS 변수로 노출한다 — 새 색상을 추가할 때는 변수로
만들고 이 표에 추가한다.

| 변수                           | 기본값                  | 용도                          |
| ------------------------------ | ----------------------- | ----------------------------- |
| `--chart-bg`                   | `#ffffff`               | 차트 배경                     |
| `--chart-border-color`         | `#e2e8f0`               | 루트·툴팁 경계선              |
| `--chart-text-color`           | `#1e293b`               | 축 라벨·범례 텍스트           |
| `--chart-text-muted`           | `#94a3b8`               | 눈금 라벨                     |
| `--chart-axis-color`           | `#cbd5e1`               | 축선·눈금선                   |
| `--chart-grid-color`           | `#e2e8f0`               | 그리드선                      |
| `--chart-focus-color`          | `#2563eb`               | 포커스 포인트/슬라이스 외곽   |
| `--chart-selection-color`      | `#2563eb`               | 브러시 선택 영역              |
| `--chart-crosshair-band-color` | `rgba(148,163,184,.18)` | 크로스헤어 카테고리 밴드 채움 |
| `--chart-mark-color`           | `#f43f5e`               | 기준선(markLines) 색          |
| `--chart-mark-band-color`      | `rgba(244,63,94,.08)`   | 기준 영역(markBands) 채움     |
| `--chart-up-color`             | `#16a34a`               | waterfall 증가 막대           |
| `--chart-down-color`           | `#dc2626`               | waterfall 감소 막대           |
| `--chart-waterfall-line-color` | `#94a3b8`               | waterfall 연결선              |
| `--chart-heat-from`            | `#dbeafe`               | heatmap 최저값 색             |
| `--chart-heat-to`              | `#1d4ed8`               | heatmap 최고값 색             |
| `--chart-annotation-color`     | `#475569`               | 주석(annotations) 점·글자     |
| `--chart-error-color`          | `#64748b`               | 에러바(errors) 선             |
| `--chart-nav-bg`               | `#f1f5f9`               | 네비게이터 스트립 배경        |
| `--chart-nav-handle`           | `#94a3b8`               | 네비게이터 창 핸들            |
| `--chart-map-from`             | `#dbeafe`               | 지도 최저값 색                |
| `--chart-map-to`               | `#1d4ed8`               | 지도 최고값 색                |
| `--chart-map-empty`            | `#e2e8f0`               | 지도 데이터 없는 지역         |
| `--chart-map-border`           | `#ffffff`               | 지도 지역 경계선              |
| `--chart-node-border`          | `#ffffff`               | 계층/플로우 노드 경계선       |
| `--chart-link-opacity`         | `0.45`                  | sankey/chord 리본 불투명도    |
| `--chart-gauge-track`          | `#e2e8f0`               | 게이지·linear-gauge 배경 트랙 |
| `--chart-gauge-needle`         | `#475569`               | 게이지 바늘/허브              |
| `--chart-menu-bg`              | `#ffffff`               | 컨텍스트 메뉴 배경            |
| `--chart-menu-hover`           | `#f1f5f9`               | 컨텍스트 메뉴 항목 hover      |
| `--chart-org-link-color`       | `#94a3b8`               | organization 커넥터           |
| `--chart-org-text-color`       | `#ffffff`               | organization 노드 이름        |
| `--chart-org-sub-color`        | `rgba(255,255,255,.85)` | organization 노드 부제        |
| `--chart-org-toggle-bg`        | `#ffffff`               | organization 접기 배지 배경   |
| `--chart-org-toggle-color`     | `#475569`               | organization 접기 배지 기호   |
| `--chart-org-photo-ring`       | `rgba(255,255,255,.9)`  | organization 노드 사진 링     |
| `--chart-org-match`            | `#f59e0b`               | organization 검색 매치 강조   |
| `--chart-org-minimap-bg`       | `rgba(255,255,255,.92)` | organization 미니맵 배경      |
| `--chart-org-minimap-node`     | `#94a3b8`               | organization 미니맵 노드      |
| `--chart-org-minimap-view`     | `rgba(59,130,246,.18)`  | organization 미니맵 뷰포트    |
| `--chart-map-line-color`       | `#64748b`               | 지도 연결선(mapLines)         |
| `--chart-map-avatar-ring`      | `#ffffff`               | 아바타 핀 외곽 링             |
| `--chart-map-pulse`            | `#db2777`               | 내 위치 펄스 링               |
| `--chart-map-cluster-bg`       | `#475569`               | 클러스터 버블 배경            |
| `--chart-map-cluster-color`    | `#ffffff`               | 클러스터 카운트 텍스트        |
| `--chart-draw-color`           | `#7c3aed`               | 그린 주석 선/영역 기본색      |
| `--chart-loading-bg`           | `rgba(255,255,255,.7)`  | 로딩 오버레이 배경            |
| `--chart-tooltip-bg`           | `#0f172a`               | 툴팁 배경                     |
| `--chart-tooltip-color`        | `#f8fafc`               | 툴팁 텍스트                   |
| `--chart-palette-1` … `-8`     | 팔레트                  | 시리즈 색 순환                |
| `--chart-animation-duration`   | `300ms`                 | 진입 트랜지션 (인라인 제어)   |

## 타이포그래피 토큰

글꼴·크기·웨이트도 변수로 노출된다 — CSS만으로 디자인 시스템의
타이포 스케일에 맞출 수 있다.

| 변수                           | 기본값         | 용도                                  |
| ------------------------------ | -------------- | ------------------------------------- |
| `--chart-font-family`          | inherit        | 전체 글꼴 (SVG 텍스트까지 상속)       |
| `--chart-font-2xs`             | `8px`          | organization 보조 라벨                |
| `--chart-font-xs`              | `9px`          | organization 접기 배지                |
| `--chart-font-sm`              | `10px`         | 데이터·주석·마크·스케일 라벨, 소형 틱 |
| `--chart-font-md`              | `11px`         | 축/틱 라벨, 툴바, 프리셋, 테이블      |
| `--chart-font-lg`              | `12px`         | 부제, 툴팁, 범례, 메뉴                |
| `--chart-font-xl`              | `13px`         | 빈 상태, 범례 페이지 네비             |
| `--chart-font-2xl`             | `14px`         | linear-gauge 라벨                     |
| `--chart-font-title`           | `15px`         | 차트 제목                             |
| `--chart-font-display`         | `20px`         | 게이지 값                             |
| `--chart-font-weight-medium`   | `500`          | 보조 강조                             |
| `--chart-font-weight-semibold` | `600`          | 라벨·카운트 강조                      |
| `--chart-font-weight-bold`     | `700`          | 제목                                  |
| `--chart-font-numeric`         | `tabular-nums` | 툴팁 값·범례 집계·데이터 표 숫자 컬럼 |

```css
/* 디자인 시스템 맞춤 — 파일 하나만 덮어쓰면 된다 */
.mc-root {
  --chart-font-family: "Pretendard", sans-serif;
  --chart-font-md: 0.75rem;
  --chart-font-title: 1.125rem;
}
```

## 형상·간격 토큰

`border-radius`/`stroke-width`/`gap`/테두리 두께도 같은 래더로
변수화되어 있다. 기본값은 이전 하드코딩 값과 동일하다.

| 변수                      | 기본값 | 주 용도                            |
| ------------------------- | ------ | ---------------------------------- |
| `--chart-radius-xs`       | `2px`  | 툴팁 스와치                        |
| `--chart-radius-sm`       | `3px`  | 범례 스와치                        |
| `--chart-radius-md`       | `4px`  | 범례 항목·프리셋·메뉴 버튼         |
| `--chart-radius-lg`       | `5px`  | 툴바 버튼                          |
| `--chart-radius-xl`       | `6px`  | 툴팁·컨텍스트 메뉴                 |
| `--chart-radius-full`     | `50%`  | 스피너 등 원형 요소                |
| `--chart-stroke-icon`     | `0.09` | pictogram 빈 아이콘 (viewBox 상대) |
| `--chart-stroke-hairline` | `0.5`  | 지도 지역 경계                     |
| `--chart-stroke-thin`     | `1`    | 히트맵 셀·마커·박스 본문           |
| `--chart-stroke-xs`       | `1.2`  | 캔들 심지·조직도 링크              |
| `--chart-stroke-sm`       | `1.4`  | 에러바·박스 수염/중앙값            |
| `--chart-stroke-md`       | `1.5`  | 포인트·내비 트랙·지도 라인         |
| `--chart-stroke-lg`       | `1.6`  | 트렌드라인                         |
| `--chart-stroke-xl`       | `1.8`  | 지도 지역·마커 선택                |
| `--chart-stroke-2xl`      | `2`    | 라인·레이더·게이지·지도 포커스     |
| `--chart-stroke-3xl`      | `2.5`  | 포커스 상태·게이지 바늘            |
| `--chart-gap-sm`          | `4px`  | 프리셋 행 간격, 범례 행 간격       |
| `--chart-gap-md`          | `6px`  | 툴바·툴팁 행·범례 항목 간격        |
| `--chart-border-width`    | `1px`  | 크롬 테두리 (툴바·메뉴·툴팁)       |

## 모션 토큰

| 변수                         | 기본값     | 주 용도                           |
| ---------------------------- | ---------- | --------------------------------- |
| `--chart-animation-duration` | `300ms`    | 진입·갱신·퇴장 트랜지션 기본 길이 |
| `--chart-duration-fast`      | `150ms`    | opacity 페이드 등 짧은 전환       |
| `--chart-easing-standard`    | `ease`     | 진입·갱신 트랜지션                |
| `--chart-easing-out`         | `ease-out` | 지도 마커 ping                    |
| `--chart-easing-in`          | `ease-in`  | 퇴장 트랜지션                     |

## 다크 모드

`.mc-root[data-theme="dark"]` 또는 `.mc-root[data-mode="dark"]`
(객체 테마 `base:"dark"`) 아래에서 위 변수가 재정의된다.
`theme: "dark"` 옵션 또는 `chart.setTheme("dark")`로 전환한다.
커스텀 테마는 같은 선택자 구조로 변수를 덮어쓰면 된다.

## 객체 테마

CSS 파일 없이 `theme` 옵션 하나로 브랜드 테마를 전달할 수 있다
(AG Charts theme overrides 해당):

```ts
mountChart(el, {
  // ...
  theme: {
    name: "brand", // data-theme 값 — CSS 훅 (선택)
    base: "dark", // 먼저 깔릴 내장 변수 세트 (기본 light)
    palette: ["#7c3aed", "#ec4899", "#f59e0b"],
    vars: {
      "--chart-bg": "#faf5ff",
      "--chart-font-title": "18px",
      "--chart-radius-md": "10px",
    },
  },
});
```

- `name`은 `.mc-root`의 `data-theme` 값이 된다 —
  `.mc-root[data-theme="brand"]` 선택자로 추가 스타일을 얹을 수 있다.
  없으면 `base` 값이 들어간다.
- `base`는 `data-mode`가 되며 내장 변수 세트를 고른다.
- `palette`는 `options.palette`가 없을 때만 쓰인다 (옵션이 이긴다).
- `vars`는 `--` 접두 키만 `.mc-root` 인라인 스타일로 주입된다.
  테마를 문자열로 되돌리면 주입한 토큰은 자동으로 제거된다.
- `chart.setTheme(themeObject)`로 런타임 전환도 가능하다.

## 토큰 익스포트

`--chart-*` 토큰의 기본값 맵을 JSON/TS로 꺼낼 수 있다 —
Figma Tokens(Tokens Studio)나 Style Dictionary 같은 디자인
토큰 파이프라인에 연결하는 용도다:

```ts
import { CHART_TOKENS } from "@moda-chart/core";

// theme.vars에 그대로 주입해 다른 모드를 강제할 수도 있다
mountChart(el, { theme: { name: "forced-dark", vars: CHART_TOKENS.dark } });
```

- `@moda-chart/core/chart-tokens.json` — `{ light, dark }` 평탄 맵.
- `CHART_TOKENS` / `CHART_TOKENS_LIGHT` / `CHART_TOKENS_DARK` — TS 상수.
- 단일 소스는 `styles.css` — `pnpm --filter @moda-chart/core gen:tokens`가
  생성물을 갱신하고 테스트가 소스와의 일치를 검증한다.

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
