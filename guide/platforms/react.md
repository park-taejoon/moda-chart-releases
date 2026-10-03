# React

```tsx
import { ChartView, useChartCore } from "@moda-chart/react";
import "@moda-chart/react/styles.css";

function App() {
  const { chart, snapshot } = useChartCore({
    type: "line",
    series: [{ name: "매출", values: [12, 24, 16, 30] }],
    categories: ["1월", "2월", "3월", "4월"],
    legend: { position: "top" },
    tooltip: { mode: "shared" },
    zoom: true,
  });
  return <ChartView chart={chart} />;
}
```

| export                  | 역할                                             |
| ----------------------- | ------------------------------------------------ |
| `useChartCore(options)` | 코어 생성 + 스냅샷 구독 (`useSyncExternalStore`) |
| `ChartView`             | 스냅샷을 DOM 계약으로 렌더하는 컴포넌트          |

`options`는 최초 마운트에 1회만 적용된다 — 이후 데이터·타입 변경은
`chart.setData` / `chart.setType` 같은 코어 메서드로 한다.

이미 만든 `ChartCore`를 공유할 때는 `ChartView`에 `chart`만 넘기면 된다 —
스냅샷 구독과 DOM 렌더는 컴포넌트가 알아서 한다.
