# Debugging Node ↔ RPG ↔ IBM i Integrations

This guide documents the moving pieces in this repository and outlines how to diagnose failures anywhere along the NestJS → Bull/BullMQ queue → RPG programs → IBM i spooled output flow.

## 1. Node.js Framework Map

| Concern | Key Modules / Packages | Debugging Tips |
| --- | --- | --- |
| **HTTP & DI framework** | `AppModule` wires the NestJS modules for static file serving, config, caching, auth, account-payable flows, Bull, and the websocket gateway. 【F:src/app.module.ts†L1-L35】 | Run `npm run start:debug` to launch the Nest dev server with an inspector, then set breakpoints inside the relevant module providers. |
| **HTTP platform** | Nest boots on `@nestjs/platform-express`, and the app serves static assets via `@nestjs/serve-static`. Socket connections use `@nestjs/platform-socket.io`. All platform adapters are declared in `package.json`. 【F:package.json†L8-L78】 | If HTTP requests stall, confirm the correct platform adapter is installed (`node_modules/@nestjs/platform-express`) and that static roots resolve to real paths via `ServeStaticModule`. |
| **Queues** | Bull (`@nestjs/bull`, `bull`) handles classic queues; BullMQ (`bullmq`) is used for streaming uploads. Redis connection info lives in `src/shared/queue/bullmq-connection.ts`. 【F:package.json†L8-L78】【F:src/shared/queue/bullmq-connection.ts†L1-L4】 | Failed jobs show up in processor hooks such as `onFailed` inside each `@Processor` class. Attach a debugger to those hooks or query BullMQ using `bull-board`/`redis-cli` for stuck jobs. |
| **ORM & IBM driver** | Sequelize (`sequelize`, `sequelize-typescript`) is paired with the DB2 for i dialect (`@sequelize/db2-ibmi`) and the native ODBC binding (`odbc`). 【F:package.json†L39-L78】 | Enable verbose SQL logging by running in non-prod (`NODE_ENV !== 'prod'`) and tailing the `Database` logger output from `customSequelizeLogger`. |
| **Websocket fan-out** | Websocket gateway/service broadcast upload summaries to browser clients. 【F:src/shared/websocket/websocket.service.ts†L1-L118】 | If browsers miss events, confirm the gateway is emitting (`emitEvent`, `emitToUser`) and that Redis pub/sub (if enabled) is reachable. |
| **Observability** | `AppLogger` wraps Nest’s logger with timing helpers for slow operations. 【F:src/shared/logger/logger.service.ts†L1-L69】 | Call `logger.timing()`/`sharedTiming()` around long-running sections to surface delays in development logs.

## 2. IBM i Connectivity & Driver Layer

1. **Container prerequisites** – The Dockerfile installs unixODBC + IBM i Access ODBC driver, copies `odbc.ini`, then builds the Nest app. Missing packages or an unreadable `odbc.ini` are the first things to check when DSN resolution fails. 【F:Dockerfile†L1-L49】
2. **DSN / ODBC configuration** – The project checks an `odbc.ini` file (copied into `/etc/odbc.ini`) that contains a baseline `AS400DB` definition, including the IBM i system IP, default package library, CCSID translation and tracing flags. 【F:src/shared/infrastructure/odbc/odbc.ini†L1-L27】
3. **Runtime connection builder** – `buildDsn()` in `connection.ts` composes the IBM i Access connection string from `DB_SYSTEM`, `DB_USERNAME`, `DB_PASSWORD`, and package overrides. Logging is enabled by default when `NODE_ENV !== 'prod'`. 【F:src/shared/infrastructure/connection.ts†L71-L134】
4. **Pool and retry tuning** – `connectAs400Db2IBMi()` configures Sequelize with explicit pool sizes, benchmark logging, and retry counts, then races authentication against a 15s timeout so hangs surface quickly. 【F:src/shared/infrastructure/connection.ts†L194-L233】
5. **Dynamic library resolution** – After connecting, the app always runs the `SETENVPRC` stored procedure via `DynamicLibraryManager`. It dynamically creates `QGPL` variables, calls the SP, trims outputs (data/work/sp libraries plus error message), updates process env vars, and drops the temp variables. Failures surface in the `DynamicLibraryManager` logger, and you can re-run the SP via `dynamicLibManager.setDynamicLibraries()` in a REPL. 【F:src/main/global-states/data/stored-procedure/dynamic-library-manager.ts†L1-L213】【F:src/shared/utils/db.utils.ts†L1-L79】
6. **Library map per environment** – `env-library-config.ts` keeps the default schema per environment. If SQL starts referencing the wrong library, confirm `NODE_ENV` and the derived `DB_*` env vars match what `DynamicLibraryManager.updateEnvironmentVariables()` prints. 【F:src/shared/config/env-library-config.ts†L4-L83】

### Diagnosing connection errors

| Symptom | Checklist |
| --- | --- |
| `Database authentication timeout` or `Failed to connect to the database` | Ensure the Docker image picked up the IBM driver, verify `DB_SYSTEM/DB_USERNAME/DB_PASSWORD` in the running container, and re-run `connectAs400Db2IBMi()` to regenerate logs from `customSequelizeLogger`. 【F:src/shared/infrastructure/connection.ts†L194-L257】 |
| `SETENVPRC stored procedure failed` | Query `QSYS2.SYSROUTINES` to confirm the SP exists (the helper already logs a warning/fallback). Check that the current user has authority to `CREATE VARIABLE` in QGPL because the manager drops/recreates them each call. 【F:src/main/global-states/data/stored-procedure/dynamic-library-manager.ts†L40-L123】 |
| Models missing tables/columns | `initializeModels()` enumerates every table that must be initialized after connection. If a model fails, it will throw before `isModelsInitialized` flips. Inspect the preceding log statements to see which initializer last ran. 【F:src/shared/infrastructure/connection.ts†L136-L191】 |

## 3. SQL Diagnostics & Model Issues

1. **Custom SQL logging** – With `NODE_ENV !== 'prod'`, `customSequelizeLogger` emits the full SQL statement and timing at `warn` level, making it easy to correlate with IBM job logs. 【F:src/shared/infrastructure/connection.ts†L117-L134】
2. **Dynamic tables** – Utility helpers in `db.utils.ts` build schema-qualified names, guard against duplicate registration, and drop temp variables. When diagnosing incorrect table names, inspect `buildTableName()`/`initializeModelWithSequelize()` to see how prefixes/suffixes are derived. 【F:src/shared/utils/db.utils.ts†L1-L79】
3. **Library-specific failures** – Compare the resolved schema from `getTableAndSchemaInfo()` / `getDynamicTableAndSchemaInfo()` with the job log library list to confirm you are hitting the intended environment libraries. 【F:src/shared/config/env-library-config.ts†L30-L83】

## 4. Bull/BullMQ Job Pipelines

* **Queue wiring** – Each processor (Flexi uploads, SOGAS, Clear Checks, etc.) is decorated with `@Processor(QUEUE_NAMES.X)` and exposes `onActive`, `onCompleted`, and `onFailed` hooks that log the Bull job lifecycle. For example, `FlexiProcessor` logs every transition, binds user context, limits parallelism, and records batch summaries back into Redis. 【F:src/main/account-payable/application/voucher/shared-services/flexi.processor.ts†L1-L175】
* **Redis connection** – `redis_connection` simply reads `REDIS_HOST`/`REDIS_PORT`. If every job immediately fails with `ECONNREFUSED`, inspect those env vars and confirm Redis is reachable. 【F:src/shared/queue/bullmq-connection.ts†L1-L4】
* **Websocket fan-out** – Job processors push interim/final summaries through `WebsocketService`, which calculates per-status totals and emits user-scoped events (`emitUploadStatusToUser`). Missing UI updates usually mean the `UserContext` lacks initials or the gateway is not subscribed. 【F:src/shared/websocket/websocket.service.ts†L1-L118】
* **Debug strategy** – Attach a debugger to the `handle()` method of the relevant processor, replay the job data (from Redis or API logs), and step through `validateHeader`, `saveVoucherData`, and the DB writes. Because the processors log invoice numbers, you can cross-reference IBM joblogs for the same invoice ID.

## 5. Spooled File & Output Queue Diagnostics

1. **Metadata captured per report** – The `SpooledMetaDataReportEntity` stores spool/PDF filenames, report type, job identifiers, output queue name/library, file path, creation timestamp, form type, and any RPG error string. This lets you correlate Node errors with IBM job output queue entries. 【F:src/main/account-payable/domain/entities/spooled-meta-data-report.entity.ts†L1-L17】
2. **Schema mapping** – `SpooledMetadataReportModelSchema` maps each property to its IBM i physical file column (`MEJOBNM`, `MEOQNM`, etc.). If fields are blank in the API, inspect this mapping to verify the RPG program actually populated the column. 【F:src/main/account-payable/data/schemas/schema.ts†L2296-L2339】
3. **Default spool config** – `spooledReportMetaData` defines default spool file name (`APREPORT`), job user (`QUSER`), and output queue/library (`DAMCOOUTQ` / `QUSRSYS`) used when generating files (e.g., ACH, IRS, 1099 reports). Override these values when testing non-production queues. 【F:src/shared/config/generate-report-config.ts†L57-L127】
4. **Repository queries** – `SpooledMetaDataReportsRepository` exposes `SpooledMetadataReports()` to paginate by `reportType`, wildcard filename, and date range; it also formats the timestamps/file paths for the UI. Use this repository to confirm whether the IBM job created entries and whether `reportsFormatter` built a valid download path. 【F:src/main/account-payable/data/repositories/spooled-meta-data-reports.repository.ts†L1-L107】【F:src/shared/formatters/dropdown.formatter.ts†L1-L66】
5. **Job correlation fields** – Because the schema persists `jobName`, `jobNumber`, `jobUser`, `jobSystemName`, and `error`, you can filter the metadata table (via SQL or the repository) for a specific failing IBM job, then jump directly to its output queue entry using the stored `outputQueueName`/`outputQueueLibrary`.

### Debugging spool/output queue issues

| Issue | How to investigate |
| --- | --- |
| Report exists in RPG queue but not in API | Query the `SpooledMetadataReport` table for the `reportType` and timestamp window using the repository’s `where` construction to ensure the filters align with how the UI queries. 【F:src/main/account-payable/data/repositories/spooled-meta-data-reports.repository.ts†L24-L94】 |
| PDF download 404s | `reportsFormatter` rewrites `filePath` using `buildFilePath(row.pdfFileName.trim())`. Verify the PDF actually exists beneath the static root configured in `ServeStaticModule` and that `filePath` aligns with the SMB/NFS mount. 【F:src/shared/formatters/dropdown.formatter.ts†L1-L66】【F:src/app.module.ts†L15-L35】 |
| Spool shows wrong queue/library | Confirm the defaults from `spooledReportMetaData` were overridden (if needed) before calling `createReports()`. When debugging, log the payload passed to `SpooledMetaDataReportInterface.createReports()` so you can match the queue metadata with IBM `WRKSPLF` output. 【F:src/shared/config/generate-report-config.ts†L57-L127】【F:src/main/account-payable/data/repositories/spooled-meta-data-reports.repository.ts†L97-L107】 |

## 6. End-to-End Checklist for Node ↔ RPG ↔ IBM i Bugs

1. **API entry point** – Confirm the relevant Nest controller/use case received the expected DTO (enable class-validator errors). Use the inspector or HTTP logs to ensure the request payload matches the RPG program’s expectations.
2. **Queue ingestion** – Make sure the job was enqueued in Bull/BullMQ (watch Redis). If it failed immediately, inspect the processor’s `onFailed()` logs for stack traces. 【F:src/main/account-payable/application/voucher/shared-services/flexi.processor.ts†L55-L111】
3. **Sequelize transaction** – Track SQL logs for the job window. If a query fails, `customSequelizeLogger` will show the statement right before the error; match it against IBM joblog statements. 【F:src/shared/infrastructure/connection.ts†L117-L134】
4. **RPG program execution** – Use the stored job metadata (`jobName`, `jobNumber`, etc.) from the spooled metadata entries to open the IBM joblog (`DSPJOBLOG`) and the output queue (`WRKOUTQ`). 【F:src/main/account-payable/domain/entities/spooled-meta-data-report.entity.ts†L1-L17】
5. **Spooled output verification** – Compare the expected spool file name / form type against `spooledReportMetaData` defaults or overrides for the use case you are testing. 【F:src/shared/config/generate-report-config.ts†L57-L127】
6. **Front-end surface** – Ensure the formatted report list returned by `reportsFormatter` uses the same timezone/format as the UI filters; mismatches can hide real reports even when IBM generated them. 【F:src/shared/formatters/dropdown.formatter.ts†L1-L66】

Following these steps gives you observability from the Nest request layer, through the queue processors, into the IBM DB2/ODBC layer, all the way to the IBM output queue that hosts the RPG-generated spooled files. When you need a click-by-click workflow for VS Code, IBM Navigator, ACS, or Data Studio, open the [debugging workflows](debugging/workflows/README.md) directory for ready-to-run checklists that build on this reference.
