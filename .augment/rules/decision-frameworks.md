---
type: "agent_requested"
description: "Example description"
---
# Decision Frameworks

**Type**: Manual  
**Description**: Decision trees and frameworks for making consistent architectural and implementation choices

## Component Creation Framework

```
IF component has > 3 unique props AND different behavior
  → Create new component
ELSE IF styling differences only
  → Use variant props with class-variance-authority (CVA)
ELSE
  → Extend existing component with additional props
```

### Example Implementation

```typescript
// ✅ Create new component (different behavior)
interface UserCardProps {
  user: User;
  onEdit: (user: User) => void;
  showActions: boolean;
}

interface AdminUserCardProps {
  user: User;
  onEdit: (user: User) => void;
  onDelete: (user: User) => void;
  onSuspend: (user: User) => void;
  showAuditLog: boolean;
  permissions: Permission[];
}

// ✅ Use variants (styling differences only)
import { cva } from 'class-variance-authority';

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md text-sm font-medium',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        outline: 'border border-input hover:bg-accent hover:text-accent-foreground',
      },
      size: {
        default: 'h-10 px-4 py-2',
        sm: 'h-9 rounded-md px-3',
        lg: 'h-11 rounded-md px-8',
      },
    },
  }
);
```

## API Endpoint Design Framework

```
IF data mutation (create/update/delete)
  → POST/PUT/PATCH with Zod validation + audit logging
IF data retrieval with complex filtering
  → GET with query params + pagination + caching
IF real-time updates needed
  → Consider WebSocket or Server-Sent Events
IF file upload required
  → Use multipart/form-data with size/type validation
```

### Implementation Examples

```typescript
// Data mutation endpoint
@Post('users')
@ApiOperation({ summary: 'Create new user' })
@UseGuards(AuthGuard)
async createUser(
  @Body() createDto: CreateUserDto,
  @CurrentUser() user: User,
): Promise<UserResponseDto> {
  // Validate permissions
  if (!this.canUserPerformAction(user, 'CREATE_USER', createDto)) {
    throw new ForbiddenException('Insufficient permissions');
  }

  const newUser = await this.usersService.create(createDto, user);
  
  // Audit logging
  await this.auditService.log({
    userId: user.id,
    action: 'CREATE',
    entity: 'User',
    entityId: newUser.id,
    newData: newUser,
  });

  return newUser;
}

// Complex filtering endpoint
@Get('users')
@ApiOperation({ summary: 'List users with filtering' })
@ApiQuery({ name: 'search', required: false })
@ApiQuery({ name: 'role', required: false })
@ApiQuery({ name: 'isActive', required: false })
@ApiQuery({ name: 'page', required: false })
@ApiQuery({ name: 'limit', required: false })
async getUsers(
  @Query() query: FindUsersQueryDto,
): Promise<PaginatedResponse<UserResponseDto>> {
  return this.usersService.findAll(query);
}

// File upload endpoint
@Post('users/:id/avatar')
@UseInterceptors(FileInterceptor('file'))
@ApiConsumes('multipart/form-data')
async uploadAvatar(
  @Param('id') id: string,
  @UploadedFile() file: Express.Multer.File,
  @CurrentUser() user: User,
) {
  // Validate file type and size
  if (!file.mimetype.startsWith('image/')) {
    throw new BadRequestException('Only image files are allowed');
  }
  
  if (file.size > 5 * 1024 * 1024) { // 5MB
    throw new BadRequestException('File size must be less than 5MB');
  }

  return this.usersService.updateAvatar(id, file, user);
}
```

## State Management Decision Tree

```
IF URL-shareable state (filters, pagination)
  → Use URL search params with nuqs
IF server data (API responses)
  → Use TanStack Query with proper cache keys
IF global UI state (theme, sidebar)
  → Use Zustand store
IF form data
  → Use react-hook-form with Zod validation
IF local component state
  → Use useState/useReducer
```

### Implementation Examples

```typescript
// URL-shareable state with nuqs
import { useQueryState } from 'nuqs';

function UsersList() {
  const [search, setSearch] = useQueryState('search', { defaultValue: '' });
  const [page, setPage] = useQueryState('page', { defaultValue: 1, parse: parseInt });
  const [role, setRole] = useQueryState('role');

  // URL automatically updates: /users?search=john&page=2&role=admin
  return (
    <div>
      <input 
        value={search} 
        onChange={(e) => setSearch(e.target.value)} 
        placeholder="Search users..."
      />
      {/* Component content */}
    </div>
  );
}

// Server data with TanStack Query
function useUsers(filters: UserFilters) {
  return useQuery({
    queryKey: ['users', filters],
    queryFn: () => userService.getUsers(filters),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
}

// Global UI state with Zustand
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
  toggleTheme: () => set((state) => ({ 
    theme: state.theme === 'light' ? 'dark' : 'light' 
  })),
  toggleSidebar: () => set((state) => ({ 
    sidebarOpen: !state.sidebarOpen 
  })),
}));

// Form data with react-hook-form + Zod
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const userSchema = z.object({
  email: z.string().email(),
  firstName: z.string().min(1),
  lastName: z.string().min(1),
});

function UserForm() {
  const form = useForm({
    resolver: zodResolver(userSchema),
    defaultValues: {
      email: '',
      firstName: '',
      lastName: '',
    },
  });

  const onSubmit = (data: z.infer<typeof userSchema>) => {
    // Handle form submission
  };

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      {/* Form fields */}
    </form>
  );
}
```

## Error Handling Decision Framework

```
IF validation error (user input)
  → Return 400 Bad Request with detailed field errors
IF authentication error
  → Return 401 Unauthorized
IF authorization error (permissions)
  → Return 403 Forbidden
IF resource not found
  → Return 404 Not Found
IF server error (unexpected)
  → Return 500 Internal Server Error + log details
IF rate limit exceeded
  → Return 429 Too Many Requests
```

### Implementation

```typescript
// Custom exception filters
@Catch(ZodError)
export class ZodExceptionFilter implements ExceptionFilter {
  catch(exception: ZodError, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();

    response.status(400).json({
      statusCode: 400,
      message: 'Validation failed',
      errors: exception.errors.map(error => ({
        field: error.path.join('.'),
        message: error.message,
      })),
    });
  }
}

// Service-level error handling
async findUserById(id: string, currentUser: User): Promise<UserResponseDto> {
  // Authorization check
  if (!this.canUserAccessResource(currentUser, 'READ', id)) {
    throw new ForbiddenException('Insufficient permissions to access this user');
  }

  const user = await this.prisma.user.findFirst({
    where: { id, deletedAt: null },
  });

  // Resource not found
  if (!user) {
    throw new NotFoundException(`User with ID ${id} not found`);
  }

  return this.mapToResponseDto(user);
}
```

## Testing Strategy Decision Framework

```
IF pure function or utility
  → Unit test with Jest
IF React component
  → Component test with React Testing Library
IF API endpoint
  → Integration test with Supertest
IF user workflow
  → E2E test with Playwright
IF performance critical
  → Add performance tests
```

### Implementation Guide

```typescript
// Pure function → Unit test
// utils/formatters.test.ts
describe('formatCurrency', () => {
  it('should format USD currency correctly', () => {
    expect(formatCurrency(1234.56, 'USD')).toBe('$1,234.56');
  });
});

// React component → Component test
// UserCard.test.tsx
describe('UserCard', () => {
  it('should render user information', () => {
    render(<UserCard user={mockUser} />);
    expect(screen.getByText(mockUser.name)).toBeInTheDocument();
  });
});

// API endpoint → Integration test
// users.e2e-spec.ts
describe('/users (e2e)', () => {
  it('should create user', () => {
    return request(app.getHttpServer())
      .post('/users')
      .send(createUserDto)
      .expect(201);
  });
});

// User workflow → E2E test
// user-registration.e2e.ts
test('user can register and login', async ({ page }) => {
  await page.goto('/register');
  await page.fill('[data-testid="email"]', 'test@example.com');
  await page.click('[data-testid="submit"]');
  await expect(page).toHaveURL('/dashboard');
});
```
