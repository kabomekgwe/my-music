---
type: "agent_requested"
description: "Example description"
---
# Performance Standards

**Type**: Manual  
**Description**: Performance targets and optimization guidelines for frontend and backend

## Performance Targets

- **Frontend**: < 3s initial load, < 100ms interactions
- **Backend**: < 200ms p95 response time
- **Bundle size**: < 500KB gzipped
- **Database queries**: < 50ms average response time
- **Memory usage**: < 512MB per service instance

## Frontend Performance

### Bundle Optimization
- **Use lazy loading** for non-critical components
- **Implement code splitting** at route level
- **Minimize JavaScript bundle size**
- **Optimize fonts and assets**
- **Use static generation when possible**
- **Implement proper image optimization**

### React Performance Patterns

```typescript
// Lazy loading components
import { lazy, Suspense } from 'react';

const UserDashboard = lazy(() => import('./UserDashboard'));

function App() {
  return (
    <Suspense fallback={<DashboardSkeleton />}>
      <UserDashboard />
    </Suspense>
  );
}

// Memoization for expensive calculations
import { useMemo } from 'react';

function UserList({ users, filters }) {
  const filteredUsers = useMemo(() => {
    return users.filter(user => 
      user.name.toLowerCase().includes(filters.search.toLowerCase())
    );
  }, [users, filters.search]);

  return (
    <div>
      {filteredUsers.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
}

// React.memo for component optimization
import { memo } from 'react';

export const UserCard = memo(({ user, onEdit }) => {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <button onClick={() => onEdit(user)}>Edit</button>
    </div>
  );
});
```

### Caching Strategies

```typescript
// TanStack Query with proper cache keys
import { useQuery } from '@tanstack/react-query';

function useUsers(filters: UserFilters) {
  return useQuery({
    queryKey: ['users', filters],
    queryFn: () => userService.getUsers(filters),
    staleTime: 5 * 60 * 1000, // 5 minutes
    cacheTime: 10 * 60 * 1000, // 10 minutes
  });
}

// Optimistic updates
import { useMutation, useQueryClient } from '@tanstack/react-query';

function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: userService.updateUser,
    onMutate: async (updatedUser) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries(['users']);

      // Snapshot previous value
      const previousUsers = queryClient.getQueryData(['users']);

      // Optimistically update
      queryClient.setQueryData(['users'], (old) => 
        old.map(user => user.id === updatedUser.id ? updatedUser : user)
      );

      return { previousUsers };
    },
    onError: (err, updatedUser, context) => {
      // Rollback on error
      queryClient.setQueryData(['users'], context.previousUsers);
    },
    onSettled: () => {
      // Refetch after mutation
      queryClient.invalidateQueries(['users']);
    },
  });
}
```

## Backend Performance

### Database Optimization

```typescript
// Efficient pagination with cursor-based approach
async function getUsersPaginated(cursor?: string, limit = 20) {
  const users = await prisma.user.findMany({
    take: limit + 1, // Take one extra to check if there's a next page
    cursor: cursor ? { id: cursor } : undefined,
    orderBy: { createdAt: 'desc' },
    where: { deletedAt: null },
  });

  const hasNextPage = users.length > limit;
  const items = hasNextPage ? users.slice(0, -1) : users;

  return {
    data: items,
    hasNextPage,
    nextCursor: hasNextPage ? items[items.length - 1].id : null,
  };
}

// Efficient queries with proper indexing
async function searchUsers(searchTerm: string) {
  return prisma.user.findMany({
    where: {
      AND: [
        { deletedAt: null },
        {
          OR: [
            { firstName: { contains: searchTerm, mode: 'insensitive' } },
            { lastName: { contains: searchTerm, mode: 'insensitive' } },
            { email: { contains: searchTerm, mode: 'insensitive' } },
          ],
        },
      ],
    },
    // Add proper indexes in schema:
    // @@index([firstName, lastName, email])
  });
}

// Batch operations for efficiency
async function updateMultipleUsers(userUpdates: Array<{ id: string; data: any }>) {
  const updatePromises = userUpdates.map(({ id, data }) =>
    prisma.user.update({
      where: { id },
      data,
    })
  );

  return Promise.all(updatePromises);
}
```

### Caching Implementation

```typescript
// Redis caching for expensive operations
import { Injectable } from '@nestjs/common';
import { Redis } from 'ioredis';

@Injectable()
export class CacheService {
  constructor(private readonly redis: Redis) {}

  async get<T>(key: string): Promise<T | null> {
    const cached = await this.redis.get(key);
    return cached ? JSON.parse(cached) : null;
  }

  async set(key: string, value: any, ttl = 3600): Promise<void> {
    await this.redis.setex(key, ttl, JSON.stringify(value));
  }

  async invalidate(pattern: string): Promise<void> {
    const keys = await this.redis.keys(pattern);
    if (keys.length > 0) {
      await this.redis.del(...keys);
    }
  }
}

// Service with caching
@Injectable()
export class UsersService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly cache: CacheService,
  ) {}

  async findById(id: string): Promise<User | null> {
    const cacheKey = `user:${id}`;
    
    // Try cache first
    const cached = await this.cache.get<User>(cacheKey);
    if (cached) return cached;

    // Fetch from database
    const user = await this.prisma.user.findUnique({
      where: { id, deletedAt: null },
    });

    // Cache the result
    if (user) {
      await this.cache.set(cacheKey, user, 300); // 5 minutes
    }

    return user;
  }
}
```

### Rate Limiting

```typescript
// Rate limiting configuration
import { ThrottlerModule } from '@nestjs/throttler';

@Module({
  imports: [
    ThrottlerModule.forRoot({
      ttl: 60, // 1 minute
      limit: 100, // 100 requests per minute
    }),
  ],
})
export class AppModule {}

// Controller with rate limiting
@Controller('users')
@UseGuards(ThrottlerGuard)
export class UsersController {
  @Post()
  @Throttle(5, 60) // 5 requests per minute for user creation
  async create(@Body() createDto: CreateUserDto) {
    return this.usersService.create(createDto);
  }
}
```

## Monitoring and Metrics

```typescript
// Performance monitoring
import { Injectable, Logger } from '@nestjs/common';
import { performance } from 'perf_hooks';

@Injectable()
export class PerformanceService {
  private readonly logger = new Logger(PerformanceService.name);

  async measureAsync<T>(
    operation: () => Promise<T>,
    operationName: string,
  ): Promise<T> {
    const start = performance.now();
    
    try {
      const result = await operation();
      const duration = performance.now() - start;
      
      this.logger.log(`${operationName} completed in ${duration.toFixed(2)}ms`);
      
      // Alert if operation is slow
      if (duration > 1000) {
        this.logger.warn(`Slow operation detected: ${operationName} took ${duration.toFixed(2)}ms`);
      }
      
      return result;
    } catch (error) {
      const duration = performance.now() - start;
      this.logger.error(`${operationName} failed after ${duration.toFixed(2)}ms`, error);
      throw error;
    }
  }
}
```

## Performance Testing

```typescript
// Load testing with Artillery or similar
// artillery.yml
config:
  target: 'http://localhost:3000'
  phases:
    - duration: 60
      arrivalRate: 10
scenarios:
  - name: 'Get users'
    requests:
      - get:
          url: '/api/users'
          headers:
            Authorization: 'Bearer {{ token }}'

// Performance assertions in tests
describe('Performance Tests', () => {
  it('should handle 100 concurrent requests', async () => {
    const requests = Array(100).fill(null).map(() =>
      request(app.getHttpServer()).get('/users')
    );

    const start = performance.now();
    const responses = await Promise.all(requests);
    const duration = performance.now() - start;

    expect(duration).toBeLessThan(5000); // 5 seconds max
    responses.forEach(response => {
      expect(response.status).toBe(200);
    });
  });
});
```
