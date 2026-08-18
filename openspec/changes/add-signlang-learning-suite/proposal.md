## Why

The current product is a one-way BISINDO sign-language *translator* (camera → Indonesian text). It helps hearing people read signs, but it does nothing to help a learner *acquire* vocabulary or *produce* signs themselves. We want to turn the existing detector into an interactive, gamified learning app so students can build a personal vocabulary of words they can both recognize (Indonesian ↔ English) and physically fingerspell, verified live through the camera. The goal is to reuse existing computer-vision technology (YOLOv8 object detection, MediaPipe Hands, the existing BISINDO model, and off-the-shelf fingerspelling recognition) rather than train new models.

> **Note on sign system (needs sign-off):** Feature 1 and Feature 2 describe fingerspelling in **American Sign Language (ASL)**, while the existing app and Feature 4 are built around **BISINDO (Indonesian sign language)**. This PRD is written to keep the fingerspelling alphabet configurable so the same flows work for either system, but the default alphabet is a product decision the team must confirm. See `design.md` → Open Questions.

## What Changes

- **Guided word-card learning (Feature 1):** A student picks a card showing an Indonesian word, guesses among 4 English options, and each chosen option flips to a picture so the student can self-correct. On the correct answer, the app shows the fingerspelling reference for the word and asks the student to sign each letter on camera; when signed correctly the word is added to their deck.
- **Camera "capture-to-learn" (Feature 2):** A student photographs a real-world object; the app detects it, shows the Indonesian and English words, presents the fingerspelling reference, verifies the student's signing on camera, and adds the learned word to their deck.
- **Deck & mastery tracking:** A per-student deck records learned words and their mastery state, and is the source of truth that quizzes draw from.
- **Camera fingerspelling recognition (shared):** A reusable capability that captures a sequence of hand signs from the camera and checks them, letter by letter, against a target word — used by Features 1, 2, and 3.
- **Quiz mode (Feature 3):** Quizzes generated from the student's learned deck, including sign-the-word, sign-a-single-letter, and additional modes proposed in `design.md`.
- **Sign-language translation (Feature 4, under review):** Formalize the existing camera → Indonesian/English text translation as a first-class capability, reusing the current BISINDO model. Scoped as review-only in this proposal.
- Adds new dependencies/tech: an object-detection path for arbitrary objects (broad YOLOv8 classes or an image-classification fallback), a fingerspelling alphabet recognizer, an Indonesian↔English word mapping/dictionary source, and per-student persistence for decks and progress.

## Capabilities

### New Capabilities
- `word-card-learning`: The guided card flow where a learner maps an Indonesian word to its English meaning via progressive picture reveals, then fingerspells it (Feature 1).
- `camera-object-learning`: The capture-a-photo → detect object → learn-and-sign flow that grows the deck from the real world (Feature 2).
- `fingerspelling-recognition`: Shared camera-based capture and per-letter verification of a learner signing a target word or letter.
- `learner-deck`: Per-student storage of learned words plus their mastery state; the source quizzes draw from.
- `sign-quiz`: Deck-driven quizzes with multiple modes that assess recognition and production of signs.
- `sign-language-translation`: Real-time camera → Indonesian/English text translation reusing the existing BISINDO detector (Feature 4, review-only).

### Modified Capabilities
<!-- None. The existing translator is not yet described as an OpenSpec capability; Feature 4 introduces `sign-language-translation` as a new spec rather than modifying an existing one. -->

## Impact

- **Existing code:** `app.py` (Flask + Socket.IO streaming), `templates/`, and `static/` gain new routes/pages and client flows. The current YOLO/MediaPipe pipeline (`bisindo.pt`, `yolov8n.pt`) is reused; `sign-language-translation` wraps today's detection loop.
- **New dependencies/data:** object-detection classes or an image-classification fallback, a fingerspelling alphabet recognizer (existing model/library), an Indonesian↔English dictionary and word→picture asset source, fingerspelling reference images per letter, and per-student persistence (DB or file store) with session/identity for decks.
- **Privacy/permissions:** camera and photo capture require explicit consent; captured frames should be processed and not retained by default.
- **Non-goals for this change:** training any new model, full account/auth system beyond identifying a student, and shipping Feature 4 beyond a reviewed spec.
