# Proposal — AI Classroom Learning Platform

**Prepared by:** Imran
**Date:** 2026-07-14
**Version:** 0.1 (draft for discussion)

---

## 1. The problem

Classroom teaching is ephemeral. A lesson is taught once; students who don't
follow it in the moment have no good way to revisit it, and the doubts they
*don't* ask out loud are invisible to everyone. Teachers can't see what
confused the class. Principals can't see what's confusing across classes.

## 2. What we're building

A multi-school SaaS platform that turns each recorded lesson into durable,
searchable learning, and turns student confusion into data.

**The loop:**

1. A lesson is captured live while the teacher is teaching (audio or video,
   English/Hindi) and tagged to a chapter → lesson.
2. An automated pipeline transcribes it and generates clean, readable study
   content for that lesson.
3. Students log in, read the lesson, and ask their doubts. An AI tutor answers
   using *that lesson's* content.
4. Every doubt is captured. Teachers get a doubt survey per lesson/chapter.
   Principals get school-wide analytics on what topics are commonly not
   understood.

## 3. Who it's for

- **Students** — read lessons, ask unlimited doubts, get instant answers.
- **Teachers** — see exactly which students struggled with which topics.
- **School admins (principal / headmaster)** — see class- and school-level
  patterns of misunderstanding.
- **Platform owner (you/us)** — onboard and bill multiple schools.

## 4. Scope

### In scope (v1)
- Multi-tenant SaaS: multiple schools, isolated data, per-school users & roles.
- Lesson ingestion: live in-class capture (audio/video) + chapter/lesson organization.
- Processing pipeline: transcription (Hi/En) → generated readable content →
  search embeddings.
- Student app: lesson reader + AI doubt-chat (answers grounded in the lesson).
- Teacher dashboard: doubt aggregation and survey per lesson/chapter.
- Admin analytics: "commonly misunderstood topics" and higher-level rollups.
- Billing & subscriptions, email notifications, super-admin/ops.
- Web app, responsive + installable as a PWA (works on phones).

### Explicitly out of scope (v1)
- Traditional ERP modules: attendance, grades, fees, timetabling, admissions.
- Native iOS/Android apps (the web app is mobile-friendly; native is a
  separate later project).
- Live/real-time classroom streaming.
- Hardware supply or installation (schools provide their own mic/camera).
- Raw video playback for students *(see Open Decision #2)*.

## 5. Technical approach (why it's efficient for one developer)

| Concern | Choice | Why |
|---|---|---|
| Web app | Next.js (PWA-ready) | Web now, installable on phones, native later if needed |
| Backend + DB + Auth + Storage | Supabase (Postgres, RLS, pgvector) | One service covers multi-tenancy, auth, storage, and vector search — saves ~6 weeks of plumbing |
| Transcription | Off-the-shelf ASR (Sarvam/Deepgram/Whisper) | Strong Hindi + English, no model training |
| Content + doubt answering | Claude/GPT via API | No model hosting; grounded (RAG) answers |
| Media storage | Cloudflare R2 (audio + content) | No egress fees; store audio, not heavy video |

**Key cost-control decision:** we store audio + generated content, not raw
video. This cuts storage/bandwidth by 20–50x with no loss to the student
experience.

## 6. Timeline

| Phase | Deliverable | Weeks |
|---|---|---|
| 0 | Foundation: multi-tenant, auth, roles, onboarding | 2–3 |
| 1 | Ingestion + processing pipeline | 5–6 |
| 2 | Student app + AI doubt-chat | 3–4 |
| 3 | Teacher dashboard & doubt survey | 2–3 |
| 4 | Admin analytics | 2–3 |
| 5 | Billing, notifications, security hardening, tests | 3–5 |
| 6 | Pilot with one school + buffer | 2–3 |

- **Pilot-ready (phases 0–2): ~3 months**
- **Launchable v1 (all phases): ~6–7 months** full-time solo.

## 7. Commercials — partnership model (no build fee)

There is **no upfront build fee**. Development is contributed in exchange for
an ongoing share of revenue — i.e. the developer takes the risk alongside the
business and shares the upside instead of billing for the build.

### Revenue share
- **20–30% of gross revenue**, paid quarterly, with the right to inspect books.
- Based on **gross revenue** (not "profit"), so it can't be diluted away by
  loading costs against it.
- **Downside protection:** if gross revenue is below [threshold] by month 12,
  the share converts to an agreed cash amount owed for the work done.

### Running costs (not part of the share)
Cloud/AI usage scales with lessons and doubt volume. At ~10 schools /
~15,000 students / ~2,000 lessons per month, expect **~₹1.1–1.5 L/month**
(≈ ₹8–10 per student/month). Covered by the business, itemized — not absorbed
into the developer's share.

### Maintenance & new features (paid separately)
Ongoing hosting oversight, fixes, and support: **₹75,000 – ₹1,25,000 / month**
retainer. New features beyond v1 quoted separately or at **₹1,500–2,500/hr**.
This is kept separate so running the product doesn't eat into the revenue share.

### IP
Given no build fee is charged, the developer retains **co-ownership of the IP**
(or a license-back), not a full assignment.

## 8. Open decisions (need your call before we finalize)

1. **Teacher review before publish?** Default in this proposal: teacher
   approves generated content before students see it (adds ~1–1.5 wk, included
   in the range). Alternative: auto-publish.
2. **Do students rewatch raw video?** Default: no (audio + content only). If
   yes, storage/bandwidth cost rises significantly and we add a video player.

## 9. Assumptions
- Schools provide their own recording (mic/camera) and internet.
- English + Hindi only for v1.
- Off-the-shelf ASR + LLM APIs are acceptable (no on-prem/self-hosted models).
- IP ownership and data-privacy terms to be set in the contract.
