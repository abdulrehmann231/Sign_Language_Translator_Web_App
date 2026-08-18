## Context

The app today is a Flask + Socket.IO server (`app.py`) that streams webcam/YouTube frames, runs a YOLOv8 BISINDO model (`bisindo.pt`) plus MediaPipe Hands for landmark overlay, and emits detected labels to the browser over a socket. There is no notion of a user, no persistence, and no "produce a sign" path — only "read a sign." This change layers a learning product on top of that pipeline. See `proposal.md` → Why for motivation. Guiding constraint: **reuse existing CV tech, train nothing new.**

## Goals / Non-Goals

**Goals:**
- Reuse the existing detection stack (YOLOv8, MediaPipe, `bisindo.pt`) and add off-the-shelf recognizers for object detection and the fingerspelling alphabet.
- Keep the fingerspelling alphabet (ASL vs BISINDO) **configurable** so flows are identical regardless of the chosen system.
- Introduce lightweight per-student persistence (deck + mastery) without a full auth system.
- Make `fingerspelling-recognition` a single shared capability consumed by Features 1, 2, and 3.

**Non-Goals:**
- Training or fine-tuning any model.
- Full account system (email/password, social login). A simple student identifier is enough for v1.
- Shipping Feature 4 beyond a reviewed spec.
- Continuous (non-fingerspelling) sign production grading — v1 grades the manual alphabet only.

## Decisions

**1. Fingerspelling recognition via existing tech (MediaPipe Hands + a pre-built alphabet classifier).**
Rationale: The manual alphabet is a well-solved, ~26-class static-hand-pose problem. MediaPipe Hands already ships in the repo for landmark extraction; landmarks can feed an existing open fingerspelling/ASL-alphabet classifier or a small rule/KNN matcher over reference landmarks — no training run required. Alternative considered: a full sign-language-production model (rejected — needs training and continuous-motion data).

**2. Object detection reuses YOLOv8, with an image-classification fallback for broad coverage.**
Rationale: `yolov8n.pt` (COCO, 80 classes) is already present and covers many everyday objects for Feature 2. For objects outside those classes, fall back to an off-the-shelf image classifier / vision API. Alternative: retrain YOLO on a custom object set (rejected — training).

**3. Indonesian↔English mapping and picture assets come from a static dictionary + asset pack.**
Rationale: Deterministic, offline-friendly, and lets us control the four-option distractors and per-word pictures for Feature 1. Detected class labels (English) map through this dictionary to Indonesian and to picture/fingerspelling assets. Alternative: live translation API (viable but adds a network dependency and non-determinism for distractor generation).

**4. Deck + mastery persisted server-side keyed by a student id.**
Rationale: Enables cross-session decks and quiz eligibility. Start with a simple store (SQLite or a JSON/file store per student id) to avoid infra weight; the `learner-deck` spec is storage-agnostic. Alternative: browser localStorage (rejected — not portable across devices, harder to quiz server-side).

**5. Camera capture reuses the existing streaming path; verification runs per-frame against an expected letter.**
Rationale: Reuse the MJPEG/Socket.IO plumbing already in `app.py`. The word verifier is a small state machine: expected-letter pointer advances only when the recognizer matches the current letter with sufficient stability across consecutive frames (mirrors the existing "accumulate over frames" approach in the translator).

### Feature 3 — additional quiz mode ideas (the requested brainstorm)
Beyond sign-the-word and sign-the-letter, candidate modes (all deck-driven):
- **Multiple choice recognition:** show an Indonesian word → pick the English (or vice-versa); the inverse of the Feature 1 card, no camera needed.
- **Picture → sign:** show the object picture, student fingerspells the word.
- **Fingerspelling → word:** app plays/animates a fingerspelled sequence, student types or picks the word (reverse fingerspelling reading).
- **Scrambled letters:** show the fingerspelling hand-shapes out of order, student signs them in the correct order.
- **Speed round / timed drill:** as many correct letters or words as possible in N seconds.
- **Spaced-repetition review:** prioritize words with the lowest mastery or longest since last correct.
- **Missing-letter fill:** show the word with one letter blanked, student signs only the missing letter.
- **Mixed exam:** randomized blend of modes across the whole deck for a "final."

### Extra product ideas (optional, beyond the four features)
- **Streaks, XP, badges** and a daily goal to drive retention.
- **Leaderboards / class mode** so a teacher can track a cohort.
- **Word categories / themed decks** (food, animals, home) for structured curricula.
- **Pronunciation/audio** of the Indonesian and English word alongside the sign.
- **Offline/PWA** packaging for classrooms with poor connectivity.
- **Accessibility:** left/right-hand mirroring option, adjustable per-letter timing, high-contrast reference images.
- **Two-way conversation practice** combining Feature 4 (read) with the fingerspelling producer (write).

## Risks / Trade-offs

- **ASL vs BISINDO mismatch** → The whole learning UX assumes fingerspelling; if the product must be BISINDO end-to-end, reference assets and the alphabet recognizer must be BISINDO. Mitigation: keep the alphabet pluggable and confirm the default early (Open Question).
- **Fingerspelling accuracy for lookalike letters** → false negatives frustrate learners. Mitigation: require stability over consecutive frames, show the expected letter, allow unlimited retries, and tune thresholds; consider a "skip letter" affordance.
- **Object detection gaps** → Feature 2 fails on out-of-vocabulary objects. Mitigation: clear "not recognized, retake" path and a classification fallback; constrain to supported categories in v1.
- **Camera privacy** → Mitigation: explicit consent, process frames in-memory, retain nothing by default; state this in the UI.
- **No real auth** → decks could be spoofed with a guessable id. Mitigation: acceptable for v1 learning tool; revisit if data sensitivity grows.
- **Distractor quality in Feature 1** → poor distractors make guessing trivial. Mitigation: curate distractors within the same category in the dictionary.

## Open Questions

- **Default sign system:** Is fingerspelling **ASL** (as literally written in Features 1–2) or **BISINDO** (matching the existing app and Feature 4)? This changes the reference assets and the alphabet recognizer choice. **Needs a product decision before implementation.**
- **Student identity model:** anonymous device id, class code, or lightweight login for v1 decks?
- **Object vocabulary scope:** limit Feature 2 to COCO's 80 classes, or add a classification fallback for broader coverage in v1?
- **Dictionary source:** curated static word list vs a translation API for Indonesian↔English and distractors?
- **Feature 4 status:** does it stay review-only, or should it be promoted into this change's implementation scope?
