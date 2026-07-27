# Visual Similarity Optimization

Use this protocol only after every hard structural, semantic, compatibility, and
coverage audit has passed.

## Defaults

```text
max_try = 3
target_relative_mae_improvement = 0.10
```

Calculate the 10% target relative to the first valid baseline, not relative to the
previous attempt.

## Version state

Maintain:

```text
baseline  first complete state passing all hard audits
best      current best valid complete state
candidate isolated trial derived from best
```

Save the complete state, including:

- `full.svg`;
- `component_manifest.json`;
- `modules/*.svg`;
- generation source or scripts;
- `final.pptx`;
- preview and QA results;
- semantic constraints and their audit evidence.

Record:

```text
baseline_mae = initial valid MAE
best_mae = baseline_mae
target_mae = baseline_mae × 0.90
```

## Candidate attempts

For each attempt:

1. Start from a fresh copy of `best`.
2. Inspect amplified diff, overlay, component diff sheet, semantic review sheet,
   foreground-weighted metrics, and component-level metrics.
3. Select a small number of high-impact components.
4. Make local, explainable changes only.
5. Generate an isolated `candidate`.
6. Rebuild modules, `full.svg`, PPTX, preview, masks, and QA.
7. Re-run every hard audit, including semantic constraint and coverage audits.
8. Calculate full-image and component metrics.
9. Compare the candidate with `best`.

Never continue editing a rejected candidate. Start the next attempt from the
current `best`.

## Acceptance and rollback

Promote a candidate only when all conditions hold:

- all hard structural and compatibility audits pass;
- all hard semantic constraints and coverage checks pass;
- no required module, text, border, group, symbol, or relationship is lost;
- `candidate_mae < best_mae`, unless the change corrects a verified semantic error
  that the original metric failed to weight appropriately;
- foreground and semantic-region metrics do not regress materially;
- non-target components do not visibly regress;
- inspection finds no semantic or aesthetic degradation hidden by aggregate MAE.

When accepting a semantic correction despite worse global MAE, record:

- the failed baseline semantic constraint;
- the corrected constraint evidence;
- the metric tradeoff;
- why semantic correctness takes precedence.

For a normal visual candidate, calculate:

```text
relative_improvement =
  (baseline_mae - candidate_mae) / baseline_mae
```

If the candidate is worse, unchanged, invalid, or semantically degraded:

```text
discard candidate
restore and retain best
```

Never replace `best` with a lower-quality candidate merely because it is newer.

## Stop conditions

Stop early when:

```text
best_mae <= baseline_mae × 0.90
```

Otherwise stop when:

- `max_try` attempts have completed;
- no safe local improvement can be identified;
- further MAE reduction would sacrifice editability, semantic correctness, or
  structural integrity.

When the 10% target is not reached, deliver the best valid state and report the
actual status. Never disguise an unmet soft target as achieved.

## Reporting

In `qa/visual_similarity_audit.json`, include at least:

```json
{
  "baseline_mae": 20.201,
  "target_mae": 18.181,
  "best_mae": 18.9,
  "relative_improvement": 0.0644,
  "target_relative_improvement": 0.1,
  "target_achieved": false,
  "attempts_used": 3,
  "max_try": 3,
  "optimization_status": "max_try_reached",
  "delivered_state": "best_valid_candidate",
  "hard_audits_passed": true,
  "semantic_audits_passed": true,
  "constraint_coverage_passed": true
}
```

Use these statuses:

- `target_achieved`;
- `max_try_reached`;
- `no_safe_improvement`;
- `baseline_already_optimal`.

Record each attempt, targeted components, changes, metrics, audit results,
acceptance decision, and rollback result.

Treat the 10% improvement as a soft target. Treat structural, semantic, coverage,
and PPTX integrity audits as hard gates.
