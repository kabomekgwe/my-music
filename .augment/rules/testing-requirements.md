---
type: "agent_requested"
description: "Example description"
---
# Testing Requirements

**Type**: Always  
**Description**: Comprehensive testing standards and requirements for quality assurance

## Mandatory Testing Coverage

- **Unit Tests**: Every component and service MUST have tests
- **Integration Tests**: All API endpoints MUST be tested
- **E2E Tests**: Critical user flows with Playwright
- **Coverage**: Minimum 80% code coverage required
- **Test naming**: `component.test.tsx`, `service.spec.ts`, `feature.e2e.spec.ts`

## Testing Strategy

- **Unit tests** for services and utilities
- **Integration tests** for API endpoints
- **Component tests** for React components with React Testing Library
- **E2E tests** for critical user flows with Playwright
- **Use Jest** for backend and frontend
- **Mock external dependencies** appropriately

## Frontend Testing (React Testing Library)

```typescript
// UserCard.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { UserCard } from './UserCard';
import { mockUser } from '@/test/mocks';

describe('UserCard', () => {
  it('should render user information correctly', () => {
    render(<UserCard user={mockUser} />);
    
    expect(screen.getByText(mockUser.firstName)).toBeInTheDocument();
    expect(screen.getByText(mockUser.email)).toBeInTheDocument();
  });

  it('should call onEdit when edit button is clicked', () => {
    const mockOnEdit = jest.fn();
    render(<UserCard user={mockUser} onEdit={mockOnEdit} isEditable />);
    
    fireEvent.click(screen.getByRole('button', { name: /edit/i }));
    expect(mockOnEdit).toHaveBeenCalledWith(mockUser);
  });

  it('should not show edit button when not editable', () => {
    render(<UserCard user={mockUser} isEditable={false} />);
    
    expect(screen.queryByRole('button', { name: /edit/i })).not.toBeInTheDocument();
  });
});
```

## Backend Testing (Jest + Supertest)

```typescript
// users.controller.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { mockUser, mockCreateUserDto } from '../test/mocks';

describe('UsersController', () => {
  let controller: UsersController;
  let service: UsersService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [
        {
          provide: UsersService,
          useValue: {
            findAll: jest.fn(),
            create: jest.fn(),
            findById: jest.fn(),
          },
        },
      ],
    }).compile();

    controller = module.get<UsersController>(UsersController);
    service = module.get<UsersService>(UsersService);
  });

  describe('create', () => {
    it('should create a user successfully', async () => {
      jest.spyOn(service, 'create').mockResolvedValue(mockUser);

      const result = await controller.create(mockCreateUserDto, mockUser);

      expect(service.create).toHaveBeenCalledWith(mockCreateUserDto, mockUser);
      expect(result).toEqual(mockUser);
    });
  });
});
```

## Integration Testing

```typescript
// users.e2e-spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from '../src/app.module';

describe('UsersController (e2e)', () => {
  let app: INestApplication;

  beforeEach(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  it('/users (POST)', () => {
    return request(app.getHttpServer())
      .post('/users')
      .send({
        email: 'test@example.com',
        firstName: 'John',
        lastName: 'Doe',
        password: 'password123',
      })
      .expect(201)
      .expect((res) => {
        expect(res.body.email).toBe('test@example.com');
        expect(res.body.firstName).toBe('John');
        expect(res.body).not.toHaveProperty('password');
      });
  });

  it('/users (GET)', () => {
    return request(app.getHttpServer())
      .get('/users')
      .expect(200)
      .expect((res) => {
        expect(res.body.data).toBeInstanceOf(Array);
        expect(res.body.pagination).toBeDefined();
      });
  });
});
```

## E2E Testing (Playwright)

```typescript
// user-management.e2e.spec.ts
import { test, expect } from '@playwright/test';

test.describe('User Management', () => {
  test('should create a new user', async ({ page }) => {
    await page.goto('/users');
    
    // Click create user button
    await page.click('[data-testid="create-user-btn"]');
    
    // Fill form
    await page.fill('[data-testid="email-input"]', 'test@example.com');
    await page.fill('[data-testid="firstName-input"]', 'John');
    await page.fill('[data-testid="lastName-input"]', 'Doe');
    
    // Submit form
    await page.click('[data-testid="submit-btn"]');
    
    // Verify success
    await expect(page.locator('[data-testid="success-message"]')).toBeVisible();
    await expect(page.locator('text=test@example.com')).toBeVisible();
  });

  test('should edit existing user', async ({ page }) => {
    await page.goto('/users');
    
    // Click edit button for first user
    await page.click('[data-testid="edit-user-btn"]:first-child');
    
    // Update name
    await page.fill('[data-testid="firstName-input"]', 'Jane');
    await page.click('[data-testid="submit-btn"]');
    
    // Verify update
    await expect(page.locator('text=Jane')).toBeVisible();
  });
});
```

## Test Utilities and Mocks

```typescript
// test/mocks/user.mock.ts
export const mockUser = {
  id: '1',
  email: 'test@example.com',
  firstName: 'John',
  lastName: 'Doe',
  isActive: true,
  createdAt: '2023-01-01T00:00:00Z',
  updatedAt: '2023-01-01T00:00:00Z',
};

export const mockCreateUserDto = {
  email: 'test@example.com',
  firstName: 'John',
  lastName: 'Doe',
  password: 'password123',
};

// test/setup.ts
import '@testing-library/jest-dom';

// Mock environment variables
process.env.NODE_ENV = 'test';
process.env.DATABASE_URL = 'postgresql://test:test@localhost:5432/test';
```

## Performance Testing

```typescript
// performance.spec.ts
import { performance } from 'perf_hooks';

describe('Performance Tests', () => {
  it('should load users list within 200ms', async () => {
    const start = performance.now();
    
    const response = await request(app.getHttpServer())
      .get('/users')
      .expect(200);
    
    const end = performance.now();
    const duration = end - start;
    
    expect(duration).toBeLessThan(200);
    expect(response.body.data.length).toBeGreaterThan(0);
  });
});
```

## Test Configuration

```json
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/test/**',
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  setupFilesAfterEnv: ['<rootDir>/test/setup.ts'],
};
```
