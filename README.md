CompressPDF
A Bash script for compressing scanned PDF files to a user-defined target size. Designed to be called directly from the Thunar file manager as a custom action, but works equally well from the terminal.

Why this exists
Ghostscript's built-in presets (/ebook, /screen) often produce an all-or-nothing result: either they barely reduce the file size, or they compress so hard that scanned text becomes unreadable. This script adds smarter logic on top — probing the actual page resolution, estimating the right compression parameters before the first full pass, and iterating adaptively when needed.

Features

Three compression modes suited to different use cases (see Modes)
Automatic size check — exits immediately if the file already fits within the limit
Page resolution probing — rasterizes one page at low DPI to estimate the source resolution and calculate a sensible starting scale, avoiding many unnecessary iterations
Adaptive iteration — recalculates scale after each pass using the actual output size rather than blind stepping
Aspect ratio preserved — uses ffmpeg's -2 height flag so pages are never stretched or squashed
High-resolution scan support — removes the PIL pixel limit that causes img2pdf to abort on large iPhone/scanner images
Thunar integration — pass the file as %f in a custom action; works on a single selected PDF


Modes
ModeHow it worksBest forPreciseGhostscript only — tries /printer, then steps through explicit DPI values (280 → 100) at JPEG quality 92, falls back to /ebook as a last resort. No rasterization.Small reductions (~5–40%). Preserves text sharpness.NormalTries GS /printer then /ebook. If neither fits, falls through to rasterization with pdftoppm + ffmpeg.General purpose. Good balance of quality and compression.AggressiveSkips GS presets entirely. Goes straight to rasterization with lower starting quality and scale.High-resolution phone/scanner images that GS barely touches.

Dependencies
ToolPackagePurposegs (Ghostscript)ghostscriptPDF rewriting and DPI-based image downsamplingpdftoppmpopplerRasterizing PDF pages to JPEGpdfinfopopplerReading page count and metadataffmpegffmpegResizing and recompressing JPEG pagesffprobeffmpegReading image dimensionspython3pythonRunning the img2pdf assembly scriptimg2pdfpython-img2pdf (pip)Assembling JPEG pages into a PDF without re-encodingzenityzenityGUI dialogs (entry, list, notifications)
Install on Arch / Manjaro
