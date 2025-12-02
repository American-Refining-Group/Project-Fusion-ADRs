# Example – RPG Testing End-to-End

## Scenario Snapshot
- **Goal:** validate that voucher postings triggered by `/vouchers` API create balanced journal entries through RPG batch job `APJRN`.
- **Inputs:** Node API, RPG program, DB2 tables (`APTRANH`, `APTRAND`), spool files.
- **Output:** signed test log showing API → RPG → DB2 consistency.

## Step-by-Step Walkthrough
1. **Seed data.** Use Data Studio to insert a test voucher header/detail row or run the NestJS endpoint to create it via `AddVoucherEntryUseCase`.
2. **Kick off RPG batch.** Submit `SBMJOB CMD(CALL PGM(TESTLIB/APJRN) PARM('TESTVCHR'))` so the IBM job processes the voucher.
3. **Monitor job.** Watch `WRKACTJOB` or IBM Navigator to ensure the job completes without MSGW status; capture the job log if warnings occur.
4. **Verify DB2 impact.** Query `APTRANH` and `APTRAND` to confirm the voucher status changed and balancing amounts match the API payload.
5. **Check spooled output.** Download the balancing report spool file (e.g., `APJRNLOG`) as PDF and compare totals to the API response.
6. **Record results.** Update the E2E test case in Azure Test Plans with pass/fail, attach the spool PDF, and mention the API request ID.

## Evidence Package
- API request/response log from NestJS (include correlation ID).
- DB2 query screenshot proving status transitions.
- Spool file PDF showing balanced totals.
