# Example – Transfer RPG Programs to Power 10

## Scenario Snapshot
- **Goal:** migrate `AP200R` and `APJRN` objects from the Power 8 DEV partition to the new Power 10 TEST partition.
- **Inputs:** SAVOBJ library save, FTP/SFTP transfer, RSTOBJ on target, validation queries.
- **Output:** confirmation that binaries exist on Power 10 with matching levels.

## Step-by-Step Walkthrough
1. **Prep save files.** On Power 8, run `SAVOBJ OBJ(AP200R APJRN) LIB(DEVLIB) DEV(*SAVF) SAVF(DEVLIB/APMIG)`.
2. **Transfer save files.** Use FTP or Navigator "Download Save File" to move `APMIG` to your workstation, then upload to Power 10 `QGPL/APMIG`.
3. **Restore objects.** On Power 10, execute `RSTOBJ OBJ(AP200R APJRN) SAVLIB(DEVLIB) DEV(*SAVF) SAVF(QGPL/APMIG) RSTLIB(TESTLIB)`.
4. **Verify authorities.** Run `DSPOBJAUT TESTLIB/AP200R` to confirm profiles match the original security matrix.
5. **Smoke test.** Call each program with a harmless parameter set (`CALL TESTLIB/AP200R PARM('CHECKONLY')`) and ensure job logs show success.
6. **Document transfer.** Update the migration tracker with save/restore job numbers, timestamps, and checksum values if used.

## Evidence Package
- SAVOBJ/RSTOBJ job logs exported as PDF.
- Screenshot of `WRKOBJ` on Power 10 proving objects exist in TESTLIB.
- Migration tracker entry referencing checksum or object level numbers.
