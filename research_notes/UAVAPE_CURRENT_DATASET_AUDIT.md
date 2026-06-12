# UAVape Current Dataset Audit

Date: 2026-06-09

This audit summarises the accepted V3 real-world UAVape dataset before the planned daylight expansion pass.

## Accepted Input Pools

| Pool | Accepted images | Vape boxes | Hard-negative images | Positive images |
|---|---:|---:|---:|---:|
| `positive_train` | 377 | 509 | 0 | 377 |
| `hard_negative_train` | 125 | 0 | 125 | 0 |
| `positive_test` | 80 | 135 | 0 | 80 |
| `hard_negative_test` | 63 | 0 | 63 | 0 |
| **Real total** | **645** | **644** | **188** | **457** |

External/additional pools used in V3 ablations:

| Pool | Accepted images | Boxes | Note |
|---|---:|---:|---|
| `scraped_positive` | 2905 | 4437 | External positive training source only. |
| `synthetic_positive` | 355 | 355 | Synthetic positive training source only. |

One unsupported/corrupt input was rejected from the scraped-positive pool: `cigarette_vape_detection_002673_2fa543d121.jpg`, because the actual image format was GIF.

## V3-E1 Fixed Real Split

| Split | Source pool | Images | Boxes |
|---|---|---:|---:|
| train | `positive_train` | 320 | 446 |
| train | `hard_negative_train` | 80 | 0 |
| val | `positive_train` | 57 | 63 |
| val | `hard_negative_train` | 45 | 0 |
| test | `positive_test` | 80 | 135 |
| test | `hard_negative_test` | 63 | 0 |

Manifest integrity check:

| Check | Result |
|---|---|
| V3-E1 manifest rows | 645 |
| Train/test filename overlap | 0 |
| Train/val filename overlap | 0 |
| Val/test filename overlap | 0 |
| High-resolution original mapping issues | 0 |

## Scale-Band Composition

| Split | Scale band | Images | Boxes |
|---|---|---:|---:|
| train | Hard negative | 80 | 0 |
| train | Tiny vape | 72 | 87 |
| train | Small vape | 101 | 140 |
| train | Medium vape | 97 | 131 |
| train | Large vape | 50 | 88 |
| val | Hard negative | 45 | 0 |
| val | Tiny vape | 13 | 14 |
| val | Small vape | 18 | 20 |
| val | Medium vape | 17 | 17 |
| val | Large vape | 9 | 12 |
| test | Hard negative | 63 | 0 |
| test | Tiny vape | 12 | 14 |
| test | Small vape | 33 | 62 |
| test | Medium vape | 33 | 55 |
| test | Large vape | 2 | 4 |

## Native Image Dimensions

The V3 real-world curated images are low-resolution exports:

| Pool | Median width | Median height | Median max side | Median megapixels |
|---|---:|---:|---:|---:|
| `positive_train` | 480 | 640 | 640 | 0.3072 |
| `hard_negative_train` | 480 | 640 | 640 | 0.3072 |
| `positive_test` | 640 | 480 | 640 | 0.3072 |
| `hard_negative_test` | 640 | 480 | 640 | 0.3072 |

High-resolution originals exist and were mapped successfully in the high-resolution audit. The V3 low-resolution files are simple aspect-preserving resized exports of same-name originals to longest edge 640.

## Interpretation

The current accepted real dataset is small-to-moderate for a one-class UAV litter detector. It has good hard-negative design and a clean held-out real test split, but the positive training evidence is limited to 320 positive training images and 446 training boxes in the V3-E1 baseline split.

The planned daylight expansion should prioritise additional positive instances, additional hard negatives, surface diversity and scene-group diversity. New splits should be built by scene group rather than individual image to avoid near-duplicate leakage.

## Scraped Contextual Vape-Litter Images

Scraped images showing vapes already littered on grass, pavement, curbs or similar outdoor surfaces are potentially more domain-relevant than product-style scraped images. However, they should not be merged silently into the controlled real-world staged pool.

Recommended treatment:

| Use | Recommendation |
|---|---|
| Main official test set | Do not include scraped contextual images. Keep test real target-domain staged/captured imagery. |
| Validation set | Avoid for the main validation set unless creating a separate external-domain diagnostic validation. |
| Training | Use as a separate labelled source pool, e.g. `scraped_contextual_positive`. |
| Ablation | Compare real-only training against real + contextual-scraped training under the same fixed detector protocol. |

This preserves source transparency while allowing the model to benefit from web-sourced contextual litter imagery.

## Current Step-1 Focus Note

The immediate V4 preparation step is still the current-dataset audit, not model training. The active checklist is:

| Audit item | Status |
|---|---|
| Accepted image counts | Completed for current V3 real pool. |
| Positive vs hard-negative counts | Completed for current V3 real pool. |
| Box counts | Completed for current V3 real pool. |
| Train/val/test split counts | Completed for old V3-E1 split. |
| Image dimensions | Completed for low-resolution exports; high-resolution originals are mapped and now active in the Dataset Engine through a migrated high-resolution COCO layer. |
| Scale-band distribution | Completed for old V3-E1 split. |
| Duplicate/corrupt/rejected files | One unsupported scraped-positive GIF masquerading as JPG noted; further duplicate audit remains a follow-up if needed. |
| Split leakage risks from filenames/groups | Filename overlap is zero; future expanded data should split by scene group. |
| Annotation accuracy | Completed for the current V3 real-image pool in the UAVape Dataset Engine, now using the high-resolution COCO layer. |
| Unannotated contextual scraped imagery | Next planned data step after this current-real checkpoint; inspect, source-separate and annotate only genuinely contextual litter imagery before inclusion in training. |

For annotation correction, the UAVape Dataset Engine should be treated as the editable source-of-truth interface. As of the V4 audit pass, the legacy low-resolution `coco_master.json` has been preserved, and a separate `coco_master_highres.json` has been generated by scaling the image dimensions, COCO bounding boxes, polygon segmentations and areas to the preserved same-name originals. The engine now prefers `data/images/originals` for full-image display, SAM2 annotation and correction while retaining the legacy low-resolution curated images as a reversible reference layer. This avoids mixing low-resolution boxes with high-resolution images during the audit.

## Step 1 Freeze: Current Real Class-Aware Pool

Generated on 2026-06-10 as the current-real baseline checkpoint before adding contextual scraped imagery or the new daylight expansion images.

| Item | Value |
|---|---:|
| Active curated real images | 641 |
| Active real annotations | 945 |
| Vape boxes | 670 |
| Lighter boxes | 275 |
| Positive-source images | 459 |
| Hard-negative-source images | 182 |
| Source type: ground_capture | 641 |
| Exact duplicate hash groups | 0 |
| Warnings | 0 |

Top active image dimensions:

| Dimension | Images |
|---|---:|
| 1536x2048 | 308 |
| 4032x3024 | 156 |
| 5712x4284 | 101 |
| 2048x1536 | 54 |
| 1200x1600 | 22 |

The active `positive` and `hard_negative` records are now combined conceptually as one audited real-image pool, while preserving the original category metadata for reporting. This is not the final V4 dataset: it is the frozen current-real checkpoint used before reviewing contextual scraped images and before importing the new real daylight expansion batch.

Generated artifacts in the UAVape Dataset Engine repository:

| Artifact | Path |
|---|---|
| Freeze report | `/Users/zainahmad/.gemini/antigravity/scratch/vape-detection-dataset/data/freezes/current_real_pool_20260610_step1/CURRENT_REAL_POOL_FREEZE_REPORT.md` |
| COCO backup | `/Users/zainahmad/.gemini/antigravity/scratch/vape-detection-dataset/data/freezes/current_real_pool_20260610_step1/backups/coco_master_highres.json` |
| Metadata backup | `/Users/zainahmad/.gemini/antigravity/scratch/vape-detection-dataset/data/freezes/current_real_pool_20260610_step1/backups/download_log.jsonl` |
| Current pool manifest | `/Users/zainahmad/.gemini/antigravity/scratch/vape-detection-dataset/data/freezes/current_real_pool_20260610_step1/manifests/current_real_annotated_pool_manifest.csv` |
| YOLO multi-class export | `/Users/zainahmad/.gemini/antigravity/scratch/vape-detection-dataset/data/yolo_export/current_real_pool_20260610_step1` |

The current YOLO export is an unsplit class-aware pool. Its `dataset.yaml` uses:

```yaml
names:
  0: vape
  1: lighter
```

## Current Split Protocol Decision

For the current-real checkpoint, split assignment should be image/group-level, not object-level. Explicit `group_id`, `scene_group`, or `station_id` should be used where available; otherwise the conservative fallback is batch plus capture timestamp minute, which keeps likely near-duplicate frames together.

The generated split proposal is therefore a protocol candidate rather than the final V4 split:

| Split | Images | Vape boxes | Lighter boxes |
|---|---:|---:|---:|
| train | 449 | 514 | 155 |
| val | 96 | 62 | 77 |
| test | 96 | 94 | 43 |

For the final V4 build, this protocol should be re-run after the contextual scraped review and new real daylight expansion are complete, with scene/station metadata preferred over filename-derived grouping.

## Lighter Treatment: Auxiliary Confuser Class

The V4 detector remains a vape-focused detector. The primary claim is vape detection, not general litter detection and not high-performance lighter detection. Lighters are included as an auxiliary labelled confuser class because they are visually similar to disposable vapes and are also a meaningful litter object, rather than a benign background item.

Here, `auxiliary` means secondary to the main research target. The lighter class is used to provide explicit supervision for a visually similar non-vape object, not to make lighter detection the headline claim. The core research question remains whether disposable vape litter can be detected reliably.

This is preferable to treating visible lighters as empty hard-negative/background images. Missing-annotation object detection papers repeatedly note that when real object instances are present but unlabelled, standard detector training can treat those regions as background, creating a confusing learning signal. Yang et al. describe object detection with missing annotations as problematic because unlabelled regions are implicitly assumed to be negative/background by standard training losses. Cermelli et al. similarly argue that if no supervision is provided for visible objects, the model can learn to associate them with background regions. Zhang et al. frame this as a missing-annotation setting where unlabelled true instances are regarded as background during training.

Hard-negative literature still supports including difficult non-target examples. Jin et al. describe hard negatives as examples that a detector rates as positive or ambiguous and show that retraining with hard negatives can improve object detection. However, for UAVape, lighters are not merely empty background: they are semantically meaningful object-like confusers. Labelling them as `lighter` therefore preserves their hard-negative role for vape discrimination while avoiding the stronger assumption that the lighter object region is background.

The methodological position is:

| Option | Interpretation | Risk |
|---|---|---|
| One-class vape with lighters unlabelled/empty | Lighter regions are treated as background/no-object. | Can produce contradictory supervision because a visible object-like confuser is taught as background. |
| Two-class vape + lighter | Vape is the primary target; lighter is an auxiliary confuser/litter class. | Adds class imbalance and a secondary class that should not become the headline claim. |

For this dissertation, the second option is the cleaner default. Vape AP/precision/recall remain the main reported outcome. Lighter metrics, if reported, should be framed as secondary diagnostic evidence for confuser handling rather than as a separate litter-detection claim.

Potential downside: adding an auxiliary class can reduce vape performance if the lighter class is poorly labelled, too sparse, or visually inconsistent, because the detector now shares capacity across two classes and must learn a more complex class boundary. However, the present current-real checkpoint has 670 vape boxes and 275 lighter boxes, which is a manageable 2.4:1 ratio for an auxiliary class. The larger risk is diversity rather than raw count: lighters should appear across enough surfaces, scales and viewpoints to avoid a brittle lighter boundary.

The expected benefit should therefore be stated as methodological, not guaranteed numerical superiority. The project should not claim that a two-class model must outperform a one-class vape model unless a one-class-vs-two-class ablation is run. The defensible claim is that the two-class setup gives cleaner supervision for visible domain-relevant confusers and enables diagnostic measurement of vape/lighter confusion.

## Vape-First Metric Hierarchy for Two-Class Training

Because the model is trained with two classes but the dissertation claim is vape detection, the evaluation hierarchy should remain vape-first:

| Priority | Metric | Role |
|---|---|---|
| 1 | Vape AP50 | Headline metric for UAV/litter literature comparability and practical detection. |
| 2 | Vape AP50-95 | Secondary localisation-quality metric retained for COCO-style rigour. |
| 3 | Vape precision and recall | Operational interpretation: false vape alarms vs missed vapes. |
| 4 | Lighter AP/P/R | Secondary diagnostic for auxiliary confuser learning. |
| 5 | Macro mAP over vape + lighter | Reportable as context, but not the headline selection criterion. |

This means the class-aware detector can be selected and discussed primarily by vape-class performance, while still using lighter metrics to check whether the auxiliary class has learned a meaningful boundary. Overall two-class macro mAP should not replace vape AP as the headline metric, because it can either unfairly penalise the model for a weak auxiliary class or inflate the result if lighter detection is easy. AP50-95 should still be reported to show localisation quality; the point is not to discard strict localisation, but to avoid making a two-class aggregate metric the dissertation headline.

Recommended wording:

> Although the detector is trained with two classes, model selection and primary reporting prioritise the vape class. The lighter class is evaluated separately as an auxiliary confuser class, included to avoid background-labelling a visually similar litter object and to diagnose vape/lighter confusion.

If a one-class baseline is later run, the key comparison would be whether vape-class AP/P/R and lighter-as-vape false positives improve under the auxiliary-class setup. Until that ablation exists, the lighter class should be justified as a cleaner annotation/training design rather than as proven performance improvement.

Benchmark sources used for this rationale:

- Yang, Liang and Carin, "Object Detection as a Positive-Unlabeled Problem" (2020): missing object annotations are problematic because unlabelled regions are implicitly assumed to be background.
- Cermelli et al., "Modeling Missing Annotations for Incremental Learning in Object Detection" (2022): objects with no supervision may be associated with background regions.
- Zhang et al., "Solving Missing-Annotation Object Detection with Background Recalibration Loss" (2020): missing-labelled areas can be regarded as background during detector training.
- Jin et al., "Unsupervised Hard Example Mining from Videos for Improved Object Detection" (2018): hard negatives are ambiguous/non-target examples that can strongly influence detector correction.

# V4 Method Rationale and Execution Plan

This section replaces the older bottom-of-draft addendum in `UAVAPE_V3_DISSERTATION_MODEL_EVALUATION_DRAFT.md` as the working V4 source of truth. The old V3 addendum remains useful history, but this document should now be treated as the current V4 method rationale and execution checklist.

## V4 Data-Build Sequence

The current Step 1 freeze is complete, but it is not the final V4 dataset. The remaining data-build sequence is:

| Stage | Action | Output |
|---|---|---|
| 1. Current real checkpoint | Freeze the audited V3 real pool and class-aware annotations. | Completed current-real baseline checkpoint. |
| 2. Contextual scraped review | Find scraped images that are actually outdoor/contextual vape-litter images, import as a separate source pool and annotate. | Training-only contextual scraped pool, source-separated. |
| 3. New real daylight expansion | Import and annotate the new real images collected on 2026-06-10. | Expanded real daylight pool with fresh scene groups. |
| 4. Final V4 audit | Recount images, boxes, classes, dimensions, sources, corrupt/rejected files, duplicate risks and leakage groups. | Final V4 dataset audit. |
| 5. Fixed split manifest | Build train/val/test by scene group and source policy. | Reproducible split CSVs. |
| 6. YOLO export | Export multi-class YOLO labels and YAML. | `names: {0: vape, 1: lighter}` export ready for Colab. |
| 7. Model/resolution benchmark | Compare model family, scale and image size using validation only. | Selected detector family, scale and image size. |
| 8. Locked-protocol data ablation | Rerun data-source/dose comparisons under the selected detector protocol. | Defensible data-source findings. |
| 9. Final test evaluation | Evaluate once on the held-out test set after choices are fixed. | Main dissertation result. |

Contextual scraped images can be part of the core training pool if they are genuinely domain-relevant, but they must remain source-labelled and must not enter the official held-out test set. The new real daylight collection is the best source for constructing a clean test subset because it was captured on a different day and under different physical setups from the current audited V3 pool.

## Data Collection and Test-Set Discipline

The older station/scene/frame write-up describes the intended collection structure, not the exact final dataset count. The corrected accepted V3 real dataset was 645 real images before the current class-aware audit changes. The current class-aware freeze contains 641 active curated real images after recent audit/rejection changes.

For the new daylight expansion, collection should keep the same practical field structure:

| Unit | Practical meaning |
|---|---|
| Station | One outdoor surface patch, such as a curb, pavement area, grass patch, gravel area or leaf/soil patch. |
| Scene A | One isolated vape, photographed from several heights/angles. |
| Scene B | One isolated lighter or hard-negative object, moved slightly within the station. |
| Scene C | A mixed cluster of vapes and/or lighters/hard negatives, repositioned within the station. |
| Frame | An accepted still image after curation and rejection. |

Minimum metadata for new real images:

| Field | Why it matters |
|---|---|
| `scene_group` | Prevents near-duplicate leakage across train/val/test. |
| `station_id` | Tracks the physical ground patch. |
| `capture_session` | Keeps same-session captures together. |
| `scene_type` | Isolated vape, isolated lighter, mixed cluster or benign negative. |
| `surface_type` | Supports stratification and failure analysis. |
| `height_angle_band` | Records low/high and top-down/oblique capture conditions. |
| `source_pool` | Keeps current real, new real and contextual scraped data separable. |

Inventory IDs are optional for this pass. They would be more disciplined, but they slow field collection; surface, scene type, scene group and height/angle band are the minimum useful metadata.

The final V4 test set should not be mixed randomly from the old V3 scenes. The safer plan is:

| Data source | Recommended split role |
|---|---|
| Current audited V3 real pool | Mainly train/validation candidate pool. |
| New 2026-06-10 real daylight batch | Reserve a deliberate scene-group-held-out subset for test; remaining groups can support train/val. |
| Contextual scraped images | Training only, or a separate diagnostic validation if needed; not official test. |
| Synthetic images | Training/ablation only; not official test. |

The split must be by scene/station group, not by object or individual near-duplicate image. A slightly imperfect percentage split is preferable to leakage.

## Candidate Detector Families

The follow-up benchmark should use a small, defensible set of model families rather than an uncontrolled leaderboard.

| Candidate | Status | Strength | Risk | Proposed role |
|---|---|---|---|---|
| YOLO26s | Latest Ultralytics family with official weights and a 2026 technical paper. | NMS-free inference, lighter head, updated training recipe and small-target-aware assignment. | Less independent benchmark history than YOLO11. | Primary modern small detector. |
| YOLO26m | Same family with greater capacity. | Tests whether Nano/small capacity was limiting UAVape. | Slower and more memory-intensive. | Primary medium-capacity test. |
| YOLO26l | Same family, large scale. | High-capacity CNN ceiling without jumping to transformers. | More expensive than S/M. | Optional CNN ceiling. |
| YOLO11s | Stable modern Ultralytics detector. | Mature comparator, similar scale to YOLO26s. | Lacks YOLO26-specific changes. | Matched small-family comparator. |
| YOLO11m | Stable medium-scale comparator. | Controls for family versus capacity effects. | More compute than S. | Matched medium-family comparator. |
| YOLO11l | Stable large-scale comparator. | Large CNN baseline with more benchmark familiarity. | More expensive. | Optional large CNN comparator. |
| YOLOv8s/m | Older but still widely used. | Literature bridge to UAV/small-object papers. | Less current than YOLO11/YOLO26. | Literature-anchor baseline. |
| RT-DETR-L/R50-style | Real-time Detection Transformer, not YOLO. | Non-CNN/global-context comparator. | Heavier and less edge-aligned. | Optional architecture-family comparator. |
| YOLO12s | Attention-centric YOLO family. | Interesting research comparator. | Less central to the current official Ultralytics path. | Appendix-only if time permits. |

## Matched Scale and Size Logic

The fair comparison is not just YOLO26 versus YOLO11; it is family and capacity compared at matched scales.

| Pair | Why it is fair |
|---|---|
| YOLO26s vs YOLO11s | Tests family changes while holding capacity roughly constant. |
| YOLO26m vs YOLO11m | Tests whether family effects persist at medium scale. |
| YOLO26s vs YOLO26m | Tests capacity inside YOLO26. |
| YOLO11s vs YOLO11m | Tests capacity inside YOLO11. |
| YOLO26l/YOLO11l vs RT-DETR-L/R50 | Controls for CNN capacity before attributing differences to transformer architecture. |
| YOLOv8s/m vs modern equivalent | Provides a literature-anchor baseline. |

Approximate model-size roles from the older addendum:

| Model | Approx. parameters | Approx. FLOPs at 640 | Role |
|---|---:|---:|---|
| YOLO26n | 2.4M | 5.4B | Edge/nano baseline; likely capacity-limited for this study. |
| YOLO26s | 9.5M | 20.7B | Main lightweight-capable modern candidate. |
| YOLO26m | 20.4M | 68.2B | Main higher-capacity modern candidate. |
| YOLO26l | 24.8M | 86.4B | Optional large CNN ceiling. |
| YOLO11s | 9.4M | 21.5B | Stable modern small comparator. |
| YOLO11m | 20.1M | 68.0B | Stable modern medium comparator. |
| YOLO11l | 25.3M | 86.9B | Stable large CNN ceiling. |
| YOLOv8s | 11.2M | 28.6B | Literature-anchor small model. |
| YOLOv8m | 25.9M | 78.9B | Literature-anchor medium model. |
| RT-DETR-R50/L-style | About 42M | About 136B | Optional transformer-style comparator. |
| RT-DETR-R101/X-style | About 76M | About 259B | Likely outside dissertation scope. |

YOLO11l and YOLO26l are not automatically too large if RT-DETR-L is being considered. They are large CNNs, but still lighter than RT-DETR-R50/R101-style detectors.

## Input-Resolution Benchmark

The old `imgsz=960` setting should be treated as an inherited candidate, not a proven optimum.

| Input size | Interpretation | Why test it |
|---:|---|---|
| 640 | Standard YOLO benchmark resolution and lowest compute point. | Strong comparability and edge efficiency. |
| 960 | Inherited V3 operating point. | Plausible candidate, but not theoretically privileged. |
| 1280 | Higher-resolution small-object setting. | Tests whether extra spatial detail helps enough to justify compute. |

Relevant benchmark logic from adjacent work:

| Evidence | Input-size relevance |
|---|---|
| Standard YOLO/UAV baselines | 640 is the common benchmark and lower-compute reference. |
| RTUAV-YOLO-style UAV studies | 640 versus 1280 comparisons show accuracy/throughput trade-offs. |
| Systematic YOLOv8 VisDrone evaluations | Resolution can materially affect small-object detection. |
| Beach-litter YOLOv5 work | Very large inputs can be used when small-object fidelity is prioritised. |
| UAVape old-source E1 | 640-source E1 performed better at 640 than the earlier high-res 960/nano run, showing assumptions can fail. |

Dissertation wording:

> Because input resolution materially affects small-object detection, a preliminary resolution benchmark was performed at 640, 960 and 1280. These settings reflect standard YOLO benchmarking, the inherited V3 operating point and a higher-resolution small-object setting respectively. The selected resolution was then fixed for subsequent model-family and data-source ablations.

## Validation, Test and Checkpoint Discipline

The validation set is used during training and protocol selection. The test set is used once after model family, scale, image size, training recipe and data recipe are fixed.

| Dataset partition | Use | Must not be used for |
|---|---|---|
| Training | Updates weights. | Final generalisation claims. |
| Validation | Selects `best.pt`, compares candidate models/image sizes and guides limited HPO. | Final unbiased performance claim. |
| Test | Final held-out estimate. | Choosing model family, image size, epoch, threshold or hyperparameters. |

Ultralytics `best.pt` selection should be kept as the framework-standard checkpoint rule for reproducibility. In many Ultralytics detection versions, fitness is dominated by `mAP50-95`, often expressed as:

```text
fitness = 0.1 * mAP50 + 0.9 * mAP50-95
```

Some versions describe detection fitness as directly equal to `mAP50-95`; therefore the exact rule should be recorded from the Colab package version. This is not unique to YOLO26. YOLOv8, YOLO11 and YOLO26 trained through the same Ultralytics pipeline share the same framework-level checkpoint rule.

For UAVape, use the standard `best.pt` rule, but report the vape-first panel:

| Priority | Metric | Role |
|---|---|---|
| 1 | Vape AP50 | Headline target-class metric for comparability with UAV/litter YOLO studies and practical detection. |
| 2 | Vape AP50-95 | Secondary localisation-quality metric and COCO-style robustness check. |
| 3 | Vape precision/recall | Operational trade-off. |
| 4 | Lighter AP/P/R | Auxiliary confuser diagnostic. |
| 5 | Macro mAP over vape + lighter | Context only, not headline. |

If the epoch with best validation AP50 differs substantially from the epoch selected by Ultralytics fitness, record both in the analysis. A custom AP50-selection sensitivity run is optional, not part of the main protocol unless strongly justified.

## Hyperparameter Tuning Position

| HPO level | When | Scope | Why |
|---|---|---|---|
| Limited sanity tuning | After detector/image-size selection and before source ablation. | Learning rate/optimizer, augmentation strength, batch/accumulation and close-mosaic schedule. | Prevents a clearly poor default recipe from distorting later data-source claims. |
| Comprehensive HPO | Only after the final detector/image/data recipe is chosen, or as a labelled final optimisation branch. | Wider Ultralytics tuning/evolution or documented search. | Too expensive and confounding to repeat for every model x resolution x source condition. |

Do not run a broad independent HPO for every candidate, because that confounds model comparison with tuning effort. Broad screening should use a common default recipe; then limited tuning should be frozen for the later source-dose ablation.

## Revised Benchmark Order

The order is based on whether each step changes the detector, training data or only inference/reporting.

| Stage | Fixed factors | Changed factor | Purpose |
|---|---|---|---|
| 0. Fixed split and metric lock | Fixed manifest and metric hierarchy | None | Prevent split/metric drift. |
| 1. Unified detector benchmark | Same split, epochs, seed, recipe | YOLO family, scale and `imgsz` | Identify model, capacity and input-scale bottlenecks. |
| 2. Nano lower-bound check | Same split and recipe | YOLO26n/YOLO11n at 640/960 | Test the lower deployability boundary inside the same benchmark. |
| 3. Tiling/SAHI branch | Leading full-frame weights | Sliced inference at 640/960 | Completed as validation-only branch; static SAHI tiling did not beat full-frame high-resolution inference. |
| 4. Validation error analysis | Candidate predictions | Failure modes, false positives, missed vapes | Decide whether a data intervention is actually needed. |
| 5. Optional targeted data ablations | Frozen detector/inference protocol | Scraped, synthetic or benign-negative additions | Investigate DCE source-transfer value only if motivated by validation errors or dissertation evidence needs. |
| 6. Limited HPO / threshold sanity check | Selected model/data/inference protocol | Small recipe or operating-point adjustments | Remove obvious recipe/threshold bottlenecks using validation only. |
| 7. Freeze final protocol | Selected model, scale, `imgsz`, inference mode, data recipe and reporting threshold | None | Prevent test-set leakage. |
| 8. Final held-out test evaluation | Fully selected protocol | None | Main unbiased result. |
| 9. Optional robustness branches | Final or shortlisted protocol | Broader HPO, custom architecture, transformer, large CNN | Future/extended work only. |

Data-source ablation should not be done before detector/image-size/inference selection, because data-source findings depend on the detector protocol. The completed tiling branch indicates that static SAHI-style slicing is not the selected inference protocol for V4, so any source-dose ablation should now use the full-frame validation winner unless a clearly labelled deployment-only branch is being studied. Source ablations should be targeted and investigative unless the project explicitly decides that the training data recipe remains a live final-model choice.

## Immediate Experiment Matrix

If compute is limited, start with S-scale resolution sweeps before repeating at M-scale.

| Run | Model | Input size | Dataset protocol | Purpose |
|---|---|---:|---|---|
| R1 | YOLO26s | 640 | Fixed V4 split | Latest small model at standard resolution. |
| R2 | YOLO26s | 960 | Fixed V4 split | Latest small model at inherited operating point. |
| R3 | YOLO26s | 1280 | Fixed V4 split | Latest small model at high-resolution setting. |
| R4 | YOLO11s | 640 | Fixed V4 split | Stable small comparator at standard resolution. |
| R5 | YOLO11s | 960 | Fixed V4 split | Stable small comparator at inherited operating point. |
| R6 | YOLO11s | 1280 | Fixed V4 split | Stable small comparator at high-resolution setting. |
| R7 | YOLOv8s or YOLOv8m | 640 | Fixed V4 split | Literature-anchor baseline. |
| R8 | YOLOv8s or YOLOv8m | Selected candidate size | Fixed V4 split | Tests whether YOLOv8 remains competitive. |

After the first sweep:

| Run | Model | Input size | Dataset protocol | Purpose |
|---|---|---:|---|---|
| M1 | YOLO26s | Selected | Fixed V4 split | Latest small-model benchmark. |
| M2 | YOLO11s | Selected | Fixed V4 split | Stable small-model comparator. |
| M3 | YOLO26m | Selected | Fixed V4 split | Latest medium-capacity benchmark. |
| M4 | YOLO11m | Selected | Fixed V4 split | Stable medium-capacity comparator. |
| M5 | YOLOv8s/m | Selected | Fixed V4 split | Literature-anchor comparison. |
| M6 | YOLO26l or YOLO11l | Selected | Fixed V4 split | Large CNN ceiling if needed. |
| T1 | RT-DETR-L/R50-style | Selected/feasible | Fixed V4 split | Optional transformer-style comparator. |

## RT-DETR Clarification

RT-DETR is not a YOLO model. It is a real-time Detection Transformer supported by Ultralytics tooling. `RT-DETR-L` is not equivalent to `YOLO11l` or `YOLO26l`; the `L` suffix only means "large" within its own family.

| Model | Family | Meaning of `L` |
|---|---|---|
| YOLO11l / YOLO26l | YOLO CNN-style detector | Large YOLO scale. |
| RT-DETR-L | Real-time Detection Transformer | Large RT-DETR scale. |
| RT-DETR-X | Real-time Detection Transformer | Extra-large RT-DETR scale. |

If RT-DETR-L is tested, at least one large YOLO CNN should also be tested so that any difference is not simply capacity.

## Tiling, Hard-Negative Mining and Custom Architecture

Threshold calibration changes the operating point, not the weights. It should follow model/data selection and use validation only.

Hard-negative mining is a targeted data repair loop after error analysis. It should add or upweight false-positive confusers observed from validation/deployment-like predictions. This is useful, but it is no longer a clean source-dose ablation unless applied consistently.

Tiling/SAHI is separate from simple image-size selection. A full-frame `imgsz=640` run and a sliced inference run at `imgsz=640` are different protocols because tiling changes the field of view and preserves local detail. Treat tiling as a small-object preservation branch after the baseline detector/data recipe is understood.

Custom architecture work should be last or future work. Adjacent cigarette, smoking and UAV litter studies often modify YOLO-like detectors with:

| Modification | Why it may help |
|---|---|
| P2 detection head | Adds higher-resolution prediction for tiny objects. |
| Attention modules | Focuses weak object evidence in clutter. |
| CARAFE/improved upsampling | Preserves fine details in feature fusion. |
| NWD/EIoU/WIoU-style losses | Makes small-box localisation less brittle. |
| Backbone replacement | Changes feature representation capacity. |
| Multi-scale fusion | Combines shallow detail and deep semantics. |

These are extended detector-engineering interventions, not the first-line V4 method.

## V4 Data Intake Checkpoint: 2026-06-10

The contextual scraped-image candidate set was manually vetted and not used for the core V4 real-data pool. The available examples were too few and not sufficiently useful to justify adding another source category at this stage.

The new real daylight field-capture batch was imported directly into the UAVape Dataset Engine annotation pool, bypassing the Review queue. This matches the current workflow because the Annotation tab now supports relabel, reject, class-aware annotation, and bbox/mask correction.

| Batch | Source folder | Images found | Imported | Skipped | Engine state |
|---|---|---:|---:|---:|---|
| `v4_real_daylight_expansion_20260610` | `/Users/zainahmad/Downloads/new real data for v4` | 574 | 574 | 0 | `curated` / `positive` / `needs_label` |

Import details:

| Item | Value |
|---|---|
| First imported filename | `v4_real_daylight_expansion_20260610_0001_img_1065.jpg` |
| Curated display copies | `data/images/curated` |
| Preserved high-resolution originals | `data/images/originals` |
| Import manifest | `data/metadata/v4_real_daylight_expansion_20260610_import_manifest.csv` |
| COCO state at import | No COCO image or annotation entries yet; annotations are created during manual annotation. |

Interpretation: these 574 images should appear in Annotate -> Unannotated after the app/backend state is refreshed. They are initially marked as positive candidates only so they are available in the annotation workflow; final image-level status can still be changed to hard negative, negative, or rejected during audit.

## V4 Real Dataset Finalisation Checkpoint: 2026-06-10

After manual annotation/audit of the new daylight expansion batch, the V4 real-image dataset was frozen, audited, split and exported to YOLO format.

Final source composition:

| Source pool | Images | Vape boxes | Lighter boxes | Total boxes |
|---|---:|---:|---:|---:|
| Audited old V3 real pool | 641 | 670 | 275 | 945 |
| New V4 daylight expansion | 563 | 658 | 437 | 1095 |
| **Combined V4 real pool** | **1204** | **1328** | **712** | **2040** |

The new V4 daylight batch originally contained 574 imported images. Eleven were rejected during audit, leaving 563 active images. All 563 active new-batch images have accepted annotations.

Final split policy:

| Rule | Rationale |
|---|---|
| Test images are drawn only from the new 2026-06-10 daylight batch. | Avoids using old V3 scene families for final held-out generalisation claims. |
| Train/validation use the audited old V3 pool plus the non-test portion of the new daylight batch. | Lets validation include both legacy and new-domain evidence without touching test. |
| New-batch split assignment uses sequential capture blocks rather than individual images. | Reduces near-duplicate leakage where explicit scene/station IDs were not collected. |
| Split selection is class-aware. | Keeps enough vape and lighter instances in validation/test to support two-class reporting. |

Final split:

| Split | Images | Vape boxes | Lighter boxes | Old V3 images | New V4 images | Groups |
|---|---:|---:|---:|---:|---:|---:|
| train | 837 | 870 | 447 | 525 | 312 | 87 |
| val | 183 | 203 | 100 | 116 | 67 | 10 |
| test | 184 | 255 | 165 | 0 | 184 | 19 |

Export verification:

| Check | Result |
|---|---|
| Exported images | 1204 |
| YOLO label files | 1204 |
| Exported boxes | 2040 |
| Class IDs | `0=vape`, `1=lighter` |
| Warnings | 0 |

Generated artifacts:

| Artifact | Path |
|---|---|
| Finalisation report | `/Users/zainahmad/.gemini/antigravity/scratch/vape-detection-dataset/data/freezes/v4_real_final_20260610/V4_REAL_DATASET_FINALISATION_REPORT.md` |
| Final manifest | `/Users/zainahmad/.gemini/antigravity/scratch/vape-detection-dataset/data/freezes/v4_real_final_20260610/manifests/v4_real_final_manifest.csv` |
| Split summary | `/Users/zainahmad/.gemini/antigravity/scratch/vape-detection-dataset/data/freezes/v4_real_final_20260610/reports/split_summary.csv` |
| COCO backup | `/Users/zainahmad/.gemini/antigravity/scratch/vape-detection-dataset/data/freezes/v4_real_final_20260610/backups/coco_master_highres.json` |
| Metadata backup | `/Users/zainahmad/.gemini/antigravity/scratch/vape-detection-dataset/data/freezes/v4_real_final_20260610/backups/download_log.jsonl` |
| YOLO export root | `/Users/zainahmad/.gemini/antigravity/scratch/vape-detection-dataset/data/yolo_export/v4_real_final_20260610` |
| YOLO YAML | `/Users/zainahmad/.gemini/antigravity/scratch/vape-detection-dataset/data/yolo_export/v4_real_final_20260610/dataset.yaml` |

The exported `dataset.yaml` uses:

```yaml
path: /Users/zainahmad/.gemini/antigravity/scratch/vape-detection-dataset/data/yolo_export/v4_real_final_20260610
train: images/train
val: images/val
test: images/test
nc: 2
names:
  0: vape
  1: lighter
```

This checkpoint is now the fixed V4 real-data base for the model/resolution benchmark. The test split should not be used for model family, image-size, epoch, threshold or hyperparameter selection.

## Optional Benign-Negative Branch

The locked V4 real dataset should remain unchanged for the first detector and resolution benchmark. It already contains the core two-class formulation: `vape` as the primary class and `lighter` as an auxiliary labelled confuser class.

Benign no-target negatives are still useful, but they serve a different role. Lighters are object-like, domain-relevant confusers and are therefore labelled. Benign negatives are target-absent aerial/ground frames that contain no vape and no lighter. Their purpose is to test or improve false-positive resistance on ordinary deployment backgrounds.

The UAVVaste dataset in `/Users/zainahmad/Downloads/UAVVasteDataset` is a suitable candidate source for this branch because it contains low-altitude UAV-style outdoor imagery. However, it should not be mixed into the fixed V4 baseline before the first benchmark, because that would change the training distribution after the split/export has already been frozen.

Recommended handling:

| Step | Decision |
|---|---|
| Baseline benchmark | Use `v4_real_final_20260610` unchanged. |
| Diagnostic benign-negative set | Sample approximately 100-150 target-absent UAVVaste images/frames as a separate no-target evaluation set. |
| Optional training branch | Add sampled UAVVaste negatives as train-only empty-label images only after baseline error analysis shows false positives on benign backgrounds. |
| Official headline test | Keep the final V4 held-out test split as the primary result because it is from the same UAVape collection protocol. |

This keeps the methodology defensible: first measure the curated real vape/lighter detector, then use UAVVaste as a controlled robustness check or hard-negative mining branch if the failure analysis justifies it.

## V4 Model Benchmark Execution Log

The frozen V4 YOLO export was transferred to the training environment as:

`v4_real_final_20260610_yolo_export.tar.gz`

The archive was extracted on RunPod to:

`/workspace/uavape_v4_model_benchmark/datasets/v4_real_final_20260610`

The dataset YAML was patched so that:

```yaml
path: /workspace/uavape_v4_model_benchmark/datasets/v4_real_final_20260610
train: images/train
val: images/val
test: images/test
nc: 2
names:
  0: vape
  1: lighter
```

RunPod environment:

| Item | Value |
|---|---|
| GPU | NVIDIA H100 80GB HBM3 |
| CUDA shown by `nvidia-smi` | 13.0 |
| Training package | `ultralytics==8.4.62` |
| Dataset verification | Passed after extraction and macOS sidecar cleanup. |

Verified counts on RunPod:

| Split | Images | Labels | Boxes | Vape boxes | Lighter boxes | Missing labels | Missing images |
|---|---:|---:|---:|---:|---:|---:|---:|
| train | 837 | 837 | 1317 | 870 | 447 | 0 | 0 |
| val | 183 | 183 | 303 | 203 | 100 | 0 | 0 |
| test | 184 | 184 | 420 | 255 | 165 | 0 | 0 |

Important methodological note: these counts verify that the frozen split survived transfer correctly. The test split remains untouched and should not be used for model/image-size selection.

Known validation-label cleanup item: Ultralytics reports one duplicate label removed for `images/val/upload_image_20260603_195809_img_2331.jpg` during validation. This has not affected comparability because it is handled consistently by the framework across runs, but the source label should be cleaned before final archival/export if possible.

Current benchmark sequence:

| Order | Model | imgsz | Status |
|---:|---|---:|---|
| 1 | YOLO26s | 640 | Completed sanity/baseline run. |
| 2 | YOLO26s | 960 | Completed. |
| 3 | YOLO26s | 1280 | Completed upper-resolution check. |
| 4 | YOLO11s | 640/960/1280 | Completed. |
| 5 | YOLOv8s | 640/960/1280 | Completed literature-anchor sweep. |
| 6 | YOLO26m / YOLO11m / YOLOv8m | 640 | Completed capacity check at standard resolution. |
| 7 | YOLO26n / YOLO11n | 640/960 | Completed lower deployability-boundary check. |

Completed validation results:

| Run | Checkpoint basis | Epoch | Precision | Recall | mAP50 | mAP50-95 |
|---|---|---:|---:|---:|---:|---:|
| YOLO26s @ 640 | Best by validation `mAP50-95(B)` | 71 | 0.82493 | 0.66468 | 0.76095 | 0.54024 |
| YOLO26s @ 640 | Last epoch | 100 | 0.80521 | 0.64634 | 0.72757 | 0.51764 |
| YOLO26s @ 960 | Best by validation `mAP50-95(B)` | 90 | 0.82190 | 0.68432 | 0.74253 | 0.54957 |
| YOLO26s @ 960 | Last epoch | 100 | 0.79821 | 0.67454 | 0.73339 | 0.53865 |
| YOLO26s @ 1280 | Best by validation `mAP50-95(B)` | 77 | 0.83866 | 0.74882 | 0.80233 | 0.60501 |
| YOLO26s @ 1280 | Last epoch | 100 | 0.81750 | 0.75050 | 0.78467 | 0.58859 |

Interpretation: the first run confirms the V4 export trains correctly. `best.pt` outperforms `last.pt`, supporting the planned use of Ultralytics checkpoint selection during validation-only model/image-size selection. Within YOLO26s, moving from `imgsz=640` to `imgsz=960` slightly increased validation mAP50-95 and recall but slightly reduced mAP50. The `1280` upper-resolution check produced the strongest aggregate validation result so far, increasing precision, recall, mAP50 and mAP50-95. Because `1280` is less deployment-feasible, it should be interpreted as an accuracy/upper-resolution result unless per-class vape AP and later model-family comparisons justify adopting it as the final protocol.

Per-class validation extraction from each saved `best.pt`:

| Run | Class | Precision | Recall | AP50 | AP50-95 |
|---|---|---:|---:|---:|---:|
| YOLO26s @ 640 | vape | 0.88375 | 0.73762 | 0.87651 | 0.64776 |
| YOLO26s @ 640 | lighter | 0.74003 | 0.60000 | 0.64725 | 0.43683 |
| YOLO26s @ 960 | vape | 0.89150 | 0.77282 | 0.85692 | 0.68355 |
| YOLO26s @ 960 | lighter | 0.75517 | 0.62000 | 0.62225 | 0.41171 |
| YOLO26s @ 1280 | vape | 0.90879 | 0.83851 | 0.91763 | 0.70669 |
| YOLO26s @ 1280 | lighter | 0.76984 | 0.66000 | 0.68792 | 0.49768 |

Vape-first interpretation: YOLO26s @ 1280 is currently strongest on the primary class, with vape AP50-95 = 0.70669 and vape AP50 = 0.91763. YOLO26s @ 960 improves vape AP50-95 over 640 but reduces vape AP50. The deployment trade-off remains open because 1280 is less edge-feasible than 640/960.

Post-overnight unified benchmark update:

The benchmark should be treated as one detector-selection experiment across family, capacity and input resolution, not as separate "accuracy" and "deployment" studies. Small models form the centre of the benchmark, medium models at 640 test capacity at standard resolution, and nano models at 640/960 test the lower deployability boundary. Large and extra-large models remain outside the main scope.

Nano lower-bound results:

| Run | Best epoch | Params / GFLOPs | Best weight size | Precision | Recall | mAP50 | mAP50-95 | Vape AP50 | Vape AP50-95 | Lighter AP50 | Lighter AP50-95 |
|---|---:|---|---|---:|---:|---:|---:|---:|---:|---:|---:|
| YOLO26n @ 640 | 74 | 2.38M / 5.2 | 5.4 MB | 0.747 | 0.643 | 0.687 | 0.459 | 0.829 | 0.598 | 0.545 | 0.320 |
| YOLO26n @ 960 | 78 | 2.38M / 5.2 | 5.4 MB | 0.773 | 0.684 | 0.729 | 0.527 | 0.835 | 0.633 | 0.623 | 0.422 |
| YOLO11n @ 640 | 89 | 2.58M / 6.3 | 5.5 MB | 0.699 | 0.681 | 0.688 | 0.456 | 0.853 | 0.587 | 0.522 | 0.325 |
| YOLO11n @ 960 | 90 | 2.58M / 6.3 | 5.5 MB | 0.843 | 0.671 | 0.765 | 0.545 | 0.878 | 0.645 | 0.653 | 0.445 |

Nano interpretation: nano models do not replace the strongest small-model protocol, but the 960-pixel nano runs are materially stronger than their 640-pixel equivalents. YOLO11n @ 960 is the best nano result and approaches YOLO26s @ 640 on vape AP50-95 while using a smaller checkpoint. This supports the view that the bottleneck is not only model capacity; preserving small-object detail through input resolution remains important even at nano scale.

Completed aggregate validation ranking:

| Rank | Run | Best epoch | Precision | Recall | mAP50 | mAP50-95 |
|---:|---|---:|---:|---:|---:|---:|
| 1 | YOLO26s @ 1280 | 77 | 0.83866 | 0.74882 | 0.80233 | 0.60501 |
| 2 | YOLO11s @ 1280 | 74 | 0.83625 | 0.75079 | 0.78559 | 0.58851 |
| 3 | YOLO11s @ 960 | 89 | 0.82587 | 0.74317 | 0.77487 | 0.56639 |
| 4 | YOLO26m @ 640 | 82 | 0.77691 | 0.75584 | 0.79481 | 0.56215 |
| 5 | YOLO26s @ 960 | 90 | 0.82190 | 0.68432 | 0.74253 | 0.54957 |
| 6 | YOLO11n @ 960 | 90 | 0.84300 | 0.67100 | 0.76500 | 0.54500 |
| 7 | YOLOv8s @ 1280 | 76 | 0.85542 | 0.66381 | 0.73735 | 0.54448 |
| 8 | YOLO26s @ 640 | 71 | 0.82493 | 0.66468 | 0.76095 | 0.54024 |
| 9 | YOLOv8s @ 960 | 89 | 0.82325 | 0.67099 | 0.73797 | 0.53573 |
| 10 | YOLOv8m @ 640 | 88 | 0.78257 | 0.71402 | 0.75316 | 0.53368 |
| 11 | YOLO26n @ 960 | 78 | 0.77300 | 0.68400 | 0.72900 | 0.52700 |
| 12 | YOLO11m @ 640 | 81 | 0.86303 | 0.68856 | 0.75816 | 0.52415 |
| 13 | YOLO11s @ 640 | 87 | 0.79727 | 0.69695 | 0.74736 | 0.50696 |
| 14 | YOLOv8s @ 640 | 69 | 0.77860 | 0.68419 | 0.71935 | 0.48731 |
| 15 | YOLO26n @ 640 | 74 | 0.74700 | 0.64300 | 0.68700 | 0.45900 |
| 16 | YOLO11n @ 640 | 89 | 0.69900 | 0.68100 | 0.68800 | 0.45600 |

Completed vape-class validation ranking:

| Rank | Run | Vape precision | Vape recall | Vape AP50 | Vape AP50-95 |
|---:|---|---:|---:|---:|---:|
| 1 | YOLO26s @ 1280 | 0.90879 | 0.83851 | 0.91763 | 0.70669 |
| 2 | YOLO26s @ 960 | 0.89150 | 0.77282 | 0.85692 | 0.68355 |
| 3 | YOLO11s @ 1280 | 0.87633 | 0.84194 | 0.87978 | 0.67722 |
| 4 | YOLO11s @ 960 | 0.88042 | 0.83833 | 0.90301 | 0.67675 |
| 5 | YOLO26m @ 640 | 0.84197 | 0.84402 | 0.88876 | 0.66603 |
| 6 | YOLOv8s @ 960 | 0.87992 | 0.80198 | 0.86633 | 0.65877 |
| 7 | YOLOv8m @ 640 | 0.87599 | 0.80429 | 0.87318 | 0.64843 |
| 8 | YOLO11m @ 640 | 0.87763 | 0.78218 | 0.88178 | 0.64840 |
| 9 | YOLO26s @ 640 | 0.88375 | 0.73762 | 0.87651 | 0.64776 |
| 10 | YOLO11n @ 960 | 0.87800 | 0.76200 | 0.87800 | 0.64500 |
| 11 | YOLO11s @ 640 | 0.91750 | 0.79208 | 0.89317 | 0.63779 |
| 12 | YOLO26n @ 960 | 0.82900 | 0.72800 | 0.83500 | 0.63300 |
| 13 | YOLOv8s @ 1280 | 0.83696 | 0.73762 | 0.80318 | 0.61764 |
| 14 | YOLO26n @ 640 | 0.84400 | 0.72800 | 0.82900 | 0.59800 |
| 15 | YOLOv8s @ 640 | 0.85827 | 0.78218 | 0.83862 | 0.59569 |
| 16 | YOLO11n @ 640 | 0.78000 | 0.80200 | 0.85300 | 0.58700 |

Current interpretation: YOLO26s @ 1280 remains strongest under both vape AP50 and vape AP50-95. Nano results do not overturn the primary benchmark, but they add an important deployment boundary: 960-pixel nano models, especially YOLO11n @ 960, are competitive with several 640-pixel small/medium settings on vape AP. Medium models at 640 still do not beat the strongest high-resolution small models, which supports the interpretation that small-object spatial detail is a primary bottleneck.

## V4 SAHI Tiling Validation Branch

SAHI-style tiled inference was evaluated as a validation-only branch using the frozen validation split. The branch used fixed 1280-pixel slices with 20% overlap and existing full-frame checkpoints; no training or epoch search was performed. The aim was to test whether lower-resolution tiled inference could preserve local small-object detail without requiring full-frame 1280 inference.

| Run | Predictions | Aggregate AP50 | Aggregate AP50-95 | Vape AP50 | Vape AP50-95 | Lighter AP50 | Lighter AP50-95 |
|---|---:|---:|---:|---:|---:|---:|---:|
| YOLO26s @ 640 + SAHI 1280/20% | 9131 | 0.63982 | 0.48436 | 0.72741 | 0.55774 | 0.55223 | 0.41098 |
| YOLO26s @ 960 + SAHI 1280/20% | 7568 | 0.69592 | 0.51027 | 0.80554 | 0.60559 | 0.58631 | 0.41494 |
| YOLO11s @ 960 + SAHI 1280/20% | 10449 | 0.68078 | 0.51124 | 0.76595 | 0.58479 | 0.59561 | 0.43769 |
| YOLO11n @ 960 + SAHI 1280/20% | 14250 | 0.67804 | 0.50773 | 0.75871 | 0.56937 | 0.59736 | 0.44609 |

Tiling decision: this branch is not selected for the final V4 protocol. All tested tiled configurations underperformed the corresponding full-frame high-resolution candidates on the primary vape metrics, and prediction counts were high relative to 302 validation annotations. The most plausible interpretation is that static slicing increased false-positive/duplicate pressure and reduced useful scene context. This is a useful negative result rather than a failed experiment: it shows that the UAVape validation set benefits more from full-frame high-resolution inference than from the tested static tiled inference protocol.

## V4 Failure-Mode Metadata Simplification

The active annotation metadata was simplified because the earlier taxonomy was too broad for the staged real-image workflow. Metadata is retained for diagnostic/failure-mode analysis, not for YOLO training.

Active fields:

| Field | Active values / interpretation |
|---|---|
| `surface` | `grass`, `pavement`, `soil`, `curb`, `unknown`. Removed gravel/leaves/indoor to reduce noisy over-labelling. |
| `lighting` | `daylight`, `overcast`, `shadowed`, `unknown`. |
| `occlusion` | `none`, `partial`, `heavy`, `unknown`; records physical hiding of the target. |
| `background_clutter` | `none`, `low`, `medium`, `high`, `unknown`; should refer mainly to competing objects/litter/vapes/lighters, not natural texture alone. |
| `motion_blur` | `none`, `slight`, `high`, `unknown`. |

Inactive or derived fields:

| Field | Decision |
|---|---|
| `condition` | Not active for V4 because the real images are staged and most objects are not crushed/degraded. |
| `viewpoint` | Not active because it is visually subjective and less useful than actual model failure analysis. |
| `scale_band` | Not manually labelled. Scale should be derived from bbox area divided by image area during dataset audit/preprocessing. |

The VLM metadata assistant should run automatically when a selected image has missing or failed metadata. Existing suggested/reviewed metadata is not force-regenerated on every click, to avoid API waste and to preserve the AI-vs-human correction audit trail. If the backend has no `OPENAI_API_KEY`, the assistant reports that explicitly and will not silently queue a failed background job.

## Key Sources To Formalise Later

| Topic | Source / URL |
|---|---|
| YOLO26 documentation | https://docs.ultralytics.com/models/yolo26 |
| YOLO11 documentation | https://docs.ultralytics.com/models/yolo11 |
| YOLOv8 documentation | https://docs.ultralytics.com/models/yolov8 |
| RT-DETR documentation | https://docs.ultralytics.com/models/rtdetr |
| YOLO26 paper | https://arxiv.org/abs/2606.03748 |
| RT-DETR paper | https://arxiv.org/abs/2304.08069 |
| COCO detection metrics | https://cocodataset.org/#detection-eval |
| VisDrone detection protocol | https://aiskyeye.com/evaluate/object-detection-2022/ |
| Object detection as positive-unlabelled problem | https://arxiv.org/abs/2002.04672 |
| Missing annotations in object detection | https://arxiv.org/abs/2204.08766 |
| Background recalibration for missing annotations | https://arxiv.org/abs/2002.05274 |
| Unsupervised hard-example mining | https://arxiv.org/abs/1808.04285 |
| Online hard-example mining | https://arxiv.org/abs/1604.03540 |
| SAHI slicing-aided inference/fine-tuning | https://arxiv.org/abs/2202.06934 |
| ASAHI adaptive slicing | https://arxiv.org/abs/2604.19233 |
