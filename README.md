# Trendy Trip

졸업작품으로 제작한 여행 장소 탐색 및 일정 추천 웹 애플리케이션입니다. 지역과 여행 조건을 선택하고, 장소를 지도에서 살펴본 뒤 여행 일정을 구성할 수 있습니다.

## 기술 구성

| 영역 | 사용 기술 |
| --- | --- |
| 프런트엔드 | Vue 3, TypeScript, Vite, Pinia, Axios |
| 백엔드 | Python, FastAPI, SQLAlchemy |
| 데이터베이스 | MySQL |
| 지도 및 경로 | 네이버 지도 API |

## 실행 전 준비

- Python 3.12 권장, Node.js와 npm, MySQL 8.x 설치
- 네이버 지도 JavaScript API용 클라이언트 ID와 경로 API용 인증 정보 준비
- 프로젝트 루트에서 `be/`와 `fe/` 디렉터리를 확인

> 이 저장소의 SQL 파일은 테이블 구조만 포함합니다. 장소 데이터가 없으면 검색과 일정 생성 결과가 비어 있거나 실패할 수 있습니다. 실제 기능을 재현하려면 별도의 장소 데이터가 필요합니다.

## 1. 데이터베이스 준비

MySQL에 `tt` 데이터베이스를 생성합니다.

```sql
CREATE DATABASE tt CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

`be/TripScheduler/db/`의 `tt_*.sql` 파일은 각 테이블의 스키마 덤프입니다. MySQL 클라이언트에서 각 파일을 `tt` 데이터베이스에 가져온 뒤 필요한 장소 데이터를 채워 주세요. 백엔드에서도 시작 시 SQLAlchemy 모델에 따라 누락된 테이블을 생성하지만, 장소 데이터는 자동으로 추가되지 않습니다.

## 2. 백엔드 설치 및 실행

프로젝트 루트에서 다음 명령을 실행합니다. Windows PowerShell 기준입니다.

```powershell
cd be
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m pip install -r TripScheduler/requirements.txt
```

`be/.env` 파일을 만들고 자신의 접속 정보로 채웁니다. 비밀번호나 API 비밀키가 포함된 `.env`는 GitHub에 업로드하지 마세요.

```dotenv
DATABASE_URL=mysql+pymysql://USER:PASSWORD@127.0.0.1:3306/tt
NAVER_API_CLIENT_ID=YOUR_CLIENT_ID
NAVER_API_CLIENT_SECRET=YOUR_CLIENT_SECRET
```

같은 `be` 디렉터리에서 서버를 실행합니다.

```powershell
python -m uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

실행 후 `http://127.0.0.1:8000/docs`에서 API 문서를 열어 서버 상태를 확인할 수 있습니다.

## 3. 프런트엔드 설치 및 실행

별도 터미널에서 프로젝트 루트의 `fe` 디렉터리로 이동합니다.

```powershell
cd fe
npm install
npm run dev
```

터미널에 표시된 로컬 주소(기본값 `http://localhost:5173`)로 접속합니다. 프런트엔드는 `http://127.0.0.1:8000`의 백엔드를 사용하도록 설정되어 있으므로 백엔드도 함께 실행해야 합니다.

## 지도 API 설정

프런트엔드 지도 스크립트의 클라이언트 ID는 현재 `fe/src/services/useNaverMap.ts`에 직접 들어 있습니다. 자신의 네이버 지도 JavaScript API 클라이언트 ID로 변경하고, 네이버 클라우드 플랫폼에서 로컬 접속 주소를 허용 도메인으로 등록해야 지도가 표시됩니다. 일정의 경로 계산에는 `be/.env`의 네이버 API 인증 정보가 사용됩니다.

## 프로젝트 구조

```text
be/
  app/                 FastAPI 서버 및 API 라우터
  TripScheduler/       여행 일정 생성 로직과 DB 스키마
  requirements.txt     백엔드 의존성
fe/
  src/                 Vue 화면 및 API 연동
  package.json         프런트엔드 실행 명령
```

## 문제 해결

- **백엔드 시작 시 DB 연결 오류**: MySQL 실행 여부, `tt` 생성 여부, `be/.env`의 `DATABASE_URL`을 확인하세요.
- **화면에 장소가 표시되지 않음**: SQL 파일에는 장소 데이터가 없으므로 데이터가 입력되었는지 확인하세요.
- **지도가 표시되지 않음**: 지도 클라이언트 ID와 허용 도메인을 확인하세요.
- **API 요청 실패**: 백엔드가 8000번 포트에서 실행 중인지 확인하세요. 다른 포트를 사용한다면 `fe/src/utils/constants.ts`의 `BACKEND_URL`과 직접 지정된 주소(`fe/src/composables/api/useHashtagSearchApi.ts`)도 수정해야 합니다.
