# Database & API Standards

**Type**: Always  
**Description**: Mandatory database and API design patterns for consistency and reliability

## Database Requirements

- **Use ULID for all primary keys** (Prisma: `@id @default(cuid())`, convert to ULID in app layer)
- **Implement soft deletes** with `deleted_at` column on ALL tables
- **Include audit logging** for all create/update/delete operations
- **Always include timestamps**: `created_at`, `updated_at`, `deleted_at` on every table
- **Use Prisma parameterized queries** to prevent SQL injection

## API Design Standards

- **Use kebab-case for API endpoints**: `/api/v1/user-profiles/{id}`
- **Paginate all list endpoints**: `?page=1&limit=20` (default limit=20, max=100)
- **Use proper HTTP status codes** with `@HttpCode()` decorator
- **Validate all requests** with DTOs and `ValidationPipe`
- **Use Swagger decorators** for API documentation: `@ApiOperation`, `@ApiTags`, `@ApiBody`

## Database Schema Template

```prisma
model ExampleTable {
  id        String    @id @default(cuid()) // Use ULID in app layer
  // ... your fields here
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  deletedAt DateTime? // Soft delete

  @@map("example_table")
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

  @@map("audit_logs")
}
```

## API Endpoint Template

```typescript
@Controller('resource-name')
@ApiTags('resource-name')
@UseGuards(AuthGuard)
export class ResourceController {
  constructor(private readonly resourceService: ResourceService) {}

  @Get()
  @ApiOperation({ summary: 'List resources with pagination' })
  @ApiQuery({ name: 'page', required: false, type: Number })
  @ApiQuery({ name: 'limit', required: false, type: Number })
  async findAll(
    @Query() query: FindResourcesQueryDto,
    @CurrentUser() user: User,
  ): Promise<PaginatedResponse<ResourceResponseDto>> {
    return this.resourceService.findAll(query, user);
  }

  @Post()
  @ApiOperation({ summary: 'Create new resource' })
  @ApiBody({ type: CreateResourceDto })
  async create(
    @Body() createDto: CreateResourceDto,
    @CurrentUser() user: User,
  ): Promise<ResourceResponseDto> {
    return this.resourceService.create(createDto, user);
  }
}
```

## Service Layer Pattern

```typescript
@Injectable()
export class ResourceService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly logger: Logger,
  ) {}

  async findAll(
    query: FindResourcesQueryDto,
    user: User,
  ): Promise<PaginatedResponse<ResourceResponseDto>> {
    try {
      const { page = 1, limit = 20 } = query;
      const skip = (page - 1) * limit;

      const [items, total] = await Promise.all([
        this.prisma.resource.findMany({
          where: { deletedAt: null, ...this.buildWhereClause(query) },
          skip,
          take: Math.min(limit, 100), // Max 100 items
          orderBy: { createdAt: 'desc' },
        }),
        this.prisma.resource.count({
          where: { deletedAt: null, ...this.buildWhereClause(query) },
        }),
      ]);

      return {
        data: items.map(item => this.mapToResponseDto(item)),
        pagination: {
          page,
          limit,
          total,
          totalPages: Math.ceil(total / limit),
        },
      };
    } catch (error) {
      this.logger.error('Failed to fetch resources', { error, query, userId: user.id });
      throw new InternalServerErrorException('Failed to fetch resources');
    }
  }
}
```
