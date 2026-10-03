# moda-chart 문서

멀티 프레임워크 헤드리스 차트 라이브러리 모노레포.

## 사용자 가이드

[guide/](./guide/README.md) — 플랫폼별·기능별 사용자 가이드.
`docs/` 변경이 main에 push되면 GitHub Action(`deploy-docs.yml`)이
공개 레포 `moda-chart-releases`로 자동 배포한다.
새 기능 개발 시 문서 추가 규칙: [guide/extending.md](./guide/extending.md).

주요 문서:

| 문서                                                               | 내용                              |
| ------------------------------------------------------------------ | --------------------------------- |
| [guide/getting-started.md](./guide/getting-started.md)             | 설치·첫 차트·템플릿 사용법        |
| [guide/architecture.md](./guide/architecture.md)                   | 코어/어댑터/스냅샷 계약           |
| [guide/features/chart-types.md](./guide/features/chart-types.md)   | 차트 타입 전체 목록과 데이터 모델 |
| [guide/features/interactions.md](./guide/features/interactions.md) | 줌/드릴다운/선택/키보드/그리기    |
| [guide/migration.md](./guide/migration.md)                         | AG Charts·IBChart API 매핑 표     |
| [guide/benchmark.md](./guide/benchmark.md)                         | AG Charts·IBChart 기능 벤치마크   |
| [guide/theming.md](./guide/theming.md)                             | `--chart-*` CSS 변수와 DOM 계약   |

## 빠른 시작

```bash
pnpm install          # 워크스페이스 전체 설치
pnpm build            # packages/* 전체 빌드 (토폴로지 순서 자동)
pnpm typecheck        # 전체 타입 검사
pnpm test             # core vitest + 어댑터 컨포먼스

pnpm dev:react        # http://localhost:5173
pnpm dev:vue          # http://localhost:5174 (Vue 3)
pnpm dev:svelte       # http://localhost:5175
pnpm dev:vue2         # http://localhost:5176 (Vue 2.7)
pnpm dev:vanilla      # http://localhost:5177 (mountChart, 프레임워크 없음)
pnpm dev              # 5개 앱 watch + 탭 셸로 통합 확인

pnpm demo             # 전체 데모 빌드 후 통합 서빙
pnpm e2e              # Playwright 데모 스모크 (dev 서버 자동 기동)
```

각 데모 앱 상단의 "기능 체크리스트" 패널에서 현재 구현된 기능을 확인할 수 있다.

### Docker

```bash
docker compose up --build                           # 프로덕션 통합 데모 서버
docker compose -f docker-compose.dev.yml up --build # watch — 소스 마운트 + HMR
```

## 배포

- **npm** — `pnpm release patch --push`로 버전·태그를 맞추면
  `publish-npm.yml`이 `@moda-chart/*` 5개 패키지를 배포한다
  (Secrets: `NPM_TOKEN`).
- **문서** — `docs/` 변경 push 시 `deploy-docs.yml`이 공개 레포
  `moda-chart-releases`로 동기화한다 (Secrets: `RELEASE_REPO_TOKEN`).
  소스 레포가 private이어도 문서만 공개된다.
- **데모/CDN 페이지** — `v*` 태그 push 시 `deploy-cdn.yml`이
  `pnpm build:site` 결과물을 Cloudflare Pages(`moda-chart-cdn`)에 배포하고
  `chart.modaolive.com`을 연결한다 (Secrets: `CLOUDFLARE_API_TOKEN`,
  `CLOUDFLARE_ACCOUNT_ID`). 루트는 5개 렌더러 데모 탭 셸이고,
  `/moda-chart.js`(IIFE, `window.ModaChart`)·`/style.css`는 CDN 번들로
  그대로 쓸 수 있다:

  ```html
  <link rel="stylesheet" href="https://chart.modaolive.com/style.css" />
  <script src="https://chart.modaolive.com/moda-chart.js"></script>
  ```
