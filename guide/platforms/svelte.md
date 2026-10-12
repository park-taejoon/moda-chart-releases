# Svelte 5

가장 간단한 진입은 `<Chart {options}>` 단일 컴포넌트다 — 코어 생성·
prop 동기화·DOM 연결·dispose를 내부에서 처리한다.

```svelte
<script lang="ts">
  import { Chart } from "@moda-chart/svelte";
  import "@moda-chart/svelte/styles.css";

  const options = {
    type: "line" as const,
    series: [{ name: "매출", values: [12, 24, 16, 30] }],
    categories: ["1월", "2월", "3월", "4월"],
  };
</script>

<Chart {options} />
```

차트 인스턴스는 `bind:chart`로 받는다. 저수준 경로는
`createChartStore` + `ChartView` 조합이다:

```svelte
<script lang="ts">
  import { ChartView, createChartStore } from "@moda-chart/svelte";

  const store = createChartStore(options);
  // $store → 스냅샷, store.chart → 액션
</script>

<ChartView chart={store.chart} />
```

| export                      | 역할                                 |
| --------------------------- | ------------------------------------ |
| `Chart`                     | options 단일 컴포넌트 (bind:chart)   |
| `createChartStore(options)` | 코어 생성 + `readable` 스토어 래핑   |
| `toChartStore(chart)`       | 외부 코어를 스토어 계약으로 래핑     |
| `ChartView`                 | `use:` 액션으로 공용 DOM 렌더러 연결 |

스냅샷 유도 값을 템플릿에 쓸 때는 비반응 getter 호출 대신 `$store`를
읽는 표현식 안에 둔다 — 스냅샷 갱신마다 재평가된다.

options을 바깥에서 바꿀 때는 `store.syncOptions(next)`를 호출한다 —
직전 값과 깊이 비교(`diffOptions`)해 달라진 키만 병합하고 사라진
키는 해제한다. 반응 블록(`$: store.syncOptions(options)`)에서 반복
호출해도 변경이 없으면 무효화되지 않는다. 같은 객체를 제자리 변이해
넘겨도 적용 상태와 비교해 반영된다 — 단 키 제거는 새 객체로 넘겨야
감지된다.

## 렌더러 선택

`ChartView`의 `renderer` prop에 `createCanvasRenderer()` 등 대체
렌더러를 넘긴다 — 기본은 내장 SVG 렌더러.
계약·lite 경로는 [plugins.md](../plugins.md) 참조.
