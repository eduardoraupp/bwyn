# Story: Create a new account

- **Story**: 002
- **Created**: 2026-10-26
- **Status**: Draft

## Story
As a new user, I want to create an account so that I can access the system.

## Acceptance tests
1. **Given** a visitor has no existing account, **When** they submit the account creation form with a valid email address and password, **Then** a new account is created and the user is informed of success.
2. **Given** a visitor submits the form with an email address already in use, **When** the system processes the request, **Then** the account is not created and the user receives an error indicating the email is already taken.
3. **Given** a visitor submits the form with a missing email address or missing password, **When** the system processes the request, **Then** the account is not created and the user receives clear validation feedback.

## Business rules and constraints
- Email address is the unique identifier used to distinguish accounts.
- At minimum, email address and password are required to create an account (assumption: no additional fields confirmed yet).
- The story must be small enough for one developer to build, test and ship without handoff (Driver: Constraints).

## Out of scope
- Email verification after account creation.
- Any fields beyond email address and password.
- Duplicate submission handling and system-unavailability behaviour.
