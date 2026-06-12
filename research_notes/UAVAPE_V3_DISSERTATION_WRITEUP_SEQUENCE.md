# UAVape V3 Dissertation Write-Up Sequence

This document is the dissertation-facing order for writing V3. It follows the V2 style: introduce the method choice, show the table or figure, then explain what the reader should learn from it. The aim is not to show every artifact. The aim is to show the smallest set of tables and figures that proves the workflow and supports the interpretation.

## Dissertation Through-Line

The V3 experiment should be written as a data-centric question, not as a model leaderboard. The wider project built a Dataset Construction Engine (DCE) that can ingest multiple image sources, support annotation, attach metadata and export YOLO-ready datasets. V3 asks what happens when that construction capability is used to create controlled training mixtures for a real UAV vape-litter detector.

The central question is:

> When a YOLO26n detector is trained for UAV vape-litter detection, do additional scraped or synthetic vape-positive images improve held-out real target-domain performance compared with a real-data baseline containing hard negatives?

This question matters because the DCE makes it technically easy to add more data, but more data is not automatically better. Scraped images may add real-world product appearance but may not match the UAV camera viewpoint, scale or ground context. Synthetic images may add controlled variation but may not transfer cleanly to real images. V3 therefore tests whether these DCE-constructed sources actually help the downstream detector, and if they do not, how they change detector behaviour.

The dissertation narrative should move in this order:

1. The DCE creates auditable source-labelled datasets, but it does not train the model.
2. The V3 ablation uses those exported datasets to test source and dose under a fixed YOLO26n protocol.
3. The official test metrics answer the main performance question.
4. The source-dose and delta figures explain how scraped and synthetic additions changed precision, recall and mAP.
5. The validation-test, hard-negative, scale-band and threshold diagnostics explain why the headline result happened.
6. The discussion returns to the wider claim: the DCE is valuable because it makes this kind of controlled source evaluation possible, even when external sources do not automatically improve final detection performance.

Written this way, the figures are not separate outputs. They are evidence in a chain: construction capability, controlled ablation, official outcome, behavioural explanation and bounded conclusion.

## Research Aim And Objectives

Aim:

> To evaluate whether DCE-constructed scraped and synthetic vape-positive datasets improve YOLO26n detection of real UAV vape-litter images, and to characterise any source-specific performance tradeoffs.

Objectives:

1. Construct source-separated YOLO datasets from real UAVape positives, hard negatives, scraped vape positives and synthetic vape positives.
2. Hold validation and test sets fixed so that all experiments are evaluated on the same real target-domain problem.
3. Train a real hard-negative baseline and matched source-dose scraped and synthetic variants.
4. Compare official fixed-test precision, recall, mAP50 and mAP50-95.
5. Analyse whether any performance changes are due to source type, source dose, threshold behaviour, hard-negative false positives or object-scale effects.
6. Interpret the findings in relation to OOD transfer and sim-to-real behaviour without claiming that the experiment discovers these phenomena universally.

## Removable Planning Note: Pilot Refinement And Final Protocol Rationale

This section is a planning insert. Keep it while drafting the dissertation, then decide whether to integrate it into the Method chapter or remove it if it makes the final write-up too complex.

What you are trying to say:

1. The final YOLO26n source-dose study did not appear from nowhere.
2. Earlier pilot/smoke-test runs showed that a broad "add all external data" ablation was too blunt.
3. The pilot work also exposed the need for fixed, compositionally consistent validation and test sets.
4. The final study therefore moved toward a cleaner source-dose design: same real foundation, same hard-negative foundation, same validation/test sets, matched external doses and source auditing.
5. The point is not to report the pilot metrics as main evidence, but to explain why the final method became more controlled.

Dissertation-safe framing:

> Preliminary pilot runs were used to validate the training, backup and reporting pipeline and to refine the experimental protocol. These pilot runs indicated that broad external-data addition was insufficient for interpreting source effects, because aggregate performance alone could not reveal whether changes were caused by source type, source dose, validation/test composition or threshold behaviour. The final protocol therefore used fixed real validation and test sets, matched positive-to-hard-negative proportions, a fixed YOLO26n detector and matched source-dose additions for scraped and synthetic data.

How this relates to the earlier V1/V2 idea:

- Do not foreground "V1", "V2" and "V3" as dissertation labels unless needed in an appendix.
- In the main dissertation, describe them as **pilot protocol refinement** or **preliminary ablation runs**.
- The older YOLOv10n runs can be mentioned only as pipeline smoke tests, not mixed into the official YOLO26n result tables.
- The final official experiment should be described as the **final controlled source-dose ablation**, not simply "V3".

Why the validation/test stratification matters:

- Detection performance can change if the evaluation set has a different balance of positive images and hard-negative no-vape images.
- Positive images test whether the detector can find real vape instances.
- Hard-negative images test whether the detector hallucinates vapes on visually similar non-vape objects, especially lighter-like distractors.
- The final validation and test sets therefore used the same positive-to-hard-negative proportion: 57 positive and 45 hard-negative images in validation, 80 positive and 63 hard-negative images in test. Both are 55.9% positive and 44.1% hard-negative.
- Validation remained for checkpoint selection only; the fixed test set remained for final held-out reporting.

Why the training-source sampling matters:

- The final ablation did not simply dump all scraped and synthetic data into training.
- Scraped and synthetic additions were tested at matched doses of 88, 177 and 355 positive images.
- Sampling used fixed seed 42 and nested subsets where possible, so the smaller dose was contained inside the larger dose.
- Sampling was stratified by `scale_band`, meaning the selected add-on subsets were chosen to keep object-scale composition approximately stable within each pathway.
- This makes within-pathway dose comparisons more meaningful: E2-E4 test increasing scraped dose, and E5-E7 test increasing synthetic dose.
- Matched-dose comparisons then become possible: E2 vs E5, E3 vs E6 and E4 vs E7.

Important accuracy wording:

- It is fair to say the study **controlled source dose and approximately stabilised object-scale composition within each pathway using stratified nested sampling by scale band**.
- It is also fair to say the study **audited native image dimensions, source composition, boxes and scale-band composition**.
- Be careful with "perfectly matched". A safer dissertation phrase is **matched or approximately controlled where possible, audited where full control was not possible**.
- Native image resolution was audited rather than fully equalised: scraped subsets had broadly similar native-resolution profiles across scraped doses, while synthetic images were much higher resolution and downscaled during YOLO training.

Where to integrate this later:

1. Add a short paragraph at the start of Chapter 3 Method: "Pilot Refinement of the Final Protocol".
2. Mention that pilot runs motivated fixed validation/test composition and matched source-dose sampling.
3. Keep the old detector/pilot details in one sentence or footnote, not as a major results section.
4. In the Results chapter, avoid discussing pilot metrics unless they are explicitly labelled preliminary.
5. In Discussion, use this as a strength: the final study is not just an end-to-end run, but a refined data-centric ablation designed to isolate source effects.

## Removable Planning Note: Final Critical Review Checklist

This section aggregates the internal review and external feedback into one actionable checklist. It should guide the final dissertation rewrite, but it does not all need to appear as a separate section in the submitted dissertation.

### Already Covered Well

- **Research question:** The current plan clearly asks whether DCE-constructed scraped and synthetic positives improve YOLO26n performance over a real + hard-negative baseline.
- **DCE scope:** The DCE is correctly framed as data construction/export, not model training or evaluation.
- **Reason for the experiment:** The write-up explains that the DCE makes external data construction possible, but the ablation tests whether that constructed data actually helps.
- **Pilot refinement:** The planning note explains that earlier pilot/smoke-test runs motivated a more controlled final protocol.
- **Hard negatives:** The write-up explains that hard negatives test vape hallucination on lighter-like distractors, not just positive detection.
- **Fixed validation/test design:** The validation/test sets are fixed, real target-domain only and compositionally matched in positive/hard-negative proportion.
- **Matched source-dose design:** The 88, 177 and 355 scraped/synthetic additions are clearly explained.
- **Training controls:** The plan identifies the fixed detector, seed, image size, epochs, real foundation, hard-negative foundation and fixed validation/test sets.
- **Scale-band stratification:** The planning note explains that object-scale composition was approximately stabilised within each pathway using `scale_band` stratified nested sampling.
- **Audited versus controlled:** The write-up correctly distinguishes between controlled/matched source-dose and scale-band sampling versus audited native resolution/source characteristics.
- **Results order:** The results move from official metrics to source-dose curves, precision/recall tradeoffs, deltas, validation-test gaps and diagnostics.
- **Discussion link:** The discussion returns to OOD, sim-to-real and why the DCE still matters.

### Add Or Strengthen In The Final Rewrite

1. **Rename internal experiment language.**

   In the final dissertation prose, avoid relying on "V3" as if the examiner knows the internal project versioning. Use phrases such as **the final controlled source-dose ablation**, **the YOLO26n source-dose study** or **the final ablation protocol**. Keep V1/V2/V3-style labels only in planning notes, appendices or file names.

2. **Add metric primacy.**

   Add a concise metric rationale explaining that `mAP50-95` is the primary metric because it is stricter and localisation-aware. Precision and recall remain important behavioural diagnostics, but they are not the sole success condition. Suggested wording:

   > The primary comparison uses mAP50-95 because it averages performance across stricter IoU thresholds and therefore better reflects localisation quality than mAP50 alone. Precision and recall are interpreted as supporting behaviour: precision indicates false-positive pressure, while recall indicates missed vape instances.

3. **Add a no-test-tuning clause.**

   Make explicit that the held-out test set was not used for checkpoint selection, model selection or threshold selection in the official metric comparison. Suggested wording:

   > The held-out test set was evaluated only after the validation-selected `best.pt` checkpoint was fixed. Confidence-threshold diagnostics were generated post hoc to study operating behaviour and were not used to choose the official checkpoint or revise the official test metrics.

4. **State expected source hypotheses before results.**

   Add a short method paragraph before the results:

   > Scraped images were expected to provide real-world product appearance diversity, but risked OOD mismatch because their viewpoint, background and object scale differ from UAV ground-facing imagery. Synthetic images were expected to provide controlled generated variation, but risked sim-to-real mismatch because generated object boundaries, texture and resolution may not match real UAV sensor imagery.

5. **Repeat hard-negative scope in limitations.**

   The hard negatives are primarily lighter-like distractors, so the final dissertation should not claim universal false-positive robustness. Suggested wording:

   > The hard-negative subset tests robustness to lighter-like distractors, not all possible non-vape litter classes.

6. **Add why YOLO26n/nano matters.**

   The fixed detector is already covered, but final method prose should briefly connect YOLO26n to the lightweight UAV/edge framing:

   > YOLO26n was selected as a fixed nano-scale detector because the study focuses on data construction rather than architecture search, and because a lightweight model better matches the UAV/near-edge deployment motivation.

7. **Clarify threshold diagnostics are post hoc.**

   The threshold curves are useful, but should be described as an operating-point analysis, not as a replacement for official metrics:

   > Threshold tuning explains how each trained checkpoint behaves under different confidence cutoffs; it does not overturn the official fixed-test comparison.

8. **Keep the main chapter compact.**

   The full prediction-level tables, full threshold sweep, workbook sheets and per-run Ultralytics plots should be appendix material. The main results should keep the compact summary tables and final comparison figures.

### Add Carefully As Hypotheses, Not Proven Causes

These are useful ML interpretation points, but they should be phrased cautiously. The data supports them as plausible mechanisms, not proven causal explanations.

1. **Artifact physics: scraped upscaling and product-shot mismatch.**

   The native-resolution audit showed that many scraped images had max side below the 960-pixel training size and were visually heterogeneous. This may have contributed to the scraped pathway's precision-oriented behaviour. Suggested cautious wording:

   > One plausible explanation is that scraped images added product-appearance detail while also introducing source artifacts from heterogeneous framing and resizing. This may have made the detector more selective, improving precision at medium dose, while reducing recall on small, real UAVape targets.

2. **Artifact physics: synthetic downscaling and sim-to-real mismatch.**

   Synthetic images were much higher resolution and downscaled during YOLO training. Suggested cautious wording:

   > The synthetic pathway also carried a mechanical domain-shift risk: selected synthetic images were substantially higher resolution than the real UAVape imagery and were downscaled during training. This may have produced object boundary and texture cues that did not match the blur, noise and scale characteristics of real UAV imagery.

3. **Small-vape connection.**

   Connect scale-band results to UAV small-object detection, but avoid inventing exact pixel sizes unless measured:

   > The scale-band diagnostics suggest that the external sources did not uniformly improve the small-object problem. Since UAV vape detection is dominated by small and tiny objects, any mismatch in how small objects appear after resizing can affect recall even when large or medium objects remain easy to detect.

4. **E6 hard-negative behaviour.**

   V3-E6 had the lowest low-threshold hard-negative FP pressure, but the explanation is interpretive:

   > A possible explanation is that medium-dose synthetic data provided clearer vape-shape variation, helping the detector separate vape-like geometry from lighter-like distractors. However, this did not translate into the best overall mAP50-95, suggesting that improved distractor rejection alone was not enough to overcome sim-to-real localisation limits.

5. **Applied engineering solution: two-stage cascade.**

   This is useful for future work, not a claim about completed experiments:

   > Future work could test a two-stage cascade in which the target-domain baseline detector is used as a high-recall localiser, while a secondary classifier trained with scraped or synthetic appearance data filters proposed crops for precision. This would use external data where it appeared strongest, as an appearance filter, rather than forcing it directly into the primary localisation model.

### Optional Visual Addition

The current final figure set contains quantitative charts and summary tables. A qualitative montage is not strictly required for the core argument, but it would strengthen a computer-vision dissertation if time allows.

Suggested optional figure:

- a side-by-side qualitative detection montage using the same real test image or a small set of representative test images;
- compare the baseline, full scraped and full synthetic checkpoints;
- show at least one true positive, one missed small vape or poor localisation case, and one hard-negative false positive or correct rejection;
- keep full qualitative galleries in the appendix.

If time is tight, the dissertation can proceed with the current 8 final figures. If a montage is easy to generate from saved predictions, add it as an extra qualitative figure rather than replacing any existing chart.

### Critical Risk To Manage

The main risk is overloading the Method chapter. The dissertation has many true details: DCE, pilot refinement, fixed splits, source-dose sampling, scale-band stratification, native-resolution audit and YOLO26n restart. Stage them in this order:

1. DCE output and source pools.
2. Pilot refinement in one short paragraph.
3. Final controlled source-dose ablation design.
4. Fixed validation/test protocol.
5. Sampling and audit controls.
6. YOLO26n training/evaluation.

This keeps the method readable while preserving the important controls.

## Removable Planning Note: Reader-Friendly Rewrite Requirements

This section defines how the final dissertation prose should be written. The goal is not to make the section shorter. The goal is to make it easier for a new examiner to follow without assuming they already understand the pipeline.

### Overall Thesis Link

The overall dissertation research question is:

> How can an integrated image-detection dataset construction engine, combined with deep-learning evaluation and workflow prototyping, overcome the structural barriers to identifying and operationalizing disposable vapes as hazardous micro-e-waste in UAV-oriented imagery for urban environments?

This YOLO26n source-dose study answers the **deep-learning evaluation** part of that wider question. The dataset sourcing and construction methodology sits earlier in the dissertation as a related but separate section. This model-evaluation section should therefore begin by making its role explicit:

> The previous section established how UAVape constructs source-labelled object-detection datasets. This section evaluates whether those constructed sources improve downstream UAV vape-litter detection when used to train a fixed lightweight detector.

### Section-Level Writing Pattern

Each major method/results block should follow this pattern:

1. **Why:** What barrier or uncertainty does this section address?
2. **What:** What was built, controlled or measured?
3. **How:** How was it implemented technically?
4. **Validation / result:** How do we know it worked, or what did it reveal?

Example:

- Why: External vape images are easy to collect, but may not transfer to UAV imagery.
- What: A matched source-dose ablation was designed.
- How: Scraped and synthetic positives were added at 88, 177 and 355 images while the real baseline, hard negatives, validation and test sets were fixed.
- Result: The official fixed test metrics showed that no external-data condition improved mAP50-95 over the real hard-negative baseline.

### Design Requirements For This Section

These design requirements should be stated before the detailed method. They are not the method itself. They define what the deep-learning evaluation component must achieve in order to answer the dissertation's wider research question. The method then explains how each requirement was implemented.

| Design requirement | Why it matters | How the final protocol satisfies it |
|---|---|---|
| DR1: Evaluate constructed data-source pathways | The DCE produces multiple possible training sources, but the dissertation must test whether those sources actually help detection. | The final ablation compares the real baseline, scraped-augmented training and synthetic-augmented training. |
| DR2: Identify whether external data improves real UAVape detection | The aim is not to improve scraped-domain or synthetic-domain performance, but real UAV-oriented vape detection. | All conditions are evaluated on fixed real UAVape validation/test sets. |
| DR3: Use a UAV-suitable lightweight detector | The model should fit the UAV/near-edge motivation and avoid turning the study into architecture search. | YOLO26n is fixed across all conditions as a nano-scale detector. |
| DR4: Use a localisation-aware primary metric | Vape litter recovery requires not just class recognition but accurate object localisation. | mAP50-95 is used as the primary validation checkpoint and official comparison metric. |
| DR5: Measure both detection success and false-positive risk | Operational vape detection must find vapes while avoiding hallucinations on similar non-vape objects. | Precision, recall, hard-negative false positives and confidence diagnostics are reported. |
| DR6: Compare source additions at controlled dose levels | A broad "more data" experiment cannot separate source effect from data quantity. | Scraped and synthetic sources are tested at matched 88, 177 and 355 positive-image doses. |
| DR7: Preserve fair real target-domain evaluation | Metric differences should not come from changing validation/test composition. | Validation and test splits are fixed and use matched positive-to-hard-negative proportions. |
| DR8: Explain model behaviour beyond headline mAP | A single aggregate metric cannot explain why a source helped, hurt or changed model behaviour. | Source-dose curves, validation-test gaps, scale-band analysis, hard-negative FP pressure and threshold diagnostics are used. |
| DR9: Produce reproducible and reusable evidence for a niche detection problem | UAV vape-litter detection is an under-documented hazardous micro-e-waste task; other researchers need traceable outputs to understand, reproduce and build on the finding. | The workflow saves source manifests, compact CSV summaries, prediction diagnostics, figures, workbooks, checkpoints and Drive backups. |

These DRs should be framed as requirements for the deep-learning evaluation component. The detailed method then shows the implementation: fixed splits, matched doses, scale-band sampling, source audits, YOLO26n training, checkpoint selection and post-hoc diagnostics.

### Reader Onboarding Rules

Do not assume the reader knows object-detection terminology. Briefly onboard these terms the first time they matter:

- **Hard negative:** a no-vape image containing visually similar distractors, used to test false-positive resistance.
- **mAP50:** approximate localisation metric at IoU 0.50.
- **mAP50-95:** stricter localisation-aware metric averaged across IoU thresholds; primary metric here.
- **Precision:** proportion of predicted vape detections that are correct.
- **Recall:** proportion of labelled vape instances detected.
- **OOD:** out-of-distribution mismatch between training-source images and target-domain UAVape imagery.
- **Sim-to-real gap:** mismatch between generated/synthetic imagery and real sensor imagery.
- **Source-dose ablation:** a controlled experiment that changes the source and amount of added training data while holding other variables fixed.

### Tone Requirements

The final write-up should be academic but pedagogical.

- Avoid compressed, documentation-like prose.
- Prefer short explanatory paragraphs over dense lists when writing the final dissertation.
- Introduce each figure by telling the reader what question it answers.
- After each figure, explain the pattern and then tie it back to the research question.
- Do not assume the examiner remembers the DCE details from earlier chapters; briefly re-anchor the section.
- Avoid overstating causal mechanisms. Use "suggests", "may indicate", "is consistent with" and "one plausible explanation is" when interpreting artifact physics.
- Keep the main prose focused on the argument and move raw/full diagnostic outputs to the appendix.

### Placement Reminder

The dataset sourcing methodology is related but separate. In the final dissertation:

1. Earlier DCE/source-construction section: explain upload, SAM2 annotation, metadata/VLM support, COCO storage and YOLO export.
2. This model-evaluation section: explain how those YOLO outputs are used in a controlled source-dose detector evaluation.
3. Discussion: join them back together by arguing that the DCE is valuable because it enables auditable downstream evaluation, not because every constructed data source automatically improves performance.

## Core Writing Rule

For each result:

1. Tell the reader what the figure/table is testing.
2. Show the figure/table.
3. Explain the main pattern in plain language.
4. State what the pattern means for the research question.
5. Avoid overclaiming beyond the fixed UAVape YOLO26n setup.

Example rhythm:

> Figure X compares official fixed-test performance across the seven V3 experiments. The figure shows that the real hard-negative baseline retained the highest localisation-aware mAP50-95, while external scraped and synthetic additions mainly changed precision and recall tradeoffs. This indicates that increasing positive training volume alone did not improve target-domain detector quality.

## Final Artifact Bundle

The final Colab report bundle has been unpacked locally here:

- [v3_yolo26n_reports_final](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final)
- [post-analysis workbook](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_post_analysis_workbook.xlsx)
- [final figures folder](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures)

Use these extracted files as the local source of truth for the V3 dissertation write-up. The Google Drive version is the Colab backup; this local folder is the copy used to write and inspect the chapter.

### Main Results Tables

| Dissertation table | Local artifact |
|---|---|
| Input pool summary | [v3_yolo26n_input_pool_summary.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_input_pool_summary.csv) |
| Native image dimension summary | [v3_yolo26n_native_image_dimension_summary.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_native_image_dimension_summary.csv) |
| Dataset composition by source | [v3_yolo26n_dataset_composition_by_source.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_dataset_composition_by_source.csv) |
| Training checkpoint summary | [v3_yolo26n_training_checkpoint_summary_incremental.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_training_checkpoint_summary_incremental.csv) |
| Official fixed-test metrics | [v3_yolo26n_official_test_metrics_incremental.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_official_test_metrics_incremental.csv) |
| Source-dose effect | [v3_yolo26n_source_dose_effect.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_source_dose_effect.csv) |
| Validation-test gap | [v3_yolo26n_validation_test_gap_summary.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_validation_test_gap_summary.csv) |
| Threshold diagnostics | [v3_yolo26n_threshold_diagnostics.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_threshold_diagnostics.csv) |
| Hard-negative false-positive summary | [v3_yolo26n_hard_negative_false_positive_summary.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_hard_negative_false_positive_summary.csv) |
| Scale-band performance | [v3_yolo26n_scale_band_performance.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_scale_band_performance.csv) |
| TP/FP confidence summary | [v3_yolo26n_tp_fp_confidence_summary.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_tp_fp_confidence_summary.csv) |

### Final Figure Inventory

| Figure | Local artifact | Dissertation section |
|---|---|---|
| Official fixed-test metric bars | [v3_yolo26n_official_test_metric_bars.png](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_official_test_metric_bars.png) | 4.3 |
| Source-dose mAP50-95 | [v3_yolo26n_source_dose_mAP50_95.png](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_source_dose_mAP50_95.png) | 4.4 |
| Source-dose precision | [v3_yolo26n_source_dose_precision.png](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_source_dose_precision.png) | 4.5 |
| Source-dose recall | [v3_yolo26n_source_dose_recall.png](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_source_dose_recall.png) | 4.5 |
| Delta against baseline | [v3_yolo26n_delta_vs_baseline.png](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_delta_vs_baseline.png) | 4.6 |
| Validation-test gap | [v3_yolo26n_validation_test_gap_map50_95.png](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_validation_test_gap_map50_95.png) | 4.7 |
| Hard-negative FP pressure | [v3_yolo26n_hard_negative_fp_pressure.png](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_hard_negative_fp_pressure.png) | 4.8 |
| Threshold F1 curves | [v3_yolo26n_threshold_f1_curves.png](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_threshold_f1_curves.png) | 4.11 |

## Chapter 3 Method: Dataset Construction And Ablation Protocol

**Method Rationale**

The method chapter should begin by separating two things that are easy for the reader to blur together. The DCE is the dataset-building system: it creates structured image and annotation outputs. The V3 Colab protocol is the evaluation system: it uses those outputs to train and test YOLO26n models. This separation matters because the research question is not whether the DCE itself trains a better detector. The question is whether datasets produced through the DCE, when organised by source and dose, improve or change detector performance.

The method therefore has to justify three design choices. First, the data sources are separated so scraped and synthetic images can be tested rather than hidden inside a mixed dataset. Second, validation and test sets are fixed and kept real target-domain only, so the evaluation remains about UAVape performance rather than source memorisation. Third, the detector and training settings are held constant, so the experiment is about dataset construction rather than architecture selection.

### 3.1 Dataset Construction Engine Overview

Reader question this section answers: What is the DCE, and where does its responsibility stop?

Draft text:

> The Dataset Construction Engine is the data-engineering component of UAVape. Its role is to ingest multiple image sources, support annotation of vape instances using SAM2-assisted labelling, attach image-level metadata using VLM support, store the resulting annotations in COCO format and export the final datasets to YOLO format. The DCE does not train the detector or perform the ablation study. Instead, it produces structured, source-labelled datasets that can later be consumed by the separate Colab training and evaluation protocol.

> UAV vape-litter detection suffers from target-domain scarcity: real low-altitude ground-facing vape imagery is limited, while external vape imagery is easier to obtain through scraping or synthesis. The DCE therefore provides a controlled construction pipeline for real target-domain positives, hard-negative non-vape distractors, scraped vape positives and synthetic vape positives. This source separation is important because the later V3 ablation can test each constructed source explicitly rather than training on an opaque mixed dataset.

Table: **Data Source Role Table**

| Source | Dissertation role | Expected benefit | Main risk |
|---|---|---|---|
| Real UAVape positives | Target-domain anchor | Learns real UAV/litter context | Limited volume |
| Hard negatives | False-positive resistance | Learns to reject vape-like distractors | Narrow negative class if only lighters |
| Scraped positives | Appearance diversity | Learns brands, colours, form factors | OOD geometry/context |
| Synthetic positives | Controlled generated variation | Tests generated source value | Sim-to-real/domain gap |

Draft text after table:

> The important methodological point is that the DCE constructs and exports these sources; it does not prove that they improve detection. Their value is tested later by the V3 YOLO26n ablation protocol.

### 3.2 Source Pools And Pre-Training Audit

Reader question this section answers: What raw material did the DCE construct and export for the V3 experiments?

Table: **Input Pool Summary**

Source artifact: [v3_yolo26n_input_pool_summary.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_input_pool_summary.csv)

| Pool | Images | Positive images | Hard negatives | Boxes |
|---|---:|---:|---:|---:|
| Positive train | 377 | 377 | 0 | 509 |
| Hard-negative train | 125 | 0 | 125 | 0 |
| Positive test | 80 | 80 | 0 | 135 |
| Hard-negative test | 63 | 0 | 63 | 0 |
| Scraped positive | 2,905 | 2,905 | 0 | 4,437 |
| Synthetic positive | 355 | 355 | 0 | 355 |

Draft text:

> Before training, each exported image pool was audited for accepted images, labels, boxes and native image dimensions. This audit checked that the DCE outputs were usable by the downstream YOLO training protocol and clarified the source-domain differences before modelling. The real UAVape images were mostly 640 by 480 or 480 by 640, while synthetic images were high-resolution and always downscaled to the 960-pixel training size. Scraped images were more heterogeneous, but the selected scraped doses had similar native-resolution profiles across E2-E4.

Table: **Native Image Dimension Summary**

Source artifact: [v3_yolo26n_native_image_dimension_summary.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_native_image_dimension_summary.csv)

| Pool | Images | Median W x H | Median max side | % max side > 960 |
|---|---:|---:|---:|---:|
| Positive train | 377 | 480 x 640 | 640 | 0.0 |
| Hard-negative train | 125 | 480 x 640 | 640 | 0.0 |
| Positive test | 80 | 640 x 480 | 640 | 0.0 |
| Hard-negative test | 63 | 640 x 480 | 640 | 0.0 |
| Scraped positive | 2,905 | 604 x 479 | 640 | 29.6 |
| Synthetic positive | 355 | 3668 x 2048 | 3668 | 100.0 |

Draft text after table:

> This matters because source differences are not just semantic. They also affect scale, resolution and visual composition. The synthetic pathway is therefore not simply "more vape images"; it is a downscaled generated source. The scraped pathway is not simply "real images"; it is a mixed external source whose visual framing differs from the UAVape target domain.

### 3.3 Fixed Validation And Test Protocol

Reader question this section answers: How was the evaluation kept fair across source conditions?

Table: **Fixed Split Composition**

| Split | Images | Positive images | Hard-negative images | Boxes |
|---|---:|---:|---:|---:|
| Validation | 102 | 57 | 45 | 63 |
| Test | 143 | 80 | 63 | 135 |

Draft text:

> The validation and test sets were fixed across all V3 experiments. Scraped and synthetic images were never introduced into validation or test. This means each experiment was judged against the same real target-domain problem: detect real vape instances while rejecting hard-negative non-vape images. Validation was used for checkpoint selection, while the fixed test set was used only for final evaluation of the selected checkpoint.

### 3.4 V3 Source-Dose Ablation Matrix

Reader question this section answers: Why is V3 a controlled ablation rather than a data-flooding experiment?

Table: **V3 Experiment Matrix**

| Experiment | Training recipe | Added external positives | Purpose |
|---|---|---:|---|
| V3-E1 | Real positives + hard negatives | 0 | Operational baseline |
| V3-E2 | E1 + scraped positives | 88 | Low scraped dose |
| V3-E3 | E1 + scraped positives | 177 | Medium scraped dose |
| V3-E4 | E1 + scraped positives | 355 | Full scraped dose |
| V3-E5 | E1 + synthetic positives | 88 | Low synthetic dose |
| V3-E6 | E1 + synthetic positives | 177 | Medium synthetic dose |
| V3-E7 | E1 + synthetic positives | 355 | Full synthetic dose |

Draft text:

> V3 replaced broad data flooding with matched source-dose ablation. The 88, 177 and 355 positive-image doses were used for both scraped and synthetic pathways. This design allows three comparisons: each source against the E1 baseline, dose-response within each pathway, and matched-dose comparison between scraped and synthetic data.

### 3.5 Model And Checkpoint Selection

Table: **Training Configuration**

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

Draft text:

> The detector was held constant so that the experiment tested dataset construction rather than model selection. The official checkpoint for each experiment was selected using validation mAP50-95. The held-out test set was then evaluated once using that selected checkpoint, preventing test metrics from influencing checkpoint selection.

Table: **Validation-Selected Checkpoints**

Source artifact: [v3_yolo26n_training_checkpoint_summary_incremental.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_training_checkpoint_summary_incremental.csv)

| Exp | Pathway | Dose | Best epoch | Val P | Val R | Val mAP50 | Val mAP50-95 |
|---|---|---:|---:|---:|---:|---:|---:|
| V3-E1 | baseline | 0 | 86 | 0.755 | 0.937 | 0.836 | 0.733 |
| V3-E2 | scraped | 88 | 70 | 0.813 | 0.905 | 0.861 | 0.764 |
| V3-E3 | scraped | 177 | 77 | 0.779 | 0.921 | 0.851 | 0.749 |
| V3-E4 | scraped | 355 | 75 | 0.834 | 0.810 | 0.858 | 0.746 |
| V3-E5 | pure_synthetic | 88 | 76 | 0.733 | 0.873 | 0.845 | 0.746 |
| V3-E6 | pure_synthetic | 177 | 94 | 0.762 | 0.873 | 0.795 | 0.699 |
| V3-E7 | pure_synthetic | 355 | 90 | 0.809 | 0.857 | 0.831 | 0.742 |

## Chapter 4 Results: Dataset Characterisation

**Results Rationale: Dataset First**

Before presenting model performance, the dissertation should show what actually changed between experiments. This prevents the results from looking like seven arbitrary YOLO runs. The first results block therefore describes the constructed datasets: which images were available, what source each experiment added, and how the source pools differed before training. This lets the reader understand the later performance differences as consequences of source construction rather than unexplained model variation.

### 4.1 Training Composition Across V3

Reader question this section answers: What changed between experiments before any model metrics were produced?

Table: **Dataset Composition by Source**

Source artifact: [v3_yolo26n_dataset_composition_by_source.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_dataset_composition_by_source.csv)

| Exp | Pathway | Dose | Real positives | Hard negatives | Scraped positives | Synthetic positives | Train images | Train boxes |
|---|---|---:|---:|---:|---:|---:|---:|---:|
| V3-E1 | baseline | 0 | 320 | 80 | 0 | 0 | 400 | 446 |
| V3-E2 | scraped | 88 | 320 | 80 | 88 | 0 | 488 | 579 |
| V3-E3 | scraped | 177 | 320 | 80 | 177 | 0 | 577 | 731 |
| V3-E4 | scraped | 355 | 320 | 80 | 355 | 0 | 755 | 1,039 |
| V3-E5 | pure_synthetic | 88 | 320 | 80 | 0 | 88 | 488 | 534 |
| V3-E6 | pure_synthetic | 177 | 320 | 80 | 0 | 177 | 577 | 623 |
| V3-E7 | pure_synthetic | 355 | 320 | 80 | 0 | 355 | 755 | 801 |

Draft text:

> The training-set composition confirms that V3 changes only the training source mixture while preserving the same validation and test sets. V3-E1 contains only the real target-domain foundation: 320 real positives and 80 hard negatives. E2-E4 add scraped positives at increasing doses, while E5-E7 add synthetic positives at matched doses. This source accounting is necessary because any performance difference must be interpreted in relation to exactly which training source changed.

### 4.2 Source Characteristics And Expected Risk

Table: **Native Dimension / Source Audit**

Use the native-dimension table in Section 3.2 when writing the final dissertation. It is repeated conceptually here because it explains the results: the synthetic pool has much larger native dimensions and is always resized/downscaled for YOLO training, while the scraped pool is heterogeneous and only partially above the 960-pixel training scale.

Draft text:

> The source audit shows why direct external-data transfer is not guaranteed. Real target-domain images, scraped images and synthetic images have different native-resolution and visual-domain characteristics. These differences motivate the later interpretation through OOD and sim-to-real behaviour: the added images can teach vape appearance without necessarily matching the target-domain localisation problem.

## Chapter 4 Results: Model Performance

**Results Rationale: From Main Answer To Explanation**

The model performance section should answer the main question first, then explain the tradeoffs. The official fixed-test table and metric bars provide the headline answer: whether any scraped or synthetic condition beat the real hard-negative baseline. The source-dose and delta figures then explain the shape of that answer. They show whether external data was completely harmful, partially useful, or useful only for specific metrics such as precision. This ordering keeps the reader anchored: first the outcome, then the reason.

### 4.3 Official Fixed Test Metrics

Table: **Official Test Metrics**

Source artifact: [v3_yolo26n_official_test_metrics_incremental.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_official_test_metrics_incremental.csv)

| Exp | Pathway | Dose | P | R | mAP50 | mAP50-95 |
|---|---|---:|---:|---:|---:|---:|
| V3-E1 | baseline | 0 | 0.695 | 0.776 | 0.776 | 0.643 |
| V3-E2 | scraped | 88 | 0.755 | 0.756 | 0.752 | 0.601 |
| V3-E3 | scraped | 177 | 0.780 | 0.709 | 0.792 | 0.634 |
| V3-E4 | scraped | 355 | 0.699 | 0.667 | 0.735 | 0.594 |
| V3-E5 | pure_synthetic | 88 | 0.737 | 0.659 | 0.715 | 0.572 |
| V3-E6 | pure_synthetic | 177 | 0.721 | 0.728 | 0.731 | 0.604 |
| V3-E7 | pure_synthetic | 355 | 0.737 | 0.719 | 0.726 | 0.599 |

![Official fixed-test metric bars](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_official_test_metric_bars.png)

Caption:

> Figure X. Official fixed-test precision, recall, mAP50 and mAP50-95 for all V3 YOLO26n experiments. All results use the validation-selected `best.pt` checkpoint and the same fixed real test set.

Draft text:

> Figure X shows that V3-E1 remained the strongest overall condition on the official fixed test set. The baseline achieved mAP50-95 of 0.643398 and recall of 0.775934. No scraped or synthetic augmentation condition exceeded this mAP50-95 value. This means that the real target-domain training foundation with hard negatives was more effective than increasing positive-image volume through mismatched external sources.

### 4.4 Source-Dose Effect On mAP50-95

![Source-dose mAP50-95](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_source_dose_mAP50_95.png)

Caption:

> Figure X. Source-dose response for strict localisation-aware mAP50-95. The dashed baseline represents V3-E1, while scraped and synthetic lines show matched external positive doses.

Draft text:

> The source-dose mAP50-95 curve is the central V3 ablation result. Scraped data never surpassed the baseline. The medium scraped dose nearly recovered performance, but remained below V3-E1, while the full scraped dose degraded further. Synthetic data showed a different pattern: low-dose synthetic performed worst, medium-dose synthetic recovered, and full-dose synthetic plateaued. This indicates that scraped and synthetic data failed in different ways rather than producing a single generic "more data is worse" pattern.

### 4.5 Precision And Recall Dose Curves

![Source-dose precision](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_source_dose_precision.png)

![Source-dose recall](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_source_dose_recall.png)

Caption:

> Figure X. Source-dose precision and recall curves. These figures separate selectivity from target-instance coverage.

Draft text:

> The precision and recall curves explain why mAP50-95 alone is not enough. Scraped data produced a precision peak at V3-E3, where precision reached 0.779879. This suggests that scraped data provided useful target-appearance information and made the detector more selective. However, recall fell at the same time, indicating that the model missed more real target-domain vapes. Synthetic data produced a smaller precision gain and a medium-dose recall recovery from E5 to E6, but this recovery still did not exceed the baseline.

### 4.6 Delta Against Baseline

Table: **Metric Deltas Against V3-E1 Baseline**

Source artifact: [v3_yolo26n_source_dose_effect.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_source_dose_effect.csv)

| Exp | Pathway | Dose | P delta | R delta | mAP50 delta | mAP50-95 delta |
|---|---|---:|---:|---:|---:|---:|
| V3-E1 | baseline | 0 | 0.000 | 0.000 | 0.000 | 0.000 |
| V3-E2 | scraped | 88 | 0.060 | -0.020 | -0.024 | -0.042 |
| V3-E3 | scraped | 177 | 0.085 | -0.067 | 0.016 | -0.009 |
| V3-E4 | scraped | 355 | 0.004 | -0.109 | -0.041 | -0.050 |
| V3-E5 | pure_synthetic | 88 | 0.042 | -0.117 | -0.061 | -0.071 |
| V3-E6 | pure_synthetic | 177 | 0.026 | -0.048 | -0.045 | -0.040 |
| V3-E7 | pure_synthetic | 355 | 0.042 | -0.057 | -0.050 | -0.045 |

![Delta against baseline](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_delta_vs_baseline.png)

Caption:

> Figure X. Metric deltas relative to V3-E1. Positive bars indicate improvement over the real hard-negative baseline; negative bars indicate degradation.

Draft text:

> The delta chart makes the main result explicit. External data mostly improved precision but reduced recall and mAP50-95. V3-E3 produced the largest precision gain, while no external-data condition produced a positive mAP50-95 delta. This supports the conclusion that external positives were not useless, but their benefit was limited to selectivity and appearance confidence rather than overall target-domain localisation quality.

### 4.7 Validation-Test Gap

Table: **Validation-Test mAP50-95 Gap**

Source artifact: [v3_yolo26n_validation_test_gap_summary.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_validation_test_gap_summary.csv)

| Exp | Pathway | Val mAP50-95 | Test mAP50-95 | Test minus val |
|---|---|---:|---:|---:|
| V3-E1 | baseline | 0.733 | 0.643 | -0.090 |
| V3-E2 | scraped | 0.764 | 0.601 | -0.163 |
| V3-E3 | scraped | 0.749 | 0.634 | -0.115 |
| V3-E4 | scraped | 0.746 | 0.594 | -0.152 |
| V3-E5 | pure_synthetic | 0.746 | 0.572 | -0.173 |
| V3-E6 | pure_synthetic | 0.699 | 0.604 | -0.095 |
| V3-E7 | pure_synthetic | 0.742 | 0.599 | -0.143 |

![Validation-test gap mAP50-95](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_validation_test_gap_map50_95.png)

Caption:

> Figure X. Difference between validation mAP50-95 and official test mAP50-95. More negative values indicate weaker transfer from validation selection to held-out test performance.

Draft text:

> The validation-test gap analysis shows that validation metrics were sometimes optimistic. Several augmented conditions achieved competitive validation performance but dropped on the fixed held-out test set. This is especially important for E5 and E7, where validation mAP50-95 appeared close to or above the baseline but test mAP50-95 remained below it. The result justifies the protocol decision to treat validation as checkpoint-selection evidence and the fixed test set as the primary evaluation.

## Chapter 4 Results: Diagnostic Behaviour

**Diagnostic Rationale: Why The Headline Result Is Not Enough**

The official metrics show that external positives did not improve the primary held-out mAP50-95 result, but they do not fully explain how the models behaved. The diagnostic section therefore asks a second-layer question: if scraped and synthetic data did not win overall, what did they change? The low-threshold hard-negative analysis exposes distractor sensitivity. The scale-band table shows whether the issue is concentrated in object size. The confidence and threshold summaries show whether the models could be made more useful through operating-point calibration. These diagnostics turn the result from "external data did not help" into a more precise explanation of source-specific tradeoffs.

### 4.8 Prediction-Level Diagnostics And Hard Negatives

Table: **Low-Threshold Hard-Negative False-Positive Summary**

Source artifact: [v3_yolo26n_hard_negative_false_positive_summary.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_hard_negative_false_positive_summary.csv)

| Exp | Pathway | Dose | Hard-neg images | FP predictions | Images with FP | FP / image | Max FP conf |
|---|---|---:|---:|---:|---:|---:|---:|
| V3-E1 | baseline | 0 | 63 | 207 | 50 | 3.29 | 0.969 |
| V3-E2 | scraped | 88 | 63 | 235 | 55 | 3.73 | 0.920 |
| V3-E3 | scraped | 177 | 63 | 231 | 59 | 3.67 | 0.949 |
| V3-E4 | scraped | 355 | 63 | 267 | 61 | 4.24 | 0.941 |
| V3-E5 | pure_synthetic | 88 | 63 | 282 | 60 | 4.48 | 0.972 |
| V3-E6 | pure_synthetic | 177 | 63 | 182 | 55 | 2.89 | 0.980 |
| V3-E7 | pure_synthetic | 355 | 63 | 251 | 59 | 3.98 | 0.973 |

![Hard-negative false-positive pressure](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_hard_negative_fp_pressure.png)

Caption:

> Figure X. Low-threshold hard-negative false-positive pressure. Predictions were generated at `conf=0.001`, so the figure compares diagnostic sensitivity rather than deployment-threshold false-positive rates.

Draft text:

> The hard-negative diagnostic figure shows how each model behaves when nearly all weak candidate detections are exposed. V3-E6 produced the lowest low-threshold hard-negative FP pressure, while V3-E5 and V3-E7 produced more false-positive candidates. This suggests that medium-dose synthetic data may have improved some distractor rejection behaviour. However, because this diagnostic used `conf=0.001`, the values should not be interpreted as final deployment false-positive rates.

### 4.9 Scale-Band Performance

Table: **Scale-Band Recall Summary**

Source artifact: [v3_yolo26n_scale_band_performance.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_scale_band_performance.csv)

| Exp | Pathway | Dose | Large R | Medium R | Small R | Tiny R | Total FP |
|---|---|---:|---:|---:|---:|---:|---:|
| V3-E1 | baseline | 0 | 1.000 | 0.982 | 0.919 | 0.929 | 390 |
| V3-E2 | scraped | 88 | 1.000 | 0.982 | 0.952 | 0.929 | 353 |
| V3-E3 | scraped | 177 | 1.000 | 1.000 | 0.935 | 0.929 | 395 |
| V3-E4 | scraped | 355 | 1.000 | 1.000 | 0.935 | 0.857 | 365 |
| V3-E5 | pure_synthetic | 88 | 1.000 | 1.000 | 0.919 | 0.929 | 424 |
| V3-E6 | pure_synthetic | 177 | 1.000 | 1.000 | 0.855 | 0.929 | 317 |
| V3-E7 | pure_synthetic | 355 | 1.000 | 0.964 | 0.839 | 0.929 | 302 |

Draft text:

> Scale-band diagnostics show that all experiments remained strong on large and medium vapes under the low-threshold diagnostic pass. The most meaningful differences occurred in small-vape recall and in the number of extra false-positive predictions around positive images. The diagnostic does not overturn the aggregate mAP result, but it helps explain that the external-data pathways mainly changed selectivity and FP pressure rather than producing a uniform recall improvement across object scales.

### 4.10 TP/FP Confidence Separation

Table: **TP/FP Confidence Summary**

Source artifact: [v3_yolo26n_tp_fp_confidence_summary.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_tp_fp_confidence_summary.csv)

| Exp | Pathway | Dose | TP median conf | FP median conf | FP p90 conf |
|---|---|---:|---:|---:|---:|
| V3-E1 | baseline | 0 | 0.869 | 0.005 | 0.220 |
| V3-E2 | scraped | 88 | 0.824 | 0.005 | 0.197 |
| V3-E3 | scraped | 177 | 0.872 | 0.005 | 0.231 |
| V3-E4 | scraped | 355 | 0.807 | 0.005 | 0.229 |
| V3-E5 | pure_synthetic | 88 | 0.836 | 0.005 | 0.347 |
| V3-E6 | pure_synthetic | 177 | 0.826 | 0.005 | 0.245 |
| V3-E7 | pure_synthetic | 355 | 0.873 | 0.006 | 0.331 |

Draft text:

> TP/FP confidence summaries show that most false positives had very low median confidence, while true positives generally had high median confidence. This indicates that threshold calibration can remove much low-confidence noise. However, high maximum false-positive confidence on hard-negative images remained present across conditions, including the baseline. This motivates hard-negative mining and qualitative failure review before any deployment claim.

### 4.11 Threshold Diagnostics

Table: **Best F1 Operating Point Per Experiment**

Source artifact: [v3_yolo26n_threshold_diagnostics.csv](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/v3_yolo26n_threshold_diagnostics.csv)

| Exp | Pathway | Dose | Best threshold | P | R | F1 | FP predictions | FP / hard-neg image | FN |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|
| V3-E1 | baseline | 0 | 0.40 | 0.725 | 0.741 | 0.733 | 38 | 0.35 | 35 |
| V3-E2 | scraped | 88 | 0.40 | 0.740 | 0.719 | 0.729 | 34 | 0.24 | 38 |
| V3-E3 | scraped | 177 | 0.50 | 0.758 | 0.719 | 0.738 | 31 | 0.27 | 38 |
| V3-E4 | scraped | 355 | 0.30 | 0.667 | 0.711 | 0.688 | 48 | 0.46 | 39 |
| V3-E5 | pure_synthetic | 88 | 0.60 | 0.707 | 0.696 | 0.701 | 39 | 0.43 | 41 |
| V3-E6 | pure_synthetic | 177 | 0.50 | 0.740 | 0.674 | 0.705 | 32 | 0.29 | 44 |
| V3-E7 | pure_synthetic | 355 | 0.50 | 0.701 | 0.696 | 0.699 | 40 | 0.43 | 41 |

![Threshold F1 curves](/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/v3_yolo26n_reports_final/figures/v3_yolo26n_threshold_f1_curves.png)

Caption:

> Figure X. F1 across confidence thresholds from 0.05 to 0.70. The figure tests whether external-data conditions can be rescued by threshold tuning.

Draft text:

> Threshold tuning improved operating points for all models, but it did not overturn the main ablation conclusion. V3-E1 remained one of the strongest balanced options across common thresholds. V3-E3 could be tuned into a strong precision/F1 point, supporting the precision-peak interpretation of scraped data, but it did not become a clear recall or mAP winner. The synthetic pathway also did not produce a stronger threshold-calibrated operating point than the baseline.

## Chapter 5 Discussion

**Discussion Rationale**

The discussion should return from the figures to the thesis-level contribution. The result is not simply that scraped or synthetic data performed worse than the baseline. The stronger conclusion is that the DCE enabled a controlled test of external data sources, and that test showed source-specific OOD and sim-to-real behaviour in this UAV vape-litter setting. The DCE remains valuable because it makes source construction auditable and testable; the ablation shows that those constructed sources still need domain-aware evaluation before they are trusted for model improvement.

### 5.1 Answer To The Main Research Question

Draft text:

> The V3 results show that external data was not plug-and-play for UAV vape-litter detection. Real target-domain data with hard negatives remained the strongest overall training foundation. Scraped and synthetic positives introduced useful appearance signal and changed model behaviour, but neither pathway improved the primary held-out localisation metric, mAP50-95.

### 5.2 Scraped Data Interpretation

Draft text:

> Scraped data should not be described as useless. It produced the highest precision condition, indicating that it helped the detector learn target-class appearance. However, it did not improve strict localisation or recall on the fixed real UAVape test set. The most defensible conclusion is that scraped data is useful for appearance learning or future auxiliary filtering, but raw scraped positives should not be blindly mixed into the primary UAV detector training pool.

### 5.3 Synthetic Data Interpretation

Draft text:

> Synthetic data showed a different response curve. Low-dose synthetic performed poorly, medium-dose synthetic recovered, and full-dose synthetic plateaued. This indicates partial transfer rather than complete failure. However, the synthetic pathway still did not beat the real target-domain baseline, suggesting that generated images did not sufficiently match the localisation conditions of the real UAVape test set.

### 5.4 Why The DCE Still Matters

Draft text:

> The DCE is justified precisely because external sources did not automatically improve performance. Its contribution is not to guarantee that scraped or synthetic data helps. Its contribution is to construct, label, metadata-tag and export external sources in a controlled format so that they can be evaluated honestly by a separate model protocol. Without this data-construction layer, external images would be difficult to audit and compare. With it, the V3 ablation could identify how each constructed source changed precision, recall, mAP, hard-negative behaviour, threshold behaviour and validation-test transfer.

### 5.5 Relationship To OOD And Sim-To-Real

Draft text:

> These findings are consistent with known OOD and sim-to-real problems. The thesis does not claim to discover domain shift. Instead, it demonstrates how domain shift appears in a specific UAV micro-litter detection setting under a controlled YOLO26n source-dose protocol. Scraped images contributed appearance diversity but suffered from target-domain mismatch; synthetic images partially transferred but plateaued below the real-data baseline.

### 5.6 Limitations

Draft text:

> The conclusions are bounded by the fixed UAVape test set, the YOLO26n detector, one random seed, and the selected scraped and synthetic source pools. The hard-negative set is important but narrow, as it focuses heavily on lighter-like distractors. The threshold diagnostics are useful but do not replace deployment calibration on a separate validation protocol. Therefore, V3 supports a data-centric source-dose conclusion, not a universal claim that scraped or synthetic data cannot help object detection.

### 5.7 Future Work

Use existing future-work list:

1. Repeat the most informative conditions across seeds.
2. Test YOLO26s or another capacity variant.
3. Expand hard negatives beyond lighters.
4. Use qualitative failures for targeted hard-negative mining.
5. Generate or collect source data targeted to the observed failure modes.
6. Calibrate thresholds using a separate deployment validation set.
7. Test external sites, altitudes, cameras or collection days.

## Minimal Final Figure Set

Use these in the main dissertation:

| Order | Figure/Table | File/artifact | Why it is included |
|---:|---|---|---|
| 1 | Data source role table | manual table | Explains DCE logic |
| 2 | V3 experiment matrix | manual table | Explains source-dose design |
| 3 | Official test metrics table | `v3_yolo26n_official_test_metrics_incremental.csv` | Formal result |
| 4 | Official metric bars | `v3_yolo26n_official_test_metric_bars.png` | Visual comparison |
| 5 | Source-dose mAP50-95 | `v3_yolo26n_source_dose_mAP50_95.png` | Core ablation result |
| 6 | Precision and recall dose curves | `v3_yolo26n_source_dose_precision.png`, `v3_yolo26n_source_dose_recall.png` | Explains tradeoff |
| 7 | Delta vs baseline | `v3_yolo26n_delta_vs_baseline.png` | Shows help/hurt relative to E1 |
| 8 | Validation-test gap | `v3_yolo26n_validation_test_gap_map50_95.png` | Explains validation optimism |
| 9 | Hard-negative FP pressure | `v3_yolo26n_hard_negative_fp_pressure.png` | Explains distractor behaviour |
| 10 | Threshold F1 curves | `v3_yolo26n_threshold_f1_curves.png` | Shows threshold tuning does not overturn conclusion |

Move detailed prediction tables, scale-band table, full threshold table, TP/FP confidence table and workbook sheets to appendix unless space allows.
