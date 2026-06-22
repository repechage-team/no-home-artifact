# NoHome Artifact

NoHome은 공공데이터 아파트 매매 실거래가 자료를 기반으로 사용자가 지역, 거래월, 가격 조건으로 매매 내역을 검색하고 Kakao Map에서 위치를 확인할 수 있는 서비스입니다.

## 저장소 구성

로컬에서는 세 저장소를 같은 상위 폴더 아래에 두는 구성을 기준으로 합니다.

```text
NoHome/
  Backend/    Spring Boot API 서버
  Frontend/   Vue/Vite 화면
  Artifact/   문서와 Docker Compose
```

예시:

```powershell
mkdir NoHome
cd NoHome

git clone <Backend-repository-url> Backend
git clone <Frontend-repository-url> Frontend
git clone <Artifact-repository-url> Artifact
```

`Artifact/docker-compose.yml`은 `../Backend`, `../Frontend` 경로를 기준으로 이미지를 빌드합니다. 폴더명이 다르면 compose 파일의 `build.context`, `env_file`, SQL mount 경로도 함께 맞춰야 합니다.

## 사전 준비

- Docker Desktop
- Git
- Backend `.env`
- Frontend `.env`

처음 실행하기 전에 각 저장소의 예시 환경 파일을 복사합니다.

```powershell
cd C:\SSAFY\workspace\NoHome\Backend
Copy-Item .env.example .env
```

```powershell
cd C:\SSAFY\workspace\NoHome\Frontend
Copy-Item .env.example .env
```

필요한 값은 각 `.env`에 직접 입력합니다.

```text
Backend/.env
  MYSQL_DATABASE=no_home
  MYSQL_USER=no_home
  MYSQL_PASSWORD=no_home_dev_password
  MYSQL_ROOT_PASSWORD=root_dev_password
  PUBLIC_DATA_SERVICE_KEY=
  KAKAO_MAP_API_KEY=

Frontend/.env
  VITE_KAKAO_MAP_API_KEY=
```

## Docker로 전체 실행

Artifact 저장소에서 Backend, Frontend, MySQL을 함께 실행합니다.

```powershell
cd C:\SSAFY\workspace\NoHome\Artifact
docker compose --env-file ..\Backend\.env up -d --build
```

실행 후 접속 주소:

```text
Frontend: http://localhost:5173
Backend:  http://localhost:8080
Health:   http://localhost:8080/api/health
```

상태 확인:

```powershell
docker compose --env-file ..\Backend\.env ps
```

로그 확인:

```powershell
docker compose --env-file ..\Backend\.env logs -f backend
docker compose --env-file ..\Backend\.env logs -f frontend
docker compose --env-file ..\Backend\.env logs -f mysql
```

종료:

```powershell
docker compose --env-file ..\Backend\.env down
```

DB 데이터까지 삭제하려면 volume을 함께 삭제합니다. 이 명령은 기존 MySQL 데이터가 사라지므로 필요한 경우에만 사용합니다.

```powershell
docker compose --env-file ..\Backend\.env down -v
```

## 코드 변경 반영

Docker Desktop에서 컨테이너를 `Stop` 후 `Start`하는 것은 기존 이미지와 기존 컨테이너를 다시 실행하는 동작입니다. 로컬 코드 변경은 보통 반영되지 않습니다.

코드 변경을 컨테이너에 반영하려면 이미지를 다시 빌드하고 컨테이너를 새 이미지로 재생성해야 합니다.

전체 재빌드:

```powershell
cd C:\SSAFY\workspace\NoHome\Artifact
docker compose --env-file ..\Backend\.env up -d --build --force-recreate
```

Frontend만 바뀐 경우:

```powershell
docker compose --env-file ..\Backend\.env up -d --build --force-recreate frontend
```

Backend만 바뀐 경우:

```powershell
docker compose --env-file ..\Backend\.env up -d --build --force-recreate backend
```

Backend와 Frontend가 모두 바뀐 경우:

```powershell
docker compose --env-file ..\Backend\.env up -d --build --force-recreate backend frontend
```

MySQL 데이터는 named volume(`no-home-mysql-data`)에 저장되므로 위 재빌드 명령만으로는 삭제되지 않습니다.

## 로컬 개발 실행

개발 중에는 MySQL만 Docker로 띄우고 Backend, Frontend를 로컬 프로세스로 실행할 수 있습니다.

Backend:

```powershell
cd C:\SSAFY\workspace\NoHome\Backend
docker compose up -d mysql
.\mvnw.cmd spring-boot:run
```

Frontend:

```powershell
cd C:\SSAFY\workspace\NoHome\Frontend
npm install
npm run dev
```

Frontend 개발 서버는 `http://localhost:5173`에서 실행되며, `/api` 요청은 Vite proxy를 통해 Backend로 전달됩니다.

## 테스트

Backend:

```powershell
cd C:\SSAFY\workspace\NoHome\Backend
.\mvnw.cmd test
```

Frontend:

```powershell
cd C:\SSAFY\workspace\NoHome\Frontend
npm.cmd test
npm.cmd run build
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

- `docs/PRD.md`: 제품 요구사항
- `docs/spec.md`: 기술 명세
- `docs/plan.md`: 개발 계획
- `docs/sprints/`: 스프린트별 작업 기록
- `아파트 매매 실거래가 자료 기술문서.pdf`: 공공데이터 API 기술 문서
