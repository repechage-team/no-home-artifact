# spec.md

## Sprint10 Repository Layout Update

As of 2026-06-12, the repository is organized as ordinary folders:

```text
Backend/
Frontend/
Artifact/
```

The frontend is no longer copied into Spring Boot static resources for local development.
Run the backend on `http://localhost:8080` and the frontend on `http://localhost:5173`.
The Vite dev server proxies `/api` requests to the backend.

## 기술 기준선

- Backend: Spring Boot.
- Persistence: MyBatis.
- Database: MySQL.
- Frontend: Vue.js + Vite. 서버와의 동적 상호작용은 Spring Boot REST API 호출을 기준으로 한다.
- Server: Spring Boot embedded Tomcat.
- Auth: 세션 기반 로그인.
- Delivery: 최종 제출물은 Vue 빌드 결과를 Spring Boot 정적 리소스에 포함한다.

## 아키텍처

계층형 구조를 사용한다.

- Controller: 요청 검증, 세션 확인, 응답 라우팅.
- Service: 비즈니스 규칙, 트랜잭션 경계, 공공데이터 가공 정책.
- Mapper: MyBatis 기반 SQL 접근.
- DTO / Model: 요청, 응답, DB 레코드 형태.

Controller에 SQL을 두지 않는다. Mapper에는 인증, 삭제 정책, 권한 판단 같은 비즈니스 결정을 넣지 않는다.

## 패키지 가이드

권장 패키지 구조:

```text
com.ssafy.home
├── member
│   ├── controller
│   ├── service
│   ├── mapper
│   └── dto
├── house
│   ├── controller
│   ├── service
│   ├── mapper
│   └── dto
├── common
│   ├── config
│   ├── exception
│   └── util
└── optional
    ├── favorite
    ├── commercial
    └── notice
```

기존 프로젝트 소스가 다른 네이밍을 사용한다면 기존 구조를 우선한다.

## M2 공공데이터 결정 초안

아파트 매매 실거래가 API는 공공데이터포털의 `국토교통부_아파트 매매 실거래가 자료`를 기준으로 한다.

확정:

- 주택 유형 범위: 아파트 매매.
- 초기 적재 지역: 서울특별시 동작구.
- 초기 API 지역 코드: `LAWD_CD=11590`.
- 초기 적재 기간: 제일 최근 조회 가능 계약년월 1개월. 실행 시점의 최신 `yyyyMM`을 기본값으로 사용하되, API 응답이 비어 있으면 직전 월 재시도 여부를 작업 로그에 남긴다.
- API 요청 기준: `LAWD_CD`는 법정동코드 10자리 중 앞 5자리, `DEAL_YMD`는 계약년월 6자리.
- 좌표 확보 방식: 추후 Kakao Map API를 사용한다. M2에서는 `lat`, `lng`를 nullable로 둔다.

추가 확정:

- 사용자가 미적재 범위를 요청하면 이미 적재된 범위를 제외하고 부족분만 추가 적재한다.
- 요청 범위 전체 재적재는 일반 사용자 요청 흐름과 분리하고, 운영/관리자용 강제 새로고침 옵션으로 둔다.

API 원본 row 중복 판정:

- `api_row_hash`는 아래 필드를 정규화한 뒤 SHA-256 등 안정적인 hash로 생성한다.
- 구성 필드: `source_api`, `lawd_cd`, `deal_ymd`, `umd_nm`, `jibun`, `apt_nm`, `deal_year`, `deal_month`, `deal_day`, `deal_amount`, `exclu_use_ar`, `floor`.
- 문자열은 trim하고, 거래금액은 쉼표 제거 전/후 정책을 일관되게 적용한다. M2에서는 원본 문자열 trim 기준을 우선한다.
- 향후 API에서 더 안정적인 거래 식별자가 제공되면 hash 구성은 migration을 통해 조정할 수 있다.

## 데이터 모델 초안

정확한 스키마는 기존 프로젝트 SQL과 샘플 데이터를 확인한 뒤 확정한다.

### `regions`

- `region_id`: 기본 키.
- `lawd_cd`: API 요청용 지역 코드. 법정동코드 10자리 중 앞 5자리. 예: 서울 동작구 `11590`.
- `legal_dong_code`: 법정동코드 10자리. 법정동 코드 데이터 확보 전까지 nullable.
- `sido`: 시도. 예: 서울특별시.
- `sigungu`: 시군구. 예: 동작구.
- `umd_nm`: 읍면동. API 응답의 `umdNm`.
- `lat`, `lng`: 지역 중심 좌표. Kakao Map API 연동 전까지 nullable.
- 유니크 후보: `lawd_cd`, `umd_nm`.

### `members`

- `member_id`: 기본 키.
- `email` 또는 `login_id`: 로그인 식별자, 유니크.
- `password_hash`: 비밀번호 해시. 평문 저장 금지.
- `name`: 사용자 이름.
- `phone`: 선택 연락처.
- `created_at`, `updated_at`.

### `houses`

- `house_id`: 기본 키.
- `region_id`: `regions.region_id` 외래 키.
- `sgg_cd`: API 응답의 시군구 코드. `LAWD_CD`와 동일한 5자리 코드로 본다.
- `umd_nm`: API 응답의 법정동명.
- `jibun`: 지번.
- `apt_nm`: 아파트명.
- `build_year`: 건축년도.
- `lat`, `lng`: Kakao Map API 연동 전까지 nullable.
- `created_at`, `updated_at`.
- 유니크 후보: `sgg_cd`, `umd_nm`, `jibun`, `apt_nm`, `build_year`.

### `house_deals`

- `deal_id`: 기본 키.
- `house_id`: 외래 키.
- `source_api`: 원천 API 식별자. 예: `RTMSDataSvcAptTrade`.
- `lawd_cd`: 요청에 사용한 `LAWD_CD`.
- `deal_ymd`: 요청에 사용한 `DEAL_YMD`.
- `house_type`: `apartment`.
- `deal_type`: `sale`.
- `deal_year`, `deal_month`, `deal_day`.
- `deal_date`: `deal_year`, `deal_month`, `deal_day`를 합친 계약일.
- `deal_amount`: 원본 거래금액 문자열.
- `deal_amount_manwon`: 쉼표를 제거한 거래금액 숫자. 단위는 만원.
- `exclu_use_ar`: 전용면적.
- `floor`.
- `apt_dong`: 아파트 동. 소유권 이전등기 완료 건에 한해 제공될 수 있으므로 nullable.
- `buyer_gbn`: 매수자 유형. nullable.
- `sler_gbn`: 매도자 유형. nullable.
- `dealing_gbn`: 거래 유형. nullable.
- `estate_agent_sgg_nm`: 중개사 소재지. nullable.
- `cdeal_type`: 해제 여부. nullable.
- `cdeal_day`: 해제 사유 발생일. nullable.
- `rgst_date`: 등기일자. nullable.
- `land_leasehold_gbn`: 토지임대부 여부. nullable.
- `api_row_hash`: 원본 row 중복 방지용 hash.
- `raw_response`: 원본 XML item을 보존한 JSON. 디버깅용 nullable.
- `created_at`, `updated_at`.
- 유니크 후보: `api_row_hash`.

### `public_data_import_batches`

- `import_batch_id`: 기본 키.
- `source_api`: 원천 API 식별자. 예: `RTMSDataSvcAptTrade`.
- `lawd_cd`: 요청 지역 코드.
- `deal_ymd`: 요청 계약년월.
- `house_type`: `apartment`.
- `deal_type`: `sale`.
- `status`: `requested` / `success` / `failed` / `partial`.
- `total_count`: API 응답 전체 건수.
- `imported_count`: 실제 신규 적재 건수.
- `skipped_count`: 중복으로 건너뛴 건수.
- `error_message`: 실패 사유.
- `requested_at`, `completed_at`.
- 유니크 후보: `source_api`, `lawd_cd`, `deal_ymd`, `house_type`, `deal_type`.

### 선택 테이블

- `favorite_regions`: 회원과 관심 지역 매핑.
- `commercial_places`: 동네 상가/업종 데이터.
- `environment_records`: 녹지, 폐수, 대기 배출 관련 데이터.
- `notices`: 공지사항 CRUD 데이터.

회원 삭제는 물리 삭제를 기준으로 한다. 회원과 연결되는 선택 기능 데이터가 생기면 외래 키 정책 또는 서비스 계층 삭제 순서를 Sprint 계약에 명시한다.

## 공공데이터 처리 정책

서비스 검색은 DB에 적재된 데이터를 우선 사용한다. API 또는 파일 데이터는 별도 적재 경로를 통해 DB에 저장한 뒤 검색 대상이 된다.

확정:

- API/파일 데이터를 DB에 먼저 적재한다.
- 공공데이터 적재는 반복 실행해도 중복이 무제한 생성되지 않아야 한다.
- 데이터 출처와 적재 기준을 추적할 수 있어야 한다.
- 공공데이터 API 키는 `application-local.properties` 같은 gitignored local config로 주입하고 저장소에 커밋하지 않는다.
- 장기 데이터 전략은 하이브리드 방식으로 한다.
  - 기본 시연/초기 범위는 선적재한다.
  - 검색은 항상 DB를 우선 조회한다.
  - 미적재 지역/기간은 부족분만 API 호출해 적재한 뒤 DB에서 다시 조회한다.
  - 사용자 검색 경험을 위해 자동 적재 기본값은 활성화한다. 자동 적재를 원하지 않는 호출자는 명시적으로 비활성화한다.
  - 검색 API와 부족분 자동 적재 연결은 별도 Sprint로 분리한다.

M2 시작 전 세부 결정 필요:

- 초기 적재 지역 범위: 서울특별시 동작구.
- 초기 적재 기간 범위: 제일 최근 조회 가능 계약년월 1개월.
- 초기 적재 주택 유형 범위: 아파트 매매.
- 미적재 요청 처리 방식: 부족분만 추가 적재. 전체 재적재는 운영/관리자용 강제 새로고침 옵션으로 분리.
- 중복 판정 방식: `source_api`, `lawd_cd`, `deal_ymd`, `umd_nm`, `jibun`, `apt_nm`, `deal_year`, `deal_month`, `deal_day`, `deal_amount`, `exclu_use_ar`, `floor` 기반 `api_row_hash`.

## Frontend / Delivery

Vue.js + Vite 기반 프론트엔드를 사용한다. 개발 중에는 Vue 프로젝트에서 Spring Boot REST API를 호출하고, 최종 제출 또는 시연 빌드에서는 Vue 빌드 결과물을 Spring Boot 정적 리소스 경로에 포함해 Spring Boot 애플리케이션 하나로 실행할 수 있게 한다.

권장 구조:

```text
backend/
└── src/main/resources/static/
    └── Vue build output

frontend/
├── src/
├── package.json
└── vite.config.*
```

실제 폴더명은 기존 소스 구조가 생기면 해당 구조를 우선한다. Vue 빌드 산출물을 Spring Boot 정적 리소스에 복사하는 방식은 구현 Sprint에서 확정한다.

## API / Route 초안

REST API 구조를 기준으로 한다. 각 기능은 명확한 API 계약을 가져야 하고, Vue 화면은 API를 호출해 데이터를 렌더링한다.

| 영역 | Method / Route | 목적 |
| --- | --- | --- |
| House | `GET /houses/search?dong=&aptName=` | 실거래가 검색. |
| House | `GET /houses/{houseId}` | 주택 기본 정보와 최근 거래 조회. |
| Commercial | `GET /commercial/search?lat=&lng=&category=` | 주변 상권 지도 표시용 상권 검색. |
| Region | `GET /regions/sido` | 시도 목록 조회. |
| Region | `GET /regions/gugun?sido=` | 시군구 목록 조회. |
| Region | `GET /regions/dong?gugun=` | 읍면동 목록 조회. |
| Member | `POST /members` | 회원 가입. |
| Member | `GET /members/me` | 현재 회원 조회. |
| Member | `PUT /members/me` | 현재 회원 수정. |
| Member | `DELETE /members/me` | 현재 회원 물리 삭제. |
| Auth | `POST /login` | 로그인. |
| Auth | `POST /logout` | 로그아웃. |

JSON 응답 구조는 다음 형태로 통일한다.

```json
{
  "success": true,
  "message": "ok",
  "data": {}
}
```

파일 다운로드, 리다이렉트, 에러 페이지처럼 JSON이 부적절한 경우를 제외하고 API 응답은 위 형식을 따른다.

## 핵심 규칙

- 비밀번호는 평문으로 저장하지 않는다.
- 회원 수정과 삭제는 현재 로그인 사용자의 권한을 확인한다.
- 회원 삭제는 물리 삭제를 기준으로 한다.
- 인증은 세션 기반 로그인을 기준으로 한다.
- 애플리케이션은 Spring Boot REST API + Vue.js 구조를 기준으로 한다.
- 최종 제출물은 Vue 빌드 결과를 Spring Boot 정적 리소스에 포함한다.
- 검색 쿼리는 가능한 경우 지역, 아파트명, 거래일 기준 인덱스를 고려한다.
- 공공데이터 적재는 반복 실행해도 중복이 무제한 생성되지 않아야 한다.
- API 키와 DB 비밀번호는 커밋하지 않는다.
- DB 변경 작업은 Service 계층에서 트랜잭션 경계를 관리한다.
- 실거래가 검색 결과는 목록과 지도에 함께 표시한다.
- 첫 추가 기능은 주변 상권 지도다.

## 검증 전략

엄격 검증:

- 회원 가입, 로그인, 로그아웃, 수정, 삭제.
- 공공데이터 적재와 중복 처리.
- 동별 검색과 아파트명 검색.
- 지역 코드, 법정동명, 아파트명 불일치 처리.
- Insert, update, delete, filtered search Mapper SQL.

완화 검증:

- 정적 페이지 레이아웃.
- 조회 결과 표시 형식.
- 데이터 정확성 검증 이후의 지도 마커 표시.

## 결정 상태

| 결정 | 상태 | 내용 |
| --- | --- | --- |
| Backend framework | 확정 | Spring Boot. |
| Frontend | 확정 | Vue.js + Vite. |
| 화면/API 구조 | 확정 | Vue 화면이 Spring Boot REST API를 호출한다. |
| 제출/실행 구조 | 확정 | 최종 제출물은 Vue 빌드 결과를 Spring Boot 정적 리소스에 포함한다. |
| 인증 방식 | 확정 | 세션 기반 로그인. |
| 회원 삭제 정책 | 확정 | 물리 삭제. |
| 공공데이터 처리 | 확정 | API/파일 데이터를 DB에 먼저 적재한다. M2 초기 범위는 서울특별시 동작구, 최신 조회 가능 계약년월 1개월, 아파트 매매다. 미적재 요청은 부족분만 추가 적재하고 전체 재적재는 운영/관리자용 강제 새로고침으로 분리한다. |
| 첫 추가 기능 | 확정 | 주변 상권 지도. |
| 지도 범위 | 확정 | 실거래가 검색 결과도 지도에 표시한다. |
