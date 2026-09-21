# StackPass

Untimed practice app for software architecture, cloud, and engineering
topics (Python, Java/JVM, Spring, architecture styles, DDD, design
patterns, AWS, Azure, AI/agentic AI, data & persistence, security,
DevOps, quality attributes, testing strategy). React Native + Expo +
TypeScript, offline-first — no backend, no login, no cloud sync.

Bootstrapped from **CitizenPass** (a Canadian citizenship exam prep app)
by reusing its proven infrastructure — navigation shell, theming, i18n
mechanism, local persistence pattern — while rebuilding everything that
was specific to the citizenship-exam domain (question-type engine,
taxonomy depth, content model, exam rules). See `docs/` for the
reasoning behind every reuse/rebuild decision.

## Status: Feature 5 of N — content authoring (Design Patterns → Creational)

**Feature 1 (scaffold + domain model):**
- Project scaffold (Expo Router, TypeScript, ESLint, Jest) — same
  tooling as CitizenPass, renamed.
- Full 4-level taxonomy: **14 themes → 95 subcategories → 464 topics**,
  generated from the approved theme list (`src/data/manifests/taxonomy.json`).
- Question domain model covering the 7 approved question types as a
  discriminated union (`src/types/index.ts`) — SINGLE, MULTI, TRUE_FALSE,
  SCENARIO, TRADE_OFF, ANTI_PATTERN, ASSERTION_REASON. (CODE, ORDERING,
  and MATCHING were scoped originally but cut by decision — see
  `docs/design-decisions.md` — because they're the only three that
  needed their own bespoke answer-UI instead of sharing one of the 3
  list-based components the other 7 all use.)
- One fully-worked pilot topic (Design Patterns → Creational →
  Singleton) with one real question of each of the 7 types, as an
  authoring template for the rest of the bank.
- Reused infra, adapted: i18n (EN/FR), theme system (6 accent schemes ×
  light/dark), AsyncStorage-backed settings persistence, Zustand store
  pattern, navigation shell (4 tabs + a Study stack).
- Working screens: Home (stats + entry points), Study (browse all 14
  themes → subcategories → topics, read-only), Settings
  (language/theme/accent — fully functional).

**Feature 2: topic selection + Practice mode**
- Practice tab is a real tri-state theme/subcategory/topic picker
  (`app/(tabs)/practice.tsx`) — check any combination, only topics with
  real content are selectable, counts shown throughout.
- `useSelectionStore` — in-memory (not persisted) "what am I about to
  practice/exam" state, shared by both Practice and Exam.
- The 3 shared answer-input components: `ChoiceList` (single/multi-select),
  `TrueFalseToggle`, orchestrated by `QuestionCard`, which is the one
  place that switches on all 7 question types.
- `scoring.ts` — answer-state shapes + `isAnswerCorrect` grading (no
  partial credit on MULTI).
- `app/practice/session.tsx` — untimed, question-by-question practice
  with immediate feedback (correct/incorrect + explanation), a
  completion screen with score, "practice again" (reshuffled) and "back
  to selection."

**Feature 3: Exam mode**
- `app/exam/setup.tsx` — question-count stepper, clamped to what's
  actually available in the current selection.
- `examSampling.ts` — stratified sampling (even quota per selected
  topic, shortfall backfilled from the rest of the pool, seeded for
  deterministic replay). Takes an injectable question-fetcher so the
  stratification math itself is unit-tested with synthetic multi-topic
  fixtures, independent of how much real content exists yet.
- `useExamStore` — the sampled question set, per-question answers, and
  progress, shared across setup/session/results/review.
- `app/exam/session.tsx` — free navigation (Previous/Next or a
  jump-to-question strip), no per-question feedback until submission —
  the one real behavioral difference from Practice.
- `app/exam/results.tsx` — score, percentage, and a per-topic
  breakdown. No pass/fail threshold or verdict banner — no single
  official exam body to benchmark against here.
- `app/exam/review.tsx` — every question shown via the same
  `QuestionCard` in its full-feedback mode, with its topic labeled.
- Tests for the seeded PRNG and the stratification algorithm
  (`tests/utils/seededRandom.test.ts`,
  `tests/services/examSampling.test.ts`).

**Feature 4 (this delivery): Progress tracking**
- Completed Practice and Exam sessions are now persisted —
  `useProgressStore` + `progressRepository.ts` (AsyncStorage, capped at
  the most recent 500 sessions). This is new: earlier features
  deliberately skipped persistence entirely; this is the first piece of
  it, and it's scoped to *completed* sessions only (an in-progress
  session still isn't saved — see gaps below).
- `breakdown.ts` — `computeTopicBreakdown`, extracted out of what used
  to be inline logic in `results.tsx`, now the one shared implementation
  used by exam Results (display), the exam submit handler (what gets
  persisted), and Practice's own per-question tally.
- `progressAggregation.ts` — pure, unit-tested functions: `computeStreak`
  (consecutive-day streak that isn't broken just because today hasn't
  happened yet), `aggregateTopicMastery` (all-time per-topic accuracy,
  weakest-first), `computeOverallStats`.
- Progress tab is now real: overview stats, day streak, per-topic
  mastery list, recent session history, and a tap-twice "clear history."
- Tests for the aggregation math and the repository's persistence/cap
  behavior (`tests/utils/progressAggregation.test.ts`,
  `tests/services/progressRepository.test.ts`).

**Feature 5 (this delivery): content authoring — Design Patterns → Creational**
- All 5 Creational topics now have real questions: Singleton (7, the
  original all-7-types template), Factory Method, Abstract Factory,
  Builder, Prototype (5 each, mixing whichever question types fit that
  pattern best) — 27 questions total, up from 7.
- Written from direct domain knowledge (GoF patterns are stable,
  foundational content — no web search needed for this batch, unlike
  future batches covering time-sensitive material like current cloud
  service capabilities).
- Every answer index was mechanically re-verified against its intended
  correct option, not just checked for being in-range — see
  `docs/design-decisions.md` for what that caught and didn't.
- `tests/utils/selectionHelpers.test.ts` updated — it had hardcoded
  "only Singleton is available" assertions that this batch broke.

**Deliberately NOT built yet** (separate, later features):
- Content for the other 459 topics, and Design Patterns' other 5
  subcategories (Structural, Behavioral, Enterprise, Concurrency,
  Anti-patterns) — only Creational is done.
- In-progress Practice/Exam session persistence (closing the app
  mid-session still loses that attempt — see `docs/design-decisions.md`,
  which also notes the `Set`-in-`AnswerState` serialization issue to
  solve first).
- App icons/splash assets (`assets/icons/*.png` referenced in
  `app.json` don't exist yet — add before running `eas build`).

## What's reused vs. rebuilt from CitizenPass

See the conversation/design notes at the top of `src/types/index.ts`
and `src/types/content.ts` for the specific, file-level reasoning.
Short version: navigation/theme/i18n/persistence *mechanisms* transfer
almost unchanged; anything encoding citizenship-exam-specific rules or
content structure (exam timer, pass threshold, source-citation
governance, 2-level taxonomy, 2 question types) does not, and was
rebuilt for this domain instead.

## Getting started

```bash
npm install
npx expo start
```

Requires Node 18+. No environment variables or backend setup — the app
runs entirely offline.

## Scripts

- `npm run start` — Expo dev server
- `npm run lint` — ESLint
- `npm run typecheck` — `tsc --noEmit`
- `npm test` — Jest
