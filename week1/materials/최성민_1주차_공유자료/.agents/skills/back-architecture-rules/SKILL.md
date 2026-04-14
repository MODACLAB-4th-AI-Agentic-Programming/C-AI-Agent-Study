---
name: back-architecture-rules
description: Use when adding modules, controllers, use cases, domain entities, infrastructure, or changing structure under back/src.
---

# Back Architecture Rules

## Core Flow

Always follow:

- main.ts -> app.factory.ts -> AppModule
- controller -> use-case -> domain -> repository contract -> infrastructure
- controller return -> ResponseInterceptor
- thrown error -> HttpExceptionFilter
- env -> validation -> config -> ConfigService

---

## Layer Responsibilities

- presentation -> HTTP entry/exit only
- application -> use-case orchestration
- domain -> business rules + contracts
- infrastructure -> technical implementations

---

## Feature Module Structure

Always structure feature modules as:

- `<feature>.module.ts`
- `application`
- `domain/entities`
- `domain/repositories`
- `infrastructure/persistence`
- `presentation/controllers`
- `presentation/dto`

---

## Dependency Rules

Always:

- presentation -> application
- application -> domain
- infrastructure -> domain

Never:

- import presentation into domain
- import infrastructure into domain
- let application depend on HTTP

---

## Placement Rules

### Controller

Always:
- keep controllers thin
- validate via DTO
- delegate to use cases

Never:
- put business logic in controllers
- call DB directly

---

### Use Case

Always:
- expose `execute()`
- orchestrate domain + repository

Never:
- use HTTP decorators
- format responses

---

### Domain

Always:
- keep domain framework-free
- enforce invariants inside entities
- define repository contracts as abstract classes

Never:
- import `@nestjs/*`
- use DTOs or ORM code

---

### Infrastructure

Always:
- implement domain contracts
- keep persistence under `infrastructure/persistence`

Never:
- leak DB logic to domain or controller

---

## Config Rules

Always:
- define config in `src/config`
- validate env in `environment.validation.ts`
- use typed `ConfigService`

Never:
- parse env manually inside feature modules

---

## Bootstrap Rules

Always:
- configure global behavior in `app.factory.ts`

Never:
- duplicate bootstrap logic

---

## Extension Workflow

Always:

1. define domain entities/contracts
2. create use cases
3. implement infrastructure
4. add controller/DTO
5. wire module
6. register in `AppModule`

---

## Avoid

- fat controllers
- framework-dependent domain
- direct DB access from presentation
- duplicated response logic
- unused scaffolding
