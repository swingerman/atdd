---
name: atdd
description: Start the ATDD workflow for a new feature. Optionally provide a feature description as an argument.
---

Start the Acceptance Test Driven Development workflow.

Invoke the `atdd` skill to guide the full workflow:

1. Understand the feature (use the argument as starting context if provided)
2. Write Given/When/Then acceptance specs in `specs/`
3. Generate/update the test pipeline
4. Run specs (expect red)
5. Implement with TDD until both test streams pass
6. Review specs for implementation leakage

If no feature description was provided as an argument, ask the user
what feature they want to build.

**User's input:** $ARGUMENTS
