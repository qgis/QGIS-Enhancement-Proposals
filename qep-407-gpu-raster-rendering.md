# QGIS Enhancement: GPU-Accelerated Raster Rendering via QRhi

**Date** 2026/01/30

**Author** Wietze Suijker (@wietzesuijker)

**Contact** wietzesuijker at gmail dot com

**Version** QGIS 3.44

## Summary

Optional GPU rendering path for tiled rasters using Qt's Rendering Hardware Interface (QRhi). Tiles upload as GPU textures; color mapping runs in shaders. CPU path remains unchanged as automatic fallback.

## Proposed Solution

GPU rendering activates automatically when:
- GDAL raster provider with tiled/chunked format (COG, tiled GeoTIFF, Zarr)
- No reprojection required
- Valid QRhi context on the init thread

Otherwise falls back to CPU. No user configuration required.

## Architecture

### Why QRhi

[Per reviewer feedback](https://github.com/qgis/QGIS/pull/64701): "OpenGL is a legacy API... We should use QRhi."

QRhi provides backend abstraction (Metal/D3D11/Vulkan/OpenGL), handles context threading, and is Qt6's rendering direction.

### Core/GUI Separation

GPU rendering requires `Qt6::GuiPrivate`. To keep core free of GUI dependencies:

- **core**: `QgsRasterLayerRenderer` defines `GpuRendererFactory` callback; `QgsRasterTileReader` handles GDAL
- **gui**: `QgsRasterGPUFactory` registers callback at init; `QgsRasterGPURenderer` runs QRhi pipeline

### Persistent Tile Caching

[Per reviewer feedback](https://github.com/qgis/QGIS/pull/64701): "caching actually does not work, because the cache gets re-created from scratch on every render"

Fixed: `QgsRasterGPUCacheManager` singleton persists across renders. Tile uploaders cached per URI. GPU textures reused via LRU cache with frame-based eviction.

## Performance

### macOS (Apple M1 Metal) - CI Measured

| Scenario | CPU (ms) | GPU (ms) | Ratio |
|----------|----------|----------|-------|
| **Local COG** ||||
| Initial Load | 41.8 | 26.4 | 1.6x |
| Pan (12 steps) | 27.1 | 26.7 | 1.0x |
| Zoom (6 levels) | 32.8 | 24.8 | 1.3x |
| **Remote COG** (/vsicurl/) ||||
| Open | 120.2 | 23.6 | 5.1x |
| Pan | 125.7 | 27.0 | 4.7x |
| Zoom | 214.0 | 24.3 | 8.8x |

### Why Remote COGs Differ

The key is **where tiles live** after the first fetch:

```
CPU:  HTTP fetch (~100ms) → decode → render → discard → refetch next frame
GPU:  HTTP fetch (once) → decode → upload to VRAM → reuse across frames
```

QGIS's raster renderer relies on GDAL's block cache, which evicts under memory pressure. For local files, re-read costs ~1ms. For remote COGs, each evicted tile = another HTTP round-trip (50-200ms).

The GPU path's `QgsRasterGPUCacheManager` keeps tiles in VRAM until LRU eviction. The 8.8x zoom ratio reflects cached tiles from earlier zoom levels—no network wait when zooming back.

### Readback Overhead

Current GPU path uses synchronous readback for QPainter integration (~3ms). This explains why local panning shows ~1.0x ratio. The included `QgsMapCanvasGPU` (QRhiWidget subclass) avoids this by rendering directly to display—not yet wired into main canvas.

## Implementation

**New classes** (`src/gui/raster/`):
- `QgsRasterGPURenderer` - QRhi pipeline with readback
- `QgsRasterGPUTileUploader` - texture upload + LRU cache
- `QgsRasterGPUCacheManager` - singleton managing uploaders
- `QgsRasterGPUFactory` - creates QRhi, registers callback
- `QgsMapCanvasGPU` - QRhiWidget for direct display (no readback)

**Shaders** (GLSL 450 → SPIR-V): `raster_colormap.frag`, `raster_rgb.frag`, `raster_paletted.frag`

**Core** (`src/core/raster/`): `QgsRasterTileReader` - tile access via `GDALReadBlock()`

**Supported formats**: Byte, UInt16, Float32, RGB/RGBA, Paletted

**Cross-platform**: Metal (macOS), D3D11 (Windows), Vulkan→OpenGL (Linux)

## Renderer Pipeline Integration

**Supported**: RGB, gray, pseudocolor, paletted renderers; brightness/contrast/gamma; hue/saturation/colorize; layer opacity; source NoData.

**CPU fallback**: Reprojection, hillshade, contour, custom nuller, non-tiled rasters.

See [qgsrastergpufactory.cpp](https://github.com/wietzesuijker/QGIS/blob/feat/gpu-raster-qrhi/src/gui/raster/qgsrastergpufactory.cpp) for extraction logic and [testqgsrastergpuparity.cpp](https://github.com/wietzesuijker/QGIS/blob/feat/gpu-raster-qrhi/tests/src/gui/testqgsrastergpuparity.cpp) for 29 GPU/CPU parity tests.

## Risks and Mitigations

| Risk | Mitigation |
|------|------------|
| GPU driver issues | Auto-fallback to CPU on QRhi init failure |
| Thread safety | GPU only on init thread; others use CPU |
| VRAM exhaustion | LRU cache with frame-based eviction |

## Backwards Compatibility

CPU path unchanged. No public API changes. GPU activates automatically when conditions met.

## Open Questions

1. **Benchmark scope**: Include `test_gui_rastergpubenchmark` in merge, or strip to minimal tests?
2. **QgsMapCanvasGPU**: Wire into main canvas now or follow-up?
3. **User toggle**: Add settings option for users with GPU issues?
4. **CI workflow**: Merge benchmark workflow into main QGIS CI?

## Future (out of scope)

- GPU reprojection
- Compute shader analysis (hillshade, slope)
- Progressive tile streaming
- Full GPU compositing (vectors/labels)

## Status

**Branch**: [`feat/gpu-raster-qrhi`](https://github.com/wietzesuijker/QGIS/tree/feat/gpu-raster-qrhi)

**CI**: [GPU Raster Benchmark](https://github.com/wietzesuijker/QGIS/actions/workflows/gpu-raster-benchmark.yml) - Linux (Mesa) + macOS (Metal)

## References

- [Prior PR discussion](https://github.com/qgis/QGIS/pull/64701)
- [Qt QRhi docs](https://doc.qt.io/qt-6/qrhi.html)
