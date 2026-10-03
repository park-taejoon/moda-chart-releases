# Vanilla DOM

어댑터 없이 `mountChart`로 직접 마운트한다 — CDN/순수 JS 환경용.

```ts
import { mountChart } from "@moda-chart/core";
import "@moda-chart/core/styles.css";

const { chart, rootEl, destroy } = mountChart(el, {
  type: "line",
  series: [{ name: "매출", values: [12, 24, 16, 30] }],
  categories: ["1월", "2월", "3월", "4월"],
  legend: true,
  tooltip: { mode: "shared" },
  zoom: true,
});

chart.setType("bar");
destroy(); // 리스너·ResizeObserver·DOM 정리
```

`MountChartOptions` = `ChartCoreOptions`. 인터랙션 배선(포인터·휠·
키보드·범례/툴바 클릭)과 ResizeObserver는 `attachChartView`가 한다 —
자체 프레임워크 어댑터를 만들 때는 이 함수를 생명주기 훅에서 1회
호출하고 반환된 cleanup을 unmount에 연결한다.
