# UAVape RunPod Recovery Note

Date recovered locally: 2026-06-10

The old RunPod network volume was successfully mounted into a recovery pod and the key UAVape V4 evidence files were recovered.

## Recovered Archive

Downloaded archive:

```text
/Users/zainahmad/Downloads/uavape_v4_recovery_evidence_20260610.tar.gz
```

Local extracted copy:

```text
/Users/zainahmad/Documents/Codex/2026-06-03/paste-this-into-the-new-chat/recovered_runpod_evidence_20260610/
```

Archive check:

| Item | Result |
|---|---:|
| Downloaded archive size | 11 MB |
| Archive entries | 172 |
| Extracted recovered files | 143 |
| Extracted evidence directory size on RunPod before compression | 50 MB |

## Evidence Recovered

The recovered bundle includes:

- `Untitled.ipynb`, the RunPod notebook used for V4 training/evaluation work.
- Initial model ablation result files under `runs/`.
- Data-source ablation result files under `runs_data_ablation/`.
- Tiling/SAHI evaluation result files under `tiling_eval/`.
- Per-run `results.csv`, `args.yaml`, `results.png`, `labels.jpg`, and confusion matrix images where present.

Important caveat: `runs_data_ablation/v4_ablate_A0_real_only/results.csv` contains only 9 epochs and is not the real-only baseline used in the data-source ablation comparison. The correct real-only baseline is the completed `runs/v4_yolo26s_img1280` run.

The full model weights were intentionally not included in the compact evidence archive. The final selected checkpoint was downloaded separately after the sealed test evaluation.

## Final Test Evaluation After Recovery

After the network volume was recovered, it was mounted into a new H100 NVL RunPod pod and the selected data-ablation checkpoint was evaluated on the sealed V4 test split.

Checkpoint evaluated:

```text
/workspace/uavape_v4_model_benchmark/runs_data_ablation/v4_ablate_S355_Y355_combined/weights/best.pt
```

Test output path on RunPod:

```text
/workspace/uavape_v4_model_benchmark/final_test/v4_ablate_S355_Y355_combined_test
```

Downloaded local archive:

```text
/Users/zainahmad/Downloads/uavape_v4_final_test_20260611.tar.gz
```

Archive check:

| Item | Result |
|---|---:|
| Downloaded archive size | 5.6 MB |
| Archive entries | 13 |

The archive contains the final test plots and previews, including precision/recall/F1 curves, confusion matrices, and validation/test batch label/prediction images. It does not include the model checkpoint.

Downloaded local checkpoint archive:

```text
/Users/zainahmad/Downloads/uavape_v4_final_bestpt_20260611.tar.gz
```

Checkpoint archive check:

| Item | Result |
|---|---:|
| Downloaded archive size | 18 MB |
| Archive entries | 1 |
| Contained checkpoint | `v4_ablate_S355_Y355_combined_best.pt` |
| Checkpoint size inside archive listing | 20,423,813 bytes |

Held-out test metrics:

| Class | Images | Instances | Precision | Recall | AP50 | AP50-95 |
|---|---:|---:|---:|---:|---:|---:|
| all | 184 | 420 | 0.849 | 0.725 | 0.851 | 0.638 |
| vape | 157 | 255 | 0.877 | 0.753 | 0.882 | 0.685 |
| lighter | 88 | 165 | 0.820 | 0.697 | 0.819 | 0.591 |

## Notebook Output Status

The recovered notebook is not empty:

| Notebook | Cells | Executed cells | Cells with outputs | Total output objects |
|---|---:|---:|---:|---:|
| `Untitled.ipynb` | 48 | 46 | 36 | 52 |
| `Untitled-checkpoint.ipynb` | 0 | 0 | 0 | 0 |

The main notebook contains saved output logs for:

- initial YOLO26/YOLO11/YOLOv8 model and input-size runs,
- tiling/SAHI validation evaluation,
- data-source ablation runs,
- final validation logs for `v4_ablate_S355_Y355_combined`.

Cell 47 contains the large saved output block for the V4 data-source ablation, including `v4_ablate_*` runs and validation logs.

## Recovered Run Families

Initial model/input-size ablation:

```text
recovered_runpod_evidence_20260610/recovery_export/workspace/uavape_v4_model_benchmark/runs/
```

Data-source ablation:

```text
recovered_runpod_evidence_20260610/recovery_export/workspace/uavape_v4_model_benchmark/runs_data_ablation/
```

Tiling/SAHI branch:

```text
recovered_runpod_evidence_20260610/recovery_export/workspace/uavape_v4_model_benchmark/tiling_eval/
```

## Interpretation

The project evidence is no longer dependent only on copied chat logs. The recovered archive preserves the notebook cell outputs and the per-run Ultralytics result artifacts needed to support the V4 method, model benchmark, tiling branch and data-source ablation discussion.

The recovered evidence archive, final test artifact archive and final checkpoint archive together preserve the dissertation evidence, appendix figures and selected model checkpoint.
