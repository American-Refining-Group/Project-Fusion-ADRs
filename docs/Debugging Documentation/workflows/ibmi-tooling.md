# IBM Navigator, ACS, Data Studio, and 5250 Toolkit

Use this playbook when an incident requires **IBM-side validation** in addition to the VS Code debugging workflow.

## 1. Before you leave VS Code

1. Capture the **job metadata** (job name/number/user, output queue, report type) from the failed API call or queue job using `SpooledMetaDataReportEntity` and `spooledReportMetaData`. 【F:src/main/account-payable/domain/entities/spooled-meta-data-report.entity.ts†L1-L17】【F:src/shared/config/generate-report-config.ts†L57-L127】
2. Note the **Sequelize model + schema** involved from the stack trace (for example, lines inside `schema.ts`). You will use these names inside IBM Data Studio. 【F:src/main/account-payable/data/schemas/schema.ts†L2296-L2339】
3. Export the **exact DSN** and user from `connection.ts` so IBM Navigator queries the same library list (`env-library-config.ts`). 【F:src/shared/infrastructure/connection.ts†L71-L233】【F:src/shared/config/env-library-config.ts†L4-L83】

## 2. IBM Navigator for i (Web)

1. Open **Work Management → Active Jobs** and filter by the captured `jobName`/`jobNumber`.
2. Drill into the job log (`Messages` tab) and compare CPF / SQL codes with the Nest log output; copy the entries into the incident ticket.
3. Open **Output Queues** and jump directly to `outputQueueLibrary/outputQueueName`. Confirm the spool file exists, matches the timestamp, and note whether it is `HLD`, `RDY`, or `SAV`.
4. If the spool file is missing, confirm whether the RPG program wrote to the default queue defined in `spooledReportMetaData` or if overrides were sent with the API request payload.

## 3. IBM i Access Client Solutions (ACS)

1. Launch ACS and connect to the same system/DSN as the API container.
2. Use **Run SQL Scripts** to run sanity queries with the schema/table names recorded earlier:
   ```sql
   SELECT MEJOBNM, MEOQNM, MEOQLB, MERPTP, LASTUPDATED
   FROM LIBRARY.SPOOLEDMETADATAREPORT
   WHERE MERPTP = 'APREPORT'
   ORDER BY LASTUPDATED DESC FETCH FIRST 20 ROWS ONLY;
   ```
   Replace `LIBRARY` with the active schema derived from `env-library-config.ts`.
3. Use **System Status** to ensure the relevant subsystems (e.g., `QHTTPSVR`, `QBATCH`) are not constrained.
4. Export spool files or job logs as PDF/HTML when attaching evidence to tickets.

## 4. IBM Data Studio

1. Create a **connection profile** pointing to the DSN from `odbc.ini` / `connection.ts`.
2. Import the **Sequelize model DDL** you are validating (for example, `SpooledMetaDataReportsRepository` queries). 【F:src/main/account-payable/data/repositories/spooled-meta-data-reports.repository.ts†L1-L107】
3. Use the **Data Output** tab to validate record counts, nullability, and any derived columns (like the formatted paths from `reportsFormatter`). 【F:src/shared/formatters/dropdown.formatter.ts†L1-L66】
4. Save SQL snippets that recreate the failing condition so another engineer can replay them without the app.

## 5. 5250 / PDM session

1. Start a 5250 session (ACS ▶ 5250 Emulator) with the same profile.
2. Run `WRKACTJOB SBS(QBATCH)` and locate the job number captured earlier to inspect it interactively.
3. Use `DSPJOBLOG` and `WRKOUTQ` with the `jobNumber/jobUser` to inspect the RPG messages.
4. For source-level investigations, open `STRPDM` → option 2 (Work with Members) and navigate to the library that owns the RPG program under test. Verify timestamps match the deployment you expect.
5. If you need to recompile, coordinate with the RPG owner and make sure VS Code’s evidence (DTO payloads, SQL statements) is attached to the service request.

## 6. Closing the loop back in VS Code

1. Record the IBM findings in your incident doc and cross-link them with the related source files (`flexi.processor.ts`, `connection.ts`, etc.).
2. Update/extend the relevant error playbook (for example, `docs/errors/504-gateway-timeouts.md`) with the new failure mode if it was not already documented.
3. Re-run the VS Code debugger session using any IBM-side configuration tweaks (library list, user profile) that surfaced during the investigation to ensure the fix is reproducible.
