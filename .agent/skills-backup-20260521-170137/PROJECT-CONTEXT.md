# Project Context: Resume Builder / EdTech App

## Projects This Covers
- GlobalResumeAI — AI-powered resume builder with job tailoring
- CAT Prep App — Mock tests, timer, analytics for CAT/MBA entrance exams
- Avantika English Coach — Conversational AI English learning platform

## Common Stack
- Framework: Next.js 15 (App Router)
- Language: TypeScript
- ORM: Prisma + PostgreSQL
- Auth: NextAuth.js
- AI: Anthropic Claude claude-sonnet-4-6 (streaming)
- Storage: Vercel Blob or AWS S3 (resumes, PDFs)
- Deploy: Vercel

## GlobalResumeAI Specifics
- Core entities: User, Resume, ResumeSection, AISuggestion, ExportJob
- Key feature: AI tailoring — Claude rewrites bullets to match job description
- PDF export: puppeteer or @react-pdf/renderer
- Version history: keep last 10 versions per resume
- IDOR risk: always check resume.userId === session.user.id

## CAT Prep App Specifics
- Core entities: User, MockTest, Question, Answer, TestAttempt, QuestionAttempt
- Timer: must persist in DB every 30s to survive browser refresh
- Score calculation: server-side only (never trust client score)
- AI feature: explain wrong answers in batch after test completes
- Analytics: subject-wise accuracy, time-per-question, improvement trend

## Avantika English Coach Specifics
- Core entities: User, Lesson, Exercise, ConversationSession, Progress
- AI: structured JSON responses (not freeform text) for grammar correction UI
- Streaming: typing effect for Claude responses (feels conversational)
- Progress: track vocabulary, grammar, fluency scores over time
- Content: exercises and lessons are seeded data (not user-generated)

## Agent Priorities for These Apps
1. UX is critical — these are user-facing consumer apps, not internal tools
2. Mobile-first design — many users on phones
3. AI response quality > speed — users expect accurate, helpful output
4. Data privacy — resumes and learning data are sensitive, handle carefully
5. Offline resilience — cache test state locally (CAT app) so lost connection doesn't lose progress
