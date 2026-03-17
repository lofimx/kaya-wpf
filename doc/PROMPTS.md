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

## Port all new behaviour from kaya-gtk to WPF

The `kaya-wpf` sister repo, [@kaya-gtk](file:///C:/Users/steven/work/kaya-gtk), has drifted from `kaya-gtk-wpf`, which this repo was extracted from. Some new features have been added and bugs have been fixed in `kaya-gtk`.

Review `kaya-gtk` to see what new features, bug fixes, refactoring, UI tweaks, and other changes have occurred since the WPF port was last in sync with the GTK app. Read through the [@PROMPTS.md](file:///C:/Users/steven/work/kaya-gtk/doc/PROMPTS.md) and [@plan/*.md](file:///C:/Users/steven/work/kaya-gtk/doc/plan/) in the `kaya-gtk` repo to see what changes were made and how they were planned.

Read the `kaya-gtk` code to understand it, then bring this repo up to feature parity. Some smaller changes may not have a prompt or plan in the `kaya-gtk` repo for you to reuse.

### BUG: Preferences insists it's not configured

Even though there is an email and password configured, the Preferences window says "Not configured" and "Force Sync" doesn't do anything.

### BUG: PDF not rendering, images render full-sized

* PDFs should render inline in the Preview window; they are only showing the PDF icon right now
* images (GIF, PNG, SVG, JPEG, etc.) should resize to fit the viewport rather than displaying scroll bars

### BUG: Sync is slow

The sync spinner seems to be running for a very long time. We had the same problem on Linux: the sync was confirming that every file was identical, when it really only needs to count files based on the file listing. Is that still happening in the WPF app and is that why the sync lasts a long time?

### BUG: Dates misaligned / always visible

In GTK, the dates on tiles are only visible when the mouse cursor hovers over the tile. This should also be the case in WPF. The date's visual alignment is quite ugly so let's force it to be right-aligned within the tile.

The date isn't right-aligned to the tile. It's only showing up immediately to the right of the title text. Also notable is that the tile alignment shifts when the date becomes visible, which it shouldn't. Last, the mouse cursor is a "pointer" (finger hand) over text, icon, and date -- it shouldn't be. If we are following the Windows convention of single-click to select and double-click to open, it should remain the standard arrow mouse pointer regardless of what part of the tile it's hovering over.

The date is only appearing when the cursor hovers the title text or the icon. It should appear whenever any part of the tile/row is hovered. Also, it's still rendering in a weird location on the right of some left-most column. Instead, let's have it render under the title text. Leave space for it so that when it renders it doesn't shift the widget layout.

### BUG: WPF Port is missing Web native host port setting

In the GTK Preferences window, there is a second tab which allows the user to set the port on which the local web browser connects to the app as a native host, over HTTP.

The browser extension found at [@kaya-wxt](file:///C:/Users/steven/work/kaya-wxt/) connects to the GTK app as a local host so that it can synchronize anga/meta/words between the browser extension's local filesystem and `~/.kaya`. The WPF port needs to exhibit this same behaviour and should also have an option (laid out according to the Windows Design Guidelines) to set the native host port its HTTP server runs on.

### BUG: PDF and SVG rendering on "New Save" window

When PDFs and SVGs are opened (or drag/dropped) in the "New Save" window, they should render so the user can see a preview. PDFs only need to show the first page. SVGs should scale to the viewport, as other images do.
