<div align="center">
  <img src=".github/hua_nobg_512.gif" alt="椛" width = "400">
  <br>

  <h1>resvg-go</h1>
  <img src="https://counter.seku.su/cmoe?name=resvgo&theme=r34" /><br>
  A SVG renderer written in Go & WASM depended on resvg without CGO<br><br>
  
</div>

## Overview

`resvg-go` is an SVG renderer implemented in pure Go and WebAssembly without CGO dependencies. This library uses the [resvg](https://github.com/RazrFalcon/resvg) Rust library compiled to WebAssembly and invoked through Go's [wazero](https://github.com/tetratelabs/wazero) runtime.

## Key Features

- **No CGO Dependencies**: Implemented in pure Go and WebAssembly
- **Various Transformations**: Support for rotation, scaling, translation, skewing, and more
- **Font Management**: Custom font loading and configuration
- **Image Processing**: PNG encoding/decoding support
- **Rendering Options**: Detailed rendering option customization

## Installation

```bash
go get github.com/kanrichan/resvg-go
```

## Basic Usage

### Simple Rendering

The simplest way to render an SVG to PNG:

```go
// Initialize a worker (not goroutine-safe!)
worker, _ := resvg.NewDefaultWorker(context.Background())
defer worker.Close()

// Render SVG to PNG
png, _ := worker.Render(svg)
```

### Rendering with Options

For more fine-grained control:

```go
// Initialize a worker
worker, _ := resvg.NewDefaultWorker(context.Background())
defer worker.Close()

// Create and configure a font database
fontdb, _ := worker.NewFontDBDefault()
defer fontdb.Close()
fontdb.LoadFontData(ttf)  // Load custom font data

// Create a pixmap (specify output image size)
pixmap, _ := worker.NewPixmap(512, 512)
defer pixmap.Close()

// Create and render an SVG tree
tree, _ := worker.NewTreeFromData(svg, &resvg.Options{
    Dpi: 96.0,
    FontFamily: "Arial",
    FontSize: 14.0,
})
defer tree.Close()
tree.ConvertText(fontdb)
tree.Render(resvg.TransformIdentity(), pixmap)

// Encode to PNG
png, _ := pixmap.EncodePNG()
```

## Advanced Features

### Transformation Functions

You can apply various transformations when rendering SVGs:

```go
// Identity transform (no transformation)
transform := resvg.TransformIdentity()

// Translation transform (move by 100 on x-axis and 50 on y-axis)
transform = resvg.TransformFromTranslate(100.0, 50.0)

// Scaling transform (scale by 2.0 horizontally and 1.5 vertically)
transform = resvg.TransformFromScale(2.0, 1.5)

// Skewing transform
transform = resvg.TransformFromSkew(0.5, 0.3)

// Rotation transform (rotate by 45 degrees)
transform = resvg.TransformFromRotate(45.0)

// Rotation around a specific point (rotate by 45 degrees around point x:100, y:100)
transform = resvg.TransformFromRotateAt(45.0, 100.0, 100.0)

// Custom transformation matrix
transform = resvg.TransformFromRow(sx, ky, kx, sy, tx, ty)
```

### Font Management

The library provides various font configuration features:

```go
// Create a font database
fontdb, _ := worker.NewFontDBDefault()
defer fontdb.Close()

// Load a font file
fontdb.LoadFontFile("/path/to/font.ttf")

// Load fonts from a directory
fontdb.LoadFontsDir("/path/to/fonts")

// Load font data directly
fontdb.LoadFontData(ttfBytes)

// Set font family preferences
fontdb.SetSerifFamily("Times New Roman")
fontdb.SetSansSerifFamily("Arial")
fontdb.SetCursiveFamily("Comic Sans MS")
fontdb.SetFantasyFamily("Impact")
fontdb.SetMonospaceFamily("Courier New")

// Check the size of the font database
count, _ := fontdb.Len()
```

### Customizing Rendering Options

You can adjust various rendering options to customize the output:

```go
options := &resvg.Options{
    ResourcesDir: "/path/to/resources",
    Dpi: 300.0,  // High-resolution output
    FontFamily: "Helvetica",
    FontSize: 16.0,
    Languages: []string{"en", "en-US"},
    
    // Set rendering modes
    ShapeRenderingMode: resvg.ShapeRenderingModeGeometricPrecision,
    TextRenderingMode: resvg.TextRenderingModeOptimizeLegibility,
    ImageRenderingMode: resvg.ImageRenderingModeOptimizeQuality,
    
    // Set default sizes
    DefaultSizeWidth: 800.0,
    DefaultSizeHeight: 600.0,
}

tree, _ := worker.NewTreeFromData(svg, options)
```

### Image Processing

The library provides PNG image processing capabilities:

```go
// Create an empty pixmap
pixmap, _ := worker.NewPixmap(width, height)
defer pixmap.Close()

// Create a pixmap from PNG data
pngData := loadPNGFromSomewhere()
pixmap, _ := worker.NewPixmapDecodePNG(pngData)
defer pixmap.Close()

// Encode pixmap to PNG
pngBytes, _ := pixmap.EncodePNG()
```

## Important Notes

- Workers are not goroutine-safe. Do not use the same worker concurrently in multiple goroutines.
- All resources (Worker, FontDB, Pixmap, Tree, etc.) must be cleaned up with Close() after use.
- Consider memory usage when processing large SVG files.

## Acknowledgements

- [resvg](https://github.com/RazrFalcon/resvg) - an SVG rendering library written in Rust
- [wazero](https://github.com/tetratelabs/wazero) - the zero dependency WebAssembly runtime for Go developers

## License

For license information, please refer to the LICENSE file.
