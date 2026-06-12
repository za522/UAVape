# UAVape V2 Method Gap Matrix

Purpose: make the methodology situation digestible. This separates planning, execution, artifact loss, and genuine methodological gaps.

## Plain-English Summary

V2 was not a bad experiment. It fixed the major V1 validation/test mismatch and ran a sensible data-composition ablation.

The problem is that V2 is currently under-explained because the post-training artifacts were not preserved. That means we have the official all-five metrics, but not the full diagnostic evidence needed to explain *why* each model behaved the way it did.

The literature benchmark adds a second issue: some things we planned are normal and defensible, but stronger object-detection papers usually go further with slice metrics, hard-case analysis, qualitative failures, and complexity/runtime reporting.

## Core Matrix

| Area | Planned for V2? | Completed in V2? | Blocked by artifact loss? | Done in stronger papers? | What this means for us |
|---|---:|---:|---:|---:|---|
| Five dataset-composition experiments | Yes | Yes | No | Yes | This part of V2 is structurally defensible. |
| Fixed validation/test set across all experiments | Yes | Yes | No | Yes | This was the main V2 improvement over V1. |
| Real-positive + hard-negative validation/test | Yes | Yes | No | Partly | Defensible if framed as target-domain evaluation. |
| Official precision, recall, mAP50, mAP50-95 | Yes | Yes | No | Yes | We have the core comparison table. |
| Best checkpoint chosen by validation mAP50-95 | Yes | Yes | No | Yes | Defensible. Do not choose checkpoints using test performance. |
| Dataset/source composition tables | Yes | Yes | No | Yes | We can explain what each experiment trained on. |
| Scale-band dataset composition | Yes | Yes | No | Yes | We know object-scale distribution, but not yet model performance by scale. |
| Training curves | Yes | No / partial logs only | Yes | Yes | Need rerun/recovered artifacts for all-five curves. |
| Confusion matrices | Yes | No / lost | Yes | Yes | Need rerun/recovered artifacts. |
| PR curves / precision-recall tradeoff | Yes | No | Yes | Yes | Needed to explain threshold behaviour and deployment tradeoff. |
| Confidence-threshold analysis | Yes | No | Yes | Yes | Needed to choose operating threshold without guessing. |
| Prediction confidence distributions | Yes | No | Yes | Often | Needed to compare whether models are conservative or overconfident. |
| Hard-negative false-positive counts | Yes | Only partial E4/E5 before reset | Yes | Yes, conceptually | Very important because hard negatives are central to UAVape. |
| False-positive / false-negative qualitative montage | Yes | No | Yes | Yes | Needed for dissertation explanation of failure modes. |
| Metadata-linked failure analysis | Yes | No | Yes | Sometimes | Metadata exists, but needs predictions joined to it. |
| Performance by scale band | Intended through metadata | No | Yes | Yes | Important because vape objects are often tiny/small. |
| External/source-shift test | No / not clearly planned | No | No | Often in broader papers | Needed only if we want claims about generalisation beyond the real target-domain test. |
| Runtime / FPS / model size / complexity table | Not strongly planned | No | Partly | Yes | Useful for UAV/deployment framing. |
| Repeated seeds | No | No | No | Sometimes | Optional, but helps if results are close. |
| Synthetic ratio ablation | No | No | No | Yes | V2 cannot conclude synthetic data is bad in general. |
| Synthetic quality/domain-gap analysis | Partly through metadata | No | Partly | Yes | Needed if synthetic data becomes a major dissertation claim. |
| Hard-negative iterative mining | No | No | No | Yes, in hard-example literature | Possible V3 improvement path, not required for V2 unless redesigning. |
| Model/image-size ablation | No | No | No | Common | Useful if we want to optimize performance, not just compare data recipes. |

## What The Metadata Framework Answers

The metadata framework answers questions about the dataset:

- which source each image came from;
- whether it was positive or hard negative;
- which split it belonged to;
- object scale band;
- bounding-box area;
- training/validation/test composition;
- whether external data changed the training distribution.

This helps explain the experiment design.

## What The Metadata Framework Does Not Answer Alone

Metadata alone does not answer model-behaviour questions:

- which images were false positives;
- which positive images were missed;
- whether tiny vapes failed more than large vapes;
- whether synthetic data caused high-confidence wrong detections;
- whether scraped data improved recall but hurt localisation;
- what confidence threshold should be used;
- whether E2 won because it found more vapes or avoided more false positives.

To answer those, metadata must be joined with prediction outputs.

## What Was A Planning Gap vs An Artifact-Loss Gap

### Artifact-Loss Gaps

These were planned or expected, but cannot be completed now without reruns/recovered outputs:

- all-five training curves;
- all-five confusion matrices;
- all-five PR curves;
- all-five confidence distributions;
- all-five hard-negative FP counts;
- all-five qualitative failure montages;
- all-five prediction-level metadata failure analysis;
- all-five scale-band performance.

This is not a conceptual planning failure. It is mainly a Colab preservation failure.

### Genuine Methodology Gaps

These were not fully built into V2:

- synthetic ratio ablation;
- synthetic quality-tier ablation;
- external/source-shift test set;
- repeated-seed reliability;
- model-size/image-size ladder;
- runtime/FPS/edge deployment table;
- hard-negative iterative mining.

These are possible V3 additions, not things V2 already promised.

## What Domain-Adjacent Papers Did That We Did Not Do

Domain-adjacent papers often added:

- deployment/runtime analysis;
- edge-device feasibility;
- external or cross-domain evaluation;
- object-size-specific evaluation;
- qualitative failure examples;
- model-family comparisons;
- clearer link between evaluation and real deployment scenario.

Mapped to UAVape, the most relevant missing pieces are:

- hard-negative false-positive analysis;
- tiny/small vape performance;
- qualitative examples of false positives and false negatives;
- inference speed/model size;
- a clearer statement that V2 evaluates target-domain real UAVape-like data, not all possible vape/litter domains.

## What Ablation Papers Did That We Did Not Do

Ablation-specific papers often added:

- one-change-at-a-time ablation tables;
- progressive combination tables;
- mAP50 and mAP50-95 together;
- precision and recall together;
- complexity metrics;
- task-specific slice metrics;
- qualitative visual comparisons;
- PR or confidence-threshold diagnostics;
- synthetic ratio/quality variation;
- sometimes repeated dataset/backbone tests.

Mapped to UAVape:

- our data-source ablation is a valid starting structure;
- our missing all-five diagnostics weaken the explanation;
- our synthetic condition is too broad to support a strong synthetic-data conclusion;
- our E2 result should be framed as target-domain hard-negative training, not universal superiority.

## Minimum V2 Write-Up Evidence

If we write V2 as final-with-limitations, we need:

- official all-five metric table;
- validation/test composition table;
- training composition table;
- validation-vs-test gap table;
- clear note that post-training artifacts were not preserved;
- cautious interpretation of E2;
- literature-backed limitation that synthetic/scraped conclusions are exploratory.

## Minimum Rerun Evidence

If we rerun to complete V2 properly, each experiment must save:

- `weights/best.pt`;
- `weights/last.pt`;
- `results.csv`;
- `results.png`;
- `confusion_matrix.png`;
- `confusion_matrix_normalized.png`;
- PR/P/R/F1 curves if generated;
- prediction CSV with image path, confidence, bbox, matched/unmatched status;
- hard-negative FP table;
- scale-band performance table;
- qualitative failure montage;
- Drive backup immediately after each run.

## V2 vs V3 Decision

### V2 Final-With-Limitations Is Reasonable If

- the dissertation claim is narrow;
- E2 is described as best under the fixed real-positive + hard-negative target protocol;
- scraped/synthetic conclusions are described as exploratory;
- missing diagnostics are acknowledged honestly;
- literature benchmark is used to justify what should be done next.

### V3 Is Better If

- the goal is a stronger final model;
- the dissertation needs a more convincing data-centric methodology;
- synthetic data is a major research question;
- we want to prove whether external data helps or hurts;
- we want deployment-ready threshold and failure-mode evidence.

## Recommended Next Step

Do not rerun everything blindly yet.

First decide the dissertation strategy:

1. **V2 final-with-limitations**: use current metrics and literature benchmark, but keep claims narrow.
2. **V2 completion rerun**: rerun the same five experiments only to recover diagnostics.
3. **V3 redesign**: use literature lessons to redesign the experiment around synthetic ratio, hard-negative FP, scale performance, and deployment threshold.

