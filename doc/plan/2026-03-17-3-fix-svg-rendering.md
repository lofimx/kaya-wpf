# Fix SVG Rendering

## Context

SVGs were not rendering in either the New Save window or the Preview window:

- **New Save**: used a `WebBrowser` control with `NavigateToString`, which showed nothing due to IE-based rendering limitations and security restrictions
- **Preview**: SVGs are included in `Filename.IsImage()` but `BitmapImage` cannot decode SVG format, so they fell through to the generic file icon

## Approach

Added the `Svg.Skia` NuGet package (v3.6.0), which uses SkiaSharp to rasterize SVGs to bitmaps natively without browser dependencies.

### `SvgRenderer.cs` (shared helper)

- Loads SVG via `SKSvg.Load()`
- Calculates scale to fit within a max dimension while preserving aspect ratio
- Renders to an `SKSurface`, encodes to PNG, wraps in `BitmapImage`
- Returns `null` on failure (caller decides fallback)

### New Save Window

- Replaced `WebBrowser` approach with `SvgRenderer.RenderToBitmap(filePath, 256)`
- Falls back to file icon if rendering fails

### Preview Window

- SVGs handled before the general `IsImage()` check (since `BitmapImage` can't load them)
- New `SetupSvgContent()` method uses `SvgRenderer.RenderToBitmap(filePath)` and displays in the same image panel as raster images
- Falls back to generic file view on failure

## Files

### New
- `src/Kaya.Wpf/SvgRenderer.cs`

### Modified
- `src/Kaya.Wpf/Kaya.Wpf.csproj` (added `Svg.Skia` package reference)
- `src/Kaya.Wpf/NewSaveWindow.xaml.cs` (replaced WebBrowser with SvgRenderer)
- `src/Kaya.Wpf/PreviewWindow.xaml.cs` (added `SetupSvgContent`, split `SetupImageContent` into `SetupRasterImageContent` + `SetupSvgContent`)
