# Vue 3

```vue
<script setup lang="ts">
import { ChartView, useChart } from "@moda-chart/vue";
import "@moda-chart/vue/styles.css";

const { chart, state } = useChart({
  type: "bar",
  series: [{ name: "매출", values: [12, 24, 16, 30] }],
  categories: ["1월", "2월", "3월", "4월"],
  legend: true,
});
</script>

<template>
  <ChartView :chart="chart" />
</template>
```

| export                 | 역할                                 |
| ---------------------- | ------------------------------------ |
| `useChart(options)`    | 코어 생성 + 스냅샷 `shallowRef` 구독 |
| `useChartState(chart)` | 외부 코어의 스냅샷만 구독            |
| `ChartView`            | DOM 계약 렌더 SFC                    |

`ChartView`는 내부적으로 코어의 공용 DOM 렌더러(`attachChartView`)를
쓴다 — DOM 계약은 5개 렌더러가 동일하다.
