---
name: back-coding-convention
description: Use when writing or modifying backend code under back/src.
---

# Back Coding Convention

## Core Rules

Always:
- follow existing NestJS + TypeScript patterns
- keep controllers thin
- use DTOs for request input and response output where needed
- use the application/use-case layer for business logic
- let the global interceptor handle response shape

Never:
- write business logic in controllers
- manually wrap responses
- bypass global validation or filters

---

## Formatting

Always:
- use 2-space indentation
- use semicolons

Never:
- mix formatting styles in a file

---

## Imports

Always:
- keep external imports first
- use `@/` alias across backend modules
- use `import type` when applicable

Prefer:
- relative imports inside the same feature module

---

## Exports

Prefer:
- named export for use cases, utils, constants, and providers
- default export for config files registered with `registerAs()`

---

## Controllers

Always:
- use `@Param`, `@Query`, `@Body`
- return DTOs or simple system payloads
- delegate to use cases

Never:
- call repositories directly
- include business rules
- shape HTTP response manually

---

## DTO Rules

Always:
- place DTOs under `presentation/dto`
- validate request DTOs with `class-validator`
- map response DTOs from domain/application results

Never:
- use DTOs in domain
- add validation decorators to domain entities

---

## Use Case Rules

Always:
- name files as `*.use-case.ts`
- expose `execute()`
- orchestrate domain + repository contracts

Never:
- depend on the HTTP layer
- depend on concrete DB implementations

---

## Domain Rules

Always:
- keep domain as plain TypeScript
- enforce invariants in entities
- throw explicit domain errors
- define repository contracts as abstract classes

Never:
- import framework code
- use DTOs or ORM-specific code
- move business rules outside domain

---

## Infrastructure Rules

Always:
- implement repository contracts
- keep persistence under `infrastructure/persistence`
- bind contracts in the feature module

Never:
- leak DB logic into domain or controller

---

## External IO

Always:
- use the shared logger
- use `core/http-client` for outbound HTTP

Never:
- create custom axios/fetch wrappers per feature
- duplicate logging setup

---

## Testing Rules

Always:
- place tests as `*.spec.ts`
- test domain rules
- test config normalization and env validation when changed

Prefer:
- add HTTP-level tests for public APIs

---

## Quick Checklist

Before finishing:

- controller thin?
- correct layer used?
- no direct DB access from controller?
- DTO validation present?
- response not manually wrapped?
- tests updated?

---

## Avoid

- fat services replacing use cases
- mixing config logic into business code
- skipping validation
- unnecessary abstraction
