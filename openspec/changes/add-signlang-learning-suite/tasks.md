## 1. Foundations & shared data

- [ ] 1.1 Decide default sign system (ASL vs BISINDO) and record it in `design.md` Open Questions resolution
- [ ] 1.2 Assemble the Indonesian↔English dictionary with per-word picture assets and per-word distractor sets
- [ ] 1.3 Assemble the fingerspelling reference images (one hand-shape per letter) for the chosen alphabet
- [ ] 1.4 Choose and provision the persistence store (SQLite or per-student file store) and a student-identity scheme

## 2. Learner deck capability

- [ ] 2.1 Implement per-student deck storage (add word, list words, dedupe) per `learner-deck` spec
- [ ] 2.2 Implement mastery state tracking and update-on-quiz-result
- [ ] 2.3 Expose an eligible-words query for the quiz capability
- [ ] 2.4 Verify deck isolation and cross-session persistence

## 3. Fingerspelling recognition capability (shared)

- [ ] 3.1 Add camera-consent gate before capture
- [ ] 3.2 Integrate an existing fingerspelling/alphabet recognizer on top of MediaPipe Hands landmarks (no training)
- [ ] 3.3 Implement single-letter match reporting against an expected letter
- [ ] 3.4 Implement the letter-by-letter word verifier state machine with consecutive-frame stability
- [ ] 3.5 Emit progress feedback (matched letters + current expected letter) to the UI

## 4. Feature 1 — word-card learning

- [ ] 4.1 Card screen: show Indonesian word + four distinct English options (one correct)
- [ ] 4.2 On guess, flip the chosen option to its picture; loop until correct
- [ ] 4.3 On correct, show the ordered fingerspelling reference for the word
- [ ] 4.4 Wire the signing step to fingerspelling-recognition; add to deck on verified success

## 5. Feature 2 — camera object learning

- [ ] 5.1 Photo capture UI and object detection via YOLOv8 (+ classification fallback)
- [ ] 5.2 Resolve detected object to Indonesian + English words via the dictionary; handle "not recognized / retake"
- [ ] 5.3 Show fingerspelling reference for the detected word
- [ ] 5.4 Wire the signing step; mark learned and add to deck (skip duplicates)

## 6. Feature 3 — sign quiz

- [ ] 6.1 Enforce minimum-deck-size gate before starting a quiz
- [ ] 6.2 Implement sign-the-word mode via fingerspelling-recognition
- [ ] 6.3 Implement sign-the-letter mode
- [ ] 6.4 Score the quiz, show results, and update mastery
- [ ] 6.5 (Stretch) add one or more additional modes from design.md (e.g. multiple-choice recognition, reverse fingerspelling)

## 7. Feature 4 — sign-language translation (review-only)

- [ ] 7.1 Confirm scope: keep review-only or promote to implementation
- [ ] 7.2 If promoted, wrap the existing BISINDO detection loop and add English-alongside-Indonesian output + confidence control

## 8. Integration, privacy & verification

- [ ] 8.1 Add routes/pages/navigation into the existing Flask app and templates
- [ ] 8.2 Add camera/photo consent copy and in-memory-only frame handling (retain nothing by default)
- [ ] 8.3 Run `openspec validate add-signlang-learning-suite --strict` and resolve findings
- [ ] 8.4 End-to-end test each flow: card → sign → deck; capture → sign → deck; deck → quiz → mastery
