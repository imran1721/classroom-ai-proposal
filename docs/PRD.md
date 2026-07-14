# PRD — AI Classroom Learning Platform

**Status:** Draft v0.1 • **Owner:** Imran • **Date:** 2026-07-14

---

## 1. Summary

A multi-tenant SaaS that converts recorded classroom lessons into readable
study content and an AI tutor grounded in that content, while capturing student
doubts as analytics for teachers and school admins.

## 2. Goals & non-goals

**Goals**
- Every recorded lesson becomes durable, readable, searchable content.
- Students get instant, lesson-grounded answers to their doubts.
- Teachers see per-lesson/chapter doubt patterns.
- Admins see school-wide topics that are commonly not understood.
- Onboard and bill multiple schools with isolated data.

**Non-goals (v1)**
- ERP (attendance, grades, fees, timetabling, admissions).
- Native mobile apps, live streaming, hardware, raw-video playback for students.

## 3. Personas & roles

| Role | Can do |
|---|---|
| **Student** | View subjects/chapters/lessons, read content, ask doubts, see own doubt history |
| **Teacher** | Upload/record lessons, organize chapters, publish content, view doubt survey for own classes |
| **School Admin** | Manage teachers/students/classes, view school-wide analytics |
| **Super Admin (platform)** | Onboard schools, manage plans/billing, ops |

Roles are per-school (a user belongs to one school/tenant).

## 4. Core concepts / data model (sketch)

```
School (tenant)
 └─ Users (student | teacher | admin), each scoped to the school
 └─ Class / Section
 └─ Subject
     └─ Chapter
         └─ Lesson
             ├─ MediaAsset (audio/video source)
             ├─ Transcript
             ├─ GeneratedContent (readable notes, sections)
             ├─ ContentChunk[] + embedding (for RAG)
             └─ Doubt[]  (student, question, answer, topic_tag, timestamp)
Subscription (per School: plan, seats, status)
```

Row-level security isolates all data by `school_id`.

## 5. Feature requirements

### 5.1 Onboarding & tenancy
- Super admin creates a school; school admin invites teachers/students (bulk
  CSV import).
- All data strictly isolated per school (RLS enforced).
- Roles and permissions per the table above.

### 5.2 Lesson ingestion
- Teacher creates Subject → Chapter → Lesson structure.
- Upload audio or video, or attach an existing recording, to a lesson.
- Lesson shows processing status: `uploaded → transcribing → generating →
  ready → (published)`.

### 5.3 Processing pipeline (async)
- **Transcribe** media (Hindi/English, auto-detect) → transcript with
  timestamps.
- **Generate content**: LLM turns transcript into structured, readable study
  notes (headings, key points, summary) in the lesson's language.
- **Chunk + embed** content into pgvector for retrieval.
- **Reliability**: job queue with retries, idempotency, per-lesson status and
  error surfacing to the teacher.
- **Publish**: *[Open Decision #1]* teacher reviews & approves before students
  see it (default), or auto-publish.

### 5.4 Student experience
- Browse own school's subjects → chapters → lessons.
- Read generated content for a published lesson.
- **Doubt chat**: ask free-text questions; AI answers grounded in that lesson's
  content (RAG), streamed, with the ability to say "not covered in this
  lesson." Optional audio playback of the source segment.
- View own doubt history.
- **Cost control**: cache/reuse answers to repeated common questions per lesson.

### 5.5 Teacher dashboard
- Per lesson and per chapter: list and count of doubts, grouped by topic.
- Which students asked what (doubt survey).
- Flag lessons/topics with unusually high doubt volume.

### 5.6 Admin analytics
- School-wide "commonly misunderstood topics" (doubts clustered/tagged by the
  LLM to topics, then aggregated).
- Rollups by class, subject, chapter, time period.
- Trends over time; export.

### 5.7 SaaS platform
- Plans, seats/usage limits, subscriptions (Razorpay), invoices.
- Email notifications (processing done, invites, digests).
- Super-admin ops console.

## 6. Non-functional requirements
- **Multi-tenant security**: no cross-school data leakage (RLS + tests).
- **Privacy**: student data handling, retention policy, deletion on request.
- **Scale target (v1)**: ~10–20 schools, ~15–30k students, ~2k lessons/month.
- **Availability**: standard managed-cloud SLAs; async pipeline tolerant to ASR/LLM outages (retries).
- **Mobile**: responsive PWA, installable; native app path preserved.
- **i18n**: English + Hindi content and UI.

## 7. Analytics detail — "commonly not understood"
- Each doubt is tagged to a topic by the LLM at capture time (cheap model).
- Topics roll up to chapter/subject.
- "Commonly not understood" = topics ranked by (distinct students asking) ×
  (doubt density), per class and per school.
- v1 uses LLM tagging + simple aggregation; deeper clustering is a later phase.

## 8. Cost model (running, at scale scenario 2k lessons/mo, 15k students)
| Item | ~Monthly |
|---|---|
| Transcription | ₹50k |
| Content generation | ₹20k |
| Doubt answering (RAG) | ₹30k |
| Storage (audio + content) | ₹2k |
| Platform infra | ₹20k |
| **Total** | **~₹1.1–1.5 L** (≈ ₹8–10/student) |

## 9. Milestones (maps to proposal phases)
0. Foundation → 1. Ingestion+Pipeline → 2. Student+Doubt-chat →
3. Teacher dashboard → 4. Admin analytics → 5. Billing/notifications/hardening
→ 6. Pilot.

## 10. Open decisions
1. Teacher review before publish? (default: yes)
2. Students rewatch raw video? (default: no — audio + content only)
3. Doubt volume caps per plan? (recommended, to bound running cost)
