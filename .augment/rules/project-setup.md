# Project Setup & Quick Commands

**Type**: Manual  
**Description**: Project initialization commands and development workflow setup

## Tech Stack Overview

- **Frontend**: Next.js (latest) - Client-side rendering only, no SSR/RSC
- **Backend**: NestJS (latest) with enterprise-grade patterns
- **Database**: PostgreSQL with Prisma (latest)
- **Auth**: better-auth with NestJS integration
- **State**: Zustand (global), TanStack Query (server state)
- **Forms**: react-hook-form + zod validation
- **Styling**: Tailwind CSS + shadcn/ui components
- **Testing**: Jest + React Testing Library + Playwright
- **Logging**: Pino (include request_id, user_id, tenant_id)
- **Queue**: Bull/BullMQ with Redis
- **Linting**: Biome

## Project Structure

```
my-music/
├── backend/          # NestJS API
│   ├── src/
│   │   ├── types/   # Backend-specific types and DTOs
│   │   ├── auth/
│   │   ├── users/
│   │   └── ...
│   ├── test/
│   ├── package.json
│   └── tsconfig.json
├── frontend/         # Next.js frontend
│   ├── src/
│   │   ├── types/   # Frontend-specific types and API interfaces
│   │   ├── app/
│   │   ├── components/
│   │   └── ...
│   ├── public/
│   ├── package.json
│   └── next.config.js
├── docs/             # Documentation
├── .augment/         # Augment rules
├── docker-compose.yml
└── README.md
```

## Quick Setup Commands

### Frontend Setup
```bash
# Create Next.js app
npx create-next-app@latest frontend --typescript --tailwind --app

# Navigate to frontend
cd frontend

# Install core dependencies
npm install @tanstack/react-query zustand react-hook-form zod
npm install better-auth nuqs class-variance-authority clsx tailwind-merge

# Install UI components
npm install @radix-ui/react-slot @radix-ui/react-dialog
npm install lucide-react

# Install dev dependencies
npm install -D @biomejs/biome @testing-library/react @testing-library/jest-dom
npm install -D @playwright/test eslint-config-next

# Install shadcn/ui
npx shadcn-ui@latest init
npx shadcn-ui@latest add button input label card
```

### Backend Setup
```bash
# Create NestJS app
npx @nestjs/cli new backend

# Navigate to backend
cd backend

# Install core dependencies
npm install @nestjs/swagger @nestjs/throttler @nestjs/config
npm install nestjs-pino pino-http pino-pretty
npm install prisma @prisma/client
npm install bull bullmq @nestjs/bull
npm install class-validator class-transformer
npm install @thallesp/nestjs-better-auth better-auth

# Install dev dependencies
npm install -D @biomejs/biome jest supertest
npm install -D @types/jest @types/supertest
npm install -D prisma

# Initialize Prisma
npx prisma init
```

### Database Setup
```bash
# Generate Prisma client
npx prisma generate

# Create and run migrations
npx prisma migrate dev --name init

# Seed database (optional)
npx prisma db seed
```

## Development Workflow

### Code Quality Setup
```bash
# Initialize Biome (run in both frontend and backend)
npx @biomejs/biome init

# Format code
npx @biomejs/biome format --write .

# Lint and fix
npx @biomejs/biome check --write .
```

### Git Hooks Setup
```bash
# Install husky for git hooks
npm install -D husky lint-staged

# Initialize husky
npx husky install

# Add pre-commit hook
npx husky add .husky/pre-commit "npx lint-staged"
```

### Package.json Scripts

#### Frontend package.json
```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "biome check .",
    "lint:fix": "biome check --write .",
    "format": "biome format --write .",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:e2e": "playwright test",
    "type-check": "tsc --noEmit"
  }
}
```

#### Backend package.json
```json
{
  "scripts": {
    "build": "nest build",
    "format": "biome format --write .",
    "start": "nest start",
    "start:dev": "nest start --watch",
    "start:debug": "nest start --debug --watch",
    "start:prod": "node dist/main",
    "lint": "biome check .",
    "lint:fix": "biome check --write .",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand",
    "test:e2e": "jest --config ./test/jest-e2e.json",
    "db:generate": "prisma generate",
    "db:migrate": "prisma migrate dev",
    "db:deploy": "prisma migrate deploy",
    "db:seed": "prisma db seed",
    "db:studio": "prisma studio"
  }
}
```

## Environment Configuration

### Frontend (.env.local)
```bash
NEXT_PUBLIC_API_URL=http://localhost:3001
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Backend (.env)
```bash
# Database
DATABASE_URL="postgresql://username:password@localhost:5432/mymusic"

# Auth
BETTER_AUTH_SECRET="your-secret-key"
BETTER_AUTH_URL="http://localhost:3001"

# External Services
GOOGLE_CLIENT_ID="your-google-client-id"
GOOGLE_CLIENT_SECRET="your-google-client-secret"

# Redis (for queues and caching)
REDIS_URL="redis://localhost:6379"

# Logging
LOG_LEVEL="info"
```

## Docker Setup

### docker-compose.yml
```yaml
version: '3.8'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: mymusic
      POSTGRES_USER: username
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

### Start Development Environment
```bash
# Start database and Redis
docker-compose up -d

# Start backend (in backend directory)
npm run start:dev

# Start frontend (in frontend directory)
npm run dev
```

## Deployment Commands

### Build for Production
```bash
# Frontend
npm run build
npm run start

# Backend
npm run build
npm run start:prod
```

### Database Deployment
```bash
# Deploy migrations to production
npx prisma migrate deploy

# Generate Prisma client
npx prisma generate
```

## Useful Development Commands

```bash
# Generate new NestJS resource
nest g resource users

# Generate Prisma migration
npx prisma migrate dev --name add_user_table

# Reset database
npx prisma migrate reset

# View database
npx prisma studio

# Add shadcn/ui component
npx shadcn-ui@latest add button

# Run tests with coverage
npm run test:cov

# Run E2E tests
npm run test:e2e
```
