# AI Agent 운영 규칙 (moneylog-project)

> 이 문서는 **Coding Agent(AI)가 작업을 시작하기 전에 참조하는 절차·의사결정 규칙**이다.
> 기술 스펙(스택 버전·데이터 모델·API·UI·에러 코드 등) 자체는 여기서 다루지 않는다 — 그것은 `CLAUDE.md`(루트)가 정본이며, 이 문서는 그것을 대체하지 않는다.
> 이 문서에 없는 내용은 general dev knowledge이므로 스스로 판단한다. **여기 있는 규칙만 이 프로젝트에 특수하다.**

---

## 1. 정본 우선순위 (충돌 시 판단 순서)

문서 간 내용이 어긋나면 아래 순서로 우선한다. **낮은 순위 문서를 고쳐서 맞추려 하지 말고, 먼저 사용자에게 질문한다.**

1. `moneylog-project/CLAUDE.md` (루트) — 기술 규칙의 단일 기준
2. `moneylog-backend/CLAUDE.md`, `moneylog-frontend/CLAUDE.md` — 해당 저장소 전용 빌드/계층 규칙만. 전역 스펙과 충돌하면 안 되고, 충돌 발견 시 루트 `CLAUDE.md`를 고친다
3. `docs/PRD.md` — 무엇을 만드는가 (요구사항)
4. `docs/ROADMAP.md` — 어떤 순서로 만드는가 + Phase별 완료 판정(DoD)의 정본

**금지**: PRD.md·ROADMAP.md의 요구사항이 CLAUDE.md의 기술 규칙과 충돌하는 것을 발견했을 때, 임의로 한쪽을 골라 구현하지 않는다. 사용자에게 어느 쪽이 최신인지 확인한다.

---

## 2. 폴리레포 경계 (반드시 지킬 것)

이 작업 공간은 **3개의 독립 Git 저장소**다. 모노레포처럼 다루면 안 된다.

| 디렉토리 | 소속 저장소 | 커밋 위치 |
|---|---|---|
| `CLAUDE.md`, `docs/*.md` | `moneylog-project` (문서 저장소) | 루트에서 커밋 |
| `moneylog-backend/**` | `moneylog-backend` (독립 저장소) | `moneylog-backend/`에서 커밋 |
| `moneylog-frontend/**` | `moneylog-frontend` (독립 저장소) | `moneylog-frontend/`에서 커밋 |
| `mcp-shrimp-task-manager/**` | 이 프로젝트와 무관한 MCP 서버 도구. 수정하지 않는다 | — |

**규칙**
- **커밋을 저장소 경계 너머로 섞지 않는다.** 백엔드 변경과 프론트엔드 변경을 한 커밋에 담지 않는다. 문서 변경(루트)과 코드 변경(하위 저장소)도 섞지 않는다.
- 루트에서 `git add`/`git commit`을 실행하기 전, `git status`로 `moneylog-backend/`·`moneylog-frontend/` 내부 파일이 스테이징되지 않았는지 확인한다. 루트 `.gitignore`가 이 두 폴더를 제외하도록 되어 있으므로, 만약 스테이징된다면 `.gitignore` 설정이 깨진 것이니 커밋하지 말고 먼저 원인을 확인한다.
- **커밋은 항상 사용자에게 확인받은 뒤 실행한다** (어느 저장소든 동일).
- `mcp-shrimp-task-manager/`는 moneylog 프로젝트의 산출물이 아니다. 이 폴더의 파일을 읽거나 참고할 수는 있어도, 프로젝트 작업으로 수정하지 않는다.

---

## 3. 파일 협조 매트릭스 (하나를 고치면 같이 고쳐야 하는 것)

| 변경 대상 | 함께 확인/수정해야 하는 대상 | 이유 |
|---|---|---|
| API 계약 (엔드포인트, 요청/응답 필드, 에러 코드) | ① 루트 `CLAUDE.md` §5 먼저 수정 → ② `moneylog-backend` 구현 → ③ `moneylog-frontend` 구현 | §13 "API 계약이 바뀌면 문서 저장소를 먼저 수정" — 순서를 어기면 백엔드와 프론트가 서로 다른 계약을 구현한다 |
| `docs/ROADMAP.md`의 Phase 순서·DoD 항목 | 해당 Phase에 걸린 §12 테스트 번호, §4 데이터 모델, §5 API 표와 정합성 확인 | ROADMAP은 "완료 판정의 정본"이므로 다른 장과 어긋나면 그 자체가 버그다 |
| `docs/PRD.md`의 요구사항 ID(`TXN-xx`, `AUTH-xx` 등) 추가/변경 | `docs/ROADMAP.md`의 "요구사항 ↔ Phase" 매핑 표(48번째 줄 부근)에 행 추가 | 표에 없는 P0 요구사항은 구현되지 않는다고 CLAUDE.md 3장이 명시 |
| 루트 `CLAUDE.md` §3(스택 버전) | `moneylog-backend/CLAUDE.md`, `moneylog-frontend/CLAUDE.md`의 실행 명령·설치 금지 목록이 여전히 일치하는지 확인 | 하위 저장소 문서는 루트를 요약 인용하므로 어긋나면 단독 클론 시 잘못된 규칙만 남는다 |
| 백엔드 `pom.xml`의 Spring Boot 버전 | `springdoc-openapi-starter-webmvc-ui` 버전 대응표(CLAUDE.md §3) 재확인 | 두 버전은 1:1 대응하며 범위 지정 시 마일스톤이 딸려 들어올 수 있다 |
| `TransactionResponse`류 DTO의 필드 | 프론트 `src/types/`의 대응 타입 | CLAUDE.md 10장 "타입은 백엔드 DTO와 이름을 맞춘다" |

---

## 4. 작업 순서 게이팅 (Phase를 건너뛰지 않는다)

- **`docs/ROADMAP.md`는 Phase 단위 진행을 강제한다.** 현재 Phase보다 앞선 Phase에 속한 파일·기능을 먼저 만들지 않는다. 예: 백엔드 Phase 2(도메인&DB)가 끝나지 않았는데 Phase 4의 컨트롤러를 먼저 작성하지 않는다.
- 작업을 시작하기 전 `docs/ROADMAP.md`에서 해당 Phase의 "작업" 목록과 "DoD" 체크리스트를 먼저 읽는다. 임의로 범위를 넓히거나 다음 Phase 일부를 앞당기지 않는다.
- **백엔드(Phase 1~6)가 프론트엔드(Phase 7~11)보다 먼저 끝나야 한다.** 프론트 화면의 DoD 대부분이 서버 응답에 의존하기 때문이다(CLAUDE.md 13장).
- 한 Phase가 끝나면 실행/검증 결과를 보고하고 **다음 Phase로 자동 진행하지 않는다.** 사용자의 진행 지시를 기다린다.
- Phase 완료 후 태그(`v0.{Phase번호}.0`)를 붙이는 것도 사용자 확인 후에 한다.

---

## 5. 재확인·재조사 절대 금지 (이미 결정된 사실)

아래는 루트 `CLAUDE.md` §3에서 이미 확정되었고, **다시 조사하거나 최신 버전으로 올리려 시도하면 안 되는** 값이다. 웹 검색이나 `npm outdated`류 명령으로 "더 새 버전이 있는지" 확인하는 시도 자체를 하지 않는다.

- Next.js **15** 고정 (16 금지) — Amplify SSR 지원 범위 때문
- TypeScript **5.x** 고정 (7 금지) — 네이티브 컴파일러가 JS API 미지원
- Spring Boot **4.1.1** + SpringDoc **3.1.1** 고정 — 마일스톤(`4.2.0-M1`) 사용 금지
- jjwt **0.13.0**, 문법은 0.12 스타일(`Jwts.SIG`, `verifyWith`) 고정
- Node.js **22 이상**(권장 24 LTS), 26 금지(LTS 아님)
- 애니메이션은 `motion`(구 framer-motion 이름만 다름), import는 `motion/react`에서만

이 값을 바꿔야 한다는 판단이 들면(예: 보안 패치, EOL) 코드를 먼저 바꾸지 말고 **사용자에게 근거를 제시하며 먼저 질문한다.**

---

## 6. 절대 생성/도입 금지 목록

아래 파일·구조·의존성을 만들거나 설치하려는 시도가 있으면 **그 전에 멈추고 사용자에게 확인한다.** (이유는 루트 CLAUDE.md 해당 장 참조, 여기서는 재설명하지 않음)

| 금지 대상 | 위치/맥락 |
|---|---|
| `moneylog-frontend/src/app/(main)/transactions/new/page.tsx` | §7 — 퀵 입력 바로 대체 |
| `moneylog-frontend/middleware.ts` | §9 — 토큰이 localStorage에 있어 무의미 |
| `moneylog-frontend/public/static/**` | §3 — Amplify 예약 경로 |
| `next.config.ts`의 `distDir` 설정 | §3 |
| `react-hook-form`, `zod`, `@hookform/resolvers`, `recharts`(교체 조건 전), `opencsv`, `commons-csv`, `framer-motion` 설치 | §3 |
| shadcn `form` 컴포넌트 추가(`npx shadcn add form`) | §3 — react-hook-form을 함께 끌어온다 |
| `moneylog-backend`의 반복 거래 스케줄러, Flyway, `application.properties`, OAuth2 의존성, Testcontainers/H2 | §1, §3, `moneylog-backend/CLAUDE.md` |
| Docker 관련 파일(`Dockerfile`, `docker-compose.yml`) | §1 — 전 범위에서 미사용 |
| `TrendLine`(월별 추이 선) 차트 컴포넌트 | §3 — 이번 범위 밖 |
| 화면 코드에서 SVG 직접 그리기, 차트 컴포넌트가 색상을 자체 결정 | §3 — `src/components/chart/` 밖으로 차트 구현이 새면 안 됨 |
| `src/components/chart/` 밖에서 Recharts/SVG 직접 참조 | §3 |

---

## 7. 모호한 상황 의사결정 트리

- **문서 간 불일치를 발견했을 때** → §1의 우선순위로 어느 쪽이 정본인지 판단하되, 코드를 먼저 작성하지 않고 발견한 불일치를 사용자에게 보고 후 확인받는다.
- **CLAUDE.md·PRD.md·ROADMAP.md 어디에도 없는 결정이 필요할 때** (예: 새 유틸 함수의 위치, 명시되지 않은 엣지케이스) → 루트 CLAUDE.md 서두의 원칙("임의로 진행하지 말고 먼저 질문한다")을 그대로 따른다. 그럴듯한 기본값을 만들어 진행하지 않는다.
- **§5의 고정 버전을 올려야 할 것 같은 신호(취약점 경고, deprecated 경고)를 만났을 때** → 버전을 바꾸지 말고, 근거(경고 메시지, CVE 등)를 사용자에게 제시하고 지시를 기다린다.
- **하위 저장소 CLAUDE.md에 없는 빌드/실행 문제를 만났을 때** → 하위 저장소 문서를 임의로 확장하지 말고, 우선 루트 CLAUDE.md 10장(코딩 컨벤션·환경 변수·실행)을 확인한다. 거기에도 없으면 질문한다.
- **ROADMAP Phase 범위를 벗어나는 요청을 받았을 때** (예: 아직 Phase 2인데 화면 작업 요청) → 선행 조건 미충족을 알리고, 그래도 진행할지 사용자에게 확인한다. 임의로 순서를 건너뛰지 않는다.

---

## 8. 문서 작성 언어

- 커밋 메시지, 코드 주석, 이 프로젝트에서 생성하는 모든 `.md` 문서는 **한국어**로 작성한다.
- 변수명·함수명·타입명은 영어(코드 표준)를 유지한다. 한국어 규칙은 서술·주석·문서에만 적용한다.
- 커밋 메시지 작성 시 `git:commit` 스킬을 사용해 컨벤셔널 커밋 형식(`feat:`, `fix:` 등) + 한글 본문을 따른다.

---

## 9. 이 문서의 갱신 규칙

- 사용자가 "규칙 업데이트"를 요청하면, 기존 규칙은 최소한으로 변경한다(불필요한 재작성 금지).
- Phase가 진행되어 실제 파일 구조(엔티티, 컨트롤러, 화면)가 생기면, §3 협조 매트릭스에 그 구체적 파일 경로를 반영하도록 갱신을 고려한다 — 지금은 Phase 1(백엔드)/Phase 7 착수 직전(프론트) 단계라 구체 파일이 거의 없다.
- 이 문서에 일반 개발 지식(BigDecimal 계산법, JWT 문법 등 이미 CLAUDE.md에 있는 설명)을 추가하지 않는다. 발견하면 제거한다.
