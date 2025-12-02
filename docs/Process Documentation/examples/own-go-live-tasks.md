# Example – Own Go-Live Tasks

## Scenario Snapshot
- **Goal:** execute all production cutover actions for release 2024.11 of the ARG Web Backend + IBM i batch changes.
- **Inputs:** Cutover checklist, deployment pipeline, IBM CL scripts, monitoring runbook.
- **Output:** Go-live report showing each step, timestamp, and owner.

## Step-by-Step Walkthrough
1. **Freeze scope.** Confirm no new PRs are merged after the go/no-go meeting and note the final commit SHA.
2. **Prep environments.** Scale staging to production-like size, warm caches via `yarn seed:prod`, and notify IBM ops about the RPG deployment window.
3. **Deploy Node backend.** Trigger the Azure DevOps release pipeline; monitor container logs until `/health` is green in production.
4. **Promote RPG objects.** Run the approved `SAVOBJ/RSTOBJ` or change management scripts to move RPG programs into PRODLIB and capture job logs.
5. **Execute smoke tests.** Call `/vouchers`, `/vendors`, and run an IBM Navigator query to confirm data flows end-to-end. Record correlation IDs.
6. **Communicate status.** Update the go-live Teams channel with start/end times, note any incidents, and attach evidence before closing the change record.

## Evidence Package
- Completed cutover checklist with timestamps.
- Azure pipeline logs plus IBM job log PDFs.
- Smoke test summary including API IDs and DB2 query outputs.
