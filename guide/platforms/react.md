# React

가장 간단한 진입은 `<Chart options>` 단일 컴포넌트다 — 코어 생성·
prop 동기화·DOM 연결·dispose를 내부에서 처리한다.

```tsx
import { Chart } from "@moda-chart/react";
import "@moda-chart/react/styles.css";

function App() {
  return (
    <Chart
      options={{
        type: "line",
        series: [{ name: "매출", values: [12, 24, 16, 30] }],
        categories: ["1월", "2월", "3월", "4월"],
        legend: { position: "top" },
        zoom: true,
      }}
    />
  );
}
```

차트 인스턴스가 필요하면 ref로 받는다 (`<Chart ref={chartRef}>` →
`chartRef.current.setData(...)`).

저수준 경로는 `useChartCore` + `ChartView` 조합이다 — 외부에서 만든
`ChartCore`를 공유하거나 스냅샷을 직접 쓸 때 쓴다:

```tsx
const { chart, snapshot } = useChartCore(options);
return <ChartView chart={chart} />;
```

| export                  | 역할                                             |
| ----------------------- | ------------------------------------------------ |
| `Chart`                 | options 단일 컴포넌트 — 생성·동기화·dispose 내장 |
| `useChartCore(options)` | 코어 생성 + 스냅샷 구독 (`useSyncExternalStore`) |
| `ChartView`             | 스냅샷을 DOM 계약으로 렌더하는 컴포넌트          |

`options`는 반응형 prop으로 동기화된다 — 렌더마다 직전 값과 깊이
비교(`diffOptions`)해 달라진 키만 차트에 부분 병합하고, 사라진 키는
해제한다. 같은 값의 새 객체는 무시하므로 인라인 리터럴도 안전하다.
단 콜백 옵션(`itemStyle`/`events` 등)은 참조 비교라 인라인 함수는
매번 변경으로 잡힌다 — 반복이 문제면 `useMemo`/`useCallback`으로
안정된 참조를 넘긴다. 같은 참조는 비교 없이 즉시 반환하므로, 수만
포인트짜리 `series`를 매 렌더 새로 만드는 패턴은 깊은 비교 비용이
든다 — 대용량 데이터는 `useMemo`로 참조를 안정화하는 게 가장 빠르다.
세밀한 수동 제어는 `chart.setOptions(patch)`(= `updateDelta`)를 쓴다.

이미 만든 `ChartCore`를 공유할 때는 `ChartView`에 `chart`만 넘기면 된다 —
스냅샷 구독과 DOM 렌더는 컴포넌트가 알아서 한다.

## 렌더러 선택

`ChartView`의 `renderer` prop에 대체 렌더러를 넘긴다 — 기본은 내장
SVG 렌더러. Canvas2D로 그리려면:

```tsx
import { createCanvasRenderer } from "@moda-chart/react";

<ChartView chart={chart} renderer={createCanvasRenderer()} />;
```

`ChartRenderer` 계약·lite 경로는 [plugins.md](../plugins.md),
DOM 계약은 [architecture.md](../architecture.md) 참조.
