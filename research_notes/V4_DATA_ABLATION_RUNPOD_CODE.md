# V4 Data-Centric Ablation RunPod Code

Paste these cells after the tiling cells in the same RunPod notebook.

This is the V4 data-centric ablation protocol. It keeps the V4 validation and test splits frozen, changes only the training split, and evaluates whether additional one-class vape source data improves the selected V4 full-frame detector protocol.

Primary protocol:

- model: `yolo26s.pt`
- image size: `1280`
- epochs: `100`
- seed: `42`
- inference: full-frame, no SAHI tiling
- headline metric: vape AP50
- secondary metric: vape AP50-95

## Ablation Matrix

The doses match the V3 source-dose logic:

| Run | Training data | Interpretation |
|---|---|---|
| `v4_ablate_A0_real_only` | V4 real train only | Real-data baseline. |
| `v4_ablate_S88_scraped` | V4 real train + 88 scraped vape images | Low scraped dose. |
| `v4_ablate_S177_scraped` | V4 real train + 177 scraped vape images | Medium scraped dose. |
| `v4_ablate_S355_scraped` | V4 real train + 355 scraped vape images | Full scraped dose. |
| `v4_ablate_Y88_synthetic` | V4 real train + 88 synthetic vape images | Low synthetic dose. |
| `v4_ablate_Y177_synthetic` | V4 real train + 177 synthetic vape images | Medium synthetic dose. |
| `v4_ablate_Y355_synthetic` | V4 real train + 355 synthetic vape images | Full synthetic dose. |
| `v4_ablate_S355_Y355_combined` | V4 real train + 355 scraped + 355 synthetic images | Optional combined-source stress test. |

The scraped and synthetic pools are expected to be one-class vape sources. This is acceptable only when their label class `0` means `vape`, because V4 also uses `0: vape` and `1: lighter`.

## Expected Source Locations

The old V3 Colab runbook used this Google Drive input folder:

```text
/content/drive/MyDrive/uavape_ablation/inputs
```

with these source zip files:

```text
external_positive.zip
synthetic_positive.zip
```

The first cell below mounts Drive and extracts those zips into the current V4 notebook workspace. The later cells then read from the extracted folders.

If you are on RunPod rather than Colab, mount Google Drive in the notebook environment first or copy the same zip files into:

```text
/workspace/uavape_v4_model_benchmark/addons/scraped/images
/workspace/uavape_v4_model_benchmark/addons/scraped/labels
/workspace/uavape_v4_model_benchmark/addons/synthetic/images
/workspace/uavape_v4_model_benchmark/addons/synthetic/labels
```

The code also checks likely old/prepared locations if you copied older V3-style folders into the benchmark workspace:

```text
/workspace/uavape_v4_model_benchmark/old_sources/external_vape_aggregated_vape_only/yolo_pool/external_positive/images
/workspace/uavape_v4_model_benchmark/old_sources/external_vape_aggregated_vape_only/yolo_pool/external_positive/labels
/workspace/uavape_v4_model_benchmark/old_sources/synthetic_promoted_only/splits/default_70_20_10_e3/train/images
/workspace/uavape_v4_model_benchmark/old_sources/synthetic_promoted_only/splits/default_70_20_10_e3/train/labels
```

If your folders differ, edit `SCRAPED_CANDIDATES` and `SYNTHETIC_CANDIDATES` in Cell 1.

## Cell 0: Mount Drive And Extract Old V3 Source Zips

Run this before the ablation build cells. It is safe to rerun.

```python
from pathlib import Path
import shutil
import zipfile

ROOT = Path("/workspace/uavape_v4_model_benchmark")
ADDONS = ROOT / "addons"
ADDONS.mkdir(parents=True, exist_ok=True)

try:
    from google.colab import drive
    drive.mount("/content/drive")
    DRIVE_INPUTS = Path("/content/drive/MyDrive/uavape_ablation/inputs")
except Exception as e:
    print("Colab Drive mount not available in this runtime:", repr(e))
    DRIVE_INPUTS = Path("/content/drive/MyDrive/uavape_ablation/inputs")

ZIP_SOURCES = {
    "scraped": DRIVE_INPUTS / "external_positive.zip",
    "synthetic": DRIVE_INPUTS / "synthetic_positive.zip",
}

def extract_source_zip(source_name, zip_path):
    out_dir = ADDONS / source_name
    if (out_dir / "images").exists() and (out_dir / "labels").exists():
        print(f"{source_name} already extracted:", out_dir)
        return out_dir
    if not zip_path.exists():
        raise FileNotFoundError(
            f"Missing {source_name} zip: {zip_path}. "
            "Check Drive path or upload the V3 input zips."
        )
    if out_dir.exists():
        shutil.rmtree(out_dir)
    out_dir.mkdir(parents=True, exist_ok=True)
    with zipfile.ZipFile(zip_path, "r") as z:
        z.extractall(out_dir)

    # If the zip contained a nested folder with images/labels, flatten to addons/source.
    image_dirs = list(out_dir.rglob("images"))
    label_dirs = list(out_dir.rglob("labels"))
    if image_dirs and label_dirs and image_dirs[0].parent != out_dir:
        nested = image_dirs[0].parent
        flat_tmp = ADDONS / f"_{source_name}_flat_tmp"
        if flat_tmp.exists():
            shutil.rmtree(flat_tmp)
        flat_tmp.mkdir(parents=True)
        shutil.copytree(nested / "images", flat_tmp / "images")
        shutil.copytree(nested / "labels", flat_tmp / "labels")
        shutil.rmtree(out_dir)
        flat_tmp.rename(out_dir)

    print(f"Extracted {source_name}: {zip_path} -> {out_dir}")
    print("images:", len(list((out_dir / "images").glob("*"))) if (out_dir / "images").exists() else "missing")
    print("labels:", len(list((out_dir / "labels").glob("*.txt"))) if (out_dir / "labels").exists() else "missing")
    return out_dir

for source_name, zip_path in ZIP_SOURCES.items():
    extract_source_zip(source_name, zip_path)
```

## Cell 1: Find And Audit Add-On Pools

```python
from pathlib import Path
import random
import shutil
import csv
import yaml
from collections import Counter

ROOT = Path("/workspace/uavape_v4_model_benchmark")
BASE = ROOT / "datasets" / "v4_real_final_20260610"
OUT_ROOT = ROOT / "datasets" / "v4_data_ablation_20260610"

SEED = 42
DOSES = [88, 177, 355]
IMG_EXTS = {".jpg", ".jpeg", ".png", ".webp", ".bmp"}

SCRAPED_CANDIDATES = [
    ROOT / "addons" / "scraped",
    ROOT / "old_sources" / "external_vape_aggregated_vape_only" / "yolo_pool" / "external_positive",
    ROOT / "experiments" / "ml" / "results" / "external_vape_aggregated_vape_only" / "yolo_pool" / "external_positive",
]

SYNTHETIC_CANDIDATES = [
    ROOT / "addons" / "synthetic",
    ROOT / "old_sources" / "synthetic_promoted_only" / "splits" / "default_70_20_10_e3" / "train",
    ROOT / "experiments" / "ml" / "results" / "synthetic_promoted_only" / "splits" / "default_70_20_10_e3" / "train",
]

def normalise_pool_dir(pool_dir):
    pool_dir = Path(pool_dir)
    if (pool_dir / "images").exists() and (pool_dir / "labels").exists():
        return pool_dir / "images", pool_dir / "labels"
    if (pool_dir / "train" / "images").exists() and (pool_dir / "train" / "labels").exists():
        return pool_dir / "train" / "images", pool_dir / "train" / "labels"
    return None, None

def list_yolo_pairs(pool_dir):
    images_dir, labels_dir = normalise_pool_dir(pool_dir)
    if images_dir is None:
        return []
    pairs = []
    for img in sorted(images_dir.iterdir()):
        if img.suffix.lower() not in IMG_EXTS:
            continue
        lab = labels_dir / f"{img.stem}.txt"
        if lab.exists():
            pairs.append((img, lab))
    return pairs

def find_first_pool(candidates, label):
    checked = []
    for candidate in candidates:
        pairs = list_yolo_pairs(candidate)
        checked.append((candidate, len(pairs)))
        if pairs:
            print(f"{label} pool selected: {candidate} | labelled images={len(pairs)}")
            return Path(candidate)
    print(f"No {label} pool found. Checked:")
    for path, count in checked:
        print(f"  {path} | labelled images={count}")
    return None

def audit_pool(pool_dir, label):
    pairs = list_yolo_pairs(pool_dir)
    cls_counts = Counter()
    malformed = []
    empty = 0
    for _, lab in pairs:
        text = lab.read_text().splitlines()
        if not text:
            empty += 1
            continue
        for line in text:
            parts = line.split()
            if len(parts) != 5:
                malformed.append(str(lab))
                continue
            cls_counts[parts[0]] += 1
    print(f"\n{label} audit")
    print("pool:", pool_dir)
    print("labelled images:", len(pairs))
    print("class ids:", dict(sorted(cls_counts.items())))
    print("empty labels:", empty)
    print("malformed labels:", len(set(malformed)))
    if set(cls_counts.keys()) - {"0"}:
        raise ValueError(f"{label} contains class IDs other than 0. Do not use until remapped/audited.")
    if len(pairs) < max(DOSES):
        raise ValueError(f"{label} has only {len(pairs)} labelled images; need at least {max(DOSES)}.")
    return pairs

SCRAPED = find_first_pool(SCRAPED_CANDIDATES, "scraped")
SYNTHETIC = find_first_pool(SYNTHETIC_CANDIDATES, "synthetic")

if SCRAPED is None:
    raise FileNotFoundError("Scraped pool not found. Copy it into /workspace/uavape_v4_model_benchmark/addons/scraped or edit SCRAPED_CANDIDATES.")
if SYNTHETIC is None:
    raise FileNotFoundError("Synthetic pool not found. Copy it into /workspace/uavape_v4_model_benchmark/addons/synthetic or edit SYNTHETIC_CANDIDATES.")

scraped_pairs = audit_pool(SCRAPED, "scraped")
synthetic_pairs = audit_pool(SYNTHETIC, "synthetic")

print("\nOK: add-on pools are one-class class-0 sources, compatible with V4 class 0 = vape.")
```

## Cell 2: Build Frozen-Val/Test Ablation Datasets

```python
from pathlib import Path
import random
import shutil
import csv
import yaml
from collections import Counter

def sample_nested_pairs(pool_dir, doses, seed=SEED):
    pairs = list_yolo_pairs(pool_dir)
    rng = random.Random(seed)
    shuffled = pairs[:]
    rng.shuffle(shuffled)
    max_n = max(doses)
    selected = shuffled[:max_n]
    return {n: sorted(selected[:n], key=lambda p: p[0].name) for n in doses}

def copy_split(src_dataset, dst_dataset, split):
    for kind in ["images", "labels"]:
        src = src_dataset / kind / split
        dst = dst_dataset / kind / split
        dst.mkdir(parents=True, exist_ok=True)
        for item in sorted(src.iterdir()):
            if item.is_file():
                shutil.copy2(item, dst / item.name)

def copy_addons(dst_dataset, pairs, prefix):
    img_dst = dst_dataset / "images" / "train"
    lab_dst = dst_dataset / "labels" / "train"
    rows = []
    for img, lab in pairs:
        img_name = f"{prefix}_{img.name}"
        lab_name = f"{prefix}_{lab.name}"
        shutil.copy2(img, img_dst / img_name)
        shutil.copy2(lab, lab_dst / lab_name)
        rows.append({"source": prefix, "image": str(img), "label": str(lab), "copied_image": img_name})
    return rows

def count_split(dataset, split):
    image_dir = dataset / "images" / split
    label_dir = dataset / "labels" / split
    image_count = len([p for p in image_dir.iterdir() if p.suffix.lower() in IMG_EXTS])
    label_count = len([p for p in label_dir.iterdir() if p.suffix.lower() == ".txt"])
    class_counts = Counter()
    box_count = 0
    for lab in label_dir.glob("*.txt"):
        for line in lab.read_text().splitlines():
            if not line.strip():
                continue
            cls = int(float(line.split()[0]))
            class_counts[cls] += 1
            box_count += 1
    return image_count, label_count, box_count, class_counts

def write_dataset_yaml(dataset):
    data = {
        "path": str(dataset),
        "train": "images/train",
        "val": "images/val",
        "test": "images/test",
        "names": {0: "vape", 1: "lighter"},
    }
    with open(dataset / "dataset.yaml", "w") as f:
        yaml.safe_dump(data, f, sort_keys=False)

def build_dataset(name, scraped=None, synthetic=None):
    scraped = scraped or []
    synthetic = synthetic or []
    dst = OUT_ROOT / name
    if dst.exists():
        shutil.rmtree(dst)
    for split in ["train", "val", "test"]:
        copy_split(BASE, dst, split)

    manifest_rows = []
    manifest_rows += copy_addons(dst, scraped, "scraped")
    manifest_rows += copy_addons(dst, synthetic, "synthetic")
    write_dataset_yaml(dst)

    with open(dst / "addon_manifest.csv", "w", newline="") as f:
        writer = csv.DictWriter(f, fieldnames=["source", "image", "label", "copied_image"])
        writer.writeheader()
        writer.writerows(manifest_rows)

    summary_rows = []
    for split in ["train", "val", "test"]:
        images, labels, boxes, class_counts = count_split(dst, split)
        summary_rows.append({
            "dataset": name,
            "split": split,
            "images": images,
            "labels": labels,
            "boxes": boxes,
            "vape_boxes": class_counts.get(0, 0),
            "lighter_boxes": class_counts.get(1, 0),
        })

    with open(dst / "dataset_summary.csv", "w", newline="") as f:
        writer = csv.DictWriter(f, fieldnames=summary_rows[0].keys())
        writer.writeheader()
        writer.writerows(summary_rows)

    print(f"\n{name}")
    for row in summary_rows:
        print(row)
    return dst

OUT_ROOT.mkdir(parents=True, exist_ok=True)

scraped_samples = sample_nested_pairs(SCRAPED, DOSES, seed=SEED)
synthetic_samples = sample_nested_pairs(SYNTHETIC, DOSES, seed=SEED)

ABLATION_DATASETS = []
ABLATION_DATASETS.append(build_dataset("v4_ablate_A0_real_only"))

for dose in DOSES:
    ABLATION_DATASETS.append(
        build_dataset(f"v4_ablate_S{dose}_scraped", scraped=scraped_samples[dose])
    )

for dose in DOSES:
    ABLATION_DATASETS.append(
        build_dataset(f"v4_ablate_Y{dose}_synthetic", synthetic=synthetic_samples[dose])
    )

ABLATION_DATASETS.append(
    build_dataset(
        "v4_ablate_S355_Y355_combined",
        scraped=scraped_samples[355],
        synthetic=synthetic_samples[355],
    )
)

ABLATIONS = [p.name for p in ABLATION_DATASETS]
print("\nAblations ready:")
for name in ABLATIONS:
    print(" ", name, OUT_ROOT / name / "dataset.yaml")
```

## Cell 3: Train The Ratio-Dose Ablations

```python
from ultralytics import YOLO
from pathlib import Path

DATA_ROOT = OUT_ROOT
RUNS_ROOT = ROOT / "runs_data_ablation"

MODEL = "yolo26s.pt"
IMGSZ = 1280
EPOCHS = 100
SEED = 42

for name in ABLATIONS:
    data_yaml = DATA_ROOT / name / "dataset.yaml"
    print("\n" + "=" * 80)
    print(name)
    print("=" * 80)

    model = YOLO(MODEL)
    model.train(
        data=str(data_yaml),
        imgsz=IMGSZ,
        epochs=EPOCHS,
        batch=16,
        seed=SEED,
        deterministic=True,
        patience=100,
        project=str(RUNS_ROOT),
        name=name,
        exist_ok=True,
        cos_lr=True,
        device=0,
        workers=8,
        val=True,
    )
```

## Cell 4: Summarise Validation Results

```python
from ultralytics import YOLO
import csv

rows = []
for name in ABLATIONS:
    weights = RUNS_ROOT / name / "weights" / "best.pt"
    data_yaml = DATA_ROOT / name / "dataset.yaml"
    if not weights.exists():
        print("Missing weights:", weights)
        continue

    model = YOLO(str(weights))
    metrics = model.val(data=str(data_yaml), split="val", imgsz=IMGSZ, batch=16, device=0, plots=True)
    box = metrics.box

    row = {
        "run": name,
        "weights": str(weights),
        "aggregate_P": float(box.mp),
        "aggregate_R": float(box.mr),
        "aggregate_mAP50": float(box.map50),
        "aggregate_mAP50_95": float(box.map),
        "vape_P": float(box.p[0]),
        "vape_R": float(box.r[0]),
        "vape_AP50": float(box.ap50[0]),
        "vape_AP50_95": float(box.ap[0]),
        "lighter_P": float(box.p[1]) if len(box.p) > 1 else None,
        "lighter_R": float(box.r[1]) if len(box.r) > 1 else None,
        "lighter_AP50": float(box.ap50[1]) if len(box.ap50) > 1 else None,
        "lighter_AP50_95": float(box.ap[1]) if len(box.ap) > 1 else None,
    }
    rows.append(row)
    print(row)

out_csv = ROOT / "v4_data_ablation_validation_summary.csv"
with open(out_csv, "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=rows[0].keys())
    writer.writeheader()
    writer.writerows(rows)

print("Saved:", out_csv)
```

## Decision Rule

Use validation only. Do not touch the V4 held-out test split until the final data recipe is chosen.

Preferred ranking:

1. vape AP50
2. vape AP50-95
3. vape recall/precision balance
4. lighter-class diagnostics
5. aggregate mAP only as context

If no add-on recipe beats `A0`, report that curated V4 real-only training remained strongest and that scraped/synthetic source additions did not improve target-domain validation under the selected detector protocol.
