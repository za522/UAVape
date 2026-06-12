# UAVape V4 Data-Centric Ablation Results

Date captured: 2026-06-10

This note records the final validation metrics from the V4 data-source ablation runs and the later sealed test evaluation of the selected recipe. The ablation used the selected validation detector configuration:

- Model: `YOLO26s`
- Input size: `1280`
- Inference: full-frame validation, no tiling
- Epochs: `100`
- Seed: `42`
- Validation split: fixed V4 validation set, 183 images, 302 effective instances after Ultralytics removed one duplicate validation label
- Final test split: sealed V4 test set, 184 images, 420 instances
- Classes: `0 = vape`, `1 = lighter`

The original real-only baseline is the previously selected `YOLO26s @ 1280` run. The ablation changes only the training data by adding source-controlled vape-only data to the V4 real training set.

Recovery note: the recovered `runs_data_ablation/v4_ablate_A0_real_only/results.csv` is a short 9-epoch run and should not be used as the real-only baseline. The baseline reported here is the earlier completed `runs/v4_yolo26s_img1280` validation run, which is the correct locked detector-protocol baseline for the data-source ablation.

## Dataset Recipes

| Run | Training Recipe | Train Images | Train Boxes | Vape Boxes | Lighter Boxes |
|---|---|---:|---:|---:|---:|
| `A0_real_only` | V4 real train only | 837 | 1317 | 870 | 447 |
| `S88_scraped` | V4 real + 88 scraped vape images | 925 | 1440 | 993 | 447 |
| `S177_scraped` | V4 real + 177 scraped vape images | 1014 | 1579 | 1132 | 447 |
| `S355_scraped` | V4 real + 355 scraped vape images | 1192 | 1864 | 1417 | 447 |
| `Y88_synthetic` | V4 real + 88 synthetic vape images | 925 | 1405 | 958 | 447 |
| `Y177_synthetic` | V4 real + 177 synthetic vape images | 1014 | 1494 | 1047 | 447 |
| `Y355_synthetic` | V4 real + 355 synthetic vape images | 1192 | 1672 | 1225 | 447 |
| `S355_Y355_combined` | V4 real + 355 scraped + 355 synthetic vape images | 1547 | 2219 | 1772 | 447 |

## Validation Results

Primary metric: vape AP50. Secondary metric: vape AP50-95.

| Run | All P | All R | All AP50 | All AP50-95 | Vape P | Vape R | Vape AP50 | Vape AP50-95 | Lighter P | Lighter R | Lighter AP50 | Lighter AP50-95 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| `A0_real_only` | 0.839 | 0.749 | 0.802 | 0.605 | 0.909 | 0.839 | 0.918 | 0.707 | 0.770 | 0.660 | 0.688 | 0.498 |
| `S88_scraped` | 0.860 | 0.739 | 0.774 | 0.601 | 0.907 | 0.797 | 0.874 | 0.696 | 0.812 | 0.680 | 0.675 | 0.506 |
| `S177_scraped` | 0.861 | 0.746 | 0.799 | 0.619 | 0.897 | 0.821 | 0.899 | 0.720 | 0.825 | 0.670 | 0.699 | 0.519 |
| `S355_scraped` | 0.833 | 0.709 | 0.777 | 0.611 | 0.910 | 0.797 | 0.891 | 0.718 | 0.756 | 0.621 | 0.663 | 0.505 |
| `Y88_synthetic` | 0.874 | 0.721 | 0.803 | 0.617 | 0.925 | 0.792 | 0.912 | 0.712 | 0.823 | 0.649 | 0.694 | 0.522 |
| `Y177_synthetic` | 0.867 | 0.716 | 0.792 | 0.598 | 0.897 | 0.822 | 0.903 | 0.692 | 0.838 | 0.610 | 0.681 | 0.504 |
| `Y355_synthetic` | 0.797 | 0.747 | 0.793 | 0.616 | 0.888 | 0.824 | 0.898 | 0.716 | 0.706 | 0.670 | 0.687 | 0.516 |
| `S355_Y355_combined` | 0.833 | 0.774 | 0.815 | 0.639 | 0.905 | 0.853 | 0.922 | 0.737 | 0.760 | 0.695 | 0.708 | 0.541 |

## Main Finding

The combined source recipe is the strongest validation result:

- Best vape AP50: `S355_Y355_combined` = 0.922, compared with A0 = 0.918.
- Best vape AP50-95: `S355_Y355_combined` = 0.737, compared with A0 = 0.707.
- Best aggregate AP50: `S355_Y355_combined` = 0.815, compared with A0 = 0.802.
- Best aggregate AP50-95: `S355_Y355_combined` = 0.639, compared with A0 = 0.605.
- Best lighter AP50-95 also occurs in the combined run: 0.541, compared with A0 = 0.498.

The data-centric result is therefore positive but nuanced: individual scraped or synthetic additions did not reliably beat the real-only baseline on the primary vape AP50 metric, but the combined scraped-plus-synthetic recipe improved both headline detection performance and stricter localization performance.

## Sealed Test Result

After selecting `S355_Y355_combined` using validation, the recovered `best.pt` checkpoint was evaluated once on the held-out test split:

```text
/workspace/uavape_v4_model_benchmark/runs_data_ablation/v4_ablate_S355_Y355_combined/weights/best.pt
```

The final test output was saved on RunPod at:

```text
/workspace/uavape_v4_model_benchmark/final_test/v4_ablate_S355_Y355_combined_test
```

| Split | Images | Instances | All P | All R | All AP50 | All AP50-95 | Vape AP50 | Vape AP50-95 | Lighter AP50 | Lighter AP50-95 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Validation | 183 | 302 | 0.833 | 0.774 | 0.815 | 0.639 | 0.922 | 0.737 | 0.708 | 0.541 |
| Test | 184 | 420 | 0.849 | 0.725 | 0.851 | 0.638 | 0.882 | 0.685 | 0.819 | 0.591 |

The test result is strong and broadly confirms the validation finding, with aggregate AP50 increasing from 0.815 to 0.851 and aggregate AP50-95 remaining essentially stable at 0.638. Vape AP50 falls from 0.922 on validation to 0.882 on test, so the final discussion should avoid treating the validation vape score as the final generalisation score. The correct final held-out vape score is AP50 = 0.882 and AP50-95 = 0.685.

## Interpretation

The ablation supports a data-centric conclusion rather than a model-architecture conclusion. The model and training recipe were held constant, while only the training data source and dose changed. Under that setup, the best result came from combining complementary external sources.

The non-monotonic dose behavior is important:

- Scraped-only data peaked at the medium dose on strict localization (`S177_scraped`, vape AP50-95 = 0.720), while full scraped dose did not improve headline vape AP50.
- Synthetic-only data at the smallest dose (`Y88_synthetic`) nearly matched the real-only baseline and improved aggregate AP50-95, but larger synthetic doses did not produce a monotonic gain.
- The combined full-dose source recipe produced the clearest improvement, suggesting that source diversity mattered more than simply adding more images from one source.

## Dissertation Framing

This can be written as:

> A fixed YOLO26s@1280 detector was used to isolate the effect of training data composition. Source-dose ablations showed that adding scraped or synthetic vape-only data individually produced mixed effects, but combining both sources yielded the best validation performance, improving vape AP50 from 0.918 to 0.922 and vape AP50-95 from 0.707 to 0.737. This indicates that the data-centric workflow can identify beneficial source mixtures rather than assuming that more data from any single source will necessarily improve performance.

Important caveat:

- Validation was used for model, resolution and data-recipe selection. The sealed test result above should be used for final generalization performance.
- Do not claim deployment readiness from these results. Drone/mobile deployment would require latency, memory, power, export, and hardware testing.

## Evidence Status

These metrics were first captured from the RunPod/Jupyter training logs pasted into the working chat on 2026-06-10.

The RunPod network volume was later recovered and a compact evidence archive was downloaded locally:

```text
/Users/zainahmad/Downloads/uavape_v4_recovery_evidence_20260610.tar.gz
```

The archive contains the saved notebook outputs plus per-run `results.csv`, `args.yaml`, plots and confusion matrices for the initial model benchmark, tiling branch and data-source ablation. A local extracted copy is available at:

```text
/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/recovered_runpod_evidence_20260610/
```

See `UAVAPE_RUNPOD_RECOVERY_20260610.md` for the recovery ledger.

The final sealed test metrics were captured from the H100 NVL RunPod notebook output on 2026-06-11 after the recovered network volume was mounted into a new GPU pod.

The final test artifact archive was also downloaded locally:

```text
/Users/zainahmad/Downloads/uavape_v4_final_test_20260611.tar.gz
```

This archive contains the final test curves, confusion matrices and label/prediction preview images.

The selected final checkpoint was downloaded separately:

```text
/Users/zainahmad/Downloads/uavape_v4_final_bestpt_20260611.tar.gz
```

It contains `v4_ablate_S355_Y355_combined_best.pt`.
