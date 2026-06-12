# Deep-Learning Evaluation of Constructed UAVape Datasets

## Section Role

The previous dataset construction section established how UAVape can ingest multiple image sources, support vape annotation, attach metadata, store annotations and export YOLO-ready datasets. This section evaluates whether those constructed datasets improve downstream object-detection performance.

The distinction is important. The Dataset Construction Engine (DCE) is not itself a detector-training system. Its role is to make different data sources usable, auditable and exportable for object-detection experiments. The deep-learning evaluation described here asks what happens after those constructed datasets are used to train a fixed detector. In other words, this section tests whether the data-construction capability translates into measurable detection improvement on real UAV-oriented vape-litter imagery.

The wider dissertation asks:

> How can an integrated image-detection dataset construction engine, combined with deep-learning evaluation and workflow prototyping, overcome the structural barriers to identifying and operationalizing disposable vapes as hazardous micro-e-waste in UAV-oriented imagery for urban environments?

This section addresses the deep-learning evaluation part of that question. It examines whether scraped and synthetic vape-positive data, constructed through the UAVape workflow, improve real target-domain detection when compared against a real-data baseline containing hard negatives.

The central evaluation question is therefore:

> When a YOLO26n detector is trained for UAV vape-litter detection, do additional DCE-constructed scraped or synthetic vape-positive images improve held-out real target-domain performance compared with a real-data baseline containing hard negatives?

This is a data-centric question rather than a model-leaderboard question. The aim is not to find the best possible detector architecture, but to test whether different constructed data sources help a fixed lightweight detector solve the target UAVape detection problem.

## Methodological Positioning

This evaluation sits between the dataset construction chapter and the broader workflow-prototyping contribution of the dissertation. The dataset construction chapter shows that UAVape can create YOLO-ready object-detection datasets from multiple image sources. However, a dataset construction engine is only useful for the research question if its outputs can be tested against the actual detection problem. This section therefore asks a narrower but necessary question: once the DCE has produced real, hard-negative, scraped and synthetic training sources, which of those sources actually improves real UAV-oriented vape detection?

The evaluation is framed as an ablation study. In machine-learning evaluation, an ablation study tests the contribution of one part of a system by changing that part while keeping the rest of the system fixed. In this dissertation, the changing component is not the detector architecture, loss function or training schedule. The changing component is the training data source. This is why the experiment is described as a data-centric ablation: it asks how the detector behaves when the training data is altered in controlled source-dose steps.

This design is motivated by the literature benchmark developed before the final experiment. Domain-adjacent object-detection studies typically do not rely only on a single headline mAP value. Stronger studies report the training/evaluation split, keep a fixed held-out test set, compare precision and recall alongside mAP, examine failure cases or hard examples, and use slice-level analysis when object scale or deployment conditions matter [CITE: UAV/object-detection evaluation studies]. Ablation-focused studies also make the changed component visible: each experiment should alter one identifiable factor while preserving the remaining protocol [CITE: ablation-study methodology / data-centric AI]. These expectations shaped the final evaluation design.

The UAVape problem makes this especially important. Disposable vapes are small hazardous micro-e-waste objects in cluttered urban ground imagery, and they can resemble lighters, pens, wrappers or batteries. A model that detects vapes in clean product imagery is not automatically useful for UAV litter recovery. The evaluation therefore has to test the constructed sources against the target setting: small real vapes in UAV-oriented images, plus hard-negative images where no vape is present. This is also why the study reports hard-negative false-positive pressure and scale-band performance, not just aggregate mAP [CITE: small-object detection / UAV litter detection].

The scraped and synthetic pathways are methodologically interesting because they represent two common ways to address data scarcity. Scraped images can increase real-world product appearance diversity, but may differ from UAV imagery in viewpoint, background, scale and framing. Synthetic images can provide controlled generated variation, but may suffer from sim-to-real mismatch when rendered or generated examples do not match real sensor conditions. The ablation therefore tests a practical question raised by the literature: whether extra non-target-domain data helps enough to overcome its domain gap [CITE: domain shift / sim-to-real synthetic data].

The result is not intended to be a universal statement that scraped or synthetic data is good or bad. It is a controlled, target-domain evaluation of how these DCE-constructed sources behaved under one fixed lightweight detector, one fixed validation/test protocol and matched source-dose levels. This is the appropriate scale of claim for the dissertation: the DCE is evaluated as a way to construct and audit source pathways, while the YOLO26n study tests whether those pathways transfer into the UAVape detection task.

## Evaluation Aim and Design Requirements

The aim of this evaluation was to test whether DCE-constructed scraped and synthetic vape-positive datasets improve YOLO26n detection of real UAVape imagery, and to characterise any source-specific performance tradeoffs.

To make this evaluation meaningful, the study was designed around the following requirements.

| Design requirement | Why it matters | How the evaluation satisfies it |
|---|---|---|
| DR1: Evaluate constructed data-source pathways | The DCE can produce multiple training sources, but the dissertation must test whether those sources help detection. | The ablation compares a real baseline, scraped-augmented training and synthetic-augmented training. |
| DR2: Identify whether external data improves real UAVape detection | The aim is not to improve scraped-domain or synthetic-domain performance, but real UAV-oriented vape detection. | All conditions are evaluated on fixed real UAVape validation and test sets. |
| DR3: Use a UAV-suitable lightweight detector | The model should fit the UAV/near-edge motivation and avoid turning the study into architecture search. | YOLO26n is fixed across all conditions as a nano-scale detector. |
| DR4: Use a localisation-aware primary metric | Vape litter recovery requires not just class recognition but accurate object localisation. | mAP50-95 is used as the primary validation checkpoint and official comparison metric. |
| DR5: Measure detection success and false-positive risk | Operational vape detection must find vapes while avoiding hallucinations on similar non-vape objects. | Precision, recall, hard-negative false positives and confidence diagnostics are reported. |
| DR6: Compare source additions at controlled dose levels | A broad "more data" experiment cannot separate source effect from data quantity. | Scraped and synthetic sources are tested at matched 88, 177 and 355 positive-image doses. |
| DR7: Preserve fair real target-domain evaluation | Metric differences should not come from changing validation/test composition. | Validation and test splits are fixed and use matched positive-to-hard-negative proportions. |
| DR8: Explain model behaviour beyond headline mAP | A single aggregate metric cannot explain why a source helped, hurt or changed model behaviour. | Source-dose curves, validation-test gaps, scale-band analysis, hard-negative FP pressure and threshold diagnostics are used. |
| DR9: Produce reproducible and reusable evidence for a niche detection problem | UAV vape-litter detection is an under-documented hazardous micro-e-waste task, so other researchers need traceable outputs. | The workflow saves manifests, CSV summaries, prediction diagnostics, figures, workbooks, checkpoints and Drive backups. |

These requirements define what the evaluation needed to achieve. The following method explains how the requirements were implemented.

## Method

### Pilot Refinement of the Final Protocol

Preliminary pilot runs were used to validate the training, backup and reporting workflow before the final controlled ablation was completed. In this context, "controlled ablation" means that the detector and evaluation protocol are held constant while the training data is changed deliberately. The purpose is to make the comparison interpretable. If a model improves after a change, the study should be able to say what changed and why that change might have mattered.

The early runs showed that a broad "add all external data" comparison was too blunt to explain source effects. If all scraped or synthetic data were added at once, any metric change could be caused by source type, source quantity, evaluation composition, object-scale composition or confidence-threshold behaviour. That would make the result difficult to defend because the experiment would show that "something changed" without isolating the likely reason.

The final protocol was therefore made more controlled. Validation and test splits were fixed. Both contained real target-domain positives and hard-negative no-vape images. Scraped and synthetic images were only introduced into training. External additions were tested at matched dose levels, and add-on subsets were sampled with fixed seed and scale-band stratification where possible. The resulting experiment was designed to isolate how source type and source dose affected a fixed YOLO26n detector.

The earlier pilot outputs are treated as protocol-refinement evidence only. They are not mixed into the official YOLO26n result tables.

### Dataset Sources

The DCE constructed and exported four relevant source types for the final evaluation: real UAVape positive images, hard-negative no-vape images, scraped vape-positive images and synthetic vape-positive images.

Real UAVape positives form the target-domain anchor. They represent the visual conditions the detector is ultimately expected to handle: small disposable vapes in UAV-oriented ground imagery. Hard negatives are no-vape images containing visually similar distractors, especially lighter-like objects. These are important because the operational task is not only to find vape instances, but also to avoid hallucinating vapes on similar non-vape litter.

Scraped positives provide real-world vape appearance diversity, including different brands, colours and form factors. However, scraped images risk out-of-distribution (OOD) mismatch because they often differ from UAV imagery in viewpoint, framing, background and object scale. Synthetic positives provide controlled generated variation, but risk sim-to-real mismatch because generated object boundaries, texture, lighting and resolution may not match real UAV sensor imagery.

Table 1 summarises the available source pools.

| Pool | Images | Positive images | Hard negatives | Boxes |
|---|---:|---:|---:|---:|
| Positive train | 377 | 377 | 0 | 509 |
| Hard-negative train | 125 | 0 | 125 | 0 |
| Positive test | 80 | 80 | 0 | 135 |
| Hard-negative test | 63 | 0 | 63 | 0 |
| Scraped positive | 2,905 | 2,905 | 0 | 4,437 |
| Synthetic positive | 355 | 355 | 0 | 355 |

Before training, the source pools were audited for image counts, box counts and native image dimensions. This audit is important because source differences are not only semantic. They also affect scale, resolution and visual appearance before images are normalised into the YOLO training size. Table 2 shows this native-dimension audit.

| Pool | Images | Median W x H | Median max side | % max side > 960 |
|---|---:|---:|---:|---:|
| Positive train | 377 | 480 x 640 | 640 | 0.0 |
| Hard-negative train | 125 | 480 x 640 | 640 | 0.0 |
| Positive test | 80 | 640 x 480 | 640 | 0.0 |
| Hard-negative test | 63 | 640 x 480 | 640 | 0.0 |
| Scraped positive | 2,905 | 604 x 479 | 640 | 29.6 |
| Synthetic positive | 355 | 3668 x 2048 | 3668 | 100.0 |

The real target-domain images were mostly 640 by 480 or 480 by 640 and were upscaled into the 960-pixel YOLO training configuration. Scraped images were more heterogeneous, while the synthetic images were substantially larger and therefore downscaled during training. This does not by itself prove that resolution differences caused later performance changes, but it establishes an important source characteristic that must be considered when interpreting OOD and sim-to-real behaviour.

### Fixed Validation and Test Protocol

The validation and test sets were fixed across all experiments. This was necessary because the study aimed to compare training-source effects, not differences in evaluation difficulty. Scraped and synthetic images were excluded from validation and test. Every model was therefore evaluated on the same real target-domain problem: detecting real vape instances and rejecting lighter-like hard negatives. This follows standard object-detection evaluation logic: when the training intervention changes, the held-out evaluation set must remain fixed so the metric change can be attributed to the intervention rather than to a new test distribution [CITE: object-detection evaluation protocol]. Table 3 gives the fixed evaluation split sizes.

| Split | Images | Positive images | Hard-negative images | Boxes |
|---|---:|---:|---:|---:|
| Validation | 102 | 57 | 45 | 63 |
| Test | 143 | 80 | 63 | 135 |

The validation and test sets used matched positive-to-hard-negative proportions. The validation set was used for checkpoint selection only. The held-out test set was not used for training, checkpoint selection, source-dose design or official threshold tuning. It was used for final official reporting, with post-hoc diagnostics computed afterward to interpret model behaviour.

### Source-Dose Ablation Design

The final ablation used a real target-domain baseline and six external-data variants. The baseline condition trained on the real positive foundation and the real hard-negative foundation. Scraped and synthetic positives were then added at matched dose levels of 88, 177 and 355 images. Table 4 defines the seven experiment conditions.

| Experiment | Training recipe | Added external positives | Purpose |
|---|---|---:|---|
| E1 | Real positives + hard negatives | 0 | Operational baseline |
| E2 | E1 + scraped positives | 88 | Low scraped dose |
| E3 | E1 + scraped positives | 177 | Medium scraped dose |
| E4 | E1 + scraped positives | 355 | Full scraped dose |
| E5 | E1 + synthetic positives | 88 | Low synthetic dose |
| E6 | E1 + synthetic positives | 177 | Medium synthetic dose |
| E7 | E1 + synthetic positives | 355 | Full synthetic dose |

This design supports three comparisons. First, each augmented condition can be compared against the real baseline. Second, low, medium and full doses can be compared within each pathway. Third, matched doses can be compared across scraped and synthetic pathways. This is the core ablation logic: the study changes the data source and dose while preserving the detector, image size, checkpoint rule and evaluation split [CITE: ablation-study design].

Where possible, add-on subsets were nested and sampled with fixed seed 42. Sampling was stratified by `scale_band`, so object-scale composition was approximately stabilised within each pathway. This does not mean every source characteristic was perfectly equalised. Native resolution and visual style remained source-specific and were audited rather than fully controlled.

### Detector and Training Configuration

YOLO26n was used as the fixed detector across all experiments. The nano-scale detector was selected because the study focuses on data construction rather than model architecture search, and because a lightweight detector is more consistent with UAV or near-edge deployment motivations. Lightweight YOLO-family detectors are commonly used as practical baselines in real-time and resource-constrained object-detection studies, so fixing a nano model keeps the evaluation aligned with the UAV deployment context while avoiding a separate architecture comparison [CITE: lightweight YOLO / UAV object detection]. Table 5 summarises the fixed training configuration.

| Parameter | Value |
|---|---|
| Detector | YOLO26n |
| Weights | `yolo26n.pt` |
| Image size | 960 |
| Batch | 16 |
| Epochs | 100 |
| Seed | 42 |
| Checkpoint rule | best validation mAP50-95 |
| Official test | fixed test set using selected `best.pt` |

The detector, image size, seed, training schedule and checkpoint rule were held constant so that the comparison remained focused on the constructed training data. The official checkpoint for each experiment was selected using validation mAP50-95. The test set was evaluated only after this checkpoint had been selected. Confidence-threshold diagnostics were generated later as post-hoc operating-point analysis and were not used to select the official checkpoint or alter the official fixed-test metrics.

### Metric Rationale

The primary comparison uses mAP50-95 because it is localisation-aware and stricter than mAP50 alone. mAP50 reports performance at a single IoU threshold of 0.50 and is useful for approximate localisation. mAP50-95 averages performance across stricter IoU thresholds from 0.50 to 0.95, making it a stronger test of whether predicted boxes accurately localise the vape object [CITE: COCO-style object-detection metrics].

Precision and recall are interpreted as behavioural metrics. Precision indicates how many predicted vape detections are correct, and therefore reflects false-positive pressure. Recall indicates how many labelled vape instances are detected, and therefore reflects missed-vape risk. For UAV vape-litter detection, both matter: a useful detector must find small hazardous objects while avoiding excessive false alarms on visually similar distractors.

## Results

### Dataset Composition Across Experiments

Before interpreting model performance, it is necessary to show what changed between training conditions. Table 6 confirms that the real positive foundation and hard-negative foundation remained constant, while the added external positive source varied by pathway and dose.

| Exp | Pathway | Dose | Real positives | Hard negatives | Scraped positives | Synthetic positives | Train images | Train boxes |
|---|---|---:|---:|---:|---:|---:|---:|---:|
| E1 | baseline | 0 | 320 | 80 | 0 | 0 | 400 | 446 |
| E2 | scraped | 88 | 320 | 80 | 88 | 0 | 488 | 579 |
| E3 | scraped | 177 | 320 | 80 | 177 | 0 | 577 | 731 |
| E4 | scraped | 355 | 320 | 80 | 355 | 0 | 755 | 1,039 |
| E5 | pure_synthetic | 88 | 320 | 80 | 0 | 88 | 488 | 534 |
| E6 | pure_synthetic | 177 | 320 | 80 | 0 | 177 | 577 | 623 |
| E7 | pure_synthetic | 355 | 320 | 80 | 0 | 355 | 755 | 801 |

This table matters because it prevents the results from appearing as seven unrelated model runs. The experiment changes the source mixture in a controlled way while preserving the same real baseline foundation, hard-negative foundation and fixed evaluation sets.

### Official Fixed-Test Performance

The official fixed-test metrics answer the main evaluation question: did any scraped or synthetic source-dose condition improve real UAVape detection over the real hard-negative baseline? Table 7 reports the official held-out test metrics.

| Exp | Pathway | Dose | P | R | mAP50 | mAP50-95 |
|---|---|---:|---:|---:|---:|---:|
| E1 | baseline | 0 | 0.695 | 0.776 | 0.776 | 0.643 |
| E2 | scraped | 88 | 0.755 | 0.756 | 0.752 | 0.601 |
| E3 | scraped | 177 | 0.780 | 0.709 | 0.792 | 0.634 |
| E4 | scraped | 355 | 0.699 | 0.667 | 0.735 | 0.594 |
| E5 | pure_synthetic | 88 | 0.737 | 0.659 | 0.715 | 0.572 |
| E6 | pure_synthetic | 177 | 0.721 | 0.728 | 0.731 | 0.604 |
| E7 | pure_synthetic | 355 | 0.737 | 0.719 | 0.726 | 0.599 |

![Official fixed-test metric bars](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_official_test_metric_bars.png)

**Figure 1.** Official fixed-test precision, recall, mAP50 and mAP50-95 for all YOLO26n source-dose experiments. All results use the validation-selected `best.pt` checkpoint and the same fixed real test set.

The real hard-negative baseline remained the strongest overall condition on the official fixed test set. E1 achieved mAP50-95 of 0.643 and recall of 0.776. No scraped or synthetic augmentation condition exceeded this mAP50-95 value. This means that increasing positive-image volume through external sources did not improve the primary localisation-aware metric in the final controlled ablation.

This does not mean that external data was useless. Several augmented conditions increased precision, indicating changed model selectivity. However, the primary held-out localisation metric did not improve over the real target-domain baseline.

### Source-Dose Effect on mAP50-95

The source-dose mAP50-95 figure shows whether scraped or synthetic data helped, plateaued or harmed performance as dose increased.

![Source-dose mAP50-95](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_source_dose_mAP50_95.png)

**Figure 2.** Source-dose response for strict localisation-aware mAP50-95. The dashed baseline represents E1, while scraped and synthetic lines show matched external positive doses.

The scraped pathway did not surpass the baseline. The medium scraped dose nearly recovered the baseline mAP50-95, but remained slightly below it. The full scraped dose then degraded further. The synthetic pathway showed a different shape: low-dose synthetic performed worst, medium-dose synthetic recovered, and full-dose synthetic plateaued below the baseline.

This pattern is more informative than a simple "external data failed" conclusion. Scraped and synthetic data produced different response curves, suggesting different forms of domain mismatch.

### Precision and Recall Tradeoffs

Precision and recall separate model selectivity from target-instance coverage. These curves explain why mAP50-95 alone is not enough to interpret behaviour.

![Source-dose precision](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_source_dose_precision.png)

![Source-dose recall](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_source_dose_recall.png)

**Figure 3.** Source-dose precision and recall curves. These figures separate false-positive control from target-instance coverage.

Scraped data produced a clear precision peak at E3, where precision reached 0.780. This suggests that scraped images contributed useful product-appearance information and made the detector more selective. However, recall decreased at the same time. The model became better at being confident when it predicted a vape, but worse at detecting all real vape instances in the fixed test set.

Synthetic data produced a smaller precision gain and a partial recall recovery from E5 to E6. However, this recovery still did not exceed the baseline. The synthetic pathway therefore appears to provide some transferable signal, but not enough to improve the primary official test result.

### Deltas Against the Baseline

The delta view makes the comparison against the real hard-negative baseline explicit. Table 8 and Figure 4 show the same comparison in numeric and visual form.

| Exp | Pathway | Dose | P delta | R delta | mAP50 delta | mAP50-95 delta |
|---|---|---:|---:|---:|---:|---:|
| E1 | baseline | 0 | 0.000 | 0.000 | 0.000 | 0.000 |
| E2 | scraped | 88 | 0.060 | -0.020 | -0.024 | -0.042 |
| E3 | scraped | 177 | 0.085 | -0.067 | 0.016 | -0.009 |
| E4 | scraped | 355 | 0.004 | -0.109 | -0.041 | -0.050 |
| E5 | pure_synthetic | 88 | 0.042 | -0.117 | -0.061 | -0.071 |
| E6 | pure_synthetic | 177 | 0.026 | -0.048 | -0.045 | -0.040 |
| E7 | pure_synthetic | 355 | 0.042 | -0.057 | -0.050 | -0.045 |

![Delta against baseline](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_delta_vs_baseline.png)

**Figure 4.** Metric deltas relative to E1. Positive values indicate improvement over the real hard-negative baseline; negative values indicate degradation.

The delta chart shows that external additions mostly improved precision while reducing recall and mAP50-95. E3 produced the largest precision gain and slightly improved mAP50, but it still did not improve mAP50-95. This distinction matters because mAP50 can reward approximate localisation, whereas mAP50-95 demands stricter localisation quality.

The result supports a nuanced interpretation: external positives changed model behaviour, especially precision, but did not improve the primary target-domain localisation metric.

### Validation-Test Gap

Validation metrics were used to select checkpoints, but the fixed test set provides the official held-out result. The validation-test gap tests whether validation-side improvements transferred to real held-out performance. Table 9 and Figure 5 compare validation and test mAP50-95.

| Exp | Pathway | Val mAP50-95 | Test mAP50-95 | Test minus val |
|---|---|---:|---:|---:|
| E1 | baseline | 0.733 | 0.643 | -0.090 |
| E2 | scraped | 0.764 | 0.601 | -0.163 |
| E3 | scraped | 0.749 | 0.634 | -0.115 |
| E4 | scraped | 0.746 | 0.594 | -0.152 |
| E5 | pure_synthetic | 0.746 | 0.572 | -0.173 |
| E6 | pure_synthetic | 0.699 | 0.604 | -0.095 |
| E7 | pure_synthetic | 0.742 | 0.599 | -0.143 |

![Validation-test gap mAP50-95](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_validation_test_gap_map50_95.png)

**Figure 5.** Difference between validation mAP50-95 and official test mAP50-95. More negative values indicate weaker transfer from validation selection to held-out test performance.

Several augmented conditions looked competitive on validation but did not transfer to the held-out test set. E5 and E7 are especially important examples: both had validation mAP50-95 close to or above the baseline, but their test mAP50-95 remained below the baseline. This confirms why the final interpretation should be based primarily on the fixed held-out test set rather than validation metrics alone.

## Diagnostic Behaviour

The official metrics answer the main performance question, but they do not explain all model behaviour. The diagnostic analyses therefore ask a second question: if scraped and synthetic data did not win overall, what did they change?

### Hard-Negative False-Positive Pressure

The low-threshold diagnostic pass exposed weak candidate detections by using `conf=0.001`. This is not a deployment threshold. It is a way to compare relative sensitivity to hard-negative distractors. Hard-negative and failure-case analysis are common ways to move beyond aggregate mAP and understand whether a detector is confusing the target class with visually similar background objects [CITE: hard-negative mining / failure-mode analysis]. Table 10 and Figure 6 summarise this hard-negative pressure.

| Exp | Pathway | Dose | Hard-neg images | FP predictions | Images with FP | FP / image | Max FP conf |
|---|---|---:|---:|---:|---:|---:|---:|
| E1 | baseline | 0 | 63 | 207 | 50 | 3.29 | 0.969 |
| E2 | scraped | 88 | 63 | 235 | 55 | 3.73 | 0.920 |
| E3 | scraped | 177 | 63 | 231 | 59 | 3.67 | 0.949 |
| E4 | scraped | 355 | 63 | 267 | 61 | 4.24 | 0.941 |
| E5 | pure_synthetic | 88 | 63 | 282 | 60 | 4.48 | 0.972 |
| E6 | pure_synthetic | 177 | 63 | 182 | 55 | 2.89 | 0.980 |
| E7 | pure_synthetic | 355 | 63 | 251 | 59 | 3.98 | 0.973 |

![Hard-negative false-positive pressure](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_hard_negative_fp_pressure.png)

**Figure 6.** Low-threshold hard-negative false-positive pressure. Predictions were generated at `conf=0.001`, so the figure compares diagnostic sensitivity rather than deployment-threshold false-positive rates.

E6 produced the lowest low-threshold hard-negative false-positive pressure. One plausible explanation is that the medium-dose synthetic data provided clearer vape-shape variation, helping the detector separate vape-like geometry from lighter-like distractors. However, this did not translate into the best overall mAP50-95. Improved distractor rejection alone was not enough to overcome the broader sim-to-real localisation limits.

### Scale-Band Performance

UAV vape detection is a small-object problem. Scale-band diagnostics test whether failures are concentrated in smaller object scales. Table 11 summarises recall by object-scale band.

| Exp | Pathway | Dose | Large R | Medium R | Small R | Tiny R | Total FP |
|---|---|---:|---:|---:|---:|---:|---:|
| E1 | baseline | 0 | 1.000 | 0.982 | 0.919 | 0.929 | 390 |
| E2 | scraped | 88 | 1.000 | 0.982 | 0.952 | 0.929 | 353 |
| E3 | scraped | 177 | 1.000 | 1.000 | 0.935 | 0.929 | 395 |
| E4 | scraped | 355 | 1.000 | 1.000 | 0.935 | 0.857 | 365 |
| E5 | pure_synthetic | 88 | 1.000 | 1.000 | 0.919 | 0.929 | 424 |
| E6 | pure_synthetic | 177 | 1.000 | 1.000 | 0.855 | 0.929 | 317 |
| E7 | pure_synthetic | 355 | 1.000 | 0.964 | 0.839 | 0.929 | 302 |

All experiments remained strong on large and medium vapes under the low-threshold diagnostic pass. The more meaningful differences appeared in small-vape recall and false-positive pressure. Since UAV-oriented vape imagery often contains small or tiny objects, this matters operationally: a detector that performs well on large examples may still struggle with the main deployment challenge.

One plausible mechanism is that external sources changed how the model represented small objects after resizing. Scraped images often differ in framing and object scale, while synthetic images were downscaled from much higher native resolutions. These differences may affect how small vape-like shapes appear after YOLO preprocessing. This should be treated as a plausible interpretation rather than a proven causal mechanism.

### Confidence and Threshold Behaviour

Confidence diagnostics showed that true positives generally had high median confidence, while false positives had very low median confidence. This indicates that threshold calibration can remove much low-confidence noise. However, high-confidence false positives remained possible on hard-negative images, including in the baseline condition. Table 12 summarises confidence separation.

| Exp | Pathway | Dose | TP median conf | FP median conf | FP p90 conf |
|---|---|---:|---:|---:|---:|
| E1 | baseline | 0 | 0.869 | 0.005 | 0.220 |
| E2 | scraped | 88 | 0.824 | 0.005 | 0.197 |
| E3 | scraped | 177 | 0.872 | 0.005 | 0.231 |
| E4 | scraped | 355 | 0.807 | 0.005 | 0.229 |
| E5 | pure_synthetic | 88 | 0.836 | 0.005 | 0.347 |
| E6 | pure_synthetic | 177 | 0.826 | 0.005 | 0.245 |
| E7 | pure_synthetic | 355 | 0.873 | 0.006 | 0.331 |

The threshold sweep then tested whether different operating points could overturn the main conclusion. Table 13 reports each experiment's best F1 threshold, and Figure 7 shows the full F1 curves.

| Exp | Pathway | Dose | Best threshold | P | R | F1 | FP predictions | FP / hard-neg image | FN |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|
| E1 | baseline | 0 | 0.40 | 0.725 | 0.741 | 0.733 | 38 | 0.35 | 35 |
| E2 | scraped | 88 | 0.40 | 0.740 | 0.719 | 0.729 | 34 | 0.24 | 38 |
| E3 | scraped | 177 | 0.50 | 0.758 | 0.719 | 0.738 | 31 | 0.27 | 38 |
| E4 | scraped | 355 | 0.30 | 0.667 | 0.711 | 0.688 | 48 | 0.46 | 39 |
| E5 | pure_synthetic | 88 | 0.60 | 0.707 | 0.696 | 0.701 | 39 | 0.43 | 41 |
| E6 | pure_synthetic | 177 | 0.50 | 0.740 | 0.674 | 0.705 | 32 | 0.29 | 44 |
| E7 | pure_synthetic | 355 | 0.50 | 0.701 | 0.696 | 0.699 | 40 | 0.43 | 41 |

![Threshold F1 curves](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_threshold_f1_curves.png)

**Figure 7.** F1 across confidence thresholds from 0.05 to 0.70. The figure tests whether external-data conditions can be rescued by threshold tuning.

Threshold tuning improved operating points for all models, but it did not overturn the main official-test conclusion. E3 could be tuned into a strong precision/F1 point, reinforcing the interpretation that medium-dose scraped data increased selectivity. However, neither scraped nor synthetic augmentation became a clear recall or mAP50-95 winner through threshold choice alone. The threshold diagnostics therefore explain operating behaviour but do not replace the official fixed-test comparison.

## Discussion

### Answer to the Evaluation Question

The final controlled source-dose ablation shows that external data was not plug-and-play for UAV vape-litter detection. The real target-domain baseline with hard negatives remained the strongest overall training recipe for official held-out recall and mAP50-95. Scraped and synthetic positives introduced useful appearance signal and changed model behaviour, but neither pathway improved the primary localisation-aware metric.

This finding is important because it separates dataset construction capability from downstream performance. The DCE made scraped and synthetic sources usable and auditable, but the model evaluation shows that external source availability alone does not guarantee target-domain improvement.

### Scraped Data Interpretation

Scraped data should not be described as useless. It produced the highest official precision condition, suggesting that it helped the detector learn target-class appearance and become more selective. However, the same pathway reduced recall and did not improve mAP50-95 over the baseline.

One plausible explanation is that scraped images added product-level appearance diversity while also introducing source-domain mismatch. Many scraped images differed from UAVape imagery in framing, background, object scale and native resolution. This may have encouraged a more conservative detector that was confident when it predicted a vape, but less able to detect the small or visually degraded vape instances in the real UAVape test set.

The most defensible conclusion is that scraped data may be useful for appearance learning or future filtering, but should not be blindly mixed into the primary UAV detector training pool.

### Synthetic Data Interpretation

Synthetic data showed a different pattern. Low-dose synthetic performed poorly, medium-dose synthetic recovered some recall and mAP50-95, and full-dose synthetic plateaued below the baseline. This suggests partial transfer rather than complete failure.

The synthetic pathway also carried a mechanical sim-to-real risk. The selected synthetic images were substantially higher resolution than the real UAVape images and were downscaled during YOLO training. This may have produced object boundary and texture cues that differed from real UAV sensor blur, noise and scale. The diagnostic result from E6 suggests that synthetic geometry may have helped reject lighter-like distractors, but that benefit was not enough to overcome the localisation gap on real held-out vapes.

Synthetic data therefore appears to provide some transferable signal, but not enough target-domain fidelity to outperform real target-domain training data in this configuration.

### Relationship to OOD and Sim-to-Real

The findings are consistent with known OOD and sim-to-real problems. The dissertation does not claim to discover domain shift. Instead, it demonstrates how domain shift appears in a specific UAV micro-litter detection setting under a controlled source-dose protocol.

Scraped data contributed appearance diversity but suffered from target-domain mismatch. Synthetic data partially transferred but plateaued below the real-data baseline. In both cases, the source-specific data was informative but not sufficient to improve the primary held-out localisation metric.

### Why the DCE Still Matters

The DCE is justified precisely because external sources did not automatically improve performance. Its contribution is not to guarantee that scraped or synthetic data helps. Its contribution is to construct, label, metadata-tag and export external sources in a controlled format so that they can be evaluated honestly by a separate model protocol.

Without this data-construction layer, the external images would be difficult to audit and compare. With it, the final ablation could identify how each constructed source changed precision, recall, mAP, hard-negative behaviour, threshold behaviour and validation-test transfer. The negative or mixed result is therefore still valuable: it shows that the DCE can reveal when a source should not be trusted as direct detector-training data.

### Limitations

The conclusions are bounded by the fixed UAVape test set, the YOLO26n detector, one random seed and the selected scraped and synthetic source pools. The hard-negative subset is important but narrow, as it focuses mainly on lighter-like distractors. Therefore, the results should be described as evidence of lighter-like hard-negative robustness, not universal robustness to all possible non-vape litter classes.

The artifact-physics explanations are plausible interpretations rather than proven causal mechanisms. Native resolution, resizing, visual style and source context may have contributed to the observed tradeoffs, but additional controlled experiments would be needed to isolate each factor causally.

The threshold diagnostics are also post-hoc. They explain operating behaviour, but they do not replace the official fixed-test comparison and were not used to tune official test metrics.

### Future Work

Future work should repeat the most informative conditions across additional seeds and test whether the source-dose conclusions hold under another lightweight detector variant such as YOLO26s. The hard-negative set should be expanded beyond lighter-like objects to include a wider range of visually similar litter and urban clutter.

Qualitative failure review should also be used to identify recurring false positives, missed vapes and poor-localisation cases. These examples can guide targeted hard-negative mining and improved synthetic generation.

Finally, the results suggest a possible two-stage cascade architecture. The real target-domain baseline could be used as a high-recall localiser, while a secondary classifier trained with scraped or synthetic appearance data could filter proposed crops for precision. This would use external data where it appeared strongest, as an appearance filter, rather than forcing it directly into the primary localisation model.

## Main Figure and Appendix Plan

The main dissertation should include the compact tables and final comparison figures used above. Full prediction-level tables, full image-level tables, full threshold sweeps, workbook sheets, per-run Ultralytics outputs and full diagnostic CSVs should be moved to the appendix. A qualitative detection montage would also be valuable if time allows, but the quantitative figure set is sufficient for the core argument.

## Addendum: Model, Input-Resolution and Follow-Up Benchmark Protocol

The V3 source-dose ablation was intentionally data-centric: the training data changed while the detector and training protocol were fixed. The YOLO26n and high-resolution follow-up results now show that detector capacity, model family and input resolution may themselves be limiting factors for UAVape. This addendum records the proposed benchmark structure before further training is interpreted, so that later choices are not made post-hoc.

This section is deliberately written as a technical memory-lock rather than final dissertation prose. It should later be reframed into a concise methodology and appendix subsection.

### Why Model Choice Is an Architectural Variable

YOLO versions should not be treated as cosmetic software updates. Each family changes aspects of the feature extractor, feature-fusion pathway, detection head, label assignment, loss design or post-processing. These choices can affect small-object learning, localisation, duplicate suppression and deployment behaviour.

| Term | Plain meaning | Relevance to UAVape |
|---|---|---|
| Backbone | The feature extractor that converts pixels into visual representations. | A stronger backbone may better separate small vape objects from pavement, leaves, wrappers, shadows and other clutter. |
| Neck | The feature-fusion stage that combines fine spatial detail with higher-level semantic context. | Important for small aerial objects because repeated downsampling can remove the limited visual signal available for vapes. |
| Detection head | The final prediction layers that output class confidence and bounding boxes. | A lighter head improves speed, but may change localisation and duplicate-detection behaviour. |
| NMS | Non-maximum suppression, a post-processing step that removes overlapping duplicate boxes. | Poor duplicate handling can either suppress true small detections or retain false overlapping predictions. |
| NMS-free / end-to-end detection | A detector trained to output final detections without a separate NMS step. | Simplifies deployment and may change duplicate handling, but should be validated on UAVape rather than assumed beneficial. |
| DFL | Distribution Focal Loss, a box-regression approach used by many modern YOLO detectors. | Can support precise localisation, but adds detection-head complexity. YOLO26 removes DFL to simplify the head. |
| STAL | Small-Target-Aware Label Assignment in YOLO26. | Directly relevant because it is intended to improve positive assignment coverage for small objects during training. |
| Attention | A mechanism that helps a model weigh important spatial regions or feature channels. | May help distinguish tiny vape objects from visually similar urban background clutter, but can increase memory and training instability. |
| P2 head | An extra high-resolution prediction layer at a shallower feature-map stride. | Common in small-object modifications because it lets the detector predict before the object is heavily downsampled. |

### Notes from the YOLO Evolution Review Paper

The arXiv review "Ultralytics YOLO Evolution: An Overview of YOLO26, YOLO11, YOLOv8, and YOLOv5 Object Detectors for Computer Vision and Pattern Recognition" is useful as a technical synthesis of how the Ultralytics-maintained YOLO line has changed. It should be treated as a review/synthesis source rather than the only authority, but it gives useful language for explaining why model choice is an architectural variable.

| Point from the paper | What it means for UAVape |
|---|---|
| YOLOv8 introduced a decoupled detection head and anchor-free prediction. | YOLOv8 remains a strong literature baseline. It is modern enough to be relevant, widely used in UAV/small-object work, and strong for real-time detection. |
| YOLO11 improves efficiency, small-object fidelity, feature selectivity and clutter handling. | This supports using YOLO11s/m as the stable modern baseline. It is not just "newer than YOLOv8"; it has architecture/training changes relevant to cluttered small-object scenes. |
| YOLO26 removes DFL, uses NMS-free/end-to-end inference, ProgLoss, STAL and MuSGD. | This directly supports testing YOLO26 because STAL and deployment simplification are relevant to tiny UAVape objects and edge/near-edge use. |
| YOLO26 is framed as deployment-first and quantization/export friendly. | Useful if the dissertation later justifies final deployment on edge devices or lightweight UAV inspection workflows. |
| Larger models improve mAP but cost latency. | Supports S/M/L benchmarking rather than assuming Nano is sufficient. |
| Dense scenes and domain adaptation remain open challenges. | This maps directly onto UAVape: tiny vape litter, hard negatives, synthetic/scraped source mismatch, and high-resolution versus old-source domain shift. |
| Hybrid CNN-transformer designs are treated as a future direction. | Supports keeping RT-DETR as an optional comparator rather than the core dissertation path. |

| Family | Main architecture idea | Expected UAVape behaviour | Proposed role |
|---|---|---|---|
| YOLOv8 | Anchor-free predictions, decoupled classification/regression head, C2f-style efficient backbone/neck, conventional NMS-based decoding. | Strong and familiar baseline; likely robust, but less current than YOLO11/YOLO26 at matched scale. | Literature anchor. |
| YOLO11 | More efficient feature blocks such as C3k2, C2PSA spatial-attention style feature selectivity, improved efficiency and small-object/clutter handling, still NMS-based. | Likely stronger than YOLOv8 at similar scale; stable modern baseline with better scrutiny than YOLO26. | Main stable comparator. |
| YOLO26 | DFL removed, NMS-free/end-to-end inference, Progressive Loss, STAL small-target-aware assignment and MuSGD optimizer. | Potentially best fit for tiny objects and edge deployment, but newer and less independently tested. | Main latest-model candidate. |

The review reinforces the corrected logic for the follow-up study: model family, model scale, image size and source data should be separated experimentally. A newer YOLO version should not be assumed superior without validation on UAVape, because architectural improvements may interact with object scale, hard negatives, resizing and source-domain mismatch.

### Candidate Detector Families

The follow-up benchmark should focus on a small set of defensible candidates rather than a large uncontrolled model leaderboard. YOLO26 and YOLO11 provide the cleanest modern comparison because both are supported in the Ultralytics ecosystem and both provide matched small, medium and large scales. YOLOv8 should remain as a literature anchor because many recent UAV and small-object papers still use or modify it. RT-DETR should remain optional because it is a different architecture family rather than a direct YOLO update.

| Candidate | Status | Strength | Risk | Proposed role |
|---|---|---|---|---|
| YOLO26s | Latest Ultralytics family with official weights and a 2026 technical paper. | NMS-free inference, lighter detection head, updated training recipe and small-target-aware assignment. | Less independent benchmark history than YOLO11 because it is newer. | Primary modern small detector. |
| YOLO26m | Same family with greater capacity. | Tests whether Nano/small capacity is limiting UAVape performance. | Slower and more memory-intensive. | Primary medium-capacity test. |
| YOLO26l | Same family, large scale. | Provides a high-capacity CNN ceiling without jumping to transformer detectors. | More expensive than S/M, but still much lighter than RT-DETR-R50/R101. | Optional CNN ceiling test. |
| YOLO11s | Stable modern Ultralytics detector. | Mature baseline, broadly scrutinised, similar scale to YOLO26s. | Does not include YOLO26-specific training and head changes. | Matched small-family comparator. |
| YOLO11m | Stable medium-scale comparator. | Controls for the possibility that YOLO26m wins only because it is larger than YOLO11s. | More compute than YOLO11s. | Matched medium-family comparator. |
| YOLO11l | Stable large-scale comparator. | Provides a high-capacity modern CNN baseline with more benchmark familiarity than YOLO26l. | More expensive and still not edge-light. | Optional large CNN comparator, especially if RT-DETR is tested. |
| YOLOv8s / YOLOv8m | Older but still widely used Ultralytics family. | Strong literature bridge because many UAV/small-object studies use YOLOv8 variants. | Less current than YOLO11/YOLO26. | Literature-anchor baseline, not main final candidate. |
| RT-DETR-L / RT-DETR-R50-style | Real-time Detection Transformer, supported by Ultralytics but not a YOLO model. | Provides a non-CNN/transformer-style comparison with stronger global-context modelling. | Heavier and less directly aligned with edge deployment. | Optional architecture-family comparator after YOLO benchmarks. |
| YOLO12s | Attention-centric YOLO family. | Interesting research comparator. | Less central to the official Ultralytics benchmark path than YOLO11/YOLO26 and potentially more memory-heavy. | Appendix-only comparator if time permits. |

### Matched Model-Scale Comparison

The fair comparison is not simply "YOLO26 versus YOLO11"; it is YOLO26 and YOLO11 compared at matched scales, with YOLOv8 used only as a literature reference point. If RT-DETR-L is included, a large YOLO CNN should also be included so that the transformer is not compared only against smaller CNNs.

| Pair | Approximate scale | Why it is fair |
|---|---:|---|
| YOLO26s vs YOLO11s | About 9.5M parameters each | Tests family differences while holding model capacity approximately constant. |
| YOLO26m vs YOLO11m | About 20M parameters each | Tests whether the family effect persists at a higher-capacity scale. |
| YOLO26s vs YOLO26m | Small versus medium within YOLO26 | Tests whether increased capacity improves UAVape detection. |
| YOLO11s vs YOLO11m | Small versus medium within YOLO11 | Tests whether capacity effects are consistent across a stable family. |
| YOLO26l or YOLO11l vs RT-DETR-L/R50 | Large CNN versus transformer-style detector | Controls for model capacity before attributing any performance difference to transformer architecture. |
| YOLOv8s/m vs YOLO11/26 equivalent | Older literature anchor versus modern family | Shows whether adopting newer families is justified beyond novelty. |

| Model | Official / reported parameters | Official / reported FLOPs at 640 | Role |
|---|---:|---:|---|
| YOLO26n | 2.4M | 5.4B | Edge/Nano baseline; useful for deployment but potentially capacity-limited. |
| YOLO26s | 9.5M | 20.7B | Main lightweight-capable modern candidate. |
| YOLO26m | 20.4M | 68.2B | Main higher-capacity modern candidate. |
| YOLO26l | 24.8M | 86.4B | Large CNN ceiling candidate. |
| YOLO11s | 9.4M | 21.5B | Stable modern small comparator. |
| YOLO11m | 20.1M | 68.0B | Stable modern medium comparator. |
| YOLO11l | 25.3M | 86.9B | Stable large CNN ceiling candidate. |
| YOLOv8s | 11.2M | 28.6B | Literature-anchor small model. |
| YOLOv8m | 25.9M | 78.9B | Literature-anchor medium model; close in parameters to YOLO11l/YOLO26l. |
| YOLOv8l | 43.7M | 165.2B | Heavier older large model; less attractive as a main candidate. |
| RT-DETR-R50 / L-style | About 42M | About 136B | Optional transformer-style comparator; heavier than YOLO11l/YOLO26l. |
| RT-DETR-R101 / X-style | About 76M | About 259B | Extra-large transformer-style comparator; likely outside dissertation scope. |

This means YOLO11l and YOLO26l are not automatically too large. They are larger than S/M but still much lighter than RT-DETR-R50/R101-style detectors. Therefore, if the study later includes RT-DETR-L, it is methodologically cleaner to also include one large YOLO CNN comparator.

### Why YOLOv8 Should Not Be Forgotten

YOLOv8 is older than YOLO11 and YOLO26, but it remains valuable because adjacent UAV and small-object studies still use it as a baseline or as the base for modified architectures. Excluding it entirely would make the study harder to relate to nearby literature. Including one YOLOv8 scale as a literature anchor lets the dissertation say that modern YOLO11/YOLO26 choices were tested against a still-common UAV baseline rather than assumed superior.

| YOLOv8 role | Justification |
|---|---|
| Literature bridge | Recent UAV/small-object papers still evaluate or modify YOLOv8 variants. |
| Baseline continuity | Helps connect UAVape results to existing small-object and aerial detection studies. |
| Not the main final model | YOLO11 and YOLO26 are newer official Ultralytics families with stronger current accuracy-efficiency positioning. |
| Recommended inclusion | YOLOv8s if compute is tight; YOLOv8m if matching medium-capacity comparisons is more important. |

### Input-Resolution Rationale

The earlier V3 runs used `imgsz=960`, but this should now be treated only as an inherited candidate setting, not as a proven optimum. The strongest protocol is to start with the standard 640 benchmark point, then iterate through 960 and 1280 for each model being seriously tested. This prevents the dissertation from claiming that 960 was justified merely because it sits between low and high resolution.

| Input size | Interpretation | Why it must be tested |
|---:|---|---|
| 640 | Standard YOLO benchmark resolution and lowest compute point. | Provides comparability with official model tables and many UAV/YOLO baselines. It also tests whether Nano/S/M models behave best at their common benchmark scale. |
| 960 | Intermediate inherited V3 operating point. | Previous old-source V3-E1 did well at 960, so it is a plausible candidate, but not a theoretical optimum. It must compete against 640 and 1280. |
| 1280 | Higher-resolution small-object setting. | Tests whether small vape objects benefit from additional spatial detail enough to justify higher compute, memory pressure and slower training/inference. |

| Evidence from adjacent literature or current results | Input size used or compared | Relevance |
|---|---:|---|
| Standard YOLO and many UAV/aerial baselines | 640 | Provides a common benchmark resolution and lower compute cost. |
| RTUAV-YOLO UAV study | Compared 640 versus 1280 | Reported that 1280 improved accuracy modestly but substantially increased training time and reduced throughput, justifying 640 for edge deployment. |
| Systematic YOLOv8 evaluation on VisDrone | Compared 640 versus 1280 | Found that input-resolution scaling can materially improve small-object detection, while model size and memory constraints can create instability. |
| Beach litter YOLOv5 study | Used very large input size with YOLOv5l6 on A100-class compute | Shows that UAV litter work may use large inputs when small-object fidelity is prioritised over lightweight deployment. |
| UAVape V3 old-source E1 result | Old 640-source E1 performed better at 960 than at 640 | Supports 960 as a candidate, but does not prove it is optimal for all model families or high-resolution source imagery. |
| UAVape high-resolution HR-E1 result | High-resolution source imagery at 960 performed poorly on official test | Shows that higher native image quality plus intermediate resizing is not automatically better; image size and source distribution must be benchmarked. |

Dissertation-facing wording should therefore be:

> Because input resolution materially affects small-object detection, a preliminary resolution benchmark was performed at 640, 960 and 1280. These settings reflect standard YOLO benchmarking, the inherited V3 operating point and a higher-resolution small-object setting respectively. The selected resolution was then fixed for subsequent model-family and data-source ablations.

The dissertation should not say that 960 was selected because it is "in the middle". A better claim is that 960 was an inherited operating point from the first V3 protocol and is now being explicitly benchmarked against 640 and 1280.

### Split Protocol and Validation/Test Discipline

The split itself should be treated as part of the benchmark protocol. A naive random split is not enough because UAVape contains positives, hard negatives, scale bands, possible near-duplicate capture sequences and distribution differences between train/validation/test pools.

| Split principle | Implementation | Reason |
|---|---|---|
| Fixed held-out test set | Test images are never used for model choice, hyperparameter choice, threshold choice or image-size choice. | Preserves the official final estimate of generalisation. |
| Validation for selection | Validation is used to select `best.pt`, image size, model family and limited hyperparameters. | Keeps model selection separate from final test reporting. |
| Stratify by target presence | Maintain positive and hard-negative proportions across splits. | Prevents validation/test from becoming accidentally easier or harder. |
| Stratify positives by scale band where feasible | Preserve tiny, small, medium and large object representation. | Important because tiny vape detection is likely the hardest regime. |
| Group near-duplicates when identifiable | Keep related captures or repeated filenames in the same split. | Prevents leakage from visually similar images crossing train/test boundaries. |
| Preserve exact manifests | Save split CSVs with image path, source pool, label path, hard-negative flag, scale band, boxes and split. | Makes the dissertation reproducible and auditable. |
| Reuse the same split for all model comparisons | Model and image-size candidates see the same train/val/test definitions. | Ensures differences are caused by model/resolution rather than split randomness. |

The current old V3 manifest remains useful because it creates exact comparability with the earlier V3-E1 baseline. However, if a new final benchmark split is constructed, it should be documented as a fixed stratified manifest and then reused across all candidate models and image sizes.

### Data Collection Protocol Clarification and Next Collection Plan

The current dissertation data-collection write-up is conceptually correct but should not claim that the staged capture protocol produced `2,490` usable real-world images. The station/scene/frame structure describes the intended capture design, not the final accepted machine-learning dataset. The arithmetic in the draft is also internally inconsistent: `50 stations x 3 scenes x 5 frames` would equal `750` candidate frames, not `2,490`.

The audited accepted real-world UAVape pools used in V3 were:

| Pool | Accepted images | Boxes | Role |
|---|---:|---:|---|
| `positive_train` | 377 | 509 | Real positive train/validation source pool |
| `hard_negative_train` | 125 | 0 | Real hard-negative train/validation source pool |
| `positive_test` | 80 | 135 | Real positive held-out test pool |
| `hard_negative_test` | 63 | 0 | Real hard-negative held-out test pool |
| **Total real accepted** | **645** | **644** | **Real target-domain data** |

The high-resolution mapping audit confirmed `645` curated real images and found no missing original-image mappings. The V3 image-format audit rejected one unsupported/corrupt input, but this was in the scraped-positive pool, not the real staged pools. Therefore, the corrected dissertation wording should say that the station/scene/frame protocol generated raw candidate captures, which were then curated, labelled, quality-checked and split into an accepted real-world dataset of 645 images containing 644 vape bounding boxes and 188 hard-negative images.

Recommended corrected wording for the data-collection method:

> Data collection was organised using three practical units: station, scene and frame. A station represented a physical outdoor ground patch, a scene represented a staged object arrangement at that station, and a frame represented an accepted image sampled from the capture process. The station-scene-frame structure was used as a capture guide rather than as a final dataset count. After curation, labelling and quality control, the accepted real-world UAVape dataset contained 645 real images: 457 positive images with 644 vape bounding boxes and 188 hard-negative images with no vape annotations.

The staged daylight method remains suitable for the next data collection phase. The next phase should not change the overall capture logic; it should strengthen the positive evidence and hard-negative coverage while preserving split discipline.

| Collection element | Keep / change | Rationale |
|---|---|---|
| Outdoor staged daylight capture | Keep | Maintains comparability with the current UAVape domain. |
| iPhone low-altitude sensor proxy | Keep if UAV deployment remains impractical | Reproduces key visual conditions: downward/oblique viewpoint, small target scale and outdoor ground clutter. |
| Stations as physical ground patches | Keep and record explicitly | Enables group-safe train/val/test splitting and reduces spatial leakage. |
| Scenes as object arrangements | Keep and record explicitly | Allows isolated positives, hard negatives and clusters to be tracked. |
| Frames as accepted still images | Keep, but do not overclaim raw frame counts | Final dataset size should be the accepted labelled image count after QC. |
| Positive inventory rotation | Keep and expand if possible | Reduces overfitting to a small number of vape colours, brands and form factors. |
| Hard-negative inventory | Keep and expand beyond lighters where possible | Tests false-positive resistance against visually similar urban litter. |
| Surfaces | Expand where possible | Grass, pavement, gravel, soil, leaves, curb edges, wet/dry surfaces and shadowed surfaces reduce background shortcut learning. |
| Scene grouping metadata | Strengthen | Images from the same station/session/object arrangement must remain in the same split. |

For the next collection pass, the immediate priority is daylight expansion rather than nighttime expansion. A reasonable target is to approximately double the positive real-world evidence while also adding hard negatives. The goal is not merely to increase total images, but to increase accepted positive instances, hard-negative diversity and scene-group diversity.

| New data target | Suggested priority |
|---|---|
| Additional daylight positive images | Highest priority |
| Additional vape boxes / varied object scales | Highest priority |
| Additional daylight hard negatives | High priority |
| More surface types and clutter conditions | High priority |
| More scene groups rather than many near-duplicates from one scene | High priority |
| Nighttime data | Future work, not part of the immediate collection plan |

Nighttime collection remains valuable future work because it would test illumination-domain robustness, glare, ISO noise, artificial lighting, shadows and wet-ground reflections. However, due to time constraints and to keep the immediate methodology simpler, the next collection pass should remain daylight-only. The future-work note should be retained in the addendum but does not need to be foregrounded in the current dissertation methodology unless the dataset is later expanded into a day/night benchmark.

For the expanded daylight dataset, every accepted image should receive metadata columns before splitting:

| Metadata field | Purpose |
|---|---|
| `scene_group` | Prevent train/val/test leakage from related frames. |
| `station_id` | Tracks physical ground patch and surface. |
| `capture_session` | Keeps same-session near-duplicates together. |
| `scene_type` | Isolated vape, isolated hard negative, mixed cluster, cluttered positive, cluttered hard negative. |
| `lighting` | For now mostly daylight; can later support dusk/night future work. |
| `surface_type` | Grass, pavement, gravel, soil, leaves, curb edge, mixed. |
| `inventory_ids` | Optional only; useful for stricter inventory control but not required for the immediate collection pass. |
| `height_angle_band` | Approximate low/high and top-down/oblique capture condition. |
| `boxes` | Number of vape annotations. |

The final split should be built from scene groups, not individual images. If ten frames come from the same station and object arrangement, all ten must go to train, validation or test together. This matters more than achieving a perfectly exact percentage split, because leakage from near-duplicate scenes would inflate validation and test performance.

For the immediate daylight expansion pass, a practical field pattern is sufficient:

| Unit | Practical definition |
|---|---|
| Station | One outdoor surface patch, such as a curb, pavement area, grass patch or gravel area. |
| Scene A | One isolated vape, photographed from a few angles/heights. |
| Scene B | One isolated lighter or hard negative, moved slightly within the same station so the background is similar but not identical. |
| Scene C | A mixed cluster of vapes and/or hard negatives, again slightly repositioned within the station and photographed from a few angles/heights. |

Each station-scene combination should become its own `scene_group`, for example `S001_A`, `S001_B`, `S001_C`, `S002_A`, `S002_B`, `S002_C`. The exact vape/lighter inventory IDs do not need to be tracked for this pass, because doing so would slow collection. Instead, the minimum required metadata are `scene_group`, `scene_type`, `surface_type` and `height_angle_band`. Reusing vapes or lighters across stations is acceptable as long as surfaces, positions, angles and clusters vary.

### V4 Class-Balance and Hard-Negative Rationale

The V4 dataset design should separate three concepts that were partially collapsed in V3: vapes as the primary target class, lighters as a visually similar secondary litter class, and benign non-target hard negatives as background examples. In V3, lighters were treated as hard negatives because the detector was single-class. If V4 introduces class-aware annotation, lighters should become a second labelled class rather than remaining blank background.

This changes the learning problem from one-class vape detection to two-class smoking-related micro-litter detection:

| Dataset component | V3 role | V4 role | Rationale |
|---|---|---|---|
| Vape | Positive class | Target class `vape` | Primary dissertation object and hazardous e-waste focus. |
| Lighter | Lighter-like hard negative | Target class `lighter` | Visually similar smoking-related litter with independent semantic value. |
| Sticks, leaves, wrappers, cables, pens, curb marks, bottle caps and other non-target clutter | Not systematically separated | Planned benign hard negatives / no-target images | Teaches the detector that visually confusing urban clutter is neither vape nor lighter. |
| False positives discovered after training | Not available before training | Hard-negative mining branch | Targeted repair loop after error analysis. |

The recommended V4 baseline should therefore include labelled vape boxes, labelled lighter boxes and no-target negative images before model-family or image-size selection begins. This is different from hard-negative mining. Planned benign negatives are part of the initial dataset specification; hard-negative mining is a later intervention that adds or upweights examples only after the trained detector reveals recurring false-positive failure modes.

Benchmark support for this balance is mixed but defensible:

| Evidence source | Relevant point | Implication for UAVape V4 |
|---|---|---|
| Ultralytics dataset-quality guidance | Background/no-object images are useful but should be monitored as a minority of the dataset; the Academy QC material flags background-image share around `0-10%` as a dataset statistic to check. | Use benign negative images deliberately, but do not let them dominate the target-object dataset. |
| Ultralytics / YOLO custom-data guidance | Detection datasets can include images with empty/no label files; YOLO learns from such no-object examples as background. | Benign hard negatives can be included as images with empty labels, not as a fake `background` class. |
| Foreground-background imbalance studies in object detection | Object detection naturally has strong foreground/background imbalance, especially for small infrequent objects; exact image-level class balance is not the only concern. | A 50/50 vape/lighter split is not required. Diversity and sufficient examples per class matter more than exact equality. |
| SSD / classical hard-negative mining literature | Hard-negative mining often controls negative-to-positive proposal/match ratios, commonly around 3:1, but this is anchor/proposal-level sampling rather than an instruction to make 75% of images negative. | Use the 3:1 idea only as context that negative evidence matters; do not directly convert it into an image-count rule. |

The practical V4 target should be:

| Component | Recommended target |
|---|---:|
| Vape boxes | `50-65%` of labelled target boxes |
| Lighter boxes | `35-50%` of labelled target boxes |
| Pure benign no-target negative images | `10-20%` of total images |
| Upper bound for benign negatives | About `25%` unless false positives dominate validation errors |

This does not require exact 50/50 class balance. A vape-to-lighter object ratio around `2:1` is defensible, and even `3:1` can be acceptable if lighter examples are diverse across surfaces, heights, angles and clutter. Ratios beyond about `4:1` should be avoided unless lighter is treated only as a side diagnostic class, because lighter AP may become unstable or under-supported.

For field collection, a practical rule is:

> For every 10 vape-containing scene groups, collect roughly 5-8 lighter-containing scene groups and 2-3 benign no-target hard-negative scene groups.

This keeps vapes central while giving lighters enough support to be evaluated as a real second class and preserving a smaller pool of true background negatives. The train/validation/test split should then stratify by scene group, target-class presence and no-target status so that no split becomes accidentally easier or harder.

If V4 adopts this two-class scheme, the Dataset Construction Engine must become class-aware. The annotation interface should provide an active class selector such as `vape` and `lighter`; accepted SAM2 masks/boxes should be saved with the selected class; overlays should display class-specific colours; COCO categories should contain both classes; and YOLO export should write multi-class labels and a dataset YAML such as:

```yaml
names:
  0: vape
  1: lighter
```

The engine should also support a lightweight custom-tag registry so extra target classes can be added during audit if needed, without changing code. In practice, this means an `Add tag` control in the annotation view writes a new COCO category, immediately exposes it in the active class selector, assigns it a display colour, and carries it through COCO and YOLO export. This should remain a practical annotation utility rather than a commitment to expand the dissertation scope; any added class must still be justified by sufficient examples and evaluation support before being treated as a formal experimental class.

The annotation store should treat COCO as the dataset source of truth. COCO records preserve the object `category_id`, bounding box and, where available, SAM2-derived segmentation mask. For the YOLO detection experiments, the training export uses the COCO `category_id` and bounding box to write YOLO-format labels; the segmentation mask is not used by standard YOLO detect training. Retaining masks is still useful methodologically because it makes the dataset reusable for future instance-segmentation models and allows other researchers to convert the same annotation source into COCO detection, COCO segmentation or YOLO detection formats.

Because V4 is no longer single-class, legacy count fields such as `num_vapes` should be replaced or supplemented by class-aware fields: `num_annotations`, `num_vapes` and `num_lighters`. This avoids confusion when a source image from the hard-negative pool contains labelled lighter objects. In the V4 interpretation, the source pool describes where the image came from, while the COCO category describes what object was annotated.

For the V4 audit pass, the old low-resolution COCO layer should remain preserved rather than overwritten. A migrated high-resolution COCO file, `data/annotations/coco_master_highres.json`, should be treated as the active editable annotation layer. This file is produced by mapping each same-name low-resolution curated image to its preserved high-resolution original and scaling COCO image dimensions, bounding boxes, segmentations and areas by the corresponding width and height factors. The Dataset Engine should display and annotate against the high-resolution originals so that manual corrections, SAM2 proposals and later YOLO exports all share one coordinate space. The legacy `coco_master.json` remains useful as an audit trail and rollback point.

### Validation Versus Test Selection Logic

The training process has two levels of selection that must not be confused.

Within a single YOLO run, the model trains on the training set and evaluates on the validation set after each epoch. These validation metrics do not update the weights directly; the weights are updated from training batches. Validation measures how the current epoch's weights perform on held-out validation data. At the end of training, YOLO saves `last.pt` from the final epoch and `best.pt` from the epoch with the strongest validation fitness. Ultralytics' default detection fitness is weighted mainly toward `mAP50-95`, with a smaller contribution from `mAP50`.

| Item | Meaning |
|---|---|
| Training set | Used to update model weights during each epoch. |
| Validation set | Used after epochs to estimate generalisation, select `best.pt`, compare candidate models/image sizes and guide limited hyperparameter choices. |
| Test set | Held back from all selection decisions and used only for final protocol evaluation. |
| `last.pt` | Weights from the final epoch. |
| `best.pt` | Weights from the epoch with the best validation fitness. |

Between candidate runs, each model/image-size configuration produces its own `best.pt`. The fair comparison is therefore between validation-selected checkpoints, not arbitrary final epochs.

| Candidate run | Checkpoint compared during model selection |
|---|---|
| YOLO26s at 640 | Its validation-selected `best.pt`. |
| YOLO26s at 960 | Its validation-selected `best.pt`. |
| YOLO26s at 1280 | Its validation-selected `best.pt`. |
| YOLO11s at 640 | Its validation-selected `best.pt`. |
| YOLO11s at 960 | Its validation-selected `best.pt`. |
| YOLO11s at 1280 | Its validation-selected `best.pt`. |

Thus, "choosing based on validation" means:

1. Train each candidate using the training split.
2. Let YOLO select `best.pt` using validation fitness within that run.
3. Compare candidate `best.pt` checkpoints using validation metrics and efficiency.
4. Choose the model family, scale and image size using validation evidence only.
5. Freeze that selected protocol.
6. Evaluate the frozen protocol on the held-out test set.

The test set should not be used to choose the best epoch, best image size, best model family, best confidence threshold or best hyperparameters. If the test set is repeatedly inspected and then used to change the protocol, it effectively becomes a second validation set and no longer provides an unbiased final estimate.

Dissertation-facing wording:

> For each candidate model and image size, training selected `best.pt` using validation-set fitness. Candidate configurations were compared using validation metrics and efficiency, while the held-out test set was reserved for final protocol evaluation after model family, image size and training settings were fixed.

### Metric and Hyperparameter Selection Logic

The new UAVape method should distinguish practical detection quality from strict localisation quality. In many detection papers, `AP` means average precision for one class or one evaluation setting, while `mAP` means mean average precision averaged across classes. For a single-class detector such as UAVape, AP and mAP can be numerically identical if they are computed at the same IoU threshold, because there is only one class to average over. The crucial difference is therefore usually the IoU threshold, not the letter `m`.

COCO-style evaluation often uses `AP` to mean AP averaged across IoU thresholds from 0.50 to 0.95. Ultralytics often reports the same idea as `mAP50-95`. This is stricter than `AP50`/`mAP50`, because it rewards very accurate box localisation. For UAVape, this strictness is valuable as a secondary localisation measure, but it may be too punitive as the sole primary metric because the target objects are very small in aerial imagery. A one- or two-pixel box shift can be operationally acceptable while being heavily penalised at high IoU thresholds such as 0.75-0.95.

The revised metric protocol is therefore:

| Metric | Use in protocol | Why it matters |
|---|---|---|
| `mAP50` / `AP50` | Primary practical detection metric for UAVape model comparison. | Matches many UAV/small-object papers and better reflects whether the model can locate tiny litter from aerial imagery. |
| `mAP50-95` / COCO-style `AP` | Secondary strict localisation metric. | Retains comparability with COCO/VisDrone-style evaluation and shows whether boxes are precisely localised, not just roughly detected. |
| `AP75` / `mAP75` where available | Middle-ground localisation guardrail. | Stricter than `AP50` but less punitive than averaging all the way to `0.95`. |
| Precision | Secondary metric and threshold-calibration target. | Measures false-positive burden, especially on hard negatives such as wrappers, leaves, pavement marks and cigarette-like litter. |
| Recall | Secondary metric and threshold-calibration target. | Measures missed vape objects, which is central for UAV inspection usefulness. |
| F1 or P/R threshold curve | Post-training operating-point analysis. | Helps choose a confidence threshold for a deployment-like use case without retraining weights. |
| Inference speed / memory | Efficiency constraint. | Keeps the study balanced with edge or near-edge deployment, even if the final dissertation is accuracy-led. |

This resolves the `mAP50` versus `mAP50-95` issue as follows: use `mAP50` as the primary practical UAVape comparison metric, report `mAP50-95` as the stricter localisation-aware secondary metric, and interpret both alongside precision, recall and subset diagnostics. If a model has slightly lower `mAP50-95` but clearly higher recall and acceptable hard-negative false positives, that trade-off should be discussed rather than automatically rejected.

Dissertation-facing wording:

> Because UAVape targets very small objects in aerial imagery, `mAP50`/`AP50` was treated as the primary practical detection metric, consistent with many UAV object-detection studies. Stricter COCO-style `mAP50-95`/`AP` and, where available, `AP75` were also reported to quantify localisation precision and retain comparability with stricter detection benchmarks.

Checkpoint selection should be distinguished from metric reporting. The default Ultralytics detection trainer saves `best.pt` using validation fitness rather than the final epoch. In many Ultralytics YOLO training reports, detection fitness is expressed as:

> Fitness = `(0.1 x mAP50) + (0.9 x mAP50-95)`

This is therefore still dominated by `mAP50-95`, while retaining a small contribution from `mAP50`. Some Ultralytics documentation/source versions describe detection fitness as directly equal to `mAP50-95`, so the exact formula should be recorded from the package version used in Colab. Either way, the checkpoint rule is localisation-heavy and stricter than choosing by `mAP50` alone. This is not unique to YOLO26; when YOLOv8, YOLO11 and YOLO26 are trained through the same Ultralytics pipeline, the same framework-level checkpoint rule applies to all of them.

For UAVape, the recommended protocol is to keep the standard Ultralytics `best.pt` rule for reproducibility and strict localisation guardrailing, then report and rank the selected checkpoints using the full metric panel with `mAP50`/`AP50` as the primary practical metric.

This avoids modifying the trainer differently across model families. It also means every candidate checkpoint has passed the stricter localisation-aware validation rule before being compared on the practical detection metric. During analysis, the `results.csv` file should still be audited to record both the epoch with best validation `mAP50` and the epoch with best validation `mAP50-95`. If those epochs diverge substantially for the final selected model, a sensitivity run can be performed with custom AP50-based checkpoint selection or epoch-wise checkpoint saving.

Adjacent papers are inconsistent in how explicitly they describe checkpoint selection. Some recent UAV/YOLO benchmark work does specify validation-based checkpoint selection, including a UAV thermal bird benchmark that selected best variants using validation `mAP@50-95`, and a drone road-marking pipeline that selected the best checkpoint using validation F1 and `mAP50-95`. Many other papers report final `AP50`, `mAP50`, `AP75` and/or `mAP50-95` without stating whether the checkpoint was selected by AP50, AP50-95, F1, early stopping, final epoch or framework default. Therefore, UAVape should explicitly state its checkpoint rule rather than assuming adjacent papers used the same one.

Hyperparameter tuning should be split into two levels:

| HPO level | When to use | Scope | Why |
|---|---|---|---|
| Limited sanity tuning | After detector/image-size selection, before source-dose ablation. | A small set of training-recipe factors such as learning rate/optimizer, augmentation strength, batch/accumulation and close-mosaic schedule. | Prevents a clearly suboptimal default recipe from distorting the data-source ablation. |
| Comprehensive HPO | Only after the final detector, image size and data recipe are selected, or as a clearly labelled final optimisation branch. | Wider search using Ultralytics tuning/evolution or another documented HPO method. | Useful for a state-of-the-art final model, but too expensive and confounding to run separately for every model x image-size x source-dose condition. |

The reason not to run comprehensive HPO inside every candidate condition is methodological, not laziness. If YOLO26m at 1280 receives a broad tuning search while YOLO11m at 960 uses defaults, the comparison is no longer model/image-size versus model/image-size; it is also tuned versus untuned. The fairer approach is to use a common default recipe for broad screening, perform limited sanity tuning on the selected detector family/image size, freeze that recipe, and then apply it consistently during the source-dose ablation.

### Revised Benchmark Order

The next evaluation should separate modelling factors in stages. The key correction is that image size should be benchmarked before being locked. The protocol should not assume 960 as the fixed control.

The order below is based on the type of intervention being made:

| Intervention type | Does it change weights? | Does it change training data? | Does it change inference/reporting? | Consequence for ordering |
|---|---:|---:|---:|---|
| Model family, model scale and image size | Yes | No | Yes | Must be selected early because every later result depends on the detector and input scale. |
| Hyperparameter tuning | Yes | No | No | Should be done after selecting a detector/image-size family, then frozen before data-source comparisons. |
| Data-source ablation | Yes | Yes | No | Should be done under a fixed detector/training recipe if the research question is about source contribution. |
| Threshold calibration | No | No | Yes | Should come after the trained model/data recipe is selected because it only chooses an operating point. |
| Error analysis | No | No | No | Should follow validation/test reporting and guide targeted next interventions. |
| Hard-negative mining | Yes | Yes | No | Should follow error analysis, because mined negatives should come from observed false-positive failure modes. |
| Tiling / SAHI | Sometimes | Sometimes | Yes | Should be treated as a separate small-object preservation protocol, not as a minor setting. |
| Custom architecture or custom loss | Yes | No | Sometimes | Should be last or future work because it changes the detector design itself. |

| Stage | Fixed factors | Changed factor | Purpose |
|---|---|---|---|
| 0. Fixed split and metric lock | Fixed train/val/test manifest | None | Prevent split or metric drift before model selection begins. |
| 1. Resolution and model-family benchmark | Same fixed E1 split, same epochs, seed and default training recipe | `imgsz=640`, `960`, `1280` across YOLO26s/m, YOLO11s/m and YOLOv8 anchor where feasible | Establish whether model family, capacity or input scale is the main bottleneck. |
| 2. Optional large-CNN ceiling | Same split and strongest candidate input size(s) | YOLO26l or YOLO11l | Test whether large CNN capacity gives a meaningful gain before moving to transformer detectors. |
| 3. Optional transformer comparison | Same split and feasible input size | RT-DETR-L/R50-style detector | Test whether a non-YOLO detector changes the failure profile. |
| 4. Limited hyperparameter sanity tuning | Selected detector family/scale and image size | Learning rate/optimizer, augmentation strength, batch/accumulation and close-mosaic schedule | Remove obvious training-recipe bottlenecks before the data-source ablation is repeated. |
| 5. Freeze detector protocol | Selected detector, image size, epochs, seed, optimiser/augmentation recipe and split | None | Create the fixed detector protocol under which data-source claims can be tested fairly. |
| 6. Locked-protocol source-dose ablation | Frozen detector protocol | Data-source pathway and dose | Repeat the V3-style data-centric ablation under the stronger selected detector. |
| 7. Validation-selected data recipe | Frozen detector family and image size | Selected data pathway/dose based on validation metrics and efficiency | Choose the training-data recipe without using the test set as a selector. |
| 8. Final held-out test evaluation | Fully selected model, image size, training recipe and data recipe | None | Produce the main unbiased test result for the selected protocol. |
| 9. Threshold calibration | Selected trained weights | Confidence threshold and possibly NMS/IoU reporting threshold | Choose an operating point on validation and report its effect on P/R/F1/false positives; this does not improve weights. |
| 10. Error analysis | Selected predictions | Failure-mode labels: missed tiny objects, hard-negative false positives, poor localisation, duplicates | Decide which targeted intervention is actually justified. |
| 11. Hard-negative mining branch | Selected detector protocol and data recipe | Add or upweight hard negatives from observed false positives | Test whether robustness improves once failure modes are known. |
| 12. Tiling / SAHI branch | Selected detector protocol and data recipe | Sliced inference only, or sliced training plus sliced inference | Test whether small-object preservation improves recall enough to justify extra compute. |
| 13. Comprehensive final HPO branch | Selected detector, image size and data recipe, or selected SAHI/HNM branch | Wider tuning search | Optimise the final candidate only if compute/time justify it, using validation only. |
| 14. Optional custom architecture/loss | Only if standard modern detectors plateau | P2 head, attention, CARAFE, modified loss or custom small-object head | Investigate detector engineering as extended work or future work. |

This means data-source ablation is not universally the first thing after model selection. It is first in this dissertation's "part two" only if the central claim being tested is whether real, scraped, synthetic and hard-negative source mixtures improve UAVape under a stronger detector. If the research question changed to "how do we reduce false positives fastest?", hard-negative mining could be moved earlier. For the V3 dissertation logic, however, doing hard-negative mining before the source-dose ablation would alter the baseline data distribution and could mask the effect of the planned source mixtures.

Threshold calibration comes after model/data selection because it is not weight training. It changes the operating point of an already-trained detector. A lower confidence threshold usually increases recall and false positives; a higher threshold usually increases precision and misses more objects. Therefore, threshold sweeps should be reported using validation-selected thresholds and then applied once to the held-out test set. They should not be used to choose between model families, image sizes or source-dose conditions on the test set.

Hard-negative mining should follow error analysis because it is a targeted data repair loop. Classical hard-example mining in object detection was introduced to focus training on difficult examples rather than abundant easy examples. In UAVape, the practical equivalent would be to inspect false positives from validation or deployment-like imagery, add the confusing non-vape examples as hard negatives, retrain, and compare against the pre-mining baseline. This is a defensible improvement branch, but it is no longer a clean source-dose ablation unless it is applied consistently across all compared source conditions.

Tiling and SAHI are also separate from image-size selection. A full-frame `imgsz=640` run and a tiled run that feeds cropped regions at `imgsz=640` are not the same experiment: tiling preserves local detail by changing the field of view seen by the detector. SAHI was explicitly proposed as a generic slicing-aided inference and fine-tuning framework for small-object detection, including aerial datasets such as VisDrone and xView. ASAHI is a newer adaptive extension for high-resolution aerial/satellite imagery that treats slicing, sliced fine-tuning and duplicate-merging as an additional protocol layer. Therefore, tiling is best treated as an optional small-object preservation branch after the baseline detector/data recipe is understood, not as a replacement for the 640/960/1280 benchmark.

Hyperparameter tuning must be handled carefully. A limited tuning pass can be performed after detector/image-size selection and before source-dose ablation if the tuned recipe is then frozen for every source-dose condition. Comprehensive tuning should not be repeated independently for every source-dose condition, because that would confound the data-source comparison with different training recipes. If a comprehensive HPO phase is later used to maximise the final model, it should happen after the final model/data protocol is chosen and should still use validation only.

The data-source ablation should not be delayed until after custom architecture experiments. Custom architecture is a separate research axis. For a clean dissertation claim, the detector family/scale and input size should be selected first, the training recipe should be locked, and then the data-source ablation should be rerun under that fixed protocol. Architecture modification should only follow if standard modern detectors, careful resolution choice, data ablation, threshold calibration, hard-negative mining and/or tiling do not reach an acceptable performance envelope.

Dissertation-facing wording:

> The follow-up methodology separated interventions by whether they changed the detector, the training distribution or the inference operating point. Model family, model scale and input resolution were benchmarked first because they define the detector itself. A limited hyperparameter sanity pass was then frozen before rerunning the source-dose ablation, so that data-source effects were not confounded by different training recipes. Post-training threshold calibration, error analysis, hard-negative mining and tiling were treated as subsequent improvement branches rather than as model-selection criteria.

### Proposed Immediate Experiment Matrix

The immediate matrix should begin with resolution sweeps. If compute is limited, run the full 640/960/1280 sweep on YOLO26s and YOLO11s first, then repeat on M-scale only if S-scale results are inconclusive or if the selected resolution changes the model ranking.

| Run | Model | Input size | Dataset protocol | Purpose |
|---|---|---:|---|---|
| R1 | YOLO26s | 640 | Fixed V3-E1 split | Latest small model at standard benchmark resolution. |
| R2 | YOLO26s | 960 | Fixed V3-E1 split | Latest small model at inherited V3 operating point. |
| R3 | YOLO26s | 1280 | Fixed V3-E1 split | Latest small model at higher-resolution small-object setting. |
| R4 | YOLO11s | 640 | Fixed V3-E1 split | Stable small comparator at standard benchmark resolution. |
| R5 | YOLO11s | 960 | Fixed V3-E1 split | Stable small comparator at inherited V3 operating point. |
| R6 | YOLO11s | 1280 | Fixed V3-E1 split | Stable small comparator at higher-resolution small-object setting. |
| R7 | YOLOv8s or YOLOv8m | 640 | Fixed V3-E1 split | Literature-anchor baseline at standard YOLO resolution. |
| R8 | YOLOv8s or YOLOv8m | 960 or selected candidate | Fixed V3-E1 split | Tests whether YOLOv8 remains competitive at the inherited or selected setting. |

After the initial resolution sweep:

| Run | Model | Input size | Dataset protocol | Purpose |
|---|---|---:|---|---|
| M1 | YOLO26s | Selected from 640/960/1280 | Fixed V3-E1 split | Latest small-model benchmark. |
| M2 | YOLO11s | Selected from 640/960/1280 | Fixed V3-E1 split | Stable small-model comparator. |
| M3 | YOLO26m | Selected from 640/960/1280 | Fixed V3-E1 split | Latest medium-capacity benchmark. |
| M4 | YOLO11m | Selected from 640/960/1280 | Fixed V3-E1 split | Stable medium-capacity comparator. |
| M5 | YOLOv8s or YOLOv8m | Selected from 640/960/1280 | Fixed V3-E1 split | Literature-anchor comparison. |
| M6 | YOLO26l or YOLO11l | Selected from 640/960/1280 | Fixed V3-E1 split | Large CNN ceiling, especially if RT-DETR is later tested. |

Optional architecture-family comparator:

| Run | Model | Input size | Dataset protocol | Purpose |
|---|---|---:|---|---|
| T1 | RT-DETR-L / R50-style | Selected or feasible input size | Fixed V3-E1 split | Non-CNN transformer-style comparator. |

### RT-DETR Clarification

RT-DETR is not a YOLO model. It is a real-time Detection Transformer. Ultralytics supports RT-DETR through the same Python ecosystem, but `RT-DETR-L` is not the same as `YOLO11l` or `YOLO26l`.

| Model name | Family | Meaning of "L" |
|---|---|---|
| YOLO11l / YOLO26l | YOLO CNN-style detector family | Large YOLO scale. |
| RT-DETR-L | Real-time Detection Transformer | Large RT-DETR scale. |
| RT-DETR-X | Real-time Detection Transformer | Extra-large RT-DETR scale. |

This makes RT-DETR useful as a non-YOLO comparator, especially because UAV/litter review literature identifies transformer-based detectors as promising for small, clustered or occluded objects. It should not replace the primary YOLO comparison unless it clearly outperforms the selected YOLO model under the same evaluation protocol.

If RT-DETR-L is tested, at least one large YOLO CNN should be tested too. Otherwise, any performance difference could reflect model capacity rather than transformer architecture.

### Custom Architecture Interpretation

Custom architecture work should be treated as a later research direction unless the standard modern detectors fail to reach acceptable performance. Adjacent cigarette, smoking and UAV litter studies often modify YOLO-like detectors by adding small-object layers, attention modules, improved upsampling or alternative localisation losses.

| Modification type | What it changes | Why it may help small objects |
|---|---|---|
| P2 detection head | Adds an extra high-resolution detection layer. | Allows very small objects to be predicted before feature maps are heavily downsampled. |
| Attention module | Reweights spatial regions or feature channels. | Helps the detector focus on weak vape/cigarette evidence in cluttered backgrounds. |
| CARAFE or improved upsampling | Reconstructs higher-resolution feature maps more intelligently than nearest-neighbour upsampling. | Preserves fine details that can be lost during feature fusion. |
| NWD, EIoU, WIoU or related losses | Changes how bounding-box localisation error is penalised. | Small boxes are highly sensitive to a one- or two-pixel error, so standard IoU can be unstable. |
| Backbone replacement | Swaps the feature extractor. | Can improve fine-grained representation or global-context modelling. |
| Multi-scale fusion | Improves how shallow detail and deeper semantic features are combined. | Helps objects that are visually small but semantically ambiguous. |

Examples from adjacent literature include YOLOv8-MNC for smoking behaviour detection, which adds a small-target layer, NWD loss, multi-head self-attention and CARAFE upsampling; SCS-YOLO for cigarette appearance defects, which modifies YOLOv7-tiny with SPD-Conv, CBAM attention, SCConv and EIoU; and WasteNet, which is not a YOLO detector but a multi-scale attention U-Net-style segmentation architecture for UAV waste localisation.

### Source Notes for Formal Referencing

The tables above are based on the current Ultralytics documentation for YOLO26, YOLO11, YOLOv8, YOLO12 and RT-DETR; the YOLO26 arXiv paper; the RT-DETR model card and paper; object-detection metric conventions; hard-example mining literature; SAHI/sliced-inference literature; and adjacent UAV/litter/small-object studies including RTUAV-YOLO, the systematic YOLOv8 VisDrone evaluation, beach litter YOLOv5 work, YOLOv8-MNC smoking detection and SCS-YOLO cigarette defect detection. Full formal citations should be inserted during final dissertation reference formatting.

Key source URLs to formalise later:

| Source | URL |
|---|---|
| Ultralytics YOLO26 documentation | https://docs.ultralytics.com/models/yolo26 |
| Ultralytics YOLO11 documentation | https://docs.ultralytics.com/models/yolo11 |
| Ultralytics YOLOv8 documentation | https://docs.ultralytics.com/models/yolov8 |
| Ultralytics RT-DETR documentation | https://docs.ultralytics.com/models/rtdetr |
| YOLO26 arXiv paper | https://arxiv.org/abs/2606.03748 |
| RT-DETR model card | https://huggingface.co/PekingU/rtdetr_r101vd |
| RT-DETR paper | https://arxiv.org/abs/2304.08069 |
| COCO detection evaluation conventions | https://cocodataset.org/#detection-eval |
| VisDrone object-detection evaluation protocol | https://aiskyeye.com/evaluate/object-detection-2022/ |
| VisDrone-DET2021 challenge metrics | https://openaccess.thecvf.com/content/ICCV2021W/VisDrone/papers/Cao_VisDrone-DET2021_The_Vision_Meets_Drone_Object_Detection_Challenge_Results_ICCVW_2021_paper.pdf |
| TPH-YOLOv5++ drone detection metrics | https://www.mdpi.com/2072-4292/15/6/1687 |
| DroneScan-YOLO small-UAV-object metrics | https://arxiv.org/abs/2604.13278 |
| UAV thermal bird YOLOv8-YOLOv26 benchmark with validation mAP50-95 selection | https://www.sciencedirect.com/science/article/pii/S235293852600114X |
| Drone road-marking pipeline with validation F1/mAP50-95 checkpoint selection | https://www.preprints.org/manuscript/202511.2296/v1 |
| Ultralytics YOLO hyperparameter tuning guide | https://docs.ultralytics.com/guides/hyperparameter-tuning |
| Online Hard Example Mining for object detection | https://arxiv.org/abs/1604.03540 |
| SAHI: Slicing Aided Hyper Inference and Fine-tuning | https://arxiv.org/abs/2202.06934 |
| ASAHI adaptive slicing for high-resolution imagery | https://arxiv.org/abs/2604.19233 |
