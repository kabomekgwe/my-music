# NestJS Backend Rules

**Type**: Auto  
**Description**: NestJS specific patterns and enterprise-grade backend development standards

## Core Principles

- **Keep controllers thin** - only handle HTTP concerns, delegate to services
- **Use proper HTTP status codes** with `@HttpCode()` decorator
- **Validate all requests** with DTOs and `ValidationPipe`
- **Use Swagger decorators** for API documentation: `@ApiOperation`, `@ApiTags`, `@ApiBody`
- **Implement dependency injection** with interfaces for testability
- **Use transactions** for multi-step database operations
- **Create domain-specific exceptions** extending NestJS base exceptions
- **Include structured logging** with Pino (include request_id, user_id, tenant_id)

## Module Organization

- **Each feature should be a self-contained module**
- **Use feature modules**: `auth/`, `users/`, etc.
- **Export services** that other modules need
- **Use `@Global()` decorator sparingly**, only for truly global services

## Controller Pattern

```typescript
@Controller('users')
@ApiTags('users')
@UseGuards(AuthGuard)
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  @ApiOperation({ summary: 'Get all users' })
  @ApiQuery({ name: 'page', required: false, type: Number })
  @ApiQuery({ name: 'limit', required: false, type: Number })
  async findAll(
    @Query() query: FindUsersQueryDto,
    @CurrentUser() user: User,
  ): Promise<PaginatedResponse<UserResponseDto>> {
    return this.usersService.findAll(query, user);
  }

  @Post()
  @ApiOperation({ summary: 'Create new user' })
  @ApiBody({ type: CreateUserDto })
  @HttpCode(201)
  async create(
    @Body() createDto: CreateUserDto,
    @CurrentUser() user: User,
  ): Promise<UserResponseDto> {
    return this.usersService.create(createDto, user);
  }
}
```

## Service Layer Pattern

```typescript
@Injectable()
export class UsersService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly logger: Logger,
  ) {}

  async findOne(id: string, user: User): Promise<UserResponseDto> {
    // Always validate permissions first
    if (!this.canUserAccessResource(user, 'READ', id)) {
      throw new ForbiddenException('Insufficient permissions');
    }

    const foundUser = await this.prisma.user.findFirst({
      where: { id, deletedAt: null },
    });

    if (!foundUser) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }

    return this.mapToResponseDto(foundUser);
  }

  async create(dto: CreateUserDto, user: User): Promise<UserResponseDto> {
    // Validate permissions
    if (!this.canUserPerformAction(user, 'CREATE', dto)) {
      throw new ForbiddenException('Insufficient permissions');
    }

    try {
      const newUser = await this.prisma.user.create({
        data: {
          ...dto,
          createdBy: user.id,
        },
      });

      // Log audit trail
      await this.auditService.log({
        userId: user.id,
        action: 'CREATE',
        entity: 'User',
        entityId: newUser.id,
        newData: newUser,
      });

      return this.mapToResponseDto(newUser);
    } catch (error) {
      this.logger.error('Failed to create user', { error, dto, userId: user.id });
      throw new InternalServerErrorException('Failed to create user');
    }
  }
}
```

## Authentication & Security

- **Use Better Auth** with `@thallesp/nestjs-better-auth` for authentication
- **Configure Better Auth** with database adapter (Prisma recommended)
- **Use guards for route protection**: `@UseGuards(AuthGuard)`
- **Implement rate limiting** with `@nestjs/throttler`
- **Validate all inputs** with class-validator
- **Use helmet** for security headers

## Better Auth Setup

```typescript
// auth.config.ts
export const auth = betterAuth({
  database: prismaAdapter(prisma),
  emailAndPassword: { enabled: true },
  socialProviders: {
    google: { 
      clientId: process.env.GOOGLE_CLIENT_ID!, 
      clientSecret: process.env.GOOGLE_CLIENT_SECRET! 
    }
  }
});

// auth.module.ts
AuthModule.forRoot(auth, { 
  disableExceptionFilter: true, 
  disableTrustedOriginsCors: true 
})

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

## Error Handling

```typescript
export class UserNotFoundException extends NotFoundException {
  constructor(id: string) {
    super(`User with ID ${id} not found`);
  }
}

export class InsufficientPermissionsException extends ForbiddenException {
  constructor(action: string, resource: string) {
    super(`Insufficient permissions to ${action} ${resource}`);
  }
}
```

## Project Structure

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
