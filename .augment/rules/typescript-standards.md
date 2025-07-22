---
type: "agent_requested"
description: "Example description"
---
# TypeScript Standards

**Type**: Always  
**Description**: Strict TypeScript configuration and type safety requirements

## Strict Configuration (REQUIRED)

- **Enable strict mode** with `exactOptionalPropertyTypes`
- **Enable `noUnusedLocals` and `noUnusedParameters`**
- **Use path mapping**: `@/*` for src imports
- **Prefer `interface` over `type`** for object shapes
- **Always type function parameters and return types** for public APIs
- **Define types locally** in each project (`backend/src/types/` and `frontend/src/types/`)
- **Duplicate common API types** between frontend and backend for consistency

## TypeScript Configuration

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "exactOptionalPropertyTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

## Type Organization

### Backend Types (`backend/src/types/`)

```typescript
// backend/src/types/user.types.ts
export interface CreateUserDto {
  email: string;
  firstName: string;
  lastName: string;
  password: string;
}

export interface UpdateUserDto {
  firstName?: string;
  lastName?: string;
  email?: string;
}

export interface UserResponseDto {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  isActive: boolean;
  createdAt: string;
  updatedAt: string;
}

export interface FindUsersQueryDto {
  page?: number;
  limit?: number;
  search?: string;
  isActive?: boolean;
  sortBy?: 'createdAt' | 'email' | 'firstName';
  sortOrder?: 'asc' | 'desc';
}
```

### Frontend Types (`frontend/src/types/`)

```typescript
// frontend/src/types/api.ts
export interface CreateUserRequest {
  email: string;
  firstName: string;
  lastName: string;
  password: string;
}

export interface UpdateUserRequest {
  firstName?: string;
  lastName?: string;
  email?: string;
}

export interface User {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  isActive: boolean;
  createdAt: string;
  updatedAt: string;
}

export interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}
```

## Type Safety Rules

### ✅ Correct Patterns

```typescript
// Always define explicit interfaces
interface UserCardProps {
  user: User;
  onEdit?: (user: User) => void;
  isEditable?: boolean;
}

// Always type function parameters and return types
async function getUserById(id: string): Promise<User | null> {
  const user = await userService.findById(id);
  return user;
}

// Use enums for constants with multiple values
enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
  MODERATOR = 'moderator',
}

// Prefer interface over type for object shapes
interface ApiResponse<T> {
  data: T;
  success: boolean;
  message?: string;
}
```

### ❌ Incorrect Patterns

```typescript
// Don't use 'any'
function processData(data: any): any { // ❌
  return data;
}

// Don't leave functions untyped
function processUser(user) { // ❌
  return user.name;
}

// Don't use 'type' for simple object shapes
type UserProps = { // ❌ - use interface instead
  name: string;
  email: string;
};

// Don't use loose typing
interface LooseProps {
  data: object; // ❌ - be specific
  callback: Function; // ❌ - define the function signature
}
```

## Generic Patterns

```typescript
// Generic API response wrapper
interface ApiResponse<T> {
  data: T;
  success: boolean;
  message?: string;
  errors?: string[];
}

// Generic paginated response
interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}

// Generic service interface
interface BaseService<T, CreateDto, UpdateDto> {
  findAll(query?: any): Promise<PaginatedResponse<T>>;
  findById(id: string): Promise<T | null>;
  create(dto: CreateDto): Promise<T>;
  update(id: string, dto: UpdateDto): Promise<T>;
  delete(id: string): Promise<void>;
}
```

## Utility Types

```typescript
// Common utility types
export type Optional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;
export type RequiredFields<T, K extends keyof T> = T & Required<Pick<T, K>>;

// API-specific types
export type CreateRequest<T> = Omit<T, 'id' | 'createdAt' | 'updatedAt'>;
export type UpdateRequest<T> = Partial<Omit<T, 'id' | 'createdAt' | 'updatedAt'>>;

// Example usage
type CreateUserRequest = CreateRequest<User>;
type UpdateUserRequest = UpdateRequest<User>;
```

## Import/Export Patterns

```typescript
// Centralized exports
// backend/src/types/index.ts
export * from './user.types';
export * from './auth.types';
export * from './api.types';

// frontend/src/types/index.ts
export * from './api';
export * from './auth';
export * from './global';

// Usage
import { User, CreateUserRequest, ApiResponse } from '@/types';
```
