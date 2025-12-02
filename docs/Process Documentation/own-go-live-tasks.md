# Own Go-Live Tasks

Use this checklist to lead a production release from start to finish.

## Prep
- Confirm the release scope, change tickets, and rollback plan.
- Ensure Docker images or builds are ready (run `yarn build`, `docker build`, etc.).
- Coordinate timing with IBM teams responsible for RPG promotions and spool queues.

## Steps
1. **Validate artifacts.** Run automated tests (`yarn lint`, `yarn test`, `yarn build`) and confirm CI is green. Record build IDs and Docker tags.
2. **Review deployment script.** Walk through `deploy.sh` (login, pull image, stop old container, start new). Update environment variables or secrets if needed.
3. **Announce freeze window.** Notify stakeholders of the deployment time, expected impact, and communication channel (Teams, Slack, bridge line).
4. **Execute deployment.** Follow the script, capturing console output and timestamps for each command.
5. **Run smoke tests.** Hit key APIs and user journeys, plus verify IBM queue/library connectivity (`DynamicLibraryManager` output, report generation, etc.).
6. **Monitor systems.** Watch logs, APM dashboards, and Bull queue UIs for anomalies during the launch window.
7. **Coordinate IBM steps.** Ensure RPG promotions, spool queue routing, or CL commands finish successfully. Capture job logs if issues appear.
8. **Communicate status.** Provide periodic updates until the release is stable, including any hotfix or rollback decisions.

## Sign-off
- Document the final status, including production verification results and any incidents.
- Archive deployment logs and share them with the operations team.
