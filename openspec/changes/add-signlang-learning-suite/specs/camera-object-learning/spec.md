## Purpose

Defines the capture-to-learn flow where a learner photographs a real-world object and the system turns it into a learnable word — detecting the object, showing its Indonesian and English names, and verifying the learner's fingerspelling before adding it to the deck.

## ADDED Requirements

### Requirement: Capture and detect an object
The system SHALL let a learner capture a photo with the camera and SHALL detect the primary object in the image, mapping it to a known word.

#### Scenario: Object is recognized
- **WHEN** a learner captures a photo of an object the system can detect
- **THEN** the system identifies the object and resolves it to a single word

#### Scenario: Object is not recognized
- **WHEN** the system cannot confidently detect a supported object in the photo
- **THEN** the system informs the learner that no object was recognized
- **AND** allows the learner to retake the photo

### Requirement: Show Indonesian and English words for the object
Once an object is detected, the system SHALL display both the Indonesian and English words for that object.

#### Scenario: Words displayed after detection
- **WHEN** an object is successfully detected
- **THEN** the system displays the Indonesian word and the English word for that object

### Requirement: Show the fingerspelling reference for the detected word
The system SHALL display the ordered fingerspelling reference (one hand-shape image per letter) for the detected word.

#### Scenario: Reference shown after detection
- **WHEN** the detected word is displayed
- **THEN** the system shows the hand-shape image for each letter of the word in order

### Requirement: Verify signing and add to deck
The system SHALL require the learner to fingerspell the detected word on camera and SHALL only mark it learned when signing is verified correct, delegating capture and per-letter checking to the fingerspelling-recognition capability.

#### Scenario: Correct signing marks the word learned
- **WHEN** the learner fingerspells the detected word and the signing is verified correct
- **THEN** the system marks the word as learned
- **AND** adds the word to the learner's deck

#### Scenario: Duplicate word already in deck
- **WHEN** the detected word is already present in the learner's deck
- **THEN** the system indicates the word is already learned
- **AND** does not create a duplicate deck entry
