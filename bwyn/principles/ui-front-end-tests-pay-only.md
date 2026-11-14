# UI/Front end Tests pay only for what they assert

For UI/Frontend:
An automated test runs only the work its assertions need. Anything else the code under test drags in, like styling, animation, real timers, real network or large fixtures, is replaced in the test environment. That keeps each test fast and the build reliable.

Hard rules:
1. No global CSS in the test page. The test environment gives the app an empty stylesheet. A test that checks a style loads the real stylesheet itself, for that test only.
2. No artificial delays in tests. User-event runs with delay: null, and tests use fake or controlled timers, never real waiting.
  3. A per-test time budget, enforced. No single test may take more than, say, 5 s. Going over it fails the build. Raising testTimeout is not an accepted fix.
4. Measure before tuning. Performance changes to the test suite start from per-test timings (jest --json), not from changing workers or timeouts.
5. Expensive queries in a loop need a reason. Prefer a narrower root (within(region)) or a cheaper query when a test runs the same getByRole many times. (This is guidance more than a hard rule.)
