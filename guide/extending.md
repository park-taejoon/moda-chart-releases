# 기능 추가 워크플로

새 기능은 아래 순서로, 같은 커밋에 반영한다 (AGENTS.md의 규칙).

## 1. 코어

`types.ts`에 옵션/스냅샷 필드를 추가하고 `core.ts`에 로직을 구현한다.
setter는 동일 값에 멱등, 스냅샷은 변경 시에만 새 참조.

## 2. 유닛 테스트

`core.test.ts`(또는 기능별 `*.test.ts`)에 케이스를 추가한다 —
상태 전이, 이벤트 발행, 멱등성, 예외 폴백.

## 3. 5개 렌더러

- `mount.ts` — DOM 계약에 새 요소/속성 추가 (`mc-` 접두어)
- React `ChartView` / Vue3·Vue2 `ChartView.vue` / Svelte `ChartView.svelte` —
  같은 DOM 계약으로 렌더 + prop/이벤트 배선

## 4. 컨포먼스 계약

DOM으로 검증 가능하면 `conformance.ts`에 it 블록을 추가한다 —
선택자 + 상호작용 + 기대 DOM. 5개 어댑터 테스트가 같은 스펙을 실행하므로
한쪽 구현이 빠지면 즉시 실패한다.

## 5. 데모

`apps/dev-*/src/data.ts`의 `FEATURES`에 항목을 추가하고 실제로 옵션을
켠다. 새 옵션을 실제로 쓰는 화면이 없으면 E2E가 검증할 대상이 없다.

## 6. E2E

`e2e/`에 스펙을 추가한다 — 역할이 크면 파일을 나눈다(features/…).
`helpers.ts`의 `apps` 루프로 작성해 5개 렌더러를 동시에 검증한다.

## 7. 문서

- `docs/guide/features/<feature>.md` — 기능 가이드
- 플랫폼별 prop/슬롯이 다르면 `platforms/*.md` 갱신
- `migration.md` — 유사 라이브러리에서 온 사용자를 위한 매핑 표에 추가

## 8. 검증

```bash
pnpm test && pnpm typecheck && pnpm build && pnpm build:apps && pnpm e2e
```
