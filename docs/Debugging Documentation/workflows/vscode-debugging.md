# VS Code ↔ NestJS ↔ IBM i Bridge

Follow this checklist whenever you need to debug a backend defect from **VS Code** all the way to **DB2/RPG artifacts**.

## 1. Prep VS Code for this repo

1. **Install extensions**
   - _ESLint_ + _Prettier_ so files match the project linters (`eslint.config.mjs`, `prettier`).
   - _NestJS Snippets_ and _IBM i (Code for IBM i)_ to jump between controllers, queues, and IBM job streams.
2. **Clone + install** – open the repo folder and run `yarn` once so VS Code can index the TypeScript sources referenced from `tsconfig.json`.
3. **Trust the workspace** so VS Code can read `.vscode/launch.json` (if checked in) or let you create a new one.

## 2. Attach to the API from VS Code

1. **Start the inspector-friendly Nest process**:
   ```bash
   npm run start:debug
   ```
   This wraps `nest start --debug 0.0.0.0:9229 --watch` per `package.json`, so file changes automatically reload. 【F:package.json†L10-L24】
2. **Create a `launch.json`** (Run and Debug ▶ `create a launch.json`) with:
   ```json
   {
     "type": "node",
     "request": "attach",
     "name": "NestJS: Attach",
     "port": 9229,
     "restart": true,
     "protocol": "inspector",
     "skipFiles": ["<node_internals>/**"]
   }
   ```
3. **Set breakpoints** inside controllers/use cases (for example `src/main/account-payable/application/voucher/shared-services/flexi.processor.ts`) and in the DB driver glue (`src/shared/infrastructure/connection.ts`). 【F:src/main/account-payable/application/voucher/shared-services/flexi.processor.ts†L1-L175】【F:src/shared/infrastructure/connection.ts†L117-L257】
4. **Trigger the failing request** via Thunder Client/REST Client or by replaying the upstream queue message. When execution pauses, inspect the job payload and confirm the DB schema/libraries match expectations from `env-library-config.ts`. 【F:src/shared/config/env-library-config.ts†L4-L83】

## 3. Debug queue workers from VS Code

1. **Run the local stack**:
   ```bash
   npm run docker:dev
   ```
   This brings up the API container plus the Bull/BullMQ workers defined in `docker-compose.yml`. 【F:package.json†L25-L33】【F:docker-compose.yml†L1-L80】
2. **Attach the debugger** to the worker container by forwarding port `9229` (add `NODE_OPTIONS=--inspect=0.0.0.0:9229` to the worker service or run it locally with `npm run start:debug`).
3. **Use watch expressions** for `job.data`, `job.attemptsMade`, and DB transaction state so you can correlate each retry with IBM job logs.
4. **Inspect Redis** with the Bull UI of your choice or `redis-cli` to confirm the job is not stuck in `waiting` after your code resumes.

## 4. Debug DB2/IBM i calls without leaving VS Code

1. **Search for SQL models quickly** – use `Go to Symbol` (`Ctrl+T`) to jump to the Sequelize model named in the error. The schema and table name mappings live alongside each model (for example, `src/main/account-payable/data/schemas/schema.ts`). 【F:src/main/account-payable/data/schemas/schema.ts†L2296-L2339】
2. **Open the DSN builder** – peek at `src/shared/infrastructure/connection.ts` while paused to see the exact DSN string and retry config your code is using.
3. **Run targeted tests** – configure another `launch.json` entry:
   ```json
   {
     "type": "node",
     "request": "launch",
     "name": "Jest: Debug current file",
     "program": "${workspaceFolder}/node_modules/.bin/jest",
     "args": ["${relativeFile}", "--runInBand"],
     "cwd": "${workspaceFolder}",
     "runtimeArgs": ["--inspect-brk"],
     "console": "integratedTerminal"
   }
   ```
   Then press `F5` inside a failing spec (for example, `add-voucher-entry.use-case.spec.ts`) to replay the business logic without starting the HTTP server.

## 5. Capture IBM evidence from VS Code

1. **Copy the job metadata** – the `SpooledMetaDataReportEntity` exposes `jobName`, `jobNumber`, `jobUser`, and `outputQueue` so you can paste them into IBM Navigator or ACS without re-running the API. 【F:src/main/account-payable/domain/entities/spooled-meta-data-report.entity.ts†L1-L17】
2. **Track spool defaults** – `spooledReportMetaData` defines which queue/library the RPG job will target; keep the file open side-by-side with your debugger to confirm overrides. 【F:src/shared/config/generate-report-config.ts†L57-L127】
3. **Log evidence inline** – use VS Code’s `Add Logpoint…` feature to emit structured JSON (job ID, invoice number, SQL state) without stopping the process. Share those snippets in incident tickets alongside IBM screenshots.

## 6. Shut down cleanly

1. Stop debugging (Shift+F5) so Nest restarts in watch mode without a debugger.
2. `Ctrl+C` the terminal running `npm run start:debug` or `npm run docker:dev`.
3. Run `npm run dev:logs` if you need to preserve the docker-compose output for root-cause analysis. 【F:package.json†L16-L32】
