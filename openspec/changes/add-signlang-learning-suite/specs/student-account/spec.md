## Purpose

Defines a lightweight sign-in that identifies a student so their deck, mastery, and video progress persist and stay private to them — without a full enterprise auth system.

## ADDED Requirements

### Requirement: Lightweight student sign-in
The system SHALL let a student sign in with a lightweight credential and SHALL associate their session with a stable student identity. It SHALL NOT require a full/enterprise auth system (no SSO or role management).

#### Scenario: Student signs in
- **WHEN** a student provides valid lightweight sign-in details
- **THEN** the system establishes a session tied to that student's identity

#### Scenario: First-time student is registered
- **WHEN** a new student signs in for the first time
- **THEN** the system creates a student identity for them
- **AND** starts an empty deck and progress for that identity

### Requirement: Bind learning data to the signed-in student
The system SHALL scope decks, mastery, and video completion to the signed-in student's identity and SHALL keep them private to that student.

#### Scenario: Data follows the student across sessions
- **WHEN** a student signs in on a later session
- **THEN** the system loads that student's own deck, mastery, and video progress

#### Scenario: Another student cannot see the data
- **WHEN** a different student signs in on the same device
- **THEN** the system shows only their own deck and progress, not the previous student's

### Requirement: Sign out
The system SHALL allow a student to sign out, ending their session so their data is no longer accessible until they sign in again.

#### Scenario: Student signs out
- **WHEN** a signed-in student chooses to sign out
- **THEN** the system ends the session
- **AND** does not display that student's deck or progress until the next sign-in
