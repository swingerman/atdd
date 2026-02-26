---
name: mutate
description: Run mutation testing to verify test quality. Detects the project language, sets up the appropriate mutation framework, runs mutations, and reports the mutation score with surviving mutants.
arguments:
  - name: ARGUMENTS
    description: Optional target path or flags (e.g., "src/auth/" to mutate only auth module)
    required: false
---

Run mutation testing on this project.

## Instructions

1. Check that both test streams are green before proceeding:
   - Run the unit tests
   - Run acceptance tests if a pipeline exists (`./run-acceptance-tests.sh`)
   - If either fails, report the failure and stop — do not mutate against a red baseline

2. Detect the project language and check if a mutation framework is already configured.
   If not, install and configure the appropriate one (see the `atdd-mutate` skill for the framework table).

3. Configure the framework to:
   - Target source code only (not tests, specs, generated files, or pipeline code)
   - Exclude `generated-acceptance-tests/`, `acceptance-pipeline/`, and `specs/`
   - If `$ARGUMENTS` is provided, scope mutations to that path

4. Run the mutation framework

5. Report results:
   - Mutation score (percentage)
   - Total mutants, killed, survived
   - List each surviving mutant with: file, line, mutation description
   - Assessment: strong (90%+), moderate (70-89%), or weak (<70%)

6. Ask whether to proceed with killing surviving mutants (invoke `/atdd:kill-mutants`)
