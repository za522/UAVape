# UAVape V3 YOLO26n Data-Centric Ablation Protocol

Purpose: define the restarted V3 methodology using the current lightweight Ultralytics nano detector, `YOLO26n`.

This document is the active V3 dissertation write-up scaffold. The earlier YOLOv10n E1-E3 outputs are retained only as a superseded pilot record. They should not be reported as the final V3 evidence unless explicitly framed as preliminary infrastructure testing.

## Restart Decision

The V3 experiment is restarted with `YOLO26n` because the main study is intended to evaluate UAVape data-construction pathways under a contemporary lightweight YOLO detector. Ultralytics currently describes YOLO26 as its latest YOLO model line, with `YOLO26n` as the nano-scale model. Using the newest nano detector keeps the dissertation framing cleaner than continuing with YOLOv10n after discovering a newer edge-oriented model family.

The restart changes one controlled infrastructure variable before the official V3 matrix begins:

- old pilot detector: `yolov10n.pt`;
- official V3 detector: `yolo26n.pt`;
- image size remains `960`;
- split logic remains fixed;
- source-dose logic remains fixed;
- official fixed validation/test protocol remains fixed.

The YOLOv10n runs should be described, if needed, as pilot runs that verified the V3 pipeline, backup process and interpretation logic. The official V3 results start again from V3-E1 with YOLO26n.

## Research Framing

### Main Contribution

UAVape is a domain-adaptable dataset construction engine for object-detection research. It supports:

- upload and curation of real target-domain images;
- hard-negative collection;
- scraped image intake, audit and labelling;
- synthetic image generation;
- SAM2-assisted annotation;
- metadata tagging and source auditing;
- YOLO dataset export;
- data-centric ablation evaluation.

### Case Study

The engine is evaluated on disposable vape litter detection in UAV-style urban imagery. This is a difficult small-object detection case because disposable vapes are:

- small or tiny in frame;
- reflective and cylindrical;
- visually similar to pens, lighters, wrappers, batteries and other litter;
- underrepresented in existing public detection datasets;
- relevant to environmental monitoring and small e-waste recovery.

### Core Research Question

How do different UAVape dataset construction pathways affect detector performance under a fixed real-world UAVape validation/test protocol?

### V3 Experimental Question

When the model, validation set and test set are fixed, how does detector performance change as curated scraped data and pure synthetic data are added at matched source-dose levels?

V3 therefore tests for source-dose behaviour:

- a beneficial low-dose range;
- a plateau where additional data adds little value;
- or a high-dose degradation pattern where additional source data reduces target-domain performance.

## Scope Controls

V3 is a data-centric ablation study. It does not attempt to optimise detector architecture.

Held fixed across all official V3 experiments:

- detector: `YOLO26n`;
- pretrained starting weights: `yolo26n.pt`;
- image size: `960`;
- training schedule: 100 epochs;
- checkpoint rule: highest validation mAP50-95;
- random seed: 42;
- validation set;
- test set;
- evaluation metrics;
- source-dose sampling logic.

This keeps the comparison focused on the dataset construction engine rather than on detector capacity.

## Detector Configuration

| Item | Official V3 setting | Rationale |
|---|---|---|
| Detector | YOLO26n | Current Ultralytics nano-scale detector; suitable for lightweight/edge-oriented framing. |
| Pretrained weights | `yolo26n.pt` | COCO-pretrained transfer learning in a small custom dataset regime. |
| Image size | 960 | Preserves small-object detail better than 640 while staying feasible on the available GPU. |
| Epochs | 100 | Allows convergence across all source-dose conditions. |
| Patience | 100 | Allows every run to complete the full schedule while still saving `best.pt`. |
| Batch | 16 | Stable batch size used across all experiments. |
| Seed | 42 | Reproducibility and fixed sampling. |
| Optimizer | `auto` | Ultralytics-selected optimiser, fixed across experiments. |
| Checkpoint rule | highest validation mAP50-95 | Stricter localisation-aware checkpoint selection. |

### Why Nano?

The nano variant is used because the dissertation is not primarily asking which detector size is best. It is asking which data construction pathway helps under a fixed lightweight detector. Nano also matches the UAV/near-edge motivation better than a larger small or medium model, and it keeps the full source-dose matrix practical.

Using a larger model such as YOLO26s could improve absolute metrics, but it would introduce model capacity as a second variable. A YOLO26s run may be useful later as a robustness or capacity check on the best data recipe, but it should not replace the fixed-detector V3 matrix.

## Dataset Splits

The validation and test sets are stratified mixed real evaluation sets. Both contain positive vape images and lighter-based hard-negative no-vape images at the same positive-to-hard-negative proportion:

| Split | Positive images | Hard-negative images | Total images | Positive proportion | Hard-negative proportion |
|---|---:|---:|---:|---:|---:|
| Validation | 57 | 45 | 102 | 55.9% | 44.1% |
| Test | 80 | 63 | 143 | 55.9% | 44.1% |

This matched composition means validation checkpoint selection and official test evaluation are exposed to comparable positive/no-vape class balance while remaining disjoint. The validation set is used only for checkpoint selection, and the test set is used only for held-out reporting.

### Fixed Validation Set

Use the fixed mixed real validation set:

- real positive images;
- lighter-based hard-negative images;
- no scraped images;
- no synthetic images.

### Fixed Test Set

Use the fixed mixed real test set:

- real positive UAVape images;
- lighter-based hard-negative images;
- no scraped images;
- no synthetic images.

This means all V3 experiments are evaluated on the target deployment-like domain.

## Source Pools

| Pool | Role |
|---|---|
| Real positives | Target-domain positive baseline. |
| Hard negatives | Controlled false-positive resistance against visually similar non-vape objects, primarily lighters. |
| Curated scraped positives | Scrape/audit/label pathway; real-world vape appearance diversity with possible domain mismatch. |
| Pure synthetic positives | Synthetic pathway; controlled generated/cutout vape imagery with different scale/resolution characteristics. |
| Optional copy-paste positives | Real-instance copy-paste branch, included only if available and audited. |

### Hard-Negative Scope

The current hard-negative subset is mainly lighter imagery. This is useful because lighters resemble disposable vapes in size, shape, colour range, material finish and litter context. However, it does not prove robustness to all possible non-vape litter classes.

Dissertation wording should therefore say **lighter-like hard-negative robustness**, not universal hard-negative robustness.

## V3 Experiment Matrix

| Experiment | Training add-on beyond real positives + hard negatives | Purpose |
|---|---|---|
| V3-E1 | none | Operational baseline. |
| V3-E2 | 88 curated scraped positives | Low scraped dose. |
| V3-E3 | 177 curated scraped positives | Medium scraped dose. |
| V3-E4 | 355 curated scraped positives | Full matched scraped dose. |
| V3-E5 | 88 pure synthetic positives | Low synthetic dose. |
| V3-E6 | 177 pure synthetic positives | Medium synthetic dose. |
| V3-E7 | 355 pure synthetic positives | Full synthetic dose. |

Optional copy-paste experiments may be added only if the input zip exists and the samples can be audited properly.

## Sampling Rules

For scraped and synthetic subsets:

- sample with fixed seed 42;
- keep subsets nested where possible:
  - 88-image subset is contained inside the 177-image subset;
  - 177-image subset is contained inside the 355-image subset;
- use stratified nested sampling by `scale_band`;
- avoid duplicate image stems;
- save sampled manifests.

The aim is not to test deliberately larger or smaller boxes. The aim is to test source-dose while keeping the object-scale composition approximately stable within each pathway.

## Known Source Characteristics

The earlier audit established these source characteristics, which should be re-generated in the YOLO26n output folder before training:

- fixed real positive and hard-negative images are mostly 640 x 480 or 480 x 640 and are upscaled into `imgsz=960`;
- scraped positives have heterogeneous native dimensions;
- pure synthetic positives are much higher resolution and are downscaled into `imgsz=960`;
- scraped additions are naturally more large/medium-heavy;
- pure synthetic additions are naturally more tiny/small-heavy.

This means cross-pathway interpretation must consider source composition as well as source quantity. Within-pathway dose comparisons remain the cleanest source-dose evidence.

### Selected Add-On Native Resolution Audit

The selected add-on subsets were also audited after dataset materialisation. This audit checks the native dimensions of the actual scraped and synthetic images used in the E2-E7 training folders before YOLO normalises them to `imgsz=960`.

| Experiment | Source | Selected images | Median width | Median height | Median max side | Median MP | % max side > 960 | % max side = 960 | % max side < 960 |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|
| V3-E2 | scraped | 88 | 607 | 496 | 644.5 | 0.285357 | 31.8182 | 2.2727 | 65.9091 |
| V3-E3 | scraped | 177 | 587 | 478 | 640.0 | 0.290004 | 27.1186 | 3.3898 | 69.4915 |
| V3-E4 | scraped | 355 | 600 | 477 | 640.0 | 0.299568 | 26.7606 | 4.7887 | 68.4507 |
| V3-E5 | synthetic | 88 | 2048 | 2048 | 2048.0 | 4.194304 | 100.0000 | 0.0000 | 0.0000 |
| V3-E6 | synthetic | 177 | 3668 | 2048 | 3668.0 | 5.971504 | 100.0000 | 0.0000 | 0.0000 |
| V3-E7 | synthetic | 355 | 3668 | 2048 | 3668.0 | 5.971504 | 100.0000 | 0.0000 | 0.0000 |

The scraped subsets are broadly similar in native-resolution profile across E2-E4: their median max side remains close to 640 pixels and roughly two-thirds of images have max side below 960. This makes it less likely that E2-E4 differences are driven solely by an obvious resolution imbalance between scraped dose subsets, although native resolution remains an audited source characteristic rather than a controlled sampling stratum.

The synthetic subsets are sharply different from scraped and real target-domain imagery: all selected synthetic images have native max side above 960 and are therefore downscaled during YOLO training. This should be considered when interpreting synthetic-pathway results, alongside object-scale composition and visual-domain characteristics.

## Required Outputs Per Experiment

Immediately back up to Drive after every train/eval cycle.

Required files:

- `weights/best.pt`;
- `weights/last.pt`;
- `results.csv`;
- `results.png`;
- confusion matrices if generated;
- PR/P/R/F1 curves if generated;
- labels plot;
- per-experiment training record CSV;
- per-experiment official test metrics CSV;
- incremental training summary CSV;
- incremental official test metrics CSV;
- dataset YAML.

The active restart backup path is:

`/content/drive/MyDrive/uavape_ablation/outputs_v3_yolo26n`

## Required Summary Tables

### Dataset Composition

For each experiment, report:

- real positive images;
- hard-negative images;
- scraped positive images;
- pure synthetic positive images;
- total train images;
- total train boxes;
- fixed validation image count;
- fixed test image count.

### Official Test Metrics

Columns:

- experiment;
- model;
- pathway;
- dose;
- precision;
- recall;
- mAP50;
- mAP50-95;
- best epoch;
- validation mAP50-95;
- interpretation.

### Source-Dose Effect

Columns:

- source pathway;
- dose count;
- precision delta vs V3-E1;
- recall delta vs V3-E1;
- mAP50 delta vs V3-E1;
- mAP50-95 delta vs V3-E1.

### Hard-Negative False Positives

Columns:

- experiment;
- hard-negative test images;
- false-positive detections;
- hard-negative images with at least one false positive;
- false positives per hard-negative image;
- max false-positive confidence.

This table directly tests false-positive resistance on the fixed lighter-based hard-negative subset.

### Validation-Test Gap

Validation metrics select `best.pt`; test metrics report held-out performance. The dissertation should discuss the gap between them, especially if a condition looks strong on validation but weaker on the fixed test set.

Running validation-test gap summary:

| Experiment | Model | Validation precision | Test precision | Precision gap | Validation recall | Test recall | Recall gap | Validation mAP50 | Test mAP50 | mAP50 gap | Validation mAP50-95 | Test mAP50-95 | mAP50-95 gap |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| V3-E1 | YOLO26n | 0.755270 | 0.694859 | -0.060411 | 0.936510 | 0.775934 | -0.160576 | 0.836100 | 0.775894 | -0.060206 | 0.733280 | 0.643398 | -0.089882 |
| V3-E2 | YOLO26n | 0.813260 | 0.755328 | -0.057932 | 0.904760 | 0.755556 | -0.149204 | 0.861380 | 0.751819 | -0.109561 | 0.763890 | 0.601064 | -0.162826 |
| V3-E3 | YOLO26n | 0.779130 | 0.779879 | +0.000749 | 0.920630 | 0.708604 | -0.212026 | 0.851250 | 0.791869 | -0.059381 | 0.749120 | 0.634267 | -0.114853 |
| V3-E4 | YOLO26n | 0.833870 | 0.698620 | -0.135250 | 0.809520 | 0.666667 | -0.142853 | 0.857860 | 0.734826 | -0.123034 | 0.745850 | 0.593865 | -0.151985 |
| V3-E5 | YOLO26n | 0.732610 | 0.737137 | +0.004527 | 0.873020 | 0.659259 | -0.213761 | 0.845240 | 0.714588 | -0.130652 | 0.745680 | 0.572183 | -0.173497 |
| V3-E6 | YOLO26n | 0.761590 | 0.721116 | -0.040474 | 0.873020 | 0.727841 | -0.145179 | 0.794940 | 0.730766 | -0.064174 | 0.698530 | 0.603659 | -0.094871 |
| V3-E7 | YOLO26n | 0.809150 | 0.737111 | -0.072039 | 0.857140 | 0.718519 | -0.138621 | 0.830700 | 0.725513 | -0.105187 | 0.741890 | 0.598678 | -0.143212 |

## Required Figures And When To Generate Them

V3 should separate **dataset-characterisation figures** from **model-behaviour figures**. Dataset-characterisation figures can be generated before training because they depend only on the manifests, annotations and source pools. Model-behaviour figures should be generated after training because they depend on predictions, curves and saved model outputs.

### Pre-Training Dataset Figures

These figures explain what the ablation changes before any model is trained:

| Figure | Source artifact | Purpose | When to generate |
|---|---|---|---|
| Dataset/source composition bar chart | `v3_yolo26n_dataset_composition_by_source.csv` | Shows exactly what each experiment trains on. | After dataset materialisation. |
| Validation/test composition table or bar chart | `v3_yolo26n_dataset_composition_by_source.csv` | Shows the matched positive/hard-negative split in validation and test. | After dataset materialisation. |
| Scale-band composition stacked bar chart | `v3_yolo26n_scale_band_composition.csv` | Shows tiny/small/medium/large/hard-negative composition across train/val/test. | After dataset materialisation. |
| Native image dimension summary | `v3_yolo26n_native_image_dimension_summary.csv` | Shows real images are upscaled to 960, synthetic downscaled and scraped mixed. | Before training. |
| Top native dimension pairs table | `v3_yolo26n_top_native_dimension_pairs_by_pool.csv` | Supports the image-size/source-quality discussion. | Before training. |
| Source-pool audit table | `v3_yolo26n_input_pool_summary.csv` | Documents accepted image/box counts and rejected files. | Before training. |

These can be made now or after training. The data already exists once Cells 1-5 complete. It is usually more efficient to save the CSVs now, train the models, and make polished dissertation figures after the full results are available.

### Post-Training Model Figures

These figures explain how each model behaved:

| Figure | Source artifact | Purpose | When to generate |
|---|---|---|---|
| Official metric comparison | `v3_yolo26n_official_test_metrics_incremental.csv` | Compares precision, recall, mAP50 and mAP50-95 across E1-E7. | After each experiment, finalised after E7. |
| Source-dose curve | official test metrics + source-dose table | Shows whether scraped/synthetic dose helps, plateaus or hurts. | After E4 for scraped; after E7 for full comparison. |
| Validation-test gap chart/table | `v3_yolo26n_validation_test_gap_summary.csv` | Shows whether validation improvements transfer to held-out test. | After experiments finish. |
| Training curves | each run's `results.csv` / `results.png` | Shows convergence and checkpoint behaviour. | After each run; compare after all runs. |
| Confusion matrices | Ultralytics run outputs | Gives quick class-level diagnostic context. | After each run. |
| PR/P/R/F1 curves | Ultralytics run outputs | Explains precision-recall tradeoff and threshold behaviour. | After each run. |
| Hard-negative false-positive chart | `v3_yolo26n_hard_negative_false_positive_summary.csv` | Tests false alarms on lighter-based no-vape images. | After prediction diagnostics. |
| Scale-band performance chart/table | `v3_yolo26n_scale_band_performance.csv` | Shows whether tiny/small vapes drive failures. | After prediction diagnostics. |
| TP/FP confidence distribution | `v3_yolo26n_tp_fp_confidence_summary.csv` and prediction table | Shows whether false positives are low-confidence or risky high-confidence detections. | After prediction diagnostics. |
| Threshold diagnostic plot | `v3_yolo26n_threshold_diagnostics.csv` | Shows how confidence threshold changes precision, recall and hard-negative FP rate. | After prediction diagnostics. |
| Qualitative montage of TP/FP/FN/poor localisation | prediction-level outputs and saved images | Grounds metric changes in visible examples. | After prediction diagnostics. |

The main dissertation should not show every generated Ultralytics plot for every experiment. It should show comparison figures where comparison is the point, and move repetitive per-experiment curves/matrices to the appendix.

## Metric Interpretation

- **True positive:** a predicted vape box correctly matches a labelled vape instance at the required IoU threshold.
- **False positive:** a predicted vape box does not match any labelled vape instance, including detections on hard-negative images.
- **False negative:** a labelled vape instance is missed.
- **True negative:** a no-vape image or region correctly has no vape prediction, but true negatives are not central in object detection because possible background regions are not fixed.

Core metrics:

- **Precision:** how many predicted vape detections are correct. Higher precision means fewer false alarms.
- **Recall:** how many labelled vape instances are detected. Higher recall means fewer missed vapes.
- **mAP50:** mean average precision at IoU 0.50; useful for approximate localisation.
- **mAP50-95:** stricter localisation-aware mAP averaged across IoU thresholds from 0.50 to 0.95; used for validation checkpoint selection.

Because disposable vape litter is hazardous micro-e-waste, recall is operationally important. Precision remains important because excessive false positives increase review burden and reduce trust.

### Primary Versus Diagnostic Objectives

The primary V3 objective is to test whether each data-construction pathway improves official held-out test performance relative to V3-E1. The main comparison is therefore the fixed test-set delta in precision, recall, mAP50 and especially mAP50-95.

The validation-test gap is a diagnostic objective, not the main success criterion. It explains whether validation gains transferred to the fixed real test set. A source-dose condition is most useful when it improves official test metrics and does not create a large validation-test gap. A condition that improves validation but not test performance is evidence of limited transfer from the added source pool to the target UAVape test domain.

Prediction-level diagnostics are also diagnostic rather than primary success criteria. The diagnostic prediction pass uses a deliberately low confidence threshold (`conf=0.001`) so that weak TP and FP candidates can be inspected. Therefore, the false-positive counts in the prediction-level tables should not be reported as deployment-threshold false-positive rates. They are useful for comparing relative model behaviour, confidence separation and hard-negative sensitivity before threshold calibration.

## Validation And Test Reporting Rule

Validation metrics and test metrics serve different purposes.

Validation metrics are used during training to select `best.pt`. In V3, `best.pt` is selected by the highest validation mAP50-95.

Test metrics are the official held-out results. The test set is evaluated once using the selected `best.pt`. Do not call these "test best" metrics. The correct wording is:

**official fixed test metrics using `best.pt`.**

## Running Official Test Summary

This table should be filled as the YOLO26n restart runs complete.

| Experiment | Model | Pathway | Dose | Precision | Recall | mAP50 | mAP50-95 | Delta precision vs E1 | Delta recall vs E1 | Delta mAP50 vs E1 | Delta mAP50-95 vs E1 | Interim interpretation |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---|
| V3-E1 | YOLO26n | baseline | 0 | 0.694859 | 0.775934 | 0.775894 | 0.643398 | 0.000000 | 0.000000 | 0.000000 | 0.000000 | Official YOLO26n operational baseline. |
| V3-E2 | YOLO26n | scraped | 88 | 0.755328 | 0.755556 | 0.751819 | 0.601064 | +0.060469 | -0.020378 | -0.024075 | -0.042334 | Improved test precision, but reduced recall and localisation-aware mAP versus E1. |
| V3-E3 | YOLO26n | scraped | 177 | 0.779879 | 0.708604 | 0.791869 | 0.634267 | +0.085020 | -0.067330 | +0.015975 | -0.009131 | Highest precision so far and improved mAP50, but lower recall and slightly lower mAP50-95 than E1. |
| V3-E4 | YOLO26n | scraped | 355 | 0.698620 | 0.666667 | 0.734826 | 0.593865 | +0.003761 | -0.109267 | -0.041068 | -0.049533 | Full scraped dose fell below E1 on recall, mAP50 and mAP50-95; precision returned close to baseline. |
| V3-E5 | YOLO26n | pure synthetic | 88 | 0.737137 | 0.659259 | 0.714588 | 0.572183 | +0.042278 | -0.116675 | -0.061306 | -0.071215 | Low synthetic dose improved precision over E1, but reduced recall and both mAP metrics; largest mAP50-95 validation-test gap so far. |
| V3-E6 | YOLO26n | pure synthetic | 177 | 0.721116 | 0.727841 | 0.730766 | 0.603659 | +0.026257 | -0.048093 | -0.045128 | -0.039739 | Medium synthetic dose recovered recall and mAP50-95 versus E5, but remained below E1 on recall and both mAP metrics. |
| V3-E7 | YOLO26n | pure synthetic | 355 | 0.737111 | 0.718519 | 0.725513 | 0.598678 | +0.042252 | -0.057415 | -0.050381 | -0.044720 | Full synthetic dose improved precision over E1, but remained below E1 on recall, mAP50 and mAP50-95; similar to E6 on official test. |

## Running Validation Checkpoint Summary

This table records checkpoint selection. It is not the official held-out test result.

| Experiment | Model | Best epoch by validation mAP50-95 | Best validation precision | Best validation recall | Best validation mAP50 | Best validation mAP50-95 | Last epoch | Last validation precision | Last validation recall | Last validation mAP50 | Last validation mAP50-95 |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| V3-E1 | YOLO26n | 86 | 0.755270 | 0.936510 | 0.836100 | 0.733280 | 100 | 0.753410 | 0.888890 | 0.830780 | 0.733210 |
| V3-E2 | YOLO26n | 70 | 0.813260 | 0.904760 | 0.861380 | 0.763890 | 100 | 0.773080 | 0.865310 | 0.813300 | 0.735520 |
| V3-E3 | YOLO26n | 77 | 0.779130 | 0.920630 | 0.851250 | 0.749120 | 100 | 0.782570 | 0.873020 | 0.831510 | 0.729790 |
| V3-E4 | YOLO26n | 75 | 0.833870 | 0.809520 | 0.857860 | 0.745850 | 100 | 0.745940 | 0.885500 | 0.821430 | 0.714940 |
| V3-E5 | YOLO26n | 76 | 0.732610 | 0.873020 | 0.845240 | 0.745680 | 100 | 0.762940 | 0.841270 | 0.809910 | 0.712230 |
| V3-E6 | YOLO26n | 94 | 0.761590 | 0.873020 | 0.794940 | 0.698530 | 100 | 0.751930 | 0.873020 | 0.787410 | 0.691070 |
| V3-E7 | YOLO26n | 90 | 0.809150 | 0.857140 | 0.830700 | 0.741890 | 100 | 0.798620 | 0.841270 | 0.831020 | 0.737530 |

### V3-E1 YOLO26n Operational Baseline Completed

V3-E1 was trained as the official YOLO26n operational baseline using 320 real positive training images and 80 lighter-based hard-negative training images. The fixed validation set contained 57 real positive images and 45 lighter-based hard-negative images. The fixed test set contained 80 real positive images and 63 lighter-based hard-negative images.

The model used `yolo26n.pt`, `imgsz=960`, batch size 16, seed 42 and the full 100-epoch schedule. Training completed in 11.05 minutes on an NVIDIA A100-SXM4-40GB GPU. Ultralytics removed one duplicate label from `positive_train__upload_image_20260603_195809_img_2331.jpg`; no corrupt train, validation or test images were reported.

The best checkpoint was selected at epoch 86 by validation mAP50-95. At this checkpoint, validation precision was 0.755270, validation recall was 0.936510, validation mAP50 was 0.836100 and validation mAP50-95 was 0.733280. The official fixed test metrics using `best.pt` were precision 0.694859, recall 0.775934, mAP50 0.775894 and mAP50-95 0.643398.

This establishes a stronger YOLO26n baseline than the earlier superseded YOLOv10n pilot baseline. The model detects approximately 77.6% of labelled test vape instances and has substantially improved precision compared with the pilot baseline, while still leaving room for source-dose additions to affect recall, precision, localisation quality and hard-negative false-positive behaviour. The validation-test gap remains important: validation recall and mAP50-95 are higher than the official held-out test metrics, so later source-dose improvements should be judged primarily against the fixed test set.

### V3-E2 YOLO26n Low-Dose Scraped Completed

V3-E2 added 88 curated scraped positive images to the same real positive and hard-negative training foundation used in V3-E1. The fixed validation and test sets were unchanged, so any metric difference reflects the training-source addition rather than an easier or harder evaluation set.

The best checkpoint was selected at epoch 70 by validation mAP50-95. Validation performance improved relative to V3-E1: precision increased from 0.755270 to 0.813260, mAP50 increased from 0.836100 to 0.861380 and mAP50-95 increased from 0.733280 to 0.763890. Validation recall decreased slightly from 0.936510 to 0.904760.

However, the official fixed test metrics did not show the same improvement. Test precision increased from 0.694859 to 0.755328, but test recall decreased from 0.775934 to 0.755556, mAP50 decreased from 0.775894 to 0.751819 and mAP50-95 decreased from 0.643398 to 0.601064. This indicates that the low-dose scraped addition improved confidence/precision behaviour but did not improve overall held-out target-domain detection quality. The larger validation-test gap in mAP50-95 suggests limited transfer from this scraped subset to the fixed real UAVape test set.

### V3-E3 YOLO26n Medium-Dose Scraped Completed

V3-E3 increased the curated scraped positive dose to 177 images while preserving the same fixed real validation and test sets. The best checkpoint was selected at epoch 77 by validation mAP50-95. Validation precision was 0.779130, validation recall was 0.920630, validation mAP50 was 0.851250 and validation mAP50-95 was 0.749120.

On the official fixed test set, V3-E3 achieved precision 0.779879, recall 0.708604, mAP50 0.791869 and mAP50-95 0.634267. Relative to V3-E1, this increased precision by 0.085020 and mAP50 by 0.015975, but reduced recall by 0.067330 and mAP50-95 by 0.009131. Relative to V3-E2, the medium scraped dose recovered much of the mAP50-95 drop and improved precision, but recall decreased further.

This suggests that medium-dose scraped data made the detector more selective and improved coarse IoU-0.50 detection quality, but did not improve the primary localisation-aware test metric over the real-data baseline. After E2-E3, the scraped pathway shows a precision-oriented tradeoff rather than a clear held-out mAP50-95 gain.

### V3-E4 YOLO26n Full-Dose Scraped Completed

V3-E4 increased the curated scraped positive dose to 355 images while preserving the same fixed real validation and test sets. The best checkpoint was selected at epoch 75 by validation mAP50-95. Validation precision was 0.833870, validation recall was 0.809520, validation mAP50 was 0.857860 and validation mAP50-95 was 0.745850.

On the official fixed test set, V3-E4 achieved precision 0.698620, recall 0.666667, mAP50 0.734826 and mAP50-95 0.593865. Relative to V3-E1, this changed precision by only +0.003761, but reduced recall by 0.109267, mAP50 by 0.041068 and mAP50-95 by 0.049533. Relative to V3-E3, the full scraped dose reduced all official test metrics.

The completed scraped pathway therefore does not support direct scraped-positive augmentation as an overall improvement strategy for the fixed real UAVape test domain. Low and medium scraped doses improved precision, with the precision peak occurring at V3-E3, but recall and strict localisation did not improve over the baseline. At the full scraped dose, the precision advantage largely disappeared and recall/mAP degraded further. The most defensible interpretation is that scraped imagery provides some target-class appearance signal, but its source-domain mismatch limits transfer to UAV-style localisation when mixed directly into the primary detector training pool.

### V3-E5 YOLO26n Low-Dose Synthetic Completed

V3-E5 added 88 pure synthetic positive images to the same real positive and hard-negative training foundation used in V3-E1. This is the matched-dose counterpart to V3-E2, allowing low-dose synthetic augmentation to be compared directly against low-dose scraped augmentation while keeping the fixed validation and test sets unchanged.

The best checkpoint was selected at epoch 76 by validation mAP50-95. Validation precision was 0.732610, validation recall was 0.873020, validation mAP50 was 0.845240 and validation mAP50-95 was 0.745680. This validation mAP50-95 was slightly higher than the V3-E1 baseline, suggesting that the selected checkpoint looked plausible under the fixed validation protocol.

On the official fixed test set, however, V3-E5 achieved precision 0.737137, recall 0.659259, mAP50 0.714588 and mAP50-95 0.572183. Relative to V3-E1, precision increased by 0.042278, but recall decreased by 0.116675, mAP50 decreased by 0.061306 and mAP50-95 decreased by 0.071215. Relative to the matched 88-image scraped condition V3-E2, V3-E5 was lower on precision, recall, mAP50 and mAP50-95.

The low-dose synthetic result therefore did not support synthetic augmentation as an immediate improvement over the real hard-negative baseline. Like the scraped additions, it showed a precision-oriented signal, but the recall and localisation-aware test losses were larger. The validation-test gap for mAP50-95 was also the largest observed at that point, indicating that the synthetic low-dose checkpoint did not transfer well from validation selection to the fixed held-out test set.

### V3-E6 YOLO26n Medium-Dose Synthetic Completed

V3-E6 increased the pure synthetic positive dose to 177 images while preserving the same fixed real validation and test sets. This is the matched-dose counterpart to V3-E3, allowing medium-dose synthetic augmentation to be compared against medium-dose scraped augmentation under the same detector, seed, image size and checkpoint-selection rule.

The best checkpoint was selected at epoch 94 by validation mAP50-95. Validation precision was 0.761590, validation recall was 0.873020, validation mAP50 was 0.794940 and validation mAP50-95 was 0.698530. Unlike V3-E5, the selected validation mAP50-95 was lower than the V3-E1 baseline, suggesting weaker validation-side evidence before official test evaluation.

On the official fixed test set, V3-E6 achieved precision 0.721116, recall 0.727841, mAP50 0.730766 and mAP50-95 0.603659. Relative to V3-E1, precision increased by 0.026257, but recall decreased by 0.048093, mAP50 decreased by 0.045128 and mAP50-95 decreased by 0.039739. Relative to V3-E5, V3-E6 recovered recall and mAP50-95, but still did not exceed the real hard-negative baseline. Relative to the matched 177-image scraped condition V3-E3, V3-E6 had higher recall, but lower precision, mAP50 and mAP50-95.

The medium synthetic dose therefore weakens the initial E5 concern but does not reverse the overall pattern. Synthetic augmentation shows some dose recovery between 88 and 177 images, especially in recall, but the official localisation-aware metric remains below the baseline.

### V3-E7 YOLO26n Full-Dose Synthetic Completed

V3-E7 increased the pure synthetic positive dose to 355 images while preserving the same fixed real validation and test sets. This is the matched-dose counterpart to V3-E4, allowing full-dose synthetic augmentation to be compared against full-dose scraped augmentation under the same detector, seed, image size and checkpoint-selection rule.

The best checkpoint was selected at epoch 90 by validation mAP50-95. Validation precision was 0.809150, validation recall was 0.857140, validation mAP50 was 0.830700 and validation mAP50-95 was 0.741890. This validation mAP50-95 slightly exceeded the V3-E1 validation baseline, suggesting that the full synthetic condition looked competitive under validation selection.

On the official fixed test set, V3-E7 achieved precision 0.737111, recall 0.718519, mAP50 0.725513 and mAP50-95 0.598678. Relative to V3-E1, precision increased by 0.042252, but recall decreased by 0.057415, mAP50 decreased by 0.050381 and mAP50-95 decreased by 0.044720. Relative to V3-E6, full-dose synthetic changed little on official test performance: precision increased by 0.015995, while recall, mAP50 and mAP50-95 decreased slightly. Relative to the matched full-dose scraped condition V3-E4, V3-E7 achieved higher precision, recall and mAP50-95, but lower mAP50.

The completed synthetic pathway therefore does not support direct pure-synthetic augmentation as an overall improvement strategy over the real hard-negative baseline in this V3 configuration. Unlike the scraped pathway, the synthetic pathway does not collapse further at full dose; instead, it partially recovers from the low-dose result and then plateaus below the baseline. This suggests that synthetic images contributed useful target-class appearance signal, but not enough target-domain localisation transfer to improve official held-out UAVape mAP50-95 when mixed directly into the primary detector training pool.

### Prediction-Level Diagnostic Pass Completed

After the E1-E7 training matrix was completed, prediction-level diagnostics were run across all official `best.pt` checkpoints using the fixed test set and a low confidence threshold (`conf=0.001`). This produced four diagnostic artifacts: `v3_yolo26n_prediction_level_table_full.csv`, `v3_yolo26n_image_level_eval_table_full.csv`, `v3_yolo26n_hard_negative_false_positive_summary.csv`, `v3_yolo26n_scale_band_performance.csv` and `v3_yolo26n_tp_fp_confidence_summary.csv`.

The hard-negative diagnostic table showed that V3-E6 produced the lowest low-threshold hard-negative false-positive count: 182 FP predictions across 63 hard-negative test images, or 2.888889 FP predictions per hard-negative image. V3-E5 produced the highest count at 282 FP predictions, followed by V3-E4 at 267 and V3-E7 at 251. Since this pass used `conf=0.001`, these values should be interpreted as relative sensitivity to hard-negative distractors rather than final operational false-positive rates.

The scale-band diagnostic table showed that all experiments remained strong on large and medium vape recall under the low-threshold diagnostic pass. The main variation occurred in small-vape recall and in the number of extra false-positive predictions around positive images. V3-E7 had the strongest low-threshold tiny-vape precision among the reported scale bands, but also reduced small-vape recall relative to V3-E1. This supports the broader V3 interpretation that external source additions mainly change selectivity and false-positive behaviour rather than delivering a uniform target-domain recall improvement.

The TP/FP confidence summary showed clear confidence separation overall: TP median confidence remained high across experiments, while FP median confidence stayed near zero. However, maximum hard-negative FP confidence remained high for every condition, including V3-E1, meaning that some hard-negative distractors can still receive confident vape predictions. This motivates threshold diagnostics, qualitative false-positive review and future hard-negative mining before deployment-style claims.

### Threshold And Source-Dose Analysis Completed

Final threshold diagnostics were generated after the prediction-level pass and saved as `v3_yolo26n_threshold_diagnostics.csv`. These diagnostics sweep confidence thresholds from 0.05 to 0.70 and report precision, recall, F1, false-positive predictions, false positives per hard-negative image and false negatives for each experiment.

The threshold sweep reinforces the main official-test conclusion. V3-E1 remains the most stable balanced baseline across common thresholds: at threshold 0.40 it achieved precision 0.724638, recall 0.740741 and F1 0.732601, while reducing hard-negative false positives to 0.349206 per hard-negative image. V3-E2 was similar at threshold 0.40, with precision 0.740458, recall 0.718519 and F1 0.729323, but did not improve recall over the baseline. V3-E3 achieved its strongest threshold-level F1 at threshold 0.50, with precision 0.757812, recall 0.718519 and F1 0.737643, which supports the interpretation that medium-dose scraped data can improve selectivity under threshold tuning, but not broad recall.

The synthetic pathway did not produce a stronger threshold-calibrated operating point than the baseline. V3-E5 carried high false-positive pressure at low thresholds. V3-E6 was cleaner on hard negatives and achieved F1 0.705426 at threshold 0.50, but still had lower recall than V3-E1 at comparable thresholds. V3-E7 improved precision as threshold increased, but its recall remained below the baseline and its best displayed F1 stayed below V3-E1's best displayed F1.

The source-dose effect table was saved as `v3_yolo26n_source_dose_effect.csv`. It confirms that no external-data condition improved mAP50-95 over V3-E1 on the official fixed test set. Scraped data produced the highest precision delta at the medium dose (V3-E3: +0.085020 precision), but did not improve mAP50-95. Synthetic data produced a partial recovery from V3-E5 to V3-E6, then plateaued at V3-E7. The final post-analysis workbook was saved as `v3_yolo26n_post_analysis_workbook.xlsx` and backed up to Google Drive.

## Consolidated V3 Insights

The completed V3 experiment shows that the strongest overall training recipe was not the largest dataset, but the most target-domain-aligned one. V3-E1, trained on real UAVape positives and real hard negatives, remained the best overall condition on the official fixed test set for recall and mAP50-95. This establishes the core data-centric result: for UAV vape-litter detection, domain alignment was more valuable than simply increasing the number of positive training examples.

The scraped pathway demonstrated a precision-oriented tradeoff. Low and medium scraped doses improved precision, with the highest official precision occurring at V3-E3. However, scraped data did not improve the primary strict localisation metric, mAP50-95, over the baseline. The full scraped dose then reduced recall and mAP further. This suggests that scraped vape imagery contributed useful target-appearance information, but its source-domain mismatch limited transfer to UAV-style localisation and recall.

The synthetic pathway demonstrated partial recovery rather than monotonic collapse. Low-dose synthetic data produced the weakest mAP50-95 result, but medium-dose synthetic recovered recall and mAP50-95 relative to low dose. Full-dose synthetic then plateaued around the medium-dose result rather than overtaking the baseline. This suggests that synthetic images provided some transferable signal, but not enough target-domain localisation fidelity to outperform real target-domain training data.

The matched-dose design was essential to these conclusions. Without the 88, 177 and 355 dose structure, the experiment would have produced only a binary result about whether external data helped or failed. The V3 source-dose design instead exposed different response patterns: scraped data produced a medium-dose precision peak followed by full-dose degradation, while synthetic data recovered from low dose and then plateaued.

The validation-test gap analysis showed that validation performance alone was not sufficient evidence of target-domain improvement. Several augmented conditions looked competitive or improved on validation, but their official fixed test metrics dropped. This supports the decision to treat the fixed held-out test set as the primary judge and validation metrics as checkpoint-selection evidence only.

The prediction-level and threshold diagnostics added two operational insights. First, most false positives had very low median confidence, meaning threshold calibration can reduce much of the low-confidence noise. Second, some hard-negative images still produced high-confidence false positives across conditions, including the baseline. This means the detector is not deployment-ready without further hard-negative mining, qualitative error review and threshold calibration on a separate validation protocol.

Overall, V3 supports a bounded claim about OOD and sim-to-real behaviour in this specific application. The thesis does not claim to discover domain shift or the sim-to-real gap. Instead, it shows how those known phenomena manifest in UAV vape-litter detection under a controlled DCE source-dose protocol. The Dataset Construction Engine is therefore justified not because every source improves performance, but because it makes external data auditable: it reveals whether scraped and synthetic sources help, hurt, or change detector behaviour before they are trusted in a deployment pipeline.

## Superseded YOLOv10n Pilot Record

Before the restart decision, E1-E3 were run using `yolov10n.pt`. These runs showed that the Colab pipeline, backup logic, validation/test reporting and source-dose interpretation worked. However, because the official V3 study now restarts with `yolo26n.pt`, those values should not be mixed into the official YOLO26n tables.

Dissertation-safe wording:

> Preliminary YOLOv10n pilot runs were used to validate the experimental pipeline. After confirming that YOLO26n was the current Ultralytics nano-scale detector, the official V3 matrix was restarted from V3-E1 using YOLO26n. The pilot metrics are therefore excluded from the final V3 source-dose comparison.

## Results Interpretation Template

After V3, each pathway should be interpreted using the same pattern:

1. Compare the pathway against V3-E1, the real + hard-negative operational baseline.
2. Compare low, medium and full dose within the same pathway.
3. Compare matched doses across pathways, for example 177 scraped vs 177 synthetic.
4. Check whether aggregate metric changes are explained by:
   - hard-negative false positives;
   - recall loss;
   - localisation loss;
   - tiny/small-object performance;
   - validation-test gap;
   - confidence behaviour.

This avoids relying only on aggregate mAP.

## Claim Boundaries

### Safe Claims

- UAVape provides a dataset construction workflow for target-domain real data, hard negatives, scraped data, synthetic data and evaluation metadata.
- Under a fixed YOLO26n baseline and fixed real UAVape test protocol, V3 measures how different data pathways affect detection.
- Matched source-dose experiments make scraped and synthetic comparisons fairer than broad unmatched source additions.
- Hard-negative FP analysis directly tests false-positive resistance against the controlled lighter-based hard-negative subset.

### Claims To Avoid

- The detector is fully deployable or completely reliable.
- Synthetic data is generally better or worse than scraped data outside this experiment.
- The DCE is proven domain-agnostic across all domains.
- YOLO26n is the best possible architecture.
- Results generalise beyond the tested UAVape target-domain distribution.
- Lighter-based hard negatives prove robustness against every possible non-vape litter class.

## Future Work

These are deliberately outside the main V3 matrix:

1. Run the best V3 data recipe with YOLO26s to test model-capacity sensitivity.
2. Add repeated seeds for V3-E1 and the best/most informative condition.
3. Expand hard negatives beyond lighters to pens, batteries, USB sticks, lip balm tubes, cables and wrappers.
4. Use failure modes to collect or generate targeted data.
5. Perform active-learning data selection.
6. Perform hard-negative mining.
7. Calibrate deployment thresholds using a separate validation protocol.
8. Test external sites, cameras, altitudes or collection days.
9. Evaluate tiling or higher image sizes if tiny-vape recall remains weak.

## Current Status

The official V3 YOLO26n matrix is complete:

- detector: `YOLO26n`;
- weights: `yolo26n.pt`;
- image size: `960`;
- completed experiments: V3-E1 to V3-E7;
- fixed validation/test sets;
- final metrics, diagnostics, workbook and figures saved under `/content/uavape_v3_yolo26n/reports`;
- Drive backup path: `/content/drive/MyDrive/uavape_ablation/outputs_v3_yolo26n`;
- old YOLOv10n outputs retained only as superseded pilot evidence.
