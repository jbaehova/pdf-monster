---
name: pdf-monster
description: Analyze PDF files by converting them into agent-readable text, OCR text, page render images, and extracted embedded images without polluting the working directory. Use when an agent needs to read, summarize, inspect, compare, quote, or reason about PDFs, especially when the foundation model cannot ingest PDFs directly or when tables, figures, scans, screenshots, or layout matter.
---

# PDF Monster

## Core Rule

Treat PDF files as source artifacts that must be converted into model-readable evidence before analysis. Do not create `output/`, `analysis/`, `pages/`, or similar folders in the user's working directory unless the user explicitly asks to save extracted artifacts.

Use the bundled `scripts/analyze_pdf.py` first. Resolve this path from the skill root, not from the user's current working directory. It prints JSON to stdout, extracts text in-place, and writes only image artifacts to an OS temporary directory unless `--save-to` is provided.

## Quick Start

Run from this skill directory or replace `scripts/analyze_pdf.py` with its absolute path:

```bash
python3 scripts/analyze_pdf.py path/to/file.pdf --json
```

When the agent is running from another directory, use the absolute installed path. In Claude Code, `${CLAUDE_SKILL_DIR}` points at this skill directory:

```bash
python3 ${CLAUDE_SKILL_DIR}/scripts/analyze_pdf.py path/to/file.pdf --json
```

## Setup

Python 3 is required. Install the recommended Python dependency with:

```bash
python3 -m pip install -r requirements.txt
```

Optional system tools improve coverage when PyMuPDF is unavailable or OCR is needed:

- Poppler: `pdfinfo`, `pdftotext`, `pdftoppm`, `pdfimages`
- Tesseract: `tesseract` plus any needed language data, such as `eng` or `kor`

For visual-heavy or scanned documents:

```bash
python3 scripts/analyze_pdf.py path/to/file.pdf --render-pages all --ocr auto --json
```

For Korean/English OCR, use Tesseract language data and pass:

```bash
python3 scripts/analyze_pdf.py path/to/file.pdf --render-pages all --ocr auto --ocr-lang kor+eng --json
```

For text-only inspection with no temporary image artifacts:

```bash
python3 scripts/analyze_pdf.py path/to/file.pdf --render-pages none --no-extract-images --ocr never --json
```

For selected pages:

```bash
python3 scripts/analyze_pdf.py path/to/file.pdf --pages 1,3-5 --render-pages all --json
```

Persist artifacts only when the user asks for reusable files:

```bash
python3 scripts/analyze_pdf.py path/to/file.pdf --save-to ./pdf-monster-artifacts --json
```

## Reading Workflow

1. Run `analyze_pdf.py` and capture stdout JSON.
2. Check `warnings`, `page_count`, `artifact_root`, and each page's `text_chars`, `ocr_text_chars`, `render_path`, and `embedded_images`.
3. Use `text` as the primary evidence when it is complete enough.
4. Inspect `ocr_text` when a page has little extracted text or appears scanned.
5. Open `render_path` for pages where layout, charts, handwriting, equations, tables, screenshots, or visual placement could change the answer.
6. Open `embedded_images[].path` when the PDF contains standalone figures that may be clearer than the page render.
7. Cite page numbers from the manifest when answering.
8. Delete temporary artifacts after finishing if `artifact_policy` is `temporary`.

The JSON includes a `cleanup_command` such as:

```bash
rm -rf -- /tmp/pdf-monster-...
```

Run it only after the image paths are no longer needed. Never delete a `--save-to` directory unless the user asks.

## Script Behavior

`analyze_pdf.py` prefers PyMuPDF when available. It falls back to Poppler CLI tools where possible:

- `pdftotext` for text extraction
- `pdfinfo` for page count
- `pdftoppm` for page renders
- `pdfimages` for embedded image extraction
- `tesseract` for OCR when installed

OCR is optional. If `tesseract` is missing, report the missing OCR capability and continue with text extraction and rendered page images.

## Other Agent Use

Install this folder where the agent discovers skills, or reference the absolute path in custom instructions:

- Claude Code: `~/.claude/skills/pdf-monster` or `.claude/skills/pdf-monster`
- Codex: `$CODEX_HOME/skills/pdf-monster` or `~/.codex/skills/pdf-monster`
- Pi Coding Agent: `~/.pi/agent/skills/pdf-monster`, `~/.agents/skills/pdf-monster`, or a `skills` settings entry
- OpenClaw: `~/.openclaw/skills/pdf-monster`, `<workspace>/skills/pdf-monster`, or `skills.load.extraDirs`
- Hermes Agent: `~/.hermes/skills/pdf-monster` or `skills.external_dirs`

For agents without a native skill loader, give them this folder and a short instruction:

```text
Use pdf-monster/SKILL.md. For PDF tasks, run:
python3 /absolute/path/to/pdf-monster/scripts/analyze_pdf.py <pdf> --json
Read the JSON from stdout. Use temporary image paths only while analyzing, then clean them up.
Do not create output folders in the user's working directory unless explicitly asked.
```

For OpenCode or similar agents, keep the repository checked out somewhere stable and reference the absolute path to `SKILL.md` or `scripts/analyze_pdf.py` in the agent's custom instructions. The only hard requirement is Python 3. PyMuPDF is the recommended dependency; Poppler and Tesseract improve fallback and OCR coverage.
