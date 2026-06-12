# UAVape V4 Literature Comparison and Discussion Anchor

Date anchored: 2026-06-10

This file anchors the current V4 comparison argument so the dissertation discussion does not drift. It combines the current UAVape V4 validation results, the sealed test result, the completed detector and data ablations, and the litter-review paper's comparison table and method notes.

## Evidence Status

Primary local evidence used here:

- `UAVAPE_V4_DATA_ABLATION_RESULTS.md`
- `UAVAPE_RUNPOD_RECOVERY_20260610.md`
- `UAVAPE_CURRENT_DATASET_AUDIT.md`
- `UAVAPE_V4_DISSERTATION_METHOD_DRAFT.md`
- `files-mentioned-by-the-user-sauce/uavape_v4_dataset_model_selection_method_and_findings.md`
- `files-mentioned-by-the-user-sauce/discussion_reworked_formal_draft.md`
- `Litter Review Paper.pdf`, extracted locally as `2026-05-25/how-can-i-install-codex-into/pdf_extracts/review.txt`

Confidence labels used below:

- Confirmed: directly supported by the current local notes or extracted review text.
- Review-limited: supported by the review paper, but the original paper should be checked before making a precise final claim.
- Needs lookup: not fully recoverable from the review text; check the original paper before dissertation submission.

## Current UAVape V4 Position

UAVape V4 is not just a single model result. The method now has three linked experimental stages:

1. Dataset construction and audit through the Dataset Construction Engine.
2. Detector protocol selection through model-family, capacity and input-size validation ablation.
3. Data-source ablation under a frozen detector protocol.

The selected validation protocol before data-source ablation was:

| Component | Selected value |
|---|---|
| Detector | YOLO26s |
| Input size | 1280 |
| Inference | Full-frame, no SAHI tiling |
| Training length | 100 epochs |
| Seed | 42 |
| Selection split | Fixed V4 validation split |
| Classes | `vape`, `lighter` |

The final data-source ablation winner on validation was:

| Recipe | All AP50 | All AP50-95 | Vape AP50 | Vape AP50-95 | Lighter AP50 | Lighter AP50-95 |
|---|---:|---:|---:|---:|---:|---:|
| Real-only baseline, YOLO26s@1280 | 0.802 | 0.605 | 0.918 | 0.707 | 0.688 | 0.498 |
| Real + 355 scraped + 355 synthetic | 0.815 | 0.639 | 0.922 | 0.737 | 0.708 | 0.541 |

Important interpretation:

- The "0.9" result is the vape-class AP50, not the aggregate two-class mAP.
- The conservative aggregate AP50 is 0.815.
- The stricter COCO-style AP50-95 is 0.737 for vape and 0.639 aggregate.
- These validation values were used for protocol/data-recipe selection, not final generalisation reporting.

The selected checkpoint was then evaluated once on the sealed real-only test split:

| Split | Images | Instances | All P | All R | All AP50 | All AP50-95 | Vape AP50 | Vape AP50-95 | Lighter AP50 | Lighter AP50-95 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Test | 184 | 420 | 0.849 | 0.725 | 0.851 | 0.638 | 0.882 | 0.685 | 0.819 | 0.591 |

Use the test values for final generalisation claims. Use the validation values to explain why the model/data recipe was selected.

## What Is Methodologically Novel About UAVape V4

The dissertation discussion should not frame V4 as "we trained YOLO and got a number." The stronger claim is that the result came from a controlled construction and selection pipeline.

### 1. Target specificity

Most reviewed datasets target generic litter, bottles, macro-litter, recycling waste, or broad waste classes. UAVape targets disposable vapes as hazardous micro-WEEE, with lighters annotated as a second class because they are visually similar confusers.

This matters because the headline research claim is not generic litter detection. It is detecting a small, visually ambiguous, environmentally hazardous item that existing datasets do not cover well.

### 2. Dataset Construction Engine as a source-control system for data

The DCE allowed images to be separated by source and role:

- real UAV-oriented / low-altitude proxy imagery,
- scraped appearance-expansion imagery,
- synthetic composites,
- hard-negative confuser imagery,
- train/validation/test split membership,
- annotation and metadata audit state.

This enabled source-dose ablations instead of uncontrolled "more data" training.

### 3. Split discipline

The V4 test split is sealed and real-only. Model family, image size, tiling, checkpoint and data-source recipe were all selected using validation only.

This is a strong discussion point because many literature comparisons are weakened by unclear or inconsistent evaluation protocols. UAVape can be framed as deliberately separating:

- validation for method selection,
- test for final generalisation,
- scraped/synthetic images for training only,
- real unseen scenes for final evaluation.

### 4. Model and input-size ablation before data ablation

The V4 data ablation was not done on an arbitrary detector. First, the project benchmarked:

- YOLO26s, YOLO11s and YOLOv8s at 640, 960 and 1280;
- YOLO26m, YOLO11m and YOLOv8m at 640;
- YOLO26n and YOLO11n at 640 and 960.

This showed that YOLO26s@1280 was the strongest validation protocol for vape detection. The result supports a small-object interpretation: preserving spatial detail at 1280 mattered more than simply moving to a medium model at 640.

### 5. Tiling was tested and rejected under the V4 validation protocol

SAHI-style tiled inference was tested with 1280-pixel slices and 20% overlap using existing full-frame checkpoints. It underperformed the full-frame high-resolution candidates and produced many predictions relative to the 302 validation annotations.

This is valuable in the discussion because SODA and other UAV litter studies use tiling to preserve detail. UAVape did not ignore tiling; it tested a tiled branch and found that, for this dataset and detector protocol, full-frame 1280 inference preserved useful context and avoided the false-positive/duplicate pressure introduced by static slicing.

### 6. Data-source ablation after protocol selection

After locking YOLO26s@1280, the data ablation changed only the training source recipe:

- real-only baseline,
- scraped-only additions at 88, 177 and 355 images,
- synthetic-only additions at 88, 177 and 355 images,
- combined 355 scraped + 355 synthetic.

The result was positive but not simplistic. Individual scraped or synthetic additions did not monotonically improve the headline vape AP50. The combined source recipe produced the best validation result, suggesting that complementary source diversity mattered more than simply adding more images from one source.

## Literature Metric Comparison

Use this table as discussion scaffolding, not as a direct leaderboard. The reviewed works differ in dataset, class definitions, annotation type, image source, input size, split protocol and whether the reported metric is validation, test or cross-dataset. Some review-table values were reported as percentages and are normalized here to 0-1 notation where useful.

| Dataset / study family | UAV? | Reported strong metric from review | Main method nuance | Comparison to UAVape V4 |
|---|---:|---|---|---|
| BDW | Yes | Faster R-CNN AP 0.903; SSD AP 0.901; YOLOv2 AP 0.704; RRPN AP@0.5 around 0.886 in review discussion | Plastic bottles only; high-resolution DJI Phantom imagery; tiled to 324x324; Oriented Bounding Boxes; split reporting is inconsistent in the review, with both 64/16/20 and 80/20 summaries appearing | High AP, but bottle-only, OBB/tiled and older detector context. Not directly comparable to vape AP50 without resolving AP definition and split from original paper |
| UAVVaste | Yes | YOLOv4 mAP@50 0.785; YOLOv3 0.784; EfficientDet-d3 0.751; SSD MobileNetv2 0.545 | One-class rubbish; small objects often under 1% of image area; COCO transfer learning; adapted anchors; georeferencing from UAV altitude/FOV/GPS; older YOLOv3/v4 and EfficientDet models | UAVape test aggregate AP50 0.851 and test vape AP50 0.882 are above these review values, but UAVape is class-specific and not a direct dataset/protocol match |
| HAIDA | Yes | YOLOv4-Tiny-3l AP50 >70% | Real-time UAV marine trash system; low-altitude 1-10m; garbage/bottle classes; georeferencing; review says training/evaluation details are limited; quantization variants were tested for hardware | UAVape test metrics are higher, but HAIDA is real-time/hardware-oriented and protocol details need original-paper confirmation |
| SODA | Yes | YOLOv8 mAP 0.796; YOLOv5 mAP 0.781; Faster R-CNN ResNeXt-101 mAP 0.468 in Table 5; review prose also reports altitude-specific mAP@0.5 examples | Six litter classes; UAV altitudes 1-30m; polygon annotations; 5x5 tiling adjusted to 640x640 for YOLOv5/YOLOv8; compared training separate altitude models vs one all-altitude model; all-altitude training helped high-altitude performance | This is one of the strongest and most relevant UAV comparisons. UAVape's test aggregate AP50 0.851 and test vape AP50 0.882 are numerically strong relative to SODA's reviewed values, but the metric column and multi-class/altitude protocol must be stated carefully |
| TrashNet | No | YOLOv5 mAP@50 0.950 | Originally image classification; controlled indoor images; one easily visible item on white poster board; six broad material classes | Higher than UAVape vape AP50, but not a fair UAV or small-clutter comparison. Useful as evidence that controlled datasets can produce very high scores |
| TACO | No | YOLOv5x AP50 0.633; YOLOv5s AP50 0.547; Faster R-CNN AP50 0.511; EfficientDet-d5 AP50 0.423 | Ground-level natural images; 60 categories; polygon masks; supports single-class and multi-class setups; introduced Transplanter for overlaying segmented litter onto new backgrounds | UAVape test AP50 is higher, but TACO is broader, more heterogeneous and category-rich. Conceptually important because it supports the use of source-controlled synthetic/background transplantation |
| PlastOPol | No | YOLOv5x AP50 0.849; YOLOv5s AP50 0.799; Faster R-CNN AP50 0.753; EfficientDet-d5 AP50 0.732 | Outdoor natural/crowded backgrounds; Marine Debris Tracker source; one litter class; mostly large objects in the review's size distribution; cross-dataset evaluation with TACO is discussed | UAVape test aggregate AP50 0.851 is approximately level with PlastOPol YOLOv5x and above YOLOv5s; UAVape test vape AP50 0.882 is higher. Comparison must note object-size and non-UAV differences |
| MJU-Waste | No | Segmentation metrics such as mIoU/IoU/precision, not directly detector AP | Controlled indoor RGBD dataset; waste held by people; polygon masks; raw/processed depth maps; 60/10/30 train/validation/test split | Not directly comparable to UAVape detector AP. Useful mainly as evidence of indoor controlled waste segmentation and explicit validation/test splitting |
| ZeroWaste | No | Detection/segmentation entries in review include low AP50 values for Mask R-CNN/RetinaNet and mIoU for DeepLabv3 | Industrial conveyor-belt recycling facility; 1920x1080 video-derived frames; optical correction/cropping/motion-blur reduction; 65/15/20 train/validation/test split | Not directly comparable to outdoor UAVape; useful for discussing domain-specific waste detection and industrial vs environmental deployment contexts |

## Validation vs Test Status

This is the most important caveat for the discussion.

| Work | What we can currently say | Status |
|---|---|---|
| UAVape V4 | Validation metrics were used for protocol/data selection. The selected checkpoint was then evaluated on the sealed real-only test split: all AP50 0.851, all AP50-95 0.638, vape AP50 0.882, vape AP50-95 0.685 | Confirmed |
| BDW | Review states a 64/16/20 train/validation/test dataset split in the dataset section, but later summarizes an 80/20 train-test model evaluation | Needs original-paper lookup before exact split wording |
| UAVVaste | Review reports the models, COCO transfer learning, anchor adaptation and mAP values, but the extracted text does not fully lock down whether Table 5 values are validation or test | Needs original-paper lookup |
| HAIDA | Review explicitly says additional training/evaluation details are not provided | Needs original-paper lookup; use cautiously |
| SODA | Review describes model comparison, altitude-specific evaluation and test-altitude logic; exact split/reporting protocol should be checked in original SODA paper | Review-limited |
| TrashNet YOLOv5 result | Review table reports 0.95 mAP@50, but TrashNet is originally classification; the exact detection adaptation/evaluation protocol is not detailed in the extracted review text | Needs original-paper lookup |
| TACO / PlastOPol | Review table notes TACO values retrieved from PlastOPol; review says PlastOPol/MJU-Waste validate using TACO and own test sets | Review-limited; original paper needed for which rows are own-test vs cross-dataset |
| MJU-Waste | 60/10/30 split is stated; metrics are segmentation-focused rather than YOLO detector AP | Confirmed for split, not directly comparable |
| ZeroWaste | 65/15/20 split is stated; task/domain differ substantially from UAVape | Confirmed for split, not directly comparable |

## How To Frame "Is Mine Better?"

The honest answer is: UAVape V4 is now in a much stronger position than V3, and the sealed test result supports a strong comparison claim. The dissertation should still avoid an unconditional leaderboard claim until the original papers' reporting protocols are checked.

Best careful framing:

> After validation-based selection, the final UAVape V4 checkpoint achieved a held-out test vape AP50 of 0.882 and vape AP50-95 of 0.685, with aggregate two-class AP50 of 0.851 and AP50-95 of 0.638. These values are competitive with, and in several cases numerically above, reviewed UAV litter detector results reported for UAVVaste, HAIDA and SODA. However, this is not a direct leaderboard comparison: the reviewed studies differ in target classes, annotation formats, image sources, input sizes, tiling strategies and evaluation protocols. The appropriate claim is that UAVape V4 provides strong held-out evidence for vape-specific micro-WEEE detection under a controlled dataset-construction and ablation pipeline.

Avoid saying:

- "UAVape beats the literature."
- "This is state of the art."
- "The model is deployment-ready."
- "TrashNet proves our model is worse/better."

Safer wording:

- "competitive with reviewed UAV litter detectors"
- "above several reported AP50/mAP@50 values under a held-out test comparison, subject to protocol differences"
- "not directly comparable because of task and protocol differences"
- "strong evidence that the DCE-enabled source mixture improved the selected detector"
- "requires careful qualification against different datasets, targets and reporting protocols"

## Discussion Points By Theme

### Architecture and recency

Many reviewed detector results use older or legacy detector families:

- BDW: YOLOv2, Faster R-CNN, SSD, RRPN.
- UAVVaste: YOLOv3, YOLOv4, EfficientDet, SSD MobileNetv2.
- HAIDA: YOLOv4-Tiny-3l.
- PlastOPol and TACO: YOLOv5 variants plus Faster R-CNN/RetinaNet/EfficientDet.
- SODA: YOLOv5 and YOLOv8.

UAVape V4 benefits from a newer YOLO26-family detector and a structured comparison against YOLO11 and YOLOv8. That is a fair discussion point, but it should be framed as a methodological difference rather than proof of superiority.

Suggested wording:

> Part of the V4 gain may reflect detector recency rather than dataset construction alone. Unlike several reviewed studies that used YOLOv2, YOLOv3, YOLOv4 or YOLOv5-era detectors, UAVape compared YOLO26, YOLO11 and YOLOv8 variants under a shared validation protocol. This makes the final result a combined product of dataset construction, modern architecture selection and high-resolution input selection.

### Input size

Input resolution is central to the V4 result. YOLO26s improved materially at 1280 compared with 640/960, and nano results also improved when moving from 640 to 960.

Suggested wording:

> The V4 benchmark indicates that small-object detail was a limiting factor. Increasing input size to 1280 produced stronger validation performance than increasing model capacity at 640. This mirrors the wider UAV litter literature's concern with downscaling and tiling, but UAVape's own tiling experiment showed that, for this dataset, full-frame high-resolution inference outperformed the tested static slicing protocol.

### Tiling

SODA used 5x5 tiling to 640x640. BDW tiled high-resolution UAV images to 324x324. Other work tiles 4K imagery into small patches for density maps or small-object processing.

UAVape tested SAHI-style tiled inference but did not select it.

Suggested wording:

> The negative tiling result is itself informative. It suggests that preserving local object detail is necessary but not sufficient; the detector also benefited from full-frame scene context. Static slicing increased the number of candidate predictions and may have increased duplicate and false-positive pressure. Therefore, the selected V4 protocol used high-resolution full-frame inference rather than tiled inference.

### Data-source construction

The data ablation is the strongest DCE discussion point.

Suggested wording:

> The data-source ablation supports the value of the Dataset Construction Engine because it shows that source labels were not merely administrative metadata. They enabled a controlled experiment in which real, scraped and synthetic imagery could be combined or isolated under a fixed detector protocol. The best validation result came from the combined scraped-plus-synthetic recipe, suggesting that appearance diversity and environmental variation were complementary when added to the real training base.

### Why aggregate AP and vape AP both matter

UAVape is a two-class detector, but the dissertation is vape-focused.

Suggested wording:

> Aggregate two-class AP is reported for transparency, but it is not the primary research metric because it averages the target class with an auxiliary confuser class. Vape AP50 is the headline metric because the research question concerns vape detection. Lighter AP is interpreted as a hard-negative boundary diagnostic rather than as a second deployment objective.

## Remaining Holes Before Final Dissertation Wording

These are the gaps to close before making the final literature-comparison paragraph surgical:

1. Download or archive the final test output directory if needed for local evidence.
2. Check the original BDW paper to resolve 64/16/20 vs 80/20 reporting.
3. Check original UAVVaste to confirm split/evaluation protocol and exact AP table context.
4. Check original HAIDA for evaluation protocol, quantization variants and whether AP50 was validation/test.
5. Check original SODA for split, validation/test wording and whether the Table 5 mAP values are test-set or cross-altitude summaries.
6. Check original PlastOPol/TACO source for which AP50 values are own-test and which are cross-dataset.
7. Decide final comparison language: target-class AP50 comparison, aggregate AP50 comparison, or both.

## Bottom-Line Dissertation Claim As Of Now

UAVape V4 is genuinely good news, but the academically defensible claim is:

> UAVape V4 shows strong held-out evidence that a vape-specific micro-WEEE detector can be competitive with reviewed UAV litter detection systems when dataset construction, model selection, input resolution and data-source composition are treated as explicit experimental variables. The selected model reaches 0.882 vape AP50 and 0.685 vape AP50-95 on the sealed test split, with aggregate two-class AP50 of 0.851 and AP50-95 of 0.638. These values are promising relative to many reviewed UAV litter results, while still requiring careful qualification against differences in task, metric and protocol.
