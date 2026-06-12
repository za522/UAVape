# UAVape V4 Colab Training Plan

This plan uses the frozen, leakage-aware V4 YOLO export:

`v4_real_final_20260610`

Do not recreate train/validation/test randomly in Colab. The split is already fixed from the dataset engine export.

## Dataset Integrity

Final frozen split:

| Split | Images | Vape boxes | Lighter boxes | Old V3 images | New V4 images | Groups |
|---|---:|---:|---:|---:|---:|---:|
| train | 837 | 870 | 447 | 525 | 312 | 87 |
| val | 183 | 203 | 100 | 116 | 67 | 10 |
| test | 184 | 255 | 165 | 0 | 184 | 19 |

Method note:

- Test is drawn only from the new V4 daylight batch.
- Train/val use old audited V3 data plus non-test V4 data.
- New V4 images were grouped into sequential capture blocks to reduce scene/near-duplicate leakage.
- Model and image-size choices must be made using validation only.
- Test is used only once after model/image-size/checkpoint protocol is locked.

## Colab Cell 1: Mount Drive And Setup Paths

```python
from google.colab import drive
drive.mount("/content/drive")

from pathlib import Path

PROJECT = Path("/content/uavape_v4_model_benchmark")
PROJECT.mkdir(parents=True, exist_ok=True)

# Upload/copy this archive to Drive first.
ARCHIVE = Path("/content/drive/MyDrive/uavape_v4/v4_real_final_20260610_yolo_export.tar.gz")
DATA_ROOT = PROJECT / "datasets"
DATA_DIR = DATA_ROOT / "v4_real_final_20260610"

print("PROJECT:", PROJECT)
print("ARCHIVE:", ARCHIVE)
print("DATA_DIR:", DATA_DIR)
```

## Colab Cell 2: Install/Pin Training Stack

Use the same Ultralytics line as previous runs if you want maximum continuity. Otherwise install current Ultralytics.

```python
!pip -q install "ultralytics==8.4.62"

import ultralytics
ultralytics.checks()
```

## Colab Cell 3: Unpack Frozen Dataset

```python
import tarfile
from pathlib import Path

DATA_ROOT.mkdir(parents=True, exist_ok=True)

if not DATA_DIR.exists():
    with tarfile.open(ARCHIVE, "r:gz") as tar:
        tar.extractall(DATA_ROOT)
else:
    print("Dataset already unpacked:", DATA_DIR)

yaml_path = DATA_DIR / "dataset.yaml"
print(yaml_path.read_text())
```

## Colab Cell 4: Patch Dataset YAML For Colab Path

The local Mac YAML contains an absolute Mac path. Patch only the `path:` field after unpacking.

```python
import yaml

yaml_path = DATA_DIR / "dataset.yaml"
data_yaml = yaml.safe_load(yaml_path.read_text())
data_yaml["path"] = str(DATA_DIR)

with open(yaml_path, "w") as f:
    yaml.safe_dump(data_yaml, f, sort_keys=False)

print(yaml_path.read_text())
```

## Colab Cell 5: Verify Counts Before Training

```python
from pathlib import Path
from collections import Counter

def count_split(split):
    img_dir = DATA_DIR / "images" / split
    lab_dir = DATA_DIR / "labels" / split
    images = sorted([p for p in img_dir.iterdir() if p.suffix.lower() in {".jpg", ".jpeg", ".png"}])
    labels = sorted(lab_dir.glob("*.txt"))
    class_counts = Counter()
    boxes = 0
    for label in labels:
        for line in label.read_text().splitlines():
            if not line.strip():
                continue
            cls = int(line.split()[0])
            class_counts[cls] += 1
            boxes += 1
    return {
        "split": split,
        "images": len(images),
        "labels": len(labels),
        "boxes": boxes,
        "vape_boxes": class_counts[0],
        "lighter_boxes": class_counts[1],
    }

for split in ["train", "val", "test"]:
    print(count_split(split))
```

Expected:

```text
train: images=837, vape=870, lighter=447
val:   images=183, vape=203, lighter=100
test:  images=184, vape=255, lighter=165
```

## Colab Cell 6: Define Benchmark Grid

Start with the small-model resolution sweep. This is the first defensible benchmark, not final tuning.

```python
BENCHMARK_RUNS = [
    {"model": "yolo26s.pt", "imgsz": 640},
    {"model": "yolo26s.pt", "imgsz": 960},
    {"model": "yolo26s.pt", "imgsz": 1280},
    {"model": "yolo11s.pt", "imgsz": 640},
    {"model": "yolo11s.pt", "imgsz": 960},
    {"model": "yolo11s.pt", "imgsz": 1280},
    # Literature anchor. Run if time allows.
    {"model": "yolov8s.pt", "imgsz": 640},
]

BENCHMARK_RUNS
```

## Colab Cell 7: Train One Candidate

Run one candidate first to confirm everything behaves.

```python
from ultralytics import YOLO

RUN = {"model": "yolo26s.pt", "imgsz": 640}

model = YOLO(RUN["model"])
results = model.train(
    data=str(yaml_path),
    project=str(PROJECT / "runs"),
    name=f'v4_{Path(RUN["model"]).stem}_img{RUN["imgsz"]}',
    imgsz=RUN["imgsz"],
    epochs=100,
    batch=16,
    device=0,
    workers=2,
    seed=42,
    deterministic=True,
    cos_lr=True,
    patience=100,
    exist_ok=True,
    plots=True,
)
```

## Colab Cell 8: Summarise Validation Best Checkpoint

```python
import pandas as pd
from pathlib import Path

run_dir = Path(results.save_dir)
csv_path = run_dir / "results.csv"
df = pd.read_csv(csv_path)
df.columns = [c.strip() for c in df.columns]

metric_col = "metrics/mAP50-95(B)"
best_idx = df[metric_col].idxmax()
best_row = df.loc[best_idx]

summary = {
    "run_dir": str(run_dir),
    "best_epoch_by_mAP50_95": int(best_row["epoch"]),
    "best_val_precision": float(best_row["metrics/precision(B)"]),
    "best_val_recall": float(best_row["metrics/recall(B)"]),
    "best_val_mAP50": float(best_row["metrics/mAP50(B)"]),
    "best_val_mAP50_95": float(best_row["metrics/mAP50-95(B)"]),
    "best_weights": str(run_dir / "weights" / "best.pt"),
}
summary
```

## Colab Cell 9: Run Official Test Only After Selection

Do not run this during the model/image-size search. Run it after the selected detector protocol is locked.

```python
from ultralytics import YOLO

BEST_WEIGHTS = "/content/path/to/selected/best.pt"
selected = YOLO(BEST_WEIGHTS)

test_results = selected.val(
    data=str(yaml_path),
    split="test",
    imgsz=SELECTED_IMGSZ,
    project=str(PROJECT / "runs"),
    name="official_test_selected_model",
    device=0,
    plots=True,
)
```

## Reporting Rule

Primary headline:

- Vape-class AP/P/R, especially AP50 and AP50-95.

Secondary:

- Lighter AP/P/R as auxiliary confuser-class diagnostics.
- Overall macro mAP as supporting context only.

