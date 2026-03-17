# Release Script

## Context

The GTK app has `bin/release.rb` (Ruby) that automates versioning, committing, tagging, and pushing releases. The WPF app needs an equivalent. Ruby is not available in the standard Windows development environment, so the script is ported to PowerShell, which is native to Windows and already used for `build.ps1`.

## Approach

Port the GTK Ruby release script to PowerShell with the same 3-step interactive flow:

1. **Determine version**: read git tags, suggest patch bump, prompt for confirmation
2. **Update source files**: update version string in `AboutWindow.xaml` (the only file containing the version in the WPF app — no `package.json`, `meson.build`, or Flatpak manifest)
3. **Commit, tag, push**: commit version bump, create annotated tag, prompt before pushing

### Implementation Details

- `Run-Git` helper uses `cmd /c` to merge stderr into stdout before PowerShell sees it (git writes progress to stderr, which PowerShell treats as errors)
- Tag list forced to array with `@(...)` to handle single-tag edge case in strict mode
- Version format validated as `X.Y.Z`

## Files

- `bin/release.ps1` (new)
