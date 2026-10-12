# Vue 3

가장 간단한 진입은 `<Chart :options>` 단일 컴포넌트다 — 코어 생성·
prop 동기화·DOM 연결·dispose를 내부에서 처리한다.

```vue
<script setup lang="ts">
import { Chart } from "@moda-chart/vue";
import "@moda-chart/vue/styles.css";

const options = {
  type: "bar" as const,
  series: [{ name: "매출", values: [12, 24, 16, 30] }],
  categories: ["1월", "2월", "3월", "4월"],
  legend: true,
};
</script>

<template>
  <Chart :options="options" />
</template>
```

차트 인스턴스는 `ref`로 받는다 — `ref="c"` → `c.value.chart`.

저수준 경로는 `useChart` + `ChartView` 조합이다:

```vue
<script setup lang="ts">
import { ChartView, useChart } from "@moda-chart/vue";

const { chart, state } = useChart(options);
</script>

<template>
  <ChartView :chart="chart" />
</template>
```

| export                 | 역할                                  |
| ---------------------- | ------------------------------------- |
| `Chart`                | options 단일 컴포넌트 (expose: chart) |
| `useChart(options)`    | 코어 생성 + 스냅샷 `shallowRef` 구독  |
| `useChartState(chart)` | 외부 코어의 스냅샷만 구독             |
| `ChartView`            | DOM 계약 렌더 SFC                     |

`ChartView`는 내부적으로 코어의 공용 DOM 렌더러(`attachChartView`)를
쓴다 — DOM 계약은 5개 렌더러가 동일하다.

`useChart`의 `options`은 `ref`/`reactive`/`computed`/getter로 넘기면
자동 동기화된다 — deep watch + `diffOptions`으로 달라진 키만 병합하고
사라진 키는 해제한다. `reactive` 객체의 제자리 변경도 반영된다 —
단 키 제거는 새 객체로 넘겨야 감지된다. 원시 객체를 넘기면 최초 값이
기준점이 된다.

## 렌더러 선택

`ChartView`의 `renderer` prop에 `createCanvasRenderer()` 등 대체
렌더러를 넘긴다 — 기본은 내장 SVG 렌더러.
계약·lite 경로는 [plugins.md](../plugins.md) 참조.
