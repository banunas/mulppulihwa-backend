# 귀농·귀촌 정책 매칭 플랫폼 (백엔드)

소셜벤처창업 프로젝트 — 귀농·귀향 초기 정착 주민에게 **본인이 받을 수 있는 지원 정책을
개인 조건에 맞춰 추천**하고, 지역(옥천군) 안에서 믿을 수 있는 사용처·일손/주택 정보를
연결해주는 서비스의 백엔드입니다.

## 왜 만들었나

귀농·귀향 초기 주민은 자신에게 해당되는 지원 정책이 있어도 신청 기간 내에 알지 못해
놓치는 경우가 많습니다. 정책은 존재하고 온라인에도 올라가 있지만, 개인의 상황(나이·직업·
소득·이주 시점 등)에 맞게 걸러서 전달해주는 구조가 없기 때문입니다. 이 서비스는 몇 가지
정보만 입력하면 신청 자격이 되는 정책만 골라 보여주는 걸 목표로 합니다.
(자세한 문제 정의: [`docs/definition/problem-definition.md`](docs/definition/problem-definition.md))

## 핵심 기능

- **맞춤 정책 매칭** — 나이·직업·소득·거주 이력 등으로 1차 필터링 후, 정책별 복합 조건
  (`condition_tree`)을 2차로 재평가해 실제 신청 가능한 정책만 최대 5개 추천. 옥천 지역
  큐레이션 정책을 최우선 노출.
- **정책 자동 수집·분석** — 복지로·그린대로(귀농센터) API에서 정책을 주기적으로 가져오고,
  Anthropic Claude로 공고문을 분석해 자격 조건·마감일·지원금을 구조화된 데이터로 저장.
  확신도가 낮은 결과는 관리자 검토 후 노출.
- **카카오 로그인** — 카카오 OAuth로 회원가입/로그인, JWT 발급.
- **지역 사용처(로컬 플레이스)** — 농자재·행정·생활 등 카테고리별 사용처 등록/조회, 주소→좌표
  변환(카카오 로컬 API), 신뢰 마크(안심 검증 큐레이션) 및 사용자 추천(endorsement).
- **구하기 게시판** — 농촌일손·주택수리 등 일자리 공고/지원, 임대 매물(주택) 등록·조회.
- **운영 대시보드** — Django Admin에 API 트래픽 요약(방문자 수, 인기 엔드포인트 등) 대시보드 제공.

## 기술 스택

| 영역 | 기술 |
|---|---|
| 언어/프레임워크 | Python, Django, Django REST Framework |
| 인증 | JWT (djangorestframework-simplejwt), 카카오 OAuth |
| DB | PostgreSQL (Supabase), Redis (캐시) |
| 정적 파일 | Whitenoise |
| 이미지 저장 | Cloudflare R2 (django-storages, S3 호환) |
| 외부 API | 카카오(로그인·로컬 검색), 복지로, 그린대로, Anthropic Claude |
| API 문서 | drf-spectacular (OpenAPI / Swagger UI) |
| 배포 | Railway (Gunicorn, Nixpacks) |

프론트엔드는 별도 저장소(React + Vite, Netlify 배포)이며 이 저장소는 백엔드만 포함합니다.
전체 아키텍처는 [`docs/architecture.md`](docs/architecture.md)에 다이어그램과 함께 정리되어
있습니다.

## 프로젝트 구조

```
config/            프로젝트 설정, URL 라우팅, DRF 예외 처리
apps/
  users/           카카오 로그인, 프로필, 내가 담은 정책(UserPolicy) 관리
  policies/        정책 모델·매칭·상세조회, 정책 파싱/수집 management command
  places/          지역 사용처 CRUD, 지오코딩, 신뢰 마크
  regions/         지역 코드 정보
  board/           구하기 게시판(일자리/주택)
  analytics/       API 요청 로깅 미들웨어 + 관리자 트래픽 대시보드
lib/
  matching/        정책 조건 트리 평가, 매칭 알고리즘
  parsing/         Claude 기반 정책/체크리스트 텍스트 구조화
  services/        지오코딩 등 외부 API 연동
  sync/            복지로·그린대로 수집 어댑터
tests/             pytest 기반 테스트
docs/              문제 정의, 기획, 아키텍처, 커밋 로그 등 프로젝트 문서
```

## 로컬 개발 환경 설정

### 요구 사항
- Python 3.12+
- PostgreSQL (Supabase 등)
- (선택) Redis — 없으면 로컬 메모리 캐시로 자동 대체

### 설치

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 환경 변수 (`.env`)

| 변수 | 필수 | 설명 |
|---|---|---|
| `SECRET_KEY` | ✅ | Django secret key |
| `DEBUG` | | `True`/`False` (기본 `False`) |
| `ALLOWED_HOSTS` | | 콤마로 구분 (기본 `localhost,127.0.0.1`) |
| `DATABASE_URL` | ✅ | PostgreSQL 연결 문자열 |
| `DIRECT_URL` | | pytest 실행 시 사용할 논-풀링 연결 문자열 |
| `REDIS_URL` | | 없으면 로컬 메모리 캐시 사용 |
| `KAKAO_CLIENT_ID` / `KAKAO_CLIENT_SECRET` / `KAKAO_REDIRECT_URI` | ✅ | 카카오 로그인 |
| `KAKAO_LOCAL_API_KEY` | | 지오코딩 폴백용 |
| `ANTHROPIC_API_KEY` | | 정책 AI 분석/파싱 |
| `BOKJIRO_API_KEY` | | 복지로 정책 수집 |
| `R2_ACCOUNT_ID` / `R2_ACCESS_KEY_ID` / `R2_SECRET_ACCESS_KEY` / `R2_BUCKET_NAME` / `R2_PUBLIC_URL` | | 이미지 저장(Cloudflare R2) |
| `CORS_ALLOWED_ORIGINS` / `CSRF_TRUSTED_ORIGINS` | | 프론트엔드 도메인 (기본 `http://localhost:3000`) |

### 실행

```bash
python manage.py migrate
python manage.py runserver
```

- API 문서(Swagger UI): `http://localhost:8000/api/docs/`
- 관리자 페이지: `http://localhost:8000/admin/`

### 테스트

```bash
pytest
```

### 주요 management command

- `sync_bokjiro` / `sync_greendaero` — 외부 정책 API에서 최신 공고 수집 (주기 실행)
- `parse_checklists` / `backfill_checklists` — 정책 준비물 체크리스트 AI 파싱/보정
- `prune_expired_policies` / `prune_policy_scope` — 만료·범위 밖 정책 정리
- `add_okcheon_policies` / `add_agri_policies` / `add_okcheon_places` — 옥천 지역 큐레이션 데이터 등록
- `regeocode_places` — 좌표 누락 사용처 재지오코딩

## API 개요

| 경로 | 설명 |
|---|---|
| `POST /api/auth/kakao/` | 카카오 로그인 |
| `POST /api/auth/token/refresh/` | JWT 재발급 |
| `GET/PATCH /api/profile/` | 내 프로필 조회/수정 |
| `GET /api/policies/match/` | 맞춤 정책 매칭 |
| `GET /api/policies/{id}/` | 정책 상세 |
| `GET /api/users/me/policies/` | 내가 담은 정책 목록/상태 관리 |
| `GET /api/places/` | 지역 사용처 목록/등록, 신뢰 추천 |
| `GET /api/board/jobs/`, `/api/board/housing/` | 구하기 게시판(일자리/주택) |
| `GET /api/regions/` | 지역 코드 목록 |

전체 스펙은 배포 후 `/api/docs/` (Swagger UI) 또는 `/api/schema/` (OpenAPI JSON)에서 확인할
수 있습니다.

## 배포

Railway에 Gunicorn으로 배포됩니다 (`Procfile` / `railway.json` 참고). 배포 시
`collectstatic → migrate → gunicorn` 순으로 실행됩니다.

## 더 알아보기

- [`docs/architecture.md`](docs/architecture.md) — 전체 시스템 아키텍처, 정책 수집/추천 흐름 다이어그램
- [`docs/plan/prd.md`](docs/plan/prd.md), [`docs/plan/platform-plan.md`](docs/plan/platform-plan.md) — 기획 문서
- [`docs/git/commit-logs/`](docs/git/commit-logs/) — 기능별 커밋 히스토리 문서
