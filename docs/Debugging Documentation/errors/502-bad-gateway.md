# 502 Bad Gateway – Diagnostic Playbook

Open this guide when the edge proxy (API Gateway, ALB, ingress) returns **HTTP 502** indicating it cannot reach or trust the backend response. 502s usually mean the NestJS container crashed, rejected TLS, or returned malformed headers. Use this sequence to restore service quickly.

## 1. Confirm the Symptom
- **Capture raw responses.** Use `curl -v` or the UI HAR to record the 502 payload, timestamp, and request ID.
- **Inspect gateway health checks.** From Azure Application Gateway or AWS ALB metrics, verify the backend health probe is failing with `BadGateway`/`502` errors.
- **Check container status.** Run `docker compose ps` or `kubectl get pods` to see if the API container restarted or is in `CrashLoopBackoff`.

## 2. Containment
1. **Fail over to a healthy instance.** Remove the failing task/pod from the load balancer until diagnostics finish.
2. **Restart dependent services carefully.** Bring up Redis, DB2 tunnel, and the NestJS API in the correct order using `dev-start.sh` or your orchestrator playbook so connection retries do not cascade.
3. **Notify front-end teams.** Provide status so they can suppress repeat retries that might overload the gateway.

## 3. Application Diagnostics
- **Check boot logs.** Run `docker logs arg-backend-dev --since=10m` for startup stack traces (missing env vars, migration failures, etc.).
- **Validate environment files.** Ensure `.env`/secret mounts include required keys like `DB2_HOST`, `REDIS_URL`, and TLS certificates referenced in `src/config`.
- **Look for memory/CPU exhaustion.** Use `docker stats` or Kubernetes metrics to confirm the process is not being OOM-killed during heavy use cases.
- **Review HTTP adapters.** Misconfigured global interceptors (e.g., streaming large files without `Content-Length`) can cause the gateway to close the connection mid-flight.

## 4. IBM i / Connectivity Checks
1. **ODBC reachability.** From inside the container, run `isql ARGDSN -v` (matching `odbc.ini`) to confirm DB2 is online and TLS negotiation succeeds.
2. **Job queue saturation.** Use IBM Navigator or 5250 `WRKACTJOB` to ensure the required subsystem has free `QZDASOINIT` jobs; no available jobs can make the API appear offline.
3. **Stored procedure regression.** If a new RPG program was deployed, confirm its signature still matches the Sequelize binding; mismatched parameter counts may crash the job and close the socket.

## 5. Resolution Paths
- **Fix configuration drift.** Redeploy the container with the correct env/secret bundle or update Helm/docker-compose manifests.
- **Patch dependency updates.** Revert or adjust recent package upgrades that changed TLS/HTTP behavior (see `package.json` changelog).
- **Add health probes.** Ensure the NestJS `/health` endpoint exercises DB2/Redis connectivity so orchestration can detect failures before 502s reach users.

## 6. Evidence Checklist
- Gateway metrics screenshot showing 502 spikes + probe status.
- Container/pod state output (`docker compose ps` or `kubectl describe pod`).
- Relevant NestJS logs proving the crash or misconfiguration.
- IBM i/DB2 validation (isql output, job log snippet) if the backend dependency caused the outage.
