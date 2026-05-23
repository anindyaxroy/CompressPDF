# pdf-compressor

A Jupyter notebook that compresses large PDF files to meet AI platform file-size limits, with no manual configuration beyond pointing it at a folder.

## Why

AI platforms impose per-file upload limits (Claude: 10 MB, others vary). Lecture slides and textbooks routinely exceed this. Existing tools either require complex CLI setup (Ghostscript) or produce negligible results (lossless-only libraries). This notebook chains four methods automatically — best first, fallback on failure — and reports exactly what happened to each file.

## Features

- **Zero-config path detection** — leave `PDF_SOURCE = None` and it processes PDFs in the same folder as the notebook
- **Four-method pipeline** — Ghostscript → pymupdf → pikepdf → PyPDF2, stopping as soon as a method reduces the file
- **Auto-installs Python dependencies** — `pymupdf`, `pikepdf`, `PyPDF2` installed at runtime if missing
- **Verbose per-method output** — shows exactly which methods succeeded or failed and why
- **Configurable thresholds** — set any size limit, DPI, and JPEG quality from a single config cell
- **Non-destructive** — originals are never modified; compressed copies go to a `compressed/` subfolder

## Compression methods

| Priority | Method | How it works | Typical reduction |
|----------|--------|-------------|-------------------|
| 1 | **Ghostscript** | Re-renders + resamples all images at target DPI | 70–90% |
| 2 | **pymupdf** | Re-renders pages as JPEG at target DPI (pure Python) | 60–85% |
| 3 | **pikepdf** | Lossless stream recompression + object deduplication | 1–5% |
| 4 | **PyPDF2** | Content stream compression per page | <1% |

Ghostscript gives the best results but requires a separate OS-level install. pymupdf is the recommended pure-Python fallback and handles image-heavy slide decks very well.

## Requirements

**Python packages** (auto-installed by the notebook):
```
pymupdf
pikepdf
PyPDF2
```

**Optional — Ghostscript** (for maximum compression):
- Windows: download from https://www.ghostscript.com/releases/gsdnld.html (tick "Add to PATH")
- macOS: `brew install ghostscript`
- Linux: `sudo apt install ghostscript`

## Quick start

1. Clone the repo or download the notebook
   ```bash
   git clone https://github.com/your-username/pdf-compressor.git
   cd pdf-compressor
   ```
2. Install dependencies (or let the notebook do it for you):
   ```bash
   pip install -r requirements.txt
   ```
3. Open `pdf_compressor.ipynb` in Jupyter or VS Code
4. Edit the config cell:
   ```python
   PDF_SOURCE = None        # None = same folder as notebook, or set a path
   SIZE_LIMIT_MB = 10.0     # adjust to your platform's limit
   PYMUPDF_DPI = 120        # lower for smaller files (try 96)
   PYMUPDF_JPEG_QUALITY = 75  # lower for smaller files (try 60)
   ```
5. **Kernel → Restart & Run All**

Compressed files are saved to `compressed/` inside your PDF folder.

## Configuration reference

| Variable | Default | Description |
|----------|---------|-------------|
| `PDF_SOURCE` | `None` | Path to PDF folder. `None` = notebook's current directory |
| `SIZE_LIMIT_MB` | `10.0` | Target size limit in MB |
| `COMPRESS_THRESHOLD_MB` | `5.0` | Skip files already smaller than this |
| `PYMUPDF_DPI` | `120` | Render DPI for pymupdf (lower = smaller) |
| `PYMUPDF_JPEG_QUALITY` | `75` | JPEG quality for pymupdf 0–100 (lower = smaller) |

## Example output

```
[COMPRESS]  lecture_01.pdf  (22.15 MB) ...
    ✗ Ghostscript: NOT FOUND
    ✓ pymupdf: 22.15 → 3.84 MB (82.7% saved)
done  [pymupdf]  22.15 → 3.84 MB  (-82.7%)

===============================================================================================
File                                                    Before    After  Saved%  Method    OK?
===============================================================================================
lecture_01.pdf                                           22.15     3.84  -82.7%  pymupdf   YES
lecture_02.pdf                                           33.67     5.21  -84.5%  pymupdf   YES
===============================================================================================
All processed files are within the 10.0 MB limit.
```

## Contributing

Pull requests are welcome. To add a new compression backend, implement a function with the signature:

```python
def compress_mymethod(input_path: pathlib.Path, output_path: pathlib.Path) -> tuple[bool, str]:
    ...
    return True, "mymethod"   # or False, "error message"
```

Then add it to the `attempts` list in `compress_pdf()`.

## License

MIT — see [LICENSE](LICENSE).
