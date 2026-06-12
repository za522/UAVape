# DCE Results Alignment Memory

Purpose: keep the DCE method/results structure stable while writing. This note separates what the DCE method is trying to test, what the results can honestly claim, and what comparisons are valid.

## 1. High-Level Method

The DCE method is not a detector-performance method by itself. It is a data-infrastructure method.

At a high level, the method is:

1. Build an integrated Dataset Construction Engine (DCE) that joins together data intake, review, annotation, metadata, synthetic generation, audit and export.
2. Test the DCE by using it to construct the UAVape dataset end to end.
3. Use the completed UAVape dataset in the separate model-evaluation workstream.

The DCE method therefore asks:

> Can the system take fragmented vape imagery and turn it into curated, annotated, metadata-enriched, model-ready datasets without relying on separate disconnected tools?

The DCE modules tested are:

- Multi-source intake: uploads/direct imports, scraped imagery and video/sourced candidates.
- Synthetic bootstrapping: generated vape composites, reviewed before training use.
- Human curation: review queue categories such as positive, hard negative, clean negative and reject.
- Human-audited SAM2 annotation: SAM2 proposes masks/boxes, but the human accepts, clears, edits or deletes them.
- VLM metadata assistance: GPT-4o-mini suggests image-level metadata fields for failure-mode analysis.
- COCO/YOLO export: COCO is the source of truth; YOLO is the model-training export format.

## 2. What The DCE Results Should Claim

The DCE results should claim that the DCE was tested through the full construction of UAVape and its supporting training sources.

The simplest headline claim is:

> The DCE successfully supported the construction of the final UAVape V4 real dataset and its auxiliary scraped/synthetic training sources, producing a model-ready YOLO export with traceable curation, annotation, metadata and source-control evidence.

The DCE results are not claiming:

- That the DCE is faster than all commercial tools.
- That a large external user study was completed.
- That every method equation has a perfectly logged value.
- That model accuracy itself is a DCE result.

Model accuracy belongs mainly in the UAVape dataset/model results. However, the DCE discussion can mention that the later data-source ablation showed why source-controlled scraped/synthetic data mattered.

## 3. Clean Results Logic

Use three levels of evidence.

### A. Source Handling

This answers whether the DCE could handle different data sources.

Use these numbers:

| Source | Supported result | Use in DCE results |
|---|---:|---|
| Final UAVape V4 real dataset | 1,204 images; 2,040 boxes | Main real-data construction result |
| New V4 daylight import | 574 imported; 563 active after audit | Shows real batch import/audit |
| Scraped training source | 2,905 images; 4,437 boxes | Shows external source construction/filtering |
| Synthetic generated pool | 1,277 images/labels | Shows synthetic candidate generation |
| Synthetic promoted pool | 355 images/labels | Shows reviewed synthetic training pool |
| Synthetic promotion rate | 27.8% | 355 / 1,277 |

Important caveat:

- For scraped data, the accepted/usable pool is known: 2,905 images and 4,437 boxes.
- A fully reliable total attempted scrape denominator is not currently established from the available logs, so do not invent a "2,905 out of 5,000" style yield unless a complete scrape-attempt log is found.

### B. Human Workflow Evidence

This answers whether the DCE reduced bottlenecks in review and annotation.

Use these numbers:

| Workflow evidence | Value | What it proves |
|---|---:|---|
| Curation log entries | 764 | Review/curation activity was logged |
| Unique curated/reviewed files | 672 | Real review state existed beyond raw folders |
| Curated positive images | 546 | Positive candidate handling |
| Curated hard negatives | 188 | Hard-negative handling |
| Rejected/reject-labelled | 30 | Rejection/audit path |
| Clean negatives | 0 | Not used in the final active counts |
| SAM2 proposals generated | 2,665 | SAM2 assistant was used at scale |
| SAM2 proposals accepted | 2,151 | Human-approved proposal path |
| SAM2 proposal accept rate | 80.7% | Proposal usefulness |
| First-click accuracy | 87.5% | Low corrective-click burden |
| Interaction efficiency | 86.5% | Correction burden remained limited |
| Mean proposal accept time | 4.54 s | Logged accept-time evidence |
| One SAM2 batch session | 135 annotations in 342.5 s | 23.6 annotations/min session throughput |

Important caveat:

- Manual baseline annotation time was not logged, so do not report a formal time-reduction percentage against manual bounding-box drawing unless a manual baseline is added.

### C. Metadata and Export Readiness

This answers whether the dataset became failure-mode-ready and model-ready.

Use these numbers:

| Metadata/export evidence | Value | What it proves |
|---|---:|---|
| VLM metadata attempts | 1,402 | Metadata assistant used at scale |
| Successful VLM suggestions | 1,224 | Valid metadata outputs |
| Failed VLM suggestions | 178 | Logged failure path |
| VLM success rate | 87.3% | Reliability of metadata suggestion layer |
| Surface suggestion coverage | 86.2% | Failure-mode field coverage |
| Lighting suggestion coverage | 90.6% | Failure-mode field coverage |
| Background clutter coverage | 100% | Failure-mode field coverage |
| Condition coverage | 42.5% | Weaker field coverage |
| Viewpoint coverage | 44.5% | Weaker field coverage |
| Occlusion coverage | 99.9% | Failure-mode field coverage |
| Motion blur coverage | 100% | Failure-mode field coverage |
| Scale band VLM coverage | 6.6% | Low because scale should be geometry-derived |
| Final YOLO export images | 1,204 | Model-ready image export |
| Final YOLO label files | 1,204 | Model-ready label export |
| Final YOLO boxes | 2,040 | Exported annotations |
| Export warnings | 0 | Structural export integrity |

Important caveat:

- Scale band should not be framed as a VLM failure. The method says scale should be calculated from bounding-box area, not guessed by the VLM.
- The human metadata correction log has only 3 recorded corrections, so AI-human agreement should not be reported as a robust metric.

## 4. What We Have In Comparison

The DCE results have three kinds of comparison.

### Internal Yield Comparisons

These compare raw/candidate pools against accepted/promoted/final pools.

Good examples:

- New real daylight batch: 574 imported -> 563 accepted after audit.
- Synthetic generation: 1,277 generated -> 355 promoted, giving a 27.8% promotion rate.
- Final export: 1,204 images -> 1,204 YOLO labels, with 2,040 boxes and 0 warnings.

### Workflow Quality Comparisons

These compare automated assistance against human audit outcomes.

Good examples:

- SAM2 generated 2,665 proposals; 2,151 were accepted.
- First-click accuracy was 87.5%, showing most accepted SAM2 annotations did not need corrective re-clicking.
- VLM suggestions succeeded in 87.3% of attempts, but some fields were much more complete than others.

### Tool-Audit Comparison

This is not a numeric benchmark against Roboflow/CVAT/Label Studio.

The valid comparison is functional:

- Commercial annotation tools usually assume the dataset already exists.
- The DCE covers earlier dataset-construction steps: sourcing, review, synthetic bootstrapping, metadata and export.
- Therefore the DCE should be framed as an integrated research data factory, not simply another annotation tool.

Do not claim the DCE outperforms commercial tools quantitatively unless a formal user study or speed benchmark exists.

## 5. Recommended Simple Dissertation Framing

Use this framing:

> The DCE was evaluated by applying it to the complete UAVape dataset-construction workflow. The system ingested and audited real UAVape imagery, supported a 2,905-image scraped training source, generated 1,277 synthetic candidates and promoted 355 synthetic composites for training use. For the real UAVape workflow, the DCE logged curation, SAM2-assisted annotation, VLM metadata suggestion and final COCO-to-YOLO export. The final real export contained 1,204 images, 1,204 label files and 2,040 boxes with zero export warnings.

This keeps the section simple:

- Source construction: real, scraped, synthetic.
- Human workflow: curation, SAM2, VLM metadata.
- Model readiness: COCO/YOLO export.

Then leave detector performance to the UAVape dataset/model results section.

## 6. Design Requirements, Method, And Results Follow-Through

This is the controlling map for the DCE chapter. The results section should follow this chain:

> Design requirement -> method feature -> measured result -> honest interpretation.

The DCE is tested by applying it to the UAVape dataset-construction workflow and its supporting scraped/synthetic sources. It is not tested as a standalone commercial annotation-tool benchmark.

| Design requirement | Method feature | What the result should prove | Evidence available | Status |
|---|---|---|---|---|
| DCE-DR1 Multi-source sourcing | Upload/direct import, scraping, video-frame support, duplicate handling | The DCE can aggregate fragmented candidate data and stop obvious duplicate/corrupt material from entering unmanaged training pools. | V4 daylight import: 574 imported -> 563 active after audit; 61.4 images/min direct-import proxy; exact duplicate hash groups = 0 in current-real freeze; scraped source = 2,905 images / 4,437 boxes. | Strong, but do not claim full scrape-attempt yield unless a complete denominator is found. |
| DCE-DR2 Synthetic generation | 2D cutout compositing with bounded controls and manual promotion | The synthetic module creates candidate images and only promoted images enter training sources. | 1,277 generated images/labels; 355 promoted images/labels; promotion rate = 27.8%. | Strong. |
| DCE-DR3 Curation and dataset balance | Review Queue with positive / hard-negative / clean-negative / reject categories and hotkeys/batch tagging | The DCE turns messy candidate imagery into explicit reviewed categories before annotation/training. | 764 curation log entries; 672 unique files; 546 positive; 188 hard negative; 30 rejected; 0 clean negatives. | Strong for categorisation; weak for formal speed comparison. |
| DCE-DR4 SAM2 annotation | Human-audited SAM2 proposal workflow | SAM2 assisted real annotation at scale while preserving human control. | 2,665 proposals; 2,151 accepted; 80.7% proposal accept rate; 87.5% first-click accuracy; 86.5% interaction efficiency; mean accept time 4.54s; one logged session: 135 annotations in 342.5s. | Strong for interaction efficiency; do not claim formal time reduction vs manual baseline unless a manual baseline is added. |
| DCE-DR5 Metadata tagging | GPT-4o-mini image-level metadata suggestions plus human review/edit | The metadata assistant can populate failure-mode fields and record failures. | 1,402 VLM attempts; 1,224 successes; 178 failures; 87.3% success rate; high coverage for background clutter, occlusion and motion blur; lower coverage for condition/viewpoint. | Strong for reliability/coverage; weak for AI-human agreement because only 3 correction records exist. |
| DCE-DR6 YOLO export integrity | COCO source of truth, YOLO export, audit/manifest output | The DCE produces a structurally valid model-ready export. | Final V4 export: 1,204 images; 1,204 label files; 2,040 boxes; 0 warnings; class IDs 0=vape, 1=lighter. | Strong. |

## 7. Design Requirement Wording That Needs Softening

Some design requirements are directionally right but should not overclaim in the final dissertation wording.

### DCE-DR3

Current wording says the curation UI proves it "significantly accelerates processing compared to standard drag-and-click sorting."

Safer wording:

> Evaluation of logged curation throughput and final category distribution to determine whether the Review Queue supported rapid, explicit sorting of candidate imagery.

Reason: we have curation logs and an elapsed-time proxy, but not a controlled drag-and-click baseline study.

### DCE-DR4

Current wording says SAM2 proves "substantial Time Reduction vs manual baselines."

Safer wording:

> Interaction metrics measuring first-click accuracy, corrective-interaction burden and proposal acceptance time. Manual-baseline comparison is discussed as a future usability benchmark unless separately measured.

Reason: we have strong SAM2 interaction logs, but no robust manual annotation baseline.

### DCE-DR5

Current wording is mostly fine, but "without exhausting the user with API failures" should be made measurable.

Safer wording:

> Quantitative evaluation of VLM suggestion reliability and per-field metadata coverage across defined environmental fields.

Reason: this aligns with the actual evidence: success/failure counts and field coverage.

## 8. What The Results Section Should Do

The DCE results section should be simple and modular. It should not be written as a scattered list of logs.

Recommended results order:

1. **Multi-source construction result.**
   - State that DCE was used to assemble the real UAVape workflow plus supporting scraped and synthetic sources.
   - Report: 1,204 final real images / 2,040 boxes; 2,905 scraped images / 4,437 boxes; 1,277 generated synthetic candidates -> 355 promoted.

2. **Curation and source-control result.**
   - Show that images were not dumped straight into training.
   - Report: curation entries, positive/hard-negative/reject counts, and explain that this created explicit source/category control.

3. **SAM2 annotation result.**
   - Show that annotation was human-audited but assisted.
   - Report proposal count, accept rate, first-click accuracy, interaction efficiency and accept time.
   - Do not report formal time reduction unless manual baseline is measured.

4. **VLM metadata result.**
   - Show that the DCE produced failure-mode-ready metadata.
   - Report VLM attempts, success rate and per-field coverage.
   - Clarify scale band is geometry-derived, not a VLM task.

5. **Export readiness result.**
   - Show that the DCE produced a model-ready dataset.
   - Report final YOLO export: 1,204 images, 1,204 labels, 2,040 boxes, 0 warnings.

The results section should end with a short interpretation:

> These results validate the DCE as a functional dataset-construction pipeline: it did not merely label images, but supported sourcing, synthetic bootstrapping, review, assisted annotation, metadata enrichment and model-ready export. Downstream detector performance is evaluated separately in the UAVape model results section.

## 9. Baseline/Proxy Notes To Avoid Drift

### Manual Annotation Baseline

If the manual Roboflow timing was genuinely measured, it can be reported as a small retrospective baseline:

- Manual platform: Roboflow manual bounding-box annotation.
- Sample size: 50 images.
- Manual mean annotation time: 13.2 seconds/image.

Use this carefully:

- SAM2 event-level mean proposal accept time from DCE logs: 4.54 seconds.
- SAM2 logged batch-session rate: 135 annotations in 342.543 seconds = 2.54 seconds/annotation.

Possible time-reduction calculations:

- Conservative event-level comparison: \(TR = (1 - 4.54 / 13.2) \times 100 = 65.6\%\).
- Logged batch-session comparison: \(TR = (1 - 2.54 / 13.2) \times 100 = 80.8\%\).

Recommended wording:

> A small retrospective manual baseline was collected by annotating 50 images with manual Roboflow bounding boxes, producing a mean annotation time of 13.2 seconds per image. Against the logged SAM2-assisted DCE batch-session rate of 2.54 seconds per annotation, this corresponds to an indicative 80.8% time reduction. This should be treated as a small baseline comparison rather than a full usability study.

Do not also report "3 seconds" unless it refers to the SAM2 batch result rounded from 2.54 seconds. Use one consistent value.

### Curation Baseline

Do not invent a drag-and-click curation baseline. The method can report DCE curation throughput and category outcomes, but the controlled comparison against a generic drag-and-click interface was not formally measured.

Recommended wording:

> Because no separate drag-and-click baseline was logged, the curation result is reported as logged DCE throughput and category distribution rather than a controlled comparative usability result.

### Scrape Attempt Denominator

The available scrape run logs do not support a clean "X attempted -> 2,905 usable" yield claim.

What the logs show:

- 15 logged scraper runs.
- 196 downloaded images.
- 16 failed downloads.
- 119 duplicate skips.
- 77 accepted into review.

What the source pool shows separately:

- Scraped training source: 2,905 images and 4,437 boxes.

Recommended wording:

> The DCE and associated source-construction workflow produced a scraped training source of 2,905 images and 4,437 boxes. The complete scrape-attempt denominator for this larger source pool was not preserved in the final logs, so the result is reported as a usable source-pool size rather than as a scrape-yield percentage.

