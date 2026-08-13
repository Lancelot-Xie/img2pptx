---
name: img2pptx
description: Reconstruct raster reference images as one-slide editable, modular, auditable PPTX files containing a complete SVG. Use when the user asks to convert, reproduce, trace, vectorize, or rebuild a PNG, JPG/JPEG, WebP, TIFF, HEIC, screenshot, diagram, infographic, scientific figure, architecture figure, or workflow image into an editable PowerPoint slide.
---

# Img2Pptx

Reconstruct an input raster reference as a one-slide PPTX that is editable, decomposable, reusable, and auditable. Treat this Skill as an execution protocol; do not reduce the task to drawing an image that merely looks similar.

## Operating contract

Apply the following rules to every task:

1. Inspect the input file, working directory, and locally available tools.
2. Create task-specific drawing, rendering, cropping, comparison, PPTX-generation, and audit scripts in the current task workspace as needed.
3. Do not assume that scripts, component names, canvas dimensions, layouts, or thresholds from a previous task apply to the current task.
4. Prefer tools already available in the environment. Do not bind the Skill to a specific programming language, browser path, or operating system.
5. Keep task-specific implementation in the current task directory so the result remains reproducible, modifiable, and auditable.
6. Continue through reconstruction, rendering, QA, correction, PPTX generation, and final audit. Do not stop after producing the first SVG draft.
7. Do not modify the original input image.

## Input normalization

Accept PNG, JPG/JPEG, WebP, and other common raster formats that local tools can decode.

For TIFF, HEIC, multi-frame images, color-profiled images, or formats with
unreliable decoding support:

1. Preserve the original file and convert the target frame to a standard PNG losslessly or at high quality.
2. Normalize color space and orientation, then use the normalized PNG as the baseline for subsequent pixel comparisons.
3. Record both the original and normalized files in the manifest.

Use the input image's actual dimensions and aspect ratio. Unless the user explicitly requests a standard slide ratio, make the PPTX page ratio match the input image.

## Required outcome

The result MUST satisfy all of the following:

- Produce a one-slide PPTX containing the complete `full.svg`; do not use a full-slide bitmap as the only embedded representation.
- A PNG fallback may also be present, but SVG MUST remain the primary vector representation and should support PowerPoint **Convert to Shape** or **Ungroup** as far as reasonably possible.
- Preserve text as editable SVG `<text>` wherever practical, and preserve graphics as vector shapes, paths, or vector groups wherever practical.
- Make every required exported module self-contained, and complete structural audits, visual comparisons, PPTX audits, and automatic corrections.
- Clearly distinguish "the SVG is embedded in full" from "all hierarchy survives conversion inside PowerPoint." The latter depends on the PowerPoint version and SVG importer; never claim it without verification.

## 1. Model at the practical editing granularity

Decompose content according to semantic integrity and real editing needs, not mechanically by geometric primitive.

An editable unit should, as far as practical:

- remain complete when isolated, with a clear boundary, independent meaning, or visual function;
- be movable, replaceable, or modifiable on its own;
- avoid producing large numbers of meaningless fragments after ungrouping.

Do not treat half a border, half an icon, an isolated decoration, or a contextless line segment as a semantic module.

### Semantic module: `semantic-unit`

Define a visual unit that independently expresses a complete concept or performs a clear function as a semantic module. Examples include a panel, flow submodule, report box, process node, structural module, relationship arrow, input box, output box, or annotation box. Identify modules from the actual input; do not force the content into a fixed type list.

### Composite submodule: `combination-submodule`

Define a group of elements that jointly carries locally complete meaning as a composite submodule. Examples include:

- icon + label;
- title + divider;
- icon row;
- one row inside a report box;
- structural formula + name;
- arrow + annotation;
- status icon + value.

### Atomic element: `atomic-element`

Define the smallest element that has no practical need for further decomposition at the target editing granularity as an atomic element. Examples include:

- editable text;
- line, rect, circle, ellipse, polygon, or polyline;
- SVG path;
- basic icon;
- small, complex vector motif;
- logo or brand mark;
- fragments of molecular structures, cells, and other scientific illustrations;
- small illustrations or specialized symbols.

"Atomic" means that further decomposition has no practical editing value, not that the element is geometrically indivisible. A complex icon may use a `<g>` or `<symbol>` with its own `id`; the manifest may classify it as atomic while still allowing its internal paths to remain editable after PowerPoint conversion.

## 2. Handle complex visual motifs

For logos, complex icons, molecular structures, small illustrations, and similar content, use the following options in order:

1. Reuse vector assets already present in the input materials, or use a reliable official SVG or specification-compliant asset when available.
2. Redraw, trace, or vectorize the motif from the reference image.
3. If necessary, generate the motif first and then convert it to vector form.
4. Use an embedded raster image ONLY when reasonable vectorization is not possible.

To preserve visual fidelity, recognizable icons, scientific symbols, and small visual motifs from the reference MUST retain their graphical identity and basic appearance. MUST NOT replace them with text, emoji, Unicode characters, a different icon, or a generic placeholder merely for implementation convenience or editability, unless the reference itself uses that representation. Prefer vector redrawing. ONLY when reliable redrawing is not possible and substitution would cause obvious visual distortion may the relevant source crop be preserved as an independent atomic raster image. In that case, record its source region, reason for use, and later replacement status in the manifest.

Do not use a source crop to replace any module that can reasonably be reconstructed as editable vector content merely to save drawing effort. Preserve a local raster crop only when the source content is inherently raster or vectorization would materially reduce visual fidelity or information accuracy, and crop only the minimum necessary region.

Do not evade the vector-reconstruction requirement by slicing the source into raster tiles and reassembling them. An allowed raster region MUST NOT unnecessarily include text, borders, arrows, connectors, or other graphics that can reasonably be reconstructed as separate editable vector elements.

Every independently identifiable icon or small visual motif MUST be cropped from the normalized source and inspected at increased scale before drawing. After reconstruction, render it independently and compare it with the enlarged source crop side by side and with an overlay or amplified diff. Verify silhouette, internal structure, orientation, stroke weight, color, negative space, and distinguishing features. Apply this even when reusing a vector asset. If the source is too ambiguous, require review and do not invent details.

Do not draw a brand logo from memory. Prefer the shape shown in the input, or use
a reliable specification-compliant asset.

When using a raster image, record the following in the manifest:

- `representation: raster-image`;
- reason for use;
- original source;
- whether it can later be replaced with a vector.

## Extract semantic constraints from the source (mandatory)

Before drawing, identify every component whose meaning depends on color,
position, direction, order, count, connectivity, containment, sign, rank, or
symbol choice. For each such component:

1. Crop it from the normalized source and inspect it at increased scale.
2. Record directly observed `source_observations`; keep inference separate from
   direct observation.
3. Define executable `visual_invariants` and `negative_constraints`.
4. Assign an `audit_method` and one unique `audit_check` to every hard
   constraint.
5. Mark uncertain observations for human review. Do not fill gaps from common
   knowledge or fabricate data.

For charts, tables, rankings, flow diagrams, architecture diagrams, scientific
figures, maps, timelines, or annotated sequences, MUST read and follow the
[Semantic Constraint and Audit Protocol](references/semantic-constraint-audit.md).
Activate only the rules relevant to the current component type; do not
mechanically apply chart rules to an ordinary illustration.

Semantic correctness is a hard gate. Low MAE, structural completeness, or a PPTX that faithfully reproduces the generated SVG cannot override errors in color meaning, axis side, direction, topology, order, or mutual exclusivity.

## 3. Build the Component Manifest

MUST create `component_manifest.json` before drawing. Use it as the source of truth for subsequent drawing, module export, parent-child containment checks, alignment checks, and component crop diffs.

Record at least the following for every component:

```json
{
  "id": "evidence_a_report",
  "label": "Structure report card",
  "parent": "panel_evidence",
  "bbox": {
    "x": 0,
    "y": 0,
    "width": 100,
    "height": 80
  },
  "level": "semantic-unit",
  "export": true,
  "editable_parts": ["title", "divider", "rows", "icons", "labels"],
  "representation": "svg-group",
  "render_strategy": "vector-reconstruction",
  "semantic_role": "structure report",
  "source_observations": [],
  "visual_invariants": [],
  "negative_constraints": [],
  "notes": "Keep outline as final child"
}
```

Prefer the following fields:

- identity and hierarchy: `id`, `label`, `parent`, `level`;
- geometry and export: `bbox`, `export`;
- editing and representation: `editable_parts`, `representation`, `render_strategy`;
- source and constraints: `asset_source`, `constraints`, `notes`;
- semantics and audit: `semantic_role`, `source_observations`, `visual_invariants`, `negative_constraints`, `audit_mapping`.

Also record the canvas, normalized input file, default padding, outline parameters, and visual-optimization parameters. Every hard semantic constraint MUST map to an audit check that is actually executed. If a constraint is unmapped, unexecuted, or lacks evidence, the constraint-coverage audit MUST fail.

## 4. Build the layout skeleton

Reconstruct these large-scale relationships first:

- canvas dimensions, aspect ratio, and main-title position;
- bounding boxes of major panels and cards, plus the direction of primary arrows;
- module spacing, column widths, row heights, and overall visual center of mass.

At this stage, establish only the structure and large-scale relationships. Do not handle complex icons or fine details yet.

Run a layout-skeleton audit and check:

- whether major panels align at the top and bottom, and whether their widths and heights are close to the source;
- whether major column gaps are reasonable and submodules visibly stay within bounds;
- whether primary arrows connect correctly and the overall visual balance is close to the source.

Proceed to detailed drawing only after the skeleton audit passes. Never hard-code the layout audit as passing.

## 5. Draw each standalone module

Draw semantic modules one by one from the input image and manifest. Possible examples include a query card, retrieval card, report card, decision bucket, LLM input, final prediction, arrow, or annotation box. These are examples, not a fixed component list.

Every independently exported module MUST:

- carry complete meaning and retain safe padding;
- use its own `<g id="...">`, preserve text as SVG `<text>` wherever practical, and use SVG vector elements for graphics wherever practical;
- contain complex icons in self-contained `<g>` or `<symbol>` elements;
- render correctly without depending on elements outside the module.

Avoid SVG features that compromise PowerPoint conversion. Prefer `text`, `rect`, `line`, `circle`, `ellipse`, `polygon`, `polyline`, and `path`. Avoid `foreignObject`, unnecessary filters, external-resource dependencies, and complex CSS. When `<defs>`, gradients, `clipPath`, or symbols are necessary, ensure that every standalone module carries all of its dependencies and verify the PowerPoint preview.

## 6. Use PowerPoint-friendly border layering

Important container borders MUST use:

```text
fill-only background + top-layer outline
```

Apply this rule to panels, cards, report boxes, buckets, input cards, final
prediction cards, and similar containers:

- The background shape supplies only the fill and uses `stroke="none"`.
- Use a separate outline shape as the last visual child of the corresponding
  semantic-module group, with `fill="none"`.
- Use a suggested `stroke-width` of `1.4px–1.8px`.
- Inset outline coordinates by `0.5px–1px`.
- Do not apply a shadow or filter to the outline. The background and outline
  layers MUST NOT draw the same border twice.
- Treat the outline as an atomic element of its container, not as an independent
  semantic-module export.

## 7. Run the Standalone Module Integrity Audit

Generate a standalone SVG for every module with `export: true`, then check each
one for:

- clipping or content within 2 px of an edge;
- overflowing text, incomplete icons, or incomplete borders;
- clipped shadows or decorations;
- semantic completeness when isolated;
- complete inclusion of dependencies such as `<defs>`, gradients, `clipPath`,
  and symbols.

MUST correct missing text, missing edges, clipping, unsafe edge proximity, or
missing dependencies.

## 8. Assemble the complete SVG

After all standalone modules pass, assemble them into `full.svg` according to
the bounding boxes in the manifest.

Preserve the actual parent-child hierarchy, for example:

```xml
<g id="panel_evidence">
  <g id="evidence_a_unit">
    <g id="capsule_a">...</g>
    <g id="arrow_a">...</g>
    <g id="structure_report_card">...</g>
  </g>
</g>
```

Do not flatten every element into sibling SVG nodes. Preserve hierarchy to
support:

- PowerPoint ungrouping;
- moving complete modules;
- further decomposition of submodules;
- independent editing of atomic elements;
- QA localization and automatic correction.

## 9. Run the Parent-Child Containment Audit

Check that every child-module bounding box lies entirely within its parent
bounding box. Preserve 6–10 px of inner padding by default; document any
exceptionally compact layout in the manifest.

Check that:

- no submodule crosses its parent's boundary;
- no submodule touches an edge without justification;
- no submodule intrudes into an adjacent panel;
- every parent-child relationship is correct;
- cards, capsules, buckets, and similar elements stay inside their assigned
  panels;
- each module's actual rendered bounds substantially match its manifest bounding
  box.

Treat every failure as a structural error and correct it.

## 10. Run the Alignment and Spacing Audit

Check:

- top and bottom alignment of panels;
- consistent left and right edges for cards of the same type;
- correct title alignment;
- correct arrow-to-module connections;
- even spacing between modules in the same column;
- consistent sizing of like elements;
- whether submodules are overcrowded or excessively sparse;
- consistent border weights;
- visual-center alignment between icons and text;
- clear text readability: no opaque shape unintentionally covers text, and no border, arrow, connector, or decorative line passes through readable glyph areas; allow intentional source-faithful cases;
- absence of doubled or misaligned outlines.

## 11. Run the Border Layering Audit

Specifically check:

- whether every important container uses a fill-only background;
- whether a separate top-layer outline exists;
- whether duplicate strokes exist;
- whether the outline is the last visual element in its group;
- whether the outline is inset;
- whether the outline is free of filters and shadows;
- whether borders remain stably visible while grouped;
- whether borders remain present after ungrouping;
- whether any border appears doubled, thickened, misaligned, or clipped.

## Run the Semantic Constraint and Coverage Audit

Audit every component whose meaning depends on visual encoding, and produce:

```text
qa/semantic_constraint_audit.json
qa/constraint_coverage_audit.json
qa/semantic_review_sheet.png
```

Check only the invariants applicable to each component type. For charts, focus
on series colors, legends, axis sides, positive and negative direction,
peaks/grouping, order, mutually exclusive regions, and forbidden overlaps. For
flow diagrams, focus on nodes, edges, arrow direction, ports, branch merging,
and topology. For tables, rankings, and sequences, focus on order, association,
span, length relationships, duplication, and omission. For scientific figures,
focus on symbols, labels, connections, direction, and color semantics.

Also check elements, colors, connections, overlaps, orders, or regions that
MUST NOT appear. Prefer direct audits of SVG groups, IDs, attributes, and
geometric relationships; supplement these with color masks, foreground
comparisons, or human review.

Generate a constraint-coverage table confirming that every hard source
observation has a constraint, every hard constraint has an audit mapping, every
check was actually executed, and the evidence is relevant. The aggregate hard
gate MUST fail if any hard semantic constraint fails, lacks coverage, or
requires review that was not performed. See the
[Semantic Constraint and Audit Protocol](references/semantic-constraint-audit.md)
for the complete rules.

## 12. Run the Original-to-Render Visual Similarity Audit

After structural audits pass, compare the source and reconstruction and produce:

- normalized original PNG;
- rendered SVG;
- 50/50 overlay;
- amplified diff;
- original crop for each component;
- rendered crop for each component;
- diff crop for each component;
- `component_diff_sheet.png`.

Calculate both whole-image MAE and component-level MAE. Do not allow large
areas of flat background to conceal local errors. MAE is a localization and
optimization signal; it does not replace containment, standalone integrity,
border layering, or semantic visual checks.

For semantically dense components, also calculate applicable
foreground-weighted, per-color, per-region, per-axis-side, or mask-based
metrics. Inspect at increased scale every region with a hard semantic
constraint, an uncertain observation, or high component error. An improvement
in whole-image MAE cannot override a semantic-constraint failure.

Never hard-code `passed: true` before calculating and evaluating the relevant
metrics. Record actual values, thresholds, attempt counts, and target status for
every visual-optimization result.

## 13. Generate and audit the PPTX

Generate a one-slide PPTX containing the complete `full.svg`.

Check:

- that the PPTX package contains an SVG;
- that a full-slide PNG is not used as the sole replacement for the SVG;
- that the embedded SVG matches `full.svg`, preferably by content hash;
- that the PPTX preview matches the rendered SVG;
- that borders are clear while grouped;
- that text is neither corrupted nor displaced;
- that the same text-readability check passes in the rendered PPTX preview, including after any font substitution or displacement;
- that the content is reasonably prepared for PowerPoint **Convert to Shape**
  or **Ungroup**;
- that nested groups remain present in the embedded SVG bytes;
- that no critical unsupported PowerPoint SVG features are present;
- that SVG fallback and relationships are correct;
- that the PPTX contains exactly one slide.

Report "embedded SVG integrity," "conversion readiness," and "fidelity after
actual conversion" separately. Claim complete preservation of hierarchy after
conversion ONLY after performing and inspecting the conversion in PowerPoint.

## 14. Complete the mandatory correction loop

Do not deliver the first version without correction. Whenever a hard failure is
found, execute:

```text
detect issue
→ locate component id
→ classify issue
→ modify bbox / coordinates / dimensions / padding / font size / path / arrow / outline
→ re-export module
→ reassemble full.svg
→ re-render
→ rerun QA
→ update audit results
```

Continue correcting until none of the following hard constraints has an
unresolved issue:

- layout skeleton;
- standalone integrity;
- parent-child containment;
- alignment and spacing;
- border layering;
- semantic constraints and negative constraints;
- constraint coverage and required human review;
- SVG/PPTX package embedding;
- structure and readability of the PPTX preview.

Do not ignore a failed hard audit merely because final files have been
generated. Establish an aggregate gate; if any hard audit fails, MUST NOT claim
that the final result passed in full.

## 15. Perform bounded, reversible, non-degrading MAE optimization

Begin visual optimization ONLY after all structural, semantic,
constraint-coverage, and PPTX-compatibility hard audits pass. Read and follow
[Visual Similarity Optimization](references/visual-optimization.md).

Attempt at most three iterations by default, targeting a 10% relative MAE
improvement against the first valid baseline. Start every attempt from a
complete copy of the current `best`, make only local and explainable changes,
and rebuild the modules, `full.svg`, PPTX, preview, semantic masks, and all QA.

Accept a candidate ONLY if it still passes every hard gate, introduces no
semantic or local regression, and improves the metrics; otherwise, roll it back.
A verified semantic correction takes precedence over global MAE, but record the
failed constraint, correction evidence, and metric tradeoff. The 10% improvement
is a soft target; structure, semantics, coverage, and embedding integrity always
remain hard gates.

## 16. Final delivery

MUST deliver:

```text
final.pptx
full.svg
component_manifest.json
modules/*.svg

qa/layout_skeleton_audit.json
qa/standalone_integrity_audit.json
qa/containment_audit.json
qa/alignment_audit.json
qa/border_layering_audit.json
qa/semantic_constraint_audit.json
qa/constraint_coverage_audit.json
qa/semantic_review_sheet.png
qa/semantic_masks/*
qa/visual_similarity_audit.json
qa/full_overlay_diff.png
qa/amplified_diff.png
qa/component_diff_sheet.png
qa/pptx_slide_preview.png
qa/pptx_preview_audit.json
```

Semantic-audit files and masks are mandatory ONLY when the source contains
components whose meaning depends on visual encoding. For all other tasks, record
`not_applicable` in the aggregate report; do not fabricate an empty passing
result.

The file intended for actual use is `final.pptx`. QA files demonstrate whether
the reconstruction is reliable, complete, decomposable, editable, and
auditable.

Before delivery, confirm:

- the final files come from the complete `best` state;
- `final.pptx`, `full.svg`, the manifest, modules, and QA are mutually
  consistent;
- all structural, semantic, constraint-coverage, and PPTX hard audits pass;
- every item requiring human semantic review has a genuine review status;
- visual-optimization status is reported truthfully;
- if the 10% soft target was not reached, the actual stopping reason is stated
  and the best valid version is delivered.
