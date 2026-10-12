# Vue 2.7

Vue 2.7의 내장 composition API를 사용한다.

가장 간단한 진입은 `<Chart :options>` 단일 컴포넌트다 — 코어 생성·
prop 동기화·DOM 연결·dispose를 내부에서 처리한다.

```vue
<script setup lang="ts">
import { Chart } from "@moda-chart/vue2";
import "@moda-chart/vue2/styles.css";

const options = {
  type: "line" as const,
  series: [{ name: "매출", values: [12, 24, 16, 30] }],
};
</script>

<template>
  <Chart :options="options" />
</template>
```

차트 인스턴스는 `ref`로 받는다 — `ref="c"` → `c.chart`.

저수준 경로는 `useChart` + `ChartView` 조합이다:

```vue
<script setup lang="ts">
import { ChartView, useChart } from "@moda-chart/vue2";

const { chart, state } = useChart(options);
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
- `useChart`의 `options`에 `ref`/`reactive`/getter를 넘기면 자동
  동기화된다 — deep watch + `diffOptions`으로 달라진 키만 병합하고
  사라진 키는 해제한다. `reactive` 객체의 제자리 변경도 반영된다 —
  단 키 제거는 새 객체로 넘겨야 감지된다.

## 렌더러 선택

`ChartView`의 `renderer` prop에 `createCanvasRenderer()` 등 대체
렌더러를 넘긴다 — 기본은 내장 SVG 렌더러.
계약·lite 경로는 [plugins.md](../plugins.md) 참조.
