# School Management System (SMS)

Offline Windows desktop application for a private school in Sudan. C# / .NET 10, WPF (MVVM), SQLite + EF Core.
Arabic (default, right-to-left) and English.

> **Status: Version 1.0.0 is under construction. Phases 1-3 (foundation, authentication, first-run setup) are written.
> Academic, finance, reporting, backup and the other modules are NOT built yet.**
> Nothing in this repository has been compiled or run yet; see `documentation/DEVELOPMENT_PROGRESS.md` and `documentation/TEST_REPORT.md`.

## Build on Windows 10/11 x64

Prerequisites on the build machine: [.NET 10 SDK](https://dotnet.microsoft.com/download), [Inno Setup 6](https://jrsoftware.org/isdl.php), internet for the first NuGet restore.

```powershell
.\scripts\Build-Release.ps1
```

That one command creates the EF migrations if missing, restores, builds, runs the tests, publishes a self-contained win-x64 build,
builds `release\SchoolManagementSystem_Setup.exe`, the portable ZIP and the documentation. It stops at the first failure.

Useful variants: `-SkipTests`, `-SkipInstaller`, `-NoReadyToRun`, `-Version 1.0.1`. Tests only: `.\scripts\Run-Tests.ps1`.

## Folder layout

| Folder | Content |
|---|---|
| `src/SMS.Shared` | `Money` (integer minor units), `Result` |
| `src/SMS.Domain` | Entities, permission catalog, role templates |
| `src/SMS.Application` | Interfaces, session, password policy |
| `src/SMS.Persistence` | EF Core DbContext, configurations, services, seeding |
| `src/SMS.Infrastructure` | Argon2id hashing, data paths, logging |
| `src/SMS.Presentation` | WPF app (login, setup wizard, main window, themes, EN/AR strings) |
| `tests/` | Unit tests and integration tests (real SQLite) |
| `installer/` | Inno Setup script |
| `scripts/` | Build, migration, portable-package and docs scripts |
| `documentation/` | Architecture, schema, ERD, guides, progress, test report |

`SMS.Reporting` (PDF/Excel) and `SMS.UITests` are created in the phases that need them.

## Where data lives

Installed: `C:\ProgramData\SchoolManagementSystem\{Database,Backups,Logs,Reports,Exports,Attachments}`.
Portable: `.\Data\...` beside the exe. The database is never stored in the installation folder.
