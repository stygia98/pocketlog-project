# 포켓로그(PocketLog) 프로젝트 개발 가이드

> **버전** 1.2 · **최종 수정** 2026-09-18
> 이 문서는 **기술 규칙의 단일 기준(Single Source of Truth)**이다.
> 코드 생성 전 반드시 이 문서를 확인하고, 문서와 충돌하는 구현을 하지 않는다.
> 문서에 없는 결정이 필요하면 임의로 진행하지 말고 먼저 질문한다.
>
> 관련 문서: `docs/PRD.md`(무엇을 만드는가) · `docs/ROADMAP.md`(어떤 순서로 만드는가 + **완료 판정의 정본**)
> **이 문서만 루트에 두고, 나머지 문서는 `docs/` 아래에 둔다.** Claude Code가 상위 디렉토리를 거슬러 올라가며 자동 로드하는 대상이 `CLAUDE.md`이기 때문이다.

---

## 1. 프로젝트 개요

- **주제**: 데이터 예측 기반 개인용 스마트 가계부 풀스택 웹 서비스
- **핵심 가치**: 수입·지출을 3초 안에 기록하고, 쌓인 데이터로 **이번 달 지출을 예측**하며, 고정지출을 사용자가 등록하지 않아도 **자동으로 찾아낸다.**
- **범위**: **로컬 개발 완료까지.** 배포 여부와 방식은 미정이며, 결정되면 별도 Phase로 추가한다. **Docker는 사용하지 않는다.**

### 이번 범위에서 명시적으로 제외한 것

**전체 목록은 `docs/PRD.md` 1장 「비목표」가 정본이다.** 여기에는 기술 규칙에 직접 영향을 주는 둘만 적는다.

- **배포** — 미정. 다만 §3의 「배포 여지」 네 항목은 지금 비용이 0이므로 지켜둔다
- **반복 거래 스케줄러** — 「고정지출 자동 감지」가 사용자 입장에서 같은 문제를 푼다. 스케줄러·미래 데이터 생성·소급 수정 처리를 전부 피한다

---

## 2. 저장소 구조 (폴리레포)

**3개의 독립된 Git 저장소**로 관리한다. 모노레포가 아니다.

```
moneylog-project/                # [저장소 1] 문서 저장소
├── .git/
├── .gitignore                   # moneylog-backend/, moneylog-frontend/ 제외
├── CLAUDE.md                    # 기술 규칙 (이 문서). 루트에 둔다
├── docs/
│   ├── PRD.md                   # 제품 요구사항
│   └── ROADMAP.md               # 개발 로드맵 · 완료 판정
│
├── moneylog-backend/            # [저장소 2] 독립 저장소
│   ├── .git/
│   ├── CLAUDE.md                # 백엔드 전용 규칙
│   ├── pom.xml
│   ├── mvnw
│   └── src/
│       ├── main/java/com/example/
│       │   ├── domain/          # 엔티티, Repository
│       │   ├── service/         # 비즈니스 로직, 집계·예측, CSV
│       │   ├── controller/      # REST API
│       │   ├── dto/             # 요청/응답 DTO
│       │   ├── config/          # Security, JWT, Swagger, CORS 설정
│       │   └── exception/       # 예외 처리
│       ├── main/resources/
│       │   ├── application.yml
│       │   ├── application-local.yml
│       │   └── db/              # seed-dev.sql, seed-perf.sql
│       └── test/
│           ├── java/com/example/
│           └── resources/application-test.yml
│
└── moneylog-frontend/           # [저장소 3] 독립 저장소
    ├── .git/
    ├── CLAUDE.md                # 프론트엔드 전용 규칙
    ├── package.json
    ├── public/                  # 정적 파일. public/static 은 만들지 않을 것
    └── src/
        ├── app/
        │   ├── (auth)/
        │   │   ├── login/
        │   │   └── signup/
        │   └── (main)/
        │       ├── dashboard/page.tsx
        │       ├── transactions/
        │       │   ├── page.tsx        # 목록 + 퀵 입력 바
        │       │   └── [id]/page.tsx   # 상세(편집)
        │       ├── budgets/page.tsx    # 예산 설정 + 카테고리 관리(한 화면)
        │       └── data/page.tsx       # CSV 가져오기/내보내기
        ├── components/
        │   ├── ui/              # shadcn/ui
        │   ├── common/          # Pagination, EmptyState, ErrorState, Skeleton
        │   ├── chart/           # CategoryDonut, BudgetBar, MonthHeatmap
        │   ├── illustration/    # Art(SVG 일러스트 세트), paper(종이 배경·질감 공용 스타일)
        │   ├── auth/            # AuthShowcase(로그인·회원가입 일러스트 패널)
        │   └── transaction/     # TransactionList, TransactionRow, QuickAddBar, TransactionForm,
        │                        # TransactionEditDialog(목록 팝업 수정), LocationPickerDialog,
        │                        # TransactionLocationDialog(위치 표시, TXN-14)
        ├── hooks/               # useTransactions, useStats, useAuth, useTheme(다크모드 토글)
        ├── lib/                 # apiClient, queryClient, money, date, utils, kakaoMap(카카오맵 SDK 지연 로드)
        └── types/
```

### ⚠️ 폴리레포 필수 설정

부모 폴더가 Git 저장소이면서 하위 폴더도 Git 저장소이므로, **부모 저장소가 하위 폴더를 추적하지 않도록 반드시 제외해야 한다.** 이 설정을 빠뜨리면 Git이 하위 폴더를 gitlink로 커밋해 버리고, 클론했을 때 빈 폴더만 남는다.

`moneylog-project/.gitignore`:
```gitignore
moneylog-backend/
moneylog-frontend/

node_modules/
.DS_Store
*.log
```

### Git 전략

| 저장소 | 담는 것 |
|---|---|
| moneylog-project | CLAUDE.md, PRD.md, ROADMAP.md |
| moneylog-backend | Spring Boot 애플리케이션 |
| moneylog-frontend | Next.js 애플리케이션 |

- 브랜치: `main`(동작 가능 상태) ← `develop` ← `feature/{작업명}`
- 커밋 메시지: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:` — 본문은 **한글**로 작성한다
- **API 계약이 바뀌면 문서 저장소를 먼저 수정**한 뒤 백엔드 → 프론트엔드 순으로 반영한다.
- **태그 규칙**: Phase 완료 시 해당 저장소에 `v0.{Phase번호}.0`. Phase 12 전체 검증 통과 시 세 저장소 모두 `v1.0.0`.

### 각 저장소의 CLAUDE.md

Claude Code는 현재 디렉토리에서 **상위 디렉토리로 거슬러 올라가며 CLAUDE.md를 찾아 로드**한다. 따라서 `moneylog-backend/`에서 실행해도 부모의 이 문서가 함께 읽힌다.

다만 저장소를 단독으로 클론하면 부모 문서가 없으므로, 각 하위 저장소에도 자체 `CLAUDE.md`를 두고 **해당 저장소에만 해당하는 규칙**(빌드 명령, 계층 규칙, 컨벤션)을 적는다. 전체 스펙은 이 문서를 정본으로 삼는다.

---

## 3. 기술 스택

### Backend
| 항목 | 선택 |
|---|---|
| 프레임워크 | Spring Boot 4.x |
| JDK | 21 |
| 빌드 | Maven (mvnw 래퍼) |
| ORM | Spring Data JPA / Hibernate |
| 보안 | Spring Security + JWT |
| JWT 라이브러리 | **jjwt 0.13.0** — `jjwt-api` + `jjwt-impl`(runtime) + `jjwt-jackson`(runtime) 3종 |
| API 문서 | SpringDoc OpenAPI (Swagger UI) |
| CSV | **라이브러리를 쓰지 않는다** (아래 ⚠️ 참조) |
| DB | PostgreSQL |
| 패키지명 | `com.example` |

### Frontend
| 항목 | 선택 |
|---|---|
| 프레임워크 | **Next.js 15** (App Router) |
| Node.js | **22 이상** (권장 24 LTS) |
| 라이브러리 | React 19, **TypeScript 5.x** (7 금지 — 아래 ⚠️ 참조) |
| 스타일 | Tailwind CSS 4 |
| UI | shadcn/ui, lucide-react |
| 서버 상태 | React Query (TanStack Query v5) |
| 폼 | **라이브러리를 쓰지 않는다** — `useState` + 수동 검증 |
| 애니메이션 | **Motion** (`motion` 패키지, 구 framer-motion) |
| 차트 | **라이브러리를 쓰지 않는다** — 자체 SVG/CSS (아래 ⚠️ 참조) |
| 토스트 | **shadcn/ui `sonner`** |
| 날짜 | **date-fns** (shadcn Calendar 의존) |

### ⚠️ 버전 관련 확정 사항

아래는 확인이 끝난 사항이다. **다시 확인하거나 다른 버전으로 바꾸지 않는다.**

- **Next.js는 15를 쓴다. 16을 쓰지 않는다.** 배포는 미정이지만 AWS Amplify Hosting compute의 SSR 지원 범위가 Next.js 12~15이므로, 16으로 올려두면 나중에 배포를 결정했을 때 되돌리는 작업이 생긴다. App Router·React 19·Tailwind 4는 모두 15에서 정상 동작하므로 이 프로젝트가 잃는 기능은 없다. **지금 15로 두는 비용이 0이므로 배포 여지를 남겨둔다.**
- **Node.js는 22 이상을 쓴다. 권장은 24 LTS다.** Node 20은 **2026-04-30에 EOL**이라 보안 패치가 끊겼다. Next.js 15가 요구하는 최소 버전은 18.18이라 20으로도 동작은 하지만, **EOL 라인을 하한으로 두면 하한이 의미가 없다.** 26은 2026-10-28까지 LTS가 아니므로 쓰지 않는다.
- **TypeScript는 5.x를 쓴다. 7을 쓰지 않는다.** TypeScript 7.0은 Go로 재작성된 네이티브 컴파일러라 빠르지만, **프로그래밍 방식 JS API가 빠져 있다**(7.1에 추가 예정). 그 API에 의존하는 도구가 전부 멈춘다 — `typescript-eslint`가 동작하지 않아 **`npm run lint`가 실패**하고, Next.js 15의 **`next build` 타입체크도 실패**한다(16.3의 `experimental.useTypeScriptCli`로만 우회 가능한데 15에는 그 옵션이 없다).
  > ⚠️ **`create-next-app`이 `typescript: "^5"`로 적어주므로 최초 설치는 안전하다.** 위험한 건 나중에 `npm install typescript@latest`나 의존성 일괄 업데이트를 돌리는 순간이다. **`package.json`의 캐럿 범위를 `^5`에서 넓히지 않는다.**
- **SpringDoc OpenAPI는 3.x를 쓴다.** `org.springdoc:springdoc-openapi-starter-webmvc-ui` 버전 3.x가 Spring Boot 4.x 대응이다. **2.8.x는 Spring Boot 3.x 전용이므로 쓰면 기동에 실패한다.**
  > ⚠️ **"3.x"로 두지 말고 정확한 버전을 `pom.xml`에 핀한다.** SpringDoc 3.x 안에서도 Boot 마이너 버전과 1:1로 대응한다(3.0.0→Boot 4.0.0, 3.0.3→4.0.5, 3.1.0→4.1.0, 3.1.1→4.1.0). 범위로 두면 Boot 4.0.x에 3.1.0이 딸려 들어와 관리 버전이 어긋난다. **`pom.xml`의 Boot 버전을 확인하고 대응하는 SpringDoc 버전을 명시적으로 적는다.**
  > ✅ **2026-09 기준 확정 조합: Spring Boot `4.1.1` + `springdoc-openapi-starter-webmvc-ui` `3.1.1`.** Boot의 `<latest>`는 `4.2.0-M1`이지만 **마일스톤이므로 쓰지 않는다.** 안정판 최신이 4.1.1이다.
- **JWT 라이브러리는 jjwt 0.13.0으로 고정한다.** 세 아티팩트가 모두 필요하며, **`jjwt-impl`과 `jjwt-jackson`은 `<scope>runtime</scope>`이 의도된 설정이다.** 컴파일 시점에는 `jjwt-api`만 참조하고 구현체는 실행 시점에 주입되는 구조이므로, "왜 3개나 있지" 하고 정리하면 기동 시 `ClassNotFoundException`이 난다.
  > ⚠️ **0.11.x → 0.12.x에서 API가 바뀌었다.** 인터넷 예제 다수가 구버전 문법(`Jwts.parser().setSigningKey(...)`, `SignatureAlgorithm.HS256`)인데, **이 API들은 0.13.0에도 deprecated 상태로 남아 있어 컴파일은 통과한다.** 즉 잘못 옮겨도 오류로 걸러지지 않고 경고만 나고 지나간다. 0.12 문법은 `Jwts.parser().verifyWith(key).build()` · `Jwts.builder().signWith(key, Jwts.SIG.HS256)` 형태다.
  > ⚠️ **`signWith`에 알고리즘을 반드시 넘긴다.** 인자 없는 `signWith(key)`를 쓰면 jjwt가 **키 길이로 알고리즘을 추론**한다(32~47B→HS256, 48~63B→HS384, 64B+→HS512). 그러면 §10의 "HS256으로 고정한다"가 코드가 아니라 **`JWT_SECRET`의 길이에 좌우된다.** 알고리즘 상수는 0.11의 `SignatureAlgorithm`이 아니라 **0.12의 `Jwts.SIG`**를 쓴다.
  > ✅ **0.12.x → 0.13.0에서 문법은 바뀌지 않았다.** 변경점은 리플렉션 클래스 로딩 최적화·Gson/BouncyCastle 업그레이드·Maven BOM 추가뿐이라 위 0.12 문법이 그대로 유효하다. GraalVM 네이티브 이미지를 쓸 때만 리플렉션 클래스명 갱신이 필요한데, 이 프로젝트는 해당 없다.
- **Spring Boot 4는 Jackson 3를 쓴다. 직렬화 관련 설정을 넣지 않는다.** Jackson 3의 기본값이 이미 ISO-8601 문자열이므로 §5의 날짜 포맷 요구는 **무설정으로 충족된다.**
  > ⚠️ **컴파일 오류로 걸러지지 않고 조용히 무시된다.** `springdoc`과 `jjwt-jackson`이 **Jackson 2(`com.fasterxml.jackson`)를 compile scope로 함께 끌고 들어오므로** 옛 상수(`SerializationFeature.WRITE_DATES_AS_TIMESTAMPS`)를 참조해도 컴파일은 통과한다. 그러나 Boot 4의 실제 직렬화 엔진은 Jackson 3(`tools.jackson`)라 그 설정이 **아무 효과도 내지 못한다.** 애초에 손대지 않는 것이 유일한 방어다.
- **애니메이션 패키지는 `motion`이다.** `framer-motion`은 이름이 바뀌기 전의 deprecated 별칭이다. `npm install motion`으로 설치하고 **import는 반드시 `motion/react`에서 한다.**
- **shadcn/ui는 React 19 + Tailwind 4를 정식 지원한다.** 단 npm으로 설치할 때 peer dependency 충돌이 나므로 **`--legacy-peer-deps` 플래그를 쓴다.** toast 컴포넌트는 deprecated이므로 **sonner**를 쓰고, 신규 프로젝트 스타일은 **new-york**을 쓴다.
- **폼 라이브러리를 도입하지 않는다.** `react-hook-form`·`zod`·`@hookform/resolvers`를 설치하지 않는다. 이 앱의 폼은 검증 규칙이 §4 제약 표로 고정되어 있어 `useState` + 수동 검증으로 충분하다.
  > ⚠️ **shadcn/ui의 `form` 컴포넌트를 추가하지 않는다.** 이 컴포넌트만 `react-hook-form` 위에 만들어져 있어, `npx shadcn add form`을 실행하면 `react-hook-form`과 `@hookform/resolvers`가 **의존성으로 함께 설치된다.** 다른 shadcn 컴포넌트(`input`, `label`, `button`, `select`, `checkbox`, `calendar`, `tabs`, `dialog` 등)는 영향이 없다.
  > ⚠️ **대신 `dirty` 판정을 직접 구현해야 한다.** `TXN-09`(이탈 확인)가 이를 요구하므로 초기값과 현재값을 직접 비교한다. **금액 필드가 특히 까다롭다** — 표시용으로 천단위 콤마를 넣으므로(§8) 비교는 반드시 **콤마를 제거한 정규화 값끼리** 한다.
- **Tailwind CSS 4는 CSS-first 설정**이다. `tailwind.config.js` 대신 `globals.css`에서 `@import "tailwindcss";` + `@theme { ... }`로 토큰을 정의한다. v3 방식으로 작성하지 않는다.
- Spring Security는 `SecurityFilterChain` 빈(람다 DSL)으로만 설정한다. `WebSecurityConfigurerAdapter`는 사용하지 않는다.

### ⚠️ 차트 라이브러리를 쓰지 않는다 — 단, Recharts로 갈아끼울 수 있게 만든다 (중요)

이 앱의 차트는 넷뿐이고 셋은 라이브러리가 오히려 방해가 된다.

| 차트 | 구현 방식 | 라이브러리가 불필요한 이유 |
|---|---|---|
| 카테고리별 지출 막대 | `div` 너비를 `%`로 | 축·툴팁·범례가 전부 불필요 |
| 예산 소진율 막대 | 〃 | 〃 |
| 일별 캘린더 히트맵 | CSS `grid-cols-7` + 배경색 단계 | Recharts에 애초에 이런 차트가 없다 |
| 카테고리 도넛 | SVG `circle` + `stroke-dasharray` | 도넛 한 개에 100KB를 넣을 이유가 없다 |
| (같은 자리에서 탭으로 전환) 카테고리 막대 · 트리맵 | `div` 너비 % · 절대 위치 `div` | 세 차트가 같은 `ChartDatum` props를 받는다. 트리맵 배치는 `chart/` 안에서만 계산한다 |

> ⚠️ **「월별 추이 선」은 이번 범위에 없다.** `PRD.md` 5.4 대시보드가 요구하는 차트는 **위 넷뿐**이므로(표 위 문장의 "넷"이 이 넷이다) `TrendLine` 컴포넌트를 만들지 않는다. 「연간 리포트·전년 동월 비교」(`PRD.md` 9장)가 범위에 들어오면 그때 SVG `polyline`으로 추가한다 — 데이터 포인트가 6~12개뿐이라 그때도 라이브러리는 필요 없다.

**대신 나중에 Recharts로 교체할 수 있도록 다음을 지킨다.** 이건 추상화 계층을 만들라는 뜻이 아니라, **props 형태를 Recharts가 그대로 받는 모양으로 맞춰두라**는 뜻이다.

```ts
// src/components/chart/CategoryDonut.tsx
// props는 Recharts의 <Pie data={...} dataKey="value" nameKey="name" /> 와 같은 모양으로 둔다.
type ChartDatum = { name: string; value: number; color?: string };
export function CategoryDonut({ data }: { data: ChartDatum[] }) { /* SVG 직접 구현 */ }
```

- **차트 컴포넌트는 `src/components/chart/` 밖으로 나가지 않는다.** 화면은 `<CategoryDonut data={...} />`만 알고, SVG인지 Recharts인지 모른다.
- **화면에서 SVG를 직접 그리지 않는다.** 한 곳이라도 새면 교체 비용이 그만큼 늘어난다.
- 교체 조건: 축 레이블·툴팁·줌·브러시 중 **둘 이상이 필요해지면** 그때 `npm install recharts`하고 `chart/` 안의 구현만 바꾼다. 화면 코드는 건드리지 않는다.
- **색상은 차트 컴포넌트가 정하지 않는다.** 카테고리 색은 DB에 저장된 값(§4)을 그대로 쓰고, 없으면 §8의 팔레트를 순서대로 배정한다.

### ⚠️ CSV 라이브러리를 쓰지 않는다

`opencsv`·`commons-csv`를 설치하지 않는다. 내보내기는 문자열 조합, 가져오기는 **한 줄 파서**로 끝난다. 다만 **직접 `split(",")`을 쓰지 않는다** — 메모에 콤마가 들어가면 열이 밀린다. 따옴표 안의 콤마를 처리하는 파서를 `service/CsvParser.java`에 20줄 내외로 구현하고 단위 테스트로 고정한다. 규칙은 RFC 4180 최소 집합(따옴표로 감싸기, 내부 따옴표는 `""`로 이스케이프)만 지원한다.

### ⚠️ 배포 여지 (지금 비용 0이므로 지켜둔다)

배포는 미정이지만, 아래 넷은 지키는 데 비용이 들지 않고 어기면 나중에 되돌리는 작업이 생긴다.

- **`public/static` 경로를 만들지 않는다.** Amplify가 배포용으로 예약한 경로다. 정적 파일은 `public/` 바로 아래나 `public/assets/`에 둔다.
- `next.config.ts`에 `distDir`을 설정하지 않는다. 빌드 출력은 `.next` 그대로 둔다.
- `next/image`를 도입하지 않는다. 이 앱에 원격 이미지가 없다.
- **시크릿을 코드에 하드코딩하지 않는다.** 로컬에서도 `.env`/환경변수로 분리한다(§10).

---

## 4. 데이터 모델

DB 스키마명: **`moneylog_db`** (소문자) · 테스트: **`moneylog_test`**

### users
| 컬럼 | 타입 | 제약 |
|---|---|---|
| id | BIGSERIAL | PK |
| email | VARCHAR(255) | UNIQUE, NOT NULL (로그인 ID) |
| password | VARCHAR(255) | NOT NULL |
| nickname | VARCHAR(50) | NOT NULL |
| created_at | TIMESTAMP | NOT NULL |
| updated_at | TIMESTAMP | NOT NULL |
| deleted_at | TIMESTAMP | NULL |

> **`users.deleted_at`은 이번 범위에서 항상 NULL이다.** 회원 탈퇴가 비목표이므로 값을 채우는 경로가 없다. 다만 스키마와 조회 조건은 유지해, 향후 탈퇴 기능 추가 시 구조를 바꾸지 않는다. 로그인·인증 시 `deleted_at IS NULL` 검사는 걸어둔다.
>
> **`provider`·`provider_id` 컬럼을 지금 만들지 않는다.** 구글 로그인이 범위 밖이므로 항상 `LOCAL`/`NULL`인 컬럼이 된다. 나중에 OAuth를 추가할 때 컬럼 2개를 더하는 마이그레이션은 사소하다.

### categories
| 컬럼 | 타입 | 제약 |
|---|---|---|
| id | BIGSERIAL | PK |
| user_id | BIGINT | FK → users.id, NOT NULL |
| name | VARCHAR(30) | NOT NULL |
| type | VARCHAR(10) | NOT NULL, `INCOME` / `EXPENSE` |
| color | CHAR(7) | NOT NULL, `#RRGGBB` |
| sort_order | INT | NOT NULL, DEFAULT 0 |
| created_at | TIMESTAMP | NOT NULL |
| updated_at | TIMESTAMP | NOT NULL |
| deleted_at | TIMESTAMP | NULL |

**인덱스**: `idx_categories_user_deleted` on `(user_id, deleted_at)`
**제약**: `(user_id, name, type)` 부분 유니크 — `WHERE deleted_at IS NULL`

> ⚠️ **부분 유니크 인덱스가 필요한 이유** — 일반 UNIQUE를 걸면 "식비"를 삭제한 뒤 다시 "식비"를 만들 수 없다. Soft Delete와 UNIQUE는 항상 이 충돌을 일으킨다. JPA의 `@Table(uniqueConstraints=...)`로는 부분 인덱스를 만들 수 없으므로 **DDL에 직접 적는다.**
> ```sql
> CREATE UNIQUE INDEX uq_categories_user_name_type
>   ON categories (user_id, name, type) WHERE deleted_at IS NULL;
> ```

### transactions
| 컬럼 | 타입 | 제약 |
|---|---|---|
| id | BIGSERIAL | PK |
| user_id | BIGINT | FK → users.id, NOT NULL |
| category_id | BIGINT | FK → categories.id, NOT NULL |
| type | VARCHAR(10) | NOT NULL, `INCOME` / `EXPENSE` |
| amount | **NUMERIC(15,2)** | NOT NULL, **> 0** |
| txn_date | DATE | NOT NULL |
| merchant | VARCHAR(100) | NULL (거래처/상호) |
| memo | VARCHAR(500) | NULL (**평문. HTML 아님**) |
| latitude | DOUBLE | NULL (위치, `TXN-14`) |
| longitude | DOUBLE | NULL (위치, `TXN-14`) |
| created_at | TIMESTAMP | NOT NULL |
| updated_at | TIMESTAMP | NOT NULL |
| deleted_at | TIMESTAMP | NULL (Soft Delete) |

> **`latitude`/`longitude`는 사용자가 프론트의 카카오맵 검색 팝업에서 지점을 고를 때만 채워진다.** 자유 텍스트인 `merchant`를 서버가 지오코딩하지 않는다 — 동명 지점이 여러 곳이라 부정확하다. 값의 출처가 사용자의 자유 입력이 아니므로 DTO에 범위 검증(`@DecimalMin`/`@DecimalMax`)을 두지 않는다.

**인덱스**
- `idx_txn_user_date` on `(user_id, txn_date DESC, deleted_at)` — 목록·월별 집계의 주 경로
- `idx_txn_user_category` on `(user_id, category_id, deleted_at)` — 카테고리별 집계

### budgets
| 컬럼 | 타입 | 제약 |
|---|---|---|
| id | BIGSERIAL | PK |
| user_id | BIGINT | FK → users.id, NOT NULL |
| category_id | BIGINT | FK → categories.id, NOT NULL |
| year_month | CHAR(7) | NOT NULL, `yyyy-MM` |
| amount | NUMERIC(15,2) | NOT NULL, **> 0** |
| created_at | TIMESTAMP | NOT NULL |
| updated_at | TIMESTAMP | NOT NULL |

**제약**: UNIQUE `(user_id, category_id, year_month)`

> **`budgets`에는 `deleted_at`이 없다.** 예산은 "지운다"가 아니라 "0으로 만든다"가 아니라 **행 자체를 제거**하는 것이 자연스럽다(예산 미설정 상태로 돌아감). 물리 삭제를 허용하는 이 프로젝트의 유일한 테이블이며, 집계에서 과거 이력을 참조하지 않으므로 안전하다.
>
> **`year_month`를 `CHAR(7)` 문자열로 두는 이유** — `yyyy-MM` 형식은 문자열 정렬이 곧 시간 정렬이라 범위 조회(`BETWEEN '2026-04' AND '2026-09'`)가 그대로 동작한다. 별도 year·month 정수 컬럼 두 개보다 쿼리가 단순하다.

### ⚠️ 금액은 `NUMERIC` / `BigDecimal`이다. `double`·`float`를 쓰지 않는다 (중요)

부동소수점은 10진 소수를 정확히 표현하지 못해 합계가 어긋난다. 가계부에서 이건 제품 결함이다.

- DB: `NUMERIC(15,2)` · Java: `java.math.BigDecimal` · TypeScript: `number`(응답은 JSON 숫자)
- **`BIGINT`(원 단위 정수)를 쓰지 않는 이유** — 이 앱은 평균·예측·소진율 계산이 본체라 나눗셈이 도처에 있다. `long` 나눗셈은 **조용히 절삭**되어 "월 평균 3건에 10,000원"이 3,333원으로 계산되고 합계가 1원씩 맞지 않는다.

#### ⚠️ `BigDecimal.equals`는 scale까지 비교한다

```java
new BigDecimal("1000").equals(new BigDecimal("1000.00"))   // false!
new BigDecimal("1000").compareTo(new BigDecimal("1000.00")) // 0  ← 이걸 쓴다
```

DB에서 읽은 값은 scale 2(`1000.00`), 코드에서 만든 값은 scale 0(`1000`)이라 **테스트에서 반드시 마주친다.** 금액 비교는 예외 없이 `compareTo(...) == 0`으로 한다. JUnit이라면 `assertThat(a).isEqualByComparingTo(b)`를 쓴다.

#### ⚠️ `BigDecimal.divide`는 scale을 넘기지 않으면 예외를 던진다

```java
// 무한소수가 나오면 ArithmeticException: Non-terminating decimal expansion
total.divide(count);
// 항상 scale과 RoundingMode를 함께 넘긴다
total.divide(new BigDecimal(count), 2, RoundingMode.HALF_UP);
```

평균 계산(`총액 ÷ 개월수`)은 3으로 나누는 순간 터진다. **이 앱의 예측 로직은 전부 나눗셈이므로 예외 없이 적용한다.**

#### ⚠️ `SUM()`은 대상 행이 없으면 `0`이 아니라 `NULL`이다

```sql
-- 거래가 하나도 없는 달을 조회하면 NULL이 돌아와 Java에서 NPE가 난다
SELECT COALESCE(SUM(amount), 0) FROM transactions WHERE ...
```

신규 가입 직후 대시보드가 이 경로를 그대로 탄다. **모든 집계 쿼리에 `COALESCE`를 건다.**

### ⚠️ 금액의 부호는 `type`으로만 표현한다. 음수 금액을 허용하지 않는다

`amount`에 음수를 허용하면 "지출 -5000"이 환불인지 입력 실수인지 알 수 없고, 집계가 이중 의미를 갖는다. **`amount > 0` 제약을 DB와 DTO 양쪽에 건다.** 환불은 반대 `type`의 거래로 기록한다.

### ⚠️ 거래의 `type`은 카테고리의 `type`과 일치해야 한다

지출 카테고리에 수입 거래가 들어가면 카테고리별 집계가 조용히 깨진다. 스키마로는 막을 수 없으므로 **`TransactionService`에서 검증**하고, 불일치 시 400 `CATEGORY_TYPE_MISMATCH`로 응답한다.

### ⚠️ 삭제된 카테고리의 과거 거래는 그대로 보여야 한다 (중요)

카테고리를 Soft Delete하면 그 카테고리를 쓰던 과거 거래가 남는다. 여기서 조회 조건을 일률적으로 `deleted_at IS NULL`로 걸면 **작년 내역이 전부 "미분류"로 보이거나 조인에서 탈락해 사라진다.**

| 대상 | `deleted_at IS NULL` 조건 |
|---|---|
| 카테고리 **목록** 조회 (선택 UI용) | **건다** — 삭제한 카테고리를 새 거래에 고를 수 없어야 한다 |
| 거래 조회 시 **카테고리 조인** | **걸지 않는다** — 과거 내역의 이름·색을 그대로 보여준다 |
| 집계 시 카테고리 그룹핑 | **걸지 않는다** — 과거 달의 합계가 바뀌면 안 된다 |

- 삭제된 카테고리가 붙은 거래는 화면에서 이름 옆에 "(삭제됨)"을 표시한다.
- **거래가 하나라도 있는 카테고리는 물리 삭제하지 않는다.** Soft Delete가 이 요구의 이유다.

### 입력값 제약 (DTO 검증과 스키마를 일치시킬 것)

| 필드 | 제약 | 이유 |
|---|---|---|
| `email` | 형식 검증, 최대 255자 | |
| `password` | **6자 이상, 그리고 UTF-8 인코딩 시 72바이트 이하** | 아래 ⚠️ 참조 |
| `nickname` | 1~50자 | |
| `category.name` | 1~30자, 필수 | 스키마 VARCHAR(30)과 일치 |
| `category.color` | `#RRGGBB` 정규식 | 임의 문자열이 인라인 스타일로 들어가지 않게 |
| `amount` | **필수, 0 초과, 최대 9,999,999,999,999.99** | `NUMERIC(15,2)` 범위 |
| `txnDate` | `yyyy-MM-dd`, 필수 | |
| `merchant` | 최대 100자, 선택 | |
| `memo` | 최대 500자, 선택 | |
| `yearMonth` | `yyyy-MM` 정규식, 필수 | 파싱 실패로 500이 나지 않게 |

#### ⚠️ 비밀번호 상한은 "문자 수"가 아니라 "바이트"다 (중요)

**`@Size(max=64)`만 걸면 한글 비밀번호에서 500이 난다.**

BCrypt의 한계는 **72바이트**다. UTF-8에서 한글 1자는 3바이트이므로 **한글 25자 = 75바이트**로 이미 한계를 넘는다. 그런데 `@Size(max=64)`는 문자 수를 세므로 이 입력을 **통과시킨다.**

그리고 최신 Spring Security의 `BCryptPasswordEncoder`는 72바이트 초과분을 **조용히 버리지 않고** `IllegalArgumentException("password cannot be more than 72 bytes")`를 **던진다**(CVE-2025-22228 대응). 즉 검증을 통과한 요청이 인코딩 단계에서 터지고, `GlobalExceptionHandler`에 매핑이 없으면 **500 `INTERNAL_ERROR`**로 나간다.

```java
// 최소 길이는 문자 수, 최대 길이는 바이트로 검증한다
@Size(min = 6, message = "비밀번호는 6자 이상이어야 합니다.")
@MaxByteLength(value = 72, message = "비밀번호가 너무 깁니다. (한글은 1자가 3바이트로 계산됩니다)")
private String password;
```

- 위반 시 **400 `INVALID_INPUT`** + 필드 메시지로 응답한다. 500이 나가면 안 된다.
- 안전장치로 `GlobalExceptionHandler`에 `IllegalArgumentException` → 400 매핑도 함께 걸어둔다.

### 공통 규칙
- 모든 엔티티는 `BaseEntity`를 상속해 `created_at`, `updated_at`을 자동 관리한다 (`@EnableJpaAuditing`).
- **Soft Delete**: `categories`·`transactions`는 물리 삭제 금지. `deleted_at`에 현재 시각을 기록한다. (`budgets`는 예외 — 위 참조)
- 로컬 개발은 `ddl-auto: update`. 단 **부분 유니크 인덱스는 Hibernate가 만들지 못하므로** `db/schema-extra.sql`에 적어두고 최초 1회 수동 적용한다.

### ⚠️ 타임존은 UTC로 고정한다

- `application.yml`에 `spring.jpa.properties.hibernate.jdbc.time_zone: UTC`를 설정한다.
- `created_at`·`updated_at`·`deleted_at`만 해당된다. **`txn_date`는 `LocalDate`(시각 없음)라 타임존 영향을 받지 않는다.**
- 표시할 때만 브라우저 로컬 시각으로 변환한다(프론트 `date-fns`).

### ⚠️ "이번 달"과 "오늘"을 서버가 판정하지 않는다 (중요)

서버는 UTC로 돌고 사용자는 KST(+09:00)다. 서버에서 `LocalDate.now()`나 `YearMonth.now()`를 부르면 **매월 1일 0시~9시 사이에 사용자는 지난달 대시보드를 보게 된다.** 같은 이유로 매일 0시~9시 사이에는 "오늘 지출"이 어제 것으로 집계된다.

`txn_date`가 타임존에 영향받지 않는 `LocalDate`라는 사실이 이 문제를 **가리기까지 한다** — 데이터는 멀쩡한데 조회 범위만 하루 어긋나므로 원인을 찾기 어렵다.

**해결: 기준 날짜를 클라이언트가 파라미터로 보낸다.**

| 파라미터 | 형식 | 의미 |
|---|---|---|
| `yearMonth` | `yyyy-MM` | 조회 대상 월. 필수 |
| `asOf` | `yyyy-MM-dd` | 사용자의 "오늘". 런레이트의 남은 일수 계산에 쓴다. 필수 |

- **서버의 집계·예측 코드에 `now()` 계열 호출이 등장하면 안 된다.** (`created_at` 자동 관리는 예외)
- 사용자가 자기 `asOf`를 조작해도 손해는 자기 데이터 예측값뿐이므로 신뢰해도 안전하다.
- 프론트는 `new Date()`로 만든 로컬 날짜를 `date-fns`의 `format(d, "yyyy-MM-dd")`로 보낸다. **`toISOString()`을 쓰지 않는다** — UTC로 변환되어 같은 문제가 프론트에서 재현된다.

### ⚠️ 연관관계는 반드시 LAZY로 지정한다 — 단 거래 목록은 fetch join이 필요하다

`@ManyToOne`의 기본값은 **EAGER**다.

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "category_id", nullable = false)
private Category category;
```

> ⚠️ **다만 `TransactionResponse`에는 카테고리 이름과 색이 들어간다.** 목록 화면이 그걸 그리기 때문이다. LAZY로 두기만 하면 목록 20건에 카테고리 조회 20번이 추가로 나간다(N+1). **거래 목록 쿼리는 `join fetch t.category`로 함께 가져온다.**
>
> 이 점이 사용자 정보를 응답에 넣지 않는 일반적인 패턴과 다르다. 카테고리는 화면에 필요하므로 **빼는 게 아니라 fetch join으로 해결한다.**

- 소유권 검증은 `transaction.getUser().getId()`로 한다. 프록시 상태에서 id만 읽으면 추가 쿼리가 발생하지 않는다.
- `TransactionResponse`에 **사용자 정보는 넣지 않는다.** 본인 데이터만 조회하므로 불필요하다.

### 검색 쿼리 주의

`LOWER(merchant) LIKE '%키워드%'`는 앞쪽 와일드카드 때문에 인덱스를 타지 못한다. MVP 데이터 규모에서는 문제없지만, **`merchant`에 인덱스를 추가해도 검색은 빨라지지 않는다는 점을 알고 있어야 한다.** 규모가 커지면 전문 검색(`pg_trgm`)을 검토한다.

---

## 5. API 명세

Base path: `/api/v1`

### 공통 응답 포맷 (아래 한 곳을 제외한 모든 REST 응답에 적용)

```json
// 성공
{ "success": true, "data": { ... }, "error": null }

// 실패
{ "success": false, "data": null,
  "error": { "code": "TRANSACTION_NOT_FOUND", "message": "거래 내역을 찾을 수 없습니다." } }
```

> **목록 API도 이 포맷을 따른다.** `PageResponse`는 최상위가 아니라 **`data` 안에** 들어간다. 프론트는 `res.data.data.content`로 접근한다.
>
> **유일한 예외는 `GET /api/v1/data/export`다.** CSV 바이트를 직접 반환하므로 봉투를 쓰지 않는다.

### 인증
| Method | Endpoint | 설명 | 인증 |
|---|---|---|---|
| POST | `/api/v1/auth/signup` | 회원가입 (email, password, nickname) | X |
| POST | `/api/v1/auth/login` | 로그인 → JWT 반환 | X |
| GET | `/api/v1/auth/me` | 내 정보 조회 | O |
| PUT | `/api/v1/auth/me` | 내 닉네임 변경 (`{ nickname }`, 1~50자) → 갱신된 내 정보(`UserResponse`) 반환 | O |

> **로그아웃은 서버 API를 만들지 않는다.** Refresh Token과 토큰 블랙리스트가 없으므로, 프론트에서 localStorage의 토큰을 제거하고 React Query 캐시를 비운 뒤 `/login`으로 이동하는 것으로 처리한다.
>
> **회원가입 시 기본 카테고리를 함께 생성한다.** 빈 카테고리 상태에서는 거래를 한 건도 넣을 수 없어 첫 화면이 막힌다. §6 참조.

### 카테고리
| Method | Endpoint | 설명 |
|---|---|---|
| GET | `/api/v1/categories` | 목록 (`?type=INCOME\|EXPENSE` 선택) |
| POST | `/api/v1/categories` | 생성 |
| PUT | `/api/v1/categories/{id}` | 수정 (name, color, sortOrder) |
| DELETE | `/api/v1/categories/{id}` | Soft Delete |

- **`type`은 생성 후 변경할 수 없다.** 수입 카테고리를 지출로 바꾸면 이미 쌓인 거래의 집계가 뒤집힌다. `PUT`의 요청 DTO에 `type`을 넣지 않는다.
- 목록은 `sortOrder ASC, id ASC` 고정. 페이지네이션 없음(개수가 수십 개를 넘지 않는다).

### 거래
| Method | Endpoint | 설명 | 요청 바디 |
|---|---|---|---|
| GET | `/api/v1/transactions` | 목록 (페이지네이션) | — |
| POST | `/api/v1/transactions` | 생성 | `TransactionCreateRequest` |
| GET | `/api/v1/transactions/{id}` | 단건 조회 | — |
| PUT | `/api/v1/transactions/{id}` | 수정 (**전체 교체**) | `TransactionUpdateRequest` |
| DELETE | `/api/v1/transactions/{id}` | Soft Delete | — |

PUT은 부분 수정이 아니라 **전체 교체**다. `merchant`·`memo`·`latitude`·`longitude`를 누락하면 null로 저장된다(값 삭제로 취급). `amount`·`txnDate`·`categoryId`·`type`은 필수이므로 누락 시 400이다.

- `TransactionCreateRequest`/`TransactionUpdateRequest`에 `latitude`·`longitude`(선택, `Double`)를 둔다 — `TXN-14`. 프론트가 위치를 지우면 두 필드를 명시적으로 `undefined`로 보내 PUT 전체 교체 규칙대로 null이 저장되게 한다.
- `TransactionResponse`에도 `latitude`·`longitude`를 그대로 내려준다.

#### 목록 쿼리 파라미터

| 파라미터 | 기본값 | 동작 |
|---|---|---|
| `page` | 0 | 0부터 시작 |
| `size` | 20 | |
| `sort` | `txnDate,desc` | **MVP에서는 고정.** API는 받되 정렬 선택 UI는 만들지 않는다. **허용 필드는 `txnDate`, `amount`, `createdAt`뿐이며, 그 외 값이 들어오면 기본값으로 대체한다.** Pageable에 임의 문자열을 그대로 넘기면 없는 프로퍼티에서 500이 난다 |
| `from` / `to` | 미지정 | `yyyy-MM-dd`. 기간 필터. 한쪽만 줘도 동작 |
| `type` | 미지정 | 미지정 시 전체. `INCOME`/`EXPENSE` |
| `categoryId` | 미지정 | |
| `keyword` | 미지정 | **`merchant` + `memo`** 부분 일치, **대소문자 무시** |

> ⚠️ **`txnDate`가 같은 거래가 흔하다**(같은 날 여러 건). 정렬이 `txnDate` 하나뿐이면 순서가 매 요청마다 달라져 **페이지네이션에서 항목이 중복되거나 누락된다.** 정렬에 **`id DESC`를 항상 2차 키로 덧붙인다.** 이건 Todo 앱처럼 `createdAt`으로 정렬할 때는 드러나지 않던 문제다.

#### 목록 응답 예시

```json
{
  "success": true,
  "data": {
    "content": [
      { "id": 1, "type": "EXPENSE", "amount": 12500.00, "txnDate": "2026-09-14",
        "merchant": "스타벅스 강남점", "memo": null,
        "category": { "id": 3, "name": "식비", "color": "#EF4444", "deleted": false } }
    ],
    "page": 0, "size": 20, "totalElements": 142, "totalPages": 8,
    "first": true, "last": false
  },
  "error": null
}
```

Spring의 `Page` 객체를 그대로 반환하지 않고 `PageResponse<T>` DTO로 변환한 뒤 `ApiResponse.data`에 담는다.

### 집계 · 예측

| Method | Endpoint | 설명 |
|---|---|---|
| GET | `/api/v1/stats/monthly?yearMonth=&asOf=` | 월 대시보드 **일괄 조회** |
| GET | `/api/v1/stats/recurring?asOf=` | 고정지출 자동 감지 |

#### ⚠️ 대시보드 집계는 엔드포인트 하나로 묶는다

요약·카테고리별·일별·런레이트·이상치는 **같은 화면이 동시에 필요로 하고 같은 원본(해당 월의 거래)에서 나온다.** 다섯 개로 쪼개면 요청 5번에 같은 테이블을 5번 스캔하고, 각각 따로 만료되어 **화면 안에서 숫자가 서로 어긋나는 순간**이 생긴다.

```json
{
  "success": true,
  "data": {
    "yearMonth": "2026-09",
    "summary": { "income": 3200000.00, "expense": 1842300.00, "net": 1357700.00 },
    "byCategory": [
      { "categoryId": 3, "name": "식비", "color": "#EF4444", "deleted": false,
        "amount": 412000.00, "ratio": 0.224 }
    ],
    "daily": [ { "date": "2026-09-01", "expense": 32000.00, "income": 0.00 } ],
    "forecast": {
      "confirmedExpense": 1842300.00,
      "projectedExpense": 2610000.00,
      "baselineDailyAvg": 85300.00,
      "daysElapsed": 15, "daysInMonth": 30,
      "basisMonths": 3
    },
    "anomalies": [
      { "categoryId": 3, "name": "식비", "currentPace": 824000.00,
        "baseline": 615000.00, "deltaRatio": 0.34 }
    ],
    "budgets": [
      { "categoryId": 3, "name": "식비", "budget": 600000.00,
        "spent": 412000.00, "usageRatio": 0.687, "exceeded": false }
    ]
  },
  "error": null
}
```

> ⚠️ **`daily`는 그 달의 모든 날을 빠짐없이 담는다.** 거래가 없는 날도 `"expense": 0.00, "income": 0.00`으로 채워 반환한다. 배열 길이는 항상 그 달의 일수(28~31)와 같고, `date`는 1일부터 말일까지 오름차순이다.
>
> **프론트가 채우게 두지 않는 이유.** ① 히트맵(§7 `/dashboard`)은 달력 격자라 어차피 모든 날의 칸을 그린다. 서버가 빠뜨리면 **프론트가 빈 날을 채우는 코드를 갖게 되고**, 그것이 §13 "예측 수식은 백엔드에만 구현한다"의 첫 구멍이 된다. ② 프론트가 채우려면 그 달의 일수를 알아야 하는데, 윤년·월말 계산을 양쪽에서 하게 된다. ③ 최대 31개 항목이라 응답 크기에 영향이 없다.
>
> ⚠️ **이건 `COALESCE`로 해결되지 않는다.** `GROUP BY txn_date`는 **행이 있는 날만** 반환하므로, 거래가 없는 날은 `SUM`이 `NULL`이 되는 게 아니라 **그룹 자체가 존재하지 않는다.** §4의 `COALESCE` 규칙과 다른 문제이며, 서비스에서 1일~말일을 순회해 빈 날을 채우거나 쿼리에서 `generate_series`로 날짜를 만들어 LEFT JOIN해야 한다.

`recurring`을 분리하는 이유는 성질이 다르기 때문이다 — **최근 3개월 전체를 스캔**하므로 비용이 크고, 결과가 월 단위로만 바뀌어 캐시 수명이 길다. 대시보드 첫 페인트를 지연시키지 않도록 별도 요청으로 둔다.

#### 예측 계산 규칙 (정본)

이 절의 수식이 백엔드 구현과 화면 문구의 기준이다. **화면에서 다시 계산하지 않는다.**

**1) 기준선(baseline)** — 직전 `N`개월(**N=3**, 당월 제외)의 지출 평균.
```
baselineDailyAvg = (직전 3개월 지출 합계) / (그 3개월의 실제 총 일수)
```
- 일수로 나눈다. 개월 수로 나누면 28일인 2월과 31일인 1월이 같은 가중치를 받는다.
- **데이터가 부족하면 예측하지 않는다.** 직전 3개월에 거래가 **한 건도 없으면** `forecast`를 `null`로 반환하고, 화면은 "예측하려면 데이터가 조금 더 필요해요"를 보여준다. 1~2개월치만 있으면 있는 만큼으로 계산하고 `basisMonths`에 실제 사용한 개월 수를 담아 화면이 "최근 1개월 기준"이라고 밝힐 수 있게 한다.

**2) 이번 달 예상 지출(런레이트)**
```
daysElapsed      = asOf의 일(day). 단 asOf가 대상 월 밖이면 그 달의 총 일수로 간주
projectedExpense = confirmedExpense + baselineDailyAvg × (daysInMonth - daysElapsed)
```
- **과거 달을 조회하면 예측하지 않는다.** 이미 끝난 달의 "예상"은 의미가 없다. `asOf`가 대상 월 이후면 `forecast.projectedExpense`는 `confirmedExpense`와 같아진다.

**3) 이상치(anomalies)** — 카테고리별로 이번 달 **속도**를 기준선과 비교한다.
```
currentPace = (이번 달 해당 카테고리 지출) / daysElapsed × daysInMonth
deltaRatio  = (currentPace - baseline) / baseline
```
- `|deltaRatio| >= 0.30`이고 `baseline > 0`일 때만 목록에 담는다. 임계값을 낮추면 매달 모든 카테고리가 "이상"이 되어 알림이 무의미해진다.
- **월초에는 노이즈가 크다.** `daysElapsed < 7`이면 이상치를 계산하지 않고 빈 배열을 반환한다. 1일에 외식 한 번 하면 식비가 3000% 증가로 나온다.

**4) 고정지출 감지(recurring)** — 최근 3개월 스캔.
```
정규화상호 = LOWER(TRIM(merchant))에서 공백·괄호·숫자 제거
조건 = 정규화상호가 최근 3개월 각각에 1건 이상 존재
     AND 각 건의 금액이 (해당 상호 금액 중앙값 ± 10%) 이내
```
- `merchant`가 비어 있는 거래는 대상에서 제외한다.
- 결과에는 `merchant`, `categoryId`, `medianAmount`, `monthsSeen`, `lastDate`를 담는다.
- **감지 결과를 자동으로 저장하지 않는다.** 화면에 "이거 고정지출로 보여요"만 표시한다. DB에 쓰기 시작하면 사용자가 지운 항목이 다음 달에 되살아나는 문제를 처리해야 하고, 그 순간 반복 거래 기능을 만드는 것과 같아진다.

### 예산
| Method | Endpoint | 설명 |
|---|---|---|
| GET | `/api/v1/budgets?yearMonth=` | 해당 월 예산 목록 (지출 카테고리 전체 + 설정된 금액) |
| PUT | `/api/v1/budgets` | **upsert**. `{ yearMonth, items: [{ categoryId, amount }] }` |

- **`POST`와 `DELETE`를 두지 않는다.** 예산 화면은 "카테고리 목록에 금액을 채워 한 번에 저장"하는 형태라 개별 생성/삭제 경로가 필요 없다. `amount`가 `0` 또는 `null`인 항목은 해당 행을 제거한다.
- 소진율은 이 API가 아니라 `stats/monthly`의 `budgets` 배열에서 내려준다(같은 화면에서 지출과 함께 보이므로).
- **소진율 계산 시 `budget == 0` 분기를 반드시 둔다.** 0으로 나누면 `Infinity`가 JSON에 실려 프론트에서 `NaN%`로 표시된다.

### 데이터 (CSV)
| Method | Endpoint | 설명 | 비고 |
|---|---|---|---|
| GET | `/api/v1/data/export?from=&to=` | 거래 내역 CSV 다운로드 | **`ApiResponse` 봉투 예외** |
| POST | `/api/v1/data/import` | CSV 업로드 (`multipart/form-data`) | 검증 결과를 봉투에 담아 반환 |

CSV 형식(헤더 고정, 내보내기·가져오기 동일):
```csv
날짜,구분,카테고리,금액,거래처,메모
2026-09-14,지출,식비,12500,스타벅스 강남점,팀 미팅
```

- **구분**은 `수입`/`지출` 한글로 쓴다. 사용자가 엑셀에서 직접 편집하는 파일이다.
- **카테고리는 이름으로 매칭한다.** 없는 이름이면 그 행을 실패로 처리하고 **자동 생성하지 않는다.** 오타 하나로 카테고리가 증식하는 것을 막는다.
- 가져오기 응답: `{ imported: 42, failed: 3, errors: [{ line: 7, reason: "카테고리 '식대'를 찾을 수 없습니다." }] }`
- **부분 성공을 허용한다.** 전부 롤백하면 100행 중 1행 오타로 99행을 다시 올려야 한다. 실패 행만 건너뛰고 결과를 알린다.
- 업로드 상한: **1MB, 5,000행.** 초과 시 400. `application.yml`에 `spring.servlet.multipart.max-file-size`를 함께 맞춘다(기본값 1MB라 조용히 413이 날 수 있다).

#### ⚠️ 내보내기 CSV에 UTF-8 BOM을 붙인다 (중요)

**BOM 없는 UTF-8 CSV를 Microsoft Excel이 열면 한글이 전부 깨진다.** Excel은 BOM이 없으면 시스템 기본 인코딩(한국어 Windows는 CP949)으로 읽기 때문이다.

"엑셀 가계부를 대체한다"는 제품이 내보낸 파일을 엑셀에서 못 여는 것은 치명적이고, **개발자 환경(VS Code, 메모장)에서는 멀쩡히 보여서 발견되지 않는다.**

```java
// 응답 본문 맨 앞에 EF BB BF 3바이트를 쓴다
out.write(new byte[]{(byte)0xEF, (byte)0xBB, (byte)0xBF});
```

- `Content-Type: text/csv; charset=UTF-8`
- `Content-Disposition: attachment; filename="moneylog_2026-09.csv"`
- **가져오기 쪽은 반대로 BOM을 제거하고 읽는다.** 안 그러면 첫 헤더가 `\uFEFF날짜`가 되어 열 매칭이 실패한다. 자기가 내보낸 파일을 자기가 못 읽는 상태가 된다.


#### ⚠️ 가져오기는 엑셀이 되돌려준 파일도 읽어야 한다 (중요)

위 BOM 항목이 **"우리 파일을 엑셀이 읽는"** 방향을 책임진다면, 이 절은 **그 반대 방향**을 책임진다. 사용자는 내려받은 CSV를 그대로 다시 올리지 않는다. **엑셀에서 열어 편집한 뒤** 올린다(`PRD.md` 4.4). 그 사이에 엑셀이 형식을 바꾼다.

| 엑셀이 바꾸는 것 | 결과 | 대응 |
|---|---|---|
| `CSV(쉼표로 분리)` 저장 시 **CP949**로 씀 | 한글 전부 깨짐 | UTF-8 엄격 디코딩 → 실패 시 **MS949 폴백** |
| 금액 셀의 천단위 서식 → `"12,500"` | `NumberFormatException` | 숫자 외 문자를 제거하고 파싱 |
| 날짜 구분자 변경 → `2026.09.14` | `DateTimeParseException` | 포맷 후보를 순서대로 시도 |

**셋 중 인코딩이 가장 중요하고 가장 발견하기 어렵다.**

`new String(bytes, UTF_8)`은 깨진 바이트에 **예외를 던지지 않고 조용히 `U+FFFD`로 치환한다.** 그러면 파싱은 "성공"하고 카테고리 이름만 깨진 문자로 바뀌어 **전 행이 `카테고리를 찾을 수 없습니다`로 실패한다.** 사용자도 개발자도 인코딩 문제를 카테고리 문제로 오인한다. 위 BOM 항목과 똑같이 **개발자 환경(VS Code로 저장)에서는 재현되지 않는다.**

```java
// UTF-8을 엄격 모드로 시도하고, 깨지면 MS949로 다시 읽는다.
// REPORT가 핵심이다 - 기본값 REPLACE로 두면 예외가 나지 않아 폴백이 영영 동작하지 않는다.
private String decode(byte[] bytes) {
    try {
        return StandardCharsets.UTF_8.newDecoder()
                .onMalformedInput(CodingErrorAction.REPORT)
                .onUnmappableCharacter(CodingErrorAction.REPORT)
                .decode(ByteBuffer.wrap(bytes)).toString();
    } catch (CharacterCodingException e) {
        return new String(bytes, Charset.forName("MS949"));
    }
}
```

- **`EUC-KR`이 아니라 `MS949`를 쓴다.** 엑셀이 쓰는 것은 EUC-KR의 상위집합인 CP949이고, `EUC-KR`로 읽으면 확장 음절(`똠`·`믜` 등)에서 다시 깨진다.
- 금액: `new BigDecimal(raw.replaceAll("[^0-9.]", ""))`. §8의 프론트 금액 입력이 쓰는 것과 같은 정규식이다.
- 날짜: `yyyy-MM-dd`(우리 내보내기 형식) → `yyyy.MM.dd` → `yyyy/MM/dd` 순으로 시도하고, 전부 실패하면 그 행만 실패 처리한다.
- **관대하게 읽되 내보내기는 바꾸지 않는다.** 내보내기 형식은 `yyyy-MM-dd` + 콤마 없는 숫자 하나뿐이다. 양쪽을 다 넓히면 왕복 검증(§12 22번)의 기준 자체가 사라진다.
- **BOM 제거는 디코딩 이후에 한다.** 순서를 뒤집으면 CP949 파일에서 엉뚱한 바이트를 잘라낸다.

#### CSV 가져오기는 중복을 검사하지 않는다

같은 파일을 두 번 올리면 거래가 두 번 등록된다. **의도된 결정이다.**

가져오기의 주 용도는 「엑셀 가계부 이전」이라는 1회성 시나리오이고, `PRD.md` 4.4가 **"실패분만 골라 다시 올림"**이라는 부분 재업로드를 전제한다. 여기에 해시 기반 중복 제거를 넣으면 `dedup_hash` 컬럼 + `WHERE deleted_at IS NULL` 부분 유니크 인덱스 + 상호명 정규화가 따라붙는데, 얻는 것은 "전체를 실수로 다시 올린 경우" 하나뿐이다.

- 화면(`/data`)에 **"이미 가져온 파일을 다시 올리면 중복 등록됩니다"**를 안내한다.
- 매달 카드사 내역을 올리는 형태로 제품이 바뀌면 그때 도입한다. 그 시점에는 중복이 치명적이 된다.

### 날짜·숫자 직렬화 포맷

- `createdAt`, `updatedAt` → **ISO-8601 UTC 문자열** (`2026-09-14T04:30:00Z`)
- `txnDate`, `from`, `to`, `asOf` → **`yyyy-MM-dd`**
- `yearMonth` → **`yyyy-MM`**
- `amount` 등 금액 → **JSON 숫자** (`12500.00`). 문자열로 감싸지 않는다
- 배열 형태(`[2026,9,14]`)로 직렬화되지 않도록 §3의 Jackson 항목을 지킨다(무설정이 정답)

### 인증 예외 케이스

토큰은 유효한데 해당 사용자가 조회되지 않는 경우는 **404가 아니라 401 `UNAUTHORIZED`**로 응답한다. 프론트는 401을 자동 로그아웃으로 처리하므로 일관된다.

---

## 6. 인증 설계

### 토큰 정책
- Access Token만 사용 (**24시간 만료**), Refresh Token 없음
- 만료 시 401 → 프론트에서 로그인 페이지로 리다이렉트
- 저장 위치: **localStorage** (MVP 기준)
- 요청 시 `Authorization: Bearer {token}`

### JWT 클레임 구성
```
sub   : user.id (숫자 문자열)
email : user.email
iat   : 발급 시각
exp   : 발급 + 24시간
```
- **`sub`는 이메일이 아니라 id를 담는다.** 인증 필터에서 PK 조회로 끝난다.
- `JwtAuthenticationFilter`는 `sub`를 파싱해 사용자 id를 얻고, `deleted_at IS NULL` 조건으로 조회한다.

### SecurityConfig 인가 경로 (필수)

```
permitAll:
  /api/v1/auth/signup
  /api/v1/auth/login
  /swagger-ui/**
  /v3/api-docs/**
  /error

그 외: authenticated
```

> ⚠️ **Swagger 경로를 빼먹으면 Phase 1의 DoD("Swagger 접속 확인")가 Phase 3에서 조용히 회귀한다.** SecurityConfig 작성 시 반드시 함께 넣는다.

#### ⚠️ CSRF 비활성화와 STATELESS 세션은 필수다 (중요)

**이 두 줄이 없으면 `POST /api/v1/auth/signup`부터 403으로 막힌다.**

`SecurityFilterChain`을 직접 정의하면 **CSRF 보호가 기본으로 켜진다.** 이 프로젝트는 쿠키가 아니라 `Authorization: Bearer` 헤더로 인증하는 stateless API이므로 CSRF 토큰을 발급하는 경로 자체가 없다. Spring Boot 4가 번들하는 Spring Security 7에서 특히 흔한 실패다.

```java
http
    .csrf(AbstractHttpConfigurer::disable)                      // JWT stateless — CSRF 토큰 경로 없음
    .sessionManagement(s -> s.sessionCreationPolicy(STATELESS))  // 세션 미사용
    .authorizeHttpRequests(auth -> auth
        .requestMatchers(...).permitAll()
        .anyRequest().authenticated()
    )
    .exceptionHandling(...)   // §9의 EntryPoint / AccessDeniedHandler
```

> **`authorizeRequests()`는 Spring Security 7에서 제거되었다.** 반드시 `authorizeHttpRequests()`를 쓴다. Boot 3 시절 예제를 그대로 옮기면 컴파일에 실패한다.

### Swagger JWT 인증 설정 (필수)

경로를 열어두는 것만으로는 부족하다. 보호된 API를 Swagger에서 실제로 호출하려면 Authorize 버튼이 있어야 한다.

```java
@SecurityScheme(
    name = "bearerAuth",
    type = SecuritySchemeType.HTTP,
    scheme = "bearer",
    bearerFormat = "JWT"
)
```
- `OpenAPI` 빈에 `addSecurityItem`으로 전역 적용하거나, 보호된 컨트롤러에 `@SecurityRequirement(name = "bearerAuth")`를 붙인다.

### CORS

- 허용 오리진: 환경변수 `CORS_ALLOWED_ORIGINS`. 쉼표로 구분된 목록을 받는다 (로컬은 `http://localhost:3000` 하나)
- 허용 메서드: `GET, POST, PUT, PATCH, DELETE, OPTIONS`
- **허용 헤더에 `Authorization`, `Content-Type`을 명시한다.** 기본값에 의존하면 프리플라이트에서 막히는 경우가 잦다.
- **`Content-Disposition`을 `exposedHeaders`에 추가한다.** CSV 다운로드 시 파일명을 읽으려면 필요하다. 빠뜨리면 브라우저가 헤더를 숨겨 파일명이 `download`가 된다
- 쿠키를 쓰지 않으므로 `allowCredentials`는 false

### 기본 카테고리 자동 생성 (필수)

카테고리가 하나도 없으면 거래를 한 건도 등록할 수 없어 가입 직후 화면이 막힌다. **`AuthService.signup()`이 같은 트랜잭션에서 아래를 생성한다.**

| type | 이름 | 색 |
|---|---|---|
| EXPENSE | 식비 | `#EF4444` |
| EXPENSE | 교통 | `#F59E0B` |
| EXPENSE | 주거/통신 | `#6366F1` |
| EXPENSE | 생활용품 | `#10B981` |
| EXPENSE | 문화/여가 | `#EC4899` |
| EXPENSE | 의료/건강 | `#14B8A6` |
| EXPENSE | 기타 | `#737373` |
| INCOME | 급여 | `#4F46E5` |
| INCOME | 기타수입 | `#737373` |

- 사용자는 이후 자유롭게 수정·삭제·추가할 수 있다. **특별 취급하는 플래그를 두지 않는다** — `is_default` 컬럼을 만들면 "기본 카테고리는 삭제 못 함" 같은 규칙이 따라붙고, 그건 아무도 요청하지 않은 제약이다.

### 보안 규칙
- 비밀번호: **BCrypt** 해싱, 6자 이상 + **UTF-8 72바이트 이하** (§4 ⚠️ 참조)
- 이메일: 형식 검증 + 중복 검사
- 모든 요청 DTO에 `@Valid` + Bean Validation
- **소유권 검증**: 거래·카테고리·예산 조회/수정/삭제 시 `user_id == 인증 사용자 id` 확인. 불일치 시 **404** 반환(존재 여부 노출 방지)
- JWT Secret 하드코딩 금지

### XSS 방어

**이 앱은 리치 텍스트를 다루지 않는다.** `memo`·`merchant`는 평문이고 React가 기본적으로 이스케이프하므로, 표본 프로젝트의 Jsoup/DOMPurify 이중 방어 체계가 필요 없다. 대신 **그 전제를 깨지 않는 것**이 방어의 전부다.

- **`dangerouslySetInnerHTML`을 쓰지 않는다.** 이 앱에 HTML을 렌더할 이유가 없다.
- **`category.color`를 인라인 스타일에 넣기 전 `#RRGGBB` 정규식으로 검증한다.** 서버가 §4 제약으로 검증하지만, 프론트에서도 한 번 확인한다. 검증 없이 `style={{ background: color }}`에 넣으면 CSS 값 주입 경로가 된다.
- 리치 텍스트 에디터를 도입하게 되면 **그때 Jsoup + DOMPurify 이중 방어를 함께 도입한다.** 에디터만 먼저 넣지 않는다.

---

## 7. 화면 목록

| 경로 | 화면 | 인증 |
|---|---|---|
| `/login` | 로그인 | X |
| `/signup` | 회원가입 | X |
| `/dashboard` | 월 대시보드 (요약·차트·예측·예산) | O |
| `/transactions` | 거래 목록 + **퀵 입력 바** (필터/검색/페이지네이션) | O |
| `/transactions/[id]` | 거래 상세 (항상 편집 가능) | O |
| `/budgets` | 카테고리별 월 예산 설정 + 카테고리 관리 | O |
| `/data` | CSV 가져오기 / 내보내기 | O |

- 미인증 상태로 보호된 경로 접근 시 `/login`으로 리다이렉트.
- 로그인 직후 기본 진입은 **`/dashboard`**.
- 인증 화면의 공통 헤더에는 **닉네임, 네비게이션, 로그아웃 버튼**을 둔다. 닉네임은 버튼이라 누르면 변경 팝업(`NicknameDialog`, `AUTH-10`)이 열린다.

### ⚠️ `/transactions/new` 페이지를 만들지 않는다 (중요)

거래 입력 필드는 여섯 개뿐이다(구분·금액·카테고리·날짜·거래처·메모). 별도 페이지로 빼면 **"목록 → 새로 만들기 클릭 → 페이지 이동 → 입력 → 저장 → 목록 복귀"** 다섯 단계가 되고, 이것이 가계부 앱을 그만두게 만드는 바로 그 마찰이다.

- `/transactions` 상단에 **퀵 입력 바**를 둔다. 한 줄에 여섯 필드가 들어가고, 저장하면 목록 맨 위에 항목이 추가되며 **폼은 비워지되 날짜와 구분은 유지**한다(연속 입력 대비).
- 모바일에서는 같은 폼이 세로로 쌓인다. 별도 화면을 만들지 않는다.
- **최근 사용 카테고리 3개를 버튼으로 노출한다.** 별도 API를 만들지 않고, 이미 받아온 거래 목록의 앞쪽에서 중복 제거해 뽑는다.
- **목록에서 행을 누르면 페이지 이동 없이 팝업(`TransactionEditDialog`)으로 수정한다.** 같은 폼 컴포넌트(`TransactionForm`)를 초기값과 함께 재사용하고, 삭제 버튼만 추가로 노출한다. 팝업은 페이지를 벗어나지 않으므로 §9의 이탈 확인 3계층(`beforeunload`/버튼/`popstate`)이 필요 없다 — 닫기 시도 시 dirty면 확인창만 띄운다(퀵 입력 바와 같은 이유).
  - `/transactions/[id]` 페이지 자체는 지우지 않고 **직접 링크·북마크로 들어왔을 때의 대체 경로**로 남긴다. 이 페이지는 원래 방식대로 §9의 3계층 이탈 가드를 그대로 쓴다.

---

## 8. UI 디자인 가이드

**방향**: 심플·모던. 장식보다 여백과 타이포그래피로 위계를 만든다.

### 컬러 (Tailwind 4 `@theme` 토큰) — Anthropic 브랜드 팔레트(Ivory/Slate/Clay)
- 배경(Ivory) `#FAF9F5` / 다크 `#1A1918`
- 카드 `#FFFFFF` / 다크 `#1F1E1D`
- 텍스트(Slate) `#141413` / 다크 `#FAF9F5`, 보조 `#73726C` / 다크 `#9C9A92`
- 액센트: **단일 컬러 1개만** — 라이트 `#B8542F`(흰 글자·링크가 WCAG AA 4.5:1 충족), 다크 `#D97757`(글자는 `#141413`). Clay 원색 `#D97757` 위에 흰 글자는 대비 3.1:1이라 라이트 모드에서 쓰지 않는다
- **수입 `#166C40`(포레스트 그린, 히트맵 초록 배경 위 AA) · 지출 `#4B4A45`(웜 그레이) · 잔액 `#B8542F`(딥 클레이)** — 액센트 규칙의 예외다. 금액의 방향을 색으로 구분하는 것이 이 앱의 핵심 정보이기 때문이다. 다크 모드는 수입 `#4FB57F` · 지출 `#B5B3AA` · 잔액 `#E8926F`. **예산 초과·음수 잔액의 경고 빨강은 지출색이 아니라 `--destructive`(`#B3452F`)를 쓴다.**
- 카테고리 팔레트 (색 미지정 시 순서대로 배정, 사용자 데이터 성격이라 브랜드 팔레트와 무관하게 유지):
  `#EF4444 #F59E0B #10B981 #4F46E5 #EC4899 #14B8A6 #8B5CF6 #F97316 #737373`

### 스타일 원칙
- 그림자 대신 **0.5px border**(`#DEDCD1`)로 면 구분. 그림자는 모달/드롭다운에만.
- 라운드: 카드 `rounded-xl`, 버튼/인풋 `rounded-lg`
- 폰트: 본문은 **Pretendard**. ⚠️ **Google Fonts에 없으므로 `next/font/google`로 불러올 수 없다.** 폰트 파일(`.woff2`)을 `src/app/fonts/`에 넣고 **`next/font/local`**로 로드한다. 가변 폰트(`PretendardVariable.woff2`) 하나면 충분하다.
  - **헤딩도 본문과 같은 Pretendard로 통일한다**(세리프 사용 안 함). `--font-heading`은 `var(--font-sans)`를 가리키며, 제목에 쓰는 `font-heading` 클래스는 그대로 둔다.
- 글자 크기는 Tailwind 기본값보다 **3px씩 키운 값**이다(`globals.css`의 `--text-*`): 본문 17px(`text-sm`) / 항목 제목 19px(`text-base`) semibold / 캡션 15px(`text-xs`). 여백·높이(rem)는 그대로다
- **금액은 `tabular-nums`를 적용한다.** 비례 숫자로 두면 목록에서 자릿수가 세로로 어긋나 읽기 어렵다. Tailwind의 `tabular-nums` 유틸리티 한 줄이다
- 컨테이너 `max-w-5xl`(대시보드는 차트가 있어 표본보다 넓다), 패딩 모바일 16px · 데스크톱 24px
- **다크 모드**: 토큰을 라이트/다크 양쪽으로 정의하고, 헤더의 아이콘 버튼으로 **사용자가 직접 전환하는 토글**을 둔다(`UX-09`). 선택값은 `localStorage`에 저장되어 재방문 시 유지되고, 선택하지 않았다면 `prefers-color-scheme`(OS 설정)을 따른다.
  > ⚠️ **class 전략(`:root.dark`)을 쓴다.** 토글이 있는 이상 미디어쿼리만으로는 "시스템과 다른 모드를 명시적으로 고르는" 요구를 만족할 수 없다. 다만 하이드레이션 이후에 클래스를 붙이면 라이트로 그려졌다가 다크로 바뀌는 FOUC가 그대로 재현되므로, **루트 레이아웃의 `<head>`에 하이드레이션 전 동기 실행되는 인라인 스크립트**를 둔다 — `localStorage`에 저장된 값(없으면 시스템 선호)으로 `<html>`에 `.dark`를 미리 붙인다. `useTheme` 훅은 그 결과를 초기 상태로 읽기만 하고, 토글 시 클래스와 `localStorage`를 함께 갱신한다.
  > ⚠️ **`@theme`을 `@media`/클래스 선택자 안에 중첩하지 않는다.** Tailwind v4에서 `@theme`은 **최상위에만 올 수 있다.** 라이트 값을 `@theme`에 한 번 선언해 유틸리티를 만들고, **다크에서는 생성된 커스텀 프로퍼티를 `:root.dark`에서 덮어쓴다.**
  > ```css
  > @theme { --color-bg: #FAF9F5; }   /* 유틸리티 생성 */
  > :root.dark { --color-bg: #1A1918; } /* 값만 교체 */
  > ```

### ⚠️ 금액 입력에 `<input type="number">`를 쓰지 않는다 (중요)

천단위 콤마(`1,250,000`)를 표시해야 하는데 `type="number"`는 콤마가 들어가는 순간 값을 **빈 문자열로 만든다.** 또 모바일에서 스피너가 뜨고 스크롤로 값이 바뀌는 사고가 난다.

```tsx
<input
  type="text"
  inputMode="numeric"       // 모바일 숫자 키패드
  value={formatted}         // "1,250,000"
  onChange={e => setRaw(e.target.value.replace(/[^\d]/g, ""))}
/>
```

- 상태는 **콤마 없는 원본 문자열**로 들고, 표시할 때만 포맷한다. `lib/money.ts`에 `formatAmount`/`parseAmount` 두 함수를 두고 모든 화면이 그것만 쓴다.
- **`dirty` 판정도 원본 문자열로 비교한다**(§3 참조). 포맷된 값끼리 비교하면 콤마 유무로 오판한다.
- 표시 포맷은 `toLocaleString("ko-KR")`을 쓴다. 별도 라이브러리가 필요 없다.

### 인터랙션 (Motion)

import는 항상 `motion/react`에서 한다.
```ts
import { motion, AnimatePresence, useReducedMotion } from "motion/react";
```

- 목록 등장: `opacity 0→1` + `y 8→0`, stagger 30ms
- 삭제: `opacity→0` + `height→0`, `AnimatePresence`
- 차트 막대: 너비 0 → 목표값, 300ms (**차트에만 예외적으로 200ms를 넘긴다**)
- **그 외 200ms 이내, 과한 모션 금지.** `useReducedMotion`으로 `prefers-reduced-motion` 존중
- **화면 진입 시 한 번 도는 연출만 예외로 200ms를 넘길 수 있다.** 스크롤·호버마다 흩어지는 효과는 두지 않는다(한 번 잘 짠 등장이 낫다).
  - 순차 등장: `.reveal-stack`(`globals.css`)을 컨테이너에 붙이면 직계 자식이 위에서부터 400ms · 60ms 간격으로 나타난다(CSS만 사용, reduced-motion이면 꺼짐). 대시보드와 로그인/회원가입이 쓴다.
  - 숫자 카운트업: 대시보드 요약 숫자(`AnimatedAmount`, 700ms). 끝나면 정확한 값으로 고정한다.
  - 헤더 메뉴 알약: `layoutId`로 활성 메뉴 사이를 미끄러져 이동한다(스프링, 사용자 동작에 대한 응답).
  - 로그인/회원가입 일러스트(`AuthShowcase`): 그래프 그려짐 → 동전 → 도장 순서로 약 2초 1회. 이후 칩 두 개만 5px 진폭으로 천천히 떠다닌다. 좁은 화면에서는 칩을 일러스트 위가 아니라 헤드라인 아래에 쌓아 문구가 함께 보이게 한다.
  - 대시보드 맨 위 한 줄(`DashboardHero`): 시간대별(05~11·11~17·17~22·22~05시) 무지출 챌린지 문구 앞에 `닉네임님`을 붙인 h1과 주요 행동(거래 기록하기)만 둔다. 그 아래 큰 영역은 광고 배너(`AdBannerCarousel`)다: 라이브러리 없이 한 장씩 그리고 5초마다 넘기며(hover·포커스·모션 감소 시 멈춤), 점 버튼이나 좌우 드래그(터치는 스와이프)로 이동한다. 문구·이미지·주소는 `lib/banners.ts` 한 곳에서 바꾸고 이미지는 `public/assets/banners/*.svg`다. **지금 3장은 임의로 넣은 자리표시**다.
- **일러스트**: 이미지 파일 없이 SVG를 직접 그린다(`components/illustration/Art.tsx`, `next/image` 미사용). 종이를 오려 붙인 듯한 납작한 도형 + 굵고 둥근 선으로 통일하고, 색은 전부 테마 토큰(`var(--primary)` 등)이라 라이트/다크를 따로 그리지 않는다. 장식이므로 `aria-hidden`이고 의미는 옆 제목·문구가 전달한다. 빈 상태·에러 상태(`EmptyState`의 `art`, `ErrorState`)와 화면 제목(`PageHeader`), 대시보드 히어로, 로그인/회원가입 패널에 쓴다. 동전 하나만 4px 진폭으로 떠오른다(`.art-bob`).
- **배경 분위기**: `body`에 Clay·수입 그린이 화면 위쪽 모서리에서 은은하게 번지는 그라데이션을 둔다(스크롤하면 사라짐). 대시보드 히어로·로그인 패널의 종이 배경(`paper.ts`)은 그 위에 얹히므로 초록을 섞지 않고 Clay 한 색만 옅게 쓴다(섞으면 탁한 회색으로 보인다). 헤더는 `bg-background/80` + `backdrop-blur`. 그림자를 쓰지 않는다는 원칙은 그대로다.

---

## 9. 상태 처리 규칙

### ⚠️ App Router 렌더링 경계

인증 토큰이 localStorage에 있고 데이터를 React Query로 가져오므로, **이 프로젝트의 페이지는 사실상 전부 클라이언트 컴포넌트다.**

- 모든 `page.tsx`에 **`"use client"`를 붙인다.**
- 서버 컴포넌트에서 데이터를 미리 가져오려 시도하지 않는다. 서버에는 토큰이 없다.
- `app/layout.tsx`(루트)만 서버 컴포넌트로 두고, Provider들은 별도 클라이언트 컴포넌트로 분리해 감싼다.
- **동적 라우트 파라미터는 `next/navigation`의 `useParams()`로 읽는다.** `/transactions/[id]`가 유일한 대상이다. Next.js 15부터 페이지가 props로 받는 `params`·`searchParams`는 **Promise**라 `React.use()`로 풀어야 하고, 동기 접근 호환 모드는 16에서 완전히 제거됐다. `useParams()`는 동기 훅이라 이 변경과 무관하므로, **props의 `params`를 쓰지 않는다.**

### ⚠️ `useSearchParams`는 Suspense 경계가 필요하다

Next.js 15에서 `useSearchParams`를 쓰는 컴포넌트는 **`<Suspense>`로 감싸지 않으면 `npm run build`가 프리렌더 단계에서 실패한다.** 개발 서버에서는 통과하다가 빌드에서 터지므로 늦게 발견된다.

해당되는 곳이 둘이다.
- `/transactions` — 기간·구분·카테고리·검색어·페이지를 URL 쿼리로 관리한다
- `/dashboard` — 조회 대상 월(`?ym=2026-09`)을 URL 쿼리로 관리한다

두 페이지 모두 실제 로직을 내부 컴포넌트로 빼고 `<Suspense fallback={<Skeleton />}>`로 감싼다.

### 목록·대시보드 상태는 URL 쿼리로 관리한다

```
/transactions?page=2&type=EXPENSE&categoryId=3&from=2026-09-01&keyword=스타벅스
/dashboard?ym=2026-09
```

- 새로고침해도 상태가 유지되고, 뒤로가기가 자연스럽게 동작하며, 링크 공유가 된다.
- 쿼리 키가 URL 상태와 1:1로 대응해 캐시가 명확해진다.
- **대시보드의 월 이동(◀ 2026년 9월 ▶)이 뒤로가기로 되돌아간다.** 이게 `useState`였다면 뒤로가기가 앱을 벗어난다.

### ⚠️ 라우트 보호는 middleware로 하지 않는다

토큰을 **localStorage**에 두므로 Next.js middleware로 라우트를 보호할 수 없다. middleware는 서버에서 실행되어 쿠키만 읽을 수 있다.

- **`middleware.ts`를 만들지 않는다.**
- `(main)` 그룹에 클라이언트 레이아웃을 두고 `useAuth`로 인증 여부를 판정한다.
- 판정이 끝나기 전에는 스켈레톤을 보여준다. 인증 상태는 마운트 이후에만 읽는다.
- 미인증이면 `router.replace("/login")`.

#### ⚠️ `useAuth`는 토큰 존재 여부가 아니라 `exp`를 봐야 한다

**"localStorage에 토큰 문자열이 있는가"만 검사하면 만료된 토큰이 판정을 통과한다.** 그러면 보호 레이아웃이 인증으로 판정 → 화면 렌더 → API 호출 → **401** → 자동 로그아웃 → `/login`이 되어, 그 왕복 동안 **보호된 화면이 노출된다.**

```ts
// 서명 검증은 서버가 한다. 프론트는 만료 시각만 읽으면 된다.
function isExpired(token: string): boolean {
  try {
    const { exp } = JSON.parse(atob(token.split(".")[1]));
    return typeof exp !== "number" || exp * 1000 <= Date.now();
  } catch {
    return true;   // 형식이 깨진 토큰도 만료로 취급
  }
}
```

- 만료로 판정되면 **즉시 토큰을 폐기**하고 미인증으로 처리한다.
- 라이브러리를 추가하지 않는다. `atob` + `JSON.parse`로 충분하다.

### React Query 쿼리 키 규약 (필수)

```ts
['transactions', { page, size, type, categoryId, from, to, keyword }]  // 목록
['transactions', id]                                                    // 단건
['categories']                                                          // 카테고리 목록
['stats', 'monthly', { yearMonth, asOf }]                               // 대시보드
['stats', 'recurring', { asOf }]                                        // 고정지출 감지
['budgets', { yearMonth }]                                              // 예산
['auth', 'me']                                                          // 내 정보
```

- 목록 캐시를 조작할 때는 **현재 화면의 필터·페이지가 포함된 키**를 대상으로 한다.
- **거래를 변경하면 `['stats']`도 함께 무효화한다.** 안 그러면 거래를 추가했는데 대시보드 합계가 그대로다. 이건 표본 프로젝트에 없던 의존이므로 놓치기 쉽다.
  ```ts
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ["transactions"] });
    queryClient.invalidateQueries({ queryKey: ["stats"] });   // 잊지 말 것
    queryClient.invalidateQueries({ queryKey: ["budgets"] }); // 소진율도 바뀐다
  }
  ```

### 낙관적 업데이트 (React Query)

**삭제에만 적용한다.** 생성·수정은 서버 응답을 기다린다.

- 삭제는 결과가 자명하고(행이 사라진다) 되돌리기도 쉽다 → `onMutate`에서 즉시 제거
- 생성은 서버가 채우는 값(`id`, 카테고리 조인 결과)이 있어 낙관적으로 그리면 임시 데이터와 실제가 어긋난다. **퀵 입력 바는 버튼에 로딩 상태만 표시**하고 응답을 기다린다. 로컬 API는 수십 ms라 체감되지 않는다
- `onMutate`: `cancelQueries` → 스냅샷 저장 → 캐시 직접 수정
- `onError`: 롤백 + 토스트 알림(`sonner`)
- `onSettled`: 위의 3종 `invalidateQueries`

> **표본 프로젝트의 "토글 연타" 문제는 이 앱에 없다.** 완료 체크박스 같은 고빈도 토글이 없기 때문이다. `mutation scope` 직렬화를 도입하지 않는다.

### ⚠️ 이탈 확인 대화상자는 두 계층으로 나눠 구현한다

`TXN-09`("변경 사항이 있는 상태로 이탈 시 확인")는 **한 줄짜리 작업이 아니다.** App Router에는 Pages Router의 `router.events`가 없고, **공식 내비게이션 차단 API도 없다.**

| 이탈 경로 | 방어 수단 |
|---|---|
| 새로고침 · 탭 닫기 · 주소창 직접 이동 | `beforeunload` 이벤트 |
| 페이지 내 "취소"·"목록으로" 버튼 | **버튼 자체 핸들러**에서 확인 후 `router.push` |
| 브라우저 뒤로가기 | `popstate` 리스너 + 취소 시 `history.pushState`로 되돌리기 |

- **서드파티(`next-navigation-guard` 등)를 도입하지 않는다.** §3 스택에 없다.
- **적용 대상은 `/transactions/[id]`뿐이다.** 퀵 입력 바는 페이지를 떠나지 않으므로 가드가 필요 없다.
- 폼이 `dirty`일 때만 가드를 켠다. 저장 직후에는 반드시 해제한다.

### 로딩 / 빈 상태 / 에러
- **로딩**: 스피너 대신 스켈레톤. 목록은 항목형 스켈레톤 5개, 대시보드는 카드형 스켈레톤
- **빈 상태**: 아이콘 + 문구 + CTA 버튼
  - 거래 없음 → "아직 기록이 없어요" + 퀵 입력 바로 포커스 이동
  - **대시보드 데이터 부족** → "예측하려면 데이터가 조금 더 필요해요" (§5의 `forecast: null` 케이스)
- **검색 결과 없음**: 빈 상태와 문구를 구분
- **에러**: 인라인 에러 카드 + 재시도 버튼. 401은 로그인으로 이동

### 경계 상황 처리

- **마지막 항목 삭제로 현재 페이지가 비는 경우**: `page > 0`이고 삭제 후 항목이 0개면 **이전 페이지로 이동**한다.
  > ⚠️ **페이지 이동은 `onMutate`가 아니라 `onSuccess`에서 한다.** `onMutate`에서 이동하면 쿼리 키가 바뀌어, 삭제 실패 시 `onError`의 롤백이 **사용자가 보고 있지 않은 캐시에 적용된다.**
- **`/transactions/[id]`에서 404**: 목록으로 리다이렉트하지 않고 **"거래 내역을 찾을 수 없습니다" 화면과 목록으로 가기 버튼**을 보여준다. Next.js `notFound()`는 쓰지 않는다(클라이언트 데이터 페칭이므로).
- **거래가 없는 달의 대시보드**: 에러가 아니라 **모두 0인 정상 상태**다. 차트 자리에 "이 달에는 기록이 없어요"를 보여주고, 월 이동 버튼은 그대로 동작해야 한다.
- **카테고리가 하나도 없는 경우**: 가입 시 자동 생성(§6)되므로 사용자가 전부 지운 경우에만 발생한다. 퀵 입력 바를 비활성화하고 "카테고리를 먼저 만들어 주세요" + `/budgets` 링크를 보여준다.

### 페이지네이션 공용 컴포넌트
`src/components/common/Pagination.tsx`
- Props: `currentPage`, `totalPages`, `onPageChange`
- 현재 페이지 주변 5개 + 처음/이전/다음/마지막
- 페이지 수 1 이하면 렌더링하지 않음
- 모바일은 "3 / 12" 형태로 축약

---

## 10. 코딩 컨벤션 · 환경 변수 · 실행

### Java
- 클래스 `PascalCase`, 메서드/변수 `camelCase`, 상수 `UPPER_SNAKE_CASE`
- DTO 네이밍: `TransactionCreateRequest`, `TransactionResponse`, `MonthlyStatsResponse`, `PageResponse<T>`, `ApiResponse<T>` — **record** 사용
- **엔티티를 컨트롤러에서 직접 반환하지 않는다.** 항상 DTO로 변환
- 엔티티에 `@Setter` 금지. 변경은 의미 있는 메서드로 (`updateAmount(BigDecimal)`, `softDelete()`)
- 생성자 주입 + `@RequiredArgsConstructor`
- Service에 `@Transactional`, 조회는 `readOnly = true`
- **집계 쿼리는 `@Query`로 명시적으로 작성한다.** 메서드 이름 규칙(`findByUserIdAndTxnDateBetween...`)으로 집계를 표현하려 들면 이름이 감당 못 할 길이가 된다

### TypeScript
- 컴포넌트 `PascalCase`, 훅 `useCamelCase`
- 타입은 `src/types/`에 모아 백엔드 DTO와 이름을 맞춘다
- `any` 금지. 불가피하면 `unknown` + 타입 가드
- 서버 상태는 React Query, UI 상태만 `useState`
- API 호출은 `src/lib/apiClient.ts`로 통일 (토큰 주입, **`ApiResponse` 언래핑**, 401 처리)
- **금액 포맷·파싱은 `lib/money.ts`, 날짜 포맷은 `lib/date.ts`만 쓴다.** 화면에서 `toLocaleString`·`format`을 직접 부르지 않는다

### 공통
- **모든 주석은 한글로 작성한다.**

### 환경 변수

프로파일은 셋이다. `application.yml`(공통) · `application-local.yml`(로컬) · `application-test.yml`(테스트, `src/test/resources`).

| 프로파일 | `ddl-auto` | DB |
|---|---|---|
| local | `update` | `moneylog_db` |
| test | `create-drop` | `moneylog_test` |

**`application.yml`에 `spring.profiles.active: local`을 기본값으로 둔다.** 지정하지 않으면 `./mvnw spring-boot:run`이 DB 접속 정보 없이 기동을 시도해 실패한다.

**JWT 서명 알고리즘은 HS256으로 고정한다.** `JWT_SECRET`은 최소 32바이트(256비트)여야 하며, 짧으면 `WeakKeyException`으로 기동에 실패한다.

> ⚠️ **"32바이트"는 디코드 후 기준이다.** Base64 문자열로 두고 디코드해서 쓰면 **32자 문자열이 24바이트**가 되어 `WeakKeyException`이 난다. 이 프로젝트는 **raw UTF-8 문자열 32자 이상**을 그대로 키로 쓴다.

`moneylog-backend/.env.example`:
```
DB_URL=jdbc:postgresql://localhost:5432/moneylog_db
DB_USERNAME=
DB_PASSWORD=
JWT_SECRET=            # raw UTF-8 32자 이상 (Base64 디코드하지 않음)
JWT_EXPIRATION=86400000
CORS_ALLOWED_ORIGINS=http://localhost:3000
```

`moneylog-frontend/.env.example`:
```
NEXT_PUBLIC_API_BASE_URL=http://localhost:8080
NEXT_PUBLIC_KAKAO_MAP_KEY=            # Kakao Developers JavaScript 키. 플랫폼>Web에 도메인 등록 필요(TXN-14)
```

> ⚠️ **카카오맵 JS 키는 Kakao Developers 콘솔의 "플랫폼 > Web"에 쓰는 도메인(포트 포함, 예: `http://localhost:3000`)이 등록돼야 동작한다.** 등록하지 않으면 SDK 스크립트 요청이 401로 거부되어 지도가 뜨지 않는다. 앱에 JS 키가 여러 개 있다면 **도메인 등록은 키 단위**이므로, `.env.local`에 넣은 키와 실제로 도메인을 등록한 키가 같은지 확인한다.

각 저장소의 `.gitignore`에 아래를 넣는다. **`.env*`만 쓰면 `.env.example`까지 무시되므로 예외 줄이 반드시 필요하다.**

```gitignore
.env*
!.env.example
```

### 실행 명령어

> ⚠️ **개발 환경은 Windows다.** 아래 POSIX 명령은 **Git Bash 기준**이다.

```bash
# DB 생성 (최초 1회)
createdb moneylog_db
createdb moneylog_test

# 백엔드
cd moneylog-backend
./mvnw spring-boot:run          # http://localhost:8080
./mvnw test
# Swagger: http://localhost:8080/swagger-ui/index.html

# 프론트엔드
cd moneylog-frontend
npm install
npm run dev                     # http://localhost:3000
npm run build
```

| 목적 | POSIX (Git Bash) | Windows (PowerShell/cmd) |
|---|---|---|
| Maven 실행 | `./mvnw spring-boot:run` | `.\mvnw.cmd spring-boot:run` |
| 테스트 | `./mvnw test` | `.\mvnw.cmd test` |
| DB 생성 | `createdb moneylog_db` | `& "C:\Program Files\PostgreSQL\17\bin\createdb" -U postgres moneylog_db` |

- **`./mvnw`는 POSIX 셸 스크립트라 PowerShell에서 실행되지 않는다.** Git Bash를 쓰거나 `mvnw.cmd`를 쓴다.
- **`createdb`는 PostgreSQL `bin`이 PATH에 없으면 명령 자체가 존재하지 않는다.** 또한 `-U postgres`를 빼면 Windows 사용자명으로 접속을 시도해 실패한다.
- **`moneylog-backend/.gitattributes`에 `mvnw text eol=lf`를 넣는다.** Git이 CRLF로 체크아웃하면 Git Bash에서 `bad interpreter` 오류가 난다.
- JDK가 여러 개 설치된 환경이라면 **`JAVA_HOME`과 PATH가 같은 버전(21)을 가리키는지 확인한다.**

---

## 11. 에러 처리

`exception/` 패키지에 `BusinessException`(+ `ErrorCode` enum)과 `GlobalExceptionHandler`(`@RestControllerAdvice`)를 구현한다.

| 예외 | 상태 | 코드 |
|---|---|---|
| 유효성 검증 실패 | 400 | `INVALID_INPUT` |
| 거래 type ≠ 카테고리 type | 400 | `CATEGORY_TYPE_MISMATCH` |
| CSV 형식 오류 / 용량·행수 초과 | 400 | `INVALID_CSV` |
| 인증 실패 / 토큰 만료 | 401 | `UNAUTHORIZED` |
| 권한 없음 | 403 | `FORBIDDEN` |
| 거래 없음 / 소유자 불일치 | 404 | `TRANSACTION_NOT_FOUND` |
| 카테고리 없음 / 소유자 불일치 | 404 | `CATEGORY_NOT_FOUND` |
| **API 경로 없음** | 404 | **`NOT_FOUND`** |
| **지원하지 않는 HTTP 메서드** | 405 | **`METHOD_NOT_ALLOWED`** |
| 이메일 중복 (회원가입) | 409 | `EMAIL_DUPLICATED` |
| 카테고리 이름 중복 | 409 | `CATEGORY_DUPLICATED` |
| 서버 오류 | 500 | `INTERNAL_ERROR` |

> ⚠️ **`NOT_FOUND`와 `TRANSACTION_NOT_FOUND`를 겸용하지 않는다.** 둘 다 404지만 메시지가 다르다.
>
> ⚠️ **아래 둘은 `@RestControllerAdvice`의 catch-all이 삼켜 500으로 나가기 쉽다.** `@ExceptionHandler(Exception.class)`가 Spring MVC 표준 예외를 종류를 가리지 않고 잡기 때문이다. **catch-all 위에 구체적인 핸들러를 두어야** 아래로 흘러가지 않는다.
>
> | 요청 | 던져지는 예외 |
> |---|---|
> | 없는 경로 | `org.springframework.web.servlet.resource.NoResourceFoundException` |
> | 잘못된 메서드 | `org.springframework.web.HttpRequestMethodNotSupportedException` |
>
> ⚠️ **미인증 요청에서는 재현되지 않는다.** Security 필터가 먼저 401로 막아 컨트롤러까지 가지 않으므로, **인증 토큰을 넣고 확인해야 한다.**

- 검증 실패 시 필드별 메시지 포함
- 스택트레이스·내부 예외 메시지를 클라이언트에 노출하지 않는다
- **`ArithmeticException`을 `GlobalExceptionHandler`에 매핑하지 않는다.** §4의 `divide` 규칙을 지키면 발생하지 않아야 하고, 매핑해두면 계산 버그가 400으로 위장되어 조용히 숨는다. 터지게 두고 500 로그로 잡는다

### ⚠️ Security 필터 단계의 401/403은 별도 처리가 필요하다

`GlobalExceptionHandler`는 `@RestControllerAdvice`라 **컨트롤러에 진입한 요청의 예외만** 잡는다. JWT가 없거나 만료돼서 `JwtAuthenticationFilter` 단계에서 거부되면 Spring Security의 기본 응답이 그대로 나가, **"모든 응답이 `{success, data, error}` 포맷"이라는 규칙이 401에서 깨진다.**

`config/`에 다음 둘을 구현해 `SecurityFilterChain`의 `exceptionHandling`에 등록한다.

- `AuthenticationEntryPoint` → 401을 `ApiResponse` 포맷(`UNAUTHORIZED`)으로 직접 write
- `AccessDeniedHandler` → 403을 `ApiResponse` 포맷(`FORBIDDEN`)으로 직접 write

---

## 12. 테스트

### 테스트 DB — PostgreSQL로 확정

**H2를 쓰지 않는다.** 집계에 `LOWER(...) LIKE`, `COALESCE`, `NUMERIC` 반올림, `date_trunc` 등 PostgreSQL 동작에 의존하므로 H2에서는 결과가 갈린다. **금액 계산이 제품의 본체인 앱에서 DB를 바꿔 테스트하는 것은 테스트를 무의미하게 만든다.** Docker를 쓰지 않기로 했으므로 Testcontainers도 배제된다.

→ **로컬 PostgreSQL에 `moneylog_test` 데이터베이스를 만들어 사용한다.** `src/test/resources/application-test.yml`에 연결 정보를 두고, `ddl-auto: create-drop`으로 매 실행마다 초기화한다.

### 단위 테스트

- Repository 테스트는 `@DataJpaTest` 기반으로 작성한다.
- ⚠️ **`@DataJpaTest`는 기본적으로 DataSource를 임베디드 DB로 교체하려 한다.**
  ```java
  @DataJpaTest
  @AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
  @ActiveProfiles("test")
  ```
- ⚠️ **`@EnableJpaAuditing`을 `@Configuration` 클래스에 두면 `@DataJpaTest`가 로드하지 않아 `created_at`이 null이 된다.** 메인 애플리케이션 클래스에 붙이거나, 별도 config를 만들고 테스트에서 `@Import`한다.
- **예측·집계 로직과 CSV 파서는 순수 단위 테스트로 검증한다.** DB 없이 입력→출력만 보면 되는 코드이므로 `@SpringBootTest`를 띄우지 않는다.

### 통합 테스트 (`@SpringBootTest` + `MockMvc`)

**테스트는 마지막에 몰아 쓰지 않는다.** 기능을 만든 Phase에서 함께 작성해 그 Phase의 완료 조건으로 삼는다.

**Phase 3에서 작성 (인증)**
1. 회원가입 성공 / 이메일 중복 시 409
2. **회원가입 시 기본 카테고리 9개가 함께 생성됨**
3. 로그인 성공 시 유효 JWT / 비밀번호 오류 시 401
4. 토큰 없이 보호된 엔드포인트 호출 시 401 **+ 응답이 `ApiResponse` 포맷인지 확인**
5. 한글 25자(75바이트) 비밀번호 가입 시 500이 아니라 400

**Phase 4에서 작성 (카테고리 · 거래)**
6. 거래 생성 → 목록 조회 시 `data.content`와 페이지네이션 필드 검증, **카테고리가 함께 내려옴**
7. Soft Delete 후 목록에서 제외 + `deleted_at` 기록
8. **타 사용자의 거래·카테고리 접근 시 404** (소유권 검증)
9. 지출 카테고리에 수입 거래 생성 시 400 `CATEGORY_TYPE_MISMATCH`
10. `amount = 0` 또는 음수 시 400
11. **삭제한 카테고리를 쓰던 과거 거래가 목록에서 사라지지 않고, 카테고리 선택 목록에서는 빠짐**
12. 같은 날짜 거래가 여러 건일 때 페이지 1·2에 **중복·누락 없음** (2차 정렬 키 검증)
13. 목록 조회 시 카테고리 조회 쿼리가 **건수에 비례해 늘지 않음** (N+1)

**Phase 5에서 작성 (집계 · 예측)**
14. 거래가 없는 달 조회 시 500이 아니라 모두 0 (`COALESCE` 검증)
15. 직전 3개월 데이터가 없으면 `forecast`가 `null`
16. 런레이트가 `확정 + 일평균 × 남은일수`와 일치 (소수 둘째 자리까지)
17. `daysElapsed < 7`이면 `anomalies`가 빈 배열
18. 고정지출 감지가 3개월 연속 동일 상호를 찾아내고, 2개월만 있는 상호는 제외
19. **`asOf`를 바꾸면 결과가 바뀌고, 서버 시각에 의존하지 않음** (§4 ⚠️ 검증)

**Phase 6에서 작성 (예산 · CSV)**
20. 예산 upsert가 기존 행을 갱신하고, `amount=0`인 항목은 행을 제거
21. 예산 0일 때 소진율이 `Infinity`/`NaN`이 아님
22. 내보낸 CSV를 **그대로 다시 가져오면 건수가 일치** (왕복 검증)
23. 내보낸 CSV의 첫 3바이트가 `EF BB BF` (BOM)
24. 메모에 콤마·따옴표·줄바꿈이 포함된 거래의 왕복 검증
25. 없는 카테고리 이름이 섞인 CSV → 해당 행만 실패, 나머지는 성공
26. **CP949로 인코딩된 CSV의 한글이 깨지지 않고 가져와진다.** 금액 `12,500`과 날짜 `2026.09.14`도 정상 파싱된다
    > ⚠️ **픽스처를 UTF-8로 만들면 이 테스트는 아무것도 검증하지 못한다.** `"...".getBytes(Charset.forName("MS949"))`로 바이트를 직접 만들어 넣는다.

**Phase 12**에서는 새 테스트를 쓰지 않고 전체 통과 여부와 검증 체크리스트만 확인한다.

### 시드 데이터

예측·집계·페이지네이션 DoD를 확인하려면 **여러 달에 걸친** 데이터가 필요하다.

- `src/main/resources/db/seed-dev.sql` — 테스트 계정 1개 + 기본 카테고리 + **최근 6개월치 거래 약 400건**
  > ⚠️ **한 달치만 넣으면 예측 로직을 전혀 검증할 수 없다.** 기준선이 직전 3개월이므로 최소 4개월치가 필요하고, 고정지출 감지를 보려면 **같은 상호·비슷한 금액의 거래를 3개월 연속으로 심어둬야 한다**(예: "넷플릭스 17,000원" 매월 5일).
- `src/main/resources/db/seed-perf.sql` — 같은 계정에 **24개월치 20,000건.** 성능 DoD 측정 전용
  > 400건으로는 성능 지표가 무의미하다. 집계 쿼리가 인덱스를 타는지는 데이터가 충분해야 드러난다.
- local 프로파일에서 수동 실행하는 용도다.
- **`completed`/`priority` 같은 DEFAULT 없는 컬럼을 시드에서 생략하지 않는다.** `type`·`amount`·`txn_date`·`category_id`는 전부 NOT NULL이며 DB DEFAULT가 없으므로 INSERT 문에 명시적으로 넣는다.

---

## 13. 작업 진행 원칙

- 한 번에 전체를 생성하지 않는다. **`ROADMAP.md`의 Phase 단위로 진행**하고, 각 Phase가 끝나면 실행/검증 결과를 보고한 뒤 멈춘다.
- 백엔드와 프론트엔드는 **별도 저장소이므로 커밋을 섞지 않는다.** 각 저장소에서 개별 커밋한다.
- **커밋 여부는 항상 사용자에게 확인받고 진행한다.**
- 스펙에 없는 기능을 임의로 추가하지 않는다. 필요하다고 판단되면 먼저 제안한다.
- **라이브러리 버전은 §3 「버전 관련 확정 사항」을 그대로 따른다.** 재조사하거나 다른 버전으로 바꾸지 않는다. §3에 없는 라이브러리만 확인 후 명시한다.
- API 계약(§5)은 백엔드·프론트 양쪽의 기준이다. 한쪽에서 임의로 바꾸지 않는다.
- **예측 수식(§5)은 백엔드에만 구현한다.** 프론트가 같은 계산을 다시 하면 두 곳이 갈라진다.

---

## 14. 향후 확장 후보 (범위 밖)

**`docs/PRD.md` 9장이 정본이다.** 이번 MVP에서는 구현하지 않는다.
