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

| 위치                                       | 내용                                                                                               |
| ------------------------------------------ | -------------------------------------------------------------------------------------------------- |
| `packages/core/src/types.ts`               | 타입 barrel — `types/` 재export                                                                    |
| `packages/core/src/types/`                 | input(옵션)/input-domain(도메인 옵션)/geometry(지오메트리 뷰)/views(스냅샷 크롬)/events/api        |
| `packages/core/src/scales.ts`              | 도메인/눈금/스케일 수학                                                                            |
| `packages/core/src/geometry.ts`            | line/area/arc path 빌더                                                                            |
| `packages/core/src/hierarchy.ts`           | 계층 정규화 + treemap/sunburst/chord                                                               |
|                                            | 레이아웃                                                                                           |
| `packages/core/src/org-layout.ts`          | 조직도 레이아웃 — orientation/접기 지원                                                            |
| `packages/core/src/render/organization.ts` | 조직도 카드/링크 이미터 — 사진·renderNode·panZoom 변환                                             |
| `packages/core/src/org-edit.ts`            | 조직도 트리 불변 편집 — 스파인 클론 + 접힘 path 리매핑 (findTreeNode/updateChildren/remapOrgPaths) |
| `packages/core/src/core.ts`                | `ChartCore` — 내장 빌더 주입 얇은 셸                                                               |
| `packages/core/src/core-impl.ts`           | 공개 API 파사드 + 스냅샷 조립                                                                      |
| `packages/core/src/lite.ts`                | lite 경로 — 빌더 선택 번들(서브패스)                                                               |
| `packages/core/src/view.ts`                | 뷰 계층 — 스켈레톤/renderInto/morph                                                                |
| `packages/core/src/domain.ts`              | 도메인 순수 계산 — 정규화/히스토그램/                                                              |
|                                            | 카테고리/라벨/LTTB·그리드 다운샘플링                                                               |
| `packages/core/src/locale.ts`              | 내장 문구 로케일 — StrKey + ko/en                                                                  |
| `packages/core/src/maps/`                  | 내장 지형 데이터 — `maps/world` 서브패스                                                           |
| `packages/core/src/view-bindings.ts`       | 인터랙션 배선 — 포인터 상태 머신 + 서브 바인더 오케스트레이션                                      |
| `packages/core/src/nav-bindings.ts`        | 네비게이터/스크롤바 드래그 바인딩                                                                  |
| `packages/core/src/toolbar-bindings.ts`    | 툴바/범례/메뉴 위임 클릭 + 전체화면                                                                |
| `packages/core/src/org-bindings.ts`        | 조직도 검색 입력 + 미니맵 바인딩                                                                   |
| `packages/core/src/keyboard-bindings.ts`   | 키보드 포인트 내비 + 포커스 좌표 해석                                                              |
| `packages/core/src/frame-channels.ts`      | 프레임 병합 — 채널별 leading+trailing rAF 큐                                                       |
| `packages/core/src/view-download.ts`       | SVG/PNG 보내기                                                                                     |
| `packages/core/src/mount.ts`               | mountChart + 보내기 헬퍼                                                                           |
| `packages/core/src/snapshot/`              | 스냅샷 부속 뷰 빌더 — 축/라벨/마크/                                                                |
|                                            | 오버레이/범례/툴팁/선택 순수 함수                                                                  |
| `packages/core/src/build/`                 | 스냅샷 지오메트리 — 타입별 순수 빌더 +                                                             |     |     | `registry.ts` 체인 (`buildGeometry`) |
| `packages/core/src/controllers/`           | 상태 슬라이스 — data/viewport/                                                                     |
|                                            | interaction/drawing/map-state/sync/organization                                                    |
| `packages/core/src/features/`              | 피처 계약 — `CoreHost`/`ChartFeature`                                                              |
|                                            | (`options.features`/`chart.use()`)                                                                 |
| `packages/core/src/hit-test.ts`            | px 좌표 → 포인트 해석 디스패처 (HitContext 주입)                                                   |
| `packages/core/src/hit-*.ts`               | 타입 패밀리별 hitTest 리졸버 — hit-radial/hit-flow/hit-misc/hit-cartesian                          |
| `packages/core/src/renderer.ts`            | `ChartRenderer` 계약 — 스냅샷→화면 경계                                                            |
| `packages/core/src/render/`                | 지오메트리 이미터 — kind별 순수 함수 +                                                             |
|                                            | `registry.ts` 체인 (`emitGeometry`)                                                                |
| `packages/core/src/svg-impl.ts`            | SVG 합성 — 축/그리드/오버레이 +                                                                    |
|                                            | 이미터 체인 주입본 (`svgInnerImpl`)                                                                |
| `packages/core/src/svg.ts`                 | 풀 진입 — 내장 이미터 주입 `svgInner`                                                              |
|                                            | (기본 `svgRenderer`는 mount.ts 소유)                                                               |
| `packages/core/src/canvas.ts`              | Canvas2D 렌더러 — `createCanvasRenderer`                                                           |
|                                            | (`canvas.mc-canvas` + `updateChartChrome` 공유)                                                    |
| `packages/core/src/canvas/`                | 캔버스 드로우 — theme(토큰 해석)/                                                                  |
|                                            | context(색·패턴·이미지 캐시)/draw(축·그리드·                                                       |
|                                            | 오버레이 합성)                                                                                     |
| `packages/core/src/canvas/geometries/`     | 캔버스 지오메트리 — 패밀리별 드로우 +                                                              |
|                                            | `../geometries.ts` 디스패처 (render/ 대칭)                                                         |
| `packages/core/src/webgl.ts`               | WebGL 렌더러 — `createWebglRenderer`.                                                              |
|                                            | GL 캔버스(지오메트리) + 2D 오버레이                                                                |
|                                            | (`canvas.mc-canvas-2d`, 크롬/미지원 패밀리)                                                        |
| `packages/core/src/webgl/`                 | GL 드로우 — context(셰이더/버퍼)·draw                                                              |
|                                            | (스냅샷→프리미티브)·path(SVG path 평탄화)                                                          |
| `packages/core/src/conformance.ts`         | 공용 계약 스펙 엔트리 —                                                                            |
|                                            | `runChartConformance` 조합기                                                                       |
| `packages/core/src/conformance/`           | 스펙 본문 — context(마운트/픽스처) +                                                               |
|                                            | basic/ibchart/ag/gap describe 그룹                                                                 |
| `packages/*/src`                           | 어댑터 — 바인딩만, 로직 없음                                                                       |
| `apps/dev-*`                               | 데모 — 기능 체크리스트 + 실제 옵션 사용                                                            |
| `e2e/`                                     | Playwright — helpers.ts의 apps 루프                                                                |
| `docs/guide/`                              | 기능·플랫폼 가이드                                                                                 |
