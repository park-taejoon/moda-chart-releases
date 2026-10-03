# 축과 스케일

## 스케일

| scale    | 대상 | 동작                             |
| -------- | ---- | -------------------------------- |
| `band`   | x    | categories(string) — 등간격 밴드 |
| `linear` | x/y  | 연속 수치 — nice 도메인으로 확장 |
| `time`   | x    | categories(Date) — 시간 눈금     |
| `log`    | y    | `yAxis.scale: "log"` — 로그 눈금 |

- x축 스케일은 `categories`의 값 타입으로 자동 결정된다 (명시하면
  `xAxis.scale`로 override).
- y축 도메인은 데이터 최대값에 여유를 둔 뒤 nice 경계로 확장한다 —
  bar 계열은 0을 포함한다. `min`/`max` 옵션으로 고정 가능.

## 축 옵션

```ts
new ChartCore({
  xAxis: { label: "월", grid: true, format: (v) => `${v}월` },
  yAxis: { label: "금액", tickCount: 6 },
});
```

- `label` — 축 제목 (`.mc-axis-label`)
- `grid` — 플롯 안쪽 그리드선 (`.mc-grid-line`)
- `format` — 눈금 라벨 포맷터
- `tickCount` — 목표 눈금 수 (nice step으로 조정)

## 그룹 카테고리 축

`categories` 항목을 `[그룹, 리프]` 튜플로 주면 x축 아래에 두 번째
틱 행(`.mc-tick-group`)이 생긴다 — 같은 그룹 라벨이 연속되는 구간은
하나의 그룹 눈금으로 묶인다 (AG Charts grouped category 해당).

```ts
categories: [
  ["상반기", "1월"],
  ["상반기", "2월"],
  ["하반기", "3월"],
  ["하반기", "4월"],
];
```

리프 라벨은 첫 틱 행(`.mc-tick-label`), 그룹 라벨은 그 아래
(`.mc-tick-group-label`)에 그려진다. 범례·툴팁·드릴다운 키·주석
매칭에는 리프 라벨이 쓰인다.

## 다중 축

`yAxis`를 배열로 주면 left/right 순서대로 축을 만든다.
시리즈의 `axis` 필드가 축 `id`를 가리킨다:

```ts
new ChartCore({
  yAxis: [
    { id: "money", position: "left", label: "금액" },
    { id: "ratio", position: "right", label: "비율", scale: "log" },
  ],
  series: [
    { name: "매출", values: [...], axis: "money" },
    { name: "성장률", values: [...], axis: "ratio", type: "line" },
  ],
});
```

## 여백

`padding: { top, right, bottom, left }` — svg 가장자리와 플롯 사이.
축 라벨/눈금이 이 영역에 그려진다.
