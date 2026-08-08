# Full-Stack Coding Standards & Guidelines
**Stack:** React + TypeScript (Frontend) | Python + FastAPI (Backend)  
**Version:** 1.0.0  
**Last Updated:** August 2026  

---

## Table of Contents
1. [General Principles](#1-general-principles)
2. [Frontend Guidelines (React + TypeScript)](#2-frontend-guidelines-react--typescript)
   - [Project Structure](#21-project-structure)
   - [TypeScript Standards](#22-typescript-standards)
   - [React Component Architecture](#23-react-component-architecture)
   - [State Management & API Layer](#24-state-management--api-layer)
   - [Styling & UI Components](#25-styling--ui-components)
3. [Backend Guidelines (Python + FastAPI)](#3-backend-guidelines-python--fastapi)
   - [Project Structure](#31-project-structure)
   - [Type Hinting & Pydantic Schemas](#32-type-hinting--pydantic-schemas)
   - [FastAPI Route Organization & Dependency Injection](#33-fastapi-route-organization--dependency-injection)
   - [Database & ORM Practices](#34-database--orm-practices)
   - [Error Handling & Async Best Practices](#35-error-handling--async-best-practices)
4. [Cross-Cutting Concerns](#4-cross-cutting-concerns)
   - [API Contract & Schema Synchronization](#41-api-contract--schema-synchronization)
   - [Authentication & Authorization](#42-authentication--authorization)
   - [Testing Strategy](#43-testing-strategy)
   - [Environment Configuration](#44-environment-configuration)
   - [Git Workflow & Commit Conventions](#45-git-workflow--commit-conventions)

---

## 1. General Principles

1. **DRY (Don't Repeat Yourself):** Encapsulate reusable logic in hooks, utility functions, or middleware.
2. **KISS (Keep It Simple, Stupid):** Avoid premature optimization and complex abstractions until explicitly required.
3. **Type Safety Across the Stack:** Ensure strict typing end-to-end from Pydantic models in Python to TypeScript interfaces in React.
4. **Separation of Concerns:** Keep UI rendering, business logic, and API calls modular and decoupled.
5. **Fail Fast & Explicitly:** Validate inputs at system boundaries (FastAPI Pydantic schemas, React form validation) and throw descriptive errors.

---

## 2. Frontend Guidelines (React + TypeScript)

### 2.1 Project Structure
Use a feature-first or modular directory layout:

```text
src/
├── assets/             # Static assets (images, icons, global CSS)
├── components/         # Shared/UI atomic components (Buttons, Inputs, Modals)
│   ├── ui/
│   └── feedback/
├── features/           # Feature modules (domain-specific logic & UI)
│   └── auth/
│       ├── api/        # API queries/mutations for auth
│       ├── components/ # Feature-specific components
│       ├── hooks/      # Feature-specific hooks
│       ├── types/      # Feature-specific TypeScript interfaces
│       └── index.ts    # Barrel export for feature module
├── hooks/              # Shared custom React hooks
├── services/           # Axios/Fetch clients, API base instances
├── store/              # Global state (Zustand / Redux Toolkit / React Query)
├── types/              # Global TypeScript declarations
├── utils/              # Pure helper functions
├── App.tsx             # Main App entry with providers
└── main.tsx            # Vite/DOM mounting point
```

### 2.2 TypeScript Standards
- **Enable Strict Mode:** Set `"strict": true` in `tsconfig.json`.
- **No `any`:** Explicitly prohibit `any`. Use `unknown` when the type is unknown and narrow it, or use generics.
- **Interfaces vs. Types:**
  - Use `interface` for object structures and component props (enables declaration merging).
  - Use `type` for unions, primitives, tuples, or mapped types.
- **Explicit Return Types:** Declare return types on functions and hooks.

```typescript
// Good: Clear interface and prop typing
export interface UserCardProps {
  userId: string;
  name: string;
  email: string;
  isActive?: boolean;
  onSelect: (id: string) => void;
}

export const UserCard: React.FC<UserCardProps> = ({ userId, name, email, isActive = false, onSelect }) => {
  return (
    <div onClick={() => onSelect(userId)} className={isActive ? "active" : ""}>
      <h3>{name}</h3>
      <p>{email}</p>
    </div>
  );
};
```

### 2.3 React Component Architecture
- **Functional Components:** Use functional components with hooks exclusively.
- **Named Exports:** Prefer named exports (`export const MyComponent = ...`) over default exports for consistent imports and better refactoring support.
- **Component Granularity:** Keep components under ~150 lines. Extract sub-views or custom hooks when logic grows.
- **Hook Dependencies:** Always fulfill ESLint `react-hooks/exhaustive-deps` rules. Avoid disabling them without explicit documented rationale.

### 2.4 State Management & API Layer
- **Server State vs. UI State:**
  - Use **TanStack Query (React Query)** or **SWR** for asynchronous server data, caching, and synchronization.
  - Use **Zustand** or **Context API** for global client UI state (theme, active modal, user session).
  - Use `useState` for local component UI state.
- **API Call Abstraction:** Wrap API calls in custom hooks or dedicated service files.

```typescript
// api/useFetchUser.ts
import { useQuery } from '@tanstack/react-query';
import { apiClient } from '@/services/apiClient';
import { User } from '../types';

const fetchUser = async (id: string): Promise<User> => {
  const response = await apiClient.get<User>(`/users/${id}`);
  return response.data;
};

export const useFetchUser = (userId: string) => {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    enabled: Boolean(userId),
  });
};
```

### 2.5 Styling & UI Components
- **Framework Consistency:** Stick to a single UI ecosystem (e.g., Material UI / MUI, Tailwind CSS, or Shadcn UI).
- **Theme Variables:** Avoid hardcoding hex colors or pixel values in components. Utilize theme tokens (spacing, typography, color palette).
- **Class/Style Organization:** Keep CSS-in-JS or custom utility classes organized outside component render logic where possible.

---

## 3. Backend Guidelines (Python + FastAPI)

### 3.1 Project Structure
Use a clean domain/layered architecture:

```text
app/
├── api/
│   ├── v1/
│   │   ├── endpoints/
│   │   │   ├── auth.py
│   │   │   ├── users.py
│   │   │   └── items.py
│   │   └── router.py
│   └── deps.py          # FastAPI dependencies (auth, db session)
├── core/
│   ├── config.py        # Pydantic BaseSettings
│   ├── security.py      # JWT, password hashing
│   └── database.py      # SQLAlchemy engine / session setup
├── models/              # DB Models (SQLAlchemy / SQLModel)
│   ├── user.py
│   └── item.py
├── schemas/             # Pydantic models (DTOs)
│   ├── user.py
│   └── item.py
├── services/            # Business logic layer
│   ├── user_service.py
│   └── item_service.py
└── main.py              # FastAPI application instantiation
```

### 3.2 Type Hinting & Pydantic Schemas
- **Strict Typing:** Annotate all function signatures, return types, and variables using standard Python typing (`type | None`, `list[str]`, etc., Python 3.10+ syntax).
- **Pydantic V2 Models:** Use Pydantic models for request body validation, response serialization, and settings management.
- **Separate Request/Response Schemas:** Never expose database models directly in endpoints. Separate creation (`UserCreate`), response (`UserResponse`), and update (`UserUpdate`) schemas.

```python
# app/schemas/user.py
from pydantic import BaseModel, EmailStr, ConfigDict

class UserBase(BaseModel):
    email: EmailStr
    full_name: str | None = None

class UserCreate(UserBase):
    password: str

class UserResponse(UserBase):
    id: int
    is_active: bool

    model_config = ConfigDict(from_attributes=True)
```

### 3.3 FastAPI Route Organization & Dependency Injection
- **APIRouter:** Divide endpoints into clean module routers using `APIRouter`.
- **Dependency Injection:** Use `Depends()` for shared cross-cutting concerns like database sessions, current authenticated user, and permissions.

```python
# app/api/v1/endpoints/users.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession

from app.api.deps import get_db, get_current_user
from app.schemas.user import UserResponse
from app.services.user_service import UserService
from app.models.user import User

router = APIRouter(prefix="/users", tags=["Users"])

@router.get("/me", response_model=UserResponse)
async def read_current_user(
    current_user: User = Depends(get_current_user),
) -> User:
    return current_user

@router.get("/{user_id}", response_model=UserResponse)
async def get_user_by_id(
    user_id: int,
    db: AsyncSession = Depends(get_db),
) -> User:
    user = await UserService.get_by_id(db, user_id)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="User not found",
        )
    return user
```

### 3.4 Database & ORM Practices
- **Async ORM:** Use `asyncio`-native drivers (e.g., `asyncpg` with SQLAlchemy 2.0+ or SQLModel).
- **Migrations:** Use Alembic for database migrations. Never modify database schemas manually.
- **Session Lifecycle:** Manage sessions via scoped async generators passed to FastAPI dependency injection (`get_db`).

### 3.5 Error Handling & Async Best Practices
- **Custom Exceptions:** Raise `HTTPException` with explicit status codes and standardized JSON detail formats.
- **Async/Await:** Use `async def` for I/O bound endpoints (DB, external HTTP calls). For CPU-heavy tasks, offload to background tasks or Celery/Redis queues.

---

## 4. Cross-Cutting Concerns

### 4.1 API Contract & Schema Synchronization
- **OpenAPI Schema:** Leverage FastAPI's automatic OpenAPI spec generation (`/docs`, `/openapi.json`).
- **Code Generation:** Use tools like `openapi-typescript` or `orval` in the frontend build pipeline to automatically generate TypeScript interfaces and API client functions from FastAPI's OpenAPI JSON.

### 4.2 Authentication & Authorization
- **Token Format:** Standardize on OAuth2 with JWT Bearer tokens.
- **Frontend Storage:** Store JWT access tokens in secure HTTP-only cookies or memory (avoid `localStorage` for sensitive production session tokens where XSS is a concern).
- **Backend Middleware:** Validate tokens via central FastAPI `Depends(get_current_user)` dependencies.

### 4.3 Testing Strategy
- **Frontend:**
  - **Unit/Integration:** Vitest + React Testing Library for testing component rendering and user interaction.
  - **E2E:** Playwright or Cypress for end-to-end flows.
- **Backend:**
  - **Unit/Integration:** `pytest` with `httpx.AsyncClient` and an isolated test database (e.g., SQLite in-memory or PostgreSQL test container).
  - **Coverage:** Aim for minimum 80% test coverage on business logic layers.

### 4.4 Environment Configuration
- **Backend:** Use `pydantic-settings` to parse `.env` files into strongly typed config objects.
- **Frontend:** Use Vite standard environment variables (`VITE_API_BASE_URL`) accessed via `import.meta.env`.
- **Validation:** Do not boot the application if required environment variables are missing.

### 4.5 Git Workflow & Commit Conventions
- **Branch Naming:** `feature/feature-name`, `bugfix/issue-description`, `hotfix/critical-fix`.
- **Conventional Commits:**
  - `feat: add user authentication endpoint`
  - `fix: correct token expiry calculation`
  - `docs: update API setup instructions`
  - `refactor: extract shared modal component`
