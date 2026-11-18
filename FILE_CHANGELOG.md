# FILE_CHANGELOG (this session, on top of SMS_merged_baseline_v3)

## New files
* `src/SMS.Persistence/Services/ExamService.cs`, `Configurations/ExamConfigurations.cs` — Exams service, EF mappings (hundredths storage, unique indexes).
* `src/SMS.Presentation/ViewModels/`: `PageViewModelBase`, `AttendanceViewModel`, `BackupViewModel`, `ExamsViewModel`, `FinanceViewModel`, `FinanceAccountsViewModel`, `FinanceSetupViewModel`, `FinanceExpensesViewModel`, `ReportsViewModel`.
* `src/SMS.Presentation/Views/`: `AttendanceView`, `BackupView`, `ExamsView`, `FinanceView`, `FinanceAccountsView`, `FinanceSetupView`, `FinanceExpensesView`, `ReportsView` (each `.xaml` + `.xaml.cs`).
* `src/SMS.Presentation/Services/`: `Parsing.cs` (money/int parsing with Arabic digits, `CsvFile`), `ReceiptWriter.cs`, `ReportCardWriter.cs`.
* `tests/SMS.IntegrationTests/ExamTests.cs`, `FileDatabaseTests.cs` (real-file SQLite); `tests/SMS.UnitTests/ExamCalculatorTests.cs`.
* `scripts/Run-Tests.ps1`, `scripts/Verify-Package.ps1`.
* `PROJECT_HANDOFF.md`, `FILE_CHANGELOG.md`, `BUILD_AND_TEST_STATUS.md`, `MISSING_FILES.md`.

## Modified files
* `Services/UiServices.cs` — `Prompt`, `PickFolder`, `PickFile`; `AppLifetime.ExitRequested`.
* `ViewModels/MainViewModel.cs` — navigation: Attendance, Exams, Finance, Reports, Backup.
* `Views/MainWindow.xaml` — DataTemplates for the new view models.
* `App.xaml.cs` — DI registrations; automatic backup at start-up and hourly; exit after restore.
* `SMS.Application/Common/Common.cs` — error codes `Common.NumberInvalid` and `Exams.*` (10).
* `SMS.Domain/Settings/SystemSettings.cs` — `Exams.OverallPassPercent`, `Exams.RequireAllSubjectsPass`.
* `SMS.Persistence/AppDbContext.cs` (Exam DbSets), `DatabaseInitializer.cs` (exam defaults), `PersistenceServiceCollectionExtensions.cs` (`IExamService`).
* `Localization/Strings.en.xaml` / `Strings.ar.xaml` — 542 keys each (was 302), identical key sets.
* `documentation/DEVELOPMENT_PROGRESS.md` — status rows updated.

## Unification step (after v4)
* `src/SMS.Application/Backup/IBackupService.cs`, `src/SMS.Persistence/Services/BackupService.cs`, `src/SMS.Persistence/SqliteFileOps.cs` — `InspectBackupFileAsync`, `BackupFileCheck`, `Summarize`.
* `src/SMS.Persistence/Configurations/AttendanceConfigurations.cs` — session check constraint.
* `src/SMS.Presentation/ViewModels/BackupViewModel.cs` — restore confirmation shows backup contents; 3 new strings (545 keys per language).
* `tests/SMS.IntegrationTests/BackupInspectTests.cs` — new.
* `UNIFICATION_REPORT.md` — new.

## Unified-tree development after unification (this is the current line of work)
* **Settings:** `Application/Settings/IAppSettingsService.cs`, `Persistence/Services/AppSettingsService.cs`, `Presentation/ViewModels/SettingsViewModel.cs`, `Views/SettingsView.xaml(.cs)`, `tests/.../SettingsTests.cs`.
* **Users:** `Application/Users/IUserAdminService.cs`, `Persistence/Services/UserAdminService.cs`, `Presentation/ViewModels/UsersViewModel.cs`, `Views/UsersView.xaml(.cs)`, `tests/.../UserAdminTests.cs`.
* **Staff UI:** `ViewModels/StaffViewModel.cs`, `Views/StaffView.xaml(.cs)`.
* `Services/UiServices.cs` — `PromptPassword` (masked input). `App.xaml.cs` — `EnsurePasswordChosenAsync` forces a new password after create/reset; `UsersViewModel` registered.
* `Common.cs` — codes `Settings.Invalid`, `Users.NameInvalid/NoRole/NotSelf/LastAdmin`, `Password.CurrentWrong/Unchanged`.
* `PersistenceServiceCollectionExtensions.cs` — `IAppSettingsService`, `IUserAdminService`.
* `Strings.en.xaml` / `Strings.ar.xaml` — 642 keys each, identical sets.
* `MainViewModel.cs` / `MainWindow.xaml` — navigation entries and templates for Staff, Users, Settings.

## Operations modules, roles, export, promotion, guardians (latest)
* **Timetable:** `Domain/Timetable/TimetableEntities.cs`, `Application/Timetable/ITimetableService.cs`, `Persistence/Services/TimetableService.cs`, `ViewModels/TimetableViewModel.cs`, `Views/TimetableView.xaml(.cs)`, `tests/.../TimetableTests.cs`.
* **Library:** `Domain/Library/LibraryEntities.cs`, `Application/Library/ILibraryService.cs`, `Persistence/Services/LibraryService.cs`, `ViewModels/LibraryViewModel.cs`, `Views/LibraryView.xaml(.cs)`, `tests/.../LibraryTests.cs`.
* **Inventory:** `Domain/Inventory/InventoryEntities.cs`, `Application/Inventory/IInventoryService.cs`, `Persistence/Services/InventoryService.cs`, `ViewModels/InventoryViewModel.cs`, `Views/InventoryView.xaml(.cs)`, `tests/.../InventoryTests.cs`.
* `Persistence/Configurations/OperationsConfigurations.cs` — mappings and database check constraints for the three modules; `AppDbContext.cs` — DbSets.
* **Roles:** `Application/Users/IRoleAdminService.cs`, `Persistence/Services/RoleAdminService.cs`, `ViewModels/RolesViewModel.cs`, `Views/RolesView.xaml(.cs)`, `tests/.../RoleAdminTests.cs`.
* **Excel export:** `Application/Export/XlsxWriter.cs` (no external library), `tests/SMS.UnitTests/XlsxWriterTests.cs`; Reports screen has an "Export Excel" button.
* **Promotion:** `ViewModels/PromotionViewModel.cs`, `Views/PromotionView.xaml(.cs)`. **Guardians:** `ViewModels/GuardiansViewModel.cs`, `Views/GuardiansView.xaml(.cs)`.
* `tests/.../MigrationConsistencyTests.cs` — fails when the model differs from the latest migration (does nothing until migrations exist).
* `Common.cs` — codes for Timetable, Library, Inventory, Roles; `PersistenceServiceCollectionExtensions.cs` — five new services; `MainViewModel.cs`, `MainWindow.xaml`, `App.xaml.cs` — navigation, templates, DI.
* `Strings.en.xaml` / `Strings.ar.xaml` — 791 keys each.
* `documentation/VERIFICATION_REPORT.md` — new.

## Second consolidation (2026-11-16)
| File / folder | Change | Reason |
|---|---|---|
| src/SMS.Persistence/BackupGate.cs | NEW | one backup/restore at a time (process-wide) |
| src/SMS.Persistence/Services/BackupService.cs | public Create/RunAutomatic/Restore wrapped by the gate; inner logic unchanged | concurrency |
| src/SMS.Application/Common/Common.cs, Strings.{en,ar}.xaml | + `Backup.Busy` / `Err_Backup_Busy` | message when the gate times out |
| tests/SMS.IntegrationTests/BackupTests.cs (+4 tests); BackupTests, BackupInspectTests, FileDatabaseTests | `[Collection("BackupGate")]` | the tests share the static gate, so they must not run in parallel |
| src/SMS.Presentation/SelfTest.cs; App.xaml.cs | NEW / `--selftest` hook | scripted start-up verification of a built package |
| scripts/* (12 scripts) | replaced by the complete set | Check-Prerequisites, Verify-Migrations, Test-Installer, Verify-Portable, Verify-Release, _Common |
| installer/SchoolManagementSystem.iss, installer/portable/README_PORTABLE.txt | replaced / added | improved installer; portable readme |
| tests/SMS.IntegrationTests/MigrationTests.cs | NEW | migration safety test |
| .github/workflows/build-windows.yml, documentation/BUILD_ON_GITHUB_AR.md, .config/dotnet-tools.json, tools/ | NEW | CI build, local dotnet-ef tool, offline checks |
Do not replace: the `[Collection("BackupGate")]` attributes; `BackupGate`; the files above with older copies.
