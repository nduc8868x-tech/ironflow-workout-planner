# Requirements Document
## GymTrack Pro — Personal Gym Schedule Manager with AI Coaching Chatbot

**Version:** 1.0  
**Status:** Draft  
**Last Updated:** March 2026

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Business Requirements](#2-business-requirements)
3. [Stakeholder Requirements](#3-stakeholder-requirements)
4. [User Requirements](#4-user-requirements)
5. [System Requirements](#5-system-requirements)
6. [Functional Requirements](#6-functional-requirements)
7. [Non-Functional Requirements](#7-non-functional-requirements)
8. [Constraints & Assumptions](#8-constraints--assumptions)
9. [Acceptance Criteria Summary](#9-acceptance-criteria-summary)

---

## 1. Introduction

### 1.1 Purpose

This document defines the requirements for **GymTrack Pro**, a web-based application that enables individuals to plan, log, and track their personal gym training schedules. The system includes an AI-powered chatbot that provides personalized advice on workouts and nutrition.

### 1.2 Scope

GymTrack Pro covers:
- User account management and onboarding
- Workout plan creation and management
- Live workout session logging
- Progress tracking and analytics
- AI chatbot for workout and nutrition consultation
- Basic nutrition logging and macro tracking

Out of scope (v1.0):
- Mobile native application (iOS/Android)
- Personal trainer management portal
- Social sharing and community features
- Integration with wearable devices (Apple Watch, Garmin, etc.)
- E-commerce or supplement purchasing

### 1.3 Definitions

| Term | Definition |
|------|------------|
| **Workout Plan** | A structured weekly template assigning exercise sessions to specific days |
| **Workout Session** | A single logged training instance with recorded sets, reps, and weights |
| **Set / Rep** | Set = one group of repetitions; Rep = a single execution of a movement |
| **Volume** | Total training load calculated as Sets × Reps × Weight |
| **1RM** | One-Rep Max — the maximum weight lifted for one complete repetition |
| **Streak** | Consecutive days of completed workout sessions |
| **TDEE** | Total Daily Energy Expenditure — estimated daily calorie burn |
| **Macro** | Macronutrients: Protein, Carbohydrates, and Fat |
| **Onboarding** | Initial profile setup flow for new users |
| **Template** | A pre-built workout program (e.g., PPL, Full Body, Upper/Lower) |

### 1.4 Document Conventions

- **[MUST]** — Mandatory requirement; the system cannot go live without it
- **[SHOULD]** — High-priority requirement; omission requires sign-off
- **[MAY]** — Optional enhancement; included if time permits
- **REQ-XXX** — Unique requirement identifier
- **US-XXX** — User story identifier

---

## 2. Business Requirements

### 2.1 Business Objectives

| ID | Objective | Success Metric |
|----|-----------|----------------|
| BO-01 | Enable individuals to independently manage gym training without a personal trainer | ≥70% of users create and complete at least one workout plan within 7 days of registration |
| BO-02 | Retain users through measurable progress visibility | ≥50% of registered users log at least 2 sessions per week after the first month |
| BO-03 | Deliver personalized AI guidance competitive with basic PT consultations | Chatbot satisfaction score ≥ 4.0/5.0 in user surveys |
| BO-04 | Establish a scalable SaaS product foundation | System handles 10,000 concurrent users without degradation |

### 2.2 Business Rules

- **BR-01:** All personal health data (weight, body measurements, nutrition) must remain private and visible only to the authenticated user.
- **BR-02:** The AI chatbot must not provide medical diagnoses or claim to replace licensed healthcare professionals. All advice must include appropriate disclaimers.
- **BR-03:** Users must be able to permanently delete their account and all associated data at any time.
- **BR-04:** The system must not share or sell user data to third parties.
- **BR-05:** Workout templates offered to users must be based on established training methodologies (not randomly generated).

---

## 3. Stakeholder Requirements

### 3.1 Stakeholder Map

| Stakeholder | Role | Primary Interest |
|-------------|------|-----------------|
| End User | Primary consumer of the product | Achieve fitness goals, easy to use |
| Product Owner | Defines product vision | User retention, feature roadmap |
| Development Team | Builds and maintains the system | Clear requirements, technical feasibility |
| Admin / Ops Team | Monitors and manages the platform | System health, user management |
| AI Provider (Anthropic) | Supplies the LLM for the chatbot | Compliant usage per API terms |

### 3.2 Stakeholder Needs

**End Users need:**
- A frictionless way to plan and log workouts without manual spreadsheets
- Reliable progress data that motivates continued training
- Instant, contextual answers to fitness questions without hiring a trainer
- Confidence their health data is private and secure

**Product Owner needs:**
- A system that can be launched as an MVP within 2 months
- Feature extensibility without major architectural changes
- Analytics on feature usage to inform future development

**Admin Team needs:**
- Dashboard for monitoring active users and platform health
- Ability to manage user accounts (ban, support) without accessing personal data

---

## 4. User Requirements

### 4.1 User Personas

#### Persona A — "The Beginner" (Primary)
- Age: 20–30, no prior gym experience
- Motivation: Start training, unsure where to begin
- Pain points: Doesn't know what exercises to do, fears injury, overwhelmed by information
- **Needs:** Guided onboarding, pre-built plans, chatbot for basic questions, simple UI

#### Persona B — "The Intermediate Self-Trainer" (Primary)
- Age: 25–40, training 6–18 months independently
- Motivation: Optimize progress, track strength gains
- Pain points: Progress plateaued, nutrition unclear, lacks structure
- **Needs:** Custom plan builder, strength tracking (1RM), macro guidance, chatbot for advanced questions

#### Persona C — "The Busy Professional" (Secondary)
- Age: 28–45, high schedule variability
- Motivation: Stay consistent despite time constraints
- Pain points: Misses sessions, hard to reschedule, no time to research
- **Needs:** Flexible scheduling, reminder notifications, quick session logging, short workout templates

### 4.2 User Stories

#### Authentication & Profile

| ID | User Story | Priority |
|----|-----------|---------|
| US-001 | As a new visitor, I want to sign up using my email or Google account so that I can access the platform quickly. | MUST |
| US-002 | As a registered user, I want to complete an onboarding questionnaire so that the system can suggest an appropriate starting plan. | MUST |
| US-003 | As a user, I want to update my body measurements (weight, height, goal) so that my profile reflects my current state. | MUST |
| US-004 | As a user, I want to delete my account and all my data permanently so that I have full control over my privacy. | MUST |

#### Workout Planning

| ID | User Story | Priority |
|----|-----------|---------|
| US-010 | As a user, I want to create a weekly workout plan by selecting days and exercises so that I have a structured training schedule. | MUST |
| US-011 | As a user, I want to browse a library of exercises filtered by muscle group and equipment so that I can find relevant exercises for my plan. | MUST |
| US-012 | As a user, I want to choose from pre-built program templates (PPL, Full Body, etc.) so that I don't have to build a plan from scratch. | MUST |
| US-013 | As a user, I want to set target sets, reps, and rest times for each exercise so that the session guides me through my planned workload. | MUST |
| US-014 | As a user, I want to duplicate my current plan to the next week so that I can easily repeat my training cycle. | MAY |

#### Workout Logging

| ID | User Story | Priority |
|----|-----------|---------|
| US-020 | As a user, I want to start an active workout session from my plan so that I can log sets in real time. | MUST |
| US-021 | As a user, I want to record the weight and reps for each set during a session so that my performance is tracked accurately. | MUST |
| US-022 | As a user, I want an automatic rest timer between sets so that I don't have to watch the clock. | SHOULD |
| US-023 | As a user, I want to see a session summary (total volume, duration, streak) when I finish a session so that I feel a sense of accomplishment. | MUST |
| US-024 | As a user, I want to mark individual sets as skipped so that I can log incomplete sessions without corrupting my data. | SHOULD |

#### Progress Tracking

| ID | User Story | Priority |
|----|-----------|---------|
| US-030 | As a user, I want to view a dashboard showing my weekly training overview and current streak so that I can see my consistency at a glance. | MUST |
| US-031 | As a user, I want to log my daily weight and view a trend chart so that I can see whether I'm on track toward my goal. | MUST |
| US-032 | As a user, I want to track my estimated 1RM for key lifts over time so that I can measure strength progression. | SHOULD |
| US-033 | As a user, I want to filter my progress charts by time range (4 weeks, 3 months, all time) so that I can analyze different periods. | SHOULD |
| US-034 | As a user, I want to earn streaks and achievement badges so that I feel motivated to stay consistent. | SHOULD |

#### AI Chatbot

| ID | User Story | Priority |
|----|-----------|---------|
| US-040 | As a user, I want to ask the chatbot about exercise selection, technique, and alternatives so that I can make informed decisions about my training. | MUST |
| US-041 | As a user, I want the chatbot to know my current plan and profile so that its answers are relevant to my specific situation. | MUST |
| US-042 | As a user, I want to ask the chatbot about nutrition, calorie targets, and macros so that I can support my training with proper diet. | MUST |
| US-043 | As a user, I want the chatbot to suggest exercise alternatives when I mention an injury or lack of equipment so that I can adapt my training. | MUST |
| US-044 | As a user, I want to see quick prompt suggestions at the start of a chat so that I don't have to think of what to ask. | SHOULD |
| US-045 | As a user, I want to review my past conversations with the chatbot so that I can reference previous advice. | SHOULD |

#### Nutrition

| ID | User Story | Priority |
|----|-----------|---------|
| US-050 | As a user, I want the system to calculate my TDEE based on my profile so that I have a baseline calorie target. | SHOULD |
| US-051 | As a user, I want to log my daily meals with calorie and macro information so that I can monitor my nutritional intake. | SHOULD |
| US-052 | As a user, I want to see daily progress bars for protein, carbs, and fat so that I know how close I am to my macro targets. | SHOULD |

---

## 5. System Requirements

### 5.1 System Context

GymTrack Pro is a web application deployed on cloud infrastructure. It integrates with:
- **Authentication provider** (Google OAuth 2.0)
- **Anthropic Claude API** (AI chatbot)
- **Cloud object storage** (exercise media: images, GIFs)
- **Email service** (account verification, notifications)

### 5.2 System-Level Requirements

| ID | Requirement | Priority |
|----|-------------|---------|
| SYS-01 | The system shall be accessible via modern web browsers (Chrome, Firefox, Safari, Edge — latest 2 major versions) | MUST |
| SYS-02 | The system shall be fully responsive and usable on screens ≥ 375px wide | MUST |
| SYS-03 | The system shall support concurrent access by at least 10,000 active users without performance degradation | MUST |
| SYS-04 | The system shall maintain 99.5% uptime measured monthly | MUST |
| SYS-05 | All data transmission shall be encrypted via HTTPS/TLS 1.2+ | MUST |
| SYS-06 | User passwords shall be hashed using bcrypt with a minimum cost factor of 12 | MUST |
| SYS-07 | The system shall perform daily automated database backups with 30-day retention | MUST |
| SYS-08 | The system shall support Vietnamese (default) and English as UI languages | SHOULD |

---

## 6. Functional Requirements

### 6.1 Authentication & Account Management

| ID | Requirement | Priority |
|----|-------------|---------|
| REQ-F-001 | The system shall allow users to register with email and password | MUST |
| REQ-F-002 | The system shall support Google OAuth 2.0 registration and login | MUST |
| REQ-F-003 | The system shall send an email verification link upon email registration | MUST |
| REQ-F-004 | The system shall provide a "Forgot Password" flow via email reset link | MUST |
| REQ-F-005 | The system shall present a 5-step onboarding wizard to new users upon first login | MUST |
| REQ-F-006 | The system shall allow users to update their profile information at any time | MUST |
| REQ-F-007 | The system shall allow users to permanently delete their account and all associated data | MUST |

### 6.2 Workout Plan Management

| ID | Requirement | Priority |
|----|-------------|---------|
| REQ-F-010 | The system shall allow users to create workout plans with a defined number of training days per week | MUST |
| REQ-F-011 | The system shall provide an exercise library containing at least 100 exercises with muscle group tags, equipment tags, and instructional images/GIFs | MUST |
| REQ-F-012 | The system shall allow users to search and filter exercises by name, muscle group, and equipment type | MUST |
| REQ-F-013 | The system shall allow users to add exercises to a plan day and configure target sets, target reps, and rest duration | MUST |
| REQ-F-014 | The system shall provide at least 5 pre-built workout program templates (e.g., PPL, Full Body 3×/week, Upper/Lower, Beginner 3×/week, Cardio + Strength) | MUST |
| REQ-F-015 | The system shall allow users to reorder exercises within a plan day via drag-and-drop or up/down controls | SHOULD |
| REQ-F-016 | The system shall allow users to duplicate an existing workout plan | MAY |

### 6.3 Workout Session Logging

| ID | Requirement | Priority |
|----|-------------|---------|
| REQ-F-020 | The system shall allow users to start a workout session from a scheduled plan day | MUST |
| REQ-F-021 | The system shall display exercises in sequence during an active session | MUST |
| REQ-F-022 | The system shall allow users to log weight (kg or lb) and reps for each set | MUST |
| REQ-F-023 | The system shall auto-populate the previous session's values as defaults for the current session | SHOULD |
| REQ-F-024 | The system shall provide a countdown rest timer between sets | SHOULD |
| REQ-F-025 | The system shall allow users to mark a set as skipped | SHOULD |
| REQ-F-026 | The system shall save a session summary upon completion including: total volume, total duration, exercise count, and date | MUST |
| REQ-F-027 | The system shall allow users to add free-text notes to a completed session | MAY |

### 6.4 Progress Tracking & Analytics

| ID | Requirement | Priority |
|----|-------------|---------|
| REQ-F-030 | The system shall display a dashboard with: current streak, sessions this week, latest logged weight, and nearest upcoming session | MUST |
| REQ-F-031 | The system shall allow users to log a daily body weight entry | MUST |
| REQ-F-032 | The system shall display a line chart of body weight over time, filterable by 4 weeks / 3 months / all time | MUST |
| REQ-F-033 | The system shall display a bar chart of weekly training volume over the past 8 weeks | MUST |
| REQ-F-034 | The system shall track and display a training streak counter | SHOULD |
| REQ-F-035 | The system shall calculate and display estimated 1RM for user-designated key lifts | SHOULD |
| REQ-F-036 | The system shall award achievement badges for milestones (first session, 7-day streak, 30-day streak, etc.) | SHOULD |
| REQ-F-037 | The system shall allow users to view the complete history of past workout sessions | MUST |

### 6.5 AI Chatbot

| ID | Requirement | Priority |
|----|-------------|---------|
| REQ-F-040 | The system shall provide a conversational AI chatbot powered by the Claude API | MUST |
| REQ-F-041 | The chatbot shall have access to the user's profile (goal, fitness level, equipment) and current workout plan to personalize responses | MUST |
| REQ-F-042 | The chatbot shall be capable of answering questions about: exercise selection, technique, muscle groups, programming concepts | MUST |
| REQ-F-043 | The chatbot shall be capable of calculating TDEE and providing macro split guidance based on the user's goal | MUST |
| REQ-F-044 | The chatbot shall suggest alternative exercises when the user mentions injury, pain, or missing equipment | MUST |
| REQ-F-045 | The chatbot shall display 3–5 suggested quick-start prompts at the beginning of a new conversation | SHOULD |
| REQ-F-046 | The system shall persist chat conversation history for at least 90 days | SHOULD |
| REQ-F-047 | The chatbot response time shall not exceed 5 seconds under normal load | MUST |
| REQ-F-048 | The chatbot shall include a disclaimer that it does not provide medical advice | MUST |

### 6.6 Nutrition Tracking

| ID | Requirement | Priority |
|----|-------------|---------|
| REQ-F-050 | The system shall calculate the user's estimated TDEE using the Mifflin-St Jeor formula | SHOULD |
| REQ-F-051 | The system shall allow users to set daily calorie and macro targets | SHOULD |
| REQ-F-052 | The system shall allow users to log meals with food name, calorie count, and macros (protein, carbs, fat) | SHOULD |
| REQ-F-053 | The system shall display daily macro progress bars for protein, carbs, and fat | SHOULD |

---

## 7. Non-Functional Requirements

### 7.1 Performance

| ID | Requirement |
|----|-------------|
| REQ-NF-001 | Page Largest Contentful Paint (LCP) shall be ≤ 2.5 seconds on a standard broadband connection |
| REQ-NF-002 | API responses for data reads shall return within 500ms at the 95th percentile |
| REQ-NF-003 | Chatbot first-token response shall begin streaming within 3 seconds |
| REQ-NF-004 | The system shall support at least 10,000 concurrent users without response time degradation beyond 20% |

### 7.2 Security

| ID | Requirement |
|----|-------------|
| REQ-NF-010 | All HTTP traffic shall be redirected to HTTPS |
| REQ-NF-011 | User passwords shall be stored as bcrypt hashes (cost factor ≥ 12) |
| REQ-NF-012 | JSON Web Tokens (JWT) shall expire after 24 hours; refresh tokens after 30 days |
| REQ-NF-013 | The system shall implement rate limiting on authentication endpoints (max 10 failed attempts per 15 minutes per IP) |
| REQ-NF-014 | The system shall protect against OWASP Top 10 vulnerabilities (SQL injection, XSS, CSRF, etc.) |
| REQ-NF-015 | User health data at rest shall be stored in an encrypted database |

### 7.3 Usability

| ID | Requirement |
|----|-------------|
| REQ-NF-020 | A new user with no gym experience shall be able to complete onboarding and start their first session within 10 minutes |
| REQ-NF-021 | The UI shall be usable without instructions for the core workout logging flow |
| REQ-NF-022 | All interactive elements shall have a minimum tap target size of 44×44px (WCAG 2.1 AA) |
| REQ-NF-023 | Color alone shall not be used to convey critical information |

### 7.4 Reliability & Availability

| ID | Requirement |
|----|-------------|
| REQ-NF-030 | The system shall maintain 99.5% uptime per calendar month (excluding planned maintenance) |
| REQ-NF-031 | Planned maintenance windows shall be scheduled outside peak hours (11 PM – 5 AM local time) |
| REQ-NF-032 | The system shall perform automated database backups daily; backups shall be retained for 30 days |
| REQ-NF-033 | Recovery Time Objective (RTO) shall be ≤ 2 hours; Recovery Point Objective (RPO) shall be ≤ 24 hours |

### 7.5 Privacy & Compliance

| ID | Requirement |
|----|-------------|
| REQ-NF-040 | The system shall comply with applicable data protection principles (GDPR-aligned where applicable) |
| REQ-NF-041 | Users shall be able to export their data in a machine-readable format (JSON or CSV) |
| REQ-NF-042 | Users shall be able to delete all their personal data within 30 days of requesting account deletion |
| REQ-NF-043 | The privacy policy and terms of service shall be accessible from all pages |

---

## 8. Constraints & Assumptions

### 8.1 Technical Constraints

- The chatbot is powered exclusively by the Anthropic Claude API; usage must comply with Anthropic's usage policies
- The web application must function as a Single Page Application (SPA) or Server-Side Rendered (SSR) app — no native mobile app in v1.0
- The exercise media library (GIFs/images) must be served via CDN to meet performance requirements
- Third-party authentication is limited to Google OAuth 2.0 for v1.0

### 8.2 Business Constraints

- MVP must be deliverable within 2 months with a team of 2–3 developers
- Infrastructure costs must remain under budget; serverless/managed services preferred
- No dependency on paid third-party fitness databases; exercise data must be owned or freely licensed

### 8.3 Assumptions

- Users have a stable internet connection (≥ 5 Mbps) during active workout sessions
- Users access the system primarily on mobile phones during gym sessions and on desktop for planning
- The system does not need to integrate with gym equipment or IoT devices in v1.0
- Users are responsible for the accuracy of the health data they input
- The AI chatbot does not retain memory between separate browser sessions unless conversation history is explicitly loaded

---

## 9. Acceptance Criteria Summary

| Requirement Group | Key Acceptance Criteria |
|-------------------|------------------------|
| Authentication | User can register, verify email, log in, and complete onboarding in under 10 minutes |
| Workout Planning | User can create a 3-day/week plan with custom exercises in under 5 minutes |
| Session Logging | User can log a full 5-exercise session with sets/reps/weight and receive a summary |
| Progress Tracking | Weight log chart renders correctly for all three time ranges; streak counter increments after each session |
| AI Chatbot | Chatbot responds to workout and nutrition questions using user's actual profile data within 5 seconds |
| Performance | LCP ≤ 2.5s; API response P95 ≤ 500ms under simulated 1,000 concurrent user load test |
| Security | Penetration test returns no Critical or High severity findings before go-live |

---

*Document Owner: Product Team | Next Review: Before Sprint 1 kickoff*
