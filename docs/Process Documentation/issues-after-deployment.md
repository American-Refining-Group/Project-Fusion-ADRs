# Issues After Deployment

Use these bullets to triage and resolve incidents quickly after a release.

## Prep
- Open `docs/DEBUGGING.md` for the Node ↔ RPG troubleshooting workflow.
- Ensure you have access to production logs, Bull queues, and IBM job logs/output queues.
- Gather the deployment details (build number, timestamp, change list).

## Steps
1. **Stabilize the environment.** Decide whether to rollback, hotfix, or continue monitoring. Document the decision and stakeholders involved.
2. **Capture evidence.** Record error messages, screenshots, API payloads, and IBM job numbers as soon as the issue is reported.
3. **Trace the flow.** Follow the debugging guide from HTTP request → queue → processor → database → RPG job → spool file to pinpoint the failing hop.
4. **Inspect persisted metadata.** Query `SpooledMetaDataReportsRepository` or relevant tables to verify whether jobs completed and what filenames/queues they used.
5. **Collect IBM artifacts.** Download job logs, spool files, or dumps referenced by the incident for later analysis.
6. **Communicate status.** Provide regular updates to the incident channel, including ETA for fixes and any customer impact.
7. **Apply fix or rollback.** Execute the chosen remediation (config change, code patch, revert) and note every command or deployment performed.
8. **Verify resolution.** Re-run the failing scenario plus a small regression set. Confirm monitoring alerts have cleared.

## Sign-off
- Publish a post-incident summary with root cause, fix, and follow-up tasks.
- Store all logs and evidence in the incident ticket for audit purposes.
