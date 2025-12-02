# Example – Approve QA Test Cases from Acceptance Criteria

## Scenario Snapshot
- **Goal:** approve QA regression tests for "AP Vendor Hold" story before UAT.
- **Inputs:** Acceptance criteria from Azure Boards, test cases drafted in Azure Test Plans, API contract for `/vendors/:id/holds`, IBM i job notes for `APSGACH` table.
- **Output:** QA suite marked "Ready" with explicit traceability to criteria and data sources.

## Step-by-Step Walkthrough
1. **Extract acceptance criteria.** Paste each bullet into a checklist so every QA case maps to a single behavior.
2. **Review drafted tests.** In Test Plans, ensure there is at least one API test (via Postman or automation) and one IBM i validation (verifying the hold row in `dataLib.APSGACH`).
3. **Validate data prep.** Confirm QA provided seed scripts or IBM Navigator instructions to create the vendor states needed for the tests.
4. **Check automation hooks.** Ensure the Jest/E2E folder contains the updated spec (e.g., `vendor-hold.e2e-spec.ts`) and that it calls the NestJS endpoint with sample data.
5. **Provide feedback.** Comment directly in Azure Test Plans or Teams, flagging any missing negative scenario (e.g., hold removal without authorization).
6. **Approve suite.** Once gaps close, change the Test Suite status to "Ready", note the acceptance criteria numbers it covers, and tag the QA lead.

## Evidence Package
- Screenshot of Azure Test Plans suite showing status = Ready.
- Link to the Jest E2E test file in Git with the commit hash.
- Comment snippet demonstrating traceability back to the acceptance criteria IDs.
