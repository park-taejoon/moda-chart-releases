# Vue 2.7

Vue 2.7의 내장 composition API를 사용한다.

```vue
<script setup lang="ts">
import { ChartView, useChart } from "@moda-chart/vue2";
import "@moda-chart/vue2/styles.css";

const { chart, state } = useChart({
  type: "line",
  series: [{ name: "매출", values: [12, 24, 16, 30] }],
});
</script>

<template>
  <ChartView :chart="chart" />
</template>
```

주의:

- 템플릿 이벤트 핸들러는 표현식만 받는다 — 조건부·캐스트는 메서드로
  추출한다 (vite:vue2 제약).
- `v-if` 분기에 같은 컴포넌트를 쓸 때는 분기별 `key`를 명시한다 —
  Vue2는 인스턴스를 재사용해 prop 교체가 무시될 수 있다.
- `ChartView`는 루트 `$el`에 렌더러를 연결한다 — 컴포넌트 루트가
  단일 `<div>`임을 유지한다.
