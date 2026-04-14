# Backend Convention

이 문서는 `back/` 보일러플레이트를 기반으로 새 프로젝트를 시작할 때 따를 백엔드 규칙입니다. 예시 구현을 설명하는 문서가 아니라, 앞으로 유지할 구조와 책임 분리를 정의하는 문서로 봐야 합니다.

## Core Flow

이 프로젝트는 아래 흐름을 기본으로 둡니다.

- bootstrap: `main.ts` -> `app.factory.ts` -> `AppModule`
- request: `controller` -> `application/use-case` -> `domain` / `repository contract` -> `infrastructure implementation`
- success response: controller return value -> `ResponseInterceptor` -> success envelope
- error response: thrown error -> `HttpExceptionFilter` -> error envelope
- config: `environment.validation.ts` -> `registerAs()` config files -> `ConfigService<AllConfigType>`

## Root Structure

`src` 최상위는 아래 역할로 나뉩니다.

- `src/main.ts`
  프로세스 시작점입니다. 실제 부팅 로직은 여기서 직접 작성하지 않고 `app.factory.ts`로 위임합니다.
- `src/app.factory.ts`
  앱 생성과 공통 부트스트랩을 담당합니다. logger, CORS, global prefix, `ValidationPipe`, `HttpExceptionFilter`, `ResponseInterceptor` 같은 전역 설정은 여기서 연결합니다.
- `src/app.module.ts`
  루트 모듈입니다. 전역 공통 모듈과 feature module을 조립합니다. 프로젝트에서 사용하는 도메인 모듈과 인프라 모듈은 여기 imports에 등록합니다.
- `src/app.controller.ts`, `src/app.service.ts`
  시스템 엔드포인트 전용입니다. `/health` 같은 시스템 체크만 두고, 일반 도메인 엔드포인트는 feature module로 분리합니다.
- `src/common`
  HTTP 전역 규칙 계층입니다. CORS 옵션, 공통 예외, 공통 응답 인터셉터를 둡니다.
- `src/config`
  환경변수 기반 설정과 검증을 둡니다. `registerAs()` config 파일, config 합성 타입, startup validation spec이 여기 있습니다.
- `src/core`
  앱 전역에서 공유하는 기반 모듈을 둡니다. `ConfigModule`, `HttpClientModule`, `LoggerModule` 같은 공통 기반은 `CoreModule`이 묶습니다.
- `src/infra`
  기술 인프라 계층입니다. rate limiter, database 같은 기술 모듈은 여기서 관리하고, 프로젝트에서 사용하는 인프라는 `AppModule`에 연결합니다.
- `src/modules`
  실제 기능 모듈 계층입니다. 규칙의 기준은 특정 예시 모듈 이름이 아니라 계층 구조 자체입니다.
- `src/utils`
  bootstrap 또는 config에서 재사용하는 작은 유틸을 둡니다.

## Bootstrap Rules

- `main.ts`는 `createApp()`과 `startApp()`만 호출합니다.
- `createApp()`은 `NestFactory.create()`와 공통 앱 설정을 담당합니다.
- `startApp()`은 host, port, prefix를 읽고 listen 및 최종 URL 로깅을 담당합니다.
- 부트스트랩 관련 global 설정은 `main.ts`가 아니라 `app.factory.ts`에 모읍니다.

새 전역 pipe/filter/interceptor/adapter 설정은 `main.ts`에 흩뿌리지 말고 `app.factory.ts`에 추가합니다.

## Common HTTP Layer

`src/common`은 기능 모듈 공용의 HTTP 규칙을 둡니다.

### CORS

- `src/common/cors/cors-options.ts`는 `ConfigService<AllConfigType>`를 받아 Nest CORS 옵션으로 변환합니다.
- 실제 원본 값은 `src/config/cors.config.ts`에서 읽습니다.
- CORS 허용 목록 해석은 여기서 끝내고, controller나 module에서 개별 CORS 결정을 하지 않습니다.

### Success Response

- `src/common/interceptors/response.interceptor.ts`가 모든 성공 응답을 아래 형태로 감쌉니다.

```ts
{
  statusCode: number;
  success: true;
  data: T;
}
```

- controller는 이 envelope를 직접 만들지 않습니다.
- controller는 raw DTO 또는 response DTO 배열만 반환합니다.
- 요청 시간 로깅도 같은 interceptor에서 수행합니다.

### Error Response

- `src/common/exceptions/http-exception.filter.ts`가 모든 예외를 공통 에러 포맷으로 정리합니다.
- 에러 응답은 아래 필드를 가집니다.

```ts
{
  statusCode: number;
  success: false;
  code: string;
  message: string | string[];
  error: string;
  traceId: string;
}
```

- `HttpException`은 status/message를 유지합니다.
- `DomainValidationError`는 `400 BAD_REQUEST`로 정규화됩니다.
- 기타 예상하지 못한 에러는 `500 INTERNAL_SERVER_ERROR`로 정리됩니다.

즉, feature module은 자체 응답 envelope를 다시 만들지 말고, 공통 성공/에러 포맷을 그대로 탑니다.

## Config Rules

설정 계층은 `ConfigModule + registerAs + startup validation` 조합을 기본으로 둡니다.

- `src/core/core.module.ts`에서 `ConfigModule.forRoot()`를 전역으로 초기화합니다.
- 실제 config 파일은 `src/config/*.config.ts`에 둡니다.
- `app`, `cors`, `throttler`, `database`처럼 도메인과 무관한 전역 설정은 여기서 관리합니다.
- 환경변수 검증은 `src/config/environment.validation.ts`에서 한 번에 수행합니다.
- config 접근은 `ConfigService<AllConfigType>`를 기본으로 합니다.

- env parsing과 validation은 DTO/class-validator가 아니라 수동 검증 함수로 처리합니다.
- `API_PREFIX`는 `normalizeApiPrefix()`를 거쳐 `/api/v1/` 같은 입력도 `api/v1` 형태로 정규화됩니다.
- 빈 문자열 prefix도 허용합니다.
- `.env.example`과 config default 값은 항상 같이 움직여야 합니다.

## Core Layer Rules

`src/core`는 앱 전역 공용 기반 모듈을 둡니다.

### CoreModule

- `CoreModule`은 `@Global()`입니다.
- 설정 초기화, HTTP 클라이언트, 로거를 한곳에서 묶습니다.
- 여러 feature module에서 반복 import하지 않는 공통 기반은 여기에 둡니다.

### Http Client

- 외부 HTTP 호출은 `src/core/http-client`를 통해 수행하는 것을 기본으로 둡니다.
- `HttpClientService`는 `HttpService`를 Promise 기반으로 감싼 thin wrapper입니다.
- timeout, redirect, 기본 헤더 같은 공통 설정은 module registration에서 관리합니다.

### Logger

- 공통 로그는 `src/core/logger/logger.service.ts`를 사용합니다.
- 개발 환경은 console transport, 운영 환경은 daily rotate file transport를 사용합니다.
- 로그 디렉토리 생성, 로그 포맷, level 기준은 logger service 내부에서 통일합니다.

## Infra Layer Rules

`src/infra`는 프레임워크 바깥 기술 인프라를 둡니다.

### Rate Limiter

- `src/infra/rate-limiter/rate-limiter.module.ts`에서 전역 rate limiting을 관리합니다.
- `APP_GUARD`로 `ThrottlerGuard`를 전역 등록합니다.
- 기본 rate limit 값은 `src/config/throttler.config.ts`에서 가져옵니다.

### Database

- `src/infra/database`는 DB 인프라 확장 지점입니다.
- `DatabaseModule`, `PgModule`, `PgPoolProvider`, `withTransaction()` 유틸이 준비돼 있습니다.
- PostgreSQL을 사용하는 프로젝트에서는 `DatabaseModule`을 `AppModule`에 연결합니다.
- DB를 사용하더라도 domain/application 계층 계약은 유지하고, 저장소 구현만 infrastructure에서 교체합니다.

## Feature Module Structure

기능 모듈은 `src/modules/<feature>` 아래에서 아래 구조를 기본으로 둡니다.

```text
<feature>/
├── <feature>.module.ts
├── application/
├── domain/
│   ├── entities/
│   └── repositories/
├── infrastructure/
│   └── persistence/
└── presentation/
    ├── controllers/
    └── dto/
```

예시 모듈이 있더라도 규칙의 핵심은 이름이 아니라 이 계층 구조를 유지하는 데 있습니다.

## Layer Responsibilities

### Presentation

`presentation`은 HTTP 입구와 응답 shape를 담당합니다.

- controller는 `Param`, `Query`, `Body`를 받고 use case를 호출합니다.
- 입력 검증은 DTO + global `ValidationPipe`에 맡깁니다.
- 출력 shape는 `ResponseDto.from(entity)` 패턴으로 맞춥니다.
- 비즈니스 로직, 저장 로직, 도메인 규칙은 직접 두지 않습니다.

- request DTO는 `class-validator`로 요청 값을 검증합니다.
- response DTO는 domain/application 결과를 HTTP 응답용 shape로 변환합니다.

### Application

`application`은 유스케이스 중심 orchestration 계층입니다.

- 파일명은 `<verb>-<feature>.use-case.ts` 형태를 유지합니다.
- 클래스명은 `GetXxxUseCase`, `ListXxxUseCase`, `UpdateXxxUseCase`처럼 시나리오를 드러냅니다.
- public 진입 메서드는 `execute()`를 기본으로 둡니다.
- repository 조회, 존재 여부 확인, 도메인 메서드 호출, 저장 순서를 조합합니다.
- HTTP 데코레이터나 응답 포맷은 여기서 다루지 않습니다.

### Domain

`domain`은 비즈니스 규칙과 저장소 계약을 둡니다.

- entity는 Nest decorator 없이 plain TypeScript class로 유지합니다.
- 상태 변경 규칙과 invariant는 entity 메서드 안에서 지킵니다.
- 규칙 위반은 `DomainValidationError` 같은 명시적 도메인 예외를 사용합니다.
- repository contract는 `interface`가 아니라 `abstract class`를 기본으로 둡니다.
- repository contract는 필요에 따라 read와 command를 분리합니다.

### Infrastructure

`infrastructure`는 domain contract의 기술 구현입니다.

- persistence 구현은 `infrastructure/persistence`에 둡니다.
- 외부 검색/메시지 브로커/서드파티 저장소가 붙으면 목적별 하위 폴더를 추가합니다.
- infrastructure는 domain repository contract를 구현하지만, presentation 규칙은 알지 못해야 합니다.

- 하나의 구현체가 여러 repository contract를 만족하면 `useExisting`으로 같은 구현체에 바인딩합니다.

## Dependency Direction

의존 방향은 항상 아래로만 흐르게 유지합니다.

- `presentation` -> `application`
- `application` -> `domain`
- `infrastructure` -> `domain`
- `app.module` -> `core`, `infra`, `modules`

지키지 말아야 할 방향:

- `domain` -> `presentation`
- `domain` -> `@nestjs/*`
- `application` -> HTTP response formatting
- `controller` -> DB/ORM client 직접 호출

## Module Wiring Rules

기능 모듈 wiring은 `<feature>.module.ts`에서 끝내는 구조를 기본으로 둡니다.

- controller 등록
- use case provider 등록
- repository contract와 구현체 바인딩
- 필요한 use case export

- concrete provider는 infrastructure에 둡니다.
- repository contract binding은 feature module에서 마무리합니다.
- 여러 contract가 같은 구현체를 가리키면 `useExisting`을 사용합니다.

## DTO and Validation Rules

- request DTO는 `presentation/dto`에 둡니다.
- 입력 검증은 `class-validator` decorator로 선언합니다.
- global validation은 `src/utils/validation-options.ts`에서 통일합니다.
- `transform`, `whitelist`, `forbidNonWhitelisted`, `enableImplicitConversion` 옵션을 유지합니다.

즉:

- DTO에 없는 값은 제거되거나 거부됩니다.
- 숫자/불리언 같은 타입은 가능한 범위에서 자동 변환됩니다.
- controller마다 별도 `ValidationPipe`를 다시 두지 않습니다.

## Testing Rules

- `src/**/*.spec.ts`
  entity, config normalizer, env validator, controller 같은 단위 테스트를 둡니다.
- `test/app.e2e-spec.ts`
  실제 HTTP 계약을 검증하는 e2e 테스트용입니다.

- `AppController`의 시스템 엔드포인트 계약
- domain entity의 규칙
- `normalizeApiPrefix()`의 config 정규화
- `validateEnvironment()`의 startup validation

새 기능을 추가할 때도 최소한 아래 셋은 유지하는 것이 맞습니다.

- domain invariant 단위 테스트
- use case 또는 controller 단위 테스트
- 실제 HTTP 기준 e2e 또는 통합 테스트

## Naming and File Rules

- use case 파일은 `*.use-case.ts`
- entity 파일은 `*.entity.ts`
- request/response DTO는 `presentation/dto`
- repository contract는 `domain/repositories`
- repository 구현은 `infrastructure/persistence`
- 모듈 조립 파일은 `<feature>.module.ts`

기능 이름은 가능한 한 실제 도메인 의미가 드러나게 짓습니다. 임시 예시 이름은 빠르게 실제 도메인 이름으로 치환합니다.

## Extension Workflow

새 기능 모듈을 붙일 때는 아래 순서를 권장합니다.

1. `src/modules/<feature>` 아래에 모듈 골격을 만든다.
2. `domain`에 entity와 repository contract를 정의한다.
3. `application`에 use case를 작성한다.
4. `infrastructure`에 persistence 구현체를 만든다.
5. `presentation`에 controller와 DTO를 만든다.
6. `<feature>.module.ts`에서 provider binding을 마무리한다.
7. `AppModule` imports에 기능 모듈을 연결한다.
8. domain/use-case/http 테스트를 추가한다.

DB를 실제로 연결하는 경우에는 이 순서 사이에서 `DatabaseModule` 연결과 provider 구현 교체가 추가됩니다.

## Do Not

- controller에 비즈니스 로직을 직접 넣지 않습니다.
- domain 계층에 Nest decorator, DTO, ORM entity를 섞지 않습니다.
- controller나 use case에서 success/error envelope를 직접 만들지 않습니다.
- feature module이 전역 config 초기화 책임을 갖지 않습니다.
- 사용하지 않는 기술 구현을 루트 실행 경로에 억지로 포함하지 않습니다.
- 빈 폴더 구조를 규칙처럼 강제하지 않습니다.
