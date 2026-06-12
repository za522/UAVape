# Machine Learning Evaluation: Dissertation-Ready Structure

> Internal working note for future runs, not dissertation text: Colab `/content/...` storage is temporary. After every training/evaluation run, immediately back up `runs`, `reports` and `figures` to Google Drive, for example `/content/drive/MyDrive/uavape_ablation/outputs_v2`. The V2 printed metric tables survived in the chat/notebook output, but the model artifacts for E1-E3 were lost after a runtime reset because `/content/uavape_full_ablation_v2/runs` was not copied to Drive. Future training cells should end with an explicit backup block:
>
> ```python
> import shutil
> from pathlib import Path
>
> DRIVE_OUTPUTS = Path("/content/drive/MyDrive/uavape_ablation/outputs_v2")
> DRIVE_OUTPUTS.mkdir(parents=True, exist_ok=True)
>
> shutil.copytree(RUNS, DRIVE_OUTPUTS / "runs", dirs_exist_ok=True)
> shutil.copytree(REPORTS, DRIVE_OUTPUTS / "reports", dirs_exist_ok=True)
> shutil.copytree(FIGS, DRIVE_OUTPUTS / "figures", dirs_exist_ok=True)
>
> print("Backed up runs, reports, and figures to:", DRIVE_OUTPUTS)
> ```
>
> For long YOLO runs, also consider copying each individual run folder immediately after that model finishes, before starting the next model. Do not wait until all experiments are complete.
>
> Internal iteration memory: V2 was created because V1 had an evaluation-design problem. The validation and test sets were not matched well enough: validation used 57 positive images plus only 19 hard-negative images, while test used 80 positive images plus 63 hard-negative images. This made test substantially harder and caused validation/test performance to diverge. V2 corrected this by using a fixed mixed real validation set for all experiments, 57 positives plus 45 hard negatives, and a fixed mixed real test set for all experiments, 80 positives plus 63 hard negatives. The purpose was to make validation and test the same type of exam: real positives plus visually confusing non-vape hard negatives. V2 also corrected dataset/graph reporting so every graph-producing cell prints the source table, saves CSVs and uses strict image-format filtering, which rejected one hidden GIF masquerading as a `.jpg` in the scraped-positive pool.
>
> Internal interpretation memory: V2 results then raised a new methodological concern. E2 performed best on the fixed real-positive plus hard-negative test set, but E2 was also the training recipe most closely matched to that validation/test domain. This does not invalidate V2, but it narrows the claim: V2 currently supports target-domain alignment and hard-negative usefulness more strongly than it supports a broad claim that scraped or synthetic positives are ineffective. Before rerunning missing model artifacts, benchmark the methodology against object-detection/YOLO ablation papers and decide whether a V3 design is needed.

Note: this document defines the intended dissertation framing for the machine-learning experiments. The final text should be checked against the completed ablation results before submission.

## Method and Results Boundary

The method section explains how the experiments were designed. It should define the data sources, train/validation/test construction, model configuration, ablation protocol and evaluation metrics.

The results section reports what the method produced. It should present dataset composition, object-scale characteristics, model performance, ablation comparisons and failure-mode behaviour.

Object-scale stratification is therefore a method decision: the validation split was constructed to preserve the distribution of tiny, small, medium and large vape instances. The evidence that this decision was necessary, such as the long-tailed bounding-box area distribution, belongs in the results as dataset characterisation.

In the final report, technical terms should be introduced through a what-why-how rhythm. The "what" can be concise and technical; the "why" and "how" should make the design choice easy to follow. For example, it is not enough to state that hard negatives were included. The report should explain that real litter scenes contain many non-vape objects that look similar to vapes, and that hard-negative images test whether the detector can reject those objects rather than simply learning to draw boxes around vape-like shapes.

## Method: Dataset and Training Protocol

### Dataset Sources

The training data was assembled from four complementary sources. Each source had a distinct role in improving detector robustness.

| Data source | Role in the dataset |
|---|---|
| Real-world staged captures | Domain anchor for outdoor, low-altitude, ground-facing imagery. |
| Scraped vape imagery | Broader intrinsic feature learning across brands, colours and form factors, with acknowledged domain mismatch. |
| Synthetic composites | Environmental variation and background decoupling. |
| Hard negatives | False-positive resistance against visually similar non-vape objects. |

Real positive images were reviewed and annotated through UAVape, with accepted annotations stored in the COCO master file. Hard negatives were retained as images with empty YOLO labels. Synthetic and external samples were used as training data only, while the held-out test set remained real.

This separation is important. Real images define the target domain, hard negatives represent confusing non-vape objects, scraped imagery broadens what the model understands as a vape, and synthetic images test whether controlled generated variation is useful. Keeping these sources separate makes it possible to ask which type of data changes detector behaviour.

Scraped vape imagery was included as a deliberate dataset-enrichment source. The initial scraping objective was to collect discarded or ground-level vape examples, but these were limited in availability. The scraping process therefore expanded to include broader vape imagery, including product-style images, brand images, handheld examples and usage-related imagery. This pivot reflects both a practical limitation of web-based dataset construction and a useful experimental condition: it tests whether broader scraped appearance diversity can improve detector learning despite domain mismatch with the target ground-litter setting.

Synthetic composites were included to test the synthetic-generation capability of UAVape as a dataset construction engine. These images are not treated as replacements for real held-out data. Instead, they test whether the engine's generated composites can provide useful training variation in scale, viewpoint and background context.

The available dataset pools are summarised below. Paths are omitted in the dissertation because the table should describe experimental role and scale rather than local implementation details.

| Pool | Intended role | Images | Labels | Boxes |
|---|---|---:|---:|---:|
| Real UAVape positive + hard-negative test set | Held-out real test set | 143 | 143 | 135 |
| Real UAVape training batch | Real training source with hard negatives | 502 | 502 | 509 |
| Generated synthetic pool | Synthetic-volume and quality diagnostic source | 1,277 | 1,277 | 1,278 |
| Promoted/accepted synthetic pool | Reviewed synthetic training augmentation source | 355 | 355 | 355 |
| Scraped vape-only YOLO pool | Scraped training diversity source after format filtering | 2,905 | 2,905 | 4,437 |

### Train, Validation and Test Construction

A fixed real held-out test set was reserved before training. This test set was reused across ablations so that changes in performance could be attributed to changes in training data rather than changes in evaluation data.

The held-out test set contains both positive images and hard-negative images. The positive subset tests whether the detector can find real vapes. The hard-negative subset tests whether the detector falsely predicts vapes on visually similar non-vape objects. This matters because real deployment imagery is not made only of vapes; it contains many distractors, and a useful detector must reject them.

The real training pool was split into training and validation subsets using object-scale stratification for positive images and a matched hard-negative proportion for hard-negative images. Scale was computed from the maximum YOLO bounding-box area as a percentage of image area. This prevented the validation set from being accidentally biased toward unusually large or unusually small vapes, while also ensuring that validation and test represented the same type of mixed real-world detection problem.

| Scale band | Rule |
|---|---|
| Tiny | max box area < 0.25% of image |
| Small | 0.25% to < 1.0% |
| Medium | 1.0% to < 5.0% |
| Large | >= 5.0% |

The train, validation and test split should be reported explicitly for each experiment. This makes the ablation auditable and shows that synthetic or scraped data were not introduced into the held-out real test set.

For dissertation readability, the final reporting order should place the synthetic-only condition before the combined scraped-plus-synthetic condition. In the Colab implementation this condition was originally named `E5_real_hardneg_synthetic`, while the combined condition was named `E4_real_hardneg_scraped_synthetic`. The dissertation-facing labels should therefore be:

| Dissertation label | Colab/run label | Meaning |
|---|---|---|
| E4 | `E5_real_hardneg_synthetic` | Real positives + hard negatives + synthetic positives |
| E5 | `E4_real_hardneg_scraped_synthetic` | Real positives + hard negatives + scraped positives + synthetic positives |

| Experiment | Training split | Validation split | Test split | Purpose of split accounting |
|---|---|---|---|---|
| E1 | Real positives only | Fixed mixed real validation | Fixed mixed real test | Establish the positive-only training baseline under the same final target evaluation setting. |
| E2 | Real positives + hard negatives | Fixed mixed real validation | Fixed mixed real test | Measure false-positive resistance while preserving the same real validation and test source. |
| E3 | Real positives + hard negatives + scraped positives | Fixed mixed real validation | Fixed mixed real test | Test scraped-data enrichment without changing validation or evaluation data. |
| E4 | Real positives + hard negatives + synthetic positives | Fixed mixed real validation | Fixed mixed real test | Isolate synthetic contribution without scraped positives. |
| E5 | Real positives + hard negatives + scraped positives + synthetic positives | Fixed mixed real validation | Fixed mixed real test | Test full dataset enrichment without contaminating validation or test data. |

The final dissertation table should replace the split descriptions above with exact image, label and box counts once the corrected ablation datasets have been built.

This table is not just bookkeeping. It is the evidence that the comparison is fair. If one experiment used a different test set, or if synthetic images entered the held-out evaluation data, the ablation result would be much weaker. Reporting the split makes the control visible.

### Model Training

YOLOv10n was used as the first detector baseline. The model was trained using pretrained weights, image size 960, cosine learning-rate scheduling and early stopping. The larger image size was chosen because vape instances frequently occupied less than 1% of the image frame.

The same model configuration, image size, seed and evaluation protocol should be used across ablations. This keeps the comparison focused on dataset composition rather than training-procedure variation.

The model is therefore held steady while the data changes. This is the core logic of the experiment: the detector architecture is not the main variable; the dataset construction choices are.

The final ablation training configuration is:

| Parameter | Value | Rationale |
|---|---:|---|
| Detector | YOLOv10n | Lightweight baseline suitable for controlled dataset ablations. |
| Pretrained weights | `yolov10n.pt` | Uses transfer learning rather than training from scratch. |
| Image size | 960 | Preserves detail for small vape instances. |
| Maximum epochs | 100 | Allows convergence for final experiments. |
| Early stopping patience | 20 | Stops training when validation performance no longer improves. |
| Batch size | 16 | Fixed across ablations to avoid hardware-dependent AutoBatch behaviour. |
| Optimizer | auto | Uses the framework's selected optimiser for the model/data setting. |
| Learning-rate schedule | cosine | Smoothly decays learning rate during training. |
| Close mosaic | 10 | Reduces heavy augmentation near the end of training. |
| Seed | 42 | Improves reproducibility across ablations. |
| Device | GPU | Required for practical training time at image size 960. |

The 100-epoch schedule is used for final ablations rather than the earlier 50-epoch smoke test. The smoke test checked that the pipeline worked; the final schedule gives each ablation a fairer opportunity to converge. Early stopping limits unnecessary compute and reduces the risk of comparing overtrained models.

The final experiments were run in Google Colab on a GPU runtime. Model-performance results should report the runtime context alongside the metrics, for example: YOLOv10n was trained on an A100 High-RAM Colab runtime with image size 960, batch size 16, maximum 100 epochs and early-stopping patience 20. Reporting this context makes the results reproducible and distinguishes final ablation runs from earlier smoke tests.

### Ablation Protocol

The ablation design is cumulative. Each experiment adds a data source or training signal on top of the previous configuration, matching the practical dataset-engine narrative of progressively enriching the training pool.

| Experiment | Training data | Validation data | Test data | Question |
|---|---|---|---|---|
| E1 | Real positives | Fixed mixed real validation | Fixed mixed real test | How well does the detector generalise when trained using only real positive examples? |
| E2 | Real positives + hard negatives | Fixed mixed real validation | Fixed mixed real test | Do hard negatives reduce false positives? |
| E3 | Real positives + hard negatives + scraped positives | Fixed mixed real validation | Fixed mixed real test | Does scraped vape data improve appearance robustness despite domain mismatch? |
| E4 | Real positives + hard negatives + synthetic positives | Fixed mixed real validation | Fixed mixed real test | Does synthetic data add value without external scraped data? |
| E5 | Real positives + hard negatives + scraped positives + synthetic positives | Fixed mixed real validation | Fixed mixed real test | Does combining scraped and synthetic enrichment improve robustness beyond either source alone? |

This design combines cumulative and isolating comparisons. E1-E3 test the effects of real positives, hard negatives and scraped positives. E4 isolates synthetic augmentation without scraped data. E5 then tests whether combining scraped and synthetic enrichment improves the detector beyond either source alone.

### Experimental Controls and Hypotheses

The control condition is E1, where the model is trained using real positive examples only. Each later experiment modifies the training data while keeping the validation and evaluation setting fixed. The held-out real test set, mixed real validation set, model family, image size, random seed, evaluation metrics and confidence/IoU evaluation settings should remain constant across experiments. This means that performance changes can be attributed primarily to changes in training data composition.

Hard negatives are included only in training and validation. They are not positive examples and do not add vape boxes. Their purpose is to teach the detector that visually similar non-vape objects should not be predicted as vapes. The expected effect is therefore improved false-positive resistance, especially in cluttered scenes or around objects such as pens, lighters, wrappers, cables and batteries.

The hard-negative test subset is used differently. It does not teach the model; it challenges the model. A positive-only model may detect vape-like objects even when no vape is present. Testing every ablation on the same hard-negative images shows whether the added training data reduces this behaviour.

| Experiment | Hypothesis | Expected effect |
|---|---|---|
| E1 real positives | Real positive data provides a baseline for vape localisation. | Reasonable localisation on clear positives, but weaker robustness to hard negatives and ambiguous scenes. |
| E2 + hard negatives | Hard negatives improve discrimination between vapes and visually similar non-vapes. | Higher precision and fewer false positives; recall may remain similar or become slightly more conservative. |
| E3 + scraped positives | Scraped vape data improves appearance diversity across brands, colours and form factors. | Higher recall and mAP on varied vape appearances, with possible domain-mismatch risk. |
| E4 + synthetic positives | Synthetic data contributes independently of scraped dataset diversity. | Helps separate synthetic scale/viewpoint effects from scraped appearance-diversity effects. |
| E5 + scraped and synthetic positives | Combining scraped appearance diversity with synthetic UAV-style variation improves robustness more than either enrichment alone. | Better overall mAP and failure-mode robustness if the two sources are complementary. |

The central hypothesis is that progressive dataset enrichment will improve robustness on the fixed real test set. Hard negatives are expected to primarily improve precision, scraped positives are expected to improve appearance generalisation and synthetic positives are expected to improve scale/viewpoint robustness.

### Evaluation Metrics

The main detection metrics are precision, recall, mAP50 and mAP50-95. Precision and recall describe detection behaviour at operating thresholds, while mAP summarises performance across confidence thresholds. Metadata-linked failure-mode analysis is used after evaluation to identify which visual conditions are associated with lower recall or F1.

These metrics answer different questions. Precision asks how often predicted vapes are correct. Recall asks how many real vapes were found. mAP50 gives a broad detection score at a standard overlap threshold, while mAP50-95 is stricter and penalises imprecise localisation. For small objects such as vapes, the stricter metric is important because a small shift in the box can represent a large localisation error.

The official checkpoint-selection rule should be stated explicitly: for each experiment, `best.pt` was selected using validation mAP50-95, and that selected checkpoint was then evaluated on the held-out test set. Validation mAP50-95 was chosen because it is stricter than mAP50 and reflects both whether the object was detected and whether the bounding box was accurately localised. mAP50 is still reported because it gives a more permissive "rough detection" score, but it should not be the checkpoint-selection metric.

Precision, recall, PR curves, confusion matrices and training curves should be used as diagnostic evidence rather than post-hoc checkpoint selectors. This prevents the test set from becoming part of model selection. If diagnostics suggest that the training recipe should change, the rigorous route is to define a revised protocol and rerun all ablations under that same protocol, rather than selectively changing one experiment after seeing its test result.

The evaluation should also include hypothesis-specific metrics. Hard-negative training is specifically intended to reduce false positives, so false positives per hard-negative test image should be reported. Synthetic augmentation is intended to improve UAV-relevant robustness, so scale- and viewpoint-stratified recall/F1 should be reported through the failure-mode analysis. Inference speed and model size can be reported as secondary practical metrics because UAV-oriented detection benefits from lightweight models, but deployment speed is not the central contribution of this work.

| Hypothesis | Primary evidence | Secondary evidence |
|---|---|---|
| Hard negatives improve false-positive resistance | False positives per hard-negative test image | Precision and qualitative hard-negative examples |
| Scraped positives improve appearance diversity | Recall, mAP50 and mAP50-95 on fixed real test set | Performance on varied colour/form-factor examples |
| Synthetic positives improve scale/viewpoint robustness | Recall/F1 by scale band and viewpoint | Metadata failure-mode heatmap |
| The detector remains practically usable | Inference speed and model size | Training/runtime notes |

## Results: Dataset Characterisation

### Dataset Composition Across Experiments

Dataset composition was reported once as a compact table. The purpose of this table is to show that each ablation changes the training data while preserving the same real held-out evaluation source.

| Experiment | Training composition | Train images | Train boxes | Validation images | Test images |
|---|---|---:|---:|---:|---:|
| E1 | Real positives | 320 | 446 | 102 | 143 |
| E2 | Real positives + hard negatives | 400 | 446 | 102 | 143 |
| E3 | Real positives + hard negatives + scraped positives | 3,305 | 4,883 | 102 | 143 |
| E4 | Real positives + hard negatives + synthetic positives | 755 | 801 | 102 | 143 |
| E5 | Real positives + hard negatives + scraped positives + synthetic positives | 3,660 | 5,238 | 102 | 143 |

The corresponding training-composition figure shows the same progression visually:

```text
Training composition by ablation

E1  real positives
E2  real positives + hard negatives
E3  real positives + hard negatives + scraped positives
E4  real positives + hard negatives + synthetic positives
E5  real positives + hard negatives + scraped positives + synthetic positives
```

E4 deliberately excludes scraped positives. Its role is to test synthetic augmentation without the much larger scraped data source, so the absence of scraped positives in E4 is intentional rather than a missing bar. E5 is the combined enrichment condition.

### Train, Validation and Test Counts

The split-accounting table shows the exact train, validation and test sizes used in the ablation. This is the evidence that the comparison is controlled: the training data changes, while the real held-out test source remains fixed after E1.

| Experiment | Train images | Validation images | Test images | Test positive boxes | Test hard negatives |
|---|---:|---:|---:|---:|---:|
| E1 | 320 | 102 fixed mixed real | 143 fixed mixed real | 135 | 63 |
| E2 | 400 | 102 fixed mixed real | 143 fixed mixed real | 135 | 63 |
| E3 | 3,305 | 102 fixed mixed real | 143 fixed mixed real | 135 | 63 |
| E4 | 755 | 102 fixed mixed real | 143 fixed mixed real | 135 | 63 |
| E5 | 3,660 | 102 fixed mixed real | 143 fixed mixed real | 135 | 63 |

The validation split is the same fixed mixed real validation set for all experiments: 57 real positive images and 45 hard-negative images. This matches the final test-set hard-negative proportion, where the fixed mixed real test contains 80 positive images and 63 hard-negative images. Scraped and synthetic images are used for training enrichment rather than validation or testing, keeping validation and test close to the target domain.

### Metadata Characteristics

Metadata distributions were used as dataset characterisation. This describes the visual conditions represented in the dataset and supports later failure-mode analysis. The purpose of this figure is not to claim that every environmental label is objectively perfect, but to give the reader a structured view of the kinds of surfaces, viewpoints, lighting conditions and object states represented in the data.

The metadata characteristics figure should be shown as a compact multi-panel count plot or small-multiples figure. The corresponding table is:

| Field | Dominant categories |
|---|---|
| Surface | pavement 229, grass 140, soil 120, tile 109 |
| Lighting | daylight 373, shadow 122, indoor 98 |
| Background clutter | none 290, low 225, medium 106, high 39 |
| Condition | intact 380, dirty 179, damaged 69, wet 32 |
| Viewpoint | top_down 302, oblique 294, close_up 64 |
| Occlusion | none 462, partial 169, heavy 29 |
| Motion blur | none 377, low 156, medium 94, high 33 |
| Scale band | medium 236, small 220, tiny 114, large 90 |

The distributions suggest that the dataset is weighted toward daylight, low-occlusion, intact-object examples, but still contains meaningful variation in viewpoint, surface type, clutter and object scale. Viewpoint is relatively balanced between top-down and oblique views, which is important for UAV-style imagery. Scale band also shows that medium and small examples dominate, while tiny examples remain a substantial subset for small-object evaluation.

This figure should not be repeated for every ablation unless dataset composition changes substantially. The corresponding failure-mode heatmap later reports whether model performance changes across these metadata categories.

### Object-Scale Distribution

Bounding-box area should be shown using a box plot or violin plot, not a normal distribution. Vape object sizes are long-tailed: most objects are small, while a minority of close-range images contain much larger boxes.

| Split | Images | Boxes | Median bbox area (%) | Mean bbox area (%) | 10th percentile (%) | 90th percentile (%) |
|---|---:|---:|---:|---:|---:|---:|
| Train | 320 | 432 | 0.733 | 2.443 | 0.146 | 6.370 |
| Validation | 57 | 77 | 0.584 | 2.282 | 0.148 | 6.915 |
| Test | 80 | 135 | 0.850 | 1.521 | 0.195 | 4.179 |

In the corrected V2 split, the equivalent unique positive-image summary was: training 320 images and 446 boxes, validation 57 images and 63 boxes, and test 80 images and 135 boxes. The validation and test splits were closely aligned in typical object scale, with median box areas of 0.811% and 0.850% of image area respectively. The test set contained more boxes per positive image than validation, meaning that it remained a slightly more demanding final recall evaluation even after the hard-negative proportion was matched.

Recommended figure:

```text
Box/violin plot: bbox area percentage by split

Train       |---[====|====]----------o-----o
Validation  |---[====|=====]-------------o
Test        |--[====|====]------o

x-axis: bbox area as % of image, preferably log-scaled
```

This figure supports two claims: the task is a small-object detection problem, and the validation split is reasonably aligned with the held-out test distribution.

### Scale-Band Composition Across Ablations

Scale bands convert continuous bounding-box size into readable object-size categories. A positive image is assigned to the largest vape box in that image: tiny, small, medium or large. A hard-negative image has no vape box, so it is shown as a background category. This makes the plot easier to interpret than raw box percentages when comparing many experiments.

The scale-band composition figure answers a specific question: what proportion of each experiment's images are hard negatives, tiny-vape images, small-vape images, medium-vape images or large-vape images? This matters because UAV-style detection is strongly affected by object scale. A model trained mostly on larger or cleaner objects may not perform as well on tiny ground-level objects.

The best representation is a faceted 100% stacked bar chart. Each bar sums to 100% of the images in that split, so the reader can compare distributions rather than raw dataset size.

Recommended figure:

```text
Scale-band composition across ablations

TRAIN
E1 | tiny | small | medium | large |
E2 | tiny | small | medium | large |
E3 | tiny | small | medium | large |
E4 | tiny | small | medium | large |

VALIDATION
E1 | tiny | small | medium | large |
E2 | tiny | small | medium | large |
E3 | tiny | small | medium | large |
E4 | tiny | small | medium | large |

TEST
E1 | fixed held-out distribution |
E2 | fixed held-out distribution |
E3 | fixed held-out distribution |
E4 | fixed held-out distribution |
```

This figure demonstrates that training composition changes across ablations while the validation and test distributions stay fixed. E1 contains only positive vape images during training, while E2-E5 include hard-negative training images. All experiments are validated and tested on the same mixed real target distribution. E4 excludes scraped positives by design, allowing synthetic augmentation to be examined without the larger scraped source.

The observed scale-band distribution also characterises the behaviour of the added data sources. Scraped positives substantially increase the proportion of large-vape training images: in E3, large-vape images form 60.4% of the training split. This supports the earlier methodological concern that scraped web imagery broadens vape appearance diversity but may not match the small-object ground-litter setting. By contrast, E4, which adds synthetic positives without scraped positives, shifts the training distribution toward smaller objects: tiny and small vape images together form approximately 62.6% of the training split. This makes E4 useful for testing whether UAVape-generated synthetic data improves small-object robustness independently of scraped web imagery.

The fixed test distribution for all experiments contains 44.1% hard-negative images, 8.4% tiny-vape images, 23.1% small-vape images, 23.1% medium-vape images and 1.4% large-vape images. The fixed validation distribution is closely matched, with 44.1% hard-negative images, 12.7% tiny-vape images, 17.6% small-vape images, 16.7% medium-vape images and 8.8% large-vape images. This composition is important because the detector must find vapes while also avoiding false positives on hard negatives.

## Results: Model Performance

### Baseline Result

The positive-only baseline establishes the minimum real-data detector performance before adding hard negatives, scraped data or synthetic augmentation. The final baseline should be reported using the corrected mixed real validation and test protocol.

| Experiment | Precision | Recall | mAP50 | mAP50-95 |
|---|---:|---:|---:|---:|
| E1 real positive only | 0.481 | 0.778 | 0.607 | 0.488 |

The baseline result should be interpreted as a positive-only training baseline evaluated on a mixed real test set. This is stricter than positive-only testing because the model must localise real vapes while also avoiding false positives on hard-negative images. E1 achieved relatively high recall but low precision, which is consistent with the experimental hypothesis: without hard-negative training, the detector can learn vape appearance but has limited evidence for rejecting visually similar non-vape objects.

### Ablation Result Table

The main quantitative result should be one table containing every ablation.

| Experiment | Training recipe | Precision | Recall | mAP50 | mAP50-95 | FP / hard-neg image | Interpretation |
|---|---|---:|---:|---:|---:|---:|---|
| E1 | Real positives | 0.481 | 0.778 | 0.607 | 0.488 | TBD | Positive-only training baseline: high recall but weak false-positive resistance. |
| E2 | Real positives + hard negatives | 0.576 | 0.763 | 0.686 | 0.561 | TBD | Hard-negative training improves precision and mAP with only a small recall reduction. |
| E3 | Real positives + hard negatives + scraped positives | 0.587 | 0.793 | 0.665 | 0.542 | TBD | Scraped data slightly improves recall but reduces mAP relative to E2. |
| E4 | Real positives + hard negatives + synthetic positives | 0.645 | 0.704 | 0.636 | 0.494 | TBD | Synthetic-only enrichment improves precision but reduces recall and mAP relative to E2. |
| E5 | Real positives + hard negatives + scraped positives + synthetic positives | 0.579 | 0.765 | 0.677 | 0.542 | TBD | Combined enrichment performs similarly to scraped-only E3 and does not exceed E2. |

Recommended figure:

```text
Grouped bar chart across ablations

metric groups: precision, recall, mAP50, mAP50-95
x-axis: E1, E2, E3, E4, E5
```

The table is the formal result. The grouped bar chart makes the direction of improvement visible.

The final ablation table should use only the corrected V2 runs. Earlier exploratory runs using an unmatched validation/test protocol should not be reported as the main result.

The E2 result supports the hard-negative hypothesis. Relative to E1, adding hard-negative training images increased precision from 0.481 to 0.576 and improved mAP50-95 from 0.488 to 0.561 on the same fixed mixed real test set. Recall decreased only slightly, from 0.778 to 0.763. This pattern suggests that hard negatives improved the detector's ability to reject confusing non-vape imagery without substantially reducing its ability to find real vape instances.

The E3 result provides mixed evidence for scraped-data enrichment. Compared with E2, adding scraped positives increased recall from 0.763 to 0.793 and precision from 0.576 to 0.587, but reduced mAP50 from 0.686 to 0.665 and mAP50-95 from 0.561 to 0.542. This suggests that scraped imagery may help the detector recognise more vape-like appearances, but does not improve localisation quality on the real UAVape test set. This is consistent with the earlier dataset-characterisation result that scraped positives are dominated by larger, cleaner, more product-like vape images rather than small ground-litter views.

The E4 result shows that synthetic-only enrichment was also mixed. Compared with E2, adding synthetic positives increased precision from 0.576 to 0.645, but reduced recall from 0.763 to 0.704 and reduced mAP50-95 from 0.561 to 0.494. This suggests that the synthetic data made the detector more conservative: predictions were more likely to be correct, but more real vape instances were missed and overall localisation quality decreased. A final-epoch sensitivity check produced higher precision and higher test mAP than the validation-selected E4 checkpoint, but it was not used as the official result because checkpoint selection must be based on validation performance rather than test-set feedback.

The E5 combined-enrichment result did not provide evidence that scraped and synthetic positives were complementary under this training recipe. Compared with E2, E5 achieved similar precision and recall but lower mAP50-95. Compared with E3, E5 was nearly identical in mAP50-95 and slightly lower in recall. This suggests that adding synthetic positives to the scraped-positive training pool did not recover the localisation loss associated with the scraped data, and the best-performing official V2 configuration remained E2.

False positives per hard-negative image should be shown as a separate compact chart because it directly tests the hard-negative hypothesis.

```text
False positives per hard-negative test image

E1  ██████████
E2  ████
E3  ███
E4  ██
```

Lower values indicate better false-positive resistance.

### Precision-Recall Tradeoff

Precision-recall behaviour should be shown across experiments using either overlaid PR curves or a precision-recall scatter plot.

Recommended figure:

```text
Recall
1.0 |
0.9 |                  E4
0.8 |          E3
0.7 |    E1
0.6 |       E2
    +-------------------------
      0.6  0.7  0.8  0.9  Precision
```

This figure is useful when one ablation improves recall but reduces precision, or when hard negatives increase precision by suppressing false positives.

### Training Curves

Training curves should not be repeated for every ablation in the main body. A representative curve for the baseline or final model is sufficient unless a specific experiment shows instability.

Recommended figure:

```text
Validation metrics over epochs

lines: precision, recall, mAP50, mAP50-95
model: representative or final selected model
```

This figure demonstrates convergence and supports the chosen epoch/early-stopping protocol.

## Results: Failure-Mode Analysis

### Metadata Failure-Mode Heatmap

Metadata-linked evaluation should be aggregated. The main body should avoid separate plots for every metadata field. The strongest format is a heatmap with failure modes as rows and experiments as columns.

Recommended figure:

```text
Recall by failure mode and ablation

                 E1     E2     E3     E4
tiny scale       0.xx   0.xx   0.xx   0.xx
small scale      0.xx   0.xx   0.xx   0.xx
oblique view     0.xx   0.xx   0.xx   0.xx
partial occl.    0.xx   0.xx   0.xx   0.xx
high clutter     0.xx   0.xx   0.xx   0.xx
motion blur      0.xx   0.xx   0.xx   0.xx
```

This figure turns metadata into an evaluation instrument. It shows whether a data source improves specific weaknesses, such as tiny-object recall, oblique-view robustness or cluttered-scene performance.

The category-level table behind the heatmap should include support counts so that weak results from tiny sample sizes are not overinterpreted.

| Field | Category | Images | Boxes | Recall | F1 |
|---|---|---:|---:|---:|---:|
| Scale | tiny | TBD | TBD | TBD | TBD |
| Scale | small | TBD | TBD | TBD | TBD |
| Viewpoint | oblique | TBD | TBD | TBD | TBD |
| Occlusion | partial | TBD | TBD | TBD | TBD |
| Clutter | high | TBD | TBD | TBD | TBD |
| Motion blur | medium/high | TBD | TBD | TBD | TBD |

The heatmap should prioritise failure modes relevant to UAV-style hazardous micro-e-waste detection: tiny/small scale, oblique or top-down viewpoint, clutter, occlusion, motion blur and challenging surfaces. Metadata categories with very small support should be omitted from the main figure or reported cautiously.

### Prediction Confidence Distribution

Prediction confidence can be shown if it provides a useful threshold discussion. The cleanest version is a histogram or density plot comparing true-positive and false-positive confidence scores for the final or best model.

Recommended figure:

```text
Prediction confidence distribution

TP confidence  ███▅▆████████▇▆▅
FP confidence  █████▆▅▃▂

x-axis: confidence score
y-axis: density/count
```

This plot should only be included if the distributions are meaningfully different. It should not be forced as a normal distribution.

### Inference Efficiency

Inference speed and model size should be reported briefly as practical context. This is secondary to detection performance, but it helps connect the work to UAV-oriented deployment constraints.

| Model | Image size | Parameters | Inference speed | Notes |
|---|---:|---:|---:|---|
| YOLOv10n | 960 | TBD | TBD | Lightweight baseline. |

### Qualitative Prediction Montage

A small montage should ground the numeric results in visual examples. It should show selected cases rather than a large gallery.

| Example type | Purpose |
|---|---|
| True positive, clear object | Shows baseline capability. |
| True positive, tiny object | Shows small-object success. |
| False negative, tiny or blurred object | Shows remaining limitation. |
| False positive on hard negative | Shows why hard negatives matter. |
| Improved case from later ablation | Shows the effect of dataset expansion. |

## Final Figure Set

| Figure | Dissertation section | Scope | Purpose |
|---|---|---|---|
| Dataset composition table/bar | Dataset characterisation | Across experiments | Shows what each ablation used. |
| Metadata characteristics distribution | Dataset characterisation | Show once | Describes the visual conditions represented in the dataset. |
| Object-scale box/violin plot | Dataset characterisation | Show once | Establishes small-object nature and split balance. |
| Scale-band stacked composition | Dataset characterisation | Across experiments if useful | Shows scale distribution across ablations and fixed test set. |
| Training curves | Model performance | Show once | Demonstrates convergence. |
| Main ablation table | Model performance | Across experiments | Reports precision, recall, mAP50 and mAP50-95. |
| Ablation grouped bar chart | Model performance | Across experiments | Visualises performance changes. |
| Hard-negative false-positive chart | Model performance | Across experiments | Directly tests false-positive resistance. |
| Precision-recall comparison | Model performance | Across experiments | Shows operating tradeoffs. |
| Metadata failure-mode heatmap | Error analysis | Across experiments | Shows where models struggle or improve. |
| TP/FP confidence distribution | Error analysis | Show once if useful | Supports confidence-threshold discussion. |
| Inference efficiency table | Practical relevance | Show once | Reports speed and model size as secondary deployment context. |
| Qualitative montage | Error analysis | Show once | Provides visual examples of successes and failures. |

## Presentation Rules

- Use all-experiment figures only when comparison is the point.
- Show object-scale distribution once unless a source-specific scale shift is central to the argument.
- Use a 100% stacked bar for scale-band composition, not repeated grouped bars.
- Use one metadata failure-mode heatmap in the main body; keep detailed per-field plots for appendix or internal analysis.
- Do not show every Ultralytics-generated plot if it repeats the same result.
- Do not include a normal distribution curve unless the data genuinely supports that assumption.
- Do not report synthetic or external data as part of the held-out test set.
