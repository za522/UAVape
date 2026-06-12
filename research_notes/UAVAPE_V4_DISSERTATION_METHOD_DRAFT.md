# UAVape V4 Dataset and Model-Selection Method Draft

## Section Role

The previous UAVape experiments showed that dataset construction choices can strongly affect downstream object-detection performance. In the V3 evaluation, the strongest official held-out result came from the real hard-negative baseline rather than from scraped or synthetic augmentation. This motivated a revised V4 method with two changes. First, the real UAVape dataset was expanded and re-audited at high resolution. Second, model selection was treated as a central methodological variable rather than an assumed fixed choice.

This section therefore describes the V4 dataset and training protocol used to develop a more defensible UAVape detector. The aim is not only to train a higher-scoring model, but to make the dataset split, class design, model family, input size and evaluation sequence explicit enough that the result can be interpreted without leakage or post-hoc test-set selection.

The central methodological question is:

> Which modern, edge-feasible YOLO-family detector and input resolution provides the strongest validation performance for detecting disposable vape litter in UAV-oriented imagery, while preserving a strictly held-out real test set for final evaluation?

This is a detector-protocol selection question. It differs from the earlier V3 data-source ablation because the V4 baseline already contains the re-audited real data, the expanded daylight collection and a two-class vape/lighter annotation scheme. Source additions such as benign negatives, scraped contextual images or synthetic images are treated as later data-centric branches, not as part of the initial model-selection baseline.

## Dataset Engineering Rationale

Disposable vape detection presents a small-object, high-confusion visual task. Vapes vary in colour, shape, branding, material finish, orientation and partial occlusion. In low-altitude UAV-oriented imagery, they occupy a small part of the frame and appear against grass, pavement, soil, curbs and other outdoor textures. They can also resemble lighters, marker pens, batteries, cables and other cylindrical litter.

The V4 dataset was therefore designed as a real-image, class-aware detector dataset rather than a simple positive-only collection. The primary class is `vape`. A second labelled class, `lighter`, is included as an auxiliary confuser class. This does not change the dissertation claim into a lighter-detection study. Instead, lighters are labelled because they are visually similar object-like litter items. Treating visible lighters as empty background would give the detector a confusing learning signal: the image contains a salient object, but the label file would imply that no object is present. Labelling lighters separately preserves their hard-negative role for vape discrimination while avoiding missing-annotation/background-labelling effects.

The V4 detector is therefore trained as:

```yaml
names:
  0: vape
  1: lighter
```

The headline claim remains vape detection. Lighter performance is reported as an auxiliary diagnostic showing whether the model learned a meaningful vape-versus-lighter boundary.

## Data Sources

V4 combines two real-image source pools:

| Source pool | Role |
|---|---|
| Audited old V3 real pool | Re-audited high-resolution version of the previous real UAVape imagery. Provides continuity with the earlier dataset and preserves already collected real scenes. |
| New V4 daylight expansion | Newly collected daylight real images. Provides additional scenes, objects, viewpoints and surfaces, and supplies the held-out test source. |

The old V3 real pool was originally created under an earlier annotation workflow. Before V4 training, it was audited in the UAVape Dataset Engine using high-resolution originals and a high-resolution COCO annotation layer. Existing bounding boxes and segmentations were migrated from the legacy low-resolution representation to the preserved high-resolution images, then manually reviewed and corrected where needed.

The new V4 daylight expansion was imported directly into the annotation workflow. Images were manually annotated using SAM2-assisted masks and bounding boxes, with class-aware labels for `vape` and `lighter`. Images that were not usable were rejected during audit.

The final V4 source composition was:

| Source pool | Images | Vape boxes | Lighter boxes | Total boxes |
|---|---:|---:|---:|---:|
| Audited old V3 real pool | 641 | 670 | 275 | 945 |
| New V4 daylight expansion | 563 | 658 | 437 | 1095 |
| **Combined V4 real pool** | **1204** | **1328** | **712** | **2040** |

The new V4 daylight batch originally contained 574 imported images. Eleven were rejected during audit, leaving 563 active images. All active new-batch images had accepted annotations before export.

## Source Resolution Characteristics

The V4 dataset deliberately preserves native high-resolution imagery. Images are not manually resized to one source resolution before training; instead, YOLO performs its standard resizing/letterboxing to the selected training `imgsz`. This avoids discarding source detail before the controlled resolution experiment.

The source resolution distribution was:

| Resolution | Images |
|---|---:|
| 4284 x 5712 | 437 |
| 1536 x 2048 | 308 |
| 4032 x 3024 | 156 |
| 3024 x 4032 | 122 |
| 5712 x 4284 | 105 |
| 2048 x 1536 | 54 |
| 1200 x 1600 | 22 |

The old V3 real pool and the new V4 daylight pool differ in dominant resolution. The old V3 pool is dominated by `1536 x 2048` images, while the new V4 pool is dominated by `4284 x 5712` images:

| Source | Dominant resolution | Count | Share |
|---|---:|---:|---:|
| Old V3 real | 1536 x 2048 | 308 / 641 | 48.0% |
| New V4 daylight | 4284 x 5712 | 437 / 563 | 77.6% |

This difference is recorded rather than hidden. It is not treated as a preprocessing error because the training protocol controls the actual network input size through `imgsz`. However, it is relevant when interpreting generalisation from train/validation to the held-out new V4 test set.

## Train-Validation-Test Split

The split was designed to reduce leakage caused by near-duplicate scene bursts. The images were not randomly mixed at individual-image level across train, validation and test. This matters because the collection protocol involved multiple photographs of related scenes, angles and object arrangements. Randomly splitting those near-duplicates would risk placing highly similar views in both training and test.

The final split policy was:

| Rule | Rationale |
|---|---|
| Test images are drawn only from the new V4 daylight batch. | Avoids mixing old V3 scene families into final held-out generalisation claims. |
| Train/validation use old V3 plus the non-test part of new V4. | Gives validation both legacy and new-domain evidence while preserving an untouched test subset. |
| New V4 split assignment uses sequential capture blocks rather than individual images. | Reduces near-duplicate leakage where explicit station/scene IDs were not recorded. |
| Split selection is class-aware. | Ensures enough vape and lighter instances are present in validation and test for two-class reporting. |

The final split was:

| Split | Images | Vape boxes | Lighter boxes | Old V3 images | New V4 images | Groups |
|---|---:|---:|---:|---:|---:|---:|
| Train | 837 | 870 | 447 | 525 | 312 | 87 |
| Validation | 183 | 203 | 100 | 116 | 67 | 10 |
| Test | 184 | 255 | 165 | 0 | 184 | 19 |

The test split is therefore not a random sample from the full dataset. It is a held-out subset from the newer collection batch. This better matches the intended claim: whether the selected detector protocol generalises to unseen real UAVape scenes rather than merely recognising near-duplicates from the same collection sequences.

## YOLO Export and Verification

The frozen dataset was exported in YOLO detection format with paired image and label folders for each split. The exported dataset used:

```yaml
path: /path/to/v4_real_final_20260610
train: images/train
val: images/val
test: images/test
nc: 2
names:
  0: vape
  1: lighter
```

Before training, the exported dataset was verified both locally and after transfer to the training environment. The RunPod training environment confirmed:

| Split | Images | Labels | Boxes | Vape boxes | Lighter boxes | Missing labels | Missing images |
|---|---:|---:|---:|---:|---:|---:|---:|
| Train | 837 | 837 | 1317 | 870 | 447 | 0 | 0 |
| Validation | 183 | 183 | 303 | 203 | 100 | 0 | 0 |
| Test | 184 | 184 | 420 | 255 | 165 | 0 | 0 |

This verification step is included because the dataset archive was moved between local storage, Google Drive/Colab and RunPod. The check confirmed that the train/validation/test structure, labels and class counts survived transfer correctly.

During Ultralytics validation, one duplicate label was automatically removed from `upload_image_20260603_195809_img_2331.jpg` in the validation split. Because the same framework behaviour applies consistently across all runs, this does not invalidate the relative benchmark. However, the duplicate should be removed from the source label file before final archival if possible.

## Model-Selection Rationale

The V4 method treats model selection as a controlled benchmark rather than assuming a nano-scale detector. The earlier V3 experiments used YOLO26n, which was appropriate for a lightweight data-source ablation but may have been capacity-limited for the expanded V4 dataset. V4 therefore compares modern YOLO-family detectors while keeping the dataset split, training schedule and validation procedure fixed.

The model families are chosen for different methodological roles:

| Model family | Role |
|---|---|
| YOLO26 | Primary current Ultralytics candidate. It introduces NMS-free inference, a lighter detection head and updated training recipe. |
| YOLO11 | Stable modern comparator with wider benchmark familiarity. |
| YOLOv8 | Literature-anchor baseline because many recent UAV and litter studies still use YOLOv8 variants. |

The benchmark should be understood as one detector-selection experiment rather than separate accuracy and deployment studies. The aim is to map the practical design space for UAVape: whether performance is limited more by model family, model capacity or input resolution. Small models form the centre of the benchmark because they are the most realistic balance between accuracy and deployability. Medium models at 640 test whether extra capacity at standard resolution can outperform higher-resolution small models. Nano models at 640 and 960 test the lower deployability boundary. Large (`l`) and extra-large (`x`) models are excluded from the main benchmark because they are outside the intended UAV/mobile feasibility scope.

The practical model scale interpretation is:

| Scale | Methodological role |
|---|---|
| Nano (`n`) | Lower deployability boundary inside the unified benchmark. |
| Small (`s`) | Benchmark centre and main feasible SOTA detector class. |
| Medium (`m`) | Capacity check at standard input size. |
| Large (`l`) / extra-large (`x`) | Excluded from the main protocol as outside the edge-feasible scope. |

## Input-Resolution Rationale

Input resolution is treated as a model-protocol variable because UAVape contains small objects in high-resolution images. Increasing `imgsz` can preserve more small-object detail, but it also increases compute quadratically and may reduce deployment feasibility.

The core feasible resolutions are:

| imgsz | Role |
|---:|---|
| 640 | Standard YOLO benchmark resolution and most deployment-feasible setting. |
| 960 | Intermediate high-detail setting; a controlled compromise between 640 efficiency and 1280 small-object preservation. |
| 1280 | Upper-resolution small-object check; useful for understanding accuracy ceiling but less realistic for phone/UAV real-time deployment. |

The revised V4 interpretation is that 960 should not be described as a universal literature standard. Instead, it is a controlled intermediate resolution selected because the source images are high-resolution and the targets are small. The 1280 result is useful as an upper-resolution check, but the main feasible SOTA comparison should focus on 640 and 960.

## Training Protocol

Training was conducted using Ultralytics `8.4.62` on a RunPod H100 environment. Each candidate was trained for 100 epochs using the same split, seed and training recipe. The validation set was used for checkpoint selection and model comparison. The test set was not used during this stage.

The core training settings were:

| Parameter | Value |
|---|---|
| Framework | Ultralytics `8.4.62` |
| Task | Object detection |
| Classes | `vape`, `lighter` |
| Epochs | 100 |
| Seed | 42 |
| Scheduler | cosine learning rate enabled |
| Checkpoint used for comparison | `best.pt` selected by validation fitness / mAP50-95-dominated criterion |
| Official test usage | Held until final model/protocol selection |

The unified benchmark sequence is:

| Run group | Purpose |
|---|---|
| Small-model resolution sweep | YOLO26s, YOLO11s and YOLOv8s at 640, 960 and 1280. Tests family differences and small-object resolution response. |
| Medium capacity check | YOLO26m, YOLO11m and YOLOv8m at 640. Tests whether capacity at standard input size can beat high-resolution small models. |
| Nano lower-bound check | YOLO26n and YOLO11n at 640 and 960. Tests whether the protocol can be compressed toward phone/UAV-edge feasibility. |

This keeps the study as one detector-selection benchmark while still spanning accuracy and deployment constraints. The final protocol is selected from validation evidence before the held-out test split is used.

## Metric Hierarchy

Because the detector is trained with two classes but the dissertation claim is vape-focused, aggregate two-class mAP is not the only metric of interest. Overall mAP averages vape and lighter performance:

```text
mAP = mean(AP_vape, AP_lighter)
```

The metric hierarchy is therefore:

| Priority | Metric | Role |
|---:|---|---|
| 1 | Vape AP50 | Headline metric for UAV/litter literature comparability and practical detection. |
| 2 | Vape AP50-95 | Secondary localisation-quality metric retained for COCO-style rigour. |
| 3 | Vape precision and recall | Operational false-positive and missed-target behaviour. |
| 4 | Lighter AP/P/R | Auxiliary diagnostic showing whether the confuser class was learned. |
| 5 | Aggregate two-class mAP | Supporting context only. |

Ultralytics training logs report aggregate validation metrics during training. Therefore, after each completed run, per-class validation metrics should be extracted from the saved `best.pt` checkpoint. This produces the vape-specific AP values needed for the main dissertation comparison without touching the held-out test set. The standard Ultralytics `best.pt` checkpoint rule is retained for reproducibility, even though the dissertation headline reports vape AP50; in the current benchmark, the leading protocol is the same under both vape AP50 and vape AP50-95.

## Test-Set Rule

The test split is reserved for the final selected detector protocol. It must not be used to choose:

- model family,
- model scale,
- input size,
- epoch/checkpoint,
- confidence threshold,
- data-source branch,
- hyperparameters.

Once the model family, scale, input resolution and any data-centric protocol choices are locked using validation results, the selected `best.pt` checkpoint is evaluated once on the held-out test split. If a deployment nano model is later evaluated, it should be reported as a secondary deployment result rather than as the primary SOTA detector unless it is explicitly selected as the final model.

## Optional Later Branches

Several possible branches are deliberately held back until the baseline model-selection stage is complete.

| Branch | Timing | Reason |
|---|---|---|
| UAVVaste benign negatives | After baseline error analysis | Tests false positives on no-target outdoor UAV backgrounds without changing the frozen baseline prematurely. |
| Contextual scraped images | Completed after baseline detector selection | Tests whether domain-relevant external vape imagery improves the fixed V4 real-data baseline. |
| Synthetic images | Completed after baseline detector selection | Tests whether synthetic vape additions improve the fixed V4 real-data baseline when model family, scale and resolution are held constant. |
| Hyperparameter tuning | After model/input protocol selection | Avoids multiplying the search space before the detector protocol is known. |
| Tiling / SAHI-style inference | After the unified detector benchmark and before final test if 1280 remains strongest | Completed as a validation-only inference branch; static 1280-slice SAHI inference did not improve over full-frame high-resolution inference. |
| Custom architecture | Last/future work | Architecture changes are detector-engineering interventions after standard model families and inference protocols are understood. |

This ordering keeps the method defensible. The first question is whether the fixed V4 real dataset supports a strong, feasible modern detector. Because high full-frame resolution remained the main driver of performance, tiled inference was tested as a targeted inference-protocol branch. The completed tiling results did not improve the validation metrics, so the branch is retained as a negative validation finding rather than selected for the final protocol. Data-source ablations were then run against the selected YOLO26s @ 1280 validation baseline to test whether additional source-controlled vape data could improve the fixed detector protocol.

## Completed Data-Source Ablation

After selecting YOLO26s @ 1280 as the strongest validation detector protocol, a controlled V4 data-source ablation was run with the model, input size, training schedule and validation split held constant. The ablation changed only the training data composition by adding source-controlled vape-only imagery to the V4 real training set.

The strongest data-source result was the combined scraped-plus-synthetic recipe:

| Run | Training recipe | All AP50 | All AP50-95 | Vape AP50 | Vape AP50-95 | Lighter AP50 | Lighter AP50-95 |
|---|---|---:|---:|---:|---:|---:|---:|
| `A0_real_only` | V4 real train only | 0.802 | 0.605 | 0.918 | 0.707 | 0.688 | 0.498 |
| `S355_Y355_combined` | V4 real + 355 scraped + 355 synthetic vape images | 0.815 | 0.639 | 0.922 | 0.737 | 0.708 | 0.541 |

This is a positive but source-specific result. Scraped-only and synthetic-only additions produced mixed, non-monotonic validation behaviour, while the combined source recipe improved aggregate AP50, aggregate AP50-95, vape AP50, vape AP50-95 and lighter AP50-95 relative to the real-only baseline. The result therefore supports a data-centric conclusion: the benefit came from identifying a useful mixture of complementary data sources, not from simply adding more images from any single source.

The full ablation table and captured RunPod/Jupyter evidence are preserved in `UAVAPE_V4_DATA_ABLATION_RESULTS.md`. These remain validation results; the selected final recipe should still be evaluated once on the sealed held-out test split before making a final generalisation claim.

## Current Execution Status

The unified benchmark has completed the small-model resolution sweeps, the medium-at-640 capacity checks, and the nano lower-bound runs at 640 and 960. The nano experiments are retained as part of the same detector-selection benchmark because they test whether the final detector can be compressed toward phone/UAV-edge feasibility without changing the dataset split or training protocol.

Nano lower-bound results:

| Run | Best epoch | Precision | Recall | mAP50 | mAP50-95 | Vape AP50 | Vape AP50-95 | Lighter AP50 | Lighter AP50-95 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| YOLO26n @ 640 | 74 | 0.747 | 0.643 | 0.687 | 0.459 | 0.829 | 0.598 | 0.545 | 0.320 |
| YOLO26n @ 960 | 78 | 0.773 | 0.684 | 0.729 | 0.527 | 0.835 | 0.633 | 0.623 | 0.422 |
| YOLO11n @ 640 | 89 | 0.699 | 0.681 | 0.688 | 0.456 | 0.853 | 0.587 | 0.522 | 0.325 |
| YOLO11n @ 960 | 90 | 0.843 | 0.671 | 0.765 | 0.545 | 0.878 | 0.645 | 0.653 | 0.445 |

These results show that nano capacity alone is not sufficient at 640, but increasing the input size to 960 substantially improves the nano protocols. YOLO11n @ 960 is the strongest nano result and approaches YOLO26s @ 640 on vape AP50-95. This supports the interpretation that the detection problem is constrained by small-object detail as well as model capacity.

Aggregate validation ranking from completed runs:

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

Vape-class validation ranking from completed runs:

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

SAHI-style tiled inference was then evaluated using fixed 1280-pixel slices with 20% overlap on selected full-frame checkpoints. The tiled branch did not improve validation performance. YOLO26s @ 960 with tiling reached vape AP50 0.80554 and vape AP50-95 0.60559, below its full-frame validation result of vape AP50 0.85692 and vape AP50-95 0.68355. YOLO11s @ 960 and YOLO11n @ 960 showed the same pattern, with tiled vape AP50 values of 0.76595 and 0.75871 respectively. The tiled runs also produced high prediction counts relative to the 302 validation annotations, suggesting false-positive and duplicate pressure from static slicing.

Current interpretation: YOLO26s @ 1280 remains the strongest completed validation protocol under both the headline vape AP50 metric and the secondary vape AP50-95 metric. This result is not driven by the auxiliary lighter class. The nano results show that a very small detector can become viable when paired with 960-pixel input, but they do not overturn the main benchmark. Overall, the benchmark suggests that input resolution and small-object preservation matter more than simply increasing model capacity at 640. The completed tiling branch supports selecting full-frame high-resolution inference for the final V4 protocol unless a later section explicitly labels further tiling work as exploratory.

This does not make an on-device deployment claim. The dissertation should frame YOLO26s @ 1280 as the strongest validation detector and upper-performance protocol for the curated UAVape dataset, not as a finished drone or phone implementation. Future UAV or iPhone deployment would require a separate optimisation stage, including export to an appropriate runtime, latency and energy measurement, quantisation or pruning if needed, and operating-threshold calibration. The nano-scale results, especially YOLO11n @ 960, provide evidence that lighter deployment-oriented variants are plausible future candidates, while the primary dissertation result remains the controlled full-frame detection protocol.
