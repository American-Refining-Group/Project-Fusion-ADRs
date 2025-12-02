# 500 Internal Server Errors – Diagnostic Playbook

Use this runbook when the API Gateway/proxy returns **HTTP 500** for a request that should succeed. A 500 indicates NestJS threw an unhandled exception or the IBM i call stack surfaced an error that was not translated to a user-facing response. Follow each section to capture evidence before applying code fixes.

## 1. Confirm the Symptom
- **Reproduce with tracing.** Trigger the failing endpoint via Postman or `curl -v` and capture the response headers (including `x-request-id`) plus the JSON error payload or HTML error page.
- **Check API logs immediately.** Run `docker logs arg-backend-dev | rg "reqId=<value>" -C3` to grab the stack trace surrounding the 500 before log rotation occurs.
- **Identify the use case.** Map the route to its controller/service (e.g., `/ap/vendors/:id` → `src/main/account-payable/application/vendors/usecases/get-vendor.usecase.ts`).

## 2. Containment
1. **Evaluate blast radius.** If the exception fires for every request, drain traffic from the instance (`docker compose scale api=0` or remove the pod from the load balancer) to stop noisy alerts.
2. **Rollback problematic releases.** Use the deployment manifest/git tag noted in `package.json` to redeploy the prior container image while the fix is prepared.
3. **Disable new features.** If the 500 originated behind a feature flag (see `src/config/feature-flags.ts`), flip it off so unaffected functionality remains online.

## 3. NestJS + Node Diagnostics
- **Enable verbose logging.** Temporarily set `LOG_LEVEL=debug` to capture per-provider logs via Winston (`src/shared/infrastructure/logger.ts`).
- **Trace interceptors and filters.** Inspect global exception filters (`src/shared/infrastructure/http/exception.filter.ts`) for missing branches that should translate known DB2 or validation errors to 4xx codes.
- **Validate DTOs.** When `class-validator` fails, it may throw before response mapping. Run the corresponding Jest suite (`npm run test <module>`) to surface schema drift.
- **Inspect async flows.** Many 500s stem from rejected promises in repositories. Open the repository under `src/main/**/data/repositories` and verify every `await` is wrapped in `try/catch` or re-thrown with context.

## 4. IBM i / DB2 Stack
1. **Correlate SQLSTATE.** From the stack trace, note the SQLSTATE or CPF message. Search IBM Navigator job logs (`WRKACTJOB` → option 5) for the same timestamp to confirm whether DB2 aborted the call.
2. **Review RPG return data.** If the CL/RPG program returns an error structure, ensure the Node adapter (`src/shared/infrastructure/ibmi/ibmi.service.ts`) maps it to a typed response instead of throwing.
3. **Check commitment control.** Use IBM Data Studio to confirm the stored procedure commits/rolls back as expected; lingering locks often manifest as CPF or SQL0911 messages bubbled up as 500s.

## 5. Resolution Paths
- **Add targeted exception handling.** Translate recurring SQL codes (e.g., duplicate keys) into HTTP 409/422 so the UI receives actionable feedback.
- **Patch the offending module.** Add tests reproducing the error path (controller → service → repository) before refactoring.
- **Coordinate IBM i fixes.** If the RPG program is defective, attach the job log, inputs, and SQL payload so the IBM team can recompile and notify when safe to redeploy.

## 6. Evidence Checklist
- HAR or curl output showing HTTP 500 + headers.
- Relevant container logs/stack traces tied to the request ID.
- IBM i job log snippet or SQLSTATE screenshot.
- Link to the code fix/rollback and automated tests proving the issue is resolved.
