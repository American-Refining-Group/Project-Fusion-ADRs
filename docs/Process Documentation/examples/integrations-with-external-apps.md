# Example – Integrations with External Apps

## Scenario Snapshot
- **Goal:** push voucher approvals from ARG Web Backend into a partner system via REST webhooks.
- **Inputs:** NestJS `VoucherController`, outbound integration service in `src/shared/infrastructure/connection.ts`, queue workers defined under `src/main/account-payable/application/voucher/usecases`.
- **Output:** documented handshake covering authentication, payload contract, and IBM i references.

## Step-by-Step Walkthrough
1. **Gather specs.** Download the partner's OpenAPI contract and compare it against `src/api-schema/voucher.yml` to understand field overlaps.
2. **Map auth requirements.** Confirm the integration user/secret are stored in Azure Key Vault or `.env` and referenced in the NestJS `ConfigService` before coding.
3. **Prototype the call.** Use VS Code REST Client or Postman to hit the partner sandbox with a payload generated from `voucher.dto.ts`.
4. **Connect to IBM i triggers.** Trace how voucher approvals originate from RPG by reviewing the job log for program `AP200` and ensuring the `AP200PRC` stored procedure entry in `table-registry.ts` matches.
5. **Implement the webhook.** Update the outbound service to call the partner API after a successful `AddVoucherEntryUseCase`. Add retries via BullMQ if the partner is down.
6. **Prove end-to-end flow.** Run the NestJS app locally (`yarn start:dev`), approve a voucher, and show the outbound HTTP 200 plus the IBM job log entry referencing the same voucher number.

## Evidence Package
- Postman collection export with the tested webhook request/response.
- Screenshot of the IBM Navigator job log referencing the voucher event.
- Link to the merge request showing the outbound service changes and associated unit test updates.
