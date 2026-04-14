---
name: front-coding-convention
description: Use when writing or modifying frontend code under front/src.
---

# Front Coding Convention

## Core Rules

Always:

- follow existing file patterns
- use Tailwind + semantic tokens
- define API calls in `apis/<domain>`
- use React Query for server state
- use Zustand for global UI state

Never:

- call APIs directly from components
- import `apis` in pages, page sections, or shared UI
- introduce new state libraries
- overuse inline styles

---

## Formatting

Always:

- keep existing file formatting

Never:

- reformat generated shadcn files
- mix formatting styles within a file

---

## Imports

Always:

- external imports first
- use `@/` alias for internal modules
- use `import type` for types

Prefer:

- relative imports only for nearby files

---

## Exports

Prefer:

- default export for components
- named export for store, utils, config, apis, and queries

---

## Components

Always:

- use functional components
- keep components focused
- keep pages composition-only

Never:

- use React.FC
- mix layout logic into pages

---

## Styling

Always:

- use Tailwind utilities
- use semantic tokens (`bg-background`, etc.)

Never:

- hardcode colors unnecessarily
- implement layout rules inside pages

---

## State Management

Always:

- use React Query for server data
- use Zustand for global UI state
- keep local state inside components
- let `queries` call `apis`

Never:

- mix server state with UI state
- call API directly from UI

---

## File Rules

### Pages

Always:

- place pages under `pages/<name>/index.tsx`
- keep `index.tsx` as the page composer
- place page-specific sections in the same page folder
- if server data is needed, use `queries` only

Never:

- import `apis` directly

---

### Data Layer

Always:

- keep API functions in `apis`
- keep query logic in `queries`

Never:

- mix responsibilities between layers

---

## Quick Checklist

Before finishing:

- following existing patterns?
- correct layer used?
- no duplicate logic?
- no direct API calls from UI?

---

## Avoid

- reimplementing layout in pages
- putting domain logic into `components/ui`
- duplicating axios/queryClient
- modifying generated code unnecessarily
