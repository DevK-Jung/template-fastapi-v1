# FastAPI Template

Python 3.12+, FastAPI, SQLAlchemy 2.0 기반의 프로덕션 레디 백엔드 템플릿입니다.

## 기술 스택

| 분류 | 기술 |
|------|------|
| Framework | FastAPI 0.115+ |
| Server | Uvicorn |
| ORM | SQLAlchemy 2.0 (async), SQLModel |
| Database | PostgreSQL (asyncpg), MySQL (aiomysql) |
| Validation | Pydantic 2.x, pydantic-settings |
| Auth | passlib (bcrypt), JWT (HS256) |
| AI/ML | OpenAI Whisper (STT) |
| Infra | Docker Compose |
| Package Manager | Poetry |

## 프로젝트 구조

```
src/fastapi_template/
├── main.py                   # 애플리케이션 진입점
├── api/
│   └── v1/
│       ├── users.py          # 사용자 관리 API
│       ├── sst_whisper.py    # STT(음성 인식) API
│       └── samples.py        # 예제 엔드포인트
├── core/
│   ├── config/
│   │   ├── settings.py       # 환경 설정
│   │   ├── database.py       # DB 엔진 및 세션
│   │   └── lifespan.py       # 앱 라이프사이클
│   ├── exception/
│   │   └── global_exception_handler.py
│   ├── middleware/
│   └── schemas/
│       └── error_response.py
└── domains/
    ├── user/
    │   ├── models.py         # SQLModel 엔티티
    │   ├── schemas.py        # Pydantic 스키마
    │   ├── service.py        # 비즈니스 로직
    │   ├── repository.py     # 데이터 접근 계층
    │   └── dependencies.py   # DI 의존성
    └── sample/
        ├── models.py
        └── schemas.py
```

## 아키텍처

```
Router (API Layer)
  └─ Service (Business Logic)
       └─ Repository (Data Access)
            └─ Database (SQLAlchemy Async)
```

- **레이어드 아키텍처**: Router → Service → Repository → DB
- **의존성 주입**: FastAPI `Depends()`를 활용한 자동 의존성 해결
- **Async-First**: 전체 DB 작업이 async/await 기반

## 주요 기능

### 사용자 관리 (CRUD)

| Method | Path | 설명 |
|--------|------|------|
| POST | `/api/v1/users` | 사용자 생성 |
| GET | `/api/v1/users/{user_id}` | 사용자 조회 |
| GET | `/api/v1/users` | 사용자 목록 조회 (필터/페이징) |
| PUT | `/api/v1/users/{user_id}` | 사용자 수정 |
| DELETE | `/api/v1/users/{user_id}` | 사용자 삭제 |

### Speech-to-Text (Whisper)

| Method | Path | 설명 |
|--------|------|------|
| POST | `/api/v1/whisper/transcribe` | 음성 파일 텍스트 변환 |
| POST | `/api/v1/whisper/transcribe-full` | 세그먼트 타이밍 포함 변환 |

지원 포맷: `mp3`, `wav`, `m4a`, `flac`, `ogg`, `mp4`, `avi`, `mov`

## 시작하기

### 사전 요구사항

- Python 3.12+
- Poetry
- Docker & Docker Compose (DB 인프라용)

### 설치

```bash
# 의존성 설치
poetry install

# 인프라 실행 (DB)
docker compose -f docker-compose.infra.yaml up -d
```

### 환경 변수 설정

`.env.local` 또는 `.env.dev` 파일을 프로젝트 루트에 생성합니다.

```env
APP_ENV=local
APP_NAME=fastapi-template
DEBUG=true
HOST=0.0.0.0
PORT=8000

# Database
DB_TYPE=postgresql+asyncpg
DB_HOST=localhost
DB_PORT=5432
DB_NAME=fastapi_db
DB_USERNAME=postgres
DB_PASSWORD=password

# Security
SECRET_KEY=your-secret-key-must-be-at-least-32-characters

# CORS
CORS_ORIGINS=["http://localhost:3000"]

# JWT
JWT_ALGORITHM=HS256
JWT_EXPIRE_MINUTES=60
```

### 실행

```bash
# APP_ENV 환경 변수로 설정 파일 선택 (기본값: local)
APP_ENV=local poetry run python -m fastapi_template.main
```

서버 실행 후:
- Swagger UI: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc

## 환경 설정

`APP_ENV` 환경 변수로 로드할 설정 파일을 지정합니다.

| APP_ENV | 설정 파일 |
|---------|----------|
| `local` | `.env.local` |
| `dev` | `.env.dev` |
| `prod` | `.env.prod` |

## 데이터베이스 지원

| DB | Driver | DB_TYPE 값 |
|----|--------|------------|
| PostgreSQL | asyncpg | `postgresql+asyncpg` |
| MySQL | aiomysql | `mysql+aiomysql` |
| SQLite | aiosqlite | `sqlite+aiosqlite` |

테이블은 앱 시작 시 자동으로 생성됩니다.

## 에러 응답 형식

모든 에러는 다음 형식으로 반환됩니다.

```json
{
  "errorCode": "NOT_FOUND",
  "path": "/api/v1/users/123",
  "trace": "..."
}
```

| HTTP Status | 설명 |
|-------------|------|
| 400 | 잘못된 요청 |
| 404 | 리소스 없음 |
| 422 | 유효성 검사 실패 |
| 500 | 서버 내부 오류 |
