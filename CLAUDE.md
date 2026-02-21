# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Interactive Mandelbrot Set explorer — a single self-contained `index.html` file (~1740 lines) with no external dependencies, no build system, and no package manager. Open directly in a browser.

## Architecture

### Renderer Classes (5 backends)

All renderers implement `render(centerX, centerY, zoom, paletteName)` returning `{ maxIter, time }`.

| Renderer | API | Precision | Notes |
|----------|-----|-----------|-------|
| `Canvas2DRenderer` | Canvas 2D | 64-bit native | CPU-based, slowest but full JS double precision |
| `WebGLRenderer` | WebGL 1/2 | 32-bit float | GLSL fragment shaders |
| `WebGLRenderer64` | WebGL 1/2 | 64-bit emulated | Extends WebGLRenderer; Dekker-Veltkamp double-single arithmetic in GLSL |
| `WebGPURenderer` | WebGPU | 32-bit float | WGSL shaders, async init |
| `WebGPURenderer64` | WebGPU | 64-bit emulated | Extends WebGPURenderer; double-single arithmetic in WGSL |

### Double-Single (DS) Precision Emulation

The 64-bit GPU renderers emulate double precision using pairs of 32-bit floats (high + low components). Key operations: `ds_add`, `ds_sub`, `ds_mul`, `ds_sqr`. The JavaScript `splitDouble(v)` function splits a JS double into `[Math.fround(v), v - Math.fround(v)]` for passing to shaders. The split constant is `4097.0` (2^12 + 1).

### State Management

Global `state` object holds viewport (`centerX`, `centerY`, `zoom`), rendering params (`iterBase`, `maxIter`, `palette`), and interaction state. Max iterations scale with zoom: `iterBase + floor(log2(zoom) * 48)`, capped at 8192.

### Renderer Switching

`setRenderer(type)` replaces the canvas element entirely (GPU contexts can't change type on an existing canvas), creates the new renderer, and falls back gracefully on error.

### Color Palettes

Six palettes (fire, ocean, rainbow, electric, grayscale, classic) defined as JS arrays and replicated in both GLSL and WGSL shader code. All use square-root smoothing: `pow(n / maxIter, 0.5)`.

## Development

No build step — edit `index.html` and refresh the browser. For serving locally:

```
python3 -m http.server 8000
```

WebGPU requires HTTPS or localhost; Canvas2D and WebGL work from `file://`.
