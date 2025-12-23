# AI Coding Spec (SDD) Template

> **Note**: This document is optimized for AI Coding Agents. It serves as the primary "Prompt" or "Context" for generating code.
> **Role**: [e.g., Senior Full Stack Engineer]
> **Task**: [e.g., Implement User Authentication Module]

## 1. Context & Objective
*Concise summary of what needs to be built and why.*
We are building a [Name] system. The goal of this task is to implement [Feature Name] which allows [User Capability].

## 2. Core Rules (Constitution)
*   **Language**: TypeScript, Python, etc.
*   **Frameworks**: Next.js 14 (App Router), FastAPI, etc.
*   **Styling**: Tailwind CSS.
*   **Testing**: Jest / Pytest.
*   **Conventions**: 
    - Use functional components.
    - Strict typing.
    - Error handling with specific custom exceptions.

## 3. Implementation Steps (Plan)
*Step-by-step instructions for the AI to follow.*

1.  **Setup**: Initialize project structure / install dependencies.
2.  **Database**: Create `User` model in `schema.prisma`.
3.  **API**: Implement `POST /auth/register` and `POST /auth/login`.
4.  **UI**: Create `SignUpForm` and `LoginForm` components.
5.  **Integration**: Connect UI to API.
6.  **Tests**: Write unit tests for API and UI components.

## 4. Technical Specifications
### 4.1 Data Structures
```typescript
interface User {
  id: string;
  email: string;
  role: 'ADMIN' | 'USER';
  createdAt: Date;
}
```

### 4.2 API Contract
- **POST /api/auth/login**
    - Input: `{ email, password }`
    - Output: `{ token, user }`
    - Errors: 401 Invalid credentials.

### 4.3 Component Interface
`<LoginForm onSuccess={(user) => void} />`

## 5. Verification Steps
*How the AI should verify its work.*
- Run `npm run test` and ensure all tests pass.
- Verify that the login flow redirects to the dashboard.
- Check that the JWT token is stored in HttpOnly cookies.

## 6. Reference Files
*List specific files to read for context (if using an agent).*
- `@/lib/db.ts`
- `@/components/ui/button.tsx`
