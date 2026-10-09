# 테마와 보내기

## 테마

`theme: "light" | "dark"` — `.mc-root[data-theme]`로 반영되고
`chart.setTheme(theme)`으로 런타임 전환한다.
색상은 전부 `--chart-*` 변수라, 테마는 변수 재정의로 구현된다
(변수 표는 `theming.md`).

## 팔레트

`palette: ["#...", ...]` — 시리즈 고정 색이 없을 때 순환하는 팔레트를
교체한다. 기본값은 `--chart-palette-1..8` 변수를 참조한다.

## 애니메이션

`animation: false | { duration }` — 진입 트랜지션. 켜지면
`.mc-root.mc-animated`가 붙고 `--chart-animation-duration`이
인라인으로 설정된다. 갱신은 CSS transition이 처리한다.

## 반응형

`responsive: false`가 아니면 렌더러가 ResizeObserver로 컨테이너
너비를 추적해 `chart.setSize(w)`를 호출한다 (없는 환경은 no-op).
명시 크기는 `width`/`height` 옵션 또는 `setSize`.

## 보내기

툴바의 `.mc-export-svg`/`.mc-export-png`/`.mc-export-csv` 버튼,
컨텍스트 메뉴의 내장 항목, 또는 API:

```ts
chart.toSVGString(); // 인라인 스타일이 굳어진 독립 SVG 문서
chart.toCSV(); // "category,시리즈명…" 헤더 + 행 CSV
```

- SVG — `downloadSVG(chart)`가 blob 다운로드를 트리거한다
- PNG — `downloadPNG(chart)`가 SVG를 2배 해상도 canvas에 그려
  data URL로 저장한다
- CSV — `downloadCSV(chart)`가 `chart.toCSV()` 결과를 저장한다.
  쉼표/따옴표가 있는 라벨은 RFC 4180 규칙으로 인용된다
- 파일명 — `exporting: { filename: "sales" }`로 기본명을 바꾼다
  (기본 `"chart"` → `chart.svg`/`chart.png`/`chart.csv`)
- 발행 이벤트: `export: { format }`
