# Sign Off on UAT (Stage) – Tool Playbook

Use the same interface stack testers rely on so approvals mirror reality.

## Core Tools
- **VS Code / REST client** – fire API smoke tests against the Stage environment endpoints listed in `.env.stage`.
- **IBM Navigator for i** – monitor Stage job queues, subsystem status, and logs while UAT runs.
- **IBM Data Studio** – query the Stage DB2 schema to verify data migrations.
- **5250 emulator** – check Stage library lists, data queues, or CL monitors.

## Guided Steps
1. **Load Stage configuration in VS Code.** Review `.env.stage` (or Azure DevOps variable group) to confirm URLs, ports, and IBM i hosts.
2. **Trigger smoke tests.** Use Thunder Client/REST Client collections stored in the repo to hit the Stage API, capturing responses and latency metrics.
3. **Monitor IBM i activity.** In Navigator, filter *Active Jobs* by the Stage subsystem (e.g., `QHTTPSVR`) and ensure no messages accumulate in the queue.
4. **Validate data persistence.** Run Data Studio queries against Stage libraries/tables affected by the release. Compare counts to pre-release baselines.
5. **Check on-host indicators.** From a 5250 session, verify Stage data queues, message queues, or batch jobs (e.g., `WRKACTJOB SBS(STAGESBS)`) look healthy.
6. **Log approval.** Update the UAT checklist with API response IDs, Navigator screenshots, and SQL outputs.

## Evidence to Capture
- REST client export showing Stage responses.
- Navigator job or subsystem snapshots.
- Data Studio query results referencing Stage libraries.
