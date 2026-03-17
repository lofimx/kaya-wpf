# Windows C# WPF Port Prompts

## 2026-03-07 Port Search, New Save, Tags, and app rename

Read [doc/plan/](./plan), then port all features from the GTK app's PROMPTS.md (in the [kaya-gtk](https://github.com/lofimx/kaya-gtk) repo) to the WPF version.

### BUG: Widgets not following theme

The "tags" widget entry background is not following the system theme.

The background of the opened file widget in the "New Save" window is not following the system theme.

The background of the tiles (for anga/saves) on the Everything screen is not following the system theme.

### BUG: Icon is not displaying

The "Save Button" icon ([doc/design/yellow-floppy3.svg](./design/yellow-floppy3.svg)) should be used for the application's window, task bar, title bar, etc.

Write a utility .NET console app but keep it in `bin/`, rather than throwing it away, since we'll need to convert other SVGs to ICO/PNG/etc. in the future on Windows. Then use the console app to convert the SVG to ICO

## Move WPF port into its own repository

The GTK portion of the old `kaya-gtk-wpf` repository has been extracted to [kaya-gtk](https://github.com/lofimx/kaya-gtk). The WPF portion has been extracted here, to this `kaya-wpf` repository.
