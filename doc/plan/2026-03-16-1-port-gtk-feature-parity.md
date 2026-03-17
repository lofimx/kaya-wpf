# Port GTK Feature Parity to WPF

## Context

The `kaya-gtk` repo drifted significantly from the WPF port. This plan covers bringing `kaya-wpf` to feature parity with GTK, excluding platform-specific features (Flatpak, Snap, DEB/RPM, Gentoo, MacOS, Native Host browser daemon for Linux).

## Changes

### Core Model Updates

- **`Filename.cs`**: Added `Extension`, `IsImage()`, `IsPdf()`, `AngaType`, `Date`, `RawTimestamp`, `DisplayTitle`. GTK's `Filename` class is rich; WPF's only had `IsValid()`. `DisplayTitle` preserves hyphens and extension for files but strips them for notes/bookmarks (matching GTK).
- **`SearchResult.cs`**: Delegates to `Filename` for type detection and title extraction. Added `IsTitleVisible()` — returns `true` only for file type (not bookmarks/notes).
- **`TagsList.cs`** (new): Tag collection with `Add`, `RemoveLast`, `WithPending`. Matches GTK's `tags_list.ts`.
- **`NativeHostRequest.cs`** (new): Discriminated union model for parsing HTTP method + path into typed requests (Health, Preflight, Config, Listing, FileWrite, Invalid). Extracted from server for testability. Matches GTK's `native_host_request.ts`.

### New Services

- **`Logger.cs`**: Writes timestamped logs to `~/.kaya/desktop-app-log` and console. Replaced all `Console.WriteLine`/`Console.Error.WriteLine` calls across the codebase.
- **`MetaService.cs`**: Loads and parses TOML metadata from `~/.kaya/meta/` using regex. Finds the most recent meta file for a given anga filename.
- **`ShareService.cs`**: HTTP POST to `{server}/api/v1/{email}/share/anga/{filename}` with Basic Auth. Returns share URL from JSON response. Copies to clipboard on success.
- **`NativeHostServer.cs`**: HTTP server on `127.0.0.1:{port}` using `HttpListener`. Same API as GTK version: health, listing, file_write, config, CORS preflight. Uses `NativeHostRequest` model for routing.
- **`FileService.cs`**: Added `ReadAngaContents()`, `ReadAngaBytes()`, `GetAngaFilePath()`.
- **`SettingsService.cs`**: Added `SyncInProgress` (non-persisted, fires `Changed`), `NativeHostPort` (persisted, default 21420). `ShouldSync()` checks `SyncEnabled`.
- **`SyncManager.cs`**: Added `TriggerSync()` for immediate sync. Sets `SyncInProgress` during sync for UI feedback.
- **`SyncService.cs`**: Ensures `~/.kaya/anga` and `~/.kaya/meta` directories exist before sync. Words sync uses count comparison to skip when counts match (was making 144+ HTTP requests per cycle).

### Preview Window (new)

- Modal window opened by double-clicking a tile on the Everything window
- Content display based on anga type:
  - **Bookmarks**: URL text + "Visit Website" button
  - **Notes**: rendered text content
  - **Images**: inline display with `Stretch="Uniform"` (no scrollbars)
  - **SVGs**: rendered to bitmap via `Svg.Skia` + SkiaSharp
  - **PDFs**: page-by-page rendering via `Windows.Data.Pdf` with Previous/Next navigation
  - **Other files**: type icon + "Open Externally" button
- Collapsible sidebar (golden ratio 61.8/38.2 split) with tags editing (pill UI), notes editing, share button
- Save creates a new meta file; Cancel/Escape closes

### Preferences Window Updates

- **Save/Clear buttons** replacing auto-save-as-typed. Clear shows a confirmation dialog.
- **Browser tab** with native host port setting (validated 1–65535)
- `SizeToContent="Height"` for flexible layout
- Threading fix: `Changed` events from background sync thread marshalled via `Dispatcher.Invoke`

### Everything Window (MainWindow) Updates

- **Double-click to open Preview** (Windows convention: click-to-select, double-click-to-open)
- **Enter key** also opens Preview on selected item
- **Sync spinner**: arc `Path` in 16x16 `Canvas` with `RotateTransform` animation, themed via `SystemColors.HighlightBrushKey`. Checks current state on setup (doesn't miss in-progress sync).
- **Tile improvements**: title only shown for file type, date shown only on row hover (via `DataTrigger` on ancestor `ListBoxItem.IsMouseOver`), `Visibility="Hidden"` reserves space to prevent layout shift
- **Auto-refresh** when sync completes
- Standard arrow cursor (no hand cursor on tiles)

### New Save Window Updates

- **Immediate upload**: triggers sync after save
- **Tags/Notes as file drop targets**: file drops on Tags entry and Notes field are passed up to the window's file handler
- **PDF preview**: first page rendered via `Windows.Data.Pdf`
- **SVG preview**: rendered via `Svg.Skia`

### App Lifecycle

- `App.xaml.cs` creates `SyncManager` and `NativeHostServer`, passes `SyncManager` to `MainWindow`
- `NativeHostServer.OnFileReceived` triggers sync + UI refresh
- Removed `StartupUri`; `MainWindow` created manually in `OnStartup`

## Bug Fixes

- **Preferences "Not configured"**: `ShouldSync()` was checking `IsCustomServerConfigured()` which excluded the default `savebutton.com` server. Now checks `SyncEnabled`.
- **Threading crash**: `SettingsService.Changed` fired from Timer thread, PreferencesWindow tried to update UI. Fixed with `Dispatcher.Invoke`.
- **Sync performance**: `SyncExistingWord` made HTTP requests for all 144 existing word directories every 60 seconds. Removed; uses count comparison like GTK.
- **Ensure anga dir**: `SyncService` now creates anga/meta directories before sync.
- **About window height**: changed to `SizeToContent="Height"`.

## Tests Added

- `TagsListTests.cs`: 10 tests (including mutation safety)
- `FilenameExtendedTests.cs`: 16 tests for new Filename properties
- `NativeHostRequestTests.cs`: 22 tests matching GTK suite (health, preflight, config, listing, file_write, validation)
- `MetaServiceTests.cs`: 1 integration test
- Total: 136 tests, all passing

## Files

### New
- `src/Kaya.Core/Models/TagsList.cs`
- `src/Kaya.Core/Models/NativeHostRequest.cs`
- `src/Kaya.Core/Services/Logger.cs`
- `src/Kaya.Core/Services/MetaService.cs`
- `src/Kaya.Core/Services/ShareService.cs`
- `src/Kaya.Core/Services/NativeHostServer.cs`
- `src/Kaya.Wpf/PreviewWindow.xaml` + `.cs`
- `src/Kaya.Wpf/SvgRenderer.cs`
- `tests/Kaya.Tests/TagsListTests.cs`
- `tests/Kaya.Tests/FilenameExtendedTests.cs`
- `tests/Kaya.Tests/NativeHostRequestTests.cs`
- `tests/Kaya.Tests/MetaServiceTests.cs`

### Modified
- `src/Kaya.Core/Models/Filename.cs`
- `src/Kaya.Core/Models/SearchResult.cs`
- `src/Kaya.Core/Services/FileService.cs`
- `src/Kaya.Core/Services/SettingsService.cs`
- `src/Kaya.Core/Services/SyncManager.cs`
- `src/Kaya.Core/Services/SyncService.cs`
- `src/Kaya.Core/Services/SearchService.cs`
- `src/Kaya.Wpf/App.xaml` + `.cs`
- `src/Kaya.Wpf/MainWindow.xaml` + `.cs`
- `src/Kaya.Wpf/NewSaveWindow.xaml` + `.cs`
- `src/Kaya.Wpf/PreferencesWindow.xaml` + `.cs`
- `src/Kaya.Wpf/AboutWindow.xaml`
- `src/Kaya.Wpf/Kaya.Wpf.csproj` (target framework, Svg.Skia)
- `tests/Kaya.Tests/SearchResultTests.cs`
