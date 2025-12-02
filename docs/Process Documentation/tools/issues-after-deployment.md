# Issues After Deployment – Tool Playbook

Use these diagnostics to triage regressions quickly.

## Core Tools
- **VS Code** – tail application logs via SSH, run `kubectl logs` (if applicable), and inspect recent commits.
- **VS Code Debug Console** – attach to local reproductions or run targeted scripts.
- **IBM Navigator for i** – review QSYSOPR, job logs, and message queues tied to the failing process.
- **5250 emulator** – query message queues (`WRKMSGF`, `DSPMSG`), cancel jobs, or rerun CL utilities.
- **IBM Data Studio** – run point-in-time queries to compare pre/post data states.

## Guided Steps
1. **Capture context in VS Code.** Pull the latest tag, review the diff for the component at fault, and open log streaming tasks (`tail -f logs/app.log`).
2. **Inspect IBM i alerts.** In Navigator, open *Basic Operations → Messages* and download relevant QSYSOPR/QSYSMSG entries; cross-check from a 5250 session if Navigator is lagging.
3. **Check jobs/subsystems.** Use Navigator or `WRKACTJOB` to identify looping/failed jobs. Drill into job logs (`Option 10`) and export them.
4. **Validate data.** From Data Studio, run targeted queries to confirm whether tables were partially updated or locked.
5. **Reproduce locally.** Back in VS Code, run the failing API or script with prod-like `.env` overrides to replicate the issue.
6. **Document RCA inputs.** Store links to Navigator logs, SQL results, and VS Code repro steps inside the `docs/delivery/issues-after-deployment.md` checklist.

## Evidence to Capture
- Log excerpts showing timestamps of failures.
- Joblog PDFs from Navigator/5250.
- SQL query outputs demonstrating data corruption or confirming integrity.
