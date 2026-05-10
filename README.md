# PDF Monster

Agent skill for turning PDFs into model-readable evidence: extracted text, optional OCR text, page render images, and embedded image files.

PDF Monster is built for coding agents that need to inspect PDFs without dumping generated folders into the user's project. By default, it writes image artifacts to an OS temporary directory and prints a JSON manifest to stdout.

## What It Does

- Extracts per-page text from PDFs
- Renders pages to PNG when layout or visual inspection matters
- Runs optional OCR through Tesseract
- Extracts embedded images for figures and screenshots
- Emits a structured JSON manifest for agents to read
- Avoids creating `output/`, `pages/`, or similar folders unless explicitly requested

## Install

Clone or copy this folder into a skill directory supported by your agent:

```bash
git clone <repo-url> pdf-monster
```

Python 3 is required. The recommended Python dependency is installed by the agent on first use when it is missing:

```bash
python3 -m pip install -r /absolute/path/to/pdf-monster/requirements.txt
```

If you run the CLI yourself, install it once from the repo root:

```bash
python3 -m pip install -r requirements.txt
```

Optional system tools are not installed automatically:

- Poppler: `pdfinfo`, `pdftotext`, `pdftoppm`, `pdfimages`
- Tesseract: `tesseract` plus language data such as `eng` or `kor`

## Use As An Agent Skill

Install the whole folder, not just `SKILL.md`, because the skill uses `scripts/analyze_pdf.py`.

Common locations:

```text
Claude Code:   ~/.claude/skills/pdf-monster
Codex:         ~/.codex/skills/pdf-monster
Pi:            ~/.pi/agent/skills/pdf-monster or ~/.agents/skills/pdf-monster
OpenClaw:      ~/.openclaw/skills/pdf-monster or <workspace>/skills/pdf-monster
Hermes:        ~/.hermes/skills/pdf-monster
```

For agents without native skill discovery, point custom instructions at the absolute path to `SKILL.md` and tell the agent to run:

```bash
python3 /absolute/path/to/pdf-monster/scripts/analyze_pdf.py <file.pdf> --json
```

## CLI Usage

Basic analysis:

```bash
python3 scripts/analyze_pdf.py file.pdf --json
```

Visual or scanned PDFs:

```bash
python3 scripts/analyze_pdf.py file.pdf --render-pages all --ocr auto --json
```

Korean and English OCR:

```bash
python3 scripts/analyze_pdf.py file.pdf --render-pages all --ocr auto --ocr-lang kor+eng --json
```

Text-only mode:

```bash
python3 scripts/analyze_pdf.py file.pdf --render-pages none --no-extract-images --ocr never --json
```

Selected pages:

```bash
python3 scripts/analyze_pdf.py file.pdf --pages 1,3-5 --render-pages all --json
```

Persist artifacts when you actually want files kept:

```bash
python3 scripts/analyze_pdf.py file.pdf --save-to ./pdf-monster-artifacts --json
```

## Output

The script prints JSON with fields such as:

- `page_count`
- `selected_pages`
- `backend`
- `artifact_root`
- `cleanup_command`
- `pages[].text`
- `pages[].ocr_text`
- `pages[].render_path`
- `pages[].embedded_images`
- `pages[].warnings`

If temporary artifacts are created, the JSON includes a `cleanup_command`. Run it only after the image paths are no longer needed.

## Notes

PyMuPDF is the preferred backend. If it is unavailable, PDF Monster falls back to Poppler CLI tools where possible. OCR is optional; when Tesseract is missing, the script reports a warning and continues with text extraction and page rendering.

## License

MIT. See [LICENSE](LICENSE).
