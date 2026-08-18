## Purpose

Defines the videos section where a student watches short ASL lessons — one for the manual alphabet (letters) and one for numbers, both with Indonesian captions — and is quizzed once a video is completed.

## ADDED Requirements

### Requirement: Provide the ASL letters and numbers lesson videos
The system SHALL provide a videos section containing at least two lessons: one teaching ASL signs for the letters and one teaching ASL signs for the numbers. Each video SHALL display Indonesian captions.

#### Scenario: Videos section lists both lessons
- **WHEN** a student opens the videos section
- **THEN** the system shows the ASL letters lesson and the ASL numbers lesson

#### Scenario: Captions are in Indonesian
- **WHEN** a student plays either lesson video
- **THEN** the system shows Indonesian captions for that video

### Requirement: Track video completion
The system SHALL track whether a student has completed each lesson video and SHALL treat a video as completed only when the student has watched it to the end.

#### Scenario: Video marked completed
- **WHEN** a student watches a lesson video to the end
- **THEN** the system records that video as completed for that student

#### Scenario: Partial watch is not completion
- **WHEN** a student stops a lesson video before the end
- **THEN** the system does not record the video as completed

### Requirement: Quiz the student after completing a video
When a student completes a lesson video, the system SHALL offer a quiz whose content matches the video: the ASL letters video leads to a letter quiz and the ASL numbers video leads to a number quiz, scored via the fingerspelling-recognition capability.

#### Scenario: Letters video triggers a letter quiz
- **WHEN** a student completes the ASL letters video
- **THEN** the system offers a quiz that asks the student to sign letters

#### Scenario: Numbers video triggers a number quiz
- **WHEN** a student completes the ASL numbers video
- **THEN** the system offers a quiz that asks the student to sign numbers

#### Scenario: Quiz result recorded against the lesson
- **WHEN** a student finishes a post-video quiz
- **THEN** the system records the quiz result for that lesson
