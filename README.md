# FLAN

FLAN stands for `Format Limits Are Relative`.

This project is a browser-first image converter and viewer built around the custom `.8kc` container format. It focuses on palette-reduced images, bit-packed pixel indices, dithering controls, reversible embedding of the original source file, and direct in-browser inspection without converting `.8kc` back to PNG first.

## What It Does

- Converts images into `.8kc` using selectable bit depths from `1` to `13`
- Supports multiple dithering models including:
  - error diffusion
  - ordered Bayer dithering
  - random threshold dithering
  - direct non-dithered quantization
- Supports multiple diffusion and threshold matrices including:
  - `Floyd-Steinberg`
  - `Atkinson`
  - `Jarvis-Judice-Ninke`
  - `Bayer 4x4`
  - `Bayer 8x8`
  - `Random Threshold`
- Includes preprocess controls for:
  - saturation
  - contrast
  - brightness
  - grayscale
  - dither strength
  - noise level
  - dot size
- Allows low-bit custom color workflows for `1-bit`, `2-bit`, and `3-bit`
- Packs palette indices tightly at the selected bit depth before zlib compression
- Stores images in a chunked container:
  - `8KCC` magic
  - `HEAD`
  - `PLET`
  - `META`
  - `DATA`
  - optional `ORIG`
  - `DONE`
- Opens `.8kc` files directly in the browser with CRC validation and chunk inspection
- Can embed the original uploaded file into the `ORIG` chunk for reversible recovery
- Can revert from the converted `.8kc` view back to the embedded original image in the browser when the original payload is browser-decodable
- Uses a retro, pixel-tool inspired UI styled around beveled controls, stone-like panels, and `Pixelify Sans`

## Container Basis

`.8kc` is a custom container format.

It is not built on top of PNG, TIFF, JPEG, RIFF, or another broadly supported metadata container. Instead, it uses its own file signature, `8KCC`, and its own chunk sequence:

- `HEAD`
- `PLET`
- `META`
- `DATA`
- optional `ORIG`
- `DONE`

The format is only inspired by IFF/PNG-style chunk framing:

- 4-byte big-endian chunk length
- 4-byte ASCII chunk type
- chunk payload
- CRC32

So structurally it resembles classic chunked containers, but it is still its own format. That is why general-purpose tools such as ExifTool do not recognize `.8kc` unless explicit support is added.

## Main Components

- [app.py](/root/flan/app.py)
  Flask server for conversion routes, static assets, runtime stats, and `.8kc` export.
- [encoder.py](/root/flan/encoder.py)
  Core image processing, dithering, palette construction, packing, `.8kc` writing, and reversible `ORIG` support.
- [viewer.py](/root/flan/viewer.py)
  Python decoder for `.8kc`, PNG reconstruction, and optional original extraction.
- [core_math.py](/root/flan/core_math.py)
  Bit packing and unpacking utilities used by the container format.
- [public/script.js](/root/flan/public/script.js)
  Browser-side converter UI and export workflow.
- [public/converter-worker.js](/root/flan/public/converter-worker.js)
  Client-side processing pipeline for fast conversions and `.8kc` assembly.
- [public/viewer.js](/root/flan/public/viewer.js)
  Browser-native `.8kc` viewer with chunk parsing, metadata inspection, zoom/pan, and original reversion.

## Reversible Extension

When `Reversible ORIG` is enabled in the converter, the original uploaded file is compressed and embedded as a binary `ORIG` chunk. The viewer can:

- display the converted `.8kc` image
- revert to the embedded original image when browser-decodable
- extract the embedded original file directly

This keeps the reversible payload out of the visible image data while preserving exact recovery.

## Viewer Features

The browser viewer currently supports:

- CRC verification for all chunks
- metadata and chunk inspection
- canvas-based direct rendering
- zoom, fit-to-panel, and pan
- pixel probe inspection
- extraction of embedded `ORIG` payloads
- reversion from converted image view back to the embedded original image when supported by the browser

## Interface Style

The current web interface intentionally uses a retro utility-panel aesthetic inspired by:

- pixel-art tools
- old game editors
- fantasy / medieval game menus
- chunky beveled desktop controls

The styling uses `Pixelify Sans`, square bevels, blue section labels, parchment-and-stone panel tones, and bright active-state button accents.

## Running Locally

## Python Prerequisites

The Python app currently expects:

- Python `3.11+`
- `Flask`
- `Pillow`
- `NumPy`

If you are setting up a fresh environment, the core dependencies are:

```bash
pip install flask pillow numpy
```

If you want process management for background execution:

- `pm2` installed globally on the system

```bash
npm install -g pm2
```

Use the project virtual environment:

```bash
./venv/bin/python app.py
```

If you want to keep it running in the background with `pm2`:

```bash
PM2_HOME=/root/flan/.pm2 pm2 start ./venv/bin/python --name flan-app -- app.py
```

## Project Status

The project currently includes:

- browser-native `.8kc` viewing
- full `.8kc` encode/decode flow
- reversible original embedding
- browser-side fast conversion and `.8kc` assembly
- extended preprocess and dithering controls
- dark mode
- local state persistence
- direct `.8kc` export from the converter
- retro-styled pixel UI

## License / Notice

Footer branding in the app:

This project is licensed under the `Apache License 2.0`.
