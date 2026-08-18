## 1. Foundations & shared data

- [ ] 1.1 Assemble the Indonesian↔English dictionary with per-word picture assets and per-word distractor sets
- [ ] 1.2 Assemble the ASL reference images for every letter (A–Z) and number (0–9)
- [ ] 1.3 Produce/license the two ASL lesson videos (letters, numbers) with Indonesian caption tracks
- [ ] 1.4 Choose and provision the persistence store (SQLite or per-student file store) and a student-identity scheme

## 2. Learner deck capability

- [ ] 2.1 Implement per-student deck storage (add word, list words, dedupe) per `learner-deck` spec
- [ ] 2.2 Implement mastery state tracking and update-on-quiz-result
- [ ] 2.3 Expose an eligible-words query for the quiz capability
- [ ] 2.4 Verify deck isolation and cross-session persistence

## 3. ASL recognition capability (shared — letters + numbers)

- [ ] 3.1 Add camera-consent gate before capture
- [ ] 3.2 Extract + normalize MediaPipe Hands landmarks; build the ASL recognizer (no-training KNN/template matcher over reference landmarks, or an existing pre-trained classifier) covering A–Z and 0–9
- [ ] 3.3 Implement single-letter and single-number match reporting against an expected target
- [ ] 3.4 Implement the letter-by-letter word verifier state machine with consecutive-frame stability
- [ ] 3.5 Handle the motion letters "J" and "Z" (animated reference + relaxed/start-end-pose match)
- [ ] 3.6 Emit progress feedback (matched letters/numbers + current expected target) to the UI

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
- [ ] 6.2 Implement sign-the-word mode via the ASL recognizer
- [ ] 6.3 Implement sign-the-letter mode
- [ ] 6.4 Implement sign-the-number mode
- [ ] 6.5 Score the quiz, show results, and update mastery
- [ ] 6.6 (Stretch) add one or more additional modes from design.md (e.g. multiple-choice recognition, reverse fingerspelling)

## 7. Feature 5 — ASL video lessons

- [ ] 7.1 Build the videos section listing the ASL letters and ASL numbers lessons
- [ ] 7.2 Play videos with Indonesian captions and track watch-to-end completion per student
- [ ] 7.3 On letters-video completion, launch a letter quiz; on numbers-video completion, launch a number quiz
- [ ] 7.4 Record post-video quiz results against the lesson

## 8. Feature 4 — sign-language translation (review-only, NOT implemented)

- [ ] 8.1 Keep the reviewed spec current; document research findings on word/sentence-level Indonesian sign translation (no implementation this change)

## 9. Integration, privacy & verification

- [ ] 9.1 Add routes/pages/navigation into the existing Flask app and templates (learn, capture, quiz, videos)
- [ ] 9.2 Add camera/photo consent copy and in-memory-only frame handling (retain nothing by default)
- [ ] 9.3 Run `openspec validate add-signlang-learning-suite --strict` and resolve findings
- [ ] 9.4 End-to-end test each flow: card → sign → deck; capture → sign → deck; deck → quiz (word/letter/number) → mastery; video → completion → quiz
