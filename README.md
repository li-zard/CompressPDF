# CompressPDF

A Bash script for compressing scanned PDF files to a user-defined target size. Designed to be called directly from the [Thunar](https://docs.xfce.org/xfce/thunar/start) file manager as a custom action, but works equally well from the terminal.

---

## Why this exists

Ghostscript's built-in presets (`/ebook`, `/screen`) often produce an all-or-nothing result: either they barely reduce the file size, or they compress so hard that scanned text becomes unreadable. This script adds smarter logic on top — probing the actual page resolution, estimating the right compression parameters before the first full pass, and iterating adaptively when needed.

---

## Features

- **Three compression modes** suited to different use cases (see [Modes](#modes))
- **Automatic size check** — exits immediately if the file already fits within the limit
- **Page resolution probing** — rasterizes one page at low DPI to estimate the source resolution and calculate a sensible starting scale, avoiding many unnecessary iterations
- **Adaptive iteration** — recalculates scale after each pass using the actual output size rather than blind stepping
- **Aspect ratio preserved** — uses ffmpeg's `-2` height flag so pages are never stretched or squashed
- **High-resolution scan support** — removes the PIL pixel limit that causes img2pdf to abort on large iPhone/scanner images
- **Thunar integration** — pass the file as `%f` in a custom action; works on a single selected PDF

---

## Modes

| Mode | How it works | Best for |
|------|-------------|----------|
| **Precise** | Ghostscript only — tries `/printer`, then steps through explicit DPI values (280 → 100) at JPEG quality 92, falls back to `/ebook` as a last resort. No rasterization. | Small reductions (~5–40%). Preserves text sharpness. |
| **Normal** | Tries GS `/printer` then `/ebook`. If neither fits, falls through to rasterization with `pdftoppm` + `ffmpeg`. | General purpose. Good balance of quality and compression. |
| **Aggressive** | Skips GS presets entirely. Goes straight to rasterization with lower starting quality and scale. | High-resolution phone/scanner images that GS barely touches. |

---

## Dependencies

| Tool | Package | Purpose |
|------|---------|---------|
| `gs` (Ghostscript) | `ghostscript` | PDF rewriting and DPI-based image downsampling |
| `pdftoppm` | `poppler` | Rasterizing PDF pages to JPEG |
| `pdfinfo` | `poppler` | Reading page count and metadata |
| `ffmpeg` | `ffmpeg` | Resizing and recompressing JPEG pages |
| `ffprobe` | `ffmpeg` | Reading image dimensions |
| `python3` | `python` | Running the img2pdf assembly script |
| `img2pdf` | `python-img2pdf` (pip) | Assembling JPEG pages into a PDF without re-encoding |
| `zenity` | `zenity` | GUI dialogs (entry, list, notifications) |

### Install on Arch / Manjaro

```bash
sudo pacman -S ghostscript poppler ffmpeg python zenity
pip install img2pdf --break-system-packages
```

### Install on Debian / Ubuntu

```bash
sudo apt install ghostscript poppler-utils ffmpeg python3 zenity
pip3 install img2pdf
```

---

## Installation

1. Download or clone the script:

```bash
git clone https://github.com/yourname/CompressPDF.git
cd CompressPDF
```

2. Make it executable:

```bash
chmod +x CompressPDF
```

3. Optionally move it somewhere on your `$PATH`:

```bash
sudo cp CompressPDF /usr/local/bin/
```

---

## Usage

### From the terminal

```bash
./CompressPDF /path/to/file.pdf
```

A dialog will ask for the target size (in MB) and the compression mode. The output file is saved next to the original with a `_compressed` suffix:

```
original.pdf  →  original_compressed.pdf
```

If the script cannot reach the target size after exhausting all iterations, it saves the best result it achieved as `_big.pdf` and shows an error.

### As a Thunar custom action

1. Open Thunar → **Edit → Configure Custom Actions → Add**
2. Fill in:
   - **Name:** Compress PDF
   - **Command:** `/usr/local/bin/CompressPDF %f`
   - **File pattern:** `*.pdf`
   - **Appears if selection contains:** Other files
3. Click **OK**

Right-click any PDF in Thunar and select **Compress PDF**.

---

## How the rasterization path works

When Ghostscript presets are not enough, the script:

1. Rasterizes page 1 at **72 DPI** (fast, low memory) to measure the real page dimensions in pixels.
2. Estimates the source DPI by comparing the bytes-per-page to the expected pixel count at 72 DPI.
3. Calculates a **starting scale** using the formula:
   `scale = sqrt(target_bytes / (width × height × pages × jpeg_factor × 1.05))`
4. Runs a full rasterization + ffmpeg pass with those parameters.
5. If the result still exceeds the target, recalculates scale as:
   `new_scale = old_scale × sqrt(target / actual) × 0.92`
   and reduces quality by 4 points, then repeats — up to 12 iterations.

This means for a typical high-resolution scan the correct parameters are found in 1–2 passes rather than 5–10.

---

## Limitations

- Works on single-file selections only (one PDF at a time).
- The rasterization path converts pages to JPEG, so it is not suitable for PDFs with vector artwork or text layers that need to remain selectable.
- Very low target sizes relative to content may result in quality loss regardless of mode.

---

## License

This project is licensed under the **MIT License** — see [LICENSE](LICENSE) for the full text.

MIT was chosen because the script is a small utility with no complex dependencies of its own, and MIT imposes the fewest restrictions on reuse, modification, and redistribution while still requiring attribution.

---

## Contributing

Bug reports and pull requests are welcome. If a particular type of PDF consistently produces poor results, opening an issue with the file size, page count, and approximate source DPI is the most helpful thing you can do.
