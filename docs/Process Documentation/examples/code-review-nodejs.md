# Example – Code Review (Node.js)

## Scenario Snapshot
- **Goal:** review PR #412 that introduces `/vouchers/:id/cancel` endpoint changes.
- **Inputs:** Git diff, Jest results, ESLint output, Swagger updates, IBM i stored procedure references.
- **Output:** review comments plus an approval summarizing coverage, risk, and IBM dependencies.

## Step-by-Step Walkthrough
1. **Prep environment.** Fetch the branch, run `yarn` to sync dependencies, and start Redis via `docker-compose up -d redis`.
2. **Run automated checks.** Execute `yarn lint`, `yarn test`, and `yarn test:e2e`; paste results into the PR so Devs know the baseline.
3. **Trace new logic.** Inspect `cancel-voucher-entry.use-case.ts` and related controller/DTO files to ensure validation pipes align with the new payload.
4. **Validate IBM calls.** Confirm the repository still calls the correct stored procedure (`AP200PRC`) and that the parameters align with DB2 column types.
5. **Check observability.** Ensure new branches log context using Winston per `docs/DEBUGGING.md` so on-call can trace cancellations.
6. **Approve or request changes.** Summarize key findings (e.g., "Approved after verifying tests and DB2 mappings") or block until fixes arrive.

## Evidence Package
- Terminal output for lint/test/e2e runs.
- Screenshot or permalink referencing the stored procedure invocation.
- PR review comment summarizing the IBM dependency validation.
