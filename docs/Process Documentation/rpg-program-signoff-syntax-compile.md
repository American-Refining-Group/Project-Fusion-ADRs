# RPG Program Signoff (Syntax/Compile)

Use this checklist to confirm an RPG program is ready to promote after compiling cleanly.

## Prep
- Obtain the latest source member and compile commands from the developer.
- Ensure access to the target IBM environment (DEV/TEST/UAT/PROD) plus the correct library list.
- Keep `DynamicLibraryManager` documentation handy for reference on library resolution.

## Steps
1. **Verify environment setup.** Run `SETENVPRC` or the equivalent CL to confirm the program sees the correct data, work, and stored procedure libraries. Record the output values.
2. **Compile with tracing enabled.** Execute the provided CRTBNDRPG/CRTRPGMOD commands. Capture the compile listing and highlight any warnings that still need signoff.
3. **Check error messaging.** Ensure the program writes failures to `errmsg_out` or another agreed data structure so Node logs surface compile/runtime errors.
4. **Validate external dependencies.** Confirm all called procedures, service programs, and files exist in the target libraries. Update the checklist if stubs or mocks were used.
5. **Align with Sequelize schema.** Cross-check field names and lengths against the Node schemas to avoid runtime truncation or mismatch.
6. **Document deployment steps.** Write down the exact commands (library overrides, binding directories, authorization lists) required so operations can reproduce the compile.

## Sign-off
- Attach the successful compile listing and command history to the ticket.
- Note any outstanding warnings and who accepted them.
