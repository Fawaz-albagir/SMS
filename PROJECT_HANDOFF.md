# PROJECT_HANDOFF — School Management System

## Consolidation (2026-11-16) - read this first
Two lines of work existed: (A) the "unified v6" line (this tree's base) and (B) the v5-v10 line. Both started from the v4 source and both implemented Users/Roles, Settings, Exams, Timetable, Library, Inventory and the UI independently. They were compared file by file; line A was kept as the base because it is a superset (it also has Guardians, Promotion, Excel export, backup inspection, file-database tests, migration-consistency tests, forced password change, an hourly automatic backup, and equivalent safeguards: last-administrator, no self-disable, role locking, teacher clash, quota). From line B only what A lacked was brought over: the backup concurrency gate with tests, the `--selftest` start-up check, the complete build/verify scripts, the improved installer script, the GitHub workflow, the offline check tools and the migration safety test. Line B's own services/screens were NOT merged (they duplicate A's) and remain only in the older ZIPs for reference: do not mix files from them into this tree.
Product name of the installer: `SchoolManagementSystem_Setup.exe`; portable: `SchoolManagementSystem_Portable.zip`.
**No installer or portable package exists yet**: the code has never been compiled. The next step is a real Windows build: push this tree to a GitHub repository (the workflow builds, migrates, tests, publishes, creates the installer and runs the install/uninstall test on `windows-latest`), or run `scripts\Build-Release.ps1 -TestInstaller` on a Windows machine with .NET 10 SDK and Inno Setup 6. Expect compile errors on the first run; fix them and repeat.


**Truth statement: nothing in this tree has been compiled, run or installed.** It was written in a Linux sandbox with no .NET SDK,
no NuGet access and no Windows. Everything below is "written" or "statically checked", never "verified".

## What this is
WPF (.NET 10, win-x64) + EF Core + SQLite school management application, Arabic/English (RTL), offline, self-contained installer.
Layers: Shared → Domain → Application → Persistence / Infrastructure → Presentation. Tests: `tests/SMS.UnitTests`, `tests/SMS.IntegrationTests`.

## Modules and their real state (written, not compiled; see documentation\VERIFICATION_REPORT.md)
| Module | Domain/EF | Service | Tests declared | UI |
|---|---|---|---|---|
| Auth, first run | yes | yes | yes | login, setup wizard, forced password change |
| Users | yes | `UserAdminService` | `UserAdminTests` | yes |
| Roles & permissions | yes | `RoleAdminService` | `RoleAdminTests` | yes (permission matrix) |
| Settings | yes | `AppSettingsService` | `SettingsTests` | yes |
| Academic structure | yes | yes | yes | yes |
| Students, enrollment | yes | yes | yes | list/form/status dialog |
| Guardians | yes | yes | yes | **yes (new)** |
| Promotion | yes | yes | yes | **yes (new)** |
| Staff / teachers / assignments | yes | yes | yes | yes |
| Attendance | yes | yes | `AttendanceTests` | yes |
| Timetable | **new** | `TimetableService` | `TimetableTests` | **yes (new)** |
| Exams & report cards | yes | `ExamService` | `ExamTests`, `ExamCalculatorTests` | yes |
| Finance | yes | yes | `FinanceTests`, `FinanceGuardTests` | yes |
| Library | **new** | `LibraryService` | `LibraryTests` | **yes (new)** |
| Inventory (append-only ledger) | **new** | `InventoryService` | `InventoryTests` | **yes (new)** |
| Reports | n/a | uses other services | `XlsxWriterTests` | yes (5 reports, CSV + **Excel**) |
| Backup & restore | yes | yes | `BackupTests`, `BackupInspectTests`, `FileDatabaseTests` | yes |

Still not built: PDF writer (HTML print instead), student profile page, ID-card print, audit-log viewer, dashboard widgets, app icon, Arabic font bundle.

## First things to do on Windows 11 (in this order)
1. Install .NET SDK 10.0.100+ (see `global.json`). `.\scripts\Run-Tests.ps1` — expect compile errors first; fix them, then failing tests.
2. `.\scripts\Add-Migration.ps1` (needs internet once for `dotnet-ef`) → commit `src\SMS.Persistence\Migrations`. Until then the app uses `EnsureCreated`.
   After this, any later schema change needs `Add-Migration.ps1 -Name ...`. A database created by `EnsureCreated` cannot be upgraded by migrations — recreate it.
3. Run the app: `dotnet run --project src\SMS.Presentation`. Walk through every new screen once (Attendance, Exams, Finance, Reports, Backup).
4. Install Inno Setup 6, then `.\scripts\Build-Release.ps1` → `release\SchoolManagementSystem_Setup.exe` + portable ZIP + SHA256SUMS. `.\scripts\Verify-Package.ps1`.
5. Follow `documentation\INSTALL_TEST_CHECKLIST.md` on a clean Windows machine.

## Known risks to check on the first build
* WPF code was never compiled: XAML binding names, `DataGrid` templates and `DisplayMemberPath="Label"` on `Option<T>` (check `Option.cs` has `Label`).
* `UiServices` now needs `Microsoft.Win32.OpenFolderDialog` (.NET 8+, fine on .NET 10).
* `ReportsView` builds columns from a `DataTable`; headers are replaced in `OnAutoGeneratingColumn`.
* Restore replaces the database through the live connection; the app then exits (`AppLifetime.ExitRequested`). Test this on a real install.
* SQLite stores marks/percentages as integer hundredths (`MarkStorage`); `Money` as minor units. Never compare decimals in SQL.
* Presentation helpers (`Parsing`, `CsvFile`, `ReceiptWriter`, `ReportCardWriter`) have no tests: the WPF project is not referenced by the test projects.
* No app icon, no bundled Arabic fonts (see `assets\fonts\README`).

## Next steps (priority)
Run the build and the tests, fix what they reveal → generate migrations → build installer → clean-machine install test on Windows 10 and 11 → then: student profile page, audit viewer, dashboard, icon and fonts, native PDF.
