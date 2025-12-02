# Code Review – RPG (Tool Playbook)

Blend SEU/PDM, VS Code, and Navigator views to review RPG updates thoroughly.

## Core Tools
- **5250 emulator with PDM/SEU** – open members, compare versions, and run syntax checks (`STRSEU` Option 14).
- **RDi or VS Code with Code for IBM i** – optional modern editor for inline diffs and linting.
- **IBM Navigator for i** – inspect job logs or spool files after test compiles.
- **VS Code** – read companion TypeScript changes if the RPG program feeds the API.

## Guided Steps
1. **Fetch the source member.** In 5250, run `STRPDM` → Option 2 (Work with members) to open the library/file the change targets. Use Option 5 to display and Option 14 to syntax-check.
2. **Compare revisions.** Use Option 3 (Copy) with `F15=Browse` or RDi/VS Code diff tools to compare the updated member against the previous production version stored in source control.
3. **Validate interfaces.** Cross-check field layouts with VS Code DTOs if the program writes to API-facing files or data queues.
4. **Compile and inspect logs.** Issue a test compile (`CRTRPGMOD`/`CRTBNDRPG`) and view the joblog via Navigator or `WRKJOB OPTION(10)` to ensure no severity > 0 messages persist.
5. **Sign off.** Record the compile listing path and any changed service programs in the PR or task comments.

## Evidence to Capture
- Screenshot of the SEU syntax check summary or RDi Problems view.
- Navigator joblog snippet with `Compilation successful` message.
- Notes showing DTO alignment if data is consumed by Node.js code.
