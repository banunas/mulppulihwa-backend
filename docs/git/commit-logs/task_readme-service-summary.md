# README에 서비스 요약 작성

**날짜**: 2026-09-17
**커밋 로그**: [c624174](https://github.com/mulppulihwa/backend/commit/c624174)

## 작업 배경

사용자가 "우리 서비스 요약해서 README.md 작성"을 요청. 기존 README.md는
`# backend` / `소셜벤처창업 프로젝트-백엔드` 두 줄뿐이라, 처음 코드를 보는
사람이 서비스가 뭘 하는지 파악할 방법이 없었음.

작업 디렉터리(`backend/`)는 `.gitignore`에 등록된 예전 잔여 폴더이고, 실제
Django 프로젝트(`apps/`, `config/`, `manage.py` 등)는 그 한 단계 위
`socialventure/` 저장소 루트에 있어서, 루트의 README.md를 갱신함.

`docs/architecture.md`, `docs/definition/problem-definition.md`,
`docs/plan/platform-plan.md`와 `apps/*/models.py`, `apps/*/urls.py`,
`config/settings.py` 등 실제 코드를 읽고 내용을 구성함.

## AS-IS

- README.md에 서비스 목적, 핵심 기능, 기술 스택, 실행 방법, API 목록 등
  아무 설명이 없었음

## TO-BE

- 서비스 개요 및 문제의식(귀농·귀향 초기 정착민의 정책 정보 격차) 추가
- 핵심 기능 정리: 맞춤 정책 매칭(1차/2차 필터링), 정책 자동 수집·AI 분석,
  카카오 로그인, 지역 사용처(신뢰 마크·추천), 구하기 게시판(일자리/주택),
  API 트래픽 관리자 대시보드
- 기술 스택 표, 프로젝트 구조(`apps/`, `lib/`, `config/`, `tests/`, `docs/`) 정리
- 로컬 개발 환경 설정: 필요 Python 버전, 설치 명령, 환경 변수 표(필수/선택 구분),
  실행·테스트 명령, 주요 management command 목록
- 주요 API 엔드포인트 요약 표, Swagger UI/OpenAPI 스키마 경로 안내
- 배포(Railway) 방식 및 관련 문서(`architecture.md`, `prd.md`,
  `platform-plan.md`, 커밋 로그 디렉터리) 링크 추가

## 주요 변경

- `README.md`: 전체 재작성 (2줄 → 서비스 요약 문서)

## 검증

- 코드 읽기 기반 요약이라 별도 실행 검증 대상 없음(문서 변경)
- 사용자에게 파일 경로 안내 후 커밋/푸시 승인받고 진행
