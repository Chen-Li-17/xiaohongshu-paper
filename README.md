# OpenClaw Paper-to-Xiaohongshu Automation Pipeline

This repository packages a reusable automation workflow: parse papers, standardize them into a paper card, generate Xiaohongshu post copy, and execute platform operations.

The canonical flow is: `paper-parse -> paper-card-analyzer -> xhs-post-factory -> xiaohongshu-operate`.

## 1. Pipeline Overview

### 1.1 `paper-parse` (Automated paper parsing)
- Purpose: parse academic PDFs into structured text and image assets.
- Key capability: extracts both text and images.
- Typical outputs:
  - `{paper_name}_content.md`: full paper content in Markdown.
  - `{paper_name}_parsed.json`: structured metadata (title, pages, figures, etc.).
  - `cover_title_authors.png`: first-page snapshot focused on title/author area.
  - `figures/figure_*.png`: high-resolution extracted figures.

This stage builds the evidence layer for downstream steps (text evidence + visual evidence).

### 1.2 `paper-card-analyzer` (Standardized paper card)
- Purpose: consume `paper-parse` artifacts and generate a research-oriented standard paper card.
- Fixed outputs:
  - `paper-card.md`
  - `paper-card.json`
  - `paper-card-feedback.md`
- Fixed section order (10 sections):
  1. Paper Snapshot
  2. Research Problem and Motivation
  3. Core Contributions
  4. Method Overview
  5. Experimental Setup
  6. Main Results and Evidence
  7. Ablation and Analysis Findings
  8. Limitations and Threats to Validity
  9. Reproducibility Notes
  10. Open Questions and Future Work

The value of this normalized format is cross-platform distribution readiness: the same structured content can be mapped to Xiaohongshu, WeChat, X/Twitter, LinkedIn, or internal knowledge systems.

### 1.3 `xhs-post-factory` (Xiaohongshu copy generation)
- Purpose: convert `pdf/md/txt/json` inputs into structured Xiaohongshu post copy (v1 defaults to paper interpretation template).
- Default outputs:
  - `xhs-post.md`
  - `xhs-post.json`
- Template routing:
  - If `post_type` is explicitly specified, use it first.
  - Otherwise route by intent keywords.
  - v1 defaults to `paper-interpretation`.
- Configurable parameters (currently supported):
  - `length`: `short` / `medium` / `long`
  - `emoji_density`: `low` / `medium` / `high`
- Reliability constraints:
  - No fabricated metrics/datasets/claims.
  - If evidence is missing, explicitly mark “not clearly stated in source”.
  - Use `evidence_notes` to trace section-level evidence.

This stage enables multi-version publishing from the same source material by changing template and style parameters.

### 1.4 `xiaohongshu-operate` (Xiaohongshu operation execution)
- Purpose: execute platform actions using provided materials; it is an operations executor, not a content rewriter.
- Multi-module capability (core feature):
  - Publishing module: image post / long-form / video, draft-first by default.
  - Interaction module: comment check, reply draft, confirm-before-send.
  - Search module: keyword crawl for Top N posts with standardized fields.
- Risk handling and fallback:
  - Stop high-risk actions and report resumable state when page structure changes, rate limits, or asset gaps occur.
  - Apply persona constraints for comments/DMs to keep tone bounded.

## 2. End-to-End Data Flow

```text
PDF
  -> paper-parse
     -> *_content.md + *_parsed.json + figures/*
  -> paper-card-analyzer
     -> paper-card.md + paper-card.json + paper-card-feedback.md
  -> xhs-post-factory
     -> xhs-post.md + xhs-post.json
  -> xiaohongshu-operate
     -> drafts / comment-operation logs / search-crawl results
```

I/O handoff rules:
- `xhs-post-factory` preferentially reuses same-folder `paper-card.*` and `paper-parse` artifacts.
- `xiaohongshu-operate` consumes finalized materials and avoids factual rewriting.

## 3. Quick Start

### 3.1 Repository layout
- `paper-parse/`: parsing (text + figure extraction)
- `paper-card-analyzer/`: standardized paper card generation
- `xhs-post-factory/`: Xiaohongshu copy factory
- `xiaohongshu-operate/`: Xiaohongshu operation execution

### 3.2 Minimal execution order
1. Parse the paper (produce `*_content.md`, `*_parsed.json`, `figures/`).
2. Generate paper card (`paper-card.md/json`).
3. Generate Xiaohongshu copy (`xhs-post.md/json`).
4. Execute draft posting, comment operations, or search crawling on platform.

### 3.3 Example command (parse stage)
```bash
uv run paper-parse/scripts/parse_paper.py --pdf /path/to/paper.pdf --output-dir ./parsed
```

## 4. Design Principles and Extensibility

This workflow is highly extensible and can be migrated to automated summarization and operation for multiple content types across multiple social platforms:

- Content expansion:
  - Extend from papers to reports, whitepapers, lecture notes, meeting transcripts, podcast transcripts, etc.
- Template expansion:
  - Add new templates under `xhs-post-factory/templates/` to support new post styles.
- Platform expansion:
  - Abstract `xiaohongshu-operate` into a platform interface and add operators for WeChat, Bilibili, X/Twitter, LinkedIn, etc.
- Workflow expansion:
  - Use `paper-card` as a normalized intermediate layer for retrieval systems, knowledge bases, CMS pipelines, and A/B publishing systems.

## 5. Reliability and Governance Recommendations

- Keep the full chain evidence-driven: every claim should be traceable to parsed text or structured fields.
- Distinguish “author-reported findings” from “operator interpretation”.
- Mark missing information explicitly; do not silently fill gaps.
- Keep draft-first as the default platform behavior to reduce accidental publishing risk.

---

If you want to productionize this pipeline, prioritize adding scheduling, observability, multi-account permissions/audit, and robust retry policies.
