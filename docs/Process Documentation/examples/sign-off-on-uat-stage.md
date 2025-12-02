# Example – Sign Off on UAT (Stage)

## Scenario Snapshot
- **Goal:** certify release 2024.11 passed Stage after business testers ran regression suite.
- **Inputs:** Stage deployment logs, Azure Test Plans UAT suite, IBM Navigator job traces, monitoring dashboards.
- **Output:** sign-off email/post summarizing scope, blockers, and readiness.

## Step-by-Step Walkthrough
1. **Verify deployment.** Confirm the Stage slot shows the new container tag via `kubectl get deploy` or Azure App Service logs and that NestJS health endpoint `/health` returns 200.
2. **Review UAT cases.** In Test Plans, check that every test tied to the release is executed with Pass/Fail recorded; chase any "Blocked" entries.
3. **Spot-check data.** Run a few `/vouchers` and `/vendors` calls against Stage plus IBM Navigator queries to ensure the Stage DB2 library contains the expected records.
4. **Monitor jobs.** Ensure scheduled RPG jobs (e.g., nightly AP batch) run in Stage by reviewing job logs/spooled files after testers complete flows.
5. **Consolidate evidence.** Compile API logs, job numbers, and QA metrics into a short summary deck or wiki entry.
6. **Announce sign-off.** Email stakeholders or update Teams/Confluence with the summary, list of known issues, and explicit "Stage Approved" statement.

## Evidence Package
- Screenshot of Stage health endpoint returning OK.
- Azure Test Plans export showing completion percentages.
- IBM Navigator job log snippet proving RPG batch success.
