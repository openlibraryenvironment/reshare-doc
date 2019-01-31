# ReShare: testing philosophy

This document briefly summarises the consensus on testing in ReShare reached by the development meeting of 29th January 2019, by Charlotte, Chas, Ian, Kristen, Mike, Niels-Erik and Steve.

## Philosophical perspective

1. Our approach to testing is pragmatic: we write tests because doing so will increase robustness and reduce development time, not merely in order to have tests.
2. Accordingly, we trust experienced and talented developers to make good judgements about how much effort to invest in tests, and reject arbitrary code-coverage thresholds.
3. “It’s hard to retrofit correctness”, as a wise man once said, so we aim to start building tests at the beginning of developing any given module, so that running those tests is part of the development cycle from the start.
4. We aim to make the passing of tests a precondition for merging code to the master branch (from which releases will be made).
5. Since the success of a UI test can be invalidated by a change in a backend module, even when the UI code is unchanged, we aim to run UI tests when committing changes to a back-end module.

## Practical outworkings

1. We feel that experiments with mocking a back-end in other projects have not provided enough value to justify the work involved.
2. We also recognise that mocks can drift out of sync with the services that they mock, so that tests may pass against mocks when they would fail against a real service, or vice versa.
3. We therefore do not intend to mock the ReShare backend, but to run UI module tests against a real ReShare service. In other words, we will not strictly speaking build unit tests for UI modules, but use integration tests.
4. To provide stability of tests, we aim to ensure that a stable back-end is provided, running locked versions of back-end modules against which known-good UI code can run, with updates of that stable testing back-end being significant events.
5. Learning from the problems with UI tests in other projects, we aim to reduce assumptions about timing in our UI tests, replacing times waits with awaiting specified conditions of the UI.

