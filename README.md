# NoHome Artifact

NoHome은 공공데이터의 아파트 매매 실거래가 정보를 기반으로 사용자가 지역별 거래 내역을 조회할 수 있는 서비스입니다.

사용자는 시/군/구, 동, 거래 월, 아파트명을 조건으로 매매 거래를 검색할 수 있으며, 검색 결과를 목록과 Kakao Map 지도에서 함께 확인할 수 있습니다.

## 저장소 구성

NoHome은 3개의 저장소로 분리되어 있습니다.

```text
NoHome/
  Backend/    Spring Boot API 서버
  Frontend/   Vue/Vite 웹 화면
  Artifact/   프로젝트 문서 및 산출물
```

각 저장소는 독립적으로 clone합니다. 로컬에서 함께 실행하려면 같은 상위 폴더 아래에 세 저장소를 배치하는 것을 권장합니다.

```powershell
mkdir NoHome
cd NoHome

git clone <Backend-repository-url> Backend
git clone <Frontend-repository-url> Frontend
git clone <Artifact-repository-url> Artifact
```

## 문서 목록

```text
Artifact/
  README.md
  AGENTS.md
  HarnessGuide.md
  SubAgentStrategy.md
  pjt.pdf
  아파트 매매 실거래가 자료 기술문서.pdf
  docs/
```

- `README.md`: 전체 프로젝트 개요와 로컬 실행 방법
- `docs/PRD.md`: 제품 요구사항
- `docs/spec.md`: 기술 명세
- `docs/plan.md`: 개발 계획
- `docs/sprints/`: 스프린트별 작업 계획과 실행 기록
- `pjt.pdf`: 프로젝트 과제 자료
- `아파트 매매 실거래가 자료 기술문서.pdf`: 공공데이터 API 기술 문서

## 사전 준비

Docker로 전체 서비스를 실행하려면 아래 프로그램이 필요합니다.

- Docker Desktop
- Git

Backend와 Frontend를 Docker 없이 로컬 프로세스로 직접 실행하려면 아래 프로그램도 필요합니다.

- Java 17 이상
- Node.js 20 이상

## Docker로 전체 실행

Backend, Frontend, MySQL을 Docker Compose로 함께 실행할 수 있습니다. 이 방식은 다른 개발자가 Java, Node.js를 직접 실행하지 않고도 프로젝트를 확인할 수 있도록 하기 위한 실행 방식입니다.

`Copy-Item .env.example .env` 명령은 저장소에 포함된 `.env.example` 내용을 로컬 실행용 `.env` 파일로 복사합니다. 따라서 각 저장소의 `.env.example`에는 실행에 필요한 기본값이나 예시값이 미리 들어 있어야 합니다.

실제 API key, DB 비밀번호처럼 개인마다 달라지는 값은 `.env.example`에 실값을 넣지 말고, 복사 후 생성된 `.env` 파일에서 수정합니다.

먼저 Backend와 Frontend의 환경 파일을 만듭니다.

```powershell
cd C:\SSAFY\workspace\NoHome\Backend
Copy-Item .env.example .env
```

```powershell
cd C:\SSAFY\workspace\NoHome\Frontend
Copy-Item .env.example .env
```

필요한 API key를 각 `.env`에 입력합니다.

```text
Backend/.env
  PUBLIC_DATA_SERVICE_KEY=
  KAKAO_MAP_API_KEY=

Frontend/.env
  VITE_KAKAO_MAP_API_KEY=
```

Artifact 저장소에서 전체 서비스를 실행합니다.

```powershell
cd C:\SSAFY\workspace\NoHome\Artifact
docker compose --env-file ..\Backend\.env up --build
```

실행 후 브라우저에서 아래 주소로 접속합니다.

```text
http://localhost:5173
```

백엔드 API는 아래 주소에서 실행됩니다.

```text
http://localhost:8080
```

상태 확인:

```text
http://localhost:8080/api/health
```

종료하려면 Artifact 저장소에서 아래 명령어를 실행합니다.

```powershell
docker compose --env-file ..\Backend\.env down
```

DB 데이터까지 삭제하려면 volume을 함께 삭제합니다.

```powershell
docker compose --env-file ..\Backend\.env down -v
```

## 로컬 개발 실행

개발 중에는 Backend와 Frontend를 로컬 프로세스로 따로 실행할 수도 있습니다.

Backend:

```powershell
cd C:\SSAFY\workspace\NoHome\Backend
Copy-Item .env.example .env
docker compose up -d mysql
.\mvnw.cmd spring-boot:run
```

Frontend:

```powershell
cd C:\SSAFY\workspace\NoHome\Frontend
Copy-Item .env.example .env
npm install
npm run dev
```

프론트엔드의 `/api` 요청은 Vite proxy를 통해 백엔드로 전달됩니다.

## 환경 변수

Backend의 `.env`에는 DB 연결 정보와 외부 API key를 설정합니다.

아래 값들은 `Backend/.env.example`에 예시값으로 준비해두고, 개발자는 `Copy-Item .env.example .env`로 복사한 뒤 필요한 값을 `Backend/.env`에서 수정합니다.

```text
MYSQL_PORT=3306
DB_URL=jdbc:mysql://localhost:3306/no_home?serverTimezone=Asia/Seoul&characterEncoding=UTF-8
DB_USERNAME=no_home
DB_PASSWORD=no_home_dev_password
PUBLIC_DATA_SERVICE_KEY=
KAKAO_MAP_API_KEY=
```

Frontend의 `.env`에는 Kakao Map JavaScript key를 설정합니다.

아래 값은 `Frontend/.env.example`에 빈 예시값으로 준비해두고, 개발자는 복사 후 `Frontend/.env`에 본인의 Kakao JavaScript key를 입력합니다.

```text
VITE_KAKAO_MAP_API_KEY=
```

## 테스트

Backend:

```powershell
cd C:\SSAFY\workspace\NoHome\Backend
.\mvnw.cmd test
```

Frontend:

```powershell
cd C:\SSAFY\workspace\NoHome\Frontend
npm run test:auto-import
npm run build
```
