## Purpose

Defines the shared camera-based capability that captures a learner's hand signs and verifies them, letter by letter, against a target word or single letter — reused by word-card learning, camera-object learning, and quizzes.

## ADDED Requirements

### Requirement: Request camera access with consent
The system SHALL request camera access before capturing signs and SHALL not capture video until the learner grants permission.

#### Scenario: Permission granted
- **WHEN** a signing step begins and the learner grants camera access
- **THEN** the system begins capturing the camera stream for recognition

#### Scenario: Permission denied
- **WHEN** the learner denies camera access
- **THEN** the system does not capture video
- **AND** informs the learner that signing verification requires the camera

### Requirement: Recognize a single fingerspelled letter
The system SHALL recognize an individual fingerspelled letter from the camera and SHALL report whether it matches an expected letter, using existing recognition technology rather than a newly trained model.

#### Scenario: Correct letter signed
- **WHEN** the expected letter is a given letter and the learner signs that letter
- **THEN** the system reports the letter as matched

#### Scenario: Wrong letter signed
- **WHEN** the learner signs a letter different from the expected letter
- **THEN** the system reports the letter as not matched
- **AND** indicates which letter was expected

### Requirement: Verify a word letter by letter
The system SHALL verify a target word by checking each of its letters in order and SHALL report the word as correctly signed only when every letter is matched in sequence.

#### Scenario: All letters correct in order
- **WHEN** the learner signs each letter of the target word correctly and in order
- **THEN** the system reports the word as correctly signed

#### Scenario: A letter is signed incorrectly
- **WHEN** the learner signs a letter that does not match the expected next letter of the target word
- **THEN** the system does not report the word as complete
- **AND** indicates the current expected letter so the learner can retry that letter

### Requirement: Provide progress feedback during signing
The system SHALL indicate signing progress by showing which letters have been matched and which letter is currently expected.

#### Scenario: Progress updates as letters match
- **WHEN** the learner matches the next expected letter of a multi-letter word
- **THEN** the system marks that letter as complete
- **AND** advances the expected letter to the following letter
