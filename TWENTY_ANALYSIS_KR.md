# Twenty 전수조사 및 활용 분석 (한국어)

이 문서는 Twenty 저장소를 전수조사하여 정리한 분석 노트다.

## 저장소 정보

- 원본: https://github.com/twentyhq/twenty
- 포크: https://github.com/bmshin94/twenty
- 공식 사이트: https://twenty.com
- 공식 문서: https://docs.twenty.com
- 라이선스: AGPL-3.0
- 버전: 0.2.1
- 모노레포 관리: Nx 22 + Yarn 4 (Node 24.5+)

## 1. 이게 뭔가

Twenty는 오픈소스 CRM이다. 단순한 고객관리 도구가 아니라 "CRM을 코드로 정의하고 배포하는 플랫폼"에 가깝다.

일반 CRM은 완성된 제품을 설정으로만 커스터마이즈한다. Twenty는 객체(테이블), 필드, 뷰, 워크플로우를 TypeScript 코드로 선언하고 워크스페이스에 배포한다. 즉 CRM 스키마를 Git으로 버전 관리한다.

```ts
import { defineObject, FieldType } from 'twenty-sdk/define';

export default defineObject({
  nameSingular: 'deal',
  namePlural: 'deals',
  labelSingular: 'Deal',
  labelPlural: 'Deals',
  fields: [
    { name: 'name', label: 'Name', type: FieldType.TEXT },
    { name: 'amount', label: 'Amount', type: FieldType.CURRENCY },
    { name: 'closeDate', label: 'Close Date', type: FieldType.DATE_TIME },
  ],
});
```

## 2. 패키지 구조 (packages/ 21개)

| 패키지 | 크기 | 역할 |
| --- | --- | --- |
| twenty-server | 106M | 백엔드. NestJS + PostgreSQL + Redis + GraphQL + BullMQ |
| twenty-front | 106M | 프론트. React 19, Jotai, Linaria, Lingui, Vite, Apollo |
| twenty-docs | 98M | 공식 문서 (Mintlify) |
| twenty-website | 50M | twenty.com 마케팅 사이트 (Next.js) |
| twenty-apps | 23M | 공식 앱 및 예제 (slack, linear, discord, exa, fathom, granola, call-recorder 등) |
| twenty-ui | 7.6M | 디자인 시스템, 아이콘 사전 |
| twenty-shared | 5.6M | 프론트/서버 공용 타입과 유틸 |
| twenty-front-component-renderer | 5.1M | 앱이 제공하는 프론트 컴포넌트 실행기 |
| twenty-sdk | 3.7M | 앱 개발 SDK (defineObject, 로직 함수, 프론트 컴포넌트) |
| twenty-companion | 2.3M | Electron 데스크탑 앱 (회의 참석, 녹음, 요약) |
| twenty-client-sdk | 1.6M | 외부에서 Twenty API를 호출하는 클라이언트 |
| twenty-emails | 1.3M | 트랜잭션 이메일 템플릿 (react-email) |
| twenty-docker | 592K | docker-compose, k8s, helm, grafana, podman 배포 세트 |
| twenty-agent-skills | 532K | AI 에이전트용 앱 개발 스킬 5종 |
| twenty-e2e-testing | 312K | Playwright E2E |
| create-twenty-app | 308K | 앱 스캐폴딩 CLI |
| twenty-oxlint-rules | 280K | 저장소 전용 린트 규칙 |
| twenty-utils | 268K | 개발 환경 스크립트 |
| twenty-zapier | 192K | Zapier 연동 |
| twenty-claude-skills | 32K | 워크스페이스 사용자용 스킬 |
| twenty-cli | 16K | 구 CLI (deprecate) |

## 3. 서버 내부 구조

도메인 모듈 (`packages/twenty-server/src/modules/`):
company, person, opportunity, task, note, attachment, calendar, messaging, call-recording, workflow, dashboard, timeline, blocklist, connected-account, contact-creation-manager, emailing, match-participant, onboarding-invite-suggestions, workspace-member 등

코어 모듈 (`packages/twenty-server/src/engine/core-modules/`):
auth, sso, two-factor-authentication, api-key, app-token, billing, feature-flag, file-storage, search, telemetry, metrics, usage-limit, throttler, i18n, workspace, workspace-invitation, tool, tool-provider, code-interpreter, logic-function, record-crud, record-share, message-queue, cron, health, open-api 등

API 계층 (`packages/twenty-server/src/engine/api/`):
- graphql: 메인 API
- rest: REST API
- mcp: Model Context Protocol 서버 (AI 에이전트가 CRM에 직접 연결)

## 4. AI 에이전트 관련 자산

| 경로 | 내용 |
| --- | --- |
| `CLAUDE.md` (AGENTS.md 심볼릭 링크) | 코드베이스 규칙, 명령어, 함정을 에이전트에게 주입 |
| `SKILLS.md` | 스킬 패밀리 3종의 대상과 설치법 |
| `.mcp.json` | 개발용 MCP 서버 설정 (postgres, playwright, context7) |
| `.claude/skills/` | 컨트리뷰터용 스킬. qa-scout, syncable-entity-* 6종 |
| `packages/twenty-agent-skills/skills/` | create-app, develop-app, manage-app, publish-app, use-twenty-mcp |
| `packages/twenty-claude-skills/skills/` | twenty-record-presentation |
| `packages/twenty-server/src/engine/api/mcp/` | Twenty를 AI의 도구로 노출하는 MCP 서버 구현 |
| `.github/workflows/claude.yml` | CI에서 에이전트 실행 |

스킬 설치:

```bash
npx skills add https://github.com/twentyhq/twenty/tree/agent-skills --list
npx skills add https://github.com/twentyhq/twenty/tree/agent-skills --skill create-app
```

## 5. 설치 및 사용법

### 그냥 써보기 (Docker)

```bash
git clone https://github.com/bmshin94/twenty.git
cd twenty/packages/twenty-docker
docker compose up -d
# http://localhost:3000
```

### 개발 환경

```bash
nvm use                                        # .nvmrc: Node 24.5+
yarn install
bash packages/twenty-utils/setup-dev-env.sh    # Postgres + Redis + DB 초기화
npx nx build twenty-shared                     # 필수 선행 빌드
yarn start                                     # front + server + worker
```

프론트 http://localhost:3001, 서버 http://localhost:3000.
로그인은 "Continue with Email"을 누르면 테스트 계정이 미리 채워진다.

### 앱 개발

```bash
npx create-twenty-app my-app
cd my-app
npx twenty app:publish --private
```

### 자주 쓰는 명령어

```bash
npx nx lint:diff-with-main twenty-server     # 변경분만 린트
npx nx test twenty-server                    # 유닛 테스트
npx nx run twenty-front:graphql:generate     # GraphQL 스키마 변경 후
npx nx database:reset twenty-server
npx jest path/to/file.spec.ts --config=packages/<pkg>/jest.config.mjs
```

### 함정

1. 브랜치 전환이나 twenty-shared 수정 후에는 `npx nx build twenty-shared --skip-nx-cache`를 먼저 실행한다. 안 그러면 엉뚱한 타입 에러를 디버깅하게 된다.
2. Nx 캐시가 오래된 성공 결과를 줄 수 있다. 검증은 패키지 디렉터리에서 `npx tsgo -p tsconfig.json --noEmit`으로 한다.
3. 번역 카탈로그(`locales/*.po`)는 커밋하지 않는다. 수천 줄의 불필요한 diff가 생긴다.

## 6. 플러그인인가, 스킬인가, MCP인가

셋 다 아니고, 셋 다 포함한 애플리케이션이다.

| 정체 | 설명 |
| --- | --- |
| 본체 | 서버 + 프론트로 이뤄진 완전한 웹 애플리케이션 |
| MCP 서버 (제공) | Twenty가 AI에게 CRM 도구를 제공 (`engine/api/mcp/`) |
| MCP 클라이언트 (사용) | 개발 시 쓰는 MCP 3종 (`.mcp.json`) |
| 스킬 | 대상별 3개 패밀리 (앱 개발자용 / 워크스페이스 사용자용 / 컨트리뷰터용) |
| 앱 | Twenty 위에 얹는 확장 (`twenty-apps/`) |

## 7. API 토큰

| 상황 | 토큰 |
| --- | --- |
| 웹 UI로 CRM 사용 | 불필요 (이메일/SSO 로그인) |
| 로컬 개발 | 불필요 |
| 외부 스크립트가 API 호출 | 필요 (API Key) |
| Zapier, 슬랙 봇, n8n 연동 | 필요 |
| AI 에이전트가 MCP로 연결 | 필요 |
| 앱 배포 (app:publish) | 필요 |
| AI 기능 사용 | 별도 LLM 제공자 키 필요 |

발급은 워크스페이스 Settings > API & Webhooks. 서버 구현은 `engine/core-modules/api-key/`에 있다.

```bash
curl -H "Authorization: Bearer <YOUR_API_KEY>" \
     -H "Content-Type: application/json" \
     https://your-instance/rest/people
```

키는 `.env`로 관리하고 커밋하지 않는다.

## 8. 왜 GitHub에서 유명한가

1. 세일즈포스 대체제라는 명확한 서사
2. Notion, Linear급의 완성도 높은 UI와 공개된 Figma 파일
3. 개발자에게 익숙한 모던 스택 (TypeScript, NestJS, React, GraphQL, Nx)
4. 높은 코드 품질. CI 워크플로우 40개 이상, E2E, Storybook, 엄격한 타입
5. package.json의 `//resolutions` 주석처럼 보안 패치 근거까지 문서화하는 관리 수준
6. 좋은 문서와 활발한 Discord로 컨트리뷰션 친화적
7. MCP 내장, AI 에이전트, 에이전트 스킬 등 AI 트렌드 정중앙
8. 투자를 받은 팀이 풀타임으로 개발
9. AGPL 기반의 오픈코어 전략으로 커뮤니티 신뢰 확보

## 9. 로컬 에이전트 구축에 주는 도움

바로 참고할 코드:

- `.mcp.json`: stdio 방식 MCP 서버 연결과 환경변수 주입 패턴
- `engine/api/mcp/`: 자체 서비스를 AI 도구로 노출하는 프로덕션 구현
- `engine/core-modules/tool/`, `tool-provider/`: 툴 등록 및 실행 레지스트리
- `engine/core-modules/code-interpreter/`: AI 생성 코드의 격리 실행
- `CLAUDE.md`: 에이전트에게 코드베이스 규칙을 주입하는 모범 사례
- `packages/twenty-agent-skills/`: SKILL.md + references + templates 구조
- `.claude/skills/qa-scout/`: 에이전트가 브라우저로 QA하고 SQL로 검증하는 워크플로우
- `.github/workflows/claude.yml`, `ci-agent-skills-drift.yaml`: CI에서 에이전트 운영

학습 순서: MCP 연결 → MCP 서버 구현 → 스킬 구조 → 툴 레지스트리 → 검증 루프.

## 10. React / PHP로 만들 수 있나

React는 이미 React다. `twenty-front`가 React 19 + TypeScript + Vite이며, 상태는 Jotai, 스타일은 Linaria, 다국어는 Lingui, 데이터는 Apollo + GraphQL을 쓴다. `twenty-sdk`로 프론트 컴포넌트를 앱으로 만들어 Twenty 화면에 삽입할 수도 있다.

PHP는 세 가지 경우를 구분해야 한다.

- Twenty 자체를 PHP로 포팅: 비추천. 워크스페이스별 동적 스키마와 마이그레이션 엔진이 핵심이라 재구현 비용이 매우 크다.
- PHP에서 Twenty와 연동: 권장. REST/GraphQL API가 열려 있어 Laravel의 Http 클라이언트로 바로 호출 가능하다.
- PHP로 유사 제품 신규 개발: 가능하지만 동적 객체 정의와 마이그레이션까지 구현하면 난이도가 급상승한다.

현실적인 전략은 백엔드로 Twenty를 그대로 쓰고, 익숙한 기술로 고객 대면 화면만 별도 구성하는 것이다.

## 11. 수익화 아이디어

### 전제: AGPL-3.0

| 하는 일 | 소스 공개 의무 |
| --- | --- |
| 사내 내부 사용 | 없음 |
| 고객사 서버에 설치 후 대행 | 해당 고객사에 한해 제공 |
| 수정한 버전을 SaaS로 서비스 | 수정 소스 전부 공개 의무 |
| API로만 연동하는 별도 서비스 | 일반적으로 없음 (경계 사례는 법률 검토 필요) |

따라서 "수정 후 SaaS 판매"는 위험하고, "구축, 운영, 확장, 연동"이 안전하다.

### Tier 1: 즉시 시작 가능

1. 셀프호스팅 구축 및 운영 대행. 구축 300~800만원, 운영 월 30~100만원. `twenty-docker`에 배포 자산이 갖춰져 있어 진입 장벽이 낮다.
2. 한국형 연동 앱 판매. 공식 앱 목록이 전부 서구권 서비스라 카카오 알림톡, 네이버웍스, 세금계산서(팝빌/바로빌), 국내 PG, 사업자등록번호 조회, 도로명주소 연동이 공백이다.
3. 업종 특화 템플릿 판매. 부동산(`twenty-apps/internal/real-estate` 참고), 학원, 병의원, 웨딩, 헬스장 등. 코드로 정의되어 재판매 마진이 높다.

### Tier 2: 3~6개월 후

4. 매니지드 호스팅. 본체를 수정하지 않고 순정 배포하며 커스텀은 전부 앱으로 분리하면 공개 의무가 발생할 수정본이 없다.
5. AI 에이전트 애드온. MCP 엔드포인트가 이미 있어 인프라를 새로 만들 필요가 없다. 팔로업 자동화, 통화 녹음 요약, 리드 스코어링 등.
6. 마이그레이션 서비스. 세일즈포스, 허브스팟, 엑셀에서의 데이터 이관.

### Tier 3: 간접 수익

7. 한국어 콘텐츠 및 강의. 자료가 거의 없어 선점 효과가 크고, 여기서 나오는 리드가 Tier 1 수주로 이어진다.
8. 오픈소스 컨트리뷰션을 통한 커리어 및 단가 상승.

### 추천 로드맵

1. 1개월: 로컬 구축, 코드 파악, hello-world 앱 따라 만들기
2. 2~3개월: 카카오 알림톡 연동 앱 완성 후 공개 (포트폴리오 겸 마케팅)
3. 3~4개월: 유입된 문의로 구축 대행 첫 수주
4. 5~6개월: 수주 경험을 업종 템플릿으로 패키징
5. 6개월 이후: AI 에이전트 애드온으로 구독 매출 확보

가장 먼저 할 하나를 고른다면 카카오톡 알림톡 연동 앱이다. 수요가 명확하고, 경쟁이 없으며, 상품과 포트폴리오 역할을 동시에 한다.

## 12. 참고 링크

- 원본 저장소: https://github.com/twentyhq/twenty
- 포크 저장소: https://github.com/bmshin94/twenty
- 문서: https://docs.twenty.com
- 앱 개발 가이드: https://docs.twenty.com/developers/extend/apps/getting-started
- 셀프호스팅(Docker Compose): https://docs.twenty.com/developers/self-host/capabilities/docker-compose
- 로컬 셋업: https://docs.twenty.com/developers/contribute/capabilities/local-setup
- 로드맵: https://github.com/orgs/twentyhq/projects/1
- Discord: https://discord.gg/cx5n4Jzs57
