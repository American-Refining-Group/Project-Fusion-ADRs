# Code Review – RPG

This checklist keeps RPG reviewers aligned with the Node-facing contracts.

## Prep
- Gather the RPG source or CL scripts under review plus any IBM project references.
- Open `src/modules/spooled-metadata-report/entities/spooled-metadata-report.entity.ts` to see the expected fields and data types.
- Keep `src/modules/spooled-metadata-report/schema.ts` handy for column names and lengths.

## Steps
1. **Validate inputs/outputs.** Confirm the RPG program reads and writes every field required by the entity (job name, queue, report type, etc.). Note any missing assignments.
2. **Check library bindings.** Ensure the program references the libraries resolved by `DynamicLibraryManager` or `SETENVPRC`. If a different library is required, document the override and reason.
3. **Review error handling.** Confirm the program returns clear status codes or messages via `errmsg_out` so Node logs show meaningful failures.
4. **Inspect queue/report usage.** Compare the output queue names and report defaults with `generate-report-config.ts`. Flag discrepancies so downstream processes keep working.
5. **Examine performance/locking.** Look for unnecessary exclusive locks, missing commitment control, or loops that could cause long-running jobs.
6. **Confirm compile instructions.** Note the exact commands (CRTBNDRPG, CRTRPGMOD, etc.), required options, and any post-compile steps so build engineers can reproduce.

## Sign-off
- Document review findings in the ticket and mention any required IBM promotions.
- Approve only after compile instructions and error handling expectations are clear.
