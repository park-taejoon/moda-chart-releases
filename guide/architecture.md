# 아키텍처

## 헤드리스 코어 + 얇은 어댑터

```
옵션 → ChartCore(상태) → getSnapshot()(불변 스냅샷) → subscribe 리스너
                                                     ↓
                          React useSyncExternalStore / Vue shallowRef /
                          Svelte readable / vanilla 직접 렌더
```

- **코어(`@moda-chart/core`)** — 모든 로직. 프레임워크에 무관하다.
  스냅샷 → SVG 문자열(`svg.ts`)과 DOM 반영·인터랙션 배선
  (`mount.ts`의 `attachChartView`)까지 코어가 제공한다. 어댑터는
  `attachChartView`를 자기 생명주기에 연결할 뿐 직접 그리지 않는다.
- **어댑터(`@moda-chart/*`)** — 코어 스냅샷을 각 프레임워크의 반응성
  시스템으로 연결하고, 마운트/언마운트에 `attachChartView`를 배선한다.
  로직을 두지 않는다.

## 핵심 계약

1. `getSnapshot()`은 상태가 바뀌기 전까지 **같은 참조**를 반환한다 —
   React `useSyncExternalStore`·Svelte `$derived`가 이에 의존한다.
2. setter는 **동일 값 재호출에 멱등** — 어댑터의 반응형 동기화 루프를
   깨지 않는다.
3. 모든 렌더러는 **같은 DOM 계약**(`mc-*` 클래스)을 만든다 —
   `runChartConformance`가 5개 렌더러를 같은 스펙으로 검증한다.
4. 이벤트는 `events` 옵션 + `chart.on(type, fn)` — 어댑터는 이벤트를
   프레임워크 emit/prop 콜백으로 변환한다.

## 파일 배치 규칙

| 위치                               | 내용                                    |
| ---------------------------------- | --------------------------------------- |
| `packages/core/src/types.ts`       | 옵션·스냅샷·이벤트 타입                 |
| `packages/core/src/scales.ts`      | 도메인/눈금/스케일 수학                 |
| `packages/core/src/geometry.ts`    | line/area/arc path 빌더                 |
| `packages/core/src/core.ts`        | 상태 로직 + 스냅샷 계산                 |
| `packages/core/src/svg.ts`         | 스냅샷 → SVG 문자열 (렌더/export 공용)  |
| `packages/core/src/mount.ts`       | DOM 반영 + 인터랙션 배선 + mountChart   |
| `packages/core/src/conformance.ts` | 5렌더러 공용 DOM 계약 스펙              |
| `packages/*/src`                   | 어댑터 — 바인딩만, 로직 없음            |
| `apps/dev-*`                       | 데모 — 기능 체크리스트 + 실제 옵션 사용 |
| `e2e/`                             | Playwright — helpers.ts의 apps 루프     |
| `docs/guide/`                      | 기능·플랫폼 가이드                      |
