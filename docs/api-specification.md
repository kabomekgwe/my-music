# API Specification
# Jazz Piano Learning SaaS Platform

**Version**: 1.0  
**Date**: 2025-01-21  
**Product**: JazzMaster Pro  
**Base URL**: `https://api.jazzmaster.pro/v1`

## 1. API Overview

### 1.1 Design Principles
- **RESTful architecture** with resource-based URLs
- **JSON-only** request and response bodies
- **Kebab-case** for endpoint URLs
- **camelCase** for JSON field names
- **Consistent error handling** with standard HTTP status codes
- **Comprehensive validation** with detailed error messages
- **Rate limiting** for API protection
- **Pagination** for list endpoints

### 1.2 Authentication
- **Bearer token authentication** using JWT
- **Token expiration**: 24 hours
- **Refresh token rotation** for security
- **Rate limiting**: 1000 requests/hour per user

### 1.3 Response Format
```json
{
  "data": {}, // Response data
  "success": true,
  "message": "Operation completed successfully",
  "meta": {
    "timestamp": "2025-01-21T10:00:00Z",
    "requestId": "req_123456789"
  }
}
```

### 1.4 Error Format
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  },
  "success": false,
  "meta": {
    "timestamp": "2025-01-21T10:00:00Z",
    "requestId": "req_123456789"
  }
}
```

## 2. Authentication Endpoints

### 2.1 User Registration
```http
POST /auth/register
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "firstName": "John",
  "lastName": "Doe"
}
```

**Response (201 Created)**:
```json
{
  "data": {
    "user": {
      "id": "user_01HN123456789",
      "email": "user@example.com",
      "firstName": "John",
      "lastName": "Doe",
      "isEmailVerified": false
    },
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "refresh_token_here"
  },
  "success": true,
  "message": "User registered successfully"
}
```

### 2.2 User Login
```http
POST /auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

### 2.3 Token Refresh
```http
POST /auth/refresh
Content-Type: application/json

{
  "refreshToken": "refresh_token_here"
}
```

### 2.4 Logout
```http
POST /auth/logout
Authorization: Bearer {accessToken}
```

## 3. User Management Endpoints

### 3.1 Get Current User
```http
GET /users/me
Authorization: Bearer {accessToken}
```

**Response (200 OK)**:
```json
{
  "data": {
    "id": "user_01HN123456789",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "profile": {
      "skillLevel": "INTERMEDIATE",
      "musicalBackground": {
        "yearsPlaying": 5,
        "instruments": ["piano"],
        "genres": ["jazz", "classical"]
      },
      "learningGoals": ["improvisation", "chord_voicings"],
      "preferences": {
        "defaultKey": "C",
        "defaultTempo": 120
      }
    },
    "subscription": {
      "tier": "PRO",
      "status": "ACTIVE",
      "currentPeriodEnd": "2025-02-21T00:00:00Z"
    }
  }
}
```

### 3.2 Update User Profile
```http
PATCH /users/me
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "firstName": "John",
  "lastName": "Smith",
  "profile": {
    "skillLevel": "ADVANCED",
    "learningGoals": ["improvisation", "composition"]
  }
}
```

## 4. AI Content Generation Endpoints

### 4.1 Generate Jazz Lick
```http
POST /ai/generate/jazz-lick
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "key": "C",
  "difficulty": "INTERMEDIATE",
  "style": "bebop",
  "tempo": 120,
  "length": 8,
  "parameters": {
    "scaleType": "major",
    "rhythmComplexity": "medium",
    "noteRange": {
      "low": "C4",
      "high": "C6"
    }
  }
}
```

**Response (201 Created)**:
```json
{
  "data": {
    "id": "content_01HN987654321",
    "type": "JAZZ_LICK",
    "title": "Bebop Lick in C Major",
    "description": "8-bar bebop lick with chromatic approach notes",
    "musicData": {
      "notes": [
        {
          "pitch": "C5",
          "duration": "8n",
          "time": "0:0:0"
        }
      ],
      "timeSignature": "4/4",
      "keySignature": "C"
    },
    "musicXML": "<?xml version=\"1.0\"?>...",
    "audioUrl": "https://cdn.jazzmaster.pro/audio/content_01HN987654321.mp3",
    "difficulty": "INTERMEDIATE",
    "key": "C",
    "tempo": 120,
    "style": "bebop",
    "tags": ["bebop", "chromatic", "intermediate"],
    "generationParameters": {
      "key": "C",
      "difficulty": "INTERMEDIATE",
      "style": "bebop"
    },
    "createdAt": "2025-01-21T10:00:00Z"
  }
}
```

### 4.2 Generate Chord Progression
```http
POST /ai/generate/chord-progression
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "key": "F",
  "length": 16,
  "style": "swing",
  "complexity": "medium",
  "parameters": {
    "includeSubstitutions": true,
    "voicingStyle": "rootless",
    "rhythmPattern": "swing"
  }
}
```

### 4.3 Generate Scale Pattern
```http
POST /ai/generate/scale-pattern
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "scaleType": "dorian",
  "key": "D",
  "difficulty": "BEGINNER",
  "patternType": "ascending",
  "parameters": {
    "octaves": 2,
    "rhythmPattern": "quarter_notes",
    "fingering": true
  }
}
```

## 5. Content Management Endpoints

### 5.1 Get User's Generated Content
```http
GET /content/generated?page=1&limit=20&type=JAZZ_LICK&difficulty=INTERMEDIATE
Authorization: Bearer {accessToken}
```

**Response (200 OK)**:
```json
{
  "data": {
    "items": [
      {
        "id": "content_01HN987654321",
        "type": "JAZZ_LICK",
        "title": "Bebop Lick in C Major",
        "difficulty": "INTERMEDIATE",
        "key": "C",
        "style": "bebop",
        "createdAt": "2025-01-21T10:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 45,
      "totalPages": 3,
      "hasNext": true,
      "hasPrev": false
    }
  }
}
```

### 5.2 Get Specific Content
```http
GET /content/generated/{contentId}
Authorization: Bearer {accessToken}
```

### 5.3 Save Content to Library
```http
POST /content/saved
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "generatedContentId": "content_01HN987654321",
  "collectionName": "Bebop Studies",
  "notes": "Great for practicing chromatic approach notes",
  "isFavorite": true
}
```

### 5.4 Get Saved Content
```http
GET /content/saved?page=1&limit=20&collection=Bebop%20Studies
Authorization: Bearer {accessToken}
```

### 5.5 Update Saved Content
```http
PATCH /content/saved/{savedContentId}
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "notes": "Updated practice notes",
  "isFavorite": false,
  "collectionName": "Advanced Studies"
}
```

### 5.6 Delete Saved Content
```http
DELETE /content/saved/{savedContentId}
Authorization: Bearer {accessToken}
```

## 6. Progress Tracking Endpoints

### 6.1 Start Practice Session
```http
POST /practice/sessions
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "title": "Morning Practice",
  "goals": ["improvisation", "chord_voicings"],
  "plannedDuration": 60
}
```

**Response (201 Created)**:
```json
{
  "data": {
    "id": "session_01HN555666777",
    "title": "Morning Practice",
    "goals": ["improvisation", "chord_voicings"],
    "startedAt": "2025-01-21T10:00:00Z",
    "status": "ACTIVE"
  }
}
```

### 6.2 End Practice Session
```http
PATCH /practice/sessions/{sessionId}/end
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "achievements": ["Practiced bebop scales", "Worked on ii-V-I progressions"],
  "notes": "Good progress on chromatic approach notes",
  "rating": 4
}
```

### 6.3 Log Practice Record
```http
POST /practice/sessions/{sessionId}/records
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "generatedContentId": "content_01HN987654321",
  "timeSpent": 15,
  "repetitions": 10,
  "difficulty": "INTERMEDIATE",
  "performance": 85,
  "notes": "Struggled with timing at first, improved by end"
}
```

### 6.4 Get Practice History
```http
GET /practice/sessions?page=1&limit=10&from=2025-01-01&to=2025-01-31
Authorization: Bearer {accessToken}
```

### 6.5 Get Progress Analytics
```http
GET /analytics/progress?period=month&skillArea=improvisation
Authorization: Bearer {accessToken}
```

**Response (200 OK)**:
```json
{
  "data": {
    "period": "month",
    "skillArea": "improvisation",
    "totalPracticeTime": 1200,
    "sessionsCount": 24,
    "averageSessionDuration": 50,
    "skillProgress": {
      "currentLevel": "INTERMEDIATE",
      "progressPercentage": 75,
      "nextMilestone": "Advanced improvisation techniques"
    },
    "practiceDistribution": {
      "jazzLicks": 40,
      "chordProgressions": 35,
      "scalePatterns": 25
    },
    "performanceMetrics": {
      "averageScore": 82,
      "improvement": 15,
      "consistency": 78
    }
  }
}
```

## 7. Subscription Management Endpoints

### 7.1 Get Subscription Plans
```http
GET /subscriptions/plans
```

### 7.2 Create Subscription
```http
POST /subscriptions
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "priceId": "price_1234567890",
  "paymentMethodId": "pm_1234567890"
}
```

### 7.3 Update Subscription
```http
PATCH /subscriptions/{subscriptionId}
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "priceId": "price_0987654321"
}
```

### 7.4 Cancel Subscription
```http
DELETE /subscriptions/{subscriptionId}
Authorization: Bearer {accessToken}
```

### 7.5 Get Usage Statistics
```http
GET /subscriptions/usage
Authorization: Bearer {accessToken}
```

## 8. Search & Discovery Endpoints

### 8.1 Search Content
```http
GET /search/content?q=bebop&type=JAZZ_LICK&difficulty=INTERMEDIATE&key=C
Authorization: Bearer {accessToken}
```

### 8.2 Get Recommendations
```http
GET /recommendations/content?type=JAZZ_LICK&limit=10
Authorization: Bearer {accessToken}
```

## 9. Rate Limiting & Error Codes

### 9.1 Rate Limits
- **Authentication endpoints**: 10 requests/minute
- **Content generation**: 100 requests/hour (Free), Unlimited (Pro)
- **General API**: 1000 requests/hour per user
- **Search endpoints**: 200 requests/hour

### 9.2 HTTP Status Codes
- **200 OK**: Successful GET, PATCH
- **201 Created**: Successful POST
- **204 No Content**: Successful DELETE
- **400 Bad Request**: Invalid request data
- **401 Unauthorized**: Invalid or missing authentication
- **403 Forbidden**: Insufficient permissions
- **404 Not Found**: Resource not found
- **409 Conflict**: Resource conflict
- **422 Unprocessable Entity**: Validation errors
- **429 Too Many Requests**: Rate limit exceeded
- **500 Internal Server Error**: Server error

---

**Document Owner**: API Architect  
**Last Updated**: 2025-01-21  
**Next Review**: 2025-02-21
