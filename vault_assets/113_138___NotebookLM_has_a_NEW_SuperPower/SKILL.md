---
name: notebooklm-powerpoint
description: >
  Converts NotebookLM slide decks into editable PowerPoint (.pptx) files and
  other formats using the NotebookLM MCP and Python. Use when the user asks to
  "download slides as PowerPoint", "convert NotebookLM slides to pptx",
  "export slides from NotebookLM", "turn slides into PowerPoint", "save slides
  as images", or mentions NotebookLM slide conversion, deck export, or
  presentation download.
---

# NotebookLM PowerPoint Converter

## When to use this skill
- User asks to "download slides as PowerPoint" or "convert to pptx"
- User says "export my NotebookLM slides" or "save slides as PowerPoint"
- User wants to edit NotebookLM-generated slides in PowerPoint or Keynote
- User asks to "export slides as images" or "save each slide as PNG"
- User mentions NotebookLM slide conversion or deck export

## How It Works

NotebookLM generates slide decks as **PDF files** — even when downloaded via the MCP.
This skill bridges that gap by:

1. **Downloading** the slide deck PDF from NotebookLM via the MCP (no browser needed)
2. **Converting** each PDF page to a high-resolution image
3. **Packaging** the images into the target format (PPTX, PNG, HTML, etc.)

Everything runs silently in the background — **no Chrome popups**.

## Prerequisites

- **NotebookLM MCP** must be connected and authenticated (`nlm login`)
- **Python virtual environment** with required packages at `/tmp/pptx-env/`
- **Poppler** for PDF rendering (installed via Homebrew: `brew install poppler`)

### First-Time Setup

If the virtual environment doesn't exist, create it:

```bash
uv venv /tmp/pptx-env && source /tmp/pptx-env/bin/activate && uv pip install pdf2image python-pptx Pillow pymupdf
```

If `poppler` is not installed:

```bash
brew install poppler
```

## Supported Output Formats

| Format | Extension | Description |
|--------|-----------|-------------|
| **PowerPoint** | `.pptx` | Editable presentation, opens in PowerPoint/Keynote/Google Slides |
| **Images** | `.png` | Individual slide images at 200 DPI |
| **PDF** | `.pdf` | Original format from NotebookLM (direct download, no conversion) |
| **Google Slides** | N/A | Via `export_artifact` MCP tool → pushes to Google Docs |
| **HTML** | `.html` | Self-contained web slideshow with navigation |

## Workflow

- [ ] Identify the notebook ID (ask user or use `notebook_list`)
- [ ] List studio artifacts with `studio_status` to find the slide deck
- [ ] Download the slide deck PDF using `download_artifact`
- [ ] Determine the target format (PPTX by default)
- [ ] Run the conversion script
- [ ] Deliver the output file to the user

## Instructions

### Step 1: Find the Slide Deck

```python
# List notebooks
mcp_notebooklm_notebook_list(max_results=20)

# Get artifacts for a specific notebook
mcp_notebooklm_studio_status(notebook_id="<NOTEBOOK_ID>")

# Look for artifacts with type: "slide_deck" and status: "completed"
```

### Step 2: Download as PDF

```python
mcp_notebooklm_download_artifact(
    notebook_id="<NOTEBOOK_ID>",
    artifact_type="slide_deck",
    output_path="/path/to/output/slides.pdf"
)
```

> **Note:** Even if you specify `.pptx` as the output path, the MCP delivers a PDF.
> Always save as `.pdf` first, then convert.

### Step 3: Convert to Target Format

Use the helper script for the conversion:

```bash
source /tmp/pptx-env/bin/activate && python3 .agent/skills/notebooklm-powerpoint/scripts/convert.py \
  --input "/path/to/slides.pdf" \
  --output "/path/to/output/slides.pptx" \
  --format pptx \
  --dpi 200
```

#### Quick Inline Conversion (if script unavailable)

For a quick PPTX conversion without the script:

```python
source /tmp/pptx-env/bin/activate && python3 -c "
from pdf2image import convert_from_path
from pptx import Presentation
from pptx.util import Inches
import os, tempfile

pdf_path = '<INPUT_PDF>'
pptx_path = '<OUTPUT_PPTX>'

images = convert_from_path(pdf_path, dpi=200)
prs = Presentation()
prs.slide_width = Inches(13.333)
prs.slide_height = Inches(7.5)

for img in images:
    slide = prs.slides.add_slide(prs.slide_layouts[6])
    tmp = tempfile.NamedTemporaryFile(suffix='.png', delete=False)
    img.save(tmp.name, 'PNG')
    slide.shapes.add_picture(tmp.name, Inches(0), Inches(0), prs.slide_width, prs.slide_height)
    os.unlink(tmp.name)

prs.save(pptx_path)
print(f'Saved: {pptx_path} ({os.path.getsize(pptx_path) / 1024 / 1024:.1f} MB)')
"
```

### Step 4: Export to Google Slides (Alternative)

If the user wants Google Slides instead of PPTX:

```python
# Get the artifact ID from studio_status first
mcp_notebooklm_export_artifact(
    notebook_id="<NOTEBOOK_ID>",
    artifact_id="<ARTIFACT_ID>",
    export_type="docs",
    title="My Presentation"
)
```

### Step 5: Generate Slide Images Only

If the user just wants individual slide images:

```bash
source /tmp/pptx-env/bin/activate && python3 .agent/skills/notebooklm-powerpoint/scripts/convert.py \
  --input "/path/to/slides.pdf" \
  --output "/path/to/output_folder/" \
  --format png \
  --dpi 300
```

## Error Handling

| Error | Meaning | Action |
|-------|---------|--------|
| `Authentication expired` | MCP tokens need refresh | Run `mcp_notebooklm_refresh_auth()` or `nlm login` |
| `No slide_deck artifacts found` | Notebook has no generated slides | Use `studio_create` to generate a slide deck first |
| `poppler not installed` | pdf2image needs poppler | Run `brew install poppler` |
| `Virtual env missing` | Python packages not installed | Run the First-Time Setup commands |
| `PDF is empty / 0 pages` | Download failed silently | Re-download with a fresh `download_artifact` call |

## Output Format

Present results clearly:

**Single deck conversion:**
> ✅ Converted **14 slides** → `Anti-Gravity_Frontiers.pptx` (59 MB)
> 📂 Saved to: `/Users/jackroberts/Antigravity Skills/Anti-Gravity_Frontiers.pptx`

**Image export:**
> ✅ Exported **14 slides** as PNG images (300 DPI)
> 📂 Saved to: `/Users/jackroberts/Antigravity Skills/slides/`
> - `slide_01.png`, `slide_02.png`, ... `slide_14.png`

## Cleanup

After conversion, optionally remove the intermediate PDF:

```bash
rm "/path/to/slides.pdf"
```

## Limitations

- Slides are exported as **images inside PPTX** — text is not individually editable
  (it preserves the exact visual layout from NotebookLM)
- For text-editable slides, use **Google Slides export** via `export_artifact`
- The MCP cannot edit individual slides — use NotebookLM's web UI for prompt-based slide editing
- PPTX file sizes are large (~4-5 MB per slide at 200 DPI) due to image-based rendering
