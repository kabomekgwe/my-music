# Technical Architecture
# Jazz Piano Learning SaaS Platform

**Version**: 1.0  
**Date**: 2025-01-21  
**Product**: JazzMaster Pro  

## 1. System Overview

### 1.1 Architecture Principles
- **Microservices-oriented**: Modular, scalable backend services
- **API-first**: RESTful APIs with OpenAPI documentation
- **Cloud-native**: Designed for cloud deployment and scaling
- **Security-first**: Zero-trust architecture with comprehensive validation
- **Performance-optimized**: Sub-200ms API responses, sub-3s page loads

### 1.2 High-Level Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Backend       │    │   External      │
│   (Next.js)     │◄──►│   (NestJS)      │◄──►│   Services      │
│                 │    │                 │    │                 │
│ • React UI      │    │ • REST APIs     │    │ • OpenAI API    │
│ • Audio Player  │    │ • Auth Service  │    │ • Stripe API    │
│ • Music Notation│    │ • AI Service    │    │ • AWS S3        │
│ • State Mgmt    │    │ • Content Mgmt  │    │ • Email Service │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │   Database      │
                    │   (PostgreSQL)  │
                    │                 │
                    │ • User Data     │
                    │ • Content       │
                    │ • AI Generations│
                    │ • Analytics     │
                    └─────────────────┘
```

## 2. Frontend Architecture (Next.js)

### 2.1 Technology Stack
- **Framework**: Next.js 14+ with App Router
- **Language**: TypeScript (strict mode)
- **Styling**: Tailwind CSS + shadcn/ui
- **State Management**: 
  - Zustand (global UI state)
  - TanStack Query (server state)
  - React Hook Form + Zod (forms)
- **Audio**: Web Audio API + Tone.js
- **Music Notation**: VexFlow
- **Authentication**: Better Auth React client

### 2.2 Project Structure
```
frontend/
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── (auth)/            # Authentication pages
│   │   ├── (dashboard)/       # Main application
│   │   ├── api/               # API routes (minimal)
│   │   └── globals.css        # Global styles
│   ├── components/            # Reusable components
│   │   ├── ui/               # Basic UI components
│   │   ├── music/            # Music-specific components
│   │   ├── ai/               # AI generation components
│   │   └── layout/           # Layout components
│   ├── lib/                  # Utilities and configurations
│   │   ├── audio/            # Audio processing utilities
│   │   ├── music/            # Music theory utilities
│   │   ├── api/              # API client
│   │   └── auth/             # Authentication client
│   ├── stores/               # Zustand stores
│   ├── hooks/                # Custom React hooks
│   └── types/                # TypeScript type definitions
├── public/                   # Static assets
└── docs/                     # Component documentation
```

### 2.3 Key Components

#### 2.3.1 Music Notation Component
```typescript
interface MusicNotationProps {
  musicXML: string;
  width?: number;
  height?: number;
  interactive?: boolean;
  onNoteClick?: (note: Note) => void;
}

export function MusicNotation({ musicXML, ...props }: MusicNotationProps) {
  // VexFlow integration for rendering standard notation
}
```

#### 2.3.2 Audio Player Component
```typescript
interface AudioPlayerProps {
  audioUrl?: string;
  musicData?: MusicData;
  tempo?: number;
  loop?: boolean;
  onPlaybackChange?: (isPlaying: boolean) => void;
}

export function AudioPlayer({ musicData, ...props }: AudioPlayerProps) {
  // Tone.js integration for audio synthesis and playback
}
```

#### 2.3.3 AI Generation Interface
```typescript
interface AIGeneratorProps {
  type: 'lick' | 'progression' | 'scale' | 'harmony';
  parameters: GenerationParameters;
  onGenerate: (result: GeneratedContent) => void;
}

export function AIGenerator({ type, parameters, onGenerate }: AIGeneratorProps) {
  // Interface for AI content generation
}
```

### 2.4 State Management Strategy

#### 2.4.1 Global UI State (Zustand)
```typescript
interface AppState {
  theme: 'light' | 'dark';
  sidebarOpen: boolean;
  currentKey: string;
  tempo: number;
  isPlaying: boolean;
}
```

#### 2.4.2 Server State (TanStack Query)
```typescript
// User content queries
const useUserContent = () => useQuery({
  queryKey: ['user-content'],
  queryFn: () => api.getUserContent(),
});

// AI generation mutations
const useGenerateContent = () => useMutation({
  mutationFn: (params: GenerationParams) => api.generateContent(params),
  onSuccess: (data) => {
    queryClient.invalidateQueries(['user-content']);
  },
});
```

## 3. Backend Architecture (NestJS)

### 3.1 Technology Stack
- **Framework**: NestJS with TypeScript
- **Database**: PostgreSQL with Prisma ORM
- **Authentication**: Better Auth with JWT
- **Validation**: class-validator + Zod
- **Documentation**: Swagger/OpenAPI
- **Logging**: Pino with structured logging
- **Caching**: Redis for session and content caching
- **File Storage**: AWS S3 for audio files
- **Queue**: Bull/BullMQ for background processing

### 3.2 Project Structure
```
backend/
├── src/
│   ├── auth/                  # Authentication module
│   ├── users/                 # User management
│   ├── content/               # Content management
│   ├── ai/                    # AI generation services
│   ├── music/                 # Music theory utilities
│   ├── audio/                 # Audio processing
│   ├── subscriptions/         # Billing and subscriptions
│   ├── analytics/             # User analytics
│   ├── common/                # Shared utilities
│   │   ├── decorators/        # Custom decorators
│   │   ├── guards/            # Authentication guards
│   │   ├── pipes/             # Validation pipes
│   │   └── filters/           # Exception filters
│   ├── config/                # Configuration
│   ├── database/              # Database migrations
│   └── types/                 # TypeScript definitions
├── test/                      # Integration tests
└── docs/                      # API documentation
```

### 3.3 Core Modules

#### 3.3.1 AI Generation Service
```typescript
@Injectable()
export class AIGenerationService {
  async generateJazzLick(params: LickGenerationParams): Promise<GeneratedLick> {
    // Integration with OpenAI or custom music AI models
    // Generate unique melodic phrases based on parameters
  }

  async generateChordProgression(params: ProgressionParams): Promise<GeneratedProgression> {
    // Generate chord progressions with variations
  }

  async generateScalePattern(params: ScaleParams): Promise<GeneratedScale> {
    // Create scale practice exercises
  }

  async harmonizeMelody(melody: Melody): Promise<HarmonizedMelody> {
    // AI-powered melody harmonization
  }
}
```

#### 3.3.2 Music Theory Service
```typescript
@Injectable()
export class MusicTheoryService {
  analyzeChordProgression(progression: ChordProgression): ProgressionAnalysis {
    // Analyze harmonic content and suggest improvements
  }

  generateScaleExercises(scale: Scale, difficulty: Difficulty): Exercise[] {
    // Create practice exercises for specific scales
  }

  validateMusicTheory(content: MusicContent): ValidationResult {
    // Ensure generated content follows music theory rules
  }
}
```

#### 3.3.3 Audio Processing Service
```typescript
@Injectable()
export class AudioProcessingService {
  async generateAudio(musicData: MusicData): Promise<AudioFile> {
    // Convert music notation to audio using synthesis
  }

  async processAudioFile(file: Buffer): Promise<ProcessedAudio> {
    // Process uploaded audio files
  }

  async createMIDI(musicData: MusicData): Promise<MIDIFile> {
    // Generate MIDI files from music notation
  }
}
```

### 3.4 Database Design Principles
- **ULID primary keys** for all tables
- **Soft deletes** with `deleted_at` column
- **Audit logging** for all mutations
- **Timestamps** on all records
- **Proper indexing** for performance
- **Data normalization** with relationship integrity

## 4. AI Integration Architecture

### 4.1 AI Service Layer
```typescript
interface AIProvider {
  generateContent(type: ContentType, params: any): Promise<GeneratedContent>;
  validateContent(content: MusicContent): Promise<ValidationResult>;
  analyzeUserProgress(history: UserHistory): Promise<ProgressAnalysis>;
}

// Multiple AI provider implementations
class OpenAIProvider implements AIProvider { }
class CustomModelProvider implements AIProvider { }
```

### 4.2 Content Generation Pipeline
1. **Parameter Validation**: Validate user input parameters
2. **AI Model Selection**: Choose appropriate model based on content type
3. **Content Generation**: Generate raw musical content
4. **Music Theory Validation**: Ensure theoretical correctness
5. **Quality Assessment**: Rate content difficulty and quality
6. **Format Conversion**: Convert to standard notation formats
7. **Audio Synthesis**: Generate corresponding audio
8. **Storage**: Save to database and file storage

### 4.3 Caching Strategy
- **Generated Content**: Cache frequently requested generations
- **User Preferences**: Cache user settings and preferences
- **Audio Files**: CDN caching for audio content
- **API Responses**: Redis caching for expensive operations

## 5. Security Architecture

### 5.1 Authentication Flow
1. **User Registration/Login**: Better Auth with email/password + social
2. **JWT Token Generation**: Secure token with expiration
3. **Token Validation**: Middleware validation on protected routes
4. **Session Management**: Redis-based session storage
5. **Refresh Token Rotation**: Automatic token refresh

### 5.2 Authorization Patterns
```typescript
@UseGuards(AuthGuard, RoleGuard)
@Roles('user', 'educator', 'admin')
@Controller('content')
export class ContentController {
  @Post()
  @RequirePermission('content:create')
  async createContent(@CurrentUser() user: User, @Body() dto: CreateContentDto) {
    // Permission-based access control
  }
}
```

### 5.3 Data Protection
- **Input Validation**: Comprehensive validation at all entry points
- **SQL Injection Prevention**: Prisma parameterized queries
- **XSS Protection**: Content sanitization and CSP headers
- **Rate Limiting**: API rate limiting per user/IP
- **Audit Logging**: Complete audit trail for all actions

## 6. Performance Optimization

### 6.1 Frontend Performance
- **Code Splitting**: Route-based and component-based splitting
- **Lazy Loading**: Defer non-critical component loading
- **Audio Optimization**: Efficient audio loading and caching
- **Notation Rendering**: Optimized VexFlow rendering
- **State Optimization**: Minimal re-renders with proper memoization

### 6.2 Backend Performance
- **Database Optimization**: Proper indexing and query optimization
- **Caching Layers**: Multi-level caching strategy
- **Background Processing**: Queue-based processing for heavy operations
- **Connection Pooling**: Efficient database connection management
- **CDN Integration**: Global content delivery for static assets

### 6.3 Monitoring & Observability
- **Application Monitoring**: Real-time performance metrics
- **Error Tracking**: Comprehensive error logging and alerting
- **User Analytics**: Detailed usage analytics and insights
- **Performance Metrics**: API response times and throughput
- **Health Checks**: Automated system health monitoring

---

**Document Owner**: Technical Lead  
**Last Updated**: 2025-01-21  
**Next Review**: 2025-02-21
