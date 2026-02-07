# Misconception AI Detector - Project Documentation

> **Version:** 1.0.0  
> **Last Updated:** February 2026  
> **Status:** Production Ready

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Technology Stack](#technology-stack)
4. [Features](#features)
5. [Database Schema](#database-schema)
6. [API Reference](#api-reference)
7. [Authentication](#authentication)
8. [Security](#security)
9. [User Interface](#user-interface)
10. [Deployment](#deployment)
11. [Environment Variables](#environment-variables)

---

## Project Overview

### Mission Statement

Misconception AI Detector is an AI-powered educational diagnostic tool that focuses on **identifying and correcting student conceptual misunderstandings** rather than simply marking answers right or wrong. The system analyzes student responses to reveal reasoning gaps and provides targeted feedback with memory aids.

### Key Value Propositions

- **PDF Processing**: Upload study materials and automatically generate diagnostic questions
- **Misconception Detection**: AI analyzes responses to identify flawed mental models
- **Personalized Feedback**: Corrective explanations with "Memory Tips" for retention
- **Progress Tracking**: Historical data and analytics to monitor learning improvement

---

## Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Frontend (React/Vite)                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐ │
│  │  Index   │  │  Auth    │  │ History  │  │    Dashboard     │ │
│  │  (Home)  │  │  Page    │  │  Page    │  │  (Analytics)     │ │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────────┬─────────┘ │
│       │             │             │                  │           │
│       └─────────────┴─────────────┴──────────────────┘           │
│                              │                                    │
│                    ┌─────────┴─────────┐                         │
│                    │   AuthContext     │                         │
│                    │   (Supabase Auth) │                         │
│                    └─────────┬─────────┘                         │
└──────────────────────────────┼───────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Lovable Cloud Backend                         │
│  ┌────────────────────────┐  ┌────────────────────────────────┐ │
│  │   Edge Functions       │  │        Database                │ │
│  │  ┌──────────────────┐  │  │  ┌──────────────────────────┐  │ │
│  │  │  process-pdf     │  │  │  │  diagnostic_sessions     │  │ │
│  │  │  (PDF → Questions)│  │  │  ├──────────────────────────┤  │ │
│  │  └──────────────────┘  │  │  │  question_responses      │  │ │
│  │  ┌──────────────────┐  │  │  ├──────────────────────────┤  │ │
│  │  │  analyze-answer  │  │  │  │  profiles                │  │ │
│  │  │  (AI Feedback)   │  │  │  └──────────────────────────┘  │ │
│  │  └──────────────────┘  │  │                                │ │
│  └────────────────────────┘  └────────────────────────────────┘ │
│                    │                                             │
│                    ▼                                             │
│           ┌────────────────┐                                     │
│           │  Lovable AI    │                                     │
│           │  (Gemini 3)    │                                     │
│           └────────────────┘                                     │
└─────────────────────────────────────────────────────────────────┘
```

### Component Structure

```
src/
├── components/
│   ├── ui/                    # shadcn/ui components
│   ├── AppHeader.tsx          # Authenticated user header
│   ├── AppHome.tsx            # Main upload interface
│   ├── DiagnosticSession.tsx  # Question/answer flow
│   ├── FeedbackCard.tsx       # AI feedback display
│   ├── FeaturesSection.tsx    # Landing page features
│   ├── Footer.tsx             # Consistent footer
│   ├── Hero.tsx               # Landing page hero
│   ├── HowItWorksSection.tsx  # Process timeline
│   ├── MultipleChoiceCard.tsx # MCQ component
│   ├── NavLink.tsx            # Navigation helper
│   ├── PublicHeader.tsx       # Public landing header
│   └── QuestionCard.tsx       # Short-response component
├── contexts/
│   └── AuthContext.tsx        # Authentication state
├── hooks/
│   ├── use-mobile.tsx         # Mobile detection
│   ├── use-toast.ts           # Toast notifications
│   └── useSessionSaver.ts     # Session persistence
├── integrations/
│   ├── lovable/               # Lovable Cloud OAuth
│   └── supabase/              # Database client & types
├── pages/
│   ├── Auth.tsx               # Sign in/up page
│   ├── Dashboard.tsx          # Analytics dashboard
│   ├── History.tsx            # Session history
│   ├── Index.tsx              # Main entry point
│   └── NotFound.tsx           # 404 page
└── lib/
    └── utils.ts               # Utility functions
```

---

## Technology Stack

### Frontend

| Technology | Version | Purpose |
|------------|---------|---------|
| React | 18.3.1 | UI framework |
| Vite | Latest | Build tool & dev server |
| TypeScript | Latest | Type safety |
| Tailwind CSS | Latest | Styling |
| shadcn/ui | Latest | Component library |
| React Router | 6.30.1 | Client-side routing |
| TanStack Query | 5.83.0 | Server state management |
| Recharts | 2.15.4 | Data visualization |
| Framer Motion | - | Animations (via shadcn) |
| date-fns | 3.6.0 | Date formatting |
| Sonner | 1.7.4 | Toast notifications |

### Backend (Lovable Cloud)

| Technology | Purpose |
|------------|---------|
| Supabase Auth | User authentication |
| Supabase PostgreSQL | Database |
| Edge Functions (Deno) | Serverless API |
| Lovable AI Gateway | AI model access |

### AI Models Used

| Model | Use Case |
|-------|----------|
| `google/gemini-3-flash-preview` | PDF processing & question generation |
| `google/gemini-3-flash-preview` | Answer analysis & feedback |

---

## Features

### 1. PDF Document Processing

**Flow:**
1. User uploads a PDF (max 10MB)
2. Edge function extracts text content
3. AI generates 4-6 diagnostic questions (mix of MCQ and short-response)
4. Questions are returned to the frontend

**Question Types:**
- **Short-Response**: Open-ended questions requiring explanation
- **Multiple-Choice**: 4 options with plausible distractors

### 2. Diagnostic Session

**Flow:**
1. User answers each question sequentially
2. AI analyzes each response against expected reasoning
3. Feedback is categorized as: `correct`, `partial`, or `misconception`
4. Session results are saved to database

**Feedback Components:**
- **Type**: Classification of understanding level
- **Misconception**: Explanation of flawed reasoning (if applicable)
- **Correct Explanation**: Full explanation of correct understanding
- **Memory Tip**: Mnemonic or analogy for retention

### 3. Progress Tracking

**History Page:**
- List of all completed sessions
- Per-session statistics
- Date/time of completion

**Dashboard:**
- Overall accuracy percentage with trend indicator
- 14-day accuracy chart
- Answer distribution pie chart
- Performance by document
- Common misconception patterns
- AI-generated insights (after 3+ sessions)

### 4. Authentication

- Email/password authentication
- Email verification required (no auto-confirm)
- OAuth support (Google) via Lovable Cloud
- Session persistence with auto-refresh

---

## Database Schema

### Tables

#### `diagnostic_sessions`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `user_id` | UUID | Reference to auth.users |
| `document_title` | TEXT | Name of uploaded document |
| `total_questions` | INTEGER | Number of questions in session |
| `correct_count` | INTEGER | Number of correct answers |
| `misconception_count` | INTEGER | Number of misconceptions |
| `partial_count` | INTEGER | Number of partial answers |
| `created_at` | TIMESTAMPTZ | Creation timestamp |
| `completed_at` | TIMESTAMPTZ | Completion timestamp |

#### `question_responses`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `session_id` | UUID | FK to diagnostic_sessions |
| `user_id` | UUID | Reference to auth.users |
| `question_text` | TEXT | The question asked |
| `question_type` | TEXT | "short-response" or "multiple-choice" |
| `student_answer` | TEXT | User's response |
| `feedback_type` | TEXT | "correct", "partial", or "misconception" |
| `misconception` | TEXT | Explanation of misconception (nullable) |
| `correct_explanation` | TEXT | Full correct explanation |
| `memory_tip` | TEXT | Retention aid (nullable) |
| `created_at` | TIMESTAMPTZ | Creation timestamp |

#### `profiles`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `user_id` | UUID | Reference to auth.users (unique) |
| `display_name` | TEXT | User's display name |
| `created_at` | TIMESTAMPTZ | Creation timestamp |
| `updated_at` | TIMESTAMPTZ | Last update timestamp |

### Row Level Security (RLS)

All tables have RLS enabled with user-scoped policies:

```sql
-- Users can only access their own data
CREATE POLICY "Users can view their own sessions" 
ON diagnostic_sessions FOR SELECT 
USING (auth.uid() = user_id);

CREATE POLICY "Users can create their own sessions" 
ON diagnostic_sessions FOR INSERT 
WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update their own sessions" 
ON diagnostic_sessions FOR UPDATE 
USING (auth.uid() = user_id);
```

### Database Functions

#### `handle_new_user()`

Trigger function that creates a profile when a new user signs up:

```sql
CREATE FUNCTION handle_new_user() RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO public.profiles (user_id, display_name)
  VALUES (NEW.id, COALESCE(NEW.raw_user_meta_data->>'display_name', 'User'));
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

---

## API Reference

### Edge Functions

#### `process-pdf`

**Endpoint:** `POST /functions/v1/process-pdf`

**Authentication:** Required (JWT)

**Request:**
```
Content-Type: multipart/form-data
Authorization: Bearer <jwt_token>

Body: FormData with "file" field (PDF, max 10MB)
```

**Response:**
```json
{
  "success": true,
  "documentTitle": "Chapter 5 - Photosynthesis",
  "questions": [
    {
      "id": "1",
      "type": "short-response",
      "question": "Explain why plants appear green...",
      "topic": "Light Absorption",
      "expectedReasoning": "Chlorophyll absorbs..."
    },
    {
      "id": "2",
      "type": "multiple-choice",
      "question": "Which molecule stores energy...",
      "topic": "Energy Storage",
      "choices": [
        { "id": "a", "text": "ATP" },
        { "id": "b", "text": "DNA" },
        { "id": "c", "text": "RNA" },
        { "id": "d", "text": "Protein" }
      ],
      "correctChoiceId": "a",
      "expectedReasoning": "ATP is the primary..."
    }
  ],
  "extractedLength": 12450
}
```

**Error Responses:**
| Status | Description |
|--------|-------------|
| 400 | Invalid file type or no file provided |
| 401 | Unauthorized (missing/invalid JWT) |
| 402 | AI credits exhausted |
| 429 | Rate limit exceeded |
| 500 | Server error |

---

#### `analyze-answer`

**Endpoint:** `POST /functions/v1/analyze-answer`

**Authentication:** Required (JWT)

**Request:**
```json
{
  "question": "Explain why plants appear green...",
  "expectedReasoning": "Chlorophyll absorbs red and blue...",
  "studentAnswer": "Because they have chlorophyll",
  "topic": "Light Absorption",
  "questionType": "short-response"
}
```

For multiple-choice:
```json
{
  "question": "Which molecule stores energy...",
  "expectedReasoning": "ATP is the primary...",
  "studentAnswer": "a",
  "topic": "Energy Storage",
  "questionType": "multiple-choice",
  "choices": [...],
  "correctChoiceId": "a"
}
```

**Response:**
```json
{
  "type": "partial",
  "misconception": "The student correctly identified chlorophyll but didn't explain how it affects light absorption and reflection.",
  "correctExplanation": "Plants appear green because chlorophyll molecules absorb red and blue wavelengths of light for photosynthesis, while reflecting green wavelengths. This reflected green light is what our eyes perceive.",
  "memoryTip": "Think of chlorophyll as being 'hungry' for red and blue light (like a picky eater), but 'full' of green light, so it throws green back at us!"
}
```

---

## Authentication

### Flow

1. **Sign Up**
   - User provides email, password, and optional display name
   - Confirmation email sent
   - User must verify email before access

2. **Sign In**
   - Email/password authentication
   - Session stored in localStorage
   - Auto-refresh enabled

3. **OAuth (Google)**
   - Handled via Lovable Cloud managed OAuth
   - No external configuration required

### Implementation

```typescript
// AuthContext provides:
interface AuthContextType {
  user: User | null;
  session: Session | null;
  isLoading: boolean;
  signUp: (email: string, password: string, displayName?: string) => Promise<{ error: Error | null }>;
  signIn: (email: string, password: string) => Promise<{ error: Error | null }>;
  signOut: () => Promise<void>;
}
```

### Protected Routes

Routes that require authentication:
- `/history` - Learning history
- `/dashboard` - Progress analytics
- PDF upload functionality on `/`

---

## Security

### Authentication Security

- JWT validation on all edge functions
- Token verification via `supabase.auth.getClaims()`
- No anonymous signups allowed
- Email verification required

### Edge Function Security

```typescript
// All edge functions validate JWT before processing
const authHeader = req.headers.get("Authorization");
if (!authHeader?.startsWith("Bearer ")) {
  return new Response(JSON.stringify({ error: "Unauthorized" }), { status: 401 });
}

const token = authHeader.replace("Bearer ", "");
const { data: claimsData, error } = await supabase.auth.getClaims(token);

if (error || !claimsData?.claims) {
  return new Response(JSON.stringify({ error: "Invalid session" }), { status: 401 });
}
```

### File Upload Security

- MIME type validation (`application/pdf` only)
- File size limit (10MB max)
- Filename sanitization
- Content length limits for AI processing

### Database Security

- Row Level Security (RLS) on all tables
- User-scoped policies for all operations
- Input sanitization in trigger functions
- No direct `auth.users` table access

### CORS Configuration

```typescript
const corsHeaders = {
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Headers": "authorization, x-client-info, apikey, content-type, x-supabase-client-platform, x-supabase-client-platform-version, x-supabase-client-runtime, x-supabase-client-runtime-version",
};
```

---

## User Interface

### Design System

- **Typography**: Custom display font for headings
- **Colors**: HSL-based semantic tokens
- **Theme**: Light/dark mode support via CSS variables
- **Animations**: Fade-in, slide-up, scale-in effects

### Page Structure

#### Public (Unauthenticated)
```
┌─────────────────────────────────────┐
│         PublicHeader                │
├─────────────────────────────────────┤
│         Hero Section                │
│   "Diagnose Your Misconceptions"    │
├─────────────────────────────────────┤
│      How It Works (Timeline)        │
│   Upload → Answer → Learn           │
├─────────────────────────────────────┤
│      Features (Bento Grid)          │
│   AI-Powered | Personalized | etc   │
├─────────────────────────────────────┤
│         Footer                      │
└─────────────────────────────────────┘
```

#### Authenticated (Logged In)
```
┌─────────────────────────────────────┐
│         AppHeader                   │
│   Logo | Dashboard | History | User │
├─────────────────────────────────────┤
│         AppHome                     │
│   Quick Stats (if available)        │
│   ┌─────────────────────────────┐   │
│   │     File Upload Zone        │   │
│   │   Drop PDF or Browse        │   │
│   └─────────────────────────────┘   │
│   Tips for best results             │
├─────────────────────────────────────┤
│         Footer                      │
└─────────────────────────────────────┘
```

### Responsive Breakpoints

- Mobile: < 640px
- Tablet: 640px - 1024px
- Desktop: > 1024px

---

## Deployment

### Production URLs

- **Preview:** https://id-preview--ed9b70cf-5e81-4e8f-ab7f-9d0ac5f14b56.lovable.app
- **Published:** https://misconceptionaidetector.lovable.app

### Deployment Process

1. **Frontend**: Click "Publish" in Lovable to deploy UI changes
2. **Edge Functions**: Auto-deployed on save
3. **Database Migrations**: Applied automatically after approval

### Build Configuration

```typescript
// vite.config.ts
export default defineConfig({
  server: {
    host: "::",
    port: 8080,
  },
  plugins: [react()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
});
```

---

## Environment Variables

### Frontend (Vite)

| Variable | Description |
|----------|-------------|
| `VITE_SUPABASE_URL` | Lovable Cloud API URL |
| `VITE_SUPABASE_PUBLISHABLE_KEY` | Public anon key |
| `VITE_SUPABASE_PROJECT_ID` | Project identifier |

### Edge Functions (Deno)

| Variable | Description |
|----------|-------------|
| `SUPABASE_URL` | Backend API URL |
| `SUPABASE_ANON_KEY` | Anon key for client |
| `SUPABASE_SERVICE_ROLE_KEY` | Admin key (when needed) |
| `LOVABLE_API_KEY` | AI gateway access |

---

## Appendix

### File Size Limits

| Type | Limit |
|------|-------|
| PDF Upload | 10 MB |
| Extracted Text | 15,000 characters |
| Display Name | 100 characters |

### Rate Limits

- AI requests subject to Lovable AI gateway limits
- HTTP 429 returned when exceeded
- HTTP 402 when credits exhausted

### Browser Support

- Chrome 90+
- Firefox 88+
- Safari 14+
- Edge 90+

---

## Legal

© 2026 Misconception AI Detector. All rights reserved.
