# Constraints

What cannot be negotiated: the technology that must be used, the rules to follow and the date to meet.

BDD acceptance-first testing is non-negotiable for all feature work.

- Every behaviour is specified as a Given/When/Then acceptance criterion before any implementation begins.
- Each acceptance criterion is automated as a JUnit 5 test (backend) or a Jest test (frontend) that exercises the full request-response cycle with mocked infrastructure adapters.
- A feature is not complete until its acceptance tests pass. No code is written that no failing acceptance criterion requires.
- Isolated unit tests are not required; adapters are covered by integration tests against real infrastructure (Testcontainers for the backend, per the accepted ADRs).
- The frontend uses Jest and React Testing Library for acceptance and integration tests.

Architectural enablers must also be tested, but BDD acceptance criteria are not required for them.
