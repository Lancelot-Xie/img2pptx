# img2pptx

Turn a raster diagram into an editable, modular, and auditable PowerPoint slide.

Create a diagram with GPT Image 2 or another AI image-generation tool, then use
`img2pptx` to reconstruct the PNG, JPG, WebP, screenshot, scientific figure, or
architecture diagram as a one-slide PPTX with a complete embedded SVG.

```text
AI-generated diagram → img2pptx → editable PPTX + SVG + modules + QA
```

## Install

### Recommended: install from GitHub

In Codex, invoke `$skill-installer` and provide this repository path:

```text
$skill-installer Install this skill from:
https://github.com/Lancelot-Xie/img2pptx/tree/main/skills/img2pptx
```

中文安装提示：

```text
$skill-installer 请从 GitHub 安装这个 skill：
https://github.com/Lancelot-Xie/img2pptx/tree/main/skills/img2pptx
```

Codex detects newly installed skills automatically. If it does not appear,
restart Codex or start a new task.

### Manual installation

For a personal skill, copy `skills/img2pptx` to:

```text
~/.agents/skills/img2pptx
```

For a repository-scoped skill, copy it to:

```text
YOUR_REPOSITORY/.agents/skills/img2pptx
```

## Use

Attach an image and use a short prompt:

```text
Use $img2pptx to turn this image into an editable PPTX.
```

中文：

```text
使用 $img2pptx，把这张图片转为可编辑的 PPTX。
```

You can also be more specific:

```text
Use $img2pptx to reconstruct diagram.png as a one-slide editable PPTX.
Preserve the original aspect ratio, text, colors, arrows, and scientific symbols.
```

The skill may also trigger automatically when a request clearly asks to convert,
rebuild, trace, or vectorize a raster reference into an editable PowerPoint slide.

## What it produces

The main deliverable is `final.pptx`. A complete run also produces:

- `full.svg` — the full vector reconstruction embedded in the PPTX;
- `component_manifest.json` — semantic components, hierarchy, geometry, and constraints;
- `modules/*.svg` — independently reusable semantic modules;
- `qa/*` — layout, containment, border, semantic, visual, and PPTX embedding audits.

## Why not just put the image on a slide?

`img2pptx` treats the reference as a reconstruction task rather than a screenshot
placement task. It aims to preserve:

- editable SVG text and vector primitives;
- meaningful nested groups and reusable modules;
- arrows, containment, sequence, color semantics, and scientific notation;
- an auditable relationship between the source image, SVG, and PPTX.

Raster crops are used only when an element cannot be reproduced reliably as a
vector without losing its visual identity.

## Editability and PowerPoint compatibility

The PPTX contains the complete `full.svg`, not only a full-slide bitmap.
Recent PowerPoint versions can usually use **Convert to Shape** and **Ungroup**
on an SVG. Exact text conversion, nested group preservation, and appearance
depend on the PowerPoint version, operating system, fonts, and SVG importer.

The skill reports these separately:

1. whether the complete SVG is embedded;
2. whether the SVG is structurally ready for conversion;
3. whether an actual PowerPoint conversion was tested.

It does not claim perfect post-conversion fidelity unless that conversion was
actually performed and inspected.

## Supported references

Common inputs include:

- AI-generated diagrams and infographics;
- research-paper architecture figures;
- scientific workflows and symbolic diagrams;
- flowcharts, process diagrams, and system overviews;
- screenshots, PNG, JPG/JPEG, and WebP files.

Use only images you have permission to reproduce.

## Repository layout

```text
skills/img2pptx/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── semantic-constraint-audit.md
    └── visual-optimization.md
```

## Status

This is an early open-source release. Real-world examples and regression tests
will be added as the workflow is tested across more diagram styles and runtime
environments.

## License

Apache License 2.0. See [LICENSE](LICENSE).

# img2pptx
Turn AI-generated diagrams and raster references into editable, modular, audited PPTX slides.
