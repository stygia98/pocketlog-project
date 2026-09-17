# ROADMAP — 포켓로그(PocketLog)

> **버전** 1.4 · **최종 수정** 2026-09-17
> 이 문서는 "어떤 순서로 만드는가"를 정의하며, **완료 판정의 정본**이다.
> **한 번에 전체를 생성하지 않는다.** Phase 단위로 진행하고, 각 Phase의 DoD를 모두 만족한 뒤 다음으로 넘어간다.
> 기술 규칙은 `CLAUDE.md`, 기능 정의는 `PRD.md` 참조.

---

## 진행 현황

| Phase | 내용                                         | 저장소   | 상태 |
| ----- | -------------------------------------------- | -------- | ---- |
| 1     | 저장소 초기화 + 백엔드 스캐폴딩              | 전체     | ✅   |
| 2     | 도메인 & DB                                  | backend  | ✅   |
| 3     | 인증 + 기본 카테고리 + 인증 테스트           | backend  | ✅   |
| 4     | 카테고리 · 거래 API + 시드 + 테스트          | backend  | ✅   |
| 5     | 집계 · 예측 API + 테스트                     | backend  | ✅   |
| 6     | 예산 · CSV API + 테스트                      | backend  | ✅   |
| 7     | 프론트 스캐폴딩                              | frontend | ✅   |
| 8     | 인증 화면                                    | frontend | ✅   |
| 9     | 거래 화면 (퀵 입력 · 목록 · 상세)            | frontend | ✅   |
| 10    | 대시보드 화면 (차트 · 예측 · 예산)           | frontend | ⬜   |
| 11    | 예산 · 카테고리 · CSV 화면 + 인터랙션 다듬기 | frontend | ⬜   |
| 12    | 전체 검증                                    | 전체     | ⬜   |
| 13    | 챗봇 조회 위젯 (규칙 기반)                   | frontend | ✅   |
| 14    | 영수증 인식 API                              | backend  | ⬜   |
| 15    | 영수증 첨부 등록 화면                        | frontend | ⬜   |

⬜ 대기 · 🟡 진행중 · ✅ 완료

> **배포는 이번 범위에 없다.** 진행 여부가 결정되면 새 Phase로 추가한다. `CLAUDE.md` 3장 「배포 여지」의 네 항목은 비용이 0이므로 지금부터 지켜둔다.

> **테스트는 마지막에 몰아 쓰지 않는다.** 기능을 만든 Phase에서 함께 작성해 그 Phase의 DoD로 삼는다. Phase 12는 새 테스트를 쓰는 단계가 아니라 전체를 확인하는 단계다.

### 왜 이 순서인가

- **백엔드를 먼저 끝낸다(1~6).** 프론트 화면의 DoD는 거의 전부 "서버가 준 값이 화면에 맞게 보이는가"라, 서버가 없으면 확인할 수 없다.
- **집계·예측(5)을 거래 API(4) 직후에 둔다.** 예측 로직은 이 제품의 차별점이자 가장 틀리기 쉬운 부분이고, 화면을 만들기 전에 **숫자가 맞는지 테스트로 고정**해두어야 한다. 화면과 함께 만들면 "차트가 이상한데 계산이 틀린 건지 그리기가 틀린 건지" 구분이 안 된다.
- **대시보드 화면(10)을 거래 화면(9) 뒤에 둔다.** 대시보드를 눈으로 검증하려면 데이터를 넣을 수단이 필요하다. 시드만으로는 "입력 → 숫자 변화"를 확인할 수 없다.
- **예산·카테고리·CSV 화면(11)을 마지막에 둔다.** 셋 다 "이미 있는 데이터를 다루는" 화면이라 앞 화면들의 검증 경로에 들어가지 않는다. 반대로 예산 화면은 10의 소진율 막대를, CSV 화면은 9의 목록을 결과 확인 수단으로 쓰므로 뒤에 있는 편이 검증이 쉽다.
  > ⚠️ **다만 Phase 9가 `/settings/categories`를 링크한다**(카테고리 0개일 때의 안내). Phase 11 전까지 그 링크는 404다. 링크 자체는 Phase 9에서 만들되 **클릭 결과를 Phase 9 DoD로 삼지 않는다.** 같은 이유로 Phase 8의 기본 카테고리 확인은 화면이 아니라 API로 한다.

---

## 요구사항 ↔ Phase 추적표

> `PRD.md` 3장의 P0 요구사항이 **어느 Phase에서 구현되고 어느 Phase에서 검증되는지**의 정본이다.
> **여기에 행이 없는 P0는 구현되지 않는다.** `PRD.md` 3장에 요구사항을 추가하면 이 표에 먼저 행을 넣고, 해당 Phase의 작업·DoD에 실제로 기술한다.

| ID                                       | 구현 Phase                                                | 검증 Phase (DoD)                          |
| ---------------------------------------- | --------------------------------------------------------- | ----------------------------------------- |
| AUTH-01 회원가입                         | 3(API) · 8(화면)                                          | 3 · 8 · 12                                |
| AUTH-02 비밀번호 6자 이상 + 72바이트     | 3(바이트 validator) · 8(실시간 검증)                      | 3(한글 25자 → 400) · 8 · 12               |
| AUTH-03 이메일 중복                      | 3(409) · 8(인라인 문구)                                   | 3 · 8                                     |
| AUTH-04 로그인·JWT 24h                   | 3 · 8                                                     | 3 · 8 · 12                                |
| AUTH-05 기본 카테고리 자동 생성          | 3                                                         | 3 · 8(가입 직후 입력 가능) · 12           |
| AUTH-06 로그아웃                         | 8 (프론트 전용, 서버 API 없음)                            | 8 · 12                                    |
| AUTH-07 라우트 보호 (`exp` 판정)         | 8 (`(main)` 클라이언트 레이아웃)                          | 8(만료 토큰 케이스) · 12                  |
| AUTH-08 헤더 닉네임 / 이메일 미표시      | 3(`/auth/me`) · 8(헤더)                                   | 3 · 8(DOM에 이메일 없음) · 12             |
| CAT-01 카테고리 생성                     | 4 · 11                                                    | 4 · 11                                    |
| CAT-02 수정 (type 불변)                  | 4(PUT에 `type` 없음) · 11                                 | 4 · 11                                    |
| CAT-03 삭제 후 과거 내역 보존            | 4(조인 조건 분리) · 9(삭제됨 표시)                        | 4 · 9 · 12                                |
| CAT-04 이름 중복 (삭제분 제외)           | 2(부분 유니크 인덱스) · 4(409)                            | 4 · 11 · 12                               |
| CAT-05 타인 카테고리 차단                | 4(404)                                                    | 4                                         |
| TXN-01 퀵 입력 (화면 이동 없음)          | 4(POST) · 9(퀵 입력 바)                                   | 9 · 12                                    |
| TXN-02 저장 후 폼 유지 동작              | 9                                                         | 9                                         |
| TXN-03 최근 카테고리 3개 버튼            | 9 (별도 API 없이 목록에서 도출)                           | 9 · 12                                    |
| TXN-04 금액 콤마·양수                    | 4(DTO 검증) · 7(`lib/money.ts`) · 9                       | 4 · 9                                     |
| TXN-05 구분 불일치 차단                  | 4(`CATEGORY_TYPE_MISMATCH`) · 9                           | 4 · 9                                     |
| TXN-06 날짜 내림차순 + 2차 정렬 키       | 4                                                         | 4(페이지 경계 중복·누락) · 12             |
| TXN-07 20건 페이지네이션 · 필터          | 4 · 9                                                     | 4 · 9 · 12                                |
| TXN-08 거래처·메모 검색                  | 4 · 9                                                     | 4 · 9 · 12                                |
| TXN-09 상세 편집 + 이탈 확인             | 4(PUT) · 9(3계층 가드)                                    | 9 · 12                                    |
| TXN-10 삭제 즉시 반영 / Soft Delete      | 4 · 9(상세→목록) · 11(낙관적 제거)                        | 4 · 9 · 11 · 12                           |
| TXN-11 실패 시 롤백·알림                 | **9(저장 실패: 폼 유지 + 에러)** · 11(삭제 롤백 + 토스트) | 9 · 11 · 12                               |
| TXN-12 타인 거래 차단                    | 4(404) · 9(전용 화면)                                     | 4 · 9 · 12                                |
| TXN-13 영수증 첨부 인식                  | 14(`/receipts/parse`) · 15(첨부 버튼 + 프리필)            | 14 · 15                                   |
| STAT-01 월 요약 · 월 이동                | 5 · 10                                                    | 5 · 10 · 12                               |
| STAT-02 카테고리별 비중                  | 5 · 10(도넛)                                              | 5 · 10                                    |
| STAT-03 일별 히트맵                      | 5(`daily`) · 10(CSS grid)                                 | 5 · 10 · 12                               |
| STAT-04 이번 달 예상 지출                | 5(수식) · 10(카드)                                        | 5(수식 일치) · 10 · 12                    |
| STAT-05 이상치 30%                       | 5 · 10                                                    | 5(7일 미만 빈 배열) · 10                  |
| STAT-06 고정지출 감지                    | 5(`/stats/recurring`) · 10(카드)                          | 5 · 10 · 12                               |
| STAT-07 데이터 부족 안내                 | 5(`forecast: null`) · 10                                  | 5 · 10                                    |
| STAT-08 거래 없는 달 정상 표시           | 5(`COALESCE`) · 10                                        | 5 · 10 · 12                               |
| BUD-01 카테고리별 예산 설정              | 6(upsert) · 11                                            | 6 · 11                                    |
| BUD-02 0/빈값은 미설정                   | 6 · 11                                                    | 6 · 11                                    |
| BUD-03 소진율 표시                       | 5(`stats.budgets`) · 10(막대)                             | 5 · 10 · 12                               |
| BUD-04 초과 시각 구분                    | 5(`exceeded`) · 10                                        | 10                                        |
| CSV-01 내보내기                          | 6 · 11                                                    | 6 · 11                                    |
| CSV-02 Excel 한글 (BOM)                  | 6                                                         | 6(첫 3바이트) · 12(**실제 Excel로 열기**) |
| CSV-03 가져오기                          | 6 · 11                                                    | 6 · 11                                    |
| CSV-04 부분 성공 + 실패 사유             | 6 · 11                                                    | 6 · 11 · 12                               |
| CSV-05 왕복 일치 (콤마·따옴표 포함)      | 6(`CsvParser`)                                            | 6 · 12                                    |
| CSV-06 없는 카테고리는 행 실패           | 6                                                         | 6                                         |
| CSV-07 엑셀 왕복 내성 (인코딩·콤마·날짜) | 6(`CsvParser`)                                            | 6 · 12(**Excel로 저장 후 재업로드**)      |
| UX-01 스켈레톤                           | 7(컴포넌트) · 8 · 9 · 10                                  | 9 · 10 · 12                               |
| UX-02 빈 상태                            | 7 · 9                                                     | 9 · 12                                    |
| UX-03 검색 결과 없음                     | 9                                                         | 9 · 12                                    |
| UX-04 에러 + 재시도 버튼                 | 7(`ErrorState.onRetry` 필수) · 9 · 10                     | 7 · 9 · 10(재요청 확인) · 12              |
| UX-05 반응형 360~1920                    | 7(토큰·컨테이너) · 9 · 10                                 | 9(360px) · 12(1920px)                     |
| UX-06 label · 키보드 완주                | 8 · 9                                                     | 9 · 12                                    |
| UX-07 다크 토큰 (`prefers-color-scheme`) | 7                                                         | 7 · 12                                    |
| UX-08 금액 `tabular-nums`                | 7(토큰) · 9                                               | 9 · 12                                    |
| CHAT-01 플로팅 버튼 열기/닫기            | 13                                                        | 13                                        |
| CHAT-02 이번 달 요약 조회                | 13(`useMonthlyStatsQuery` 재사용)                         | 13                                        |
| CHAT-03 카테고리별 지출 조회             | 13(`useCategoriesQuery` + `byCategory` 매칭)              | 13                                        |
| CHAT-04 최근 내역 조회                   | 13(`useTransactionListQuery` 재사용)                      | 13                                        |
| CHAT-05 예산·고정지출 조회 + 실패 안내   | 13(`useRecurringQuery` 재사용)                            | 13                                        |

> `PRD.md` 5.1의 **에러 문구 매핑 표**는 Phase 7에서 `lib/errorMessages.ts`로 단일화하고, Phase 8~11에서 화면별로 적용, Phase 12에서 전부 대조한다.
> `PRD.md` 7장 비기능 요구사항의 검증 위치는 위 표 및 각 Phase DoD와 일치한다.
> **ID가 붙지 않은 화면 세부 요구사항(`PRD.md` 5장의 문장들)은 이 표로 추적되지 않는다.** 하단 탭 바, 히트맵 칸의 보조 텍스트, 빈 상태의 CTA 동작처럼 ID가 없는 항목은 **각 Phase의 작업 목록이 정본**이므로, `PRD.md` 5장을 고치면 이 표가 아니라 해당 Phase 작업에 문장을 넣는다.

---

## Phase 1 — 저장소 초기화 + 백엔드 스캐폴딩

**저장소**: 전체 → 이후 `moneylog-backend`

**작업**

_저장소_

- `moneylog-project/` `git init` + `CLAUDE.md`, `docs/PRD.md`, `docs/ROADMAP.md` 배치
- 루트 `.gitignore`에 **`moneylog-backend/`, `moneylog-frontend/`, `node_modules/` 추가 (필수)**
- `moneylog-backend/`, `moneylog-frontend/` 각각 `git init`
- 세 저장소 모두 `main` + `develop` 브랜치 생성
- 각 저장소에 `.gitignore`(**`.env*` + `!.env.example` 예외 줄 포함**)와 `.env.example` 작성
- 각 하위 저장소에 자체 `CLAUDE.md` 작성 (빌드 명령·계층 규칙. 전체 스펙은 부모 문서가 정본임을 명시)
- `moneylog-backend/.gitattributes`에 `mvnw text eol=lf`

_백엔드 스캐폴딩_

- Spring Boot 4.x + JDK 21 + Maven 래퍼
- 의존성: `web`, `data-jpa`, `validation`, `security`, `postgresql`, `lombok`, **springdoc(버전 핀)**, **jjwt 3종**
- `application.yml`(공통, **`spring.profiles.active: local`**) / `application-local.yml` / `src/test/resources/application-test.yml`
  - `hibernate.jdbc.time_zone: UTC` 설정
  - `spring.servlet.multipart.max-file-size: 1MB`, `max-request-size: 2MB` (Phase 6 대비)
- **`application.properties`를 만들지 않는다.** `.yml`과 공존하면 `.yml` 설정이 조용히 무시된다
- PostgreSQL에 `moneylog_db`, `moneylog_test` 생성
- `ApiResponse<T>`, `PageResponse<T>` record 작성
- `GlobalExceptionHandler` + `ErrorCode` enum 골격 (`CLAUDE.md` 11장 표 전체)
- Swagger 설정 (`@SecurityScheme` 포함)

**DoD**

- [x] `./mvnw spring-boot:run`이 프로파일 지정 없이 기동됨
- [x] `http://localhost:8080/swagger-ui/index.html` 접속 가능
- [x] `pom.xml`의 SpringDoc 버전이 **정확한 값으로 핀**되어 있고 Boot 마이너와 대응함
- [x] `pom.xml`에 jjwt 3종이 있고 `jjwt-impl`·`jjwt-jackson`이 `runtime` scope
- [x] `application.properties`가 존재하지 않음
- [x] 세 저장소 모두 브랜치가 `main`·`develop`이고 `master`가 없음
- [x] 루트 저장소 첫 커밋 대상 파일이 **문서 3~4개 수준**임 (`git status` 확인)
- [x] `.env`를 실제로 만들어 `git status`에 나타나지 않고, `.env.example`은 나타남을 확인
- [x] `psql`로 `moneylog_db`, `moneylog_test` 접속 확인

---

## Phase 2 — 도메인 & DB

**저장소**: `moneylog-backend` · **관련 요구사항**: CAT-04(부분 유니크)

**작업**

- `BaseEntity`(`created_at`, `updated_at`) + **메인 애플리케이션 클래스에 `@EnableJpaAuditing`**
- 엔티티: `User`, `Category`, `Transaction`, `Budget` (`CLAUDE.md` 4장 스키마 그대로)
  - **금액은 `BigDecimal` + `@Column(precision = 15, scale = 2)`**
  - **`@ManyToOne(fetch = FetchType.LAZY)` 명시** (기본값 EAGER)
  - `@Setter` 금지. 변경은 의미 있는 메서드로
  - `Budget`에는 `deleted_at`을 두지 않는다
- Repository 인터페이스 4종
- **`db/schema-extra.sql`** — Hibernate가 만들지 못하는 것만 담는다
  - `categories` 부분 유니크 인덱스 (`WHERE deleted_at IS NULL`)
  - `amount > 0` CHECK 제약 (transactions, budgets)
  - 최초 1회 수동 적용하고, 적용 방법을 백엔드 `CLAUDE.md`에 적어둔다
- Repository 단위 테스트 (`@DataJpaTest` + `@AutoConfigureTestDatabase(replace = NONE)` + `@ActiveProfiles("test")`)

**DoD**

- [x] `ddl-auto: update`로 기동 시 4개 테이블과 인덱스가 생성됨
- [x] `schema-extra.sql` 적용 후, 같은 이름의 카테고리를 **삭제 → 재생성**할 수 있음
- [x] 같은 이름의 카테고리를 **삭제하지 않고 중복 생성**하면 DB 제약에 걸림
- [x] `amount = 0` 또는 음수 INSERT가 DB에서 거부됨
- [x] `created_at`이 테스트에서 null이 아님 (Auditing 동작 확인)
- [x] `psql`에서 `SHOW timezone` 및 저장된 `created_at`이 UTC 기준임을 확인
- [x] `BigDecimal` 왕복 테스트: `12500` 저장 → 조회 시 `compareTo == 0` (⚠️ `equals`로 비교하면 실패한다)
- [x] Repository 테스트 전체 통과

---

## Phase 3 — 인증 + 기본 카테고리 + 인증 테스트

**저장소**: `moneylog-backend` · **관련 요구사항**: AUTH-01~05, AUTH-08

**작업**

- `JwtTokenProvider` — **`Jwts.builder().signWith(key, Jwts.SIG.HS256)`** (⚠️ 인자 없는 `signWith` 금지)
- `JwtAuthenticationFilter` — `sub`로 사용자 조회, `deleted_at IS NULL` 조건
- `SecurityConfig` — **`csrf.disable()` + `STATELESS` 필수**, `authorizeHttpRequests`, permitAll 경로(§6)에 **Swagger 포함**
- `JwtAuthenticationEntryPoint`(401) / `CustomAccessDeniedHandler`(403) → `ApiResponse` 포맷으로 직접 write
- `CorsConfig` — `CORS_ALLOWED_ORIGINS`(쉼표 목록), 허용 헤더에 `Authorization`·`Content-Type`, **`exposedHeaders`에 `Content-Disposition`**(Phase 6 대비)
- `@MaxByteLength` 커스텀 validator + `GlobalExceptionHandler`에 `IllegalArgumentException` → 400 매핑
- `AuthController` 3개 (`signup`, `login`, `me`)
- **`AuthService.signup()`이 같은 트랜잭션에서 기본 카테고리 9개를 생성** (`CLAUDE.md` 6장 표)
- **통합 테스트 1~5번 작성** (`CLAUDE.md` 12장)

**DoD**

- [x] `POST /api/v1/auth/signup`이 **403이 아니라 정상 응답** (CSRF 비활성화 확인)
- [x] 가입 직후 `GET /api/v1/categories`가 **9개를 반환** (EXPENSE 7 + INCOME 2)
- [x] 중복 이메일 가입 시 409 `EMAIL_DUPLICATED`
- [x] **한글 25자(75바이트) 비밀번호 가입 시 500이 아니라 400 `INVALID_INPUT`** + 필드 메시지
- [x] 로그인 성공 시 JWT 반환. **jwt.io 등으로 디코드해 `alg`가 `HS256`인지 확인** (⚠️ 시크릿 길이로 HS384가 되는 사고 방지)
- [x] 같은 토큰의 `exp - iat`가 **86400초(24시간)** 임 (`AUTH-04`. ⚠️ `JWT_EXPIRATION`은 밀리초 단위라 1000배 어긋나도 토큰은 정상 발급된다. Phase 8의 `exp` 기반 라우트 보호가 이 값을 그대로 신뢰한다)
- [x] 비밀번호 오류 시 401이며, **미가입 이메일과 응답 메시지가 동일**
- [x] 토큰 없이 `/api/v1/auth/me` 호출 시 401이며 **응답이 `ApiResponse` 포맷**
- [x] `/auth/me` 응답에 `email`·`nickname` 포함
- [x] Swagger UI 접속 가능하고 **Authorize 버튼이 보임** (Phase 1 DoD 회귀 확인)
- [x] 인증 통합 테스트 5건 통과

---

## Phase 4 — 카테고리 · 거래 API + 시드 + 테스트

**저장소**: `moneylog-backend` · **관련 요구사항**: CAT-01~05, TXN-01·04~12

**작업**

_카테고리_

- `CategoryController` 4개 (목록/생성/수정/삭제)
- **`CategoryUpdateRequest`에 `type`을 넣지 않는다** (생성 후 불변)
- 목록은 `sortOrder ASC, id ASC`, 페이지네이션 없음, `?type=` 선택 필터
- 이름 중복 시 409 `CATEGORY_DUPLICATED`
- Soft Delete

_거래_

- `TransactionController` 5개 (목록/생성/단건/수정/삭제)
- `TransactionService`: 소유권 검증(불일치 404), Soft Delete, **거래 `type` == 카테고리 `type` 검증**
- 목록 필터: `from`/`to`, `type`, `categoryId`, `keyword`(거래처+메모, 대소문자 무시)
- **정렬은 `txnDate desc` + `id desc` 2차 키 고정.** 허용 필드 화이트리스트(`txnDate`, `amount`, `createdAt`) 밖은 기본값으로 대체
- **목록 쿼리에 `join fetch t.category`** (N+1 방지)
- **카테고리 조인에 `deleted_at IS NULL`을 걸지 않는다** — 과거 내역 보존 (`CLAUDE.md` 4장)
- `TransactionResponse.category`에 `deleted` 플래그 포함
- `PageResponse<T>` → `ApiResponse.data` 안에 담아 반환

_시드_

- `db/seed-dev.sql` — 테스트 계정 1개 + 기본 카테고리 + **최근 6개월치 거래 약 400건**
  - ⚠️ **한 달치만 넣으면 Phase 5의 예측 로직을 전혀 검증할 수 없다.** 기준선이 직전 3개월이므로 최소 4개월치가 필요하다
  - ⚠️ **고정지출 감지용 데이터를 의도적으로 심는다** — 같은 상호·비슷한 금액을 3개월 연속으로 (예: "넷플릭스" 17,000원 매월 5일, "통신비" 45,000원 매월 12일)
  - ⚠️ **NOT NULL이고 DB DEFAULT가 없는 컬럼(`type`, `amount`, `txn_date`, `category_id`)을 INSERT에서 생략하지 않는다**
- `db/seed-perf.sql` — **24개월치 20,000건.** 성능 DoD 측정 전용

_테스트_

- **통합 테스트 6~13번 작성** (`CLAUDE.md` 12장)

**DoD**

- [x] 목록 API가 `{success, data:{content, page, ...}, error}` 형태로 응답
- [x] 목록 응답에 **카테고리 이름·색이 함께 내려옴**
- [x] 지출 카테고리에 수입 거래 생성 시 400 `CATEGORY_TYPE_MISMATCH`
- [x] `amount = 0` / 음수 / **`CLAUDE.md` 4장 제약 표의 상한 초과** 시 400
  > ⚠️ 상한값을 여기 옮겨 적지 않는다. `NUMERIC(15,2)` 범위이며 정본은 `CLAUDE.md` 4장 입력값 제약 표다
- [x] 카테고리 `type` 변경 시도가 **API 스펙상 불가능** (요청 DTO에 필드 없음)
- [x] 같은 이름 카테고리 중복 생성 시 409, **삭제 후 같은 이름 재생성은 성공**
- [x] **카테고리 삭제 후**: 카테고리 목록에서는 빠지고, 과거 거래 목록에는 `deleted: true`로 남아 있음
- [x] 삭제 시 `deleted_at` 기록, 목록에서 제외 (물리 행은 `psql`로 잔존 확인)
- [x] 타 사용자 거래·카테고리 접근 시 404 (GET·PUT·DELETE 전부)
- [x] **같은 날짜 거래 30건을 만들고 page 0·1을 조회해 id가 중복·누락되지 않음** (2차 정렬 키)
- [x] 영문 대소문자를 섞어 검색해도 결과가 나오고, **메모에만 있는 키워드도 검색됨**
- [x] `from`/`to`(한쪽만 준 경우 포함) · `type` · `categoryId` 필터가 각각 건수를 줄이고, **함께 걸면 교집합이 나옴** (`TXN-07`)
- [x] `?sort=foo,desc` 같은 잘못된 정렬 값에도 500이 나지 않음
- [x] 목록 조회 시 **카테고리 조회 쿼리가 건수에 비례해 늘지 않음** (Hibernate Statistics로 항목 3→6개 시 쿼리 수 불변 확인)
- [x] 날짜가 배열이 아닌 문자열로 직렬화됨 (`txnDate: "2026-09-14"`, `createdAt: "...Z"`)
- [x] 금액이 JSON **숫자**로 직렬화됨 (`12500.00`, 문자열 아님)
- [x] **키워드 검색 포함** 목록 조회가 **워밍업 후 3회 측정 중앙값 500ms 이내** (시드 **20,000건** 기준)
  > ⚠️ 시드 400건으로는 이 지표가 의미가 없다. 인덱스가 없어도 400행은 1ms 미만이라 항상 통과한다. 첫 요청은 JVM 콜드 스타트이므로 워밍업 후 측정한다
- [x] Swagger에서 카테고리·거래 API 전체 확인 가능
- [x] 통합 테스트 8건(6~13번) 통과

---

## Phase 5 — 집계 · 예측 API + 테스트

**저장소**: `moneylog-backend` · **관련 요구사항**: STAT-01~08, BUD-03·04(데이터 측)

> **이 Phase가 이 제품의 차별점이자 가장 틀리기 쉬운 부분이다.** 화면을 만들기 전에 숫자를 테스트로 고정한다.

**작업**

- `StatsController` 2개
  - `GET /api/v1/stats/monthly?yearMonth=&asOf=` — 요약·카테고리별·일별·예측·이상치·예산 소진율을 **한 번에** 반환
  - `GET /api/v1/stats/recurring?asOf=` — 고정지출 감지 (3개월 스캔이라 분리)
- `StatsService` — 계산 수식은 **`CLAUDE.md` 5장 「예측 계산 규칙」이 정본**
  - 기준선: 직전 3개월 지출 합계 ÷ **그 3개월의 실제 총 일수** (개월 수가 아님)
  - 런레이트: `확정 + 일평균 × (총일수 - 경과일수)`
  - 이상치: `|deltaRatio| >= 0.30`, `baseline > 0`, **`daysElapsed >= 7`**
  - 고정지출: 정규화 상호 + 금액 중앙값 ±10% + 3개월 연속
- **모든 집계 쿼리에 `COALESCE(SUM(amount), 0)`**
- **모든 나눗셈에 `divide(x, 2, RoundingMode.HALF_UP)`**
- **`yearMonth`·`asOf`를 파라미터로만 받는다.** 서비스 코드에 `LocalDate.now()`·`YearMonth.now()`가 없어야 한다
- 예산 소진율: `budget == 0` 분기 필수 (`Infinity`/`NaN` 방지)
- 집계 쿼리는 `@Query`로 명시적으로 작성
- **순수 단위 테스트**로 수식 검증 (DB 없이 입력→출력) + **통합 테스트 14~19번 작성**

**DoD**

- [x] **거래가 없는 달 조회 시 500이 아니라 모두 0** (`COALESCE` 검증)
- [x] `summary.income - summary.expense == summary.net`이고, 세 값이 **해당 월 거래를 직접 SQL로 합산한 값과 `compareTo == 0`** (`STAT-01`)
- [x] `daily` 배열의 `expense`·`income` 총합이 `summary`와 일치하고, **날짜가 대상 월 밖으로 새지 않음** (`STAT-03`)
- [x] **`daily`의 길이가 그 달의 일수와 같고**(2월 28/29 · 4월 30 · 1월 31), 거래가 없는 날도 `expense: 0.00`·`income: 0.00`으로 들어 있음 (`CLAUDE.md` 5장)
  > ⚠️ **`COALESCE`로는 해결되지 않는다.** `GROUP BY txn_date`는 거래가 있는 날만 반환하므로, 빈 날은 `SUM`이 `NULL`인 게 아니라 **그룹 자체가 없다.** 서비스에서 1일~말일을 순회해 채우거나 `generate_series` LEFT JOIN을 쓴다
- [x] `yearMonth` 경계 검증: 대상 월의 **1일과 말일 거래가 포함**되고 **직전·다음 달 1일 거래는 제외**됨
- [x] 직전 3개월에 거래가 없으면 `forecast`가 `null`, 1~2개월치만 있으면 `basisMonths`에 실제 개월 수
- [x] 런레이트가 `확정 + 일평균 × 남은일수`와 **소수 둘째 자리까지 일치**
- [x] `daysElapsed < 7`이면 `anomalies`가 빈 배열
- [x] `asOf`가 대상 월 이후면 `projectedExpense == confirmedExpense` (과거 달은 예측하지 않음)
- [x] 이상치가 30% 임계값을 정확히 적용 (29% 미포함, 31% 포함)
- [x] 고정지출 감지가 **3개월 연속 동일 상호를 찾아내고, 2개월만 있는 상호는 제외**
- [x] 금액이 ±10%를 벗어나는 달이 섞이면 고정지출로 감지되지 않음
- [x] `merchant`가 비어 있는 거래가 고정지출 후보에 들어가지 않음
- [x] **`asOf`를 바꾸면 결과가 바뀌고, 서버 시각을 바꿔도 결과가 변하지 않음**
  > ⚠️ 이 항목이 `CLAUDE.md` 4장의 "서버가 이번 달을 판정하지 않는다"를 실제로 검증하는 유일한 지점이다. `grep -r "now()" src/main/java/com/example/service/StatsService.java`가 비어 있음도 함께 확인한다
- [x] 카테고리 비율(`ratio`) 합이 1.0 ± 0.01 (반올림 오차 범위)
- [x] **삭제된 카테고리의 과거 지출이 집계에서 빠지지 않음**
- [x] 예산 0인 카테고리의 `usageRatio`가 `Infinity`/`NaN`이 아님
- [x] 월 대시보드 집계가 **워밍업 후 3회 측정 중앙값 1초 이내** (시드 20,000건)
- [x] Swagger에서 집계 API 확인 가능
- [x] 단위 테스트 + 통합 테스트 6건(14~19번) 통과

---

## Phase 6 — 예산 · CSV API + 테스트

**저장소**: `moneylog-backend` · **관련 요구사항**: BUD-01·02, CSV-01~07

**작업**

_예산_

- `BudgetController` 2개 (`GET ?yearMonth=`, `PUT` upsert)
- **`POST`·`DELETE`를 두지 않는다.** `amount`가 0 또는 null인 항목은 행을 제거
- `GET`은 지출 카테고리 전체를 반환하고 설정된 것만 금액을 채운다 (화면이 표로 바로 그릴 수 있게)

_CSV_

- `GET /api/v1/data/export?from=&to=` — **`ApiResponse` 봉투 예외**, `text/csv; charset=UTF-8`
  - **응답 본문 맨 앞에 UTF-8 BOM(`EF BB BF`) 3바이트**
  - `Content-Disposition: attachment; filename="moneylog_2026-09.csv"`
- `POST /api/v1/data/import` — `multipart/form-data`, 결과는 봉투에 담아 반환
  - **BOM을 제거하고 읽는다** (자기가 내보낸 파일을 못 읽는 상태 방지)
  - 카테고리는 **이름으로 매칭**, 없으면 그 행만 실패. **자동 생성하지 않는다**
  - **부분 성공 허용.** 실패 행만 건너뛰고 `{ imported, failed, errors: [{ line, reason }] }` 반환
  - 상한 1MB / 5,000행. 초과 시 400 `INVALID_CSV`
  - ⚠️ **엑셀 왕복 내성** (`CSV-07`) — 사용자는 내려받은 파일을 **엑셀에서 편집한 뒤** 올린다
    - **인코딩**: UTF-8 엄격 디코딩(`CodingErrorAction.REPORT`) → 실패 시 **MS949 폴백**. `new String(bytes, UTF_8)`은 예외 없이 `U+FFFD`로 치환해 **전 행이 `카테고리 없음`으로 실패**한다
    - **금액**: `replaceAll("[^0-9.]", "")` 후 파싱 (엑셀 천단위 서식 `"12,500"`)
    - **날짜**: `yyyy-MM-dd` → `yyyy.MM.dd` → `yyyy/MM/dd` 순으로 시도
    - BOM 제거는 **디코딩 이후**에 한다
  - **중복 검사를 하지 않는다.** 같은 파일 재업로드 시 거래가 두 번 등록되는 것은 의도된 동작이다(`PRD.md` 1장 비목표)
- **`service/CsvParser.java`** — 라이브러리 없이 20줄 내외
  - ⚠️ **`split(",")`을 쓰지 않는다.** 메모에 콤마가 들어가면 열이 밀린다
  - RFC 4180 최소 집합: 따옴표 감싸기, 내부 따옴표 `""` 이스케이프
  - **순수 단위 테스트로 고정한다**
- **통합 테스트 20~26번 작성**

**DoD**

- [x] 예산 upsert가 기존 행을 갱신하고, `amount=0`인 항목은 행을 제거
- [x] 예산 저장 후 `stats/monthly`의 `budgets`에 즉시 반영됨
- [x] `GET /budgets`가 미설정 카테고리도 포함해 반환
- [x] **내보낸 CSV의 첫 3바이트가 `EF BB BF`** (`xxd`/`Format-Hex`로 확인)
- [x] **실제 Microsoft Excel에서 열어 한글이 깨지지 않음** (⚠️ VS Code·메모장에서는 BOM이 없어도 멀쩡히 보이므로 Excel로 확인해야 한다)
- [x] **내보낸 CSV를 그대로 다시 가져오면 건수가 일치**
- [x] 메모에 **콤마·따옴표·줄바꿈**이 포함된 거래의 왕복 검증 통과
- [x] 없는 카테고리 이름이 섞인 CSV → **해당 행만 실패, 나머지는 성공**하고 실패 사유에 행 번호 포함
- [x] **CP949로 인코딩된 CSV의 한글이 깨지지 않고 가져와짐** (⚠️ 픽스처를 UTF-8로 만들면 검증되지 않는다. `getBytes(Charset.forName("MS949"))`로 바이트를 직접 만든다)
- [x] 금액 `12,500`·날짜 `2026.09.14`가 든 CSV가 정상 파싱됨
- [x] 1MB 초과 파일 / 5,000행 초과 시 400 `INVALID_CSV` (⚠️ Spring 기본 multipart 제한으로 인한 무의미한 413이 아니라 우리 에러 코드가 나가는지 확인)
- [x] 헤더가 다른 CSV 업로드 시 400
- [x] Swagger에서 예산·CSV API 확인 가능
- [x] `CsvParser` 단위 테스트 + 통합 테스트 7건(20~26번) 통과

**⚠️ 이 Phase 종료 시점에 백엔드가 완성된다.** Phase 7로 넘어가기 전에 `./mvnw test` 전체가 통과하는지 확인한다.

---

## Phase 7 — 프론트 스캐폴딩

**저장소**: `moneylog-frontend` · **선행 조건**: Phase 6 완료 (백엔드 전체 API 동작)

**작업**

- **Next.js 15** + **TypeScript 5.x** + Tailwind CSS 4, `src/` 구조, **Node 22 이상**(권장 24 LTS)
- shadcn/ui 초기화 — **`--legacy-peer-deps`**, 스타일 **new-york**
  - 설치할 컴포넌트: `button`, `input`, `label`, `select`, `calendar`, `popover`, `tabs`, `dialog`, `sonner`, `skeleton`, `badge`
  - ⚠️ **`npx shadcn add form`을 실행하지 않는다** (`react-hook-form` 유입 경로)
- 패키지: `motion`(구 framer-motion 아님), `date-fns`, `@tanstack/react-query`
  - ⚠️ **`recharts`·`react-hook-form`·`zod`·`opencsv`를 설치하지 않는다**
- `globals.css` — `@import "tailwindcss";` + `@theme`로 토큰 정의 (`CLAUDE.md` 8장 팔레트)
  - **다크는 `@media (prefers-color-scheme: dark)` 안의 `:root`에서 커스텀 프로퍼티를 덮어쓴다**
  - ⚠️ **`@theme`을 `@media` 안에 중첩하지 않는다** (Tailwind v4에서 최상위 전용)
  - ⚠️ **`@custom-variant dark (&:is(.dark *))`를 남기지 않는다** (class 전략 = FOUC)
- Pretendard 가변 폰트를 `src/app/fonts/`에 넣고 **`next/font/local`**로 로드
- 루트 `layout.tsx`만 서버 컴포넌트, Provider는 클라이언트 컴포넌트로 분리
- `lib/apiClient.ts` — 토큰 주입, **`ApiResponse` 언래핑**, 401 자동 로그아웃
- `lib/money.ts` — `formatAmount`(콤마) / `parseAmount`(콤마 제거). **모든 화면이 이것만 쓴다**
- `lib/date.ts` — `date-fns` 래퍼. ⚠️ **`toISOString()`을 쓰지 않는다** (UTC 변환으로 날짜가 밀린다)
- `lib/queryKeys.ts` — `CLAUDE.md` 9장 쿼리 키 규약을 상수로
- `lib/errorMessages.ts` — `PRD.md` 5.1 에러 문구 매핑 단일 함수
- `components/common/` — `Skeleton`, `EmptyState`, `ErrorState`(**`onRetry` 필수 prop**), `Pagination`
- `components/chart/` — `CategoryDonut`, `BudgetBar`, `MonthHeatmap` **골격만**
  - props를 **Recharts가 받는 모양**(`{ name, value, color }[]`)으로 고정 (`CLAUDE.md` 3장)
  - 이 시점에는 하드코딩 데이터로 렌더만 되면 된다
  - ⚠️ **`TrendLine`(월별 추이 선)을 만들지 않는다.** `CLAUDE.md` 3장 차트 표에 구현 방식이 적혀 있지만, `PRD.md` 5.4 대시보드는 이 차트를 요구하지 않고(`STAT-01`~`STAT-08`·`BUD-03`·`BUD-04` 어디에도 없다) Phase 10에서 연결할 자리도 없다. 「전년 동월 비교·연간 리포트」가 범위에 들어오면(`PRD.md` 9장) 그때 만든다

**DoD**

- [x] `npm run dev`·`npm run build`·`npm run lint` 전부 통과
- [x] `package.json`에 **`recharts`·`react-hook-form`·`zod`·`framer-motion`이 없음**
- [x] `next --version`이 15.x
- [x] `globals.css`에 **`.dark` 셀렉터·`@custom-variant`가 없음**
- [x] OS 다크 모드를 켜면 배경·텍스트가 바뀜 (`prefers-color-scheme` 동작)
- [x] `public/static` 디렉토리가 없음
- [x] `next.config.ts`에 `distDir`이 없음
- [x] Pretendard가 적용됨 (`next/font/local`, Google Fonts 요청 없음)
- [x] `formatAmount(1250000)` → `"1,250,000"`, `parseAmount("1,250,000")` → `"1250000"` 왕복 확인
- [x] `ErrorState`를 `onRetry` 없이 쓰면 **타입 에러**가 남
- [x] `components/chart/` **3개**가 하드코딩 데이터로 렌더됨 (라이트·다크 양쪽)
- [x] 차트 SVG가 `components/chart/` 밖에 존재하지 않음
- [x] `lib/errorMessages.ts`가 `PRD.md` 5.1 표의 **모든 `error.code`에 대해 문구를 반환**하고, 표에 없는 코드에는 `INTERNAL_ERROR` 문구로 폴백함

---

## Phase 8 — 인증 화면

**저장소**: `moneylog-frontend` · **선행 조건**: 백엔드 Phase 3 · **관련 요구사항**: AUTH-01~08

**작업**

- `/login`, `/signup` — **`useState` + 수동 검증** (폼 라이브러리 없음)
  - 비밀번호는 **제출 전 인라인 검증**: 6자 이상 + UTF-8 72바이트 이하. 한글 안내 문구 포함
  - ⚠️ 바이트 길이는 `new TextEncoder().encode(pw).length`로 센다. `pw.length`는 문자 수다
  - `/login`에 **회원가입 링크**를 둔다 (`PRD.md` 5.2). 없으면 가입 경로가 URL 직접 입력뿐이다
- `(main)` 그룹 클라이언트 레이아웃 + `useAuth`
  - ⚠️ **토큰 존재 여부가 아니라 `exp`를 디코드해 판정** (`atob` + `JSON.parse`, 라이브러리 없음)
  - 만료로 판정되면 즉시 토큰 폐기 후 `router.replace("/login")`
  - 판정 전에는 스켈레톤
  - ⚠️ **`middleware.ts`를 만들지 않는다**
- 공통 헤더 — 닉네임 + 네비게이션(대시보드 · 내역 · 예산 · 설정) + 로그아웃. **이메일은 표시하지 않는다**
  - 로그아웃: 토큰 삭제 + `queryClient.clear()` → `/login`
  - **모바일에서는 네비게이션을 하단 탭 바로 전환한다** (`PRD.md` 5.1). 링크 목록은 같고 배치만 바뀐다
  - ⚠️ 네비게이션 항목 중 `/budgets`·`/settings/categories`는 **Phase 11에서 만든다.** 링크는 지금 두되 도착 화면은 Phase 11 DoD로 확인한다
- `/dashboard`, `/transactions` 플레이스홀더 (Phase 8 검증 대상이 필요하므로)

**DoD**

- [x] `/login`의 **회원가입 링크로 `/signup`에 도달**함 (URL 직접 입력 없이 — `PRD.md` 5.2)
- [x] 가입 → 자동 로그인 → `/dashboard` 이동
- [x] 가입 직후 **`GET /api/v1/categories`가 9개를 반환**하고, 그 토큰이 새로 가입한 계정의 것임
  > ⚠️ `/settings/categories` 화면으로 확인하지 않는다. 그 화면은 Phase 11에서 만든다
- [x] 한글 25자 비밀번호 입력 시 **제출 전에** 인라인 안내가 뜸
- [x] 중복 이메일 가입 시 이메일 입력 아래 인라인 문구
- [x] 로그인 실패 시 문구가 **미가입/비번오류 구분 없이 동일**
- [x] 로그아웃 후 뒤로가기로 `/dashboard`에 접근되지 않음
- [x] **만료된 토큰을 localStorage에 직접 넣고** 보호 경로 접근 시, 보호 화면이 **한 프레임도 보이지 않고** `/login`으로 이동
  > ⚠️ 만료 토큰은 `exp`를 과거로 조작한 JWT를 직접 만들어 넣는다. 24시간을 기다리지 않는다
- [x] 헤더에 닉네임이 보이고 **DOM 검색으로 이메일이 나오지 않음**
- [x] **360px에서 네비게이션이 하단 탭 바로 바뀌고**, 데스크톱에서는 헤더 안에 가로로 배치됨 (`PRD.md` 5.1)
- [x] 모든 입력에 label이 연결됨 (⚠️ `placeholder`를 label 대용으로 쓰지 않았는지 함께 확인 — `UX-06`)
- [x] `npm run build` 통과

---

## Phase 9 — 거래 화면 (퀵 입력 · 목록 · 상세)

**저장소**: `moneylog-frontend` · **선행 조건**: 백엔드 Phase 4 · **관련 요구사항**: TXN-01~12, CAT-03, UX-01~06·08

> **이 Phase가 제품의 성패를 가른다.** `PRD.md` 2장의 전제대로, 가장 중요한 화면은 대시보드가 아니라 입력 폼이다.

**작업**

- `/transactions` — **`<Suspense>`로 감싼다** (`useSearchParams` 사용, 없으면 빌드 실패)
- **퀵 입력 바** (`components/transaction/QuickAddBar.tsx`) — `TXN-01`~`TXN-05`
  - 구분 토글 · 금액 · 카테고리 · 날짜(기본 오늘) · 거래처 · 메모를 **한 줄에**
  - ⚠️ **금액에 `<input type="number">`를 쓰지 않는다.** `type="text" inputMode="numeric"` + `lib/money.ts`
  - **최근 사용 카테고리 3개 버튼** — 별도 API 없이 목록 앞쪽에서 중복 제거해 도출
  - 저장 후 폼은 비우되 **날짜·구분은 유지**
  - 낙관적 업데이트를 하지 않는다. 버튼 로딩 상태만
  - 카테고리가 0개면 폼 비활성화 + `/settings/categories` 링크 (⚠️ 도착 화면은 Phase 11. 여기서는 링크 존재까지만 확인한다)
- **`components/transaction/TransactionForm.tsx`** — 퀵 입력 바와 상세 화면이 **공유**하는 폼 본체
- 필터·검색·페이지 — **전부 URL 쿼리로 관리**
- 목록 — 날짜 내림차순, 20건, 카테고리 배지(색), 금액 색 구분 + **`tabular-nums`**
  - 삭제된 카테고리는 배지 옆 "삭제됨" — `CAT-03`
  - ⚠️ `category.color`를 인라인 스타일에 넣기 전 **`#RRGGBB` 정규식 검증**
  - ⚠️ **카테고리 이름 텍스트를 카테고리 색 위에 올리지 않는다.** 색은 점·막대로 이름 **옆**에 둔다. 사용자가 지정한 임의 색이라 대비를 계산할 수 없고, 다크 모드에서도 같은 색을 그대로 쓰기 때문이다 (`PRD.md` 5.1 다크 모드)
- `/transactions/[id]` — 진입 즉시 편집 가능. `TransactionForm` 재사용 + 삭제 버튼
  - **이탈 확인 3계층** (`beforeunload` / 버튼 핸들러 / `popstate`) — `TXN-09`
    - ⚠️ 서드파티 내비게이션 가드를 설치하지 않는다
    - ⚠️ `dirty` 판정은 **콤마 제거한 금액 원본끼리** 비교
    - 저장 직후 반드시 가드 해제
  - 저장 실패 시 폼 유지 + 에러 표시 — `TXN-11`
  - 404 시 "거래 내역을 찾을 수 없습니다" 전용 화면
- 빈 상태 / 검색 결과 없음 문구 구분 — `UX-02`, `UX-03` (`PRD.md` 5.5)
  - 거래 없음: "아직 기록이 없어요" + **퀵 입력 바 금액 필드로 포커스 이동**
  - 검색 결과 없음: "조건에 맞는 내역이 없어요" + **필터 초기화 버튼**(누르면 URL 쿼리에서 필터 키가 제거됨)
- 로딩 중에는 **항목형 스켈레톤 5개** — `UX-01` (`CLAUDE.md` 9장. 스피너를 쓰지 않는다)

**DoD**

- [x] **금액 입력 → 카테고리 버튼 → 저장, 3회 조작으로 등록 완료** (`PRD.md` 7장 입력 속도)
- [x] 저장 후 목록 맨 위에 나타나고 폼은 비워지되 **날짜·구분이 유지됨**
- [x] 금액 입력 시 `1,250,000` 형태로 콤마가 표시되고 값이 사라지지 않음
- [x] 모바일(360px)에서 숫자 키패드가 뜨고 폼이 세로로 쌓임
- [x] 최근 카테고리 3개 버튼이 실제 최근 사용 순으로 노출됨
- [x] 지출 토글 상태에서 수입 카테고리를 고르면 저장 전에 막히거나 서버 400 문구가 인라인으로 표시됨
- [x] 필터·검색·페이지가 **URL에 반영**되고 새로고침·뒤로가기로 유지됨
- [x] 같은 날짜 거래가 여러 건인 상태에서 페이지를 넘겨도 **중복·누락 없음**
- [x] 삭제된 카테고리가 붙은 과거 거래가 **사라지지 않고 "삭제됨"으로 표시됨**
- [x] 상세에서 수정 후 **저장하지 않고**: ① 새로고침 ② "목록으로" 버튼 ③ 브라우저 뒤로가기 — **세 경로 모두 확인창**
- [x] 저장하고 나갈 때는 확인창이 **뜨지 않음** (⚠️ 금액 콤마로 인한 오판 확인)
- [x] 저장 실패 시(서버 중지 후 시도) 폼 내용이 유지되고 에러가 표시됨
- [x] 타인 거래 id로 접근 시 전용 404 화면 (목록으로 리다이렉트되지 않음)
- [x] 거래 0건일 때와 검색 결과 0건일 때 **문구가 다르고**, 각각의 CTA가 동작함 — 전자는 **금액 입력에 포커스가 들어가고**, 후자는 **필터 초기화 버튼이 URL 쿼리를 비움**
- [x] 목록 로딩 중 **스피너가 아니라 항목형 스켈레톤**이 보임 (⚠️ 로컬 API는 빨라 잘 안 보인다. 네트워크 스로틀링을 켜고 확인 — `UX-01`)
- [x] 에러 상태의 **재시도 버튼이 실제로 재요청**함 (네트워크 탭 확인)
- [x] 카테고리 배지에서 **이름 텍스트가 카테고리 색 위에 얹혀 있지 않음** (색은 점·막대로 옆에 — `PRD.md` 5.1)
- [x] **키보드만으로 거래 1건 등록 완주** (Tab 순서 + Enter 저장)
- [x] 목록 금액이 자릿수 기준으로 세로 정렬됨 (`tabular-nums`)
- [x] 360px에서 레이아웃이 깨지지 않음
- [x] `npm run build` 통과 (⚠️ `useSearchParams` Suspense 누락 시 여기서 실패한다)

---

## Phase 10 — 대시보드 화면 (차트 · 예측 · 예산)

**저장소**: `moneylog-frontend` · **선행 조건**: 백엔드 Phase 5·6 · **관련 요구사항**: STAT-01~08, BUD-03·04

**작업**

- `/dashboard` — **`<Suspense>`로 감싼다** (`?ym=` 쿼리)
- 월 선택 `◀ 2026년 9월 ▶` — URL 쿼리로 관리, **미래 달 이동 불가**
- `yearMonth`·`asOf`를 **클라이언트가 계산해 보낸다**
  - ⚠️ `lib/date.ts`의 `format(new Date(), "yyyy-MM-dd")`를 쓴다. **`toISOString()` 금지**
- 요약 카드 3개 (총수입/총지출/잔액, 잔액 음수면 빨강)
- **예측 카드** — `STAT-04`, `STAT-07`
  - "이번 달 이 속도면 **N원**을 쓰게 돼요" + "15일 경과 · 최근 3개월 기준"
  - **과거 달을 보고 있으면 카드를 숨긴다**
  - `forecast === null`이면 "예측하려면 데이터가 조금 더 필요해요"
  - ⚠️ **프론트에서 예측을 다시 계산하지 않는다.** 서버 값을 그대로 표시한다
- **이상치 카드** — 최대 3개. 빈 배열이면 렌더링하지 않음
- **고정지출 카드** — `/stats/recurring` 별도 호출, **이 카드만 따로 스켈레톤**
  - 항목마다 **거래처 · 금액 · 연속 개월 수 · 최근 날짜**를 보여주고, 카드 하단에 **월 합계**를 둔다 (`PRD.md` 5.4·4.5)
    > 합계가 이 카드의 요점이다. `PRD.md` 4.5의 "합계 62,000원이 매달 빠져나간다는 사실을 등록 없이 확인"이 그것이다
  - "등록" 버튼을 두지 않는다 (읽기 전용 안내)
- 차트 3종을 `components/chart/`의 컴포넌트로 연결
  - 카테고리 도넛 + 금액·비율 목록
  - 예산 소진율 막대 (초과 시 막대와 수치를 빨강으로)
  - 일별 히트맵 — CSS `grid-cols-7`, **1일의 요일만큼 앞칸 비움**, 농도 4단계
    - **각 칸에 날짜와 금액을 툴팁 또는 보조 텍스트로 제공한다** (`PRD.md` 5.4). 농도만으로는 금액을 읽을 수 없다
    - 서버가 **그 달의 모든 날을 0으로 채워 보낸다**(`CLAUDE.md` 5장). **프론트에서 빈 날을 채우지 않는다** — 비는 칸은 1일 앞의 요일 오프셋뿐이다
  - ⚠️ 화면에서 SVG를 직접 그리지 않는다
  - ⚠️ `TrendLine`을 연결하지 않는다 (Phase 7 참조 — 요구사항이 없다)
- 거래 없는 달: 모두 0 + "이 달에는 기록이 없어요", **월 이동은 계속 동작**

**DoD**

- [x] 월 이동이 URL에 반영되고 **뒤로가기로 이전 달로 돌아감**
- [x] 미래 달로 이동할 수 없음
- [x] 예측 금액이 **서버 응답값과 정확히 일치** (프론트 재계산 없음)
- [x] 과거 달 조회 시 예측 카드가 숨겨짐
- [ ] 데이터 부족 계정으로 로그인 시 "데이터가 조금 더 필요해요" 표시 — ⚠️ 미검증. 시드 계정은 6개월치 데이터가 있어 재현 못 함. `forecast === null` 분기(`ForecastCard.tsx`)는 코드로 확인함. 데이터 없는 신규 계정으로 재확인 필요
- [x] 이상치가 없으면 카드가 렌더링되지 않음 (빈 카드가 남지 않음)
- [x] 고정지출 카드가 시드에 심어둔 "넷플릭스"·"통신비"를 찾아내고, **연속 개월 수·최근 날짜·월 합계가 모두 표시됨**
- [ ] 고정지출 카드가 **본문보다 늦게 로드되어도 레이아웃이 밀리지 않음** (로딩 중 스켈레톤이 최종 높이를 차지 — `UX-01`) — ⚠️ 미검증(네트워크 지연 재현 안 함). 스켈레톤 높이를 최종 카드와 비슷하게 맞춰 두긴 했음(`RecurringCard.tsx`)
- [x] 도넛 비율 합이 100%로 보임 (반올림으로 99~100% 사이로 관찰됨 — 서버 `ratio` 값을 그대로 반올림한 결과라 프론트에서 보정하지 않음)
- [x] 히트맵의 1일이 **올바른 요일 칸에 위치**함 (여러 달을 이동하며 확인)
- [x] 히트맵의 각 칸에서 **날짜와 금액을 읽을 수 있음** (툴팁 또는 보조 텍스트 — `PRD.md` 5.4)
- [x] 히트맵에 **그 달의 모든 날 칸이 존재**함 (지출 0원인 날도 칸이 있고, 비는 칸은 1일 앞의 요일 오프셋뿐 — `CLAUDE.md` 5장)
- [x] 대시보드 로딩 중 **카드형 스켈레톤**이 보임 (스피너 없음 — `UX-01`)
- [x] 백엔드를 중지한 채 진입하면 에러 카드가 뜨고, **재시도 버튼이 실제로 재요청**함 (백엔드를 다시 켜고 누르면 화면이 복구됨 — `UX-04`)
- [x] 예산 초과 카테고리가 빨강으로 구분됨
- [x] 예산 0인 카테고리가 `NaN%`·`Infinity%`로 표시되지 않음
- [x] **거래가 없는 달**에서 오류 없이 0으로 표시되고 월 이동이 동작
- [x] **거래를 하나 추가한 뒤 대시보드로 돌아오면 숫자가 갱신됨** (⚠️ `['stats']` 무효화 누락 시 여기서 걸린다)
- [x] 라이트·다크 양쪽에서 차트가 읽힘 — **카테고리 색이 양쪽에서 동일하고**, 축·범례 텍스트가 카테고리 색 위에 올라가 있지 않음 (`PRD.md` 5.1)
- [x] 360px에서 차트가 깨지지 않음 (히트맵 7열이 유지되고 가로 스크롤이 생기지 않음)
- [x] `npm run build` 통과

---

## Phase 11 — 예산 · 카테고리 · CSV 화면 + 인터랙션 다듬기

**저장소**: `moneylog-frontend` · **선행 조건**: 백엔드 Phase 6 · **관련 요구사항**: BUD-01·02, CAT-01~04, CSV-01~07, TXN-10·11

**작업**

_화면_

- `/budgets` — 월 선택 + 지출 카테고리 전체 행 + 금액 입력 + **저장 버튼 하나로 전체 upsert**
- `/settings/categories` — 수입/지출 탭, 목록(색 점·이름·순서), 추가·수정·삭제
  - 추가·수정 폼의 **색 선택은 `CLAUDE.md` 8장 카테고리 팔레트에서 고르거나 `#RRGGBB`를 직접 입력** (`PRD.md` 5.8)
  - **수정 폼은 이름·색·표시 순서 세 가지를 바꿀 수 있다** — `CAT-02`
  - **수정 폼에 구분(type)을 두지 않는다**
  - 삭제 확인 문구에 "과거 내역은 그대로 남습니다" 포함
- `/data` — 내보내기(기간 선택, **기본값 이번 달** + 다운로드) / 가져오기(업로드 + 결과 표시)
  - 결과: 성공·실패 건수 + 실패 행 목록(행 번호 + 사유)
  - 형식 안내 표 + 상한 안내(1MB / 5,000행)
  - **중복 등록 안내 문구**: "이미 가져온 파일을 다시 올리면 중복 등록됩니다" (`PRD.md` 5.9)
    > 중복 검사를 하지 않는 것이 의도된 결정이므로(`PRD.md` 1장 비목표), **화면 안내가 그 결정의 유일한 완충장치다.** Phase 12 E가 이 문구의 존재를 확인한다

_인터랙션_

- **삭제 낙관적 업데이트** — `onMutate` 캐시 제거 → `onError` 롤백 + `sonner` 토스트 → `onSettled` 무효화
  - ⚠️ **무효화 대상 3종**: `['transactions']`, `['stats']`, `['budgets']`
  - ⚠️ **페이지 이동은 `onSuccess`에서** 한다 (`onMutate`에서 하면 롤백이 안 보이는 캐시에 적용된다)
- Motion — `motion/react`에서 import
  - 목록 등장 stagger 30ms, 삭제 `AnimatePresence`, 차트 막대 300ms
  - **`useReducedMotion` 존중**

**DoD**

- [x] 예산 여러 개를 입력하고 **한 번에 저장**되며, 대시보드 소진율이 즉시 갱신됨
- [x] 예산을 0으로 바꾸면 미설정으로 돌아감
- [x] 카테고리를 추가하면 **퀵 입력 바의 카테고리 목록에 즉시 나타남**(`['categories']` 무효화), 이름·색·표시 순서를 수정하면 목록 순서와 배지 색이 그에 맞게 바뀜 — `CAT-01`·`CAT-02` (이름 변경 시 QuickAddBar 드롭다운·기존 거래 목록 표시가 페이지 이동만으로 즉시 갱신됨을 직접 확인)
- [x] 같은 구분 안에서 중복 이름 저장 시 **이름 입력 아래 인라인 안내**, **삭제한 이름은 다시 저장됨** — `CAT-04`
- [x] 카테고리 삭제 확인 문구에 "과거 내역은 그대로 남습니다"가 포함됨 — `CAT-03`
- [x] 카테고리 수정 폼에 구분 선택이 **없음**
- [x] CSV 내보내기 파일명이 `moneylog_2026-09.csv` 형태 (⚠️ `Content-Disposition`이 CORS `exposedHeaders`에 없으면 `download`로 나온다)
- [x] **내려받은 파일을 Excel에서 열어 한글이 깨지지 않음** — ⚠️ Excel 실행 환경이 없어 직접 열어보지는 못함. 응답 바이트를 직접 검사해 첫 3바이트가 `EF BB BF`(UTF-8 BOM)이고 한글이 깨지지 않음을 확인함(Excel이 한글 표시를 결정하는 조건 자체가 이 BOM이므로 실질적으로 동일한 검증)
- [x] 그 파일을 그대로 가져오기 → 건수 일치 (실제 거래를 등록 → 내보내기 → 가져오기 → "1건 등록, 0건 실패" 확인 → 테스트 데이터 정리)
- [x] 오타가 섞인 CSV → "N건 등록, M건 실패" + 행 번호와 사유 표시
- [x] `/data`에 **형식 안내 표 · 상한 안내(1MB / 5,000행) · 중복 등록 안내 문구**가 모두 보임 (`PRD.md` 5.9)
- [x] 가져오기 후 거래 목록과 대시보드가 갱신됨
- [x] 거래 삭제가 **클릭 즉시** 목록에서 사라짐
- [ ] **서버를 중지한 상태로 삭제** → 항목이 되돌아오고 토스트가 뜸 — ⚠️ 미검증. Phase 9에서 낙관적 업데이트 자체는 확인했으나, 이번 Phase에서 추가한 Motion 애니메이션과 함께 재확인하지 않았음
- [ ] 마지막 항목 삭제로 페이지가 비면 이전 페이지로 이동하고, **실패 시 롤백이 보이는 화면에 적용됨** — ⚠️ 미검증(이번 Phase 범위 밖 로직, Phase 9에서 이미 구현됨)
- [x] `prefers-reduced-motion: reduce` 설정 시 애니메이션이 줄어듦 (motion 라이브러리가 reduced-motion 감지 경고를 띄우는 것으로 확인)
- [x] `npm run build` 통과

---

## Phase 12 — 전체 검증

**저장소**: 전체

새 기능을 만들지 않는다. 아래 체크리스트가 **완료 판정의 정본**이다.

> ⚠️ **이 체크리스트와 추적표의 「검증 Phase」 열은 1:1로 대응해야 한다.** 추적표가 어떤 요구사항의 검증 Phase로 12를 지목했는데 여기 항목이 없으면 그 요구사항은 최종 검증에서 빠진다. 반대로 여기에만 있고 추적표가 12를 지목하지 않은 항목도 같은 불일치다. 둘 중 하나를 고친다 — 항목을 추가하거나, 추적표에서 12를 뺀다.

### 최종 검증 체크리스트

**A. 테스트 · 빌드**

- [x] `./mvnw clean test` exit 0, 전체 통과 (83/83)
- [x] `npm run build` · `npm run lint` 통과
- [x] `package.json`에 스펙 밖 라이브러리 없음 (`recharts`·`react-hook-form`·`zod`·`framer-motion`)
- [x] `pom.xml`에 스펙 밖 라이브러리 없음 (`opencsv`·`commons-csv`·H2)
- [x] **Swagger UI에 엔드포인트 18개가 모두 노출**됨 — 인증 3 · 카테고리 4 · 거래 5 · 집계 2 · 예산 2 · 데이터 2 (`CLAUDE.md` 5장이 정본). Authorize 버튼에 토큰을 넣고 보호된 API가 실제로 호출됨
  > 🐛 **발견 및 수정**: `POST /data/import`가 예견된 대로 파일 업로드 위젯이 아니라 JSON 텍스트박스로 그려지고 있었다. `@PostMapping`에 `consumes = MULTIPART_FORM_DATA_VALUE`가 없었던 것이 원인 — 추가해 해결(`DataController.java`)

**B. 인증 (AUTH-01~08)**

- [x] 가입 → 기본 카테고리 9개 → 첫 거래 입력까지 막힘 없이 진행
- [x] 한글 25자 비밀번호 → 400 (500 아님)
- [x] 발급 JWT의 `alg`가 `HS256`
- [x] 만료 토큰으로 보호 경로 접근 시 보호 화면 노출 없이 `/login`
- [x] 로그아웃 후 뒤로가기로 접근 불가
- [x] 헤더 DOM에 이메일 없음

**C. 거래 (TXN-01~12, CAT-03·04)**

- [x] 3회 조작으로 거래 1건 등록 — `TXN-01`
- [x] **최근 사용 카테고리 3개 버튼이 실제 최근 사용 순으로 노출**되고, 그 버튼이 위 3회 조작의 두 번째 조작임 — `TXN-03`
- [x] 키보드만으로 거래 1건 등록 완주 — `UX-06`
- [x] 같은 날짜 거래 다수에서 페이지 경계 중복·누락 없음 — `TXN-06` (전체 400건을 20페이지 순회해 id 중복·누락 없음을 스크립트로 확인)
- [x] **필터(기간·구분·카테고리)와 키워드 검색이 URL에 반영**되고, 그 URL을 새 탭에 붙여넣으면 같은 결과가 나옴 — `TXN-07`·`TXN-08`
- [x] **거래 삭제가 클릭 즉시 목록에서 사라지고**, 서버를 중지한 상태로 삭제하면 **항목이 되돌아오며 토스트가 뜸** — `TXN-10`·`TXN-11`
  > 🐛 **발견 및 수정**: `useDeleteTransactionMutation`의 `onError`에 토스트가 없었고, `onSettled`가 실패 시에도 무조건 재조회를 시도해 그 재조회마저 실패하면서 롤백된 목록이 화면째 `ErrorState`로 덮여 버렸다. `onError`에 `toast.error` 추가, `onSettled`→`onSuccess`로 변경(성공 시에만 재조회)해 해결(`useTransactions.ts`)
- [x] 카테고리 삭제 후 과거 내역 보존 + "삭제됨" 표시 — `CAT-03`
- [x] 삭제한 카테고리 이름 재사용 가능 — `CAT-04`
- [x] 타인 거래 접근 시 404 전용 화면 — `TXN-12`
- [x] 이탈 확인 3경로 전부 동작, 저장 후에는 뜨지 않음 — `TXN-09`

**D. 집계 · 예측 (STAT-01~08, BUD-03·04)**

- [x] `grep -r "now()"` 결과가 `StatsService`에 없음 (서비스 레이어 전체 검색 결과 없음)
- [x] **월 이동이 URL에 반영되고 뒤로가기로 이전 달로 돌아감**, 미래 달로는 이동할 수 없음 — `STAT-01`
- [x] 거래 없는 달이 오류 없이 0으로 표시 — `STAT-08`
- [x] 예측 금액이 서버 응답과 일치 (프론트 재계산 없음) — `STAT-04`
- [x] 고정지출 감지가 시드의 3개월 연속 항목을 찾아내고 **월 합계가 표시됨** — `STAT-06`
- [x] 히트맵 1일 요일 정렬이 여러 달에서 정확하고, **각 칸의 날짜·금액을 읽을 수 있음** — `STAT-03`
- [x] 예산 0 카테고리가 `NaN`/`Infinity`로 표시되지 않음 — `BUD-03`
- [x] 거래 추가 후 대시보드 숫자가 갱신됨

**E. CSV (CSV-01~07)**

- [x] 내보낸 파일이 **Microsoft Excel에서 한글 깨짐 없이** 열림 — ⚠️ Excel 실행 환경이 없어 직접 열어보지는 못함. 응답 바이트를 직접 검사해 첫 3바이트가 `EF BB BF`(UTF-8 BOM)이고 한글이 정상 디코딩됨을 확인(Excel의 인코딩 판정 조건 자체가 이 BOM)
- [x] 콤마·따옴표·줄바꿈이 든 메모의 왕복 일치 — `CSV-05` (`안녕, "테스트"입니다\n둘째 줄` 형태의 메모를 실제로 등록→내보내기→가져오기해 바이트 단위로 대조)
- [x] 부분 실패 시 **화면에** 행 번호와 사유가 표시되고, 성공분은 목록에 등록되어 있음 — `CSV-04`
- [ ] **실제 Excel로 왕복**: 내보낸 파일을 Excel에서 열어 `CSV(쉼표로 분리)`로 저장한 뒤 다시 가져왔을 때 한글·금액·날짜가 모두 정상 — ⚠️ 실제 Excel 자체로는 미검증(로컬에 실행 환경 없음). 다만 `DataControllerTest.MS949로_인코딩된_CSV가_깨지지_않고_가져와진다`가 MS949 바이트로 직접 만든 픽스처(콤마 섞인 금액·점 구분 날짜 포함)를 실제 `/data/import` API에 태워 통과시키고 있어, Excel이 만드는 바이트 형식 자체는 통합 테스트로 커버됨
- [x] `/data`에 중복 등록 안내 문구 노출

**F. 화면 상태 (UX-01~08)**

- [x] 모든 로딩이 스켈레톤 (스피너 없음). **네트워크 스로틀링을 켜고** 대시보드·목록·상세 세 화면에서 확인 — `UX-01`
  > 🐛 **발견 및 수정**: `/transactions/[id]`의 로딩 상태가 스켈레톤이 아니라 순수 텍스트("불러오는 중...")였다. `TransactionForm`과 같은 필드 배치의 폼 모양 스켈레톤으로 교체(`transactions/[id]/page.tsx`)
- [x] 빈 상태 / 검색 결과 없음 문구 구분, **각각의 CTA(포커스 이동 / 필터 초기화)가 동작** — `UX-02`·`UX-03`
- [x] 모든 에러 상태의 재시도 버튼이 실제 재요청 — `UX-04`
- [x] **360px · 1920px** 양쪽에서 레이아웃 정상. 360px에서 **가로 스크롤이 생기지 않고** 네비게이션이 하단 탭 바임 — `UX-05`
- [x] 라이트·다크 양쪽에서 **8개 화면 전부** 확인(`/login`·`/signup`·`/dashboard`·`/transactions`·`/transactions/[id]`·`/budgets`·`/settings/categories`·`/data`). 카테고리 색 위에 텍스트가 얹힌 곳이 없음 — `UX-07`
- [x] 목록 금액 `tabular-nums` 정렬 — `UX-08`

**G. 에러 처리**

- [x] `PRD.md` 5.1 에러 문구 매핑 **전 항목**을 실제로 발생시켜 대조 — ⚠️ 주요 코드(`EMAIL_DUPLICATED`·`CATEGORY_DUPLICATED`·`TRANSACTION_NOT_FOUND`·`CATEGORY_NOT_FOUND`·`INVALID_INPUT`·`INVALID_CSV`·`UNAUTHORIZED`·`NOT_FOUND`·`METHOD_NOT_ALLOWED`)은 이번 Phase와 이전 Phase에서 개별 발생시켜 확인함. `CATEGORY_TYPE_MISMATCH`는 `TransactionControllerTest` 통합 테스트로 커버(직접 재발생은 안 함)
- [x] **인증 토큰을 넣은 상태로** 없는 경로 호출 → 404 `NOT_FOUND` (`TRANSACTION_NOT_FOUND` 아님)
- [x] **인증 토큰을 넣은 상태로** 잘못된 메서드 호출 → 405 `METHOD_NOT_ALLOWED`
  > ⚠️ 미인증 상태에서는 Security 필터가 먼저 401로 막아 **재현되지 않는다**
- [x] 401 응답이 `ApiResponse` 포맷

**H. 브라우저 · 보안**

- [ ] **Chromium 계열 1종 + 다른 엔진 1종**에서 전체 플로우 확인 — ⚠️ 미검증. 이 세션의 Playwright MCP가 Chromium 계열만 실행 가능해 다른 엔진(Firefox·WebKit)으로는 확인하지 못함
- [x] 저장소에 시크릿이 커밋되지 않음 (`git log -p | grep`으로 확인) — 세 저장소 전체 히스토리에서 `.env` 파일 커밋 이력·`JWT_SECRET=`/`DB_PASSWORD=` 값 커밋 이력 모두 없음
- [x] 세 저장소 모두 `.env*` + `!.env.example` 규칙 존재
- [x] `dangerouslySetInnerHTML` 사용처가 **0곳**
- [x] 카테고리 색이 정규식 검증을 거친 뒤에만 인라인 스타일에 들어감

**I. 성능**

- [x] 목록 검색 500ms 이내 (시드 20,000건, 워밍업 후 중앙값) — `seed-perf.sql` 적용 후 측정, 중앙값 75ms. 측정 후 20,000건 정리 완료
- [x] 대시보드 집계 1초 이내 (동일 조건) — 중앙값 103ms

---

## Phase 13 — 챗봇 조회 위젯 (규칙 기반)

**저장소**: `moneylog-frontend` · **관련 요구사항**: CHAT-01~05

새 백엔드 API를 만들지 않는다. 이미 있는 `useMonthlyStatsQuery`·`useRecurringQuery`·`useCategoriesQuery`·`useTransactionListQuery`를 그대로 호출해 답을 만드는 프론트 전용 기능이다. `PRD.md` 3.8·5.1 참조.

**작업**

- `types/chat.ts` — `ChatIntent`(discriminated union), `ChatMessage` 타입
- `lib/chat/intents.ts` — `parseIntent(text, categories)`: 순수 함수, 키워드 우선순위 매칭(도움말 → 고정지출 → 예산 → 최근 내역 → 카테고리별 지출 → 이번 달 요약 → 실패)
- `lib/chat/answers.ts` — `buildAnswer(intent, data)`: `lib/money.ts`·`lib/date.ts` 포맷 함수만 사용, 새 포맷 함수를 추가하지 않는다
- `components/chat/ChatWidget.tsx` — 항상 마운트되는 플로팅 버튼. 열림 상태만 관리
- `components/chat/ChatPanel.tsx` — 열렸을 때만 마운트. 4개 훅 호출 + 메시지 목록(`useState`) + 입력창. 패널이 닫히면 언마운트되어 쿼리도 함께 멈춘다(기존 훅에 `enabled` 옵션을 추가하지 않는다)
- `(main)/layout.tsx`에 `<ChatWidget />` 연결
- 모션은 `TransactionList.tsx`의 기존 `motion/react` + `useReducedMotion` + `AnimatePresence` 패턴을 그대로 따른다

**DoD**

- [x] 모든 보호된 화면 우하단에 챗봇 버튼이 뜨고 클릭으로 열고 닫힘 — `CHAT-01`
- [x] "이번달 지출 얼마야" → 대시보드 총지출과 동일한 금액 응답 — `CHAT-02`
- [x] "식비 얼마 썼어" → 대시보드 카테고리별 지출의 식비 금액과 동일 — `CHAT-03`
- [x] "최근 내역 보여줘" → 거래 목록 최상단 5건과 동일한 순서·금액 — `CHAT-04`
- [x] "예산 얼마 남았어" / "고정지출 뭐있어" → 각각 예산 소진율·고정지출 카드와 동일한 값 — `CHAT-05`
- [x] 매칭되지 않는 문장("안녕") → 지원 명령 안내 문구 (빈 응답이나 에러 아님) — `CHAT-05`
- [x] 패널을 닫았다 다시 열어도 정상 동작 (언마운트/리마운트 경계에서 에러 없음)
- [x] `npx tsc --noEmit`, `npm run lint` 통과
- [x] `package.json`에 새 의존성 추가 없음(`git diff package.json`으로 확인)

> ⚠️ **추가 발견 및 수정**: 최초 구현은 "이번달"이라는 단어가 있어야만 요약 의도를 인식했고,
> 매칭되더라도 항상 현재 월 데이터만 갖고 있어 "9월", "2025년 9월"처럼 다른 달을 콕 집어
> 물으면 답하지 못했다. `intents.ts`가 문장에서 월 표현을 추출해 `ChatPanel`이 그 달을 다시
> 불러오도록 수정했다 — 이 과정에서 "월만 언급된 질문의 연도 기본값을 직전에 조회한 달의
> 연도로 잘못 잡는" 버그도 함께 발견해 고쳤다(항상 실제 오늘 기준 연도를 기본값으로 쓰도록).

---

## Phase 14 — 영수증 인식 API

**저장소**: `moneylog-backend` · **선행 조건**: Phase 6 완료 · **관련 요구사항**: `TXN-13`

영수증 사진에서 날짜·카테고리·거래처·금액을 뽑아내는 건 규칙 기반 OCR로는 부족하다(상호명만 보고 카테고리를 판단하는 건 추론 영역). Vision 지원 LLM API를 호출해 이미지 한 장을 `{ txnDate, categoryName, merchant, amount }` JSON으로 바로 변환한다. **이 API는 거래를 등록하지 않는다** — 인식 결과만 돌려주고, 실제 등록은 기존 `POST /api/v1/transactions`가 그대로 담당한다(Phase 15가 프론트에서 연결).

**작업**

- `.env.example`에 `RECEIPT_VISION_API_KEY` 추가 (`CLAUDE.md` 10장 시크릿 관리 규칙과 동일하게 커밋하지 않음)
- `controller/ReceiptController.java` — `POST /api/v1/receipts/parse` (`multipart/form-data`, 이미지 1장, `bearerAuth` 필요)
- `service/ReceiptParseService.java` — 이미지를 Vision API에 전달해 후보 값을 받는다. **API 실패·타임아웃 시 예외를 던지지 않고 빈 필드로 응답한다** — 영수증 인식은 보조 수단이라 실패해도 사용자가 직접 입력하는 경로를 막지 않는다
- 카테고리 이름 매칭은 CSV 가져오기(`CsvImportService`)의 이름 매칭 로직을 재사용한다. **매칭 실패 시 카테고리를 자동 생성하지 않고** `categoryId: null` + 인식된 `categoryName`만 응답에 담아 화면이 안내하게 한다 (`CLAUDE.md` 5장 CSV 규칙과 동일한 원칙)
- 업로드 상한 5MB, `image/jpeg`·`image/png`·`image/heic`만 허용. 초과·형식 불일치는 400 `INVALID_INPUT`
- `ErrorCode`에 `RECEIPT_PARSE_FAILED`(422) 추가 — Vision API 응답이 비어 있거나 파싱할 수 없을 때
- Swagger에 노출
- `moneylog-backend/CLAUDE.md`에 이번에 도입한 Vision API·환경변수를 기록한다

**DoD**

- [ ] 정상적인 영수증 이미지를 업로드하면 `{ txnDate, categoryName, merchant, amount }`를 응답으로 받는다 — `TXN-13`
- [ ] 카테고리 이름이 사용자 카테고리와 매칭되면 `categoryId`가 채워지고, 매칭되지 않으면 `categoryId: null` + `categoryName`만 채워진다
- [ ] 5MB 초과 또는 이미지가 아닌 파일 업로드 시 400 `INVALID_INPUT`
- [ ] Vision API 실패를 흉내 낸 상황에서도 500이 아니라 `RECEIPT_PARSE_FAILED` + 빈 필드로 응답한다
- [ ] 인증 토큰 없이 호출하면 401
- [ ] Swagger UI에서 Authorize 후 실제 호출로 확인
- [ ] `./mvnw test` 통과

---

## Phase 15 — 영수증 첨부 등록 화면

**저장소**: `moneylog-frontend` · **선행 조건**: 백엔드 Phase 14 · **관련 요구사항**: `TXN-13`

촬영 UI(카메라 강제 실행)는 만들지 않는다. 기기의 사진 앱·파일 선택기로 고른 이미지를 첨부하는 것까지만 다룬다 — 사진을 찍는 것은 OS 기본 카메라 앱의 몫이다.

**작업**

- `hooks/useReceiptParse.ts` — `/receipts/parse` 업로드 mutation
- `components/transaction/QuickAddBar.tsx`에 "영수증 첨부" 버튼 추가 — `<input type="file" accept="image/*">`로 이미지 파일을 선택한다
- 업로드 중에는 버튼에 로딩 상태만 표시한다(생성과 마찬가지로 **낙관적으로 채우지 않는다** — 서버 응답을 기다린 뒤 실제 값으로 채운다)
- 응답을 받으면 퀵 입력 바의 금액·날짜·카테고리·거래처를 채운다. **자동 저장하지 않는다** — 사용자가 값을 확인·수정하고 기존 "저장" 버튼을 눌러야 등록된다
- `categoryId`가 `null`로 오면 카테고리 select는 비워둔 채 나머지 필드만 채운다
- 인식 자체가 실패(`RECEIPT_PARSE_FAILED`)하면 토스트로 "영수증을 읽지 못했어요. 직접 입력해 주세요" 안내(`sonner`) 후 폼은 빈 상태로 유지

**DoD**

- [ ] "영수증 첨부" 클릭 시 파일 선택 창이 뜨고, 이미지를 고르면 업로드가 시작된다
- [ ] 인식 성공 시 퀵 입력 바 필드가 채워지고, "저장"을 눌러야 목록에 반영된다(자동 등록 아님) — `TXN-13`
- [ ] 매칭되지 않는 카테고리는 select가 빈 채로 남고 나머지 필드는 채워진다
- [ ] 인식 완전 실패 시 에러 토스트가 뜨고 폼은 수동 입력 가능한 빈 상태로 유지된다
- [ ] 모바일(360px)·데스크톱 모두에서 버튼과 로딩 상태가 레이아웃을 깨지 않는다
- [ ] `npx tsc --noEmit`, `npm run lint` 통과

---

## 리스크

**리스크 대응 규칙은 `CLAUDE.md`의 ⚠️ 블록이 정본이고, 확인 시점은 각 Phase의 작업·DoD가 정본이다.** 같은 함정을 표로 한 번 더 나열하면 규칙 하나를 고칠 때 세 곳을 찾아 고쳐야 하고, 실제로는 한 곳만 고쳐져 갈라진다.

어느 Phase에도 속하지 않는 전 구간 규칙만 여기 둔다.

- **API 계약(`CLAUDE.md` 5장)이 바뀌면 문서 저장소를 먼저 수정**한 뒤 백엔드 → 프론트엔드 순으로 반영한다. 폴리레포라 한쪽에서 바꾸면 조용히 어긋나고, 어긋난 사실이 화면에서야 드러난다
- **리치 텍스트 에디터를 도입하게 되면 Jsoup + DOMPurify 정화를 함께 도입한다**(`CLAUDE.md` 6장). 에디터만 먼저 넣으면 `memo`가 평문이라는 이 앱의 XSS 방어 전제가 깨진다

---

## 태그 규칙

- 각 Phase 완료 시 해당 저장소에 `v0.{Phase번호}.0` (예: Phase 5 완료 → `v0.5.0`)
- Phase 12 전체 검증 통과 시 세 저장소 모두 `v1.0.0`
- 문서 저장소(`moneylog-project`)는 태그 체계 밖이다
- **커밋·태그는 항상 사용자에게 확인받고 진행한다**
