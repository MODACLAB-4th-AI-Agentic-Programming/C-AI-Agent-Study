---
name: front-architecture-rules
description: Use when adding pages, routes, layout changes, shared UI, or placing files under front/src.
---

# Front Architecture Rules

## Core Flow

Always follow:

- route-config -> menu -> layout
- siteConfig -> layoutPreset -> Layout
- pages/sections -> queries -> apis -> axiosInstance

---

## Layer Responsibilities

- apis -> pure API functions
- queries -> React Query hooks + query keys
- pages -> composition only
- components/ui -> primitives only (no domain logic)
- components/layout -> app shell
- components/menu -> navigation
- store -> global UI state only

---

## Placement Rules

### Pages

Always:

- create pages at `src/pages/<name>/index.tsx`
- keep `index.tsx` as the page composer
- keep pages composition-only
- split page-specific sections inside the same folder

Never:

- implement layout/shell logic inside pages
- call `apis` directly from pages or page sections

---

### Navigation

Always:

- define navigation via `route-config`

Never:

- hardcode menu separately
- duplicate route/meta/menu definitions

---

### Shared UI

Place components based on responsibility:

- primitive -> `components/ui`
- shell -> `components/layout`
- navigation -> `components/menu`
- page-specific -> inside page folder

Never:

- move domain-specific UI into `components/ui`

---

### Data Flow

Always:

- define API functions in `apis/<domain>`
- define React Query hooks in `queries/<domain>`
- use `queries` in pages, page sections, and UI that consumes server state

Never:

- call APIs directly from UI
- import `apis` directly in pages, page sections, or shared UI

---

## Source of Truth

Always:

- derive menu from route-config
- keep SEO metadata in route-config
- define layout defaults in siteConfig

Never:

- duplicate route-related data
- define menu outside route system

---

## Extension Workflow

Always:

1. identify correct layer
2. reuse existing structure
3. apply minimal change
4. promote to shared only if reused

---

## Avoid

- page-level shell implementation
- domain logic in `components/ui`
- duplicated routing/menu structures
- creating unused structures
