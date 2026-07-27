# Semantic Constraint and Audit Protocol

## Contents

1. Activation and scope
2. Source observation
3. Constraint schema
4. Content-specific invariants
5. Negative constraints
6. Audit implementation
7. Constraint coverage
8. Human semantic review
9. Required outputs
10. Example

## 1. Activation and scope

Apply this protocol to every component whose meaning depends on color, position,
direction, order, count, connectivity, containment, scale, sign, rank, or symbol
choice. Typical examples include charts, plots, tables, rankings, flow diagrams,
architectures, scientific figures, maps, timelines, and annotated sequences.

Do not apply chart-specific rules to unrelated illustrations. First classify each
semantic component, then select only the relevant invariant types.

Treat semantic correctness as a hard gate. A visually attractive or low-MAE
reconstruction must fail if it reverses, merges, duplicates, or invents meaning.

## 2. Source observation

Before drawing a semantic component:

1. Crop it from the normalized source at a useful inspection scale.
2. Identify its semantic type and visual encoding.
3. Record observable facts without guessing hidden data.
4. Separate direct observations from inferences.
5. Record uncertain observations with confidence and require human review.

Use `source_observations` entries such as:

```json
{
  "id": "obs_gain_series",
  "component": "description_gain_chart",
  "statement": "Red bars form two positive clusters above the zero line; navy marks remain below the line.",
  "evidence_bbox": {"x": 1350, "y": 275, "width": 455, "height": 135},
  "basis": "direct",
  "confidence": 0.99
}
```

Do not infer numerical values that are not legible. Preserve observed geometry and
relationships when exact data cannot be recovered.

## 3. Constraint schema

For every meaning-bearing observation, create at least one testable
`visual_invariants` or `negative_constraints` entry. Use:

```json
{
  "id": "inv_navy_below_zero",
  "component": "description_gain_chart",
  "type": "axis-side",
  "subject": "navy_bars",
  "relation": "below-or-on",
  "reference": "zero_axis",
  "expected": true,
  "severity": "hard",
  "source_observation": "obs_gain_series",
  "audit_method": "svg-geometry",
  "audit_check": "semantic_constraint_audit.chart.navy_below_zero"
}
```

Support these common fields:

- `id`: stable constraint identifier;
- `component`: owning semantic component;
- `type`: invariant category;
- `subject`: element or series being constrained;
- `relation`: expected spatial, categorical, or topological relationship;
- `reference`: axis, region, parent, peer, legend, sequence, or other reference;
- `expected`: allowed value, range, order, count, set, or boolean;
- `forbidden`: disallowed value, color, region, overlap, direction, or relation;
- `tolerance`: justified numeric tolerance when needed;
- `severity`: `hard`, `review`, or `advisory`;
- `source_observation`: source evidence identifier;
- `audit_method`: structural, geometric, raster, OCR, or human-review method;
- `audit_check`: exact output check path.

Every hard constraint must have a concrete audit method and output path. Do not
accept prose-only hard constraints.

## 4. Content-specific invariants

### Charts and plots

Check applicable properties:

- series color and legend consistency;
- axis side, sign, and baseline relationship;
- positive/negative direction;
- bar, point, or line count when visually recoverable;
- peak, valley, cluster, and gap locations;
- ordering and monotonic relationships;
- grouping and series separation;
- exclusive color regions;
- line continuity and endpoint placement;
- error-bar or interval orientation;
- category label alignment;
- stacked versus overlapping series;
- emphasized versus background series.

Prefer semantic checks on SVG element IDs and geometry. Use raster color masks as
supplementary evidence, not as the only representation of chart meaning.

### Flowcharts, architectures, and workflows

Check applicable properties:

- node identity and count;
- edge source, destination, and direction;
- arrowhead orientation;
- branch and merge topology;
- input/output port attachment;
- containment and nesting;
- ordering of stages;
- crossing versus connection;
- repeated module consistency;
- forbidden shortcuts or invented edges.

### Tables, rankings, and sequences

Check applicable properties:

- row and column order;
- header-to-cell association;
- rank order;
- bar-length or score monotonicity;
- token or item sequence;
- span boundaries and bracket membership;
- emphasized cells and category colors;
- missing, duplicated, or shifted entries.

### Scientific figures and symbolic diagrams

Check applicable properties:

- label-to-object association;
- symbol identity;
- topology and connectivity;
- orientation and direction;
- color semantics;
- repeated motif count;
- relative location of annotations;
- formula, index, superscript, and subscript meaning;
- forbidden substitutions of visually similar symbols.

## 5. Negative constraints

Record what must not appear, not only what must appear. Typical negative
constraints include:

- forbidden color in a semantic region;
- forbidden element on one side of an axis;
- forbidden overlap or stacking;
- forbidden edge or arrow connection;
- forbidden duplicate label, node, series, or token;
- forbidden reordering;
- forbidden crossing treated as a junction;
- forbidden invented icon, value, or annotation;
- forbidden disappearance of a low-contrast source element.

Use negative constraints whenever an incorrect reconstruction could still look
plausible at thumbnail scale.

## 6. Audit implementation

Implement audits in this priority order:

1. Inspect SVG structure, IDs, element attributes, and geometry.
2. Compare rendered component crops using foreground and semantic masks.
3. Use OCR or color segmentation only when structural checks are unavailable.
4. Require human review for uncertain or non-machine-verifiable constraints.

For charts, compute region-aware diagnostics where applicable:

- foreground-only MAE;
- per-color mask precision and recall;
- error above and below a baseline separately;
- error inside exclusive semantic regions;
- connected-component count;
- overlap between forbidden series masks;
- ordering or rank agreement.

Do not let white background dominate a metric. Do not let lower global MAE override
a failed hard semantic constraint.

Audit actual values. Never emit `passed: true` merely because an expected component
or JSON field exists.

## 7. Constraint coverage

Generate `qa/constraint_coverage_audit.json`. For every source observation and
constraint, record:

```json
{
  "constraint_id": "inv_navy_below_zero",
  "component": "description_gain_chart",
  "severity": "hard",
  "audit_check": "semantic_constraint_audit.chart.navy_below_zero",
  "implemented": true,
  "executed": true,
  "passed": true,
  "evidence": {
    "navy_positive_element_count": 0,
    "zero_axis_y": 346
  }
}
```

Fail the coverage audit when:

- a hard source observation has no constraint;
- a hard constraint has no audit mapping;
- a mapped audit did not execute;
- evidence is absent or unrelated;
- a hard constraint failed;
- uncertainty requiring review was not reviewed.

Report coverage counts for observations, hard constraints, implemented checks,
executed checks, passed checks, review items, and uncovered items.

## 8. Human semantic review

Generate a zoomed review sheet for:

- every component with hard semantic constraints;
- every component with uncertain observations;
- the highest-error semantic components;
- every component whose meaning relies on color, axis side, connectivity, or order.

Show source, reconstruction, overlay, amplified diff, and relevant masks at a
readable scale. Record reviewer status as `passed`, `failed`, or `needs-review`.

Do not claim human review occurred unless the rendered sheet was actually
inspected. If no human review is possible, keep the item as unresolved and do not
pass a hard review requirement.

## 9. Required outputs

When this protocol activates, produce:

```text
qa/semantic_constraint_audit.json
qa/constraint_coverage_audit.json
qa/semantic_review_sheet.png
qa/semantic_masks/
```

Only create masks relevant to the selected constraints. Avoid meaningless generic
masks.

Include semantic and coverage audits in the hard-audit summary and correction loop.

## 10. Example

For a gain chart with a zero line, red positive peaks, and navy negative marks:

```json
{
  "source_observations": [
    {
      "id": "obs_gain_color_sign",
      "statement": "Red encodes positive peaks and navy encodes negative marks.",
      "basis": "direct",
      "confidence": 0.99
    }
  ],
  "visual_invariants": [
    {
      "id": "inv_red_above_zero",
      "type": "axis-side",
      "subject": "red_bars",
      "relation": "above-or-on",
      "reference": "zero_axis",
      "severity": "hard",
      "source_observation": "obs_gain_color_sign",
      "audit_method": "svg-geometry"
    },
    {
      "id": "inv_navy_below_zero",
      "type": "axis-side",
      "subject": "navy_bars",
      "relation": "below-or-on",
      "reference": "zero_axis",
      "severity": "hard",
      "source_observation": "obs_gain_color_sign",
      "audit_method": "svg-geometry"
    }
  ],
  "negative_constraints": [
    {
      "id": "neg_navy_in_positive_clusters",
      "type": "forbidden-overlap",
      "subject": "navy_bars",
      "forbidden": "positive_peak_regions",
      "severity": "hard",
      "source_observation": "obs_gain_color_sign",
      "audit_method": "svg-geometry-and-color-mask"
    }
  ]
}
```

The component must fail if navy elements extend above the zero axis or overlap the
red-only peak regions, even when full-image MAE is low and the PPTX preview matches
the generated SVG.
