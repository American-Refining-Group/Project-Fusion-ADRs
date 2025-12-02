# Approve QA Test Cases from Acceptance Criteria

Run through this checklist before you sign off on any manual or automated QA plan.

## Prep
- Open the relevant Jest specs (e.g., `src/modules/voucher/voucher.controller.spec.ts`).
- Ensure you can run the common scripts: `yarn test`, `yarn test:cov`, and `yarn lint`.
- Have the IBM validation queries ready, such as repository calls in `SpooledMetaDataReportsRepository`.

## Steps
1. **Trace each acceptance criterion.** For every bullet in the user story, confirm there is at least one test case referencing the same inputs and expected outputs.
2. **Verify automation coverage.** Identify which scenarios are already handled by Jest or integration tests. Mark any gaps that still need manual execution and explain why automation is not feasible.
3. **Check data setup.** Document how to seed required DB2 tables (fixtures, scripts, or RPG jobs). Include file/library names so QA can prep the environment quickly.
4. **List execution commands.** Provide the exact CLI command(s) QA should run (`yarn test voucher`, `yarn test --watch`, IBM CL commands, etc.).
5. **Define pass/fail evidence.** Describe what proof QA must attach (console output, screenshots, spool file IDs) to show the test case passed.
6. **Review regression impact.** Ensure smoke/regression suites are triggered whenever applicable. Note which pipelines (Azure, Jenkins) must show green checks before approval.

## Sign-off
- Record who approved the test cases and the date.
- Store links to the final test plan and any exported reports for traceability.
