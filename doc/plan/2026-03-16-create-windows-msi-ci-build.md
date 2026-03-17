# Create Windows `.msi` on CI build

## Context

The GTK app produces platform artifacts (Flatpak, DEB, RPM, Snap, DMG) via GitHub Actions workflows triggered on `main` push (nightly) and tag push (release). The WPF app needs a similar workflow producing a Windows `.msi` installer.

## Approach

Use GitHub Actions with a `windows-latest` runner to:

1. Build the .NET 9 WPF app as a self-contained, single-file publish for `win-x64`
2. Run tests
3. Package the published output into an `.msi` using the `dotnet-wix` tooling (WiX v5 via `wix` .NET tool)
4. On tag push (`v*`), attach the `.msi` to the GitHub Release

### Build Steps

- `dotnet test` — run all tests
- `dotnet publish` — self-contained, single-file, trimmed, `win-x64`
- Install WiX v5 via `dotnet tool install --global wix`
- Use a `.wxs` WiX source file to define the MSI: install to Program Files, add Start Menu shortcut, set icon
- `wix build` to produce the `.msi`

### Workflow Triggers

- `push` to `main` — build + test + produce `.msi` as artifact (nightly)
- `push` tag `v*` — build + test + produce `.msi` + attach to GitHub Release

## Files

- `.github/workflows/windows.yml` — GHA workflow
- `packaging/SaveButton.wxs` — WiX v5 source file
