# 504 Gateway Timeouts – Diagnostic Playbook

This checklist explains how to confirm, isolate, and remediate 504 Gateway Timeout errors raised by ARG-WEB-BACKEND while customer use cases are executed. Follow it sequentially so evidence collected in each layer (gateway, NestJS service, DB2/RPG job, IBM i job logs) remains correlated.

## 1. Confirm the Symptom
- **Capture raw responses.** Use the frontend HAR, Postman, or `curl -v https://api.example.com/vendors` to show the 504 plus the request ID/trace headers passed through the gateway.
- **Check the upstream duration.** If the edge proxy exposes metrics (Azure Application Gateway, AWS ALB, Nginx ingress), confirm the backend timed out (usually >30s) rather than the client disconnecting early.
- **Note the impacted use case.** Record which NestJS controller and use case folder handled the request (for example, `/account-payable/voucher` maps to `src/main/account-payable/application/voucher`).

## 2. Fast Containment Steps
1. **Re-route traffic if needed.** If the health probe also fails, take the pod/task out of rotation with `kubectl scale` or the compose equivalent to stop cascading 504s.
2. **Restart long-running workers.** Use `npm run dev:status` to list containers and restart only the API container (`docker compose restart arg-backend-dev`) so Redis, DB2, and IBM i connectivity remain warm.
3. **Notify IBM i operators.** The RPG job queue may be hung; have operations verify if any `QZDASOINIT` jobs are stuck in MSGW.

## 3. API Layer Diagnostics
- **Enable HTTP trace logging.** Set `LOG_LEVEL=debug` and restart the API so Winston emits per-request spans (see `src/shared/infrastructure/logger.ts`).
- **Correlate use cases.** Use the request ID from the gateway to search `docker logs arg-backend-dev | rg "reqId=<value>"` and confirm which NestJS provider handled the call.
- **Look for synchronous blockers.** 504s often occur because a controller calls a repository that performs sequential DB2 reads. Check the relevant use case in `src/main/account-payable/application/**/usecases` for `await` chains that could be parallelized with `Promise.all`.
- **Inspect Bull/BullMQ queues.** If the use case enqueues background work, confirm the queue consumer is healthy: `docker exec -it arg-backend-dev redis-cli -n 0 LLEN bull:<queue>:wait`.

## 4. Database + IBM i Connectivity
1. **Sequelize connection pool.** Open `src/shared/infrastructure/connection.ts` and verify the configured pool size/timeouts. During the incident, run `SELECT * FROM QSYS2.DATABASE_CONNECTION_POOL_INFO` in IBM Data Studio to ensure handles are not exhausted.
2. **ODBC driver health.** From the container shell, run `isql ARGDSN -v` using the DSN defined in `odbc.ini` to ensure credentials still work.
3. **Job log inspection.** Use IBM Navigator (or 5250 `WRKACTJOB`) to inspect the job handling the request. Look for CPF/CPI messages referencing the stored procedure called from `src/main/account-payable/data/repositories`.
4. **Spooled output + data queues.** If the RPG program writes to an output queue, capture the spool file to see if it halted awaiting input.

## 5. Performance Profiling
- **Trace NestJS latency.** Wrap the problematic use case with the built-in logging interceptor or temporarily add `performance.now()` measurements per repository call. Keep the diff local while debugging.
- **Measure DB2 statement time.** Temporarily enable `logging: (sql) => this.logger.debug(...)` in the Sequelize repository (many already exist, e.g., `voucher.repository.ts` line ~640) to catch slow SQL.
- **Redis cache misses.** For endpoints backed by caching (`@nestjs/cache-manager`), use `redis-cli monitor | rg <key>` to confirm whether cache churn is forcing DB2 calls every time.

## 6. Resolution Paths
- **Increase upstream timeout only after fixes.** If a query legitimately needs >30s, ship an SLA adjustment along with proof that the IBM i program cannot be optimized further.
- **Optimize the RPG/DB2 routine.** Share captured SQL and job log evidence with the IBM i team so they can add indexes or refactor the CL/RPG procedure.
- **Add pagination or batching.** When a use case fetches thousands of rows (common in voucher exports), implement paging at the repository level and expose pagination params in the controller DTO.
- **Backfill monitoring.** Create a Grafana alert or Azure Monitor rule watching the `504` rate plus the NestJS duration histogram so future spikes auto-page the team.

## 7. Evidence Checklist
- HAR or curl output with 504 status and headers.
- Gateway metrics screenshot showing upstream timeout.
- Container logs filtered by request ID with timestamps for controller → repository calls.
- IBM i job log or spool capture tied to the same timestamp.
- Root-cause summary + remediation PR/CL link.
