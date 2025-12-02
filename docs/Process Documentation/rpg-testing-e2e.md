# RPG Testing End-to-End

Follow this checklist to validate an RPG flow from API call through spool file creation.

## Prep
- Review `docs/DEBUGGING.md`, especially the Node → Bull → RPG → spool verification section.
- Confirm access to the target IBM environment, job logs, and output queues.
- Prepare the REST client or automated script that will trigger the RPG job (Postman, Jest, curl).

## Steps
1. **Trigger the scenario.** Call the appropriate API or queue event and record the request payload, time, and environment.
2. **Monitor Node processing.** Tail the service logs (`yarn start:dev`, `kubectl logs`, etc.) to ensure the request entered the correct queue and processor.
3. **Confirm RPG execution.** Capture the job name, number, and user from IBM job logs. Note any messages or warnings.
4. **Verify database writes.** Query the repositories (e.g., `SpooledMetaDataReportsRepository`) or DB2 tables to confirm rows were inserted/updated.
5. **Check spool/output queues.** Locate the generated reports, download them if needed, and compare to expected content/format.
6. **Document variances.** Record any mismatched fields, missing files, or timing issues plus the log excerpts that prove the problem.
7. **Repeat for edge cases.** Run negative and boundary scenarios listed in the acceptance criteria.

## Sign-off
- Attach screenshots or text dumps of job logs, spool files, and API responses.
- Record who performed the test, the date, and the environment.
