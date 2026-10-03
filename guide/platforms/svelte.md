# Svelte 5

```svelte
<script lang="ts">
  import { ChartView, createChartStore } from "@moda-chart/svelte";
  import "@moda-chart/svelte/styles.css";

  const store = createChartStore({
    type: "line",
    series: [{ name: "매출", values: [12, 24, 16, 30] }],
    categories: ["1월", "2월", "3월", "4월"],
  });
  // $store → 스냅샷, store.chart → 액션
</script>

<ChartView chart={store.chart} />
```

| export                      | 역할                                 |
| --------------------------- | ------------------------------------ |
| `createChartStore(options)` | 코어 생성 + `readable` 스토어 래핑   |
| `toChartStore(chart)`       | 외부 코어를 스토어 계약으로 래핑     |
| `ChartView`                 | `use:` 액션으로 공용 DOM 렌더러 연결 |

스냅샷 유도 값을 템플릿에 쓸 때는 비반응 getter 호출 대신 `$store`를
읽는 표현식 안에 둔다 — 스냅샷 갱신마다 재평가된다.
