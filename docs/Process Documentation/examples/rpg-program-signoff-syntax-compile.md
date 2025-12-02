# Example – RPG Program Signoff (Syntax/Compile)

## Scenario Snapshot
- **Goal:** certify that RPGLE member `APINVR` compiles cleanly before promoting to TEST.
- **Inputs:** Source member, compile command `CRTBNDRPG`, joblog, comparison with previous module level.
- **Output:** signoff note referencing compile logs and object versions.

## Step-by-Step Walkthrough
1. **Load source.** Use PDM or Code for IBM i to edit `APINVR` in `DEVLIB/QRPGLESRC` and ensure comments note the change ticket.
2. **Run compile.** Submit `SBMJOB CMD(CRTBNDRPG PGM(TESTLIB/APINVR) SRCFILE(DEVLIB/QRPGLESRC)) LOG(*JOBLOGSVR)`.
3. **Review joblog.** In IBM Navigator, check the submitted job for any RNF/RPG messages; copy them into OneNote even if informational.
4. **Capture listing.** Save the spool file `QSYSPRT` from the compile job, export as PDF, and attach to the story.
5. **Verify object level.** Use `DSPOBJD OBJ(TESTLIB/APINVR) DETAIL(*SERVICE)` to confirm the creation timestamp and compiler level.
6. **Post signoff.** Update the release checklist with compile date/time, job number, and spool file path.

## Evidence Package
- PDF of the compile listing with zero severity > 20 errors.
- Screenshot of IBM Navigator joblog filtered to the compile job.
- Release checklist entry referencing object level/time stamp.
