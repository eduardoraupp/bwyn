# Story: Create a new account

- **Story**: 001-the-first-user-journey-is
- **Created**: 2026-10-26
- **Status**: Draft

## Story
As a new user, I want to fill in my name, surname and age and press a Register button so that I can create an account.

## Acceptance tests
1. **Given** the user navigates to the account creation page, **When** the page loads, **Then** a form is displayed containing a Name field, a Surname field, an Age field and a button labelled "Register".
2. **Given** the form is displayed, **When** the user enters a valid name, surname and age and presses Register, **Then** the system accepts the submission and confirms that the account has been created.
3. **Given** the form is displayed, **When** the user presses Register without filling in one or more fields, **Then** the system does not submit the form and indicates which fields are missing.
4. **Given** the form is displayed, **When** the user enters a value that is not a positive whole number in the Age field, **Then** the system does not submit the form and indicates that the age is invalid.
5. **Given** the form is displayed, **When** the user double-clicks Register, **Then** only one account creation request is sent.
6. **Given** the form is displayed, **When** the user enters a value containing only whitespace in the Name or Surname field and presses Register, **Then** the system does not submit the form and indicates that the field is invalid.

## Business rules and constraints
- The form must contain exactly three input fields: Name, Surname and Age (FR-001).
- Submission must be blocked when any field is empty or contains only whitespace (FR-003).
- Submission must be blocked when Age is not a positive whole number (FR-004).
- The Register button must not trigger more than one submission for the same user action (FR-005).
- The user must receive feedback identifying which fields failed validation (FR-006).
- The user must receive confirmation after a successful submission (FR-007).
- The frontend must be written in TypeScript (ADR 0011).
- The frontend must use Inversify with annotations for dependency injection where services are involved (ADR 0002).
- The story must be small enough for one developer to build, test and ship without handoff (Driver: Constraints).
- Assumption: "Age" is stored and validated as a positive whole number (integer ≥ 1).
- Assumption: A confirmation message displayed on the same page is sufficient; redirection after registration is not required unless clarified.
- Assumption: The backend endpoint that receives the registration data already exists or will be provided separately; this specification covers the frontend form only.

## Out of scope
- Navigation to the account creation page.
- Redirection after successful registration.
- Backend implementation of the account creation endpoint.
- Maximum and minimum accepted age values (pending clarification).
- Maximum character length or character restrictions for Name and Surname (pending clarification).