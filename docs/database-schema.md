# Database Schema Design
# Jazz Piano Learning SaaS Platform

**Version**: 1.0  
**Date**: 2025-01-21  
**Product**: JazzMaster Pro  

## 1. Schema Overview

### 1.1 Design Principles
- **ULID primary keys** for all tables
- **Soft deletes** with `deleted_at` column
- **Audit timestamps** on all records
- **Proper indexing** for performance
- **Foreign key constraints** for data integrity
- **JSON columns** for flexible metadata storage

### 1.2 Database Technology
- **Database**: PostgreSQL 15+
- **ORM**: Prisma
- **Migrations**: Prisma Migrate
- **Indexing**: B-tree, GIN, and partial indexes
- **Full-text search**: PostgreSQL built-in search

## 2. Core Schema

### 2.1 User Management

```prisma
model User {
  id                String    @id @default(cuid())
  email             String    @unique
  emailVerified     DateTime?
  firstName         String
  lastName          String
  username          String?   @unique
  avatar            String?
  timezone          String    @default("UTC")
  locale            String    @default("en")
  isActive          Boolean   @default(true)
  lastLoginAt       DateTime?
  createdAt         DateTime  @default(now())
  updatedAt         DateTime  @updatedAt
  deletedAt         DateTime?

  // Relationships
  profile           UserProfile?
  sessions          Session[]
  accounts          Account[]
  subscriptions     Subscription[]
  generatedContent  GeneratedContent[]
  savedContent      SavedContent[]
  progressRecords   ProgressRecord[]
  practiceSession   PracticeSession[]
  auditLogs         AuditLog[]

  @@map("users")
  @@index([email])
  @@index([username])
  @@index([isActive, deletedAt])
}

model UserProfile {
  id                String    @id @default(cuid())
  userId            String    @unique
  skillLevel        SkillLevel @default(BEGINNER)
  musicalBackground Json?     // Previous experience, instruments, etc.
  learningGoals     Json?     // User's learning objectives
  preferences       Json?     // UI preferences, default settings
  practiceSchedule  Json?     // Preferred practice times and duration
  createdAt         DateTime  @default(now())
  updatedAt         DateTime  @updatedAt
  deletedAt         DateTime?

  // Relationships
  user              User      @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("user_profiles")
  @@index([skillLevel])
}

enum SkillLevel {
  BEGINNER
  INTERMEDIATE
  ADVANCED
  PROFESSIONAL
}
```

### 2.2 Authentication & Sessions

```prisma
model Account {
  id                String    @id @default(cuid())
  userId            String
  type              String    // oauth, email, etc.
  provider          String    // google, facebook, email
  providerAccountId String
  refreshToken      String?
  accessToken       String?
  expiresAt         Int?
  tokenType         String?
  scope             String?
  idToken           String?
  sessionState      String?
  createdAt         DateTime  @default(now())
  updatedAt         DateTime  @updatedAt

  // Relationships
  user              User      @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
  @@map("accounts")
}

model Session {
  id           String    @id @default(cuid())
  sessionToken String    @unique
  userId       String
  expires      DateTime
  createdAt    DateTime  @default(now())
  updatedAt    DateTime  @updatedAt

  // Relationships
  user         User      @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("sessions")
  @@index([expires])
}

model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime

  @@unique([identifier, token])
  @@map("verification_tokens")
}
```

### 2.3 Subscription & Billing

```prisma
model Subscription {
  id                String            @id @default(cuid())
  userId            String
  stripeCustomerId  String?           @unique
  stripeSubscriptionId String?        @unique
  stripePriceId     String?
  status            SubscriptionStatus @default(INACTIVE)
  tier              SubscriptionTier   @default(FREE)
  currentPeriodStart DateTime?
  currentPeriodEnd   DateTime?
  cancelAtPeriodEnd  Boolean          @default(false)
  canceledAt         DateTime?
  trialStart         DateTime?
  trialEnd           DateTime?
  metadata           Json?
  createdAt          DateTime         @default(now())
  updatedAt          DateTime         @updatedAt
  deletedAt          DateTime?

  // Relationships
  user               User             @relation(fields: [userId], references: [id], onDelete: Cascade)
  usageRecords       UsageRecord[]

  @@map("subscriptions")
  @@index([status])
  @@index([tier])
  @@index([currentPeriodEnd])
}

enum SubscriptionStatus {
  ACTIVE
  INACTIVE
  PAST_DUE
  CANCELED
  UNPAID
  TRIALING
}

enum SubscriptionTier {
  FREE
  PRO
  EDUCATOR
}

model UsageRecord {
  id             String       @id @default(cuid())
  subscriptionId String
  feature        String       // 'ai_generation', 'audio_export', etc.
  quantity       Int          @default(1)
  metadata       Json?
  recordedAt     DateTime     @default(now())

  // Relationships
  subscription   Subscription @relation(fields: [subscriptionId], references: [id], onDelete: Cascade)

  @@map("usage_records")
  @@index([subscriptionId, feature])
  @@index([recordedAt])
}
```

### 2.4 Content Management

```prisma
model GeneratedContent {
  id              String        @id @default(cuid())
  userId          String
  type            ContentType
  title           String
  description     String?
  parameters      Json          // Generation parameters used
  musicData       Json          // Musical content (notes, chords, etc.)
  musicXML        String?       // Standard music notation
  audioUrl        String?       // Generated audio file URL
  difficulty      Difficulty    @default(INTERMEDIATE)
  key             String?       // Musical key
  tempo           Int?          // BPM
  timeSignature   String?       // e.g., "4/4"
  style           String?       // Jazz style/genre
  tags            String[]      // Searchable tags
  isPublic        Boolean       @default(false)
  likes           Int           @default(0)
  views           Int           @default(0)
  metadata        Json?         // Additional metadata
  createdAt       DateTime      @default(now())
  updatedAt       DateTime      @updatedAt
  deletedAt       DateTime?

  // Relationships
  user            User          @relation(fields: [userId], references: [id], onDelete: Cascade)
  savedBy         SavedContent[]
  practiceRecords PracticeRecord[]

  @@map("generated_content")
  @@index([type])
  @@index([difficulty])
  @@index([key])
  @@index([style])
  @@index([isPublic, deletedAt])
  @@index([userId, createdAt])
}

enum ContentType {
  JAZZ_LICK
  CHORD_PROGRESSION
  SCALE_PATTERN
  HARMONY_EXERCISE
  MELODY
  COMPLETE_PIECE
}

enum Difficulty {
  BEGINNER
  INTERMEDIATE
  ADVANCED
  EXPERT
}

model SavedContent {
  id                 String           @id @default(cuid())
  userId             String
  generatedContentId String
  collectionName     String?          // User-defined collection
  notes              String?          // User's personal notes
  isFavorite         Boolean          @default(false)
  createdAt          DateTime         @default(now())
  updatedAt          DateTime         @updatedAt

  // Relationships
  user               User             @relation(fields: [userId], references: [id], onDelete: Cascade)
  generatedContent   GeneratedContent @relation(fields: [generatedContentId], references: [id], onDelete: Cascade)

  @@unique([userId, generatedContentId])
  @@map("saved_content")
  @@index([userId, collectionName])
  @@index([isFavorite])
}
```

### 2.5 Learning & Progress Tracking

```prisma
model ProgressRecord {
  id                 String           @id @default(cuid())
  userId             String
  generatedContentId String?          // Optional: specific content practiced
  skillArea          String           // e.g., "improvisation", "chord_voicings"
  difficultyLevel    Difficulty
  score              Float?           // 0-100 performance score
  timeSpent          Int              // Minutes spent
  completionRate     Float?           // 0-100 percentage
  notes              String?          // User or system notes
  metadata           Json?            // Additional tracking data
  recordedAt         DateTime         @default(now())

  // Relationships
  user               User             @relation(fields: [userId], references: [id], onDelete: Cascade)
  generatedContent   GeneratedContent? @relation(fields: [generatedContentId], references: [id], onDelete: SetNull)

  @@map("progress_records")
  @@index([userId, skillArea])
  @@index([userId, recordedAt])
  @@index([difficultyLevel])
}

model PracticeSession {
  id              String           @id @default(cuid())
  userId          String
  title           String?
  duration        Int              // Minutes
  goals           String[]         // Session goals
  achievements    String[]         // What was accomplished
  notes           String?          // Session notes
  rating          Int?             // 1-5 session rating
  metadata        Json?            // Additional session data
  startedAt       DateTime
  endedAt         DateTime?
  createdAt       DateTime         @default(now())
  updatedAt       DateTime         @updatedAt

  // Relationships
  user            User             @relation(fields: [userId], references: [id], onDelete: Cascade)
  practiceRecords PracticeRecord[]

  @@map("practice_sessions")
  @@index([userId, startedAt])
}

model PracticeRecord {
  id                 String           @id @default(cuid())
  practiceSessionId  String
  generatedContentId String
  timeSpent          Int              // Minutes on this content
  repetitions        Int              @default(1)
  difficulty         Difficulty
  performance        Float?           // 0-100 performance score
  notes              String?
  createdAt          DateTime         @default(now())

  // Relationships
  practiceSession    PracticeSession  @relation(fields: [practiceSessionId], references: [id], onDelete: Cascade)
  generatedContent   GeneratedContent @relation(fields: [generatedContentId], references: [id], onDelete: Cascade)

  @@map("practice_records")
  @@index([practiceSessionId])
  @@index([generatedContentId])
}
```

### 2.6 System & Analytics

```prisma
model AuditLog {
  id         String    @id @default(cuid())
  userId     String?
  action     String    // CREATE, UPDATE, DELETE, LOGIN, etc.
  entity     String    // Table/resource name
  entityId   String?   // ID of affected record
  oldData    Json?     // Previous state
  newData    Json?     // New state
  metadata   Json?     // Additional context
  ipAddress  String?
  userAgent  String?
  createdAt  DateTime  @default(now())

  // Relationships
  user       User?     @relation(fields: [userId], references: [id], onDelete: SetNull)

  @@map("audit_logs")
  @@index([userId])
  @@index([action])
  @@index([entity])
  @@index([createdAt])
}

model SystemMetrics {
  id          String    @id @default(cuid())
  metricName  String    // e.g., "daily_active_users", "ai_generations"
  value       Float
  dimensions  Json?     // Additional metric dimensions
  recordedAt  DateTime  @default(now())

  @@map("system_metrics")
  @@index([metricName, recordedAt])
}

model FeatureFlag {
  id          String    @id @default(cuid())
  name        String    @unique
  description String?
  isEnabled   Boolean   @default(false)
  rolloutPercentage Float @default(0) // 0-100
  conditions  Json?     // Targeting conditions
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt

  @@map("feature_flags")
}
```

## 3. Indexes & Performance

### 3.1 Critical Indexes
```sql
-- User lookup indexes
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);
CREATE INDEX CONCURRENTLY idx_users_active ON users(is_active, deleted_at);

-- Content search indexes
CREATE INDEX CONCURRENTLY idx_content_search ON generated_content 
  USING GIN(to_tsvector('english', title || ' ' || description));
CREATE INDEX CONCURRENTLY idx_content_public ON generated_content(is_public, deleted_at);
CREATE INDEX CONCURRENTLY idx_content_user_date ON generated_content(user_id, created_at DESC);

-- Progress tracking indexes
CREATE INDEX CONCURRENTLY idx_progress_user_skill ON progress_records(user_id, skill_area);
CREATE INDEX CONCURRENTLY idx_progress_date ON progress_records(user_id, recorded_at DESC);

-- Subscription indexes
CREATE INDEX CONCURRENTLY idx_subscription_status ON subscriptions(status, current_period_end);
```

### 3.2 Partitioning Strategy
```sql
-- Partition audit logs by month for performance
CREATE TABLE audit_logs_y2025m01 PARTITION OF audit_logs
  FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');

-- Partition metrics by date
CREATE TABLE system_metrics_y2025m01 PARTITION OF system_metrics
  FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');
```

## 4. Data Migration Strategy

### 4.1 Initial Setup
```bash
# Initialize Prisma
npx prisma init

# Generate initial migration
npx prisma migrate dev --name init

# Generate Prisma client
npx prisma generate

# Seed initial data
npx prisma db seed
```

### 4.2 Seed Data
```typescript
// prisma/seed.ts
const seedData = {
  users: [
    // Demo users for development
  ],
  featureFlags: [
    { name: 'ai_generation_v2', isEnabled: false },
    { name: 'advanced_analytics', isEnabled: true },
  ],
  systemMetrics: [
    // Initial metrics
  ]
};
```

---

**Document Owner**: Database Architect  
**Last Updated**: 2025-01-21  
**Next Review**: 2025-02-21
