---
name: frontend-api-service-builder
description: >
  Create frontend API service classes, the useApiCall hook, and the ApiError class
  following the composition pattern. Use when adding new API endpoints, creating service files,
  setting up API error handling, or migrating existing services from BaseAxiosRequest inheritance.
  Covers frontend/src/api/, frontend/src/hooks/, and related types.
---

# API Service Builder

Build frontend API service classes, the `useApiCall` hook, and the `ApiError` error class following the composition pattern.

## When This Skill MUST Be Used

**ALWAYS invoke this skill when the user's request involves ANY of these:**

- Creating a new API service file in `frontend/src/api/`
- Adding the `useApiCall` hook to the project or a component
- Creating or modifying the `ApiError` class
- Migrating an existing service from `BaseAxiosRequest` inheritance to composition
- Adding new API endpoint methods to an existing service
- Setting up the API foundation (bootstrap) for the first time
- Typing API response interfaces

**If the user mentions "API service", "useApiCall", "ApiError", "endpoint", or "service class" in the context of the frontend — use this skill.**

## Critical Safety Rules

**NEVER remove or modify the JWT auth interceptors in `frontend/src/api/axios.config.ts`.**

The `apiClient` in `axios.config.ts` contains critical authentication logic (token attachment, 401 refresh, logout redirect). Breaking this will lock all users out of the application.

**NEVER delete existing service files during migration** — create the new version alongside, verify it works, then remove the old one.

**NEVER hardcode API base URLs** — always use the configured `apiClient` which inherits `/api` base URL and auth interceptors.

```
frontend/src/api/
├── baseRequest.ts        # LEGACY — old JWT auth base class (DO NOT use for new services)
├── axios.config.ts       # PROTECTED — apiClient instance + ApiError class + BaseResponse
├── *.service.ts          # Service files (safe to create/modify)
└── index.ts              # API aggregator (safe to update)
```

**What IS safe:**
- Creating new `*.service.ts` files
- Adding methods to existing services
- Creating `useApiCall` hook in `frontend/src/hooks/`
- Updating `frontend/src/api/index.ts` to add new services
- Adding response types to service files or `frontend/src/app/support/types.ts`

## Architecture / Overview

The API layer uses a **composition pattern** where services import a shared Axios client rather than extending a base class:

| Component | Purpose | Location |
|-----------|---------|----------|
| **apiClient** | Shared Axios instance with auth interceptors | `frontend/src/api/axios.config.ts` |
| **ApiError** | Typed error class with private constructor + static factory | `frontend/src/api/axios.config.ts` |
| **BaseResponse** | Common response shape (`message`, `status`) | `frontend/src/api/axios.config.ts` |
| **Service classes** | Domain-specific API methods (arrow function properties) | `frontend/src/api/<domain>.service.ts` |
| **API aggregator** | Single `api` object exporting all services | `frontend/src/api/index.ts` |
| **useApiCall hook** | React hook for loading/error/data state management | `frontend/src/hooks/useApiCall.ts` |
| **Response types** | Typed interfaces for API responses | Inline in service files or `frontend/src/app/support/types.ts` |

### How apiClient Works

`apiClient` is an Axios instance with `/api` base URL. It has two interceptors:
1. **Request**: reads `access_token` from Redux store → attaches as `Bearer` token
2. **Response**: on 401 → calls `/api/refresh-token` with cookie → updates store token → retries request. On refresh failure → dispatches `logout()` → redirects to `/login`

### Composition vs Inheritance

| Aspect | Old Pattern (inheritance) | New Pattern (composition) |
|--------|--------------------------|--------------------------|
| Base | `class X extends BaseAxiosRequest` | `class X` imports `apiClient` |
| HTTP calls | `this.baseRequest.get(...)` | `apiClient.get(...)` |
| Auth | Inherited from base class | Shared via imported `apiClient` |
| Import source | `baseRequest.ts` | `axios.config.ts` |

## Operations

### Operation 1: Bootstrap (First-Time Setup)

**Run this ONLY if `axios.config.ts` and `useApiCall.ts` don't exist yet.**

**Steps:**
1. Check if `frontend/src/api/axios.config.ts` exists
2. If missing, create it using the **api-axios-config** template
3. Check if `frontend/src/hooks/useApiCall.ts` exists
4. If missing, create it using the **use-api-call-hook** template
5. Ensure `frontend/src/api/index.ts` re-exports `BaseResponse`: `export type { BaseResponse } from './axios.config'`

**Important notes:**
- `BaseResponse` lives in `axios.config.ts` — `index.ts` only re-exports it with `export type { BaseResponse } from './axios.config'`
- Do NOT remove `baseRequest.ts` — the legacy `AuthService` still depends on plain `axios` calls
- The `useApiCall` hook must import `ApiError` from `'../api/axios.config'`

### Operation 2: Create a New API Service

**Steps:**
1. Determine the domain name (e.g., `products`, `machines`, `orders`)
2. Define response interfaces — each endpoint gets a typed interface extending `BaseResponse`
3. Create the service file at `frontend/src/api/<domain>.service.ts` using the **api-service-class** template
4. Export the singleton: `export const <domain>Service = new <ClassName>()`
5. Import and add the service to the `api` object in `frontend/src/api/index.ts`
6. Add shared types to `frontend/src/app/support/types.ts` if reused across services

**Important notes:**
- Service methods MUST be arrow functions (class properties), not regular methods — ensures correct `this` binding when passed as callbacks to `useApiCall`
- Always import `BaseResponse` from `'./axios.config'`, not from `'src/api'`
- Each method destructures `{ data }` from the Axios response and returns it directly
- Response interfaces stay local (no `export`) unless consumed by components
- Export param types and list item types if components need them

### Operation 3: Migrate an Existing Service

Convert a service from `BaseAxiosRequest` inheritance to the composition pattern.

**Steps:**
1. Read the existing service file completely
2. Note all methods, response types, and imports
3. Create the new version using the **api-service-class** template:
   - Remove `extends BaseAxiosRequest`
   - Replace `this.baseRequest` calls with `apiClient` calls
   - Change import to `import { apiClient, type BaseResponse } from './axios.config'`
   - Keep all method signatures and response types identical
4. Update imports in all files that use this service (Grep for `import.*from.*'src/api/<service>'`)
5. Verify the `api` object in `index.ts` still exports the service correctly

**Important notes:**
- Do NOT change method names or signatures — consuming code must not need updates
- Do NOT change response types — they're part of the contract
- Keep the old file until migration is verified

### Operation 4: Use useApiCall in a Component

**Steps:**
1. Import `useApiCall` from `src/hooks/useApiCall`
2. Import `api` from `src/api`
3. Destructure: `const { execute, data, isLoading, error } = useApiCall(api.<service>.<method>)`
4. Call `execute(args)` in event handlers or `useEffect`
5. Use `isLoading` for loading states, `error` for error display, `data` for rendering

**Important notes:**
- Use destructuring rename for clarity: `const { execute: fetchProducts } = useApiCall(...)`
- `execute` is a **stable function reference** (zero-dep `useCallback`) — safe to use as an effect dependency
- `execute` returns `T | null` — null on error
- Call `reset()` to clear state on unmount or navigation

## Configuration / Key Locations

```
frontend/src/
├── api/
│   ├── baseRequest.ts          # LEGACY — old JWT auth base class (DO NOT use for new services)
│   ├── axios.config.ts         # apiClient + ApiError + BaseResponse (source of truth)
│   ├── auth.service.ts         # Auth service — uses plain axios (not apiClient)
│   ├── users.service.ts        # User management service
│   ├── machines.service.ts     # Machine operations service
│   ├── bottles.service.ts      # Bottle management service
│   ├── products.service.ts     # Product management service
│   └── index.ts                # API aggregator (re-exports BaseResponse + api object)
├── hooks/
│   ├── useApiCall.ts           # Generic API call hook
│   └── ...
└── app/support/
    └── types.ts                # Shared type definitions
```

**Key behaviors:**
- `BaseResponse` is defined in `axios.config.ts` — imported from there by services; re-exported from `index.ts` for component use
- New services import `{ apiClient, type BaseResponse }` from `'./axios.config'`
- `AuthService` is a special case — uses plain `axios` (not `apiClient`) for login/refresh endpoints
- All services are aggregated in the `api` object in `index.ts`

## Troubleshooting

- **"apiClient is not defined"** — Ensure `axios.config.ts` exists and exports `apiClient`. Run Bootstrap first.
- **"401 errors on new service"** — Verify `apiClient` interceptors are intact in `axios.config.ts`. Never copy interceptor logic into a service.
- **"useApiCall not found"** — Ensure the hook exists at `frontend/src/hooks/useApiCall.ts`. Run Bootstrap first.
- **"Type mismatch on response"** — Check that the response interface extends `BaseResponse` and matches the actual API shape.
- **"Service not on api object"** — Import the service and add it to the `api` object in `frontend/src/api/index.ts`.
- **"execute causes re-renders"** — `execute` is stable (zero-dep useCallback via fnRef). Do not wrap it in another useCallback.

## Decision Framework

When the user requests API-related frontend changes:

1. **Need a new endpoint?** → Add a method to an existing service or create a new service (Operation 2)
2. **Need foundational setup (ApiError, useApiCall)?** → Run Bootstrap first (Operation 1)
3. **Want to modernize an existing service?** → Migrate it (Operation 3)
4. **Want to use an API call in a component?** → Show useApiCall usage (Operation 4)
5. **Need shared response types?** → Add to `frontend/src/app/support/types.ts`
6. **Unsure if foundation exists?** → Check for `axios.config.ts` and `useApiCall.ts` first

## Required Templates

This skill references the following templates for all code patterns:

| Template | Purpose | Location |
|----------|---------|----------|
| **api-axios-config** | `apiClient` instance + `ApiError` class + `BaseResponse` | `.claude/templates/api-axios-config.md` |
| **api-service-class** | Service class with typed arrow-function methods | `.claude/templates/api-service-class.md` |
| **use-api-call-hook** | `useApiCall` generic React hook with fnRef pattern | `.claude/templates/use-api-call-hook.md` |

**All code patterns MUST come from these templates.** Do not write inline code — reference the template instead.

## Example Requests

- "Create an API service for managing orders" → Operation 2: Create new service
- "Add the useApiCall hook to the project" → Operation 1: Bootstrap
- "Set up ApiError class" → Operation 1: Bootstrap
- "Migrate machines service to the new pattern" → Operation 3: Migrate
- "How do I use useApiCall in my component?" → Operation 4: Component usage
- "Add a new endpoint to products service" → Operation 2: Add method to existing service
- "Set up the API foundation" → Operation 1: Bootstrap
