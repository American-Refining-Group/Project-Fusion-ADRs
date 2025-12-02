# Transfer RPG Programs to Power 10

Use this checklist when moving RPG assets to new Power 10 hardware.

## Prep
- Gather the list of programs, service programs, and files being migrated.
- Confirm the target Power 10 library structure and any new naming conventions.
- Coordinate downtime or promotion windows with operations.

## Steps
1. **Inventory current libraries.** Run `SETENVPRC` (or review `DynamicLibraryManager` output) in the source environment and record the data, work, and stored procedure libraries for each program.
2. **Validate target libraries.** Compare the recorded list with the Power 10 libraries. Document any renames so Node configs (`DB_DATA_LIB_*`, etc.) can be updated.
3. **Migrate source members.** Copy the source (SAVLIB/RSTLIB, RCLSTG) and verify member counts and timestamps after transfer.
4. **Recompile on Power 10.** Execute the compile scripts in the new environment. Capture listings and confirm there are no hardware-specific warnings.
5. **Update application configs.** Modify environment variables, table registries, or Sequelize schemas if library names or file paths changed.
6. **Run validation tests.** Execute representative API flows or RPG harnesses to confirm the new programs behave identically. Record job numbers and spool outputs.
7. **Communicate completion.** Notify Node and IBM teams of the cutover, including any clean-up tasks on legacy hardware.

## Sign-off
- Store migration logs, compile listings, and config diffs in the ticket.
- Confirm rollback instructions exist before closing the task.
