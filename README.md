# TypeScript Coding Guidelines for NestJS + Next.js Project

This repository contains comprehensive coding guidelines and best practices for a TypeScript project consisting of:
- **NestJS API Backend** - A scalable Node.js framework for building efficient server-side applications
- **Next.js Frontend** - A React framework with server-side rendering and modern development features

## 📚 Documentation Structure

### Core Guidelines
- [🏗️ Project Structure](./docs/project-structure.md) - Overall project organization and folder structure
- [📝 TypeScript Standards](./docs/typescript-standards.md) - TypeScript configuration and coding standards
- [🎨 Code Formatting](./docs/code-formatting.md) - ESLint, Prettier, and formatting rules

### Backend (NestJS)
- [🚀 NestJS Guidelines](./docs/nestjs/README.md) - Complete NestJS development guide
- [🏛️ Architecture Patterns](./docs/nestjs/architecture.md) - Controllers, services, modules, and DTOs
- [🔒 Authentication & Security](./docs/nestjs/auth-security.md) - Auth patterns and security best practices
- [🗄️ Database Patterns](./docs/nestjs/database.md) - Database interaction and ORM patterns
- [🧪 Testing Strategies](./docs/nestjs/testing.md) - Unit, integration, and e2e testing
- [📊 Logging & Monitoring](./docs/nestjs/logging.md) - Logging and monitoring guidelines

### Frontend (Next.js)
- [⚛️ Next.js Guidelines](./docs/nextjs/README.md) - Complete Next.js development guide
- [🧩 Component Patterns](./docs/nextjs/components.md) - React component organization and best practices
- [🎨 Styling Guidelines](./docs/nextjs/styling.md) - CSS-in-JS and styling approaches
- [🔄 State Management](./docs/nextjs/state-management.md) - React state management patterns
- [🌐 API Integration](./docs/nextjs/api-integration.md) - Backend integration with React Query/SWR
- [⚡ Performance](./docs/nextjs/performance.md) - Next.js performance optimization guidelines

### Cross-Cutting Concerns
- [🔗 Shared Types](./docs/shared/types.md) - Shared TypeScript interfaces and types
- [🔄 Git Workflow](./docs/shared/git-workflow.md) - Git conventions and workflow
- [📦 Dependency Management](./docs/shared/dependencies.md) - Package management guidelines
- [🚀 CI/CD Pipeline](./docs/shared/cicd.md) - Continuous integration and deployment

## 🚀 Quick Start

### Prerequisites
- Node.js 18+ 
- npm/yarn/pnpm
- TypeScript 5+

### Project Setup
```bash
# Clone the repository
git clone <repository-url>
cd my-music

# Install dependencies for both backend and frontend
npm run install:all

# Start development servers
npm run dev
```

### Development Workflow
1. Follow the [Git Workflow](./docs/shared/git-workflow.md) for branching and commits
2. Ensure code passes linting and formatting: `npm run lint && npm run format`
3. Write tests for new features following [Testing Guidelines](./docs/nestjs/testing.md)
4. Update documentation when adding new features or patterns

## 🛠️ Configuration Files

### Root Level
- `tsconfig.json` - Base TypeScript configuration
- `.eslintrc.js` - ESLint configuration
- `.prettierrc` - Prettier formatting rules
- `package.json` - Root package configuration

### Backend (NestJS)
- `apps/api/tsconfig.json` - Backend-specific TypeScript config
- `apps/api/nest-cli.json` - NestJS CLI configuration

### Frontend (Next.js)
- `frontend/tsconfig.json` - Frontend-specific TypeScript config
- `frontend/next.config.js` - Next.js configuration

## 📋 Code Quality Standards

### Mandatory Checks
- ✅ TypeScript compilation without errors
- ✅ ESLint rules compliance
- ✅ Prettier formatting
- ✅ Unit test coverage > 80%
- ✅ Integration tests for API endpoints
- ✅ Component tests for React components

### Recommended Practices
- 📝 Comprehensive JSDoc comments for public APIs
- 🏷️ Consistent naming conventions
- 🔒 Proper error handling and validation
- 📊 Performance monitoring and logging
- 🔄 Regular dependency updates

## 🤝 Contributing

1. Read the relevant guidelines in the `docs/` directory
2. Follow the established patterns and conventions
3. Ensure all tests pass and code quality checks succeed
4. Submit pull requests with clear descriptions

## 📞 Support

For questions about these guidelines or project setup:
- Create an issue in this repository
- Refer to the detailed documentation in the `docs/` folder
- Check existing patterns in the codebase for examples

---

**Note**: This documentation is living and should be updated as the project evolves and new patterns emerge.
