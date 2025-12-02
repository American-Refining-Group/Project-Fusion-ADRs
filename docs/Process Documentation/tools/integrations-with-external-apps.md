# Integrations with External Apps – Tool Walkthrough

Follow these VS Code, IBM i, and Navigator actions to validate each integration point end-to-end.

## Core Tools
- **VS Code** – inspect NestJS integration modules, `.env` examples, and queue processors.
- **Docker Compose terminal / VS Code tasks** – spin up stubs or mock servers locally.
- **IBM Navigator for i** – confirm remote database or MQ credentials stored in system values or service entries.
- **IBM Data Studio** – verify DB2 views or staging tables that feed external systems.
- **5250 emulator** – run CL commands (`WRKSRVTBLE`, `WRKENVVAR`) to confirm IBM i-side configuration.

## Guided Steps
1. **Open the integration module in VS Code.** Use the file Explorer to review the NestJS provider, HTTP client, and DTO definitions. Note required secrets.
2. **Start local dependencies.** Launch VS Code’s integrated terminal to run `docker-compose up integration-mocks` or the service-specific mock script so you can replay callbacks.
3. **Validate IBM i credentials.** In Navigator, open *Configuration and Service → Service Entries* to verify the user profile, endpoint, and certificate assigned to the integration. Export the entry details if updates are needed.
4. **Check DB2 staging artifacts with Data Studio.** Run diagnostics queries to ensure data is landing in the expected staging tables or views that the integration consumes.
5. **Confirm IBM i environment variables via 5250.** Execute `WRKENVVAR LEVEL(*SYS)` or `WRKSRVTBLE` to verify keys that should align with the `.env` entries used in VS Code.
6. **Exercise the flow.** From VS Code, issue the relevant API call (via Thunder Client/REST Client) and monitor both the NestJS logs and Navigator job logs for errors.

## Evidence to Capture
- Screenshot of Navigator service entry parameters.
- CLI output from `WRKENVVAR` showing matching keys.
- Local log excerpt proving the mock/external call succeeded.
