# MISSING_FILES / missing work

1. `src/SMS.Persistence/Migrations/*` — generate on Windows with `scripts\Add-Migration.ps1` (needs .NET SDK 10 and internet once for `dotnet-ef`). **Do this before the first real installation.**
2. `release/SchoolManagementSystem_Setup.exe`, `SchoolManagementSystem_Portable.zip`, `SHA256SUMS.txt` — built by `scripts\Build-Release.ps1` (Inno Setup 6 needed for the .exe).
3. Not built yet: student profile page, ID-card printing screen, audit-log viewer, dashboard widgets, qualifications editor.
4. Native PDF writer (receipts and report cards are HTML printed from the browser; Excel and CSV export exist).
5. `assets/app.ico`, bundled Arabic fonts.
6. Tests for Presentation helpers and view models.
7. `SMS_merged_baseline_v5.zip` — never available; see `UNIFICATION_REPORT.md`.

## Backup concurrency (owner-deferred item) - now implemented in source
`BackupGate` (static, process-wide) serialises `CreateBackupAsync`, `RunAutomaticIfDueAsync` and `RestoreAsync`: manual backups and restores wait up to 60 s for their turn (error `Backup.Busy` after that), the automatic backup never waits and skips the round. Four tests were written (`BackupTests`: gate held -> automatic skipped; manual waits then completes; restore waits then completes; four parallel manual backups all succeed and produce four files). **They have never been executed.** Treat the item as "implemented, unverified" until `Run-Tests.ps1` has passed on Windows.
