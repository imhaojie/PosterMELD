<div align="center">
  <img src="docs/readme/postermeld-logo.svg" width="118" alt="PosterMELD logo" />

  <h1>PosterMELD</h1>

  <p><strong>Multi-Agent Paper-to-Poster Generation for Controllable Design Diversity<br />with Editable Print-Ready Outputs</strong></p>

  <p>
    <a href="https://imhaojie.github.io/PosterMELD/"><strong>Project Page</strong></a>
    · <a href="https://arxiv.org/abs/2608.02218"><strong>Paper</strong></a>
    · <a href="poster_generation/"><strong>Poster Generation</strong></a>
    · <a href="benchmark_eval/"><strong>Benchmark Evaluation</strong></a>
    · <a href="skills/postermeld/"><strong>Agent Skill</strong></a>
    · <a href="#quick-start"><strong>Quick Start</strong></a>
  </p>

  <p>
    <a href="https://arxiv.org/abs/2608.02218"><img src="https://img.shields.io/badge/arXiv-2608.02218-B31B1B?style=flat-square&logo=arxiv&logoColor=white" alt="arXiv 2608.02218" /></a>
    <img src="https://img.shields.io/badge/Python-3.11-3776AB?style=flat-square&logo=python&logoColor=white" alt="Python 3.11" />
    <img src="https://img.shields.io/badge/Output-Editable_PPTX-0B7A75?style=flat-square&logo=microsoftpowerpoint&logoColor=white" alt="Editable PPTX output" />
    <img src="https://img.shields.io/badge/Templates-24-7256B8?style=flat-square" alt="24 templates" />
    <img src="https://img.shields.io/badge/Benchmark-621_papers-D99A27?style=flat-square" alt="621-paper benchmark" />
    <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-17233A?style=flat-square" alt="MIT License" /></a>
  </p>
</div>

<p align="center">
  <img src="docs/readme/teaser.png" width="100%" alt="PosterMELD qualitative comparison" />
</p>

PosterMELD generates **editable PowerPoint posters** and PNG previews from scientific papers. It selects a template, writes sections to fit the layout, places figures and tables from the paper, and checks the result before export.

## Repository Modules

The two modules run independently:

| Module | Purpose | Entry point |
|---|---|---|
| [`poster_generation/`](poster_generation/) | PDF-to-PPTX/PNG generation | `python -m src.workflow.pipeline` or `postermeld` |
| [`benchmark_eval/`](benchmark_eval/) | PRR/CHE, Universal Score, and Keypoint BERTScore evaluation | `python -m prr_che.evaluate`, `python -m universal_score.evaluate`, and `python -m keypoint_bertscore.*` |

[`skills/postermeld/`](skills/postermeld/) lets Codex and Claude Code install and run the generation module.

[`docs/`](docs/) contains the project website and paper figures.

## Highlights

- **Multi-agent generation:** agents share paper content and layout state to write sections, place visuals, and review the poster.
- **Editable outputs:** text, figures, tables, section bars, and logos are separate PowerPoint objects.
- **Design controls:** 16 landscape and 8 portrait templates, with style, density, background, header, and seed settings.
- **PDF extraction:** MinerU extracts text and visual assets, with Marker as a fallback.
- **Quality checks:** geometry checks and VLM review identify problems for targeted repair.
- **Evaluation:** PRR, conditional CHE, Universal Score, and Keypoint BERTScore run in a separate package.

### Results

| Method | PRR (%) ↑ | CHE ↑ | Universal ↑ | Editable | Cost / request ↓ |
|---|---:|---:|---:|:---:|---:|
| GPT-Image-2 | **85.2** | 2.698 | **4.948** | No | **$0.18** |
| Codex+Skill | 82.8 | 2.716 | 4.876 | Yes | $10.78 |
| Paper2Poster | 0.2 | - | 2.772 | Yes | $0.34 |
| P2P | 24.2 | 3.071 | 4.033 | No | $0.35 |
| PosterGen | 15.8 | 3.163 | 3.901 | Yes | $0.28 |
| **PosterMELD** | **81.3** | **3.247** | **4.456** | **Yes** | **$0.38** |
| Human reference | 98.7 | 3.287 | 4.995 | Yes | - |

PRR is computed over all requests. CHE is conditional on print-ready outputs, while Universal Score keeps missing generations in the full-benchmark aggregate.

## Method

<p align="center">
  <img src="docs/readme/framework.png" width="100%" alt="PosterMELD generation pipeline" />
</p>

Generation steps:

1. Parse the PDF into grounded text, figures, tables, formulas, and metadata.
2. Select a template and expose slot geometry, reading order, and capacity budgets.
3. Distill poster keypoints and compose sections within those budgets.
4. Place visuals, render an editable PPTX, and produce a consistent PNG preview.
5. Apply local and global quality checks, followed by bounded repair when needed.

## Quick Start

### Agent Skill

Install the lightweight Skill in Codex:

```bash
python ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo Shannon4Science/PosterMELD \
  --path skills/postermeld
```

For Claude Code, place the same [`skills/postermeld/`](skills/postermeld/) directory at `~/.claude/skills/postermeld`.

Then ask the host agent:

```text
Use $postermeld to generate a polished editable poster from /absolute/path/paper.pdf.
Use the complete visual pipeline with teaser, background, logos, and VLM review.
```

The Skill checks the environment, runs PosterMELD, and returns a PNG and editable PPTX. Configure the model, VLM, image-generation, and MinerU services in [`poster_generation/.env.example`](poster_generation/.env.example) to use the complete pipeline.

### Manual generation

```bash
git clone https://github.com/Shannon4Science/PosterMELD.git
cd PosterMELD/poster_generation

python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
pip install -e . --no-deps
cp .env.example .env

postermeld data/0409_demo/paper.pdf \
  --layout-template auto \
  --disable-generated-teaser \
  --disable-generated-background
```

Enable the complete visual pipeline after configuring the VLM, image-generation, and MinerU endpoints in `.env`:

```bash
postermeld /path/to/paper.pdf \
  --layout-template auto \
  --poster-style navy_serif \
  --visual-density rich \
  --enable-generated-teaser \
  --enable-generated-background \
  --enable-vlm-layout-review \
  --enable-visual-legibility-review
```

See the [generation guide](poster_generation/README.md) for all controls, output files, templates, and backend configuration.

### Evaluate generated posters

```bash
cd ../benchmark_eval
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env

python -m prr_che.evaluate \
  --manifest examples/manifest.jsonl \
  --output-dir outputs/prr_che \
  --model gpt-5.5
```

See the [evaluation guide](benchmark_eval/README.md) for manifest schemas, aggregation rules, GPU evaluation, and reproducibility details.

## Benchmark

<p align="center">
  <img src="docs/readme/benchmark-composition.png" width="72%" alt="Composition of the PosterMELD benchmark" />
</p>

The benchmark contains 621 papers from 14 publication-source groups across 10 research domains. Request-level metrics include missing generations; conditional visual-quality metrics use only print-ready outputs.

| Metric | Scope |
|---|---|
| **PRR** | Binary request-level print readiness |
| **CHE** | Craftsmanship, Harmony, and Expressiveness for print-ready outputs |
| **Universal Score** | Ten reference-aware poster criteria with missing outputs scored as zero |
| **Keypoint BERTScore** | OCR-based content coverage against ordered paper keypoints |

<p align="center">
  <img src="docs/readme/case-study.png" width="100%" alt="Qualitative poster comparison across five papers" />
</p>

## Project Layout

```text
PosterMELD/
├── skills/postermeld/        Codex and Claude Code interface to the full runtime
├── poster_generation/        complete generation subproject
│   ├── assets/               conference and bundled visual assets
│   ├── config/               prompts and pipeline configuration
│   ├── data/                 runnable example papers
│   ├── scripts/              batch-generation utilities
│   ├── src/                  agents, layout, tools, state, and workflow
│   ├── templates/            16 landscape + 8 portrait templates
│   ├── tests/                generation regression tests
│   └── requirements.txt
├── benchmark_eval/           standalone benchmark-evaluation subproject
│   ├── common/               manifest, API, image, and JSON utilities
│   ├── prr_che/              print-ready and conditional aesthetic metrics
│   ├── universal_score/      reference-aware universal evaluation
│   ├── keypoint_bertscore/   OCR and content-coverage evaluation
│   ├── examples/             manifest and annotation examples
│   └── tests/                offline evaluation tests
├── docs/                     GitHub Pages project site and paper figures
├── Makefile                  repository-level validation commands
└── LICENSE
```

## Validation

Run both offline validation suites from the repository root:

```bash
make test
```

Or validate each module independently:

```bash
make test-generation
make test-evaluation
```

## License

PosterMELD is released under the [MIT License](LICENSE).
