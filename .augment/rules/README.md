# Augment Rules Index

This directory contains focused rule files that guide development practices for the my-music project. Each rule file is designed to be automatically detected and applied by Augment based on the context of your work.

## Rule Types

- **Always**: Applied to every conversation and code generation
- **Auto**: Automatically detected and applied based on file context
- **Manual**: Must be manually referenced with @rule-name

## Rule Files Overview

### 🚨 Critical Rules (Always Applied)

#### [`security-critical.md`](./security-critical.md) - **Type: Always**
- Authentication and authorization patterns
- Input validation requirements
- Security checklists and verification steps
- Critical security patterns that must be followed

#### [`database-api-standards.md`](./database-api-standards.md) - **Type: Always**
- Database schema requirements (ULID, soft deletes, timestamps)
- API design standards (kebab-case, pagination, validation)
- Prisma patterns and templates
- Service layer implementation patterns

#### [`naming-conventions.md`](./naming-conventions.md) - **Type: Always**
- File naming standards (PascalCase, kebab-case)
- Variable naming patterns (booleans, arrays, functions)
- API naming conventions (endpoints, params, JSON fields)
- Quick reference table for all naming patterns

#### [`typescript-standards.md`](./typescript-standards.md) - **Type: Always**
- Strict TypeScript configuration requirements
- Type organization and management
- Interface vs type usage guidelines
- Type safety patterns and examples

#### [`testing-requirements.md`](./testing-requirements.md) - **Type: Always**
- Mandatory testing coverage (80% minimum)
- Testing strategy for different code types
- Test file naming conventions
- Testing patterns and examples

### ⚙️ Framework-Specific Rules (Auto-Applied)

#### [`nextjs-frontend.md`](./nextjs-frontend.md) - **Type: Auto**
- Next.js specific patterns and requirements
- Client-side only development rules
- State management decision trees
- Component templates and UI/UX standards
- Performance optimization guidelines

#### [`nestjs-backend.md`](./nestjs-backend.md) - **Type: Auto**
- NestJS enterprise patterns
- Controller and service layer guidelines
- Module organization principles
- Authentication setup with Better Auth
- Error handling patterns

### 📋 Quality & Performance Rules (Manual)

#### [`performance-standards.md`](./performance-standards.md) - **Type: Manual**
- Performance targets and benchmarks
- Frontend optimization techniques
- Backend caching and database optimization
- Monitoring and metrics implementation
- Performance testing strategies

#### [`decision-frameworks.md`](./decision-frameworks.md) - **Type: Manual**
- Component creation decision trees
- API endpoint design frameworks
- State management choice guidelines
- Error handling decision patterns
- Testing strategy frameworks

#### [`project-setup.md`](./project-setup.md) - **Type: Manual**
- Project initialization commands
- Development environment setup
- Package.json scripts and configurations
- Docker and deployment setup
- Quick reference for common commands

## How to Use These Rules

### Automatic Application
Most rules are automatically applied based on your current work context:
- Working on React components → `nextjs-frontend.md` rules apply
- Working on NestJS controllers → `nestjs-backend.md` rules apply
- All security and naming rules are always active

### Manual Reference
For specific guidance, reference rules manually:
```
@performance-standards - for optimization guidance
@decision-frameworks - for architectural decisions
@project-setup - for setup and configuration help
```

### Rule Priority
1. **Always** rules (security, naming, TypeScript) - highest priority
2. **Auto** rules (framework-specific) - applied based on context
3. **Manual** rules (performance, decisions) - applied when referenced

## Rule File Structure

Each rule file follows this structure:
```markdown
# Rule Title

**Type**: Always|Auto|Manual
**Description**: Brief description of what this rule covers

## Section 1
- Specific, actionable guidelines
- Clear do/don't statements
- Code examples where helpful

## Section 2
- More guidelines...
```

## Updating Rules

When updating rules:
1. Keep rules specific and actionable
2. Use clear examples for complex patterns
3. Maintain the Type designation for proper application
4. Update this index if adding new rule files

## Quick Reference

| Rule File | Type | Primary Use Case |
|-----------|------|------------------|
| `security-critical.md` | Always | Authentication, validation, security patterns |
| `database-api-standards.md` | Always | Database schema, API design, service patterns |
| `naming-conventions.md` | Always | File and variable naming standards |
| `typescript-standards.md` | Always | Type safety and TypeScript configuration |
| `testing-requirements.md` | Always | Testing coverage and patterns |
| `nextjs-frontend.md` | Auto | React/Next.js development |
| `nestjs-backend.md` | Auto | NestJS backend development |
| `performance-standards.md` | Manual | Performance optimization |
| `decision-frameworks.md` | Manual | Architectural decision guidance |
| `project-setup.md` | Manual | Project initialization and setup |

## Integration with .augment-guidelines

These rule files are extracted and organized from the main `.augment-guidelines` file to provide:
- Better organization and maintainability
- Context-aware rule application
- Focused guidance for specific development tasks
- Easier updates and modifications

The original `.augment-guidelines` file remains as a comprehensive reference, while these focused rule files provide targeted guidance during development.
