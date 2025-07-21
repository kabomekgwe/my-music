# Security & Authentication Rules

**Type**: Always  
**Description**: Critical security requirements that must be followed in all code

## Authentication & Authorization

- **NEVER store sensitive data in client storage** - use sessionStorage only for non-sensitive data
- **Validate ALL user inputs** with Zod schemas on client AND server
- **Use Better Auth** with these required plugins: magic-link, email-otp, twoFactor, admin, organization, passkey, haveIBeenPwned, oneTimeToken, openAPI
- **Implement rate limiting** on all API endpoints using `@nestjs/throttler`
- **Use guards for route protection**: `@UseGuards(AuthGuard)` on all protected controllers
- **Validate user permissions** before any data mutation: `if (!this.canUserPerformAction(user, 'CREATE', dto)) throw new ForbiddenException()`
- **Enable CSP headers** and XSS prevention through proper escaping
- **Use helmet** for security headers in NestJS

## Input Validation Layers (MANDATORY)

1. Client-side: Zod schema validation
2. API Gateway: Rate limiting + basic validation
3. Controller: DTO validation with class-validator
4. Service: Business rule validation
5. Database: Prisma schema constraints

## Security Checklist (VERIFY BEFORE COMMIT)

- [ ] All user inputs validated with Zod schemas
- [ ] SQL injection prevention (Prisma parameterized queries)
- [ ] XSS prevention (proper escaping/sanitization)
- [ ] CSRF protection (Better Auth handles this)
- [ ] Rate limiting implemented on endpoints
- [ ] Sensitive data not logged or stored in client
- [ ] Audit logging for all data mutations
- [ ] Authentication required for protected routes
- [ ] Authorization checks for resource access

## Authentication Patterns (ALWAYS IMPLEMENT)

### Route-level protection
```typescript
@UseGuards(AuthGuard)
@Controller('protected-resource')
```

### Component-level protection
```typescript
'use client';
import { useSession } from '@/lib/auth-client';

export function ProtectedComponent() {
  const { data: session, isLoading } = useSession();

  if (isLoading) return <LoadingSkeleton />;
  if (!session) return <LoginPrompt />;

  return <ActualComponent />;
}
```

### API-level validation
```typescript
async create(dto: CreateDto, @CurrentUser() user: User) {
  // Always validate user has permission for this action
  if (!this.canUserPerformAction(user, 'CREATE', dto)) {
    throw new ForbiddenException('Insufficient permissions');
  }
}
```
