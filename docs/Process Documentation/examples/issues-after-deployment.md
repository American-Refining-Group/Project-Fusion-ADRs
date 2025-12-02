# Example – Issues After Deployment

## Scenario Snapshot
- **Goal:** triage a production incident where `/vendors` returns 500 after release.
- **Inputs:** Application Insights logs, IBM job logs, rollback plan, hotfix branch.
- **Output:** incident timeline with root cause, fix, and prevention items.

## Step-by-Step Walkthrough
1. **Acknowledge alert.** Update ticket or respond to user, include RPG Team.
2. **Gather data.** Pull logs from `docs/DEBUGGING.md` instructions (Winston correlation IDs) and run `DSPJOBLOG` for any related IBM batch jobs.
3. **Reproduce safely.** Hit endpoint in Stage / reproduce, with the same payload to see if the issue repeats; capture the API response and DB queries.
4. **Identify root cause.** Compare the failing code to the previous commit; e.g., missing column mapping referencing `X` table.
5. **Issue fix.** Create a hotfix branch, patch the repository, run `yarn test` and targeted RPG tests if required, then deploy via emergency pipeline.
6. **Close incident.** Document the timeline, attach logs, note the production verification steps, and file follow-up actions (e.g., add regression test).

## Evidence Package
- Incident log with timestamps/actions.
- Before/after API responses plus correlation IDs.
- Hotfix PR link and deployment confirmation screenshot.
