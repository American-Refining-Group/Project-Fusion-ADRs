# 503 Service Unavailable – Diagnostic Playbook

Use this runbook when clients receive **HTTP 503** responses, usually indicating the backend is overloaded, throttled, or intentionally taken out of service. These steps help determine whether resource exhaustion, planned maintenance, or IBM i dependencies are responsible.

## 1. Confirm the Symptom
- **Capture the response.** Record the 503 payload/body and headers to see if it originates from the gateway (e.g., `Retry-After` present) or from NestJS itself.
- **Check incident calendar.** Verify whether maintenance windows or deploy freezes are in effect before treating it as an outage.
- **Review gateway metrics.** Use Application Gateway/ALB dashboards to confirm backend response codes and whether health probes also fail.

## 2. Containment
1. **Scale horizontally if possible.** Increase the replica/task count using your orchestrator or `docker compose up --scale api=2` to relieve pressure.
2. **Throttle heavy jobs.** Pause batch/BullMQ queues via `bull-board` or CLI to free CPU for synchronous traffic.
3. **Communicate status.** Update the status page/Slack with ETA and mitigation steps so users know when to retry.

## 3. API + Infrastructure Diagnostics
- **Inspect resource limits.** Check `docker stats`, Kubernetes HPA metrics, or VM monitoring for CPU/memory saturation.
- **Examine DB connection pools.** In `src/shared/infrastructure/connection.ts`, verify pool settings align with IBM i limits; run `SELECT * FROM QSYS2.DATABASE_CONNECTION_POOL_INFO` to see active handles during the incident.
- **Redis/backing service availability.** Timeouts to Redis or other dependencies (S3, messaging) can bubble up as 503. Review connection error logs in the NestJS container.
- **Deployment health.** Ensure the latest release finished migrations and warmed caches; partially rolled deployments often sit behind 503s while readiness probes fail.

## 4. IBM i Focus
1. **Subsystem availability.** Use 5250 `WRKSBS`/`WRKACTJOB` to confirm the subsystem hosting the RPG programs is active and not held.
2. **Job queue backlog.** If `QSYSOPR` shows numerous jobs in `JOBQ` or `MSGW`, IBM i may not accept new requests, causing the API to return 503s.
3. **Navigator performance metrics.** Review CPU and temporary storage usage; heavy RPG batch jobs can starve interactive jobs invoked by Node.

## 5. Resolution Paths
- **Tune autoscaling.** Adjust HPA or compose resource reservations so the service scales before saturation.
- **Optimize expensive endpoints.** Add caching/pagination to large exports that monopolize worker threads.
- **Coordinate maintenance.** When IBM i maintenance is planned, return a friendlier maintenance page and set accurate `Retry-After` headers via NestJS filters.

## 6. Evidence Checklist
- Screenshot/log of 503 responses with timestamps.
- Metrics proving resource limits (CPU/memory/connection pool) during the event.
- IBM i subsystem/job status snapshot.
- Documented mitigation (scaling, throttle, maintenance window) plus follow-up tasks.
