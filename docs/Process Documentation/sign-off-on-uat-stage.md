# Sign Off on UAT (Stage)

Use these bullets to release a build into the stage/UAT environment confidently.

## Prep
- Verify the `.env` or pipeline variables reference the `uat` section in `env-library-config.ts`.
- Review the Swagger output for the build (`/api-docs`) to confirm endpoints and DTOs.
- Ensure access to the shared output queues and data libraries used in UAT.

## Steps
1. **Confirm configuration.** Double-check service URLs, credentials, and library overrides match the UAT plan. Document any differences from PROD.
2. **Deploy the build.** Run the approved deployment steps (`yarn build`, Docker image push, `deploy.sh`, etc.) and capture console output.
3. **Smoke test APIs.** Execute a short list of high-value requests (voucher upload/download, report generation). Record status codes and payload snippets.
4. **Validate IBM outputs.** Trigger at least one report or job that touches IBM i resources. Confirm spool files appear in the expected queues/libraries.
5. **Review monitoring/alerts.** Ensure logging, metrics, and alert rules are pointing to the correct environment (e.g., UAT-specific Log Analytics workspace).
6. **Collect tester readiness info.** Share URLs, credentials, and known limitations with the UAT testers so they can start immediately.

## Sign-off
- Document the deployment timestamp, build SHA, and who performed the release.
- Obtain written approval from QA/business before promoting further.
