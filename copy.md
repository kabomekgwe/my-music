# .augment-guidelines

## Project Overview
Full-stack application using Next.js (client-side only) and NestJS with enterprise-grade patterns.

## Tech Stack

### Frontend
- **Framework**: Next.js (latest) - Client-side rendering only, no SSR/RSC
- **Language**: TypeScript with strict mode
- **Styling**: Tailwind CSS + shadcn/ui components
- **State**: Zustand (global), TanStack Query (server state)
- **Forms**: react-hook-form + zod validation
- **Auth**: better-auth client SDK
- **Testing**: Jest + React Testing Library + Playwright
- **Linting**: Biome

### Backend
- **Framework**: NestJS (latest)
- **Database**: PostgreSQL with Prisma (latest)
- **Auth**: better-auth with NestJS integration
- **Logging**: Pino (include request_id, user_id, tenant_id in all logs)
- **File Storage**: Cloudinary, Minio, or S3
- **Queue**: Bull/BullMQ with Redis
- **Observability**: OpenTelemetry (vendor-agnostic)
- **Testing**: Jest + Supertest
- **Linting**: Biome

## Core Patterns

### Frontend Patterns
- Always use `'use client'` directive - no server components
- Route groups: `(auth)` for public, `(dashboard)` for authenticated
- Secure storage: Use sessionStorage only, never store sensitive data
- WCAG 2.1 AA compliance required
- Support last 2 versions of major browsers
- CSP headers and XSS prevention

### Backend Patterns
- REST API with kebab-case: `/api/v1/survey-templates/{id}`
- ULID for all primary keys
- Soft deletes with `deleted_at` column on all tables
- Audit logging for all create/update/delete operations
- Always include `created_at`, `updated_at`, `deleted_at` timestamps
- Validate all inbound data - never trust client
- Paginate lists: `?page=1&limit=20` (default limit=20, max=100)

### Authentication Setup
Use better-auth with these plugins:
- magic-link
- email-otp
- twoFactor
- admin
- organization
- passkey
- haveIBeenPwned
- oneTimeToken
- openAPI

Social providers: Google, Apple, Facebook

## Architecture & Project Structure

### Project Structure
```
my-music/
├── backend/          # NestJS API
│   ├── src/
│   │   ├── types/   # Backend-specific types and DTOs
│   │   ├── auth/
│   │   ├── users/
│   │   └── ...
│   ├── test/
│   ├── package.json
│   └── tsconfig.json
├── frontend/         # Next.js frontend
│   ├── src/
│   │   ├── types/   # Frontend-specific types and API interfaces
│   │   ├── app/
│   │   ├── components/
│   │   └── ...
│   ├── public/
│   ├── package.json
│   └── next.config.js
├── docs/             # Documentation
├── docker-compose.yml
└── README.md
```

### Database Schema Requirements
```prisma
model ExampleTable {
  id        String    @id @default(cuid()) // Use ULID in app layer
  // ... fields
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  deletedAt DateTime? // Soft delete
}

model AuditLog {
  id       String   @id @default(cuid())
  userId   String?
  tenantId String?
  action   String   // CREATE, UPDATE, DELETE
  entity   String   // Table name
  entityId String
  oldData  Json?
  newData  Json?
  diff     Json?
  metadata Json?
  ip       String?
  createdAt DateTime @default(now())
}
```

### NestJS Backend Structure (backend/)
```
src/
├── auth/             # Authentication module
├── users/            # Users module
├── common/           # Shared utilities (decorators, guards, pipes)
├── config/           # Configuration files
├── database/         # Database migrations and seeds
├── types/            # Backend types, DTOs, and interfaces
│   ├── auth.types.ts
│   ├── user.types.ts
│   ├── api.types.ts
│   └── index.ts
├── app.module.ts
└── main.ts
```

### Next.js Frontend Structure (frontend/)
```
src/
├── app/             # App Router (Next.js 13+)
│   ├── (auth)/     # Route groups
│   ├── api/        # API routes
│   ├── globals.css # Global styles
│   ├── layout.tsx  # Root layout
│   └── page.tsx    # Home page
├── components/      # Reusable components
│   ├── ui/         # Basic UI components
│   ├── layout/     # Layout components
│   └── features/   # Feature-specific components
├── hooks/          # Custom React hooks
├── lib/            # Utility functions and configurations
├── stores/         # State management (Zustand/Redux)
├── styles/         # Additional styles
└── types/          # Frontend-specific types
    ├── api.ts      # API response types
    ├── auth.ts     # Authentication types
    └── global.ts   # Global type definitions
```

## Design Guidelines

### UI/UX
- Use Preline UI for inspiration:
  - Admin: https://preline.co/templates/admin/index.html
  - AI Chat: https://preline.co/templates/ai-prompt/ai-chat.html
  - Components: https://preline.co/index.html
- Tailwind utility classes only (no custom CSS)
- Loading states with skeletons
- Optimistic updates for better UX
- Micro-interactions on all interactive elements

### Component Structure
```typescript
// Every component must have:
// 1. TypeScript interfaces
// 2. ARIA labels for accessibility
// 3. Loading and error states
// 4. Unit tests
// 5. Keyboard navigation support
```

## TypeScript Standards

### Strict Configuration
- Use strict TypeScript settings with `exactOptionalPropertyTypes`
- Enable `noUnusedLocals` and `noUnusedParameters`
- Use path mapping for clean imports: `@/*` for src

### Type Safety Rules
- Always define explicit interfaces for API requests/responses
- Define types locally in each project (`backend/src/types/` and `frontend/src/types/`)
- Prefer `interface` over `type` for object shapes
- Use enums for constants with multiple values
- Always type function parameters and return types for public APIs
- Duplicate common API types between frontend and backend for consistency
- Use API schema generation tools when possible to maintain type consistency

## NestJS Best Practices

### Module Organization
- Each feature should be a self-contained module
- Use feature modules: `auth/`, `users/`, etc.
- Export services that other modules need
- Use `@Global()` decorator sparingly, only for truly global services

### Controller Guidelines
- Keep controllers thin - only handle HTTP concerns
- Use proper HTTP status codes with `@HttpCode()` decorator
- Always validate requests with DTOs and `ValidationPipe`
- Use Swagger decorators for API documentation
- Example:
```typescript
@Controller('users')
@ApiTags('users')
export class UsersController {
  @Get()
  @ApiOperation({ summary: 'Get all users' })
  async findAll(@Query() query: FindUsersQueryDto): Promise<UserResponseDto[]> {
    return this.usersService.findAll(query);
  }
}
```

### Service Layer Patterns
- Services contain business logic and should be easily testable
- Use dependency injection with interfaces for better testability
- Handle errors appropriately with custom exceptions
- Use transactions for multi-step operations
- Example:
```typescript
@Injectable()
export class UsersService {
  constructor(
    private readonly userRepository: UserRepository,
    private readonly logger: Logger,
  ) {}

  async findOne(id: string): Promise<UserResponseDto> {
    const user = await this.userRepository.findById(id);
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    return this.mapToResponseDto(user);
  }
}
```

### Database Patterns
- Use Prisma with PostgreSQL
- ULID for all primary keys
- Soft deletes with `deleted_at` column on all tables
- Always include `created_at`, `updated_at`, `deleted_at` timestamps
- Audit logging for all create/update/delete operations
- Example schema:
```prisma
model User {
  id        String    @id @default(cuid()) // Use ULID in app layer
  email     String    @unique
  firstName String
  lastName  String
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  deletedAt DateTime? // Soft delete

  @@map("users")
}
```

### Authentication & Security
- Use Better Auth with `@thallesp/nestjs-better-auth` for authentication
- Configure Better Auth with database adapter (Prisma recommended)
- Use guards for route protection: `@UseGuards(AuthGuard)`
- Implement rate limiting with `@nestjs/throttler`
- Validate all inputs with class-validator
- Use helmet for security headers
- Example Better Auth setup:
```typescript
// auth.config.ts
export const auth = betterAuth({
  database: prismaAdapter(prisma),
  emailAndPassword: { enabled: true },
  socialProviders: {
    google: { clientId: process.env.GOOGLE_CLIENT_ID!, clientSecret: process.env.GOOGLE_CLIENT_SECRET! }
  }
});

// auth.module.ts
AuthModule.forRoot(auth, { disableExceptionFilter: true, disableTrustedOriginsCors: true })

// main.ts setup
const app = await NestFactory.create(AppModule, { bodyParser: false });
app.use((req, res, next) => {
  if (!req.path.startsWith('/api/auth/')) {
    express.json()(req, res, next);
  } else {
    next();
  }
});
```

### Error Handling
- Create domain-specific exceptions
- Use global exception filters
- Log errors appropriately
- Return consistent error responses
- Example:
```typescript
export class UserNotFoundException extends NotFoundException {
  constructor(id: string) {
    super(`User with ID ${id} not found`);
  }
}
```

## Next.js Frontend Best Practices

### Component Organization
- Use PascalCase for component files: `UserCard.tsx`
- Organize components by purpose: `ui/`, `layout/`, `features/`
- Keep components focused and reusable
- Use TypeScript interfaces for component props
- Example:
```typescript
interface UserCardProps {
  user: User;
  showActions?: boolean;
}

export function UserCard({ user, showActions = false }: UserCardProps) {
  return (
    <div className="user-card bg-white rounded-lg shadow-md p-4">
      <h3 className="text-lg font-semibold text-gray-900">
        {user.firstName} {user.lastName}
      </h3>
      <p className="text-sm text-gray-600">{user.email}</p>
      {showActions && (
        <button className="mt-2 px-3 py-1 bg-blue-500 text-white rounded">
          Edit
        </button>
      )}
    </div>
  );
}
```

### File Structure Guidelines
- Use PascalCase for component files: `UserProfile.tsx`
- Use camelCase for TypeScript files: `apiClient.ts`
- Group related functionality in feature folders
- Use App Router for routing: `app/` directory
- Store global styles in `app/globals.css`

### Styling Approach
- Use Tailwind CSS as primary styling solution
- Apply Tailwind classes directly in components
- Use CSS Modules for component-specific styles when needed
- Follow mobile-first responsive design
- Use CSS custom properties for theming
- Example:
```typescript
export default function Dashboard() {
  return (
    <div className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold text-gray-900 mb-6">Dashboard</h1>
    </div>
  );
}
```

### State Management
- Use Zustand for global client-side state
- Use TanStack Query for server state management
- Use Better Auth React client for authentication state
- Forms: react-hook-form + zod validation
- Keep state minimal and derived when possible
- Example:
```typescript
// lib/auth-client.ts
import { createAuthClient } from "better-auth/react";

export const authClient = createAuthClient({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
});

export const { useSession, signIn, signOut, signUp } = authClient;

// stores/app.ts
import { create } from 'zustand';

interface AppState {
  theme: 'light' | 'dark';
  sidebarOpen: boolean;
  toggleTheme: () => void;
  toggleSidebar: () => void;
}

export const useAppStore = create<AppState>((set) => ({
  theme: 'light',
  sidebarOpen: false,
  toggleTheme: () => set((state) => ({ theme: state.theme === 'light' ? 'dark' : 'light' })),
  toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
}));
```

### API Integration
- Create service layer for API calls
- Use consistent error handling across services
- Implement proper loading states
- Cache responses when appropriate
- Use Better Auth client for authentication
- Example:
```typescript
// lib/auth-client.ts
import { createAuthClient } from "better-auth/react";

export const authClient = createAuthClient({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
});

// components/LoginForm.tsx
import { authClient } from "@/lib/auth-client";

export function LoginForm() {
  const { signIn } = authClient;

  const handleSubmit = async (email: string, password: string) => {
    await signIn.email({ email, password });
  };
}

export class UsersService {
  async findAll(params: FindUsersParams): Promise<PaginatedResponse<User>> {
    const response = await apiClient.get<PaginatedResponse<User>>('/users', params);
    if (response.success) {
      return response.data;
    }
    throw new Error(response.message || 'Failed to fetch users');
  }
}
```

### Performance Guidelines
- Use static generation when possible
- Implement proper image optimization
- Minimize JavaScript bundle size
- Use lazy loading for non-critical components
- Optimize fonts and assets
- Implement proper caching strategies

## Type Management & Cross-cutting Concerns

### Type Organization
- Define backend types in `backend/src/types/` directory
- Define frontend types in `frontend/src/types/` directory
- Use consistent naming: `CreateUserDto`, `UserResponseDto`
- Export from centralized index files in each project
- Duplicate common API types between projects for consistency
- Example backend types:
```typescript
// backend/src/types/user.types.ts
export interface CreateUserDto {
  email: string;
  firstName: string;
  lastName: string;
  password: string;
}

export interface UserResponseDto {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  createdAt: string;
  updatedAt: string;
}
```

- Example frontend types:
```typescript
// frontend/src/types/api.ts
export interface CreateUserRequest {
  email: string;
  firstName: string;
  lastName: string;
  password: string;
}

export interface User {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  createdAt: string;
  updatedAt: string;
}
```

### Code Quality Standards
- Use ESLint with TypeScript rules
- Format code with Prettier
- Maintain 80%+ test coverage
- Use Husky for pre-commit hooks
- Follow conventional commit messages
- Example commit: `feat(auth): implement JWT refresh token mechanism`

### Git Workflow
- Use feature branches: `feature/user-authentication`
- Create PRs for all changes
- Require code review before merge
- Use semantic versioning for releases
- Keep commits atomic and descriptive

## Testing Requirements

### Mandatory Testing
- **Unit Tests**: Every component and service
- **Integration Tests**: All API endpoints
- **E2E Tests**: Critical user flows with Playwright
- **Coverage**: Minimum 80% code coverage

### Test File Naming
- Components: `component.test.tsx`
- Services: `service.spec.ts`
- E2E: `feature.e2e.spec.ts`

### Testing Strategy
- Unit tests for services and utilities
- Integration tests for API endpoints
- Component tests for React components with React Testing Library
- E2E tests for critical user flows with Playwright
- Use Jest for backend and frontend
- Mock external dependencies appropriately

## API Documentation
- Maintain OpenAPI/Swagger spec
- Use NestJS decorators for auto-generation
- Sync Postman collection
- Document all endpoints with examples

## Performance Targets
- Frontend: < 3s initial load, < 100ms interactions
- Backend: < 200ms p95 response time
- Bundle size: < 500KB gzipped

## Security Requirements
- Zero-trust architecture
- Rate limiting on all endpoints
- Input validation everywhere
- Audit trail for compliance
- No sensitive data in client storage
- CSP headers with nonce-based scripts
- XSS prevention through proper escaping

## Deployment
- Frontend: Vercel (preferred) or Netlify
- Backend: Railway, Render, or Fly.io
- Database: Supabase, Neon, or Railway PostgreSQL
- Redis: Upstash or Railway

### Environment Management
- Use `.env` files for configuration
- Never commit secrets to version control
- Use different configs for dev/staging/prod
- Validate environment variables on startup
- Use TypeScript for config validation

### Documentation Standards
- Maintain comprehensive README files
- Document API endpoints with Swagger
- Use JSDoc for complex functions
- Keep architecture decision records (ADRs)
- Update docs with code changes

## Development Workflow
- Feature branches with PR reviews
- Biome for code formatting
- Conventional commits
- Update README for significant changes
- Document architectural decisions

### Package Management
- Always use package managers (npm/yarn/pnpm) for dependencies
- Never manually edit package.json for dependencies
- Manage dependencies separately for backend and frontend directories
- Keep dependencies up to date with security patches

### Code Review Guidelines
- Review PRs within 24 hours
- Focus on logic, security, and maintainability
- Provide constructive feedback with examples
- Ensure tests pass and coverage is maintained
- Check for proper error handling and validation

## Memory Management

### When to Add Memories
- Starting new features
- Discovering patterns
- Solving complex problems
- Integration specifics

### Guidelines Organization
```
project/
├── .augment-guidelines          # This file
├── app/
│   └── .augment-guidelines      # Frontend-specific
├── src/
│   └── .augment-guidelines      # Backend-specific
└── e2e/
    └── .augment-guidelines      # Testing patterns
```

## Quick Commands

### Frontend
```bash
npx create-next-app@latest --typescript --tailwind --app
npm install @tanstack/react-query zustand react-hook-form zod
npm install -D @biomejs/biome @testing-library/react playwright
```

### Backend
```bash
nest new api
npm install @nestjs/swagger @nestjs/throttler nestjs-pino
npm install prisma @prisma/client bull bullmq
npm install -D @biomejs/biome jest supertest
```

### Initialize
```bash
# Biome setup
npx @biomejs/biome init
npx @biomejs/biome check --write .

# Prisma setup
npx prisma init
npx prisma generate
```