# Integrations with External Apps

Follow these bullets to onboard or troubleshoot any third-party system that exchanges files or API calls with ARG.

## Prep
- Open `src/modules/voucher/voucher.controller.ts` to see every exposed route plus Swagger decorators.
- Review the processor under `src/modules/voucher/processors` (e.g., `flexi.processor.ts`) to understand how background jobs run.
- Skim `docs/DEBUGGING.md` for the end-to-end NestJS ↔ IBM troubleshooting flow.

## Steps
1. **Describe the entry point.** List the HTTP method, URL, and DTO for each integration. Example bullet: “`POST /voucher/flexi/upload` expects `FlexiUploadDto` with fields …”.
2. **Show authentication expectations.** Mention how the partner should authenticate (API key, OAuth, VPN allow list) and point to the middleware or guard that enforces it.
3. **Outline the processing pipeline.** Create a mini-sequence such as “Upload hits controller → Bull queue (`flexi`) → `FlexiProcessor` batches and validates → repository saves data → websocket notifies UI”.
4. **Call out required IBM assets.** Note which RPG programs, libraries, or spool files the integration ultimately touches so IBM teams can prep promotions.
5. **Provide monitoring hooks.** Include the loggers, metrics, or Bull board to watch (`/admin/queues`, Kibana index, etc.) along with the command to tail logs (`yarn start:dev` locally, `kubectl logs` in prod).
6. **List debugging steps.** Link the relevant section of `docs/DEBUGGING.md` plus any SQL scripts or joblog retrieval commands unique to this integration.

## Sign-off
- Share the document with the external partner so they can validate the payload shape and SLA.
- Capture any agreed retry/alerting rules and store them next to the integration ticket.
