# RPG Program Signoff (Syntax/Compile) – Tool Playbook

Ensure every compile uses the correct IBM i tooling and artifacts.

## Core Tools
- **5250 emulator** – run `STRSEU` Option 14 or `CRTBNDRPG` commands.
- **PDM** – track members and capture compile history via Option 8 (Attributes) or 14 (Compile).
- **IBM Navigator for i** – review job logs/spooled compile listings.
- **VS Code / RDi** – store notes about compile options (`DFTACTGRP`, binding directories, etc.).

## Guided Steps
1. **Open the source in SEU.** `STRPDM` → library/file → Option 2 → member → Option 14 for syntax. Resolve any flagged lines before continuing.
2. **Kick off the compile from PDM.** Use Option 14 and pick the appropriate command (`CRTBNDRPG`, `CRTRPGMOD`, or `CRTSQLRPGI`). Document the options in a VS Code scratch file so you can repeat the build.
3. **Monitor job status.** Press `F1` on any compile message, then review the joblog via Navigator (*Jobs → Active → <job> → Messages*).
4. **Capture spool output.** In Navigator, download the `QSYSPRT` or `QRPGLESRC` listing for attachment to the ticket.
5. **Sign off.** Update the tool-specific checklist in `docs/delivery/rpg-program-signoff-syntax-compile.md` noting the library, object, and compile command.

## Evidence to Capture
- Screenshot of SEU syntax check summary.
- Joblog excerpt with `Errors = 0` and `Severity < 30`.
- Spool file (PDF/TXT) attached to the work item.
