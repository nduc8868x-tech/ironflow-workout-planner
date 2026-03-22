# Technical Specification
## GymTrack Pro — Personal Gym Schedule Manager with AI Coaching Chatbot

**Version:** 1.0  
**Status:** Draft  
**Last Updated:** March 2026  
**Audience:** Engineering team, technical leads, QA

---

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Architecture](#2-architecture)
3. [Data Models](#3-data-models)
4. [Screen Inventory](#4-screen-inventory)
5. [User Flows](#5-user-flows)
6. [API Specification](#6-api-specification)
7. [AI Chatbot Specification](#7-ai-chatbot-specification)
8. [Feature Specifications](#8-feature-specifications)
9. [Infrastructure & Deployment](#9-infrastructure--deployment)
10. [Security Specification](#10-security-specification)
11. [Testing Strategy](#11-testing-strategy)
12. [Open Technical Decisions](#12-open-technical-decisions)

---

## 1. System Overview

### 1.1 Product Summary

GymTrack Pro is a responsive web application that allows users to:
1. Plan and manage personal gym training schedules
2. Log workout sessions with real-time set/rep/weight tracking
3. Visualize training and body composition progress over time
4. Consult an AI chatbot for contextual workout and nutrition guidance

### 1.2 Technology Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| **Frontend** | Next.js 14 (App Router) + TypeScript | SSR/SSG for performance, strong ecosystem |
| **Styling** | Tailwind CSS + shadcn/ui | Rapid UI development, consistent design system |
| **State Management** | Zustand + React Query | Client state + server state separation |
| **Backend** | Next.js API Routes (Node.js runtime) | Monorepo simplicity for MVP |
| **Database** | PostgreSQL 16 | ACID compliance, relational data model |
| **Cache / Session** | Redis 7 | Session storage, API response caching |
| **ORM** | Prisma | Type-safe DB access, migration management |
| **Authentication** | NextAuth.js v5 | OAuth + credentials, JWT sessions |
| **AI Chatbot** | Anthropic Claude API (claude-sonnet-4-6) | Large context window, strong reasoning |
| **File Storage** | Cloudflare R2 | S3-compatible, cost-efficient CDN delivery |
| **Email** | Resend | Transactional email (verification, notifications) |
| **Charts** | Recharts | React-native charting library |
| **Hosting** | Vercel (frontend + API) | Zero-config deployment, edge network |
| **DB Hosting** | Supabase (PostgreSQL) | Managed PostgreSQL + connection pooling |

### 1.3 Dependency Map

```
Browser ──HTTPS──► Next.js App (Vercel)
                        │
              ┌─────────┼──────────┐
              ▼         ▼          ▼
        PostgreSQL    Redis    Claude API
        (Supabase)  (Upstash)  (Anthropic)
              │
        Cloudflare R2
        (Media Assets)
```

---

## 2. Architecture

### 2.1 Application Layers

```
┌─────────────────────────────────────────────────┐
│                  Presentation Layer              │
│  Next.js Pages + Components + Tailwind CSS       │
├─────────────────────────────────────────────────┤
│                 Application Layer                │
│  Next.js API Routes  │  Server Actions           │
├─────────────────────────────────────────────────┤
│                  Domain Layer                    │
│  Business Logic  │  Validation (Zod)             │
├─────────────────────────────────────────────────┤
│              Data Access Layer                   │
│  Prisma ORM  │  Redis Client  │  Claude API SDK  │
├─────────────────────────────────────────────────┤
│               Infrastructure Layer               │
│  PostgreSQL  │  Redis  │  R2  │  Resend          │
└─────────────────────────────────────────────────┘
```

### 2.2 Routing Structure

```
/                          → Landing page (public)
/auth/register             → Registration
/auth/login                → Login
/auth/verify-email         → Email verification
/onboarding                → Multi-step onboarding wizard
/dashboard                 → Main dashboard (protected)
/schedule                  → Weekly/monthly calendar view
/schedule/new              → Create new workout plan
/schedule/[planId]         → Edit existing plan
/session/[sessionId]       → Active workout session
/library                   → Exercise library
/library/[exerciseId]      → Exercise detail
/progress                  → Progress charts and stats
/nutrition                 → Nutrition log
/chat                      → AI chatbot
/profile                   → User profile settings
/settings                  → App settings
/admin                     → Admin dashboard (role-gated)
```

### 2.3 Authentication Flow

```
User → /auth/login → NextAuth.js
         │
    ┌────┴─────┐
    ▼          ▼
 Credentials  Google OAuth
    │               │
    ▼               ▼
 bcrypt         OAuth token
 verify         validation
    │               │
    └──────┬─────────┘
           ▼
      JWT issued
   (24h access token)
   (30d refresh token)
           │
           ▼
    Protected routes
    via middleware
```

---

## 3. Data Models

### 3.1 Entity Relationship Overview

**Core entities:** `User`, `WorkoutPlan`, `PlanDay`, `PlanExercise`, `Exercise`, `WorkoutSession`, `SessionSet`  
**Supporting entities:** `WeightLog`, `ChatConversation`, `ChatMessage`, `NutritionLog`, `Achievement`

### 3.2 Full Schema (Prisma)

```prisma
// ──────────────── USER ────────────────
model User {
  id                String    @id @default(cuid())
  email             String    @unique
  emailVerified     DateTime?
  name              String?
  avatarUrl         String?
  passwordHash      String?   // null for OAuth users

  // Body profile
  dateOfBirth       DateTime?
  gender            Gender?
  heightCm          Float?
  weightKg          Float?
  fitnessGoal       FitnessGoal?
  fitnessLevel      FitnessLevel?
  weeklyFrequency   Int?      // days/week target
  sessionDuration   Int?      // target minutes
  availableEquipment Equipment[]

  // Meta
  onboardingDone    Boolean   @default(false)
  preferredUnit     Unit      @default(METRIC)
  language          String    @default("vi")
  role              Role      @default(USER)
  createdAt         DateTime  @default(now())
  updatedAt         DateTime  @updatedAt

  // Relations
  weightLogs        WeightLog[]
  workoutPlans      WorkoutPlan[]
  workoutSessions   WorkoutSession[]
  chatConversations ChatConversation[]
  nutritionLogs     NutritionLog[]
  achievements      UserAchievement[]
  accounts          Account[]         // NextAuth
  sessions          Session[]         // NextAuth
}

// ──────────────── WORKOUT PLAN ────────────────
model WorkoutPlan {
  id            String    @id @default(cuid())
  userId        String
  user          User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  name          String
  description   String?
  templateType  String?   // e.g., "PPL", "FULL_BODY", "CUSTOM"
  daysPerWeek   Int
  isActive      Boolean   @default(true)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  planDays      PlanDay[]
}

model PlanDay {
  id            String    @id @default(cuid())
  planId        String
  plan          WorkoutPlan @relation(fields: [planId], references: [id], onDelete: Cascade)
  dayOfWeek     Int       // 0=Sunday … 6=Saturday
  name          String    // e.g., "Push Day", "Leg Day"
  order         Int       @default(0)

  planExercises PlanExercise[]
  sessions      WorkoutSession[]
}

model PlanExercise {
  id            String    @id @default(cuid())
  planDayId     String
  planDay       PlanDay   @relation(fields: [planDayId], references: [id], onDelete: Cascade)
  exerciseId    String
  exercise      Exercise  @relation(fields: [exerciseId], references: [id])
  targetSets    Int       @default(3)
  targetReps    Int       @default(10)
  restSeconds   Int       @default(90)
  notes         String?
  order         Int       @default(0)
}

// ──────────────── EXERCISE LIBRARY ────────────────
model Exercise {
  id              String    @id @default(cuid())
  name            String    @unique
  slug            String    @unique
  description     String?
  instructions    String[]  // step-by-step
  muscleGroups    MuscleGroup[]
  equipment       Equipment[]
  difficulty      Difficulty
  imageUrl        String?
  gifUrl          String?
  videoUrl        String?
  isPublic        Boolean   @default(true)
  createdAt       DateTime  @default(now())

  planExercises   PlanExercise[]
  sessionSets     SessionSet[]
}

// ──────────────── SESSION LOGGING ────────────────
model WorkoutSession {
  id              String    @id @default(cuid())
  userId          String
  user            User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  planDayId       String?
  planDay         PlanDay?  @relation(fields: [planDayId], references: [id])
  date            DateTime
  startedAt       DateTime?
  completedAt     DateTime?
  durationSeconds Int?
  totalVolume     Float?    // kg·reps
  notes           String?
  status          SessionStatus @default(IN_PROGRESS)

  sets            SessionSet[]
}

model SessionSet {
  id              String    @id @default(cuid())
  sessionId       String
  session         WorkoutSession @relation(fields: [sessionId], references: [id], onDelete: Cascade)
  exerciseId      String
  exercise        Exercise  @relation(fields: [exerciseId], references: [id])
  setNumber       Int
  weightKg        Float?
  reps            Int?
  durationSeconds Int?      // for timed exercises
  isSkipped       Boolean   @default(false)
  completedAt     DateTime?
}

// ──────────────── WEIGHT LOG ────────────────
model WeightLog {
  id          String    @id @default(cuid())
  userId      String
  user        User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  weightKg    Float
  loggedAt    DateTime  @default(now())
  notes       String?
}

// ──────────────── CHATBOT ────────────────
model ChatConversation {
  id          String    @id @default(cuid())
  userId      String
  user        User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  title       String?
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt

  messages    ChatMessage[]
}

model ChatMessage {
  id              String    @id @default(cuid())
  conversationId  String
  conversation    ChatConversation @relation(fields: [conversationId], references: [id], onDelete: Cascade)
  role            MessageRole  // USER | ASSISTANT
  content         String
  tokensUsed      Int?
  createdAt       DateTime  @default(now())
}

// ──────────────── NUTRITION ────────────────
model NutritionLog {
  id          String    @id @default(cuid())
  userId      String
  user        User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  date        DateTime
  mealType    MealType
  foodName    String
  calories    Float
  proteinG    Float     @default(0)
  carbsG      Float     @default(0)
  fatG        Float     @default(0)
  createdAt   DateTime  @default(now())
}

// ──────────────── ACHIEVEMENTS ────────────────
model Achievement {
  id          String    @id @default(cuid())
  key         String    @unique   // e.g., "STREAK_7", "FIRST_SESSION"
  name        String
  description String
  iconUrl     String?

  userAchievements UserAchievement[]
}

model UserAchievement {
  userId        String
  achievementId String
  user          User        @relation(fields: [userId], references: [id], onDelete: Cascade)
  achievement   Achievement @relation(fields: [achievementId], references: [id])
  earnedAt      DateTime    @default(now())

  @@id([userId, achievementId])
}

// ──────────────── ENUMS ────────────────
enum Role           { USER ADMIN }
enum Gender         { MALE FEMALE OTHER PREFER_NOT_TO_SAY }
enum FitnessGoal    { LOSE_WEIGHT BUILD_MUSCLE IMPROVE_ENDURANCE STAY_HEALTHY ATHLETIC_PERFORMANCE }
enum FitnessLevel   { BEGINNER INTERMEDIATE ADVANCED }
enum Equipment      { BARBELL DUMBBELL CABLE MACHINE BODYWEIGHT RESISTANCE_BAND KETTLEBELL PULL_UP_BAR }
enum MuscleGroup    { CHEST BACK SHOULDERS BICEPS TRICEPS FOREARMS QUADS HAMSTRINGS GLUTES CALVES ABS FULL_BODY CARDIO }
enum Difficulty     { BEGINNER INTERMEDIATE ADVANCED }
enum Unit           { METRIC IMPERIAL }
enum SessionStatus  { IN_PROGRESS COMPLETED SKIPPED }
enum MessageRole    { USER ASSISTANT }
enum MealType       { BREAKFAST LUNCH DINNER SNACK PRE_WORKOUT POST_WORKOUT }
```

---

## 4. Screen Inventory

### 4.1 Public Screens (Unauthenticated)

| Screen ID | Route | Name | Key Components |
|-----------|-------|------|----------------|
| SCR-01 | `/` | Landing Page | Hero section, feature highlights, pricing teaser, CTA buttons |
| SCR-02 | `/auth/register` | Registration | Email/password form, Google OAuth button, T&C checkbox |
| SCR-03 | `/auth/login` | Login | Email/password form, Google OAuth button, "Forgot password" link |
| SCR-04 | `/auth/forgot-password` | Forgot Password | Email input, submit confirmation |

### 4.2 Onboarding (First-login only)

| Screen ID | Route | Name | Content |
|-----------|-------|------|---------|
| SCR-05 | `/onboarding` | Onboarding Wizard | **Step 1:** Goal selection (5 options with icons) |
| | | | **Step 2:** Body stats (height, weight, DOB, gender) |
| | | | **Step 3:** Fitness level (Beginner / Intermediate / Advanced) |
| | | | **Step 4:** Schedule preference (days/week, minutes/session) |
| | | | **Step 5:** Equipment availability (multi-select checkboxes) |

### 4.3 Core Application Screens (Authenticated)

| Screen ID | Route | Name | Key Components |
|-----------|-------|------|----------------|
| SCR-10 | `/dashboard` | Dashboard | Streak card, weekly progress ring, weight card, next session card, recent activity feed |
| SCR-11 | `/schedule` | Schedule Calendar | Weekly calendar view, session status indicators (planned/done/missed), quick-add button |
| SCR-12 | `/schedule/new` | Create Plan | Plan name input, days/week selector, template picker, day-by-day exercise builder |
| SCR-13 | `/schedule/[planId]` | Edit Plan | Same as Create, pre-populated with existing data |
| SCR-14 | `/session/[sessionId]` | Active Session | Exercise list with set logger, rest timer, progress bar, finish button |
| SCR-15 | `/session/[sessionId]/summary` | Session Summary | Total volume, duration, PR indicators, streak badge, share button |
| SCR-16 | `/library` | Exercise Library | Search bar, muscle group filter, equipment filter, exercise card grid |
| SCR-17 | `/library/[exerciseId]` | Exercise Detail | GIF/image, instructions, muscle diagram, personal history chart |
| SCR-18 | `/progress` | Progress | Tabs: Body / Strength / Activity. Charts: weight trend, volume trend, streak calendar |
| SCR-19 | `/nutrition` | Nutrition Log | Date picker, meal sections, macro progress bars, add food modal |
| SCR-20 | `/chat` | AI Chatbot | Conversation interface, quick prompts panel, conversation history sidebar |
| SCR-21 | `/profile` | Profile Settings | Edit personal info, update goal, change password, data export, delete account |
| SCR-22 | `/settings` | App Settings | Unit preference, language, notification toggles |

### 4.4 Admin Screens

| Screen ID | Route | Name | Key Components |
|-----------|-------|------|----------------|
| SCR-30 | `/admin` | Admin Dashboard | User count, DAU/WAU, session count, system health indicators |
| SCR-31 | `/admin/users` | User Management | User table with search, status filter, ban/unban action |

---

## 5. User Flows

### Flow 1 — Registration & Onboarding

```
[Landing Page]
     │
     ▼ click "Get Started Free"
[Registration Screen]
     │
  ┌──┴──┐
  │     │
Email  Google OAuth
  │     │
  ▼     ▼
[Email Verification]  ──────── (OAuth skips this)
     │
     ▼
[Onboarding Step 1: Goal]
     │
     ▼
[Onboarding Step 2: Body Stats]
     │
     ▼
[Onboarding Step 3: Fitness Level]
     │
     ▼
[Onboarding Step 4: Schedule Preference]
     │
     ▼
[Onboarding Step 5: Equipment]
     │
     ▼
 System generates suggested plan template
     │
     ▼
[Dashboard] ← first visit, with onboarding tips
```

### Flow 2 — Create Workout Plan

```
[Dashboard] or [Schedule]
     │
     ▼ click "Create Plan"
[Create Plan Screen]
     │
  ┌──┴──┐
  │     │
Custom  Template
plan    selection
  │     │
  └──┬──┘
     │
     ▼
 Name plan → Select training days
     │
     ▼
 For each day:
   Search/browse exercise library
   → Add exercises
   → Set target sets, reps, rest time
   → Drag to reorder
     │
     ▼
 Save Plan → [Schedule Calendar]
 (plan appears on selected days)
```

### Flow 3 — Execute Workout Session

```
[Schedule Calendar] — sees today's planned session
     │
     ▼ click "Start Session"
[Active Session Screen]
     │
     ▼
 Exercise 1 displayed
     │
     ▼ for each set:
   Enter weight + reps → "Done Set" button
     │
     ▼
 Rest timer counts down
     │
     ▼ after all sets:
 "Next Exercise" → repeat until last exercise
     │
     ▼
 "Finish Workout" button
     │
     ▼
[Session Summary Screen]
  Shows: total volume, duration, streaks, PRs
     │
     ▼
 Return to [Dashboard] or [Schedule]
```

### Flow 4 — Chatbot Consultation

```
[Any screen] → click chatbot FAB or nav item
     │
     ▼
[Chat Screen]
  Chatbot greeting + 4 quick prompt suggestions
     │
  ┌──┴──┐
  │     │
Click  Type
prompt  freely
  │     │
  └──┬──┘
     │
     ▼
 System builds context:
   { user profile + current plan + conversation history }
     │
     ▼
 Claude API call (streaming response)
     │
     ▼
 Response streams into chat
     │
     ▼ if response includes exercise suggestions:
 Deep link card: "Add to Plan →" button
     │
     ▼
 User continues conversation
```

### Flow 5 — Track Progress

```
[Dashboard] → click "Progress"
     │
     ▼
[Progress Screen — Body tab]
     │
  ┌──┴──────────────────────┐
  │                         │
  ▼                         ▼
Log new weight          Select time range
(input + save)          (4w / 3m / All)
  │                         │
  └──────────┬──────────────┘
             │
             ▼
        Weight trend chart updates
             │
             ▼ switch to Strength tab
        Select exercise → view 1RM progression chart
             │
             ▼ switch to Activity tab
        Streak calendar + weekly volume bars + badges
```

---

## 6. API Specification

### 6.1 Conventions

- **Base URL:** `/api/v1`
- **Authentication:** Bearer JWT in `Authorization` header
- **Format:** JSON request/response bodies
- **Errors:** `{ "error": { "code": "ERROR_CODE", "message": "Human-readable message" } }`
- **Pagination:** `?page=1&limit=20` query params; response includes `{ data, total, page, limit }`

### 6.2 Authentication Endpoints

```
POST   /api/auth/register         Register with email + password
POST   /api/auth/login            Login (returns JWT + refresh token)
POST   /api/auth/logout           Invalidate session
POST   /api/auth/refresh          Exchange refresh token for new access token
POST   /api/auth/verify-email     Verify email with token from email link
POST   /api/auth/forgot-password  Send password reset email
POST   /api/auth/reset-password   Reset password with token
```

### 6.3 User & Profile Endpoints

```
GET    /api/users/me              Get current user profile
PATCH  /api/users/me              Update profile fields
DELETE /api/users/me              Delete account (queues data deletion)
GET    /api/users/me/export       Export all user data as JSON
POST   /api/users/me/weight       Log a new weight entry
GET    /api/users/me/weight       Get weight history (supports ?from&to)
```

### 6.4 Workout Plan Endpoints

```
GET    /api/plans                 List user's workout plans
POST   /api/plans                 Create new plan
GET    /api/plans/:planId         Get plan with days and exercises
PATCH  /api/plans/:planId         Update plan metadata
DELETE /api/plans/:planId         Delete plan
POST   /api/plans/:planId/clone   Duplicate plan

GET    /api/plans/:planId/days            List days in plan
POST   /api/plans/:planId/days            Add day to plan
PATCH  /api/plans/:planId/days/:dayId     Update day
DELETE /api/plans/:planId/days/:dayId     Remove day

POST   /api/plans/:planId/days/:dayId/exercises          Add exercise to day
PATCH  /api/plans/:planId/days/:dayId/exercises/:peId    Update exercise config
DELETE /api/plans/:planId/days/:dayId/exercises/:peId    Remove exercise from day
POST   /api/plans/:planId/days/:dayId/exercises/reorder  Reorder exercises
```

### 6.5 Exercise Library Endpoints

```
GET    /api/exercises                   List exercises (search + filter)
GET    /api/exercises/:exerciseId       Get exercise detail
  Query params: ?q=&muscleGroup=&equipment=&difficulty=&page=&limit=
```

### 6.6 Session Endpoints

```
POST   /api/sessions                    Create/start session from plan day
GET    /api/sessions                    List user's sessions (paginated)
GET    /api/sessions/:sessionId         Get session with all sets
PATCH  /api/sessions/:sessionId         Update session (notes, status)
DELETE /api/sessions/:sessionId         Delete session

POST   /api/sessions/:sessionId/sets             Log a set
PATCH  /api/sessions/:sessionId/sets/:setId      Update a set
DELETE /api/sessions/:sessionId/sets/:setId      Delete a set

POST   /api/sessions/:sessionId/complete         Mark session complete (triggers stats recalculation)
```

### 6.7 Progress Endpoints

```
GET    /api/progress/summary            Dashboard summary stats (streak, weekly sessions, etc.)
GET    /api/progress/weight             Weight trend data (supports ?range=4w|3m|all)
GET    /api/progress/volume             Weekly volume trend
GET    /api/progress/strength           Strength progression per exercise
  Query params: ?exerciseId=&range=
GET    /api/progress/achievements       User's earned achievements
```

### 6.8 Chat Endpoints

```
GET    /api/chat/conversations                          List conversations
POST   /api/chat/conversations                          Start new conversation
GET    /api/chat/conversations/:id                      Get conversation with messages
DELETE /api/chat/conversations/:id                      Delete conversation

POST   /api/chat/conversations/:id/messages             Send message (streaming response)
  → Server-Sent Events (SSE) or chunked JSON stream
  → Response: `Content-Type: text/event-stream`
  → Events: `data: {"type":"delta","content":"..."}` then `data: {"type":"done","tokensUsed":N}`
```

### 6.9 Nutrition Endpoints

```
GET    /api/nutrition                   List nutrition logs (by date)
POST   /api/nutrition                   Log a meal entry
PATCH  /api/nutrition/:logId            Update meal entry
DELETE /api/nutrition/:logId            Delete meal entry
GET    /api/nutrition/summary           Daily macro summary for a date (?date=)
```

---

## 7. AI Chatbot Specification

### 7.1 Model Configuration

```json
{
  "model": "claude-sonnet-4-6",
  "max_tokens": 1024,
  "stream": true,
  "system": "<system_prompt>"
}
```

### 7.2 System Prompt Template

The system prompt is dynamically composed per request using the user's current data:

```
You are a knowledgeable and supportive fitness coach for GymTrack Pro.

## User Profile
- Name: {name}
- Goal: {fitnessGoal}
- Fitness level: {fitnessLevel}
- Age: {age} | Gender: {gender}
- Height: {heightCm}cm | Current weight: {latestWeightKg}kg
- Available equipment: {equipment}

## Current Workout Plan
{planSummary}
(includes plan name, training days, and exercise list per day)

## Recent Activity
- Sessions this week: {sessionsThisWeek}
- Current streak: {streakDays} days
- Last session: {lastSessionDate} — {lastSessionName}

## Instructions
- Provide evidence-based, practical advice tailored to the user's goal, level, and equipment.
- When suggesting exercises, prefer those matching the user's available equipment.
- If the user asks about injury or pain, always advise consulting a healthcare professional first.
- For nutrition questions, use established formulas (Mifflin-St Jeor for TDEE).
- Keep responses concise and actionable. Use bullet points for exercise lists.
- When relevant, suggest next steps the user can take within the app (e.g., "You can add this exercise to your plan").
- Never diagnose medical conditions. Always include a disclaimer for health-related advice.
- IMPORTANT: You are not a doctor. Do not provide medical diagnoses.
```

### 7.3 Context Window Management

| Component | Approximate Tokens |
|-----------|-------------------|
| System prompt | ~400 |
| User profile + plan | ~300–600 |
| Conversation history | Last 10 messages ≈ 1,000–2,000 |
| Current user message | ~50–200 |
| **Total input budget** | **~2,000–3,200** |
| **Response (max_tokens)** | **1,024** |

Strategy: Keep last 10 messages in context. For longer conversations, summarize older messages rather than truncating.

### 7.4 Streaming Implementation

```typescript
// Server: POST /api/chat/conversations/:id/messages
export async function POST(req: Request, { params }) {
  const { content } = await req.json();
  const user = await getAuthUser(req);
  const context = await buildUserContext(user.id); // profile + plan + recent activity

  const messages = await buildMessageHistory(params.id);
  messages.push({ role: "user", content });

  const stream = await anthropic.messages.stream({
    model: "claude-sonnet-4-6",
    max_tokens: 1024,
    system: buildSystemPrompt(context),
    messages,
  });

  // Save assistant message to DB after stream completes
  stream.on("message", async (message) => {
    await saveMessage(params.id, "assistant", message.content[0].text);
  });

  return new Response(stream.toReadableStream(), {
    headers: { "Content-Type": "text/event-stream" },
  });
}
```

### 7.5 Quick Prompt Suggestions

Default suggestions shown at the start of a conversation, dynamically selected based on user goal:

| User Goal | Quick Prompts |
|-----------|--------------|
| Lose weight | "What should I eat before a workout?", "How many calories should I target?", "Can you adjust my plan for fat loss?", "What cardio do you recommend?" |
| Build muscle | "Am I eating enough protein?", "How should I progressively overload?", "What exercises can replace cable rows?", "Review my current split" |
| Improve endurance | "How do I structure cardio and strength training?", "What's a good warm-up routine?", "How do I prevent overtraining?", "Recommend a 5K training addition" |
| General | "What muscles does my plan focus on?", "Suggest a recovery day routine", "Explain progressive overload", "What should I eat post-workout?" |

---

## 8. Feature Specifications

### 8.1 Onboarding Wizard

**Component:** `/onboarding` — 5-step modal wizard with progress bar

**State:** Persisted in localStorage during onboarding; committed to database on final step submission.

**Steps:**

| Step | Fields | Validation |
|------|--------|-----------|
| 1 — Goal | `fitnessGoal` (required, single select) | Must select one |
| 2 — Body Stats | `heightCm`, `weightKg`, `dateOfBirth`, `gender` (all optional but encouraged) | Height: 50–300cm; Weight: 20–300kg |
| 3 — Fitness Level | `fitnessLevel` (required) | Must select one |
| 4 — Schedule | `weeklyFrequency` (1–7), `sessionDuration` (15–180 min) | Integer range validation |
| 5 — Equipment | `availableEquipment` (multi-select) | At least one selection required |

**Post-onboarding:** System auto-selects the most appropriate workout template and creates a default plan. User is redirected to the dashboard with the plan already visible.

### 8.2 Active Session Screen

**Real-time session logging — key UX requirements:**

- Screen must not sleep during an active session (use `WakeLock API`)
- Previous session's weight/rep values auto-filled as placeholder for each set
- Sets display as a vertical list: uncompleted → gray, completed → green checkmark
- Rest timer starts automatically when a set is marked done; can be adjusted ±15s
- "Previous" and "Next" exercise navigation available at all times
- Session auto-saves every 30 seconds to prevent data loss

**Set logging data structure per set:**
```typescript
{
  setNumber: number;
  weightKg: number | null;
  reps: number | null;
  durationSeconds: number | null; // for timed exercises
  isSkipped: boolean;
  completedAt: Date | null;
}
```

**1RM Estimation formula (Epley):**
```
1RM = weight × (1 + reps / 30)
```
Only calculated when reps ≤ 10 (higher reps produce unreliable estimates).

### 8.3 Progress Charts

| Chart | Type | X-axis | Y-axis | Time Ranges |
|-------|------|--------|--------|-------------|
| Weight trend | Line chart | Date | kg | 4w, 3m, All |
| Weekly volume | Bar chart | Week (Mon–Sun) | kg·reps (total) | 8 weeks fixed |
| Strength (1RM) | Line chart | Date | Estimated 1RM (kg) | 3m, All |
| Session frequency | Heatmap calendar | Day of month | Color = session done/missed | Current month |

**All charts use Recharts.** Responsive containers, tooltips on hover/tap, animated on first render.

### 8.4 Streak System

- A streak increments when the user completes at least 1 session on a given calendar day
- A streak breaks if no session is logged for 2 consecutive calendar days (allows 1 rest day)
- Streak is stored as `{ currentStreak: number, longestStreak: number, lastSessionDate: Date }` on the `User` model
- Streak is recalculated server-side on session completion

### 8.5 TDEE Calculation

Using the **Mifflin-St Jeor equation:**

```
BMR (male)   = 10 × weight(kg) + 6.25 × height(cm) − 5 × age + 5
BMR (female) = 10 × weight(kg) + 6.25 × height(cm) − 5 × age − 161

Activity multipliers:
  Sedentary (0–1 session/week):    BMR × 1.2
  Lightly active (2–3 sessions):   BMR × 1.375
  Moderately active (4–5 sessions): BMR × 1.55
  Very active (6–7 sessions):      BMR × 1.725

Calorie targets by goal:
  Lose weight:    TDEE − 300 to −500 kcal
  Build muscle:   TDEE + 200 to +400 kcal
  Maintenance:    TDEE
```

Calculation runs on the server and is stored per user, recalculated when weight or activity changes.

---

## 9. Infrastructure & Deployment

### 9.1 Environments

| Environment | Purpose | URL |
|-------------|---------|-----|
| Development | Local dev | `localhost:3000` |
| Staging | QA and UAT | `staging.gymtrackpro.app` |
| Production | Live users | `gymtrackpro.app` |

### 9.2 Environment Variables

```bash
# Database
DATABASE_URL=postgresql://...
REDIS_URL=redis://...

# Authentication
NEXTAUTH_SECRET=...
NEXTAUTH_URL=...
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...

# AI
ANTHROPIC_API_KEY=...

# Storage
R2_ACCOUNT_ID=...
R2_ACCESS_KEY_ID=...
R2_SECRET_ACCESS_KEY=...
R2_BUCKET_NAME=...
R2_PUBLIC_URL=...

# Email
RESEND_API_KEY=...
EMAIL_FROM=noreply@gymtrackpro.app
```

### 9.3 CI/CD Pipeline

```
Push to branch
    │
    ▼
GitHub Actions: lint + typecheck + unit tests
    │
    ▼ (pass)
Build Docker image
    │
    ▼ push to main
Vercel: automatic preview deployment
    │
    ▼ PR merged to main
Vercel: production deployment
    │
    ▼
Run DB migrations (prisma migrate deploy)
    │
    ▼
Smoke tests on production URL
```

### 9.4 Performance Targets & Caching Strategy

| Resource | Cache Strategy | TTL |
|----------|---------------|-----|
| Exercise library | Redis cache | 24 hours |
| User profile | React Query cache | 5 minutes |
| Progress chart data | Redis cache (per user) | 15 minutes |
| Static assets (JS/CSS) | CDN (Vercel Edge) | Immutable |
| Exercise GIFs/images | Cloudflare R2 + CDN | 30 days |

---

## 10. Security Specification

### 10.1 Authentication & Authorization

- All protected routes validated in Next.js middleware before page render
- API routes validate JWT on every request; no exceptions
- Role check (`USER` vs `ADMIN`) enforced on every admin endpoint
- Users can only access their own data — all DB queries include `WHERE userId = currentUser.id`

### 10.2 Input Validation

All request bodies validated with **Zod** schemas before processing. Example:

```typescript
const weightLogSchema = z.object({
  weightKg: z.number().min(20).max(500),
  loggedAt: z.string().datetime().optional(),
  notes: z.string().max(500).optional(),
});
```

### 10.3 Rate Limiting

| Endpoint Group | Limit | Window |
|----------------|-------|--------|
| Auth (login/register) | 10 requests | 15 minutes / IP |
| Chat messages | 30 requests | 1 minute / user |
| General API | 200 requests | 1 minute / user |

Implemented via Upstash Redis + `@upstash/ratelimit`.

### 10.4 Data Protection

- Passwords: bcrypt, cost factor 12
- JWTs: signed with HS256, 24h access / 30d refresh
- Database: all connections via TLS; row-level security enabled on Supabase
- PII fields (name, email, DOB) treated as sensitive; not logged in application logs
- Chat messages encrypted at rest in PostgreSQL

---

## 11. Testing Strategy

### 11.1 Test Pyramid

```
         ┌───────────┐
         │  E2E (10%)│  Playwright — critical user journeys
         ├───────────┤
         │Integration│  Jest + Supertest — API routes + DB
         │   (30%)   │
         ├───────────┤
         │  Unit     │  Jest + React Testing Library
         │  (60%)    │  — utility functions, components, hooks
         └───────────┘
```

### 11.2 Critical E2E Scenarios

1. User registers → completes onboarding → views dashboard
2. User creates a workout plan → starts session → logs all sets → finishes session → views summary
3. User sends message to chatbot → receives context-aware response
4. User logs weight for 7 days → views weight chart with correct trend
5. Admin logs in → views user dashboard → cannot access individual user's workout data

### 11.3 Performance Testing

- Load test with 1,000 concurrent users using k6
- Target: P95 API response < 500ms, zero 5xx errors under normal load
- Chatbot endpoint load tested separately: 50 concurrent streaming sessions

---

## 12. Open Technical Decisions

| ID | Decision | Options | Recommendation | Status |
|----|----------|---------|---------------|--------|
| TD-01 | Exercise database source | Build in-house vs. use ExerciseDB API vs. Wger open-source data | Use Wger open-source data (seeded into own DB for full control) | **Open** |
| TD-02 | Chat history context length | Last 10 messages / last 5 / summarize after 10 | Last 10 messages + summarization after 20 | **Open** |
| TD-03 | Notification delivery | Browser Push API (PWA) vs. email vs. both | Both — email for reminders, Push for active session alerts | **Open** |
| TD-04 | Unit support | Metric only vs. metric + imperial | Both (stored as metric, displayed per user preference) | **Decided: both** |
| TD-05 | Plan sharing | Private only vs. shareable link | Private in v1.0, shareable link in v1.2 | **Decided: private first** |
| TD-06 | Chatbot memory across sessions | Stateless (context from DB only) vs. persistent memory | Stateless — load last 10 messages from DB on each conversation open | **Decided: stateless** |
| TD-07 | 1RM formula | Epley vs. Brzycki vs. Multiple + average | Epley (most commonly used, simplest to explain) | **Decided: Epley** |

---

*Document Owner: Engineering Lead | Paired with: Requirements.md v1.0 | Next Review: Before Sprint 1 kickoff*
