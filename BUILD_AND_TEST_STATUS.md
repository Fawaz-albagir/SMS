# BUILD_AND_TEST_STATUS

## Consolidation update (2026-11-16)
This tree is the result of merging two diverged lines of work (details: `UNIFICATION_REPORT.md`, section "Second consolidation").
**Still true: nothing here has been compiled, run, tested or installed.** The environment that produced it has no .NET SDK, no Windows, no Inno Setup and no network.

* Test methods declared: 235 (`[Fact]` + `[Theory]`: 179 integration, 56 unit), including 4 new backup-concurrency tests and the migration safety tests. None has been executed.
* New in this step: process-wide `BackupGate` (manual backup / automatic backup / restore can no longer overlap), `--selftest` start-up check (`SelfTest.cs`), the complete script set (`Check-Prerequisites`, `Verify-Migrations`, `Test-Installer`, `Verify-Portable`, `Verify-Release`, ...), the improved Inno Setup script, the GitHub Actions workflow `.github/workflows/build-windows.yml`, and the offline Roslyn check tools in `tools/`.
* Static checks re-run on this exact tree: `python tools/static_checks.py` -> ALL CHECKS PASSED (text analysis only, not a build), plus an independent consistency script (permission codes, dynamic localization keys, DI, DbSet configuration, XAML binding names): no real failures.


**Nothing in this tree has been built, run, tested or installed.** Full detail: `documentation\VERIFICATION_REPORT.md`.

| Item | Status |
|---|---|
| `dotnet build` | NOT RUN |
| Unit + integration tests (209 Fact + 18 Theory declared) | NOT RUN |
| EF migrations | NOT GENERATED |
| Setup.exe | NOT BUILT |
| Portable ZIP | NOT BUILT |
| UI on Windows 10 / 11 | NOT RUN |

Static checks that were run (text analysis): balanced braces in 171 C# files; all ErrorCodes defined and translated; 791 identical string keys in EN and AR with no missing key; XAML well-formed; every view model registered and templated; every service registered; every DbSet configured.

To obtain real results on Windows: `.\scripts\Run-Tests.ps1` → `.\scripts\Add-Migration.ps1` → `.\scripts\Build-Release.ps1` → `.\scripts\Verify-Package.ps1` → `documentation\INSTALL_TEST_CHECKLIST.md`.
