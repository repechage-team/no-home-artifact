# NoHome 발표 구성안 (Outline)

> SSAFY 프로젝트 발표용 슬라이드 구성안. 목차 9개 섹션 기준으로 재배열.
> 형식: 요건 충족 + 기술 깊이 + 시연 균형 / 발표 시간 목표: 10~12분
> 자료 출처: `docs/deliverables`(요구사항명세·WBS·간트·AI보고서) + `docs/diagrams`(클래스·ERD·유스케이스) + `no-home-backend/docs`
>
> 범례 — 강조: `⭐` / **시각자료 상태**: ✅ 자산 있음(경로 표기) · 🟡 자동 생성됨(본 구성안 내) · ⬜ 앱 스크린샷 필요(별도) · 👥 팀 사진(플레이스홀더)

핵심 한 줄 메시지:
> **"공공데이터 실거래가를, API를 몰라도, 말 한마디로 검색한다."**

추가 기능(차별점) = **AI 어시스턴트** (Spring AI Tool Calling 기반 자연어 검색·화면조작)

---

## 목차 (9 섹션)

1. 기획 배경 · 목표 (추가 기능 중심)
2. 추진 계획 — 팀 전체 일정 & 개인별 일정
3. 시장 분석 — 경쟁 제품/서비스 비교, 차별화 전략
4. 개발 결과 — 핵심 기술 · 구현 내용
5. 개발 환경 & 전체 시스템 구조도
6. 화면 흐름도 및 시연
7. 적용 패턴 및 핵심 알고리즘
8. 기대 효과
9. 개발 후기 — 팀 사진, 개인별 회고

---

## 섹션 1. 기획 배경 · 목표 (추가 기능 중심)

### 1-0. 타이틀
- 프로젝트명 `NoHome` / 팀명 **패자부활전**(이정헌·최민식·전효준) / 발표자·날짜
- **한 줄 정의 후보** (최종 택1 — AI 차별점 + 공공데이터 조회 함의):
  1. (기준) **공공데이터 기반 실거래가 조회 및 AI 어시스턴트 서비스**
  2. 공공데이터 실거래가 조회 + 자연어 AI 어시스턴트 서비스
  3. 공공데이터로 찾고, AI로 조작하는 실거래가 어시스턴트 서비스
  4. 공공데이터 실거래가 조회와 AI 어시스턴트를 결합한 부동산 서비스
  - 부제(선택): "말하면 검색하고, 화면까지 움직인다"
- **시각자료**: ✅ [mainScreen.png](../screens/mainScreen.png)(대표 화면: 검색+지도)

### 1-1. 문제 정의 — 왜? ⭐
- **메시지**: 실거래가는 공개돼 있지만 일반인은 **접근**이 어렵다 — 문제의 본질은 "접근성"
- **페인포인트** (구체화):
  1. 국토부 공공데이터 API는 **XML 응답 + 법정동코드(LAWD_CD) + 계약년월(DEAL_YMD)** — 비개발자에겐 진입장벽
  2. 매매/전세/월세가 **서로 다른 API에 흩어짐** (RTMSDataSvcAptTrade / RTMSDataSvcAptRent)
  3. 기존 서비스는 데이터 질문엔 답해도, **대화형으로 화면(필터·페이지·지도)을 조작하는 경험이 없음** — 사용자가 직접 필터를 일일이 만져야 함
- **AI 기능의 중요성 부각**: 단순 검색 UI나 단발성 질문답변을 넘어, NoHome은 (강조) **자연어 대화로 검색·필터·페이지·지도까지 조작**하는 AI 어시스턴트 → 조작 부담을 근본적으로 제거하는 것이 핵심 차별
- **시각자료**: 🟡 원본 XML vs 결과 카드 텍스트 대비 + ✅ [gangnamguSearch.png](../screens/gangnamguSearch.png)(결과 예시)

### 1-2. 필수 기능 & 목표
- **메시지**: 과제 필수 요건을 기준선으로 확정
- 필수: 실거래가 수집(F701)·검색(F702), 회원 CRUD+로그인(F712~716), 주택정보(F720), 지도 표시
- DB 선적재 + 검색은 DB 우선 조회 하이브리드 전략
- 회원 삭제는 물리 삭제, 인증은 로그인 기반

### 1-3. 추가 기능 = AI 어시스턴트 ⭐
- **메시지**: 필수를 넘어, "말로 검색하고 화면까지 조작"하는 AI 챗봇을 추가
- 도입 동기: 진입장벽(API·필터 조작)을 자연어로 완전히 제거
- 목표: ① 질문 모드(데이터 요약 답변) ② 에이전트 모드(필터·검색·화면 자동 조작)
- 기술 기반: Spring AI 1.1.2 + gpt-4o-mini (SSAFY GMS)
- 단계: 질문모드 → 에이전트 MVP → 액션/필터 확장 → **혼합형 재설계(단일 `/assistant`)·단기 대화기억** (2026-06-21~24)
- **시각자료**: ✅ [04-personal-ai-service](../diagrams/images/usecase/04-personal-ai-service.png)(AI 검색 유스케이스) + ⬜ 챗봇 대화→화면 변화 전후 스크린샷

---

## 섹션 2. 추진 계획 — 팀 전체 일정 & 개인별 일정

### 2-1. 전체 마일스톤 타임라인
- **메시지**: 단계적 추진 (일정은 git 커밋 실측 **2026-06-12~06-24**)
- **중요 맥락**: Sprint 0~12는 **기존 원본 소유자(최민식)의 개인 작업 과정** — 팀은 이 프로젝트를 온보딩한 것. 팀 공동 기능추가 구간은 **6/18 이후**(회원·AI·심화)
- 마일스톤 로드맵: M0 계획 / M1 골격·DB / M2 실거래가 데이터 / M3 회원·인증 / M4 화면·지도 / M7 제출
- **시각자료**: ✅ [wbs-gantt.md](../deliverables/wbs-gantt.md) Mermaid 간트 + [no-home-deliverables.xlsx](../deliverables/no-home-deliverables.xlsx)(간트 시트)

### 2-2. 팀 결성 배경 & 개인별 역할 ⭐
- **팀 결성 배경**: 각자 원래 페어가 취업해 **남은 셋이 팀 구성** → 팀명 **패자부활전**. 그중 **최민식의 기존 `no-home` 프로젝트를 온보딩 + 기능 추가**하는 방향으로 결정 (프로젝트명 no-home 유지)
- **역할 분담** (6/12 회의록 기준, 대분류):

  | 담당 | 역할 |
  | --- | --- |
  | **최민식** | 공공데이터 연동 + 카카오맵 출력 + 심화 기능 |
  | **전효준** | 회원 인증 (세션 기반 → 토큰으로 변경) |
  | **이정헌** | AI 챗봇 및 어시스턴트 + **팀장**(전체 일정·진행상황 관리·역할 분담, 발표자료 메인) |

- **시각자료**: ✅ [wbs-gantt.md](../deliverables/wbs-gantt.md) 간트의 담당자별 스윔레인 + 담당·일정 요약표

### 2-3. 발표·제출 요구사항 (회의록 메모)
- **발표**: 15~20분 + **시연 3~5분 UCC 영상**(로컬 동작 실패 대비, YouTube 업로드)
- **필수 산출물**: 요구사항정의서 · UseCase Diagram · WBS · ERD · API설계서 · Class Diagram · 화면정의서 · 발표 PPT · 시연 영상
- **필수 스택**: Vue · MyBatis · Spring Boot / **채점**: 10항목(5·4·3·2·1)
- (현황: 요구사항정의서·UseCase·WBS·ERD·Class Diagram·화면(스크린샷)은 확보 / **API설계서·화면정의서**는 미완 — 별도 작업)

---

## 섹션 3. 시장 분석 — 경쟁 제품/서비스 비교, 차별화 전략

### 3-1. 경쟁 서비스 비교
- **메시지**: 위에서 아래로 갈수록 NoHome만 가능 — 핵심 차별은 **자연어 AI 화면조작**
- 비교 매트릭스 (행 순서: 공통 → 부분 → NoHome 단독)

  | 항목 | NoHome | 호갱노노 | 직방 | 네이버 부동산 | KB부동산 |
  | --- | --- | --- | --- | --- | --- |
  | 실거래가 제공 | ✅ | ✅ | ✅ | ✅ | ✅ |
  | 지도 표시 | ✅ | ✅ | ✅ | ✅ | ✅ |
  | AI 검색 챗봇 | ✅ | △ | △ | △ | △ |
  | **공공데이터 직접 연동·적재** | ✅ | ❌ | ❌ | ❌ | ❌ |
  | **AI 어시스턴트로 화면조작** | ✅ | ❌ | ❌ | ❌ | ❌ |

  > 타사 값은 사용자 보유 근거 스크린샷 기준. △ = 부분 지원(검색 챗봇은 일부 제공). 발표 전 최종 확인.
  > 행 순서: 공통(전부 ✅) → 부분(타사 △) → NoHome 단독(타사 ❌)으로 갈수록 NoHome 부각
- 차별 포인트: 대부분 검색 UI·일부 AI 검색은 있으나, **자연어로 필터·페이지·지도를 직접 조작하는 AI 에이전트는 NoHome만**
- **시각자료**: 위 매트릭스 + 🟡 타사 화면 근거(보유 스크린샷)

### 3-2. 차별화 전략
- **메시지**: NoHome의 3대 차별
  1. **자동 임포트** — 사용자가 API를 몰라도 검색 시 데이터가 채워짐
  2. **AI 어시스턴트** — 말로 검색·필터·페이지·지도 조작 (capability-driven)
  3. **공공데이터 정합성 + RAG** — 출처 보존·중복제거·멱등 적재한 **실거래 RDB를, 벡터 임베딩 없이 Tool Calling으로 조회해 근거 기반 답변**(structured retrieval 기반 RAG 구조)
- **시각자료**: 차별화 3축 다이어그램

---

## 섹션 4. 개발 결과 — 핵심 기술 · 구현 내용

### 4-1. 필수 기능 구현 현황 ⭐(평가 포인트)
- **메시지**: 과제 필수 기능을 빠짐없이 구현·검증
- 요구사항 매핑표 (F701/F702/F712~716/F720/지도) + 구현 근거 → [requirements-spec.md](../deliverables/requirements-spec.md) §3
- **시각자료**: ✅ [01-overview](../diagrams/images/usecase/01-overview.png)(전체 유스케이스: 비회원·회원·관리자) + 명세서 §3 매핑표

### 4-2. 핵심기능 ① 실거래가 검색 + 자동 임포트
- 5개 거래유형(매매/전세/월세/전월세/전체), 지역 3단계, 가격 이중 슬라이더, 정렬·페이징
- 전월세 스키마 마이그레이션(deposit/monthly_rent/contract_term 등, dealMode 매핑)
- 검색 = DB 우선, 부족분만 자동 적재 (구체 알고리즘은 섹션 7)
- **시각자료**: ✅ [02-house-service](../diagrams/images/usecase/02-house-service.png) + [house-search-class.svg](../diagrams/assets/house-search-class.svg) + [house-erd.svg](../diagrams/assets/house-erd.svg) + 스크린샷 [mainScreen](../screens/mainScreen.png)·[gangnamguSearch](../screens/gangnamguSearch.png)·[thirdHighPrice](../screens/thirdHighPrice.png)

### 4-3. 핵심기능 ② 회원·인증·개인화
- 가입/조회/수정/물리삭제/로그인/로그아웃 — BCrypt, JWT(HS256) access·refresh, HttpOnly·Secure 쿠키, refresh 블랙리스트, 운영 fail-closed
- **관리자 권한 분리**: `/api/members/search`는 관리자(`notice.admin-emails`)만, 일반회원 403
- **개인화·운영(추가기능)**: 관심지역(`/api/interest-regions`, F-INT), 공지사항(`/api/notices`, F711) — 비회원 조회·관리자 CRUD
- **시각자료**: ✅ [03-account-service](../diagrams/images/usecase/03-account-service.png) + [05-notice-operation](../diagrams/images/usecase/05-notice-operation.png) + [member-auth-class.svg](../diagrams/assets/member-auth-class.svg) + [member-erd.svg](../diagrams/assets/member-erd.svg) + 스크린샷 [signup](../screens/signup.png)·[login](../screens/login.png)·[memberInfo](../screens/memberInfo.png)·[memberSearch](../screens/memberSearch.png)·[notice](../screens/notice.png)·[accountDelete](../screens/accountDelete.png)

### 4-4. 추가기능 ③ AI 어시스턴트 — 구현 ⭐⭐
- **단일 엔드포인트 `POST /api/ai/assistant`** — LLM이 tool calling으로 질문/조작을 분기 (레거시 `/chat`·`/agent` 모드토글 제거, breaking change로 통합)
- 응답 계약 `AssistantResponse { type: answer|command, answer, command, notice }`
- 액션 6종(applyFiltersAndSearch/setFilters/paginate/selectItem/mapFocus/reset) + clarify 가드 + 필터 9종
- 진화: 질문모드(Phase1) → 에이전트 MVP(액션4·필터5) → Phase2 확장 → **혼합형 재설계(단일화)·단기기억 도입** (상세: [ai-usage-report.md](../deliverables/ai-usage-report.md))
- **시각자료**: ✅ [04-personal-ai-service](../diagrams/images/usecase/04-personal-ai-service.png) + [ai-chatbot-class.svg](../diagrams/assets/ai-chatbot-class.svg) + AI보고서 §4 액션/필터 목록

### 4-5. 검증 현황 (수치) ⭐
- **메시지**: 테스트로 품질을 증명
- 단위/통합 **백엔드 177건 + 프론트 51건 = 228건 그린** (Failures/Errors 0), E2E·브라우저 시나리오 검증
  - <!-- 출처: 2026-06-24 최신 master 실측. backend `./mvnw.cmd test` surefire 36클래스 177건 / frontend `npm test` 51건 -->
- Sprint별 빌드·테스트·라이브 검증 후 Reviewer Pass
- **시각자료**: 🟡 테스트 수 추이 표(아래) + 시나리오 체크리스트(AI보고서 §6)

  | 단계 | 백엔드 | 프론트 |
  | --- | --- | --- |
  | 질문 모드 | 78 | 15 |
  | 에이전트 MVP | 105 | 28 |
  | Phase 2 | 113 | 44 |
  | 재설계 | 144 | 44 |
  | **최신 master 실측** | **177** | **51** |

---

## 섹션 5. 개발 환경 & 전체 시스템 구조도

### 5-1. 전체 시스템 구조도 ⭐
- **메시지**: 표준 3-tier + Docker 단일 실행
- Vue3 SPA → (Vite proxy /api) → Spring Boot REST → MyBatis → MySQL
- 외부 연동 3종: 국토부 공공데이터 API, SSAFY GMS(LLM), Kakao Map
- Docker Compose로 backend/frontend/MySQL 통합 기동
- **시각자료**: ✅ [backend-packages.svg](../diagrams/assets/backend-packages.svg)(패키지 아키텍처) + [06-external-systems](../diagrams/images/usecase/06-external-systems.png)(외부 연동) + [requirements-spec.md](../deliverables/requirements-spec.md) §2.1 3-tier(ASCII+Mermaid)

### 5-2. 개발 환경 & 기술 스택
- Backend: Java 17, Spring Boot 3.5.9, MyBatis 3, MySQL 8
- AI: Spring AI 1.1.2 (ChatClient, Tool Calling), gpt-4o-mini
- Frontend: Vue 3.5(Options API/SPA), Vite 5, Kakao Maps SDK
- 인프라/협업: Docker Compose, Git(브랜치 전략), 문서 기반 Sprint
- 환경값: 공공데이터 키 2종, GMS 키, Kakao 키 (gitignored)
- **시각자료**: 🟡 계층별 스택 표(텍스트) + [common-infra-class.svg](../diagrams/assets/common-infra-class.svg)

---

## 섹션 6. 화면 흐름도 및 시연

### 6-1. 화면 흐름도
- **메시지**: 사용자 동선 한눈에
- **시각자료**: 🟡 화면 흐름 플로우차트(Mermaid, 아래)

```mermaid
flowchart LR
    A[진입] --> B[거래유형 선택]
    B --> C[지역 3단계<br/>시도→구→동]
    C --> D[가격·기간 필터]
    D --> E[검색]
    E --> F[결과 목록 + 지도 마커]
    F --> G[매물 선택/상세]
    F --> H[지도 포커스]
    F --> I{AI 어시스턴트}
    I -->|질문| J[데이터 요약 답변]
    I -->|조작| D
    A --> K[회원 로그인]
    K --> L[내 정보 / 관심지역]
```

### 6-2. 시연 — UCC 영상 (DEMO) ⭐
- **방식**: **라이브 시연 대신 UCC 영상(3~5분, YouTube 업로드)** — 로컬 동작 실패 대비 (회의록 결정)
- 영상 콘티 4단계 (상세: [demo-script.md](demo-script.md)):
  1. 지역+거래유형 검색 → 결과+지도 ([gangnamguSearch.png](../screens/gangnamguSearch.png))
  2. 가격 필터 조정 → 재검색 ([thirdHighPrice.png](../screens/thirdHighPrice.png))
  3. AI에게 "월세로 보여줘, 보증금 1억 이하" → 화면 자동 변화
  4. 회원 로그인 → 내 정보 ([login.png](../screens/login.png) · [memberInfo.png](../screens/memberInfo.png))
- 음성: Clova 또는 육성 + 자막
- **시각자료**: ⬜ UCC 영상(제작 필요) + ✅ 단계별 화면 [docs/screens/](../screens/)

---

## 섹션 7. 적용 패턴 및 핵심 알고리즘 ⭐⭐(기술 깊이)

### 7-1. AI: Tool Calling + returnDirect 패턴 (구현 완료)
- 단일 `/api/ai/assistant`에서 LLM이 tool calling으로 분기 (분리형 → 혼합형 재설계)
- LLM에 2종 도구 부여:
  - 데이터 조회 도구 `searchSeoulAptDeals` (returnDirect=false → LLM 텍스트 답변, finishReason=STOP)
  - 페이지 액션 도구 6종 (returnDirect=true → 구조화 명령 AgentCommand 직접 반환, finishReason=returnDirect로 결정적 식별)
- "아무 도구도 안 부름"을 1급 선택지로 둬 의도 오분류(불만·모호 → 억지 action) 구조적 완화
- Phase 0 PoC로 Spring AI 1.1.2 returnDirect 실거동 실측 → A안 확정 후 정식 구현
- **시각자료**: ✅ [ai-chatbot-class.svg](../diagrams/assets/ai-chatbot-class.svg) + [ai-usage-report.md](../deliverables/ai-usage-report.md) §2·§4(발화→도구분기)

### 7-2. AI: Capability-driven Agent
- 단일 출처(filterSchema) + 프론트 최종 강제
- 프론트가 capabilities + currentFilters 동봉 → LLM은 allow-list 힌트로만 사용 → 프론트가 인식 키만 적용, 미인식 키 drop + 사후 안내
- 효과: 메인 필터 추가(전월세) 시 filterSchema 한 곳만 수정해도 에이전트 자동 적응 (capability drift 차단)
- 폼↔filterSchema 동기화 테스트(`emptyFilters()` 키 ⊆ `filterSchema`)로 drift 재발 차단
- 안전장치: 휘발성 대화기억(개인정보 미저장), 분당 10회·500자 제한, **거래월은 LLM이 아닌 서버 결정적 가드**(dealYmdError/AgentCommandGuards)
- **시각자료**: ✅ [ai-usage-report.md](../deliverables/ai-usage-report.md) §2·§4.5(capability·서버 가드)

### 7-3. AI: 단기 대화기억 + 저장소 선택 근거 ⭐(기술 의사결정)
- 멀티턴 맥락: `MessageWindowChatMemory`(InMemory, window 10, SystemMessage 보존) + `MessageChatMemoryAdvisor`를 중앙 ChatClient에 부착
- 대화 키 `conversationId = memberId:<세션 UUID>` — 사용자 격리 + 세션 분리. 원문은 백엔드 JVM 힙에만, 브라우저엔 키(UUID)만
- **저장소 트레이드오프 비교**: InMemory(채택) vs localStorage vs RDB vs Redis
  - InMemory 채택 이유: 의존성 0·최속·원문 비영속(프라이버시), "닫으면 초기화" 요구 부합 / 한계: 단일 인스턴스·재기동 휘발
  - 전환 시점: 스케일아웃 시 Redis+TTL, 영구보존 요구 시 RDB (ChatMemoryRepository 빈 1곳 교체로 저비용 전환)
- 멀티턴 검증: "마포구 시세" → "방금 그 지역?" → 맥락 응답 / 세션 격리·종료 초기화 확인
- **시각자료**: ✅ [ai-usage-report.md](../deliverables/ai-usage-report.md) §5.1 저장소 4안 비교표

### 7-4. 데이터: DB 우선 + 라이브 검색 성능 개선 ⭐
- 기본: 검색 → DB coverage 판단 → 부족분만 공공데이터 호출 → XML 파싱 → `api_row_hash` 중복제거 → 적재 → 재조회 (완료 batch는 import 이력으로 skip, 멱등)
- **성능 개선(미캐시 첫 조회 17초 문제)**:
  - 응답 경로/저장 경로 분리 → **API 응답 우선 + DB 백그라운드 batch 저장**
  - row-by-row(거래당 지역·아파트 upsert+select) → **batch upsert + INSERT IGNORE**로 DB 왕복 대폭 감소
  - 공공 API `numOfRows` 100 → 1000 (1,532건: 16페이지 → 2페이지)
  - 라이브 결과 렌더 키: `resultKey → apiRowHash → dealId` 폴백
- 거래유형 매핑: 월세 0원=전세 / 초과=월세, resultCode 성공 `{"00","000"}`, 응답코드 03=빈 결과(정상)
- **시각자료**: ✅ [publicdata-import-class.svg](../diagrams/assets/publicdata-import-class.svg) + [publicdata-import-erd.svg](../diagrams/assets/publicdata-import-erd.svg) + [wbs-gantt.md](../deliverables/wbs-gantt.md) 2.6

### 7-5. 보안·인코딩 패턴
- 인증: JWT HS256 + BCrypt + fail-closed(운영 secret 검증), `/members/search` 관리자 권한 분리
- 지역 매핑: SeoulLawdCodeResolver(서울 25개 자치구) + 법정동 catalog 머지
- 인코딩: mojibake 복구(SET NAMES utf8mb4, DB 읽기 시 보정)
- GMS 키 부재 시 graceful: AI만 503, 나머지 정상 기동(EnvironmentPostProcessor)
- **시각자료**: ✅ [member-auth-class.svg](../diagrams/assets/member-auth-class.svg) + [ai-usage-report.md](../deliverables/ai-usage-report.md) §3.3·§5.3·§5.4(인코딩·graceful·로깅)

---

## 섹션 8. 기대 효과

### 8-1. 기대 효과
- **사용자**: 공공데이터 진입장벽 제거 → 누구나 실거래가 탐색 / 자연어로 손쉬운 조작
- **기술**: Spring AI Tool Calling 실전 적용 사례 / DB 우선 + 자동 적재로 응답성·정합성 확보
- **확장성**: capability-driven 구조로 신규 필터·기능 저비용 확장
- **운영**: 개인정보 미기록 로깅 정책 + fail-closed 보안으로 안전 배포
- **시각자료**: 🟡 사용자/기술/확장 임팩트 3블록(텍스트)

---

## 섹션 9. 개발 후기 — 팀 사진, 개인별 회고

### 9-1. 트러블슈팅 회고 ⭐(설득력)
- 실제 문제 4~5선 (문제→원인→해결):
  1. 공공데이터 페이지네이션 누락(142건 중 10건) → 전체 페이지 적재
  2. resultCode "000" 오분류 → 성공 화이트리스트 보정(실측 근거)
  3. **미캐시 첫 조회 17초 지연** → 응답/저장 경로 분리(응답 우선 + 백그라운드 batch), numOfRows 1000
  4. Kakao 지도 사라짐(Vue 리렌더가 비-Vue DOM 제거) → 지도 상태 reactive 밖으로
  5. AI 인코딩 환각(mojibake 입력) → seed 더블 인코딩 수정
- **시각자료**: 🟡 문제→원인→해결 3컬럼 표 + [ai-usage-report.md](../deliverables/ai-usage-report.md) §3.3

### 9-2. 팀 회고 & 사진
- **팀 메시지**: **패자부활전**이라는 결성 배경(남은 셋이 모인 팀)에도, 짧은 기간 안에 **명확한 역할 분배와 일정 관리**를 바탕으로 프로젝트를 성공적으로 마무리
- **개인별 회고** (각자 1~2줄 작성):
  - 최민식: <!-- TODO: 회고 -->
  - 전효준: <!-- TODO: 회고 -->
  - 이정헌: <!-- TODO: 회고 -->
- 잘된 점: 문서 기반 개발, 자동 임포트 UX, AI 차별화 / 한계: 서울 한정, 서울 전체 검색 시 API 호출량, 좌표 정책
- **시각자료**: 👥 팀 사진(플레이스홀더) + 🟡 회고 카드 3개

### 9-3. 마무리 & Q&A
- 한 줄 메시지 재노출 + 깃/데모 링크 + 감사
- 예상 Q&A: `script.md` 하단 참조

---

## 발표 분량 가이드

> **회의록 요구: 발표 15~20분 + 시연 3~5분 UCC 영상.** 아래는 10~12분 기준 초안 — 실제 구성은 발표 시 가지치기/확장하여 조정.

| 섹션 | 슬라이드 | 시간 |
| --- | --- | --- |
| 1 기획배경·목표 | 1-0~1-3 | 2분 |
| 2 추진계획 | 2-1~2-3 | 1분 |
| 3 시장분석 | 3-1~3-2 | 1분 |
| 4 개발결과 | 4-1~4-5 | 2.5분 |
| 5 환경·구조도 | 5-1~5-2 | 1분 |
| 6 화면흐름·시연 | 6-1~6-2 | 2분 |
| 7 패턴·알고리즘 | 7-1~7-5 | 2.5분 |
| 8 기대효과 | 8-1 | 0.5분 |
| 9 개발후기 | 9-1~9-3 | 1분 |

> 시간 압박 시 축소: 3(시장분석 1장)·5-2(스택)·7(알고리즘 2개) → 최소 압축
> 강조 확대: 7-1/7-2/7-3(AI 패턴·단기기억)·4-4(AI 구현)·6-2(시연)

## 시각자료 현황 (재정비 결과)

✅ **확보됨 — `docs/diagrams`·`docs/deliverables`에 존재, 슬라이드에 바로 삽입**
- 패키지 아키텍처: `diagrams/assets/backend-packages.svg` (5-1)
- 클래스: `ai-chatbot`·`house-search`·`member-auth`·`publicdata-import`·`common-infra`·`notice-interest`-class.svg (4·5·7장)
- ERD: `house-erd`·`member-erd`·`publicdata-import-erd`.svg (4·7장)
- 유스케이스 6종: `diagrams/images/usecase/01~06-*.png|svg` (1·4·5장)
- WBS·간트: `deliverables/wbs-gantt.md`(Mermaid)·`no-home-deliverables.xlsx` (2장)
- 요구사항 매핑표: `deliverables/requirements-spec.md` §3 (4-1)
- AI 상세(분기·저장소 4안·정책): `deliverables/ai-usage-report.md` (4-4·7장)

🟡 **본 구성안에 자동 생성됨**
- 화면 흐름도 Mermaid (6-1) / 경쟁사 비교 매트릭스 초안·검증 필요 (3-1) / 테스트 추이 표 (4-5) / 3-tier 구조도(명세서 §2.1)

✅ **앱 스크린샷 — `docs/screens/`에 확보됨**
- mainScreen·gangnamguSearch·thirdHighPrice(검색+지도) / login·signup·memberInfo·memberSearch·accountDelete(회원) / notice(공지) / desc

⬜ **제작 필요(별도 단계)**
- UCC 시연 영상(3~5분, YouTube) / 챗봇 대화 전후 캡처 / API설계서·화면정의서 산출물

👥 **팀이 제공**
- 팀 사진, 개인별 회고 문구(최민식·전효준·이정헌), 경쟁사 비교 타사 값 최종 확인
