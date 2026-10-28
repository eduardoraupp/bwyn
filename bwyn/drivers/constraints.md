# Constraints

What cannot be negotiated: the technology that must be used, the rules to follow and the date to meet.

BDD acceptance-first testing (non-negotiable)
- Behaviour is specified as Given/When/Then acceptance criteria, each automated as a JUnit 5 test.
- Acceptance tests run the full request-response cycle with mocked infrastructure adapters, and are the last tests to turn green.
- Nothing is built that a failing acceptance criterion doesn't require. Isolated unit tests are not required.
- Adapters get integration tests against real infrastructure.
- The frontend uses Jest and React Testing Library.
