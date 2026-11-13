---
name: testing
description: Choose worthwhile tests, define behavior and system boundaries, and write or review reliable tests. Use whenever tests are considered, planned, written, changed, reviewed, or removed, including deciding not to add tests. Includes Vanta Obsidian guidance.
---

# Testing

Use this skill before deciding whether to add tests, as well as when writing or changing them. Loading it does not require adding tests or using TDD.

Read the repository's applicable instructions, test configuration, and nearby test utilities first. For Vanta's Obsidian repository, also read [Testing in Obsidian](references/obsidian.md). Repository conventions govern tools and commands. Do not apply Vanta-specific rules to other projects.

## Choose what the test proves

State the customer-visible behavior or contract at the system boundary. Name the failure the test would catch. Test returned results, persisted changes, errors, or meaningful external effects. Avoid assertions about private helpers, internal call order, or incidental implementation choices.

Choose the smallest system that can prove that behavior. Cover business-rule permutations in focused subsystem tests. Use fewer integration and end-to-end tests to prove critical connections and customer flows. A mocked dependency cannot prove that the real integration works.

Balance the confidence gained against setup, runtime, diagnosis, flakiness, and maintenance costs. Reuse existing coverage. Do not add tests that repeat the implementation, assert what static checks already prove, or cover temporary details. If additional tests add little value, explain why and name the appropriate verification instead.

If a test needs many unrelated dependencies or permutations, reconsider the boundary. Extract a meaningful subsystem when it simplifies the design. Do not expose private helpers or introduce forwarding interfaces solely to make them mockable.

## Design the cases

Name tests as specific statements of behavior, such as "denies completion by a reviewer without permission." Cover the happy path and relevant failure, permission, and boundary cases. Stop when additional cases no longer add meaningful confidence.

Use Arrange, Act, Assert. Keep setup local enough to explain the case. Group related cases under the system's name and context. Several assertions are appropriate when they jointly establish one outcome.

Use factories with explicit overrides for values that determine the result. Prefer small in-memory dependency implementations at explicit interfaces. Verify an interaction only when the interaction itself is part of the contract, such as sending a required notification.

Keep cases independent. Reset mutable state and restore clocks, stubs, and other global changes. Control time, network responses, randomness, and ordering. Use explicit parameter tables when cases share a behavior; split tests when the table requires complicated branching.

## Change and verify

For unfamiliar untested code, consider a characterization test before changing behavior. Separate any necessary extraction from the behavior change. Preserve coverage for behavior that remains when refactoring or deleting code.

Use [tdd](../tdd/SKILL.md) when requested or when a bug has a cheap local regression target. Otherwise choose test-first or test-after based on the task. A regression test should demonstrate the original failure where practical, then pass with the fix.

Run the narrowest relevant test command, then affected integration checks where the behavior requires them. Inspect the actual result. Remove focused-test markers, debug output, and accidental skips before handoff. Report what the tests prove, commands and results, and any unverified behavior. Use manual exploration when automation cannot economically cover the relevant uncertainty; describe that limit explicitly.
