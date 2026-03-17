# Kaya WPF Port — Implementation Plan

## Context

Port the Kaya GTK4/TypeScript bookmarking and notes app to C#/WPF targeting .NET 9. The app saves bookmarks (URLs) and notes locally to `%USERPROFILE%\.kaya`, with optional server sync. The WPF version must be feature-equivalent to the GTK version, support dark/light theming, and use TDD with xUnit.

## Solution Structure

```
├── Kaya.sln
├── build.ps1                       # PowerShell build script
├── src/
│   ├── Kaya.Core/                  # Models + Services (no WPF dependency)
│   │   ├── Kaya.Core.csproj
│   │   ├── Models/
│   │   │   ├── Anga.cs
│   │   │   ├── IClock.cs
│   │   │   ├── DroppedFile.cs
│   │   │   ├── Filename.cs
│   │   │   ├── Meta.cs
│   │   │   └── KayaTimestamp.cs
│   │   └── Services/
│   │       ├── FileService.cs
│   │       ├── SettingsService.cs
│   │       ├── CredentialService.cs
│   │       ├── SyncService.cs
│   │       └── SyncManager.cs
│   └── Kaya.Wpf/                   # WPF UI layer
│       ├── Kaya.Wpf.csproj
│       ├── App.xaml / App.xaml.cs
│       ├── MainWindow.xaml / MainWindow.xaml.cs
│       └── PreferencesWindow.xaml / PreferencesWindow.xaml.cs
├── tests/
│   └── Kaya.Tests/
│       ├── Kaya.Tests.csproj
│       ├── AngaTests.cs
│       ├── MetaTests.cs
│       ├── TimestampTests.cs
│       ├── FilenameTests.cs
│       └── DroppedFileTests.cs
└── doc/
    └── plan/
        └── plan.md
```

## NuGet Dependencies

| Package | Project | Purpose |
|---------|---------|---------|
| `AdysTech.CredentialManager` | Kaya.Core | Windows Credential Manager access |
| `xunit` + `xunit.runner.visualstudio` | Kaya.Tests | Test framework |
| `Microsoft.NET.Test.Sdk` | Kaya.Tests | Test runner |

No other external libraries needed — `HttpClient`, `System.Text.Json`, and `System.IO` cover HTTP, JSON, and file operations.

## .NET 9 Dark/Light Theme

.NET 9 WPF ships with a built-in Fluent theme that respects the Windows system theme (dark/light) automatically. Enable it in `App.xaml`:

```xml
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <ResourceDictionary Source="pack://application:,,,/PresentationFramework.Fluent;component/Themes/Fluent.xaml" />
        </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

Set `ThemeMode="System"` on the application to auto-switch with the OS setting.

## Implementation Steps (TDD)

### Phase 1: Project scaffolding
1. Create `Kaya.sln`, `Kaya.Core.csproj` (net9.0 class library), `Kaya.Wpf.csproj` (net9.0-windows WPF app), `Kaya.Tests.csproj` (net9.0 xUnit)
2. Add NuGet references
3. Create `build.ps1`

### Phase 2: Models (test-first)
Write tests first, then implement each model to make tests pass.

4. **IClock** — `IClock` interface with `DateTimeOffset Now()`. `SystemClock` and `FrozenClock` implementations. No tests needed (trivial).
5. **KayaTimestamp** — Tests: verify `Plain` format (`2005-08-09T123456`) and `WithNanos` format (`2005-08-09T123456_000000000`). Use `DateTimeOffset` and tick-based nanos.
6. **Anga** — Tests: URLs produce `.url` files with `[InternetShortcut]` format; non-URLs produce `.md` files.
7. **Meta** — Tests: TOML generation, triple-quote escaping, multi-line notes.
8. **Filename** — Tests: validation against URL-illegal characters (spaces, `#?&+=!*'()<>[]{}"@^~\`;\|`).
9. **DroppedFile** — Tests: timestamped filenames, URI-encoding of original filenames, binary content preservation.

### Phase 3: Services
10. **FileService** — Ensure `%USERPROFILE%\.kaya\{anga,meta}` directories exist. Save anga, meta, and dropped files with collision-safe naming (try plain timestamp, fall back to nanos).
11. **SettingsService** — Store settings in `%USERPROFILE%\.kaya\settings.json`. Properties: `ServerUrl`, `Email`, `SyncEnabled`, `LastSyncError`, `LastSyncSuccess`. Changed event for UI binding.
12. **CredentialService** — Wraps `AdysTech.CredentialManager` for `GetPassword()`, `SetPassword()`, `ClearPassword()` targeting `Kaya` credential in Windows Credential Manager.
13. **SyncService** — Port sync logic using `HttpClient`. Basic auth, multipart form-data uploads. Sync anga, meta, and words directories bidirectionally. Validate filenames before upload.
14. **SyncManager** — Timer-based sync every 60 seconds using `System.Threading.Timer`. Start/stop lifecycle.

### Phase 4: WPF UI
15. **App.xaml** — Fluent theme, `ThemeMode="System"`, app-level keyboard shortcuts.
16. **MainWindow** — Text entry for URL/note, multiline text box for optional note, Save button, drag-and-drop target area, toast-like notifications (e.g. using a Snackbar-style overlay or a disappearing StatusBar message).
17. **PreferencesWindow** — Server URL, email, password fields. Sync status display. Force Sync button. Open via menu or `Ctrl+,`.

### Phase 5: Build script & integration
18. **build.ps1** — PowerShell script with commands: `build`, `test`, `run`, `clean`, `publish`.

## Key Design Decisions

- **Kaya.Core has no WPF dependency** — all models and services are in a plain .NET class library, making them fully testable without UI.
- **Settings as JSON** — no Windows Registry. A simple `settings.json` in `.kaya` keeps config portable and inspectable. Use `System.Text.Json` for serialization.
- **Credentials separate from settings** — passwords go through Windows Credential Manager only, never written to disk in plaintext.
- **`DateTimeOffset` instead of Temporal API** — C#'s `DateTimeOffset` provides UTC-aware timestamps. For nanosecond precision in filenames, use `DateTime.Ticks` (100ns granularity, close enough to the GTK app's nanosecond naming).
- **Toast notifications** — WPF has no built-in toast/snackbar. Implement as a simple auto-hiding `TextBlock` overlay in `MainWindow`, keeping it lightweight without pulling in a UI framework.

## Build Script (`build.ps1`)

```powershell
param([string]$Command = "build")
switch ($Command) {
    "build"   { dotnet build }
    "test"    { dotnet test }
    "run"     { dotnet run --project src/Kaya.Wpf }
    "clean"   { dotnet clean }
    "publish" { dotnet publish src/Kaya.Wpf -c Release }
}
```

## Verification

1. `.\build.ps1 test` — all xUnit tests pass
2. `.\build.ps1 run` — app launches, respects system dark/light theme
3. Enter a URL and save — file appears in `%USERPROFILE%\.kaya\anga\` as `.url`
4. Enter text and save — file appears as `.md`
5. Open preferences, configure server, verify settings persist in `settings.json`
6. Drag and drop a file — saved to `anga` with URL-encoded name
