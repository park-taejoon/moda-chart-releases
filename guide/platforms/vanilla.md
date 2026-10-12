# Vanilla DOM

어댑터 없이 `mountChart`로 직접 마운트한다 — CDN/순수 JS 환경용.

```ts
import { mountChart } from "@moda-chart/core";
import "@moda-chart/core/styles.css";

const m = mountChart(el, {
  type: "line",
  series: [{ name: "매출", values: [12, 24, 16, 30] }],
  categories: ["1월", "2월", "3월", "4월"],
  legend: true,
  tooltip: { mode: "shared" },
  zoom: true,
});
const { chart, rootEl, destroy } = m;

chart.setType("bar");
// 옵션 동기화 — 마운트 옵션과 깊이 비교해 달라진 키만 반영.
// next에서 빠진 키는 해제되므로 목표 상태 전체를 넘긴다.
m.update({ type: "bar", series: next, categories: nextCats });
destroy(); // 리스너·ResizeObserver·DOM + 코어 dispose 정리
```

`MountChartOptions` = `ChartCoreOptions`. 인터랙션 배선(포인터·휠·
키보드·범례/툴바 클릭)과 ResizeObserver는 `attachChartView`가 한다 —
자체 프레임워크 어댑터를 만들 때는 이 함수를 생명주기 훅에서 1회
호출하고 반환된 cleanup을 unmount에 연결한다.

## 렌더러 선택

`mountChart(el, { renderer })`의 `renderer`에 `createCanvasRenderer()`
등 대체 렌더러를 넘긴다 — 기본은 `svgRenderer`(내장 SVG 체인).
계약·lite 경로는 [plugins.md](../plugins.md) 참조.
