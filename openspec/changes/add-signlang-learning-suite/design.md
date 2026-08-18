## Context

The app today is a Flask + Socket.IO server (`app.py`) that streams webcam/YouTube frames, runs a YOLOv8 BISINDO model (`bisindo.pt`) plus MediaPipe Hands for landmark overlay, and emits detected labels to the browser over a socket. There is no notion of a user, no persistence, and no "produce a sign" path — only "read a sign." This change layers a learning product on top of that pipeline. See `proposal.md` → Why for motivation. Guiding constraint: **reuse existing CV tech, train nothing new.**

## Goals / Non-Goals

**Goals:**
- Reuse the existing detection stack (YOLOv8, MediaPipe) and add an off-the-shelf recognizer for object detection and for the **ASL alphabet + numbers**.
- Teach and recognize **ASL** — the manual alphabet (A–Z) **and numbers (0–9)** — as the single, confirmed sign system for the learning flows.
- Introduce lightweight per-student persistence (deck + mastery + video completion) without a full auth system.
- Make `fingerspelling-recognition` a single shared recognizer (letters and numbers) consumed by Features 1, 2, 3, and the video quizzes.

**Non-Goals:**
- Training a recognition model **from scratch** (a light fine-tune of an existing classifier on public ASL datasets is acceptable if accuracy demands it — see Architecture).
- Full account system (email/password, social login). A simple student identifier is enough for v1.
- Implementing Feature 4 — it stays a reviewed, research-only spec.
- Continuous (non-fingerspelling) sign production grading — v1 grades static ASL letters and numbers only (note: ASL "J" and "Z" involve motion and are handled as the small motion exception in Architecture).

## Decisions

**1. ASL letter + number recognition via existing tech (MediaPipe Hands + a pre-built classifier or landmark matcher).**
Rationale: The manual alphabet and digits are a well-solved, ~36-class static-hand-pose problem (26 letters + 10 numbers). MediaPipe Hands already ships in the repo for landmark extraction; landmarks can feed an existing open ASL alphabet/number classifier or a small KNN/template matcher over reference landmarks — no from-scratch training. See the Sign Detection Architecture section for the full pipeline and the training-vs-no-training answer. Alternative considered: a full sign-language-production model (rejected — needs training and continuous-motion data).

**2. Object detection reuses YOLOv8, with an image-classification fallback for broad coverage.**
Rationale: `yolov8n.pt` (COCO, 80 classes) is already present and covers many everyday objects for Feature 2. For objects outside those classes, fall back to an off-the-shelf image classifier / vision API. Alternative: retrain YOLO on a custom object set (rejected — training).

**3. Indonesian↔English mapping and picture assets come from a static dictionary + asset pack.**
Rationale: Deterministic, offline-friendly, and lets us control the four-option distractors and per-word pictures for Feature 1. Detected class labels (English) map through this dictionary to Indonesian and to picture/fingerspelling assets. Alternative: live translation API (viable but adds a network dependency and non-determinism for distractor generation).

**4. Deck + mastery persisted server-side keyed by a student id.**
Rationale: Enables cross-session decks and quiz eligibility. Start with a simple store (SQLite or a JSON/file store per student id) to avoid infra weight; the `learner-deck` spec is storage-agnostic. Alternative: browser localStorage (rejected — not portable across devices, harder to quiz server-side).

**5. Camera capture reuses the existing streaming path; verification runs per-frame against an expected letter/number.**
Rationale: Reuse the MJPEG/Socket.IO plumbing already in `app.py`. The word verifier is a small state machine: the expected-target pointer advances only when the recognizer matches the current letter (or number) with sufficient stability across consecutive frames (mirrors the existing "accumulate over frames" approach in the translator).

**6. Video lessons are hosted assets with caption tracks; completion + post-video quiz are app state.**
Rationale: The two ASL lessons (letters, numbers) are plain video assets with Indonesian caption tracks (e.g. WebVTT), so no CV is involved in playback. The app tracks watch-to-end completion and, on completion, launches the matching quiz (letter quiz after the letters video, number quiz after the numbers video) through the same `fingerspelling-recognition` + `sign-quiz` path. Alternative: gate quizzes behind a manual "I finished" button (rejected — completion should reflect actual viewing).

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

## Sign Detection Architecture

There are **two independent detection problems** in this product. They have very different difficulty, and only one of them is in scope for implementation.

### A. ASL letter + number recognition (Features 1, 2, 3, 5) — IN SCOPE

**What it is:** classify a single static hand pose into one of ~36 classes (A–Z + 0–9). This is a small, well-understood pose-classification problem, not "sign language translation."

**Pipeline (per camera frame):**
1. **Hand localization + landmarks — MediaPipe Hands (already in the repo).** For each frame it returns 21 3-D hand landmarks. This does the hard vision work for us and is pre-trained.
2. **Normalize landmarks.** Translate to a wrist-origin, scale by hand size, and (optionally) mirror for handedness, so the features are position/'distance-from-camera' invariant. Result: a small fixed-length feature vector per frame.
3. **Classify the pose → a letter or number.** Two viable options, both avoiding from-scratch training:
   - **(Recommended, zero training) Landmark template / KNN matcher.** Store a handful of reference landmark vectors per class (captured once, or from a public dataset) and match the live vector by nearest-neighbor / cosine similarity. Adding numbers is just adding 10 more classes of reference samples. Fully transparent and easy to tune.
   - **(Also fine) An existing pre-trained ASL classifier.** Reuse an off-the-shelf ASL alphabet/number model (there are public models and datasets such as ASL alphabet sets, Sign Language MNIST for letters, and ASL digit sets). If a single model doesn't cover both letters and numbers, combine a letter model and a number model, or fall back to the KNN approach which covers both uniformly.
4. **Temporal stability gate.** Require the same predicted class across N consecutive frames above a confidence threshold before accepting it — this reuses the existing "accumulate over frames" idea and kills flicker/false positives.
5. **Verifier state machine.** For a word, hold a pointer to the expected next letter; accept and advance only when step 4 confirms that letter. For a single-letter or single-number quiz item, one confirmed match scores it.

**Do we need to train a model? — Short answer: NO (not from scratch).**
- MediaPipe already provides the trained hand-detection/landmark model.
- The 36-class letter+number classifier can be a **no-training KNN/template matcher** over landmarks, or an **existing pre-trained** classifier.
- **Optional, only if accuracy is insufficient:** a *light fine-tune* of a small classifier head on **public** ASL datasets (minutes on a CPU/GPU, no data collection or architecture design). That's an upgrade path, not a prerequisite.
- **Motion exception:** ASL "J" and "Z" (and no numbers) involve movement. Handle them as a tiny special case — match the letter's start-and-end poses in sequence, or, for v1 simplicity, present them with a short animated reference and a relaxed match. Everything else is static.

**Adding numbers changes almost nothing:** numbers 0–9 are additional static classes fed through the exact same pipeline — 10 more sets of reference landmarks (KNN) or 10 more labels (classifier). The recognizer, the stability gate, and the verifier are unchanged.

### B. BISINDO word/sentence translation (Feature 4) — UNDER REVIEW, NOT IN SCOPE

**What it is:** read Indonesian sign language from video and produce Indonesian/English **words and sentences**. This is a fundamentally harder problem: continuous, two-handed, grammar-carrying signing with non-manual markers, and no reliable off-the-shelf model exists for open-vocabulary sentence translation. The existing `bisindo.pt` YOLO model only classifies a **fixed, limited set** of isolated signs — useful as a demo, not a general translator.

**Conclusion:** we keep Feature 4 as a documented, review-only spec (isolated-sign demo only) and continue research. If pursued later, realistic paths are (a) staying at the isolated-word level with a curated vocabulary, or (b) adopting a future pre-trained continuous-sign model — either would be its own change. This is why Feature 4 is explicitly deferred.

## Risks / Trade-offs

- **ASL vs BISINDO mismatch** → The whole learning UX assumes fingerspelling; if the product must be BISINDO end-to-end, reference assets and the alphabet recognizer must be BISINDO. Mitigation: keep the alphabet pluggable and confirm the default early (Open Question).
- **Fingerspelling accuracy for lookalike letters** → false negatives frustrate learners. Mitigation: require stability over consecutive frames, show the expected letter, allow unlimited retries, and tune thresholds; consider a "skip letter" affordance.
- **Object detection gaps** → Feature 2 fails on out-of-vocabulary objects. Mitigation: clear "not recognized, retake" path and a classification fallback; constrain to supported categories in v1.
- **Camera privacy** → Mitigation: explicit consent, process frames in-memory, retain nothing by default; state this in the UI.
- **No real auth** → decks could be spoofed with a guessable id. Mitigation: acceptable for v1 learning tool; revisit if data sensitivity grows.
- **Distractor quality in Feature 1** → poor distractors make guessing trivial. Mitigation: curate distractors within the same category in the dictionary.

## Resolved Decisions

- **Sign system: ASL, letters AND numbers (CONFIRMED).** Features 1, 2, 3, and 5 teach and recognize the ASL manual alphabet (A–Z) and numbers (0–9). BISINDO is only referenced by the review-only Feature 4.
- **Feature 4: research-only (CONFIRMED).** Not implemented in this change; kept as a documented spec pending research (see Architecture § B).
- **Student identity: lightweight login (CONFIRMED).** A minimal sign-in (e.g. username/display name + simple credential, or a class code) identifies a student so their deck, mastery, and video completion persist and stay isolated. Explicitly NOT a full/enterprise auth system (no SSO, roles, or heavy account-recovery flows). Captured in the new `student-account` capability.
- **Feature 2 object scope: start with YOLOv8's 80 COCO classes (CONFIRMED for v1).** `yolov8n.pt` is already in the repo, reliable, offline, and localizes objects. An image-classification fallback for broader coverage is a **later** enhancement, added only if learners hit "not recognized" too often — not v1.
- **Indonesian↔English source: curated static dictionary (CONFIRMED for v1).** Because each word also needs a picture and three quality distractors — which a translation API cannot provide — a hand-curated dictionary (English, Indonesian, picture, distractors) is the source of truth. It only needs to cover the card vocabulary plus the ~80 YOLO labels. A **free translation API (e.g. LibreTranslate / MyMemory) is an optional later fallback**, used solely to auto-translate a detected object label that is not yet in the dictionary; not required for v1.
- **ASL recognizer: KNN / landmark matcher over MediaPipe Hands (CONFIRMED for v1).** MediaPipe landmarks already ship in the repo; a nearest-neighbor match over a few reference samples per sign covers all 36 letters+numbers uniformly with zero training. Preferred over a pre-trained classifier because most ready-made ASL models are letters-only (numbers would need a second model). Fallback if look-alike accuracy is weak: a light fine-tune of a small classifier on public ASL datasets (see Architecture § A).
- **Videos: recorded in Indonesian narration; Indonesian captions optional but recommended (CONFIRMED).** The team records the two lessons with spoken-Indonesian instruction. Because part of the audience is deaf/hard-of-hearing, on-screen Indonesian text/captions are strongly recommended for accessibility, but not a hard requirement for v1.

## Open Questions

- **Motion letters:** confirm the v1 handling for the moving signs "J" and "Z" (animated reference + relaxed match vs start/end-pose matching).
- **Lightweight-login mechanism:** exact form — display name + PIN, email magic link, or a teacher-issued class code + student name? (All qualify as "lightweight"; pick per how classrooms will onboard.)
- **Caption production:** if Indonesian captions are added, confirm the format (e.g. WebVTT) and whether they are burned-in or a toggleable track.
