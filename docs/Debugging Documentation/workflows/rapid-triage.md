# Rapid Incident Triage (Full Stack)

This runbook combines the VS Code and IBM i workflows so on-call engineers can restore service fast while gathering the right evidence.

## 1. Confirm the symptom

1. Check the **API logs** (`npm run dev:logs` for docker-compose or your log aggregator) to confirm HTTP status and error signature. 【F:package.json†L16-L32】
2. Open the matching **error playbook** in `docs/errors` (500/502/503/504 or DB2) to follow its containment steps while you continue the deeper triage.
3. If the incident is user-specific, capture the **request payload** or queue job contents (Flexi uploads, ACH batches, etc.) from the Nest debugger before retrying. 【F:src/main/account-payable/application/voucher/shared-services/flexi.processor.ts†L1-L175】

## 2. Stabilize the environment

1. **Restart only what’s needed** – use `npm run docker:down` or restart the worker container whose queue is failing; avoid bouncing DB2 or Redis unless those services are confirmed unhealthy. 【F:package.json†L25-L33】
2. If Redis queues are flooded, temporarily pause the processor by commenting out `@OnQueueActive()` handlers or scaling workers to zero via docker-compose.
3. Capture a snapshot of `docker-compose ps` and any Kubernetes/VM state so platform teams can see what changed.

## 3. Debug in VS Code

1. Attach the VS Code debugger (`npm run start:debug`) and reproduce the failing route or job.
2. Use **logpoints** to emit DB2 DSN, schema, and invoice IDs so IBM operators can align job logs without waiting for screenshots. 【F:src/shared/infrastructure/connection.ts†L117-L233】
3. Inspect Sequelize’s SQL output (`customSequelizeLogger`) to see if the failure happens before or after the IBM stored procedure runs.

## 4. Cross-check on IBM i tools

1. Jump into IBM Navigator with the captured `jobNumber/jobUser` and review the job log + output queue status (see the IBM tooling guide for exact clicks).
2. If SQLSTATE errors appear, rerun the query inside ACS or Data Studio using the same schema from `env-library-config.ts` to confirm it fails outside the app too. 【F:src/shared/config/env-library-config.ts†L4-L83】
3. For RPG runtime errors, pull the source member from PDM and verify its compile timestamp; confirm whether a newer program was supposed to be moved as part of the current release.

## 5. Communicate status

1. Update the incident channel or ticket with:
   - Impacted endpoints/queues and the HTTP status observed.
   - Whether the `docs/errors/<symptom>.md` steps have been completed.
   - IBM job IDs / spool files reviewed and their status.
2. If a code fix is needed, create a short-term feature flag or revert referencing the modules touched (controllers, processors, repositories) and cite the last known good commit.

## 6. Capture learning

1. Add a **post-incident note** to the relevant docs:
   - `docs/errors/...` for symptom-specific handling.
   - `docs/debugging/workflows/...` if a new tool workflow emerged.
2. Check the code into a branch, add regression tests (`npm run test` / `npm run test:debug`), and run `npm run lint` before opening a PR. 【F:package.json†L10-L34】
3. During the PR review, link the incident ticket and call out any IBM i configuration changes so RPG/operations teams can audit them.
