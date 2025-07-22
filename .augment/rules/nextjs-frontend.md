---
type: "agent_requested"
description: "Example description"
---
# Next.js Frontend Rules

**Type**: Auto  
**Description**: Next.js specific patterns and requirements for client-side development

## Core Requirements

- **Always use `'use client'` directive** - NO server components or SSR
- **Use route groups**: `(auth)` for public pages, `(dashboard)` for authenticated pages
- **Define explicit TypeScript interfaces** for all component props
- **Include ARIA labels** for accessibility (WCAG 2.1 AA compliance)
- **Implement loading and error states** for all components
- **Include skeleton components** for loading states
- **Support keyboard navigation** on all interactive elements

## State Management Decisions

- **URL-shareable state** → Use URL search params with nuqs
- **Server data** → Use TanStack Query with proper cache keys
- **Global UI state** → Use Zustand store
- **Form data** → Use react-hook-form + Zod validation
- **Local component state** → Use useState/useReducer

## Component Template

```typescript
'use client';

interface ComponentNameProps {
  // Always define explicit props interface
  data: DataType;
  onAction?: (item: DataType) => void;
  loading?: boolean;
  className?: string;
}

export function ComponentName({
  data,
  onAction,
  loading = false,
  className
}: ComponentNameProps) {
  // Always include loading and error states
  if (loading) {
    return <ComponentNameSkeleton />;
  }

  return (
    <div
      className={cn("base-styles", className)}
      role="region" // Always include ARIA labels
      aria-label="Component description"
    >
      {/* Component content */}
    </div>
  );
}

// Always include skeleton for loading states
function ComponentNameSkeleton() {
  return (
    <div className="animate-pulse">
      <div className="h-4 bg-gray-200 rounded w-3/4 mb-2"></div>
      <div className="h-4 bg-gray-200 rounded w-1/2"></div>
    </div>
  );
}
```

## State Management Examples

### Zustand Store
```typescript
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

### Better Auth Client
```typescript
// lib/auth-client.ts
import { createAuthClient } from "better-auth/react";

export const authClient = createAuthClient({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
});

export const { useSession, signIn, signOut, signUp } = authClient;

// components/LoginForm.tsx
import { authClient } from "@/lib/auth-client";

export function LoginForm() {
  const { signIn } = authClient;

  const handleSubmit = async (email: string, password: string) => {
    await signIn.email({ email, password });
  };
}
```

## UI/UX Standards

- **Use Preline UI** for design inspiration
- **Tailwind utility classes only** (no custom CSS)
- **Include loading states** with skeletons for all components
- **Implement optimistic updates** for better UX
- **WCAG 2.1 AA compliance** required
- **Support keyboard navigation** on all interactive elements

## Performance Guidelines

- **Use lazy loading** for non-critical components
- **Implement proper caching strategies**
- **Minimize JavaScript bundle size**
- **Optimize fonts and assets**
- **Use static generation when possible**
- **Implement proper image optimization**

## Route Structure

```
src/app/
├── (auth)/           # Public pages
│   ├── login/
│   └── register/
├── (dashboard)/      # Authenticated pages
│   ├── profile/
│   └── settings/
├── api/             # API routes
├── globals.css      # Global styles
├── layout.tsx       # Root layout
└── page.tsx         # Home page
```
