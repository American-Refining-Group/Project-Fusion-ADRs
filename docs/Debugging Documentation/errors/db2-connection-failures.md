# DB2 Connection Failures (SQLSTATE 08001/08004) – Diagnostic Playbook

Follow this guide when the API logs show `SequelizeConnectionError`, `SQL0901`, `SQL30081N`, or SQLSTATE 08001/08004 errors indicating the Node process cannot open or retain DB2 connections on IBM i. These issues often manifest as HTTP 500/503 responses upstream, so documenting them separately keeps DB/ODBC actions clear.

## 1. Confirm the Symptom
- **Collect application logs.** Use `docker logs arg-backend-dev | rg -n "SequelizeConnection" -C3` to capture the first error plus retry attempts.
- **Check DB2 availability.** From IBM Data Studio or ACS, run a simple query (`SELECT CURRENT_DATE FROM SYSIBM.SYSDUMMY1`) to confirm the LPAR accepts connections.
- **Identify affected pools.** Note which Sequelize configuration (primary vs reporting) failed by checking `src/shared/infrastructure/connection.ts` or module-specific datasource files.

## 2. Containment
1. **Recycle idle handles.** Restart only the NestJS API container (`docker compose restart api`) so Redis/jobs remain intact while new DB2 sessions are negotiated.
2. **Throttle traffic.** Temporarily reduce concurrency (HPA min replicas, worker count) so DB2 is not flooded with retries that prolong the outage.
3. **Escalate to IBM i ops.** If the error references CPF/SQL messages (e.g., CPF9898), alert the IBM operator to inspect `QSYSOPR` for system-wide issues.

## 3. IBM i Diagnostics
- **Active job review.** Run `WRKACTJOB SBS(QUSRWRK)` and filter for `QZDASOINIT` to ensure jobs are not stuck in `MSGW`. Responding to the message often frees the port.
- **Connection limits.** Execute `SELECT * FROM QSYS2.NETSTAT_JOB_INFO` to see socket states; if the host reached its limit, you may need to end stale jobs with `ENDPJ` or `ENDJOB`.
- **Authority changes.** SQL08004 commonly signals revoked credentials. Verify the service profile still has the correct library list and authorities.

## 4. Application Diagnostics
- **Pool exhaustion.** Confirm `max`/`acquire` timeout settings in the Sequelize config are balanced with IBM i capacity (default `pool.max=5`). Increase only when IBM i operators approve additional jobs.
- **TLS/driver drift.** Check `odbc.ini` and the IBM i Access ODBC driver version used in the container. Recent OS/PTF updates may require a new driver level.
- **Long-running transactions.** Use `QSYS2.ACTIVE_JOB_INFO` to find transactions holding locks for >30s; they prevent new sessions from starting until commit/rollback.

## 5. Resolution Paths
- **Restart the host listener.** IBM ops can recycle `QUSRWRK` or DB2 TCP/IP servers if sockets are wedged.
- **Adjust pool/backoff.** Implement exponential backoff in `src/shared/infrastructure/connection.ts` and expose metrics so chronic spikes trigger alerts before saturation.
- **Coordinate credential updates.** When passwords/SSL certificates rotate, update Kubernetes secrets or `.env` simultaneously to avoid rejected logins.

## 6. Evidence Checklist
- Exact log excerpts with SQLSTATE/SQLCODE.
- isql or Data Studio screenshots proving success/failure after fixes.
- IBM i job/subsystem status before and after remediation.
- Link to configuration changes (Sequelize pool, secrets, driver updates) deployed to resolve the incident.
