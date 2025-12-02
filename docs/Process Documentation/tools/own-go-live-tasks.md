# Own Go-Live Tasks – Tool Playbook

Coordinate release weekend activities across IDEs and IBM i consoles.

## Core Tools
- **VS Code** – run deployment scripts, edit `.env` overrides, and capture release notes.
- **Terminal/Docker in VS Code** – execute `deploy.sh` or Azure DevOps pipeline steps manually.
- **IBM Navigator for i** – stop/start subsystems, view QSYSOPR messages, and monitor job queues.
- **5250 emulator** – run CL commands (`ENDSBS`, `STRSBS`, `SBMJOB`) when Navigator access is constrained.
- **IBM Data Studio** – run migration SQL and verify post-go-live data sanity checks.

## Guided Steps
1. **Review the runbook in VS Code.** Open `docs/delivery/own-go-live-tasks.md` and annotate the timeline with assignees.
2. **Prepare environment switches.** Use VS Code to edit `.env.production` or pipeline variables, committing changes if required.
3. **Execute application deployment.** Run `deploy.sh` or the relevant CI task inside VS Code’s terminal; watch for failures and rerun if necessary.
4. **Flip IBM i jobs.** Via Navigator or 5250, stop the legacy subsystem, deploy new programs, then `STRSBS` or `SBMJOB` to resume operations.
5. **Run validation SQL.** In Data Studio, execute the go-live verification queries (row counts, default flags) and store outputs.
6. **Log status.** Update the go-live tracker in VS Code/DevOps with timestamps, Navigator screenshots, and SQL results.

## Evidence to Capture
- Terminal output from deployment scripts.
- Navigator or 5250 logs showing subsystem restarts.
- Data Studio query exports for post-deploy validation.
