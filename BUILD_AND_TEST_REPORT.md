# BUILD AND TEST REPORT - School Management System (consolidated source)

Date: 2026-11-16

## Result in one line
**The installer and the portable package were NOT produced.** The source could not be compiled in the environment that did this work, so nothing below "static checks" has been verified by a compiler, a test run or an installation.

## Why there is no Setup.exe
The work environment is Linux with no .NET SDK, no `wine`/`mono`/`msbuild`, no Inno Setup, and no network access (NuGet and package mirrors are unreachable; checked with `curl` and `apt-get update`). No Windows machine and no remote build service (for example GitHub Actions) was available, and no repository credentials were provided. A genuine `SchoolManagementSystem_Setup.exe` can only be produced by a successful Windows build, so none is delivered and no placeholder file was created.

## Status table
| Item | Status |
|---|---|
| Source consolidated into one tree | Done (see below) |
| Restore NuGet packages | NOT RUN |
| Release build of the solution | NOT RUN (never compiled) |
| Unit and integration tests (235 methods declared: 56 unit, 179 integration) | NOT RUN - none passed, none failed, none executed |
| EF Core migration generation | NOT RUN (the build scripts generate it on the build machine) |
| Self-contained win-x64 publish | NOT RUN |
| `SchoolManagementSystem_Setup.exe` | NOT BUILT |
| `SchoolManagementSystem_Portable.zip` | NOT BUILT |
| SHA256 of installer / portable | not applicable (files do not exist); `SHA256SUMS.txt` covers only the files that were delivered |
| Installation, first launch, database creation, uninstall | NOT TESTED |
| Printing, Excel export, Arabic/English layout on screen | NOT TESTED |

## What was done
1. Inspected every uploaded ZIP and report: the v3-v10 packages, the "unified v6" source with its verification report, and the build package with its GitHub workflow.
2. Found two diverged lines of work from the same v4 ancestor; compared them file by file; chose the more complete line as the base (it contains all modules plus Guardians, Promotion, Excel export, backup inspection, file-database and migration-consistency tests, forced password change, hourly automatic backup, and equivalent safeguards). Details: `UNIFICATION_REPORT.md`.
3. Brought over what the base lacked: the backup concurrency gate, the `--selftest` start-up check, the complete build/verify scripts, the improved Inno Setup script (output `SchoolManagementSystem_Setup.exe`), the portable README, the GitHub Actions workflow, the migration safety test, the dotnet-ef tool manifest and the offline Roslyn check tools.
4. Backup concurrency (reviewed as requested): a process-wide gate now serialises manual backup, automatic backup and restore. Four tests were written for it. **Implemented in source; not executed.**
5. Ran static checks on the final tree.

## Static checks that were run (text analysis, not a compiler)
Run on the delivered tree: `python tools/static_checks.py` -> "ALL CHECKS PASSED" (one heuristic warning about XAML bindings, which was inspected by hand and found to be a false positive); plus an independent script that verified: balanced braces/parentheses in all C# files; every permission code used exists in the catalog and each service's `Can(...)` actions are valid for its module; every `ErrorCodes.*` is defined and has English and Arabic text; English and Arabic string files have identical keys; all keys used by the UI exist, including keys built from enum names; XAML is well-formed; every requested view model is registered and has a view; every application service is registered; every `DbSet` has an EF configuration; every `SettingKeys`/`SequenceNames` member used exists; XAML binding names resolve against the view models. The independent script flagged three items that were then examined by hand: (1) dynamic-key prefixes it had no enum mapping for (`Act_`, `Emp_*`, `Inv_Kind_`, `Mod_`, `Pro_Outcome_`, `Rel_`, `Rep_`): the keys were compared with the enum values and match; (2) the Finance sub-screens have no top-level template because they are hosted as tabs inside the Finance screen; (3) binding names of the Academic Structure screen that belong to row DTO types declared in other files (not resolved by the script, not inspected line by line). So item (3) is unverified.

Additional symbol-level checks (`python tools/symbol_checks.py`, approximating a few compiler errors): every service method called through `GetRequiredService<I..Service>()` or `Svc<I..Service>()` exists on its interface (260 calls), the number of arguments matches the interface signature in all of them, and every referenced enum member exists. A record-constructor argument-count check found no real mismatch (its only hits were generic-comma false positives).

**These checks cannot find type errors, missing overloads, nullability problems or runtime failures.** Expect compile errors on the first real build.

## Known limitations and risks
* Never compiled: assume there are compile errors until a build succeeds.
* No EF migration exists yet; until the build scripts generate one the app would create the schema with `EnsureCreated`. The scripts and workflow generate and verify the migration before publishing.
* Backup concurrency tests and the whole test suite have never run.
* WPF behaviour (RTL layout, printing, DataGrid editing, PasswordBox handling) is unverified.
* The installer script has never been compiled with Inno Setup.

## How a real result will be obtained
Push this tree to a GitHub repository: `.github/workflows/build-windows.yml` restores, generates and verifies the migration, builds Release, runs all tests, publishes self-contained win-x64, builds the installer, runs an install / start-up self-test / uninstall test, and uploads `SchoolManagementSystem_Setup.exe`, `SchoolManagementSystem_Portable.zip`, `SHA256SUMS.txt` and the real test report as artifacts. Alternatively run `scripts\Build-Release.ps1 -TestInstaller` on Windows 10/11 x64 with the .NET 10 SDK and Inno Setup 6. This has not been done.
