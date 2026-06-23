# NoHome Artifact

NoHome 전체 서비스를 실행하기 위한 Docker Compose와 프로젝트 문서를 관리하는 저장소입니다.

NoHome은 공공데이터 아파트 매매 실거래가와 아파트 전월세 실거래가를 기반으로, 사용자가 지역/거래월/거래유형/가격 조건으로 실거래가를 검색하고 Kakao Map에서 위치를 확인할 수 있는 서비스입니다.

## 저장소 구성

로컬에서는 다음 폴더명을 기준으로 둡니다.

```text
no-home/
  no-home-backend/    Spring Boot API 서버
  no-home-frontend/   Vue/Vite 화면
  no-home-artifact/   문서와 Docker Compose
```

`no-home-artifact/docker-compose.yml`은 `../no-home-backend`, `../no-home-frontend` 경로를 기준으로 이미지를 빌드합니다.

## 사전 준비

- Docker Desktop
- Git
- Backend `.env`
- Frontend `.env`

처음 실행하기 전에 각 저장소의 예시 환경 파일을 복사합니다.

```powershell
cd C:\SSAFY\no-home\no-home-backend
Copy-Item .env.example .env
```

```powershell
cd C:\SSAFY\no-home\no-home-frontend
Copy-Item .env.example .env
```

Backend `.env`의 주요 값:

```text
MYSQL_DATABASE=no_home
MYSQL_USER=no_home
MYSQL_PASSWORD=no_home_dev_password
MYSQL_ROOT_PASSWORD=root_dev_password
PUBLIC_DATA_SERVICE_KEY=
PUBLIC_DATA_APT_RENT_SERVICE_KEY=
KAKAO_MAP_API_KEY=
```

- `PUBLIC_DATA_SERVICE_KEY`: 국토교통부 아파트 매매 실거래가 API key
- `PUBLIC_DATA_APT_RENT_SERVICE_KEY`: 국토교통부 아파트 전월세 실거래가 API key

Frontend `.env`의 주요 값:

```text
VITE_KAKAO_MAP_API_KEY=
```

## Docker로 전체 실행

Artifact 저장소에서 Backend, Frontend, MySQL을 함께 실행합니다.

```powershell
cd C:\SSAFY\no-home\no-home-artifact
docker compose up -d --build
```

접속 주소:

```text
Frontend: http://localhost:5173
Backend:  http://localhost:8080
Health:   http://localhost:8080/api/health
```

상태 확인:

```powershell
docker compose ps
```

로그 확인:

```powershell
docker compose logs -f backend
docker compose logs -f frontend
docker compose logs -f mysql
```

종료:

```powershell
docker compose down
```

DB 데이터까지 삭제하려면 volume을 함께 삭제합니다. 기존 MySQL 데이터가 사라지므로 필요한 경우에만 사용합니다.

```powershell
docker compose down -v
```

## 코드 변경 반영

Docker Desktop에서 컨테이너를 `Stop` 후 `Start`하는 것은 기존 이미지를 다시 실행하는 동작입니다. 로컬 코드 변경을 반영하려면 이미지를 다시 빌드해야 합니다.

전체 재빌드:

```powershell
docker compose up -d --build --force-recreate
```

Frontend만 변경:

```powershell
docker compose up -d --build --force-recreate frontend
```

Backend만 변경:

```powershell
docker compose up -d --build --force-recreate backend
```

## 검색 기능 요약

브라우저 검색은 다음 거래 유형을 지원합니다.

| 거래 유형 | 의미 | 가격 필터 |
| --- | --- | --- |
| 매매 | 아파트 매매 실거래가 | 매매가 |
| 전세 | 아파트 전월세 API 중 월세 0원 | 보증금 |
| 월세 | 아파트 전월세 API 중 월세 0원 초과 | 보증금 + 월세 |
| 전월세 | 전세 + 월세 | 가격 필터 없음 |
| 전체 | 매매 + 전세 + 월세 | 가격 필터 없음 |

`서울특별시`를 선택한 경우 시군구를 반드시 선택해야 검색합니다. 서울 전체 자동수집은 호출량이 커서 브라우저 검색 옵션에서 제공하지 않습니다.

## 테스트

Backend:

```powershell
cd C:\SSAFY\no-home\no-home-backend
.\mvnw.cmd test
```

Frontend:

```powershell
cd C:\SSAFY\no-home\no-home-frontend
npm.cmd test
npm.cmd run build
```

## 문서 목록

```text
no-home-artifact/
  README.md
  docs/
    PRD.md
    spec.md
    plan.md
    sprints/
```

참고 API 문서:

- `아파트 매매 실거래가 자료 기술문서.pdf`
- `아파트 전월세 실거래가 자료 기술문서.pdf`
