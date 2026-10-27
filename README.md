# Kavvi: case study

[**Kavvi**](https://kavvi.io) is an AI learning platform in Arabic and English. I designed it, wrote every service, run production alone, and handle billing and users. The source code is private; this page explains what it does and the engineering decisions behind it.

**Numbers since the September 2026 launch:** 458 sign-ups, 8+ paying subscribers, 8,300+ AI runs, 4,900+ generated concepts, about $0.09 of AI cost per learner. All users came from organic short videos; no paid ads.

## Two products

**Kavvi Subjects** turns a student's own lecture into a course on that lecture.

- A lecture PDF is read in full by an "architect" step that designs a learning path, then split into lessons and concepts (at most seven per lecture)
- Each concept has an explanation, diagrams and a live voice-over, while the PDF turns to the page being explained
- A chat beside each concept answers follow-up questions from the student's own material
- After each concept, a 3-question quiz. Wrong answers are explained again and retried, and every mistake goes into a mistake bank so the system knows each student's weak points
- Flashcards and exams are built from the student's own material, and a smart schedule turns lecture times and study availability into a plan with reminders

**Kavvi Courses** builds a personal course around any goal ("Python from zero", "German for my job"), one short story-style episode a day, in English or Egyptian Arabic. Learners answer questions, write real code, work in live spreadsheets and speak out loud, and every answer is checked.

## How it is built

```
React PWA ─┬─ Node/Express API ── MongoDB
           │        │
           │        └─ BullMQ workers (Valkey): lesson and episode generation, reminders
           └─ Python FastAPI service: PDF reading, RAG, lesson design, quizzes, answer checking
                    │
                    └─ LLMs routed per task through OpenRouter
```

Everything runs on one Linux VPS with Docker Compose behind Caddy, with a headless-Chromium PDF service, a sandboxed code and spreadsheet runner, S3-compatible storage, and ElevenLabs voice (Azure as fallback).

## Decisions worth explaining

**Answers are verified, not trusted.** Every question is solved twice independently and compared; when the two disagree, a stronger model re-checks. Validators also reject scratch-work leaking into questions.

**Failures are visible.** I removed silent fallbacks that hid bad output. An errors page in the admin console shows what failed.

**AI cost is measured per call and capped.** Each task uses a different model chosen by quality and price, with per-plan budgets and a daily spend cap, which keeps AI cost at about $0.09 per learner.

**Retrieval over a whole lecture, not chunks.** The system reads the entire lecture first and designs a path through it, instead of answering questions from the few chunks that match.

**Payments fit how Egyptian students pay.** InstaPay and wallet transfers with receipt upload and fraud flags (access starts at once and an admin reviews), plus card payments through Kashier. Regional prices, weekly, monthly and term plans, and a reverse trial.

**Operations.** Playwright tests, one-command rollbacks, an admin console for learners, spend, trials and plan locks, and a launch report.

## Contact

Ahmed Mansour: [ahmedismansour000@gmail.com](mailto:ahmedismansour000@gmail.com) · [Portfolio](https://ahmedmansour.vercel.app) · [LinkedIn](https://www.linkedin.com/in/ahmed-mansour-430ab2341/)

I'm happy to walk through the code on a call.
