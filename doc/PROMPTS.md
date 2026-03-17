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

Read [@PLAN.md](file:///C:/Users/steven/work/kaya-wpf/doc/plan/PLAN.md). Ask any questions you have before implementing.

The GTK portion of the [@kaya-gtk-wpf](file:///C:/Users/steven/work/kaya-gtk-wpf) repository has been extracted to [@kaya-gtk](file:///C:/Users/steven/work/kaya-gtk). We need to do the same thing for the WPF portion in [@wpf](file:///C:/Users/steven/work/kaya-gtk-wpf/wpf). Bring that portion of the `kaya-gtk-wpf` codebase into the root of this repository. Bring along the README, build scripts, and so on.

This repository will remain **completely and only a port of the GTK app** and, as such, doesn't require the full prompt history or plans from the GTK app. Instead, prompts for the WPF portion (as you will see in the old `kaya-gtk-wpf` repo) are about how to port features from the GTK version and tweak the WPF app behaviour to ensure there are no bugs.
