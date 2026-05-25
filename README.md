# BattleshipGraphicsProjects

An umbrella repository for the experiments and tooling that turn historical warship illustrations into 3D-ready assets. It groups together several independent sub-projects (image extraction, image-to-3D pipelines, a Unity MCP integration, and reference scans) under one tree.

The two big pillars right now are:

1. **Florence-2 Warship Extractor** (this repo's `src/`) — extract warship illustrations from historical PDFs.
2. **ProjectBroadside** — the downstream pipelines that take those images and produce 3D meshes / Unity content.

## Repository layout

| Path | What it is |
| --- | --- |
| `src/warship_extractor/` | Python package: Florence-2-based extractor (`cli.py`, `pipeline/`, `detection/`, `processing/`, `core/`, `config/`, `utils/`). |
| `tests/` | Pytest suite (`unit/`, `integration/`) for the warship extractor. |
| `pyproject.toml` / `poetry.lock` | Poetry config for the extractor. Python `>=3.9,<3.12`. |
| `FLORENCE_2_WARSHIP_EXTRACTION_PLAN.md` | Architectural plan for the extractor. |
| `Docs/` | Cross-project documentation (2D pipeline, strategy notes, ComfyUI workflow prompts, archive). |
| `Scans/` | Source PDF / scanned material used as input. |
| `BAttleships/` | Reference warship images. |
| `ProjectBroadside/` | The 3D-asset side of the project (see below). |

### ProjectBroadside subprojects

| Path | What it is |
| --- | --- |
| `ProjectBroadside/BattleshipMaker/` | First-iteration 2D→3D pipeline (Python; uses Gemini for view-detection / vectorization). |
| `ProjectBroadside/BattleshipMaker2/` | Reworked staged pipeline: image generation → dataset prep → 3DGS training → splat refinement → mesh conversion → final output. |
| `ProjectBroadside/HullCutter/` | Placeholder / stub for hull-extraction tooling. |
| `ProjectBroadside/MCPUnityRockstar/` | MCP Unity Editor integration (Node.js MCP server + Unity Editor package). |
| `ProjectBroadside/PDFScans/` | Scanned PDF inputs used by the 3D pipelines. |
| `ProjectBroadside/ProjectBroadside.Scripts/` | Unity C# scripts (gameplay, ECS, authoring, components, core, data). |

## Florence-2 Warship Extractor

A pipeline for pulling warship illustrations out of Jane's Fighting Ships and similar naval archives, using Microsoft's Florence-2 vision-language model.

### Highlights

- Florence-2-Large for object detection
- Multi-prompt strategy to catch different illustration styles
- High-resolution PDF rasterization (300+ DPI by default)
- Non-Maximum Suppression to drop duplicate detections
- CUDA-aware with CPU fallback; dynamic batch sizing
- CLI (`warship-extract`) for extract / batch / report / info

### Install

Requires Python 3.9–3.11 and [Poetry](https://python-poetry.org). A CUDA-compatible GPU is recommended.

```bash
poetry install
poetry shell
```

### CLI

```bash
# Extract from a single PDF
warship-extract extract input.pdf --output-dir results/

# Batch process a folder of PDFs
warship-extract batch pdf_folder/ --output-dir results/

# Generate an analysis report
warship-extract report results/ --format html

# System / environment info
warship-extract info
```

### Python API

```python
from warship_extractor.pipeline import ExtractionPipeline
from warship_extractor.config import Settings

settings = Settings()
pipeline = ExtractionPipeline(settings)
results = pipeline.extract_from_pdf("janes_1900.pdf")

for detection in results["detections"]:
    print(f"Found {detection['label']} with confidence {detection['confidence']:.2f}")
```

### Configuration

The extractor reads settings from environment variables or `Settings`:

```bash
FLORENCE_MODEL_NAME="microsoft/Florence-2-large"
FLORENCE_DEVICE="cuda"
PDF_DPI=300
DETECTION_CONFIDENCE_THRESHOLD=0.7
OUTPUT_DIR="./extracted_warships"
SAVE_ANNOTATED_IMAGES=true
```

### Tests

```bash
poetry run pytest
poetry run pytest --cov=src/warship_extractor
poetry run pytest tests/unit/test_detector.py
```

`pyproject.toml` already wires up `pytest --cov` against `src/warship_extractor` by default.

## ProjectBroadside

`ProjectBroadside/` is a heterogeneous workspace, not a single Python package:

- **`BattleshipMaker/`** — earlier Python experiment that takes 2D ship views (side / top), splits multi-view images, and tries to produce a base 3D mesh. See `BattleshipMaker/Prompt.md` and `framework_proposal.md`.
- **`BattleshipMaker2/`** — current staged pipeline using image generation, 3D Gaussian Splatting training, splat refinement, and mesh conversion. See `BattleshipMaker2/README.md` and `config.yaml`.
- **`MCPUnityRockstar/`** — a Model Context Protocol server that exposes the Unity Editor as tools to MCP-aware clients (Node-based server in `Server~/` plus a Unity `Editor/` package).
- **`ProjectBroadside.Scripts/`** — the Unity-side C# code (ECS, gameplay, authoring, components).

Each subproject has its own README / setup notes; this top-level README only orients you.

## Status

Active, multi-project, exploratory. Sub-projects move at different speeds — treat the per-subproject documentation as the source of truth for whichever pipeline you're working in.
