# AGENTS.md

이 문서는 이 프로젝트를 작업할 때 가장 먼저 보는 루트 인덱스입니다. 프로젝트 맥락, 영역별 규칙, 스킬, 자주 수정하는 위치를 빠르게 찾기 위한 목차로 사용합니다.

## Project Overview

- 이 저장소는 `front`와 `back`이 분리된 풀스택 보일러플레이트입니다.
- 프로젝트별 맥락은 `PROJECT.md`에, 구현 규칙은 각 영역의 `CONVENTION.md`와 `.agents/skills`에 둡니다.
- 새 프로젝트를 시작할 때는 보일러플레이트 규칙과 프로젝트 자체 설명을 분리해서 관리합니다.

## Reading Order

작업을 시작할 때는 아래 순서로 읽습니다.

1. `PROJECT.md`
2. 작업 대상 영역의 `CONVENTION.md`
3. 작업 대상 영역의 `.agents` 스킬

## Core Rules

Always:

- 기존 구조와 패턴을 먼저 재사용한다
- 구현 전에 관련 문서와 스킬을 먼저 확인한다
- 최소 변경 원칙을 지킨다
- 변경된 파일과 이유를 마지막에 요약한다

Never:

- 관련 없는 파일을 함께 수정한다
- 기존 구조로 해결 가능한데 새 패턴을 만든다
- 사용하지 않는 코드나 빈 구조를 남긴다

## Reference Map

### Project

- 프로젝트 설명: `docs/PROJECT.md`

### Frontend

- 규칙 문서: `front/docs/CONVENTION.md`
- 구조 스킬: `.agents/skills/front-architecture-rules/SKILL.md`
- 코딩 스킬: `.agents/skills/front-coding-convention/SKILL.md`
- 실행/개요: `front/README.md`

### Backend

- 규칙 문서: `back/docs/CONVENTION.md`
- 구조 스킬: `.agents/skills/back-architecture-rules/SKILL.md`
- 코딩 스킬: `.agents/skills/back-coding-convention/SKILL.md`
- 실행/개요: `back/README.md`

## Required Skills

프론트 작업 시 항상 적용:

- `front-architecture-rules`
- `front-coding-convention`

백엔드 작업 시 항상 적용:

- `back-architecture-rules`
- `back-coding-convention`

## Agent Responsibilities

### Core Agents

- `code-mapper`
  구현 전에 코드 구조와 영향 범위를 먼저 파악해야 할 때 사용합니다. 코드를 수정하지 않습니다.
- `task-distributor`
  큰 작업을 하위 작업으로 나누고 ownership을 분리해 병렬 진행해야 할 때 사용합니다.
- `frontend-developer`
  프론트 UI와 클라이언트 로직 구현이 필요할 때 사용합니다.
- `backend-developer`
  백엔드 API와 서버 비즈니스 로직 구현이 필요할 때 사용합니다.
- `code-reviewer`
  correctness뿐 아니라 유지보수성, 구현 품질, 위험한 선택까지 넓게 리뷰해야 할 때 사용합니다.
- `architect-reviewer`
  모듈 경계, 결합도, 장기 유지보수성, 설계 일관성을 구조 관점에서 점검해야 할 때 사용합니다.

### Specialized Agents

- `product-manager`
  무엇을 지금 만들고 무엇을 미룰지, 범위와 acceptance criteria를 먼저 정리해야 할 때 사용합니다.
- `project-manager`
  여러 작업 흐름의 의존성, 단계, 리스크, 전달 순서를 관리해야 할 때 사용합니다.
- `ui-designer`
  구현 전에 구체적인 UI 구조, 상호작용, 상태 표현을 정리해야 할 때 사용합니다. 코드를 직접 수정하지 않습니다.
- `api-designer`
  API 계약, request/response schema, 호환성, 버전 전략을 구현 전에 정리해야 할 때 사용합니다.
- `postgres-pro`
  PostgreSQL 스키마, 쿼리 성능, 인덱스, 락, 트랜잭션, 마이그레이션을 깊게 검토해야 할 때 사용합니다.
- `seo-specialist`
  crawlability, metadata, canonical, rendering, 정보 구조 등 기술 SEO 관점 검토가 필요할 때 사용합니다.

## Workflow Rules

기본 작업 흐름은 아래 순서를 따릅니다.

1. 요구사항 이해
2. 관련 문서와 스킬 확인
3. 핵심 agent로 구조 분석과 작업 분해 (`code-mapper`, `task-distributor`)
4. 필요 시 범위/우선순위/일정 정리 (`product-manager`, `project-manager`)
5. 담당 영역 구현 (`frontend-developer`, `backend-developer`)
6. 필요 시 전문 검토 (`ui-designer`, `api-designer`, `postgres-pro`, `seo-specialist`)
7. 결과 검토 (`code-reviewer`, `architect-reviewer`)
8. 변경 요약

구체적인 agent 선택은 작업 범위와 복잡도에 따라 조정합니다.

## Common Conventions

- TypeScript 기반이며 경로 별칭 `@`를 사용합니다.
- `front` 앱 코드는 4-space 들여쓰기를 기본으로 둡니다.
- `back` 코드는 2-space 들여쓰기와 세미콜론을 기본으로 둡니다.
- 프론트와 백엔드는 독립 배포를 전제로 작성합니다.
- 샘플/예시 텍스트는 프로젝트 시작 시 실제 도메인 내용으로 교체합니다.

## Front Quick Map

- 앱 진입점: `front/src/main.tsx`
- 라우트/SEO 메타: `front/src/routes/route-config.tsx`
- 사이트 설정/레이아웃 프리셋: `front/src/config/site.ts`
- 공통 셸: `front/src/components/layout/Layout.tsx`
- 공용 UI primitive: `front/src/components/ui`
- 페이지: `front/src/pages`
- 서버 데이터 계층: `front/src/queries`, `front/src/apis`

## Back Quick Map

- 프로세스 시작: `back/src/main.ts`
- 전역 부트스트랩: `back/src/app.factory.ts`
- 루트 모듈 조립: `back/src/app.module.ts`
- 공통 응답 규칙: `back/src/common/interceptors/response.interceptor.ts`
- 공통 예외 규칙: `back/src/common/exceptions/http-exception.filter.ts`
- 설정: `back/src/config`
- 공통 기반 모듈: `back/src/core`
- 기술 인프라: `back/src/infra`
- 기능 모듈: `back/src/modules`

## Quick Change Map

- 프로젝트 목적/대상 사용자/핵심 기능
    - `PROJECT.md`
- 프론트 사이트 메타/레이아웃 프리셋
    - `front/src/config/site.ts`
- 프론트 새 페이지 추가
    - `scripts/create-page`
    - `front/src/routes/route-config.tsx`
- 프론트 공통 셸 수정
    - `front/src/components/layout`
    - `front/src/components/menu`
- 프론트 서버 데이터 연결
    - `front/src/queries`
    - `front/src/apis`
- 백엔드 새 기능 모듈 추가
    - `scripts/create-module`
    - `back/src/app.module.ts`
- 백엔드 응답 필드/DTO 수정
    - `back/src/modules/<feature>/presentation/dto`
    - `back/src/modules/<feature>/presentation/controllers`
- 백엔드 설정/prefix/env 검증
    - `back/src/config`
    - `back/.env.example`
- 백엔드 공통 응답/예외 규칙
    - `back/src/common/interceptors/response.interceptor.ts`
    - `back/src/common/exceptions/http-exception.filter.ts`
