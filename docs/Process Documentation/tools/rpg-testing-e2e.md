# RPG Testing End-to-End – Tool Playbook

Use 5250 tooling plus VS Code and Navigator observers to execute a full workflow.

## Core Tools
- **5250 emulator** – run CL commands, submit batch jobs, and view interactive screens.
- **PDM/SEU** – tweak test data copybooks or driver programs.
- **IBM Navigator for i** – monitor submitted jobs (`WRKACTJOB` equivalent) and capture job logs/spool files.
- **VS Code / REST client** – trigger API calls if Node.js endpoints are part of the flow.
- **IBM Data Studio** – verify DB2 tables before/after the RPG process runs.

## Guided Steps
1. **Reset data state.** Run Data Studio scripts or `RUNSQLSTM` (via 5250) to seed prerequisite records.
2. **Launch the driver program.** In 5250, start the transaction (e.g., `CALL PGM(LIB/PGM)` or submit the batch job) and record the job name.
3. **Monitor execution in Navigator.** Watch the job under *Active Jobs*; if it spawns child jobs, open each joblog and export messages.
4. **Verify downstream API effects.** If the RPG routine feeds the Node API, issue the relevant request from VS Code’s REST client or Thunder Client and compare payloads against expected values.
5. **Validate tables.** Use Data Studio to run `SELECT` queries before and after execution, ensuring delta counts match requirements.
6. **Document pass/fail.** Update the `docs/delivery/rpg-testing-e2e.md` checklist with job names, run timestamps, and SQL result summaries.

## Evidence to Capture
- Data Studio screenshots showing before/after row counts.
- Navigator joblog or spool download for the RPG run.
- REST client output verifying API parity when applicable.
