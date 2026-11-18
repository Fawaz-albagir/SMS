# UNIFICATION_REPORT

`SMS_merged_baseline_v5.zip` was **not available** in this environment, so it was neither inspected nor merged. Nothing below
describes its contents. If it exists, upload it: its Attendance/Backup/Exams screens and services must be compared with the ones here
(see "Known risk" at the end) before the two lines of work are treated as one.

## 1. Files and packages actually examined
| Package | Content | Result of comparison with the unified tree |
|---|---|---|
| `files_9.zip` → `SMS_merged_baseline.zip` + `MISSING_FILES.md` | early merged baseline (135 files) | every file is identical or an older version of a file in the unified tree; nothing unique |
| `SchoolManagementSystem_Source_Phase4.zip` | Phase 4 source (130 files) | same: 115 identical, 15 older versions (incl. older `Fx.cs`, `Phase4RulesTests.cs`); nothing unique |
| `files_1_.zip` | 7 Finance files | 2 identical; `FinanceService`/`IFinanceService` have the same methods but the older one lacks the duplicate-payment guard; nothing unique |
| `files_2_.zip` | 10 test + string files | tests identical; both string files are strict subsets (300 keys) of the unified 545 |
| `files_3_.zip` | 10 domain/application files | 8 identical; `Common.cs` and `SystemSettings.cs` are subsets of the unified versions |
| `files_11.zip` → `SMS_merged_baseline_v3.zip` (147 files), `STATUS_v3.md`, `my_parallel_versions_NOT_IN_BUILD.zip` (6 files) | v3 baseline, its status report, parallel Attendance/Backup code | v3 is a strict subset of the unified tree (only my later changes differ) |
| `AttendanceTests.cs` | test file | identical to the copy in the tree |
| `SMS_full_source_v4.zip` (produced earlier) | v3 + Exams, UI screens, tests, scripts | **adopted as the base** |

Comparison was by file path and MD5, then by key/constant/method-set checks for files that differed (string keys, `ErrorCodes`, `SettingKeys`,
interface methods, DbSets, DI registrations). A file is "older" only when a script proved its content is a subset of the unified one.

## 2. Base
The unified tree = v4 = v3 + this session's work. No older package contained anything v4 lacked, except the items in section 3.

## 3. Merged from the parallel implementation (selective, after compatibility review)
* `InspectBackupFileAsync` + `BackupFileCheck` (school name, student and payment counts, hash) — added **additively** to `IBackupService`
  (v4's `VerifyBackupFileAsync` and all existing backup tests are untouched). The restore dialog now shows what the file contains. Tests: `BackupInspectTests`.
* Database check constraint `CK_StudentAttendance_Session (0..12)` on student attendance.

**Not merged, deliberately:** the parallel `IAttendanceService` (different record names: `AttendanceSheetRow`, `AttendanceSummaryRow`, `DailyAttendanceSummary`) and
`IBackupService` (different signatures, in `SMS.Application.Abstractions`). The tree's `AttendanceTests` and `BackupTests` are written against the v4 shapes,
and the Attendance/Backup screens bind to them; replacing them would break both. The parallel note limit (200) is lower than `AttendanceMath.MaxNoteLength` (300), so 300 was kept.

## 4. Features mentioned in reports but with no implementation files found
* Whatever `SMS_merged_baseline_v5.zip` contains (file missing).
* `STATUS_v3.md` says another agent was working on Exams: no such files exist in any package, so Exams was implemented here (service, EF, tests, screen).
* Timetable, Library, Inventory, Users: only permission entries exist.

## 5. Unresolved problems
* Nothing has been compiled or run. Expect compile errors on the first Windows build.
* No migrations, no installer, no portable ZIP.
* **Known risk:** if v5 holds its own Exams/Attendance/Backup implementations, they may duplicate or contradict these (different type names, DbSets, string keys, DI lines).
  They must be compared file by file when v5 is available; do not copy v5 over this tree or this tree over v5.

## Second consolidation (2026-11-16)
Compared: this tree (line A, 171 C# files incl. tests) against the v5-v10 line (line B, 159 C# files incl. tests; 56 of the 148 shared C# paths differ) and the later build package (line B + scripts/workflow). Result: line A kept as the base; line B contributed the items listed in `FILE_CHANGELOG.md` ("Second consolidation"). Line B's Users/Settings/Timetable/Library/Inventory/Exams services and screens were not merged: line A has equivalent implementations with equivalent safeguards, and merging two implementations of the same module without a compiler would add risk, not value. Not carried over from line B: its print service (line A prints through its own report writers), its statistics screen (line A has the five-report screen with CSV/Excel), its tests for the duplicated services.
Verification of this consolidation was text-level only (see `BUILD_AND_TEST_REPORT.md`).
