# Transfer RPG Programs to Power 10 – Tool Playbook

Coordinate tooling between source and target IBM i partitions.

## Core Tools
- **5250 emulator** – package objects (`SAVOBJ`, `SAVLIB`) and restore them on the Power 10 LPAR.
- **IBM Navigator for i** – manage network file shares (IFS) and monitor save/restore jobs.
- **VS Code / Code for IBM i** – push source members via Git to the Power 10 repo workspace.
- **IBM Data Studio** – confirm Power 10 DB2 catalogs contain required tables before moving programs.

## Guided Steps
1. **Verify prerequisites.** In Data Studio, connect to the Power 10 DB2 catalog and run sanity queries (`SELECT COUNT(*) FROM <table>`) to ensure dependencies already exist.
2. **Package the objects on the source LPAR.** Using 5250, run `SAVOBJ` or `SAVLIB` to a save file in QGPL (or a transport library). Note the save file name.
3. **Move the save file.** From Navigator, download the save file or copy it through an IFS share/SFTP to the Power 10 system.
4. **Restore on Power 10.** In a Power 10 5250 session, run `RSTOBJ`/`RSTLIB`, pointing to the transferred save file. Monitor job progress in Navigator.
5. **Sync source members.** Use VS Code’s Code for IBM i extension (or Git) to push updated source into the Power 10 source library for long-term maintenance.
6. **Validate objects.** Run `DSPPGM` or `WRKOBJ` on Power 10 to confirm attributes (owner, activation group) and document the results in the transfer checklist.

## Evidence to Capture
- Data Studio output verifying target tables.
- `SAVOBJ`/`RSTOBJ` command snippets and joblog exports.
- VS Code screenshot of the Code for IBM i deploy log (if used).
