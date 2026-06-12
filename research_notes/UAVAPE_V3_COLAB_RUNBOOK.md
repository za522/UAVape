# UAVape V3 YOLO26n Colab Runbook

Paste these cells into Colab in order.

This is the clean V3 restart runbook. It replaces the earlier YOLOv10n pilot run with the current lightweight Ultralytics nano detector, `yolo26n.pt`.

Core rule: V3 is a data-centric ablation. Keep the detector, image size, split logic, seed and evaluation protocol fixed across all experiments.

## Experiment Matrix

| Experiment | Add-on beyond real positives + hard negatives | Purpose |
|---|---|---|
| V3-E1 | none | Operational baseline. |
| V3-E2 | 88 scraped positives | Low scraped dose. |
| V3-E3 | 177 scraped positives | Medium scraped dose. |
| V3-E4 | 355 scraped positives | Full matched scraped dose. |
| V3-E5 | 88 pure synthetic positives | Low synthetic dose. |
| V3-E6 | 177 pure synthetic positives | Medium synthetic dose. |
| V3-E7 | 355 pure synthetic positives | Full synthetic dose. |
| V3-E8 | 177 scraped + 177 pure synthetic positives | Combined medium-dose source test. |

Optional copy-paste experiments are auto-enabled only if `copy_paste_synthetic.zip` exists in the input folder.

## Cell 0: Optional Clean Restart

Run this if you want to wipe the current Colab workspace and the new YOLO26n Drive output folder before rebuilding everything. It does not touch the old `outputs_v3` folder unless you edit the path.

```python
from google.colab import drive
drive.mount("/content/drive")

from pathlib import Path
import shutil

WORK = Path("/content/uavape_v3_yolo26n")
DRIVE_OUTPUTS = Path("/content/drive/MyDrive/uavape_ablation/outputs_v3_yolo26n")

CLEAR_RESTART_OUTPUTS = True

if CLEAR_RESTART_OUTPUTS:
    if WORK.exists():
        shutil.rmtree(WORK)
        print("Removed Colab workspace:", WORK)
    if DRIVE_OUTPUTS.exists():
        shutil.rmtree(DRIVE_OUTPUTS)
        print("Removed Drive restart outputs:", DRIVE_OUTPUTS)
    print("Clean YOLO26n restart ready.")
else:
    print("CLEAR_RESTART_OUTPUTS is False; nothing removed.")
```

## Cell 1: Setup, Mount Drive, Extract Inputs

```python
from google.colab import drive
drive.mount("/content/drive")

!pip install -q -U ultralytics tabulate openpyxl xlsxwriter

from pathlib import Path
import shutil, zipfile, json, random, os, time, math
from collections import defaultdict

import numpy as np
import pandas as pd
from PIL import Image

WORK = Path("/content/uavape_v3_yolo26n")
DRIVE_INPUTS = Path("/content/drive/MyDrive/uavape_ablation/inputs")
DRIVE_OUTPUTS = Path("/content/drive/MyDrive/uavape_ablation/outputs_v3_yolo26n")

RAW = WORK / "raw"
DATASETS = WORK / "datasets"
RUNS = WORK / "runs"
FIGS = WORK / "figures"
REPORTS = WORK / "reports"
REJECTED = WORK / "rejected_inputs"

for p in [RAW, DATASETS, RUNS, FIGS, REPORTS, REJECTED, DRIVE_OUTPUTS]:
    p.mkdir(parents=True, exist_ok=True)

SEED = 42
rng = random.Random(SEED)

IMG_EXTS = {".jpg", ".jpeg", ".png", ".bmp", ".webp", ".tif", ".tiff", ".heic", ".heif", ".avif"}
VALID_PIL_FORMATS = {"JPEG", "PNG", "BMP", "WEBP", "TIFF", "HEIF", "AVIF"}

ZIP_MAP = {
    "positive_train": "positive_train.zip",
    "hard_negative_train": "hard_negative_train.zip",
    "positive_test": "positive_test.zip",
    "hard_negative_test": "hard_negative_test.zip",
    "scraped_positive": "external_positive.zip",
    "synthetic_positive": "synthetic_positive.zip",
}

optional_copy_paste_zip = DRIVE_INPUTS / "copy_paste_synthetic.zip"
if optional_copy_paste_zip.exists():
    ZIP_MAP["copy_paste_synthetic"] = "copy_paste_synthetic.zip"

def show_table(df, name, max_rows=100, save=True):
    print(f"\n=== {name} ===")
    display(df.head(max_rows))
    if save:
        out = REPORTS / f"{name}.csv"
        df.to_csv(out, index=False)
        print(f"Saved CSV: {out}")
    print(f"\nCopy/paste table: {name}")
    print(df.head(max_rows).to_markdown(index=False))

def backup_path(src, dest):
    src = Path(src)
    dest = Path(dest)
    if src.exists():
        if src.is_dir():
            shutil.copytree(src, dest, dirs_exist_ok=True)
        else:
            dest.parent.mkdir(parents=True, exist_ok=True)
            shutil.copy2(src, dest)

def backup_all(note=""):
    print(f"\nBacking up to Drive {note}".strip())
    backup_path(REPORTS, DRIVE_OUTPUTS / "reports")
    backup_path(FIGS, DRIVE_OUTPUTS / "figures")
    backup_path(RUNS, DRIVE_OUTPUTS / "runs")
    backup_path(DATASETS, DRIVE_OUTPUTS / "datasets")
    print("Backup complete:", DRIVE_OUTPUTS)

def extract_zip(zip_name, target_dir):
    target_dir.mkdir(parents=True, exist_ok=True)
    zip_path = DRIVE_INPUTS / zip_name
    if not zip_path.exists():
        raise FileNotFoundError(f"Missing input zip: {zip_path}")
    with zipfile.ZipFile(zip_path, "r") as z:
        z.extractall(target_dir)
    print(f"Extracted {zip_name} -> {target_dir}")

for pool, zip_name in ZIP_MAP.items():
    extract_zip(zip_name, RAW / pool)

for meta_name in ["download_log.jsonl", "metadata_ai_review_log.jsonl", "uavape_metadata.csv", "metadata.csv"]:
    src = DRIVE_INPUTS / meta_name
    if src.exists():
        shutil.copy2(src, REPORTS / meta_name)
        print(f"Copied metadata: {meta_name}")
```

## Cell 2: Audit Inputs And Build Source Pools

```python
def find_images_dir(pool_dir):
    candidates = list(pool_dir.rglob("images"))
    return candidates[0] if candidates else pool_dir

def find_labels_dir(pool_dir):
    candidates = list(pool_dir.rglob("labels"))
    return candidates[0] if candidates else pool_dir

def strict_image_audit(pool_name):
    pool_dir = RAW / pool_name
    images_dir = find_images_dir(pool_dir)
    labels_dir = find_labels_dir(pool_dir)
    rejected_pool = REJECTED / pool_name
    rejected_pool.mkdir(parents=True, exist_ok=True)

    removed = []
    image_files = [p for p in images_dir.iterdir() if p.is_file()]
    for img_path in image_files:
        bad = False
        reason = ""

        if img_path.suffix.lower() not in IMG_EXTS:
            bad = True
            reason = f"unsupported extension {img_path.suffix}"
        else:
            try:
                with Image.open(img_path) as im:
                    fmt = im.format
                    im.verify()
                if fmt not in VALID_PIL_FORMATS:
                    bad = True
                    reason = f"actual image format {fmt} not allowed"
            except Exception as e:
                bad = True
                reason = str(e)

        if bad:
            label_path = labels_dir / f"{img_path.stem}.txt"
            shutil.move(str(img_path), rejected_pool / img_path.name)
            if label_path.exists():
                shutil.move(str(label_path), rejected_pool / label_path.name)
            removed.append({"pool": pool_name, "file": img_path.name, "reason": reason})

    return removed

all_removed = []
for pool in ZIP_MAP:
    all_removed.extend(strict_image_audit(pool))

removed_df = pd.DataFrame(all_removed)
if len(removed_df):
    show_table(removed_df, "v3_yolo26n_rejected_corrupt_or_unsupported_inputs")
else:
    print("No corrupt/unsupported inputs rejected.")

def list_image_label_pairs(pool_name):
    pool_dir = RAW / pool_name
    images_dir = find_images_dir(pool_dir)
    labels_dir = find_labels_dir(pool_dir)
    pairs = []

    for img_path in sorted(images_dir.iterdir()):
        if not img_path.is_file() or img_path.suffix.lower() not in IMG_EXTS:
            continue
        label_path = labels_dir / f"{img_path.stem}.txt"
        if not label_path.exists():
            continue
        pairs.append({"pool": pool_name, "image": img_path, "label": label_path, "stem": img_path.stem})

    return pairs

def yolo_box_area_pct(label_path):
    areas = []
    for line in label_path.read_text().splitlines():
        parts = line.strip().split()
        if len(parts) >= 5:
            try:
                areas.append(float(parts[3]) * float(parts[4]) * 100.0)
            except ValueError:
                pass
    return max(areas) if areas else 0.0

def scale_band_from_area(area_pct):
    if area_pct == 0:
        return "Hard negative"
    if area_pct < 0.25:
        return "Tiny vape"
    if area_pct < 1.0:
        return "Small vape"
    if area_pct < 5.0:
        return "Medium vape"
    return "Large vape"

def add_scale_fields(items):
    out = []
    for item in items:
        area = yolo_box_area_pct(item["label"])
        row = dict(item)
        row["max_box_area_pct"] = area
        row["scale_band"] = scale_band_from_area(area)
        out.append(row)
    return out

items_by_pool = {pool: add_scale_fields(list_image_label_pairs(pool)) for pool in ZIP_MAP}

pool_summary = []
for pool, items in items_by_pool.items():
    pool_summary.append({
        "pool": pool,
        "images_with_labels": len(items),
        "boxes": sum(len([x for x in item["label"].read_text().splitlines() if x.strip()]) for item in items),
        "hard_negative_images": sum(item["scale_band"] == "Hard negative" for item in items),
        "positive_images": sum(item["scale_band"] != "Hard negative" for item in items),
    })
show_table(pd.DataFrame(pool_summary), "v3_yolo26n_input_pool_summary")
backup_all("after input audit")
```

## Cell 3: Native Image Dimension Audit

Run this before training. It characterises source resolution, but it does not change the fixed `imgsz=960` protocol.

```python
dimension_rows = []
dimension_pair_rows = []

for pool, items in items_by_pool.items():
    dims = []
    for item in items:
        try:
            with Image.open(item["image"]) as im:
                w, h = im.size
            dims.append((w, h))
            dimension_pair_rows.append({"pool": pool, "width": w, "height": h})
        except Exception as e:
            print("Could not read dimensions:", item["image"], e)

    if dims:
        widths = np.array([d[0] for d in dims])
        heights = np.array([d[1] for d in dims])
        max_sides = np.maximum(widths, heights)
        min_sides = np.minimum(widths, heights)
        megapixels = widths * heights / 1_000_000
        dimension_rows.append({
            "pool": pool,
            "images": len(dims),
            "median_width": float(np.median(widths)),
            "median_height": float(np.median(heights)),
            "min_width": int(widths.min()),
            "max_width": int(widths.max()),
            "min_height": int(heights.min()),
            "max_height": int(heights.max()),
            "median_max_side": float(np.median(max_sides)),
            "median_min_side": float(np.median(min_sides)),
            "median_megapixels": float(np.median(megapixels)),
            "pct_max_side_gt_960": float((max_sides > 960).mean() * 100),
            "pct_max_side_eq_960": float((max_sides == 960).mean() * 100),
            "pct_max_side_lt_960": float((max_sides < 960).mean() * 100),
        })

dimension_df = pd.DataFrame(dimension_rows)
show_table(dimension_df, "v3_yolo26n_native_image_dimension_summary", max_rows=20)

dimension_pairs_df = (
    pd.DataFrame(dimension_pair_rows)
    .groupby(["pool", "width", "height"], as_index=False)
    .size()
    .rename(columns={"size": "count"})
    .sort_values(["pool", "count"], ascending=[True, False])
    .groupby("pool")
    .head(10)
)
show_table(dimension_pairs_df, "v3_yolo26n_top_native_dimension_pairs_by_pool", max_rows=100)
backup_all("after native image dimension summary")
```

## Cell 4: Create Fixed Splits And V3 Experiments

This cell uses stratified nested source-dose sampling. Each larger dose contains the smaller dose while approximately preserving the source pool's scale-band distribution.

```python
positive_train_items = items_by_pool["positive_train"]
hardneg_train_items = items_by_pool["hard_negative_train"]
positive_test_items = items_by_pool["positive_test"]
hardneg_test_items = items_by_pool["hard_negative_test"]
scraped_items = items_by_pool["scraped_positive"]
synthetic_items = items_by_pool["synthetic_positive"]
copy_paste_items = items_by_pool.get("copy_paste_synthetic", [])

rng = random.Random(SEED)

positive_by_band = defaultdict(list)
for item in positive_train_items:
    positive_by_band[item["scale_band"]].append(item)

positive_train_split = []
positive_val_split = []
target_val_positive = 57
total_positive = len(positive_train_items)

for band, items in sorted(positive_by_band.items()):
    local_items = items[:]
    rng.shuffle(local_items)
    n_val = round(len(local_items) * target_val_positive / total_positive)
    positive_val_split.extend(local_items[:n_val])
    positive_train_split.extend(local_items[n_val:])

rng.shuffle(positive_train_split)
rng.shuffle(positive_val_split)

while len(positive_val_split) > target_val_positive:
    positive_train_split.append(positive_val_split.pop())
while len(positive_val_split) < target_val_positive:
    positive_val_split.append(positive_train_split.pop())

target_val_hardneg = 45
hardneg_train_shuffled = hardneg_train_items[:]
rng.shuffle(hardneg_train_shuffled)
hardneg_val_split = hardneg_train_shuffled[:target_val_hardneg]
hardneg_train_split = hardneg_train_shuffled[target_val_hardneg:]

fixed_val_split = positive_val_split + hardneg_val_split
fixed_test_split = positive_test_items + hardneg_test_items

def stratified_nested_samples(items, doses, strat_col="scale_band", seed=SEED):
    local_rng = random.Random(seed)
    doses = sorted(doses)
    if not items:
        return {n: [] for n in doses}

    by_group = defaultdict(list)
    for item in items:
        by_group[item.get(strat_col, "unknown")].append(item)

    for group_items in by_group.values():
        local_rng.shuffle(group_items)

    total = len(items)
    group_names = sorted(by_group)
    group_targets = {g: len(by_group[g]) / total for g in group_names}
    group_used = {g: 0 for g in group_names}
    ordered = []

    while len(ordered) < total:
        best_group = None
        best_deficit = None
        for g in group_names:
            if group_used[g] >= len(by_group[g]):
                continue
            expected = group_targets[g] * (len(ordered) + 1)
            deficit = expected - group_used[g]
            if best_deficit is None or deficit > best_deficit:
                best_deficit = deficit
                best_group = g
        if best_group is None:
            break
        ordered.append(by_group[best_group][group_used[best_group]])
        group_used[best_group] += 1

    return {n: ordered[:min(n, len(ordered))] for n in doses}

DOSES = [88, 177, 355]
scraped_nested = stratified_nested_samples(scraped_items, DOSES)
synthetic_nested = stratified_nested_samples(synthetic_items, DOSES)
copy_paste_nested = stratified_nested_samples(copy_paste_items, DOSES) if copy_paste_items else {}

V3_EXPERIMENTS = {
    "V3_E1_real_hardneg_baseline": {
        "short": "V3-E1",
        "pathway": "baseline",
        "dose": 0,
        "train": positive_train_split + hardneg_train_split,
        "val": fixed_val_split,
        "test": fixed_test_split,
        "description": "Operational baseline: real positives + lighter-based hard negatives.",
    },
    "V3_E2_scraped_088": {
        "short": "V3-E2",
        "pathway": "scraped",
        "dose": 88,
        "train": positive_train_split + hardneg_train_split + scraped_nested[88],
        "val": fixed_val_split,
        "test": fixed_test_split,
        "description": "Low-dose curated scraped positives.",
    },
    "V3_E3_scraped_177": {
        "short": "V3-E3",
        "pathway": "scraped",
        "dose": 177,
        "train": positive_train_split + hardneg_train_split + scraped_nested[177],
        "val": fixed_val_split,
        "test": fixed_test_split,
        "description": "Medium-dose curated scraped positives.",
    },
    "V3_E4_scraped_355": {
        "short": "V3-E4",
        "pathway": "scraped",
        "dose": 355,
        "train": positive_train_split + hardneg_train_split + scraped_nested[355],
        "val": fixed_val_split,
        "test": fixed_test_split,
        "description": "Full matched-dose curated scraped positives.",
    },
    "V3_E5_synthetic_088": {
        "short": "V3-E5",
        "pathway": "pure_synthetic",
        "dose": 88,
        "train": positive_train_split + hardneg_train_split + synthetic_nested[88],
        "val": fixed_val_split,
        "test": fixed_test_split,
        "description": "Low-dose pure synthetic positives.",
    },
    "V3_E6_synthetic_177": {
        "short": "V3-E6",
        "pathway": "pure_synthetic",
        "dose": 177,
        "train": positive_train_split + hardneg_train_split + synthetic_nested[177],
        "val": fixed_val_split,
        "test": fixed_test_split,
        "description": "Medium-dose pure synthetic positives.",
    },
    "V3_E7_synthetic_355": {
        "short": "V3-E7",
        "pathway": "pure_synthetic",
        "dose": 355,
        "train": positive_train_split + hardneg_train_split + synthetic_nested[355],
        "val": fixed_val_split,
        "test": fixed_test_split,
        "description": "Full-dose pure synthetic positives.",
    },
    "V3_E8_scraped177_synthetic177": {
        "short": "V3-E8",
        "pathway": "scraped_plus_synthetic",
        "dose": 354,
        "train": positive_train_split + hardneg_train_split + scraped_nested[177] + synthetic_nested[177],
        "val": fixed_val_split,
        "test": fixed_test_split,
        "description": "Combined medium-dose source test: 177 scraped positives + 177 pure synthetic positives.",
    },
}

if copy_paste_items:
    for idx, dose in enumerate(DOSES, start=9):
        V3_EXPERIMENTS[f"V3_E{idx}_copy_paste_{dose:03d}"] = {
            "short": f"V3-E{idx}",
            "pathway": "copy_paste_synthetic",
            "dose": dose,
            "train": positive_train_split + hardneg_train_split + copy_paste_nested[dose],
            "val": fixed_val_split,
            "test": fixed_test_split,
            "description": f"{dose}-image real-instance copy-paste synthetic positives.",
        }

print("Fixed split sizes")
print("positive train:", len(positive_train_split))
print("positive val:", len(positive_val_split))
print("hard-negative train:", len(hardneg_train_split))
print("hard-negative val:", len(hardneg_val_split))
print("test positives:", len(positive_test_items))
print("test hard negatives:", len(hardneg_test_items))

experiment_plan_df = pd.DataFrame([
    {
        "experiment": name,
        "short": cfg["short"],
        "pathway": cfg["pathway"],
        "dose": cfg["dose"],
        "train_images": len(cfg["train"]),
        "val_images": len(cfg["val"]),
        "test_images": len(cfg["test"]),
        "description": cfg["description"],
    }
    for name, cfg in V3_EXPERIMENTS.items()
])
show_table(experiment_plan_df, "v3_yolo26n_experiment_plan", max_rows=20)
```

## Cell 5: Materialise YOLO Datasets

```python
def copy_items_to_split(items, dataset_dir, split):
    images_out = dataset_dir / split / "images"
    labels_out = dataset_dir / split / "labels"
    images_out.mkdir(parents=True, exist_ok=True)
    labels_out.mkdir(parents=True, exist_ok=True)

    copied = []
    for item in items:
        prefix = item["pool"]
        new_stem = f"{prefix}__{item['image'].stem}"
        img_out = images_out / f"{new_stem}{item['image'].suffix.lower()}"
        label_out = labels_out / f"{new_stem}.txt"

        shutil.copy2(item["image"], img_out)
        shutil.copy2(item["label"], label_out)

        boxes = len([x for x in item["label"].read_text().splitlines() if x.strip()])
        copied.append({
            "split": split,
            "source_pool": item["pool"],
            "image": img_out.name,
            "label": label_out.name,
            "boxes": boxes,
            "scale_band": item["scale_band"],
            "max_box_area_pct": item["max_box_area_pct"],
            "hard_negative": item["scale_band"] == "Hard negative",
        })
    return copied

all_manifest_rows = []

for exp_name, cfg in V3_EXPERIMENTS.items():
    dataset_dir = DATASETS / exp_name
    if dataset_dir.exists():
        shutil.rmtree(dataset_dir)
    dataset_dir.mkdir(parents=True, exist_ok=True)

    rows = []
    rows.extend(copy_items_to_split(cfg["train"], dataset_dir, "train"))
    rows.extend(copy_items_to_split(cfg["val"], dataset_dir, "val"))
    rows.extend(copy_items_to_split(cfg["test"], dataset_dir, "test"))

    for row in rows:
        row["experiment"] = exp_name
        row["experiment_short"] = cfg["short"]
        row["pathway"] = cfg["pathway"]
        row["dose"] = cfg["dose"]
    all_manifest_rows.extend(rows)

    yaml_text = f"""path: {dataset_dir}
train: train/images
val: val/images
test: test/images
names:
  0: vape
"""
    (dataset_dir / f"{exp_name}.yaml").write_text(yaml_text)

manifest_df = pd.DataFrame(all_manifest_rows)
show_table(manifest_df, "v3_yolo26n_manifest", max_rows=20)
manifest_df.to_csv(REPORTS / "v3_yolo26n_manifest_full.csv", index=False)

composition_df = (
    manifest_df
    .groupby(["experiment_short", "experiment", "pathway", "dose", "split", "source_pool"], as_index=False)
    .agg(
        images=("image", "count"),
        boxes=("boxes", "sum"),
        hard_negative_images=("hard_negative", "sum"),
    )
)
composition_df["positive_images"] = composition_df["images"] - composition_df["hard_negative_images"]
show_table(composition_df, "v3_yolo26n_dataset_composition_by_source", max_rows=200)

scale_comp_df = (
    manifest_df
    .groupby(["experiment_short", "experiment", "pathway", "dose", "split", "scale_band"], as_index=False)
    .agg(images=("image", "count"), boxes=("boxes", "sum"))
)
split_totals = scale_comp_df.groupby(["experiment", "split"])["images"].transform("sum")
scale_comp_df["percent"] = scale_comp_df["images"] / split_totals * 100.0
show_table(scale_comp_df, "v3_yolo26n_scale_band_composition", max_rows=250)

backup_all("after dataset materialisation")
```

## Cell 5B: Add V3-E8 Only In An Existing Notebook

Run this **only if you already ran the old Cells 4-5 before V3-E8 existed**. This registers and materialises just the new combined medium-dose dataset without rebuilding E1-E7.

If you rerun the updated Cells 4-5 from scratch, skip this cell.

```python
E8_NAME = "V3_E8_scraped177_synthetic177"

V3_EXPERIMENTS[E8_NAME] = {
    "short": "V3-E8",
    "pathway": "scraped_plus_synthetic",
    "dose": 354,
    "train": positive_train_split + hardneg_train_split + scraped_nested[177] + synthetic_nested[177],
    "val": fixed_val_split,
    "test": fixed_test_split,
    "description": "Combined medium-dose source test: 177 scraped positives + 177 pure synthetic positives.",
}

dataset_dir = DATASETS / E8_NAME
if dataset_dir.exists():
    shutil.rmtree(dataset_dir)
dataset_dir.mkdir(parents=True, exist_ok=True)

cfg = V3_EXPERIMENTS[E8_NAME]
rows = []
rows.extend(copy_items_to_split(cfg["train"], dataset_dir, "train"))
rows.extend(copy_items_to_split(cfg["val"], dataset_dir, "val"))
rows.extend(copy_items_to_split(cfg["test"], dataset_dir, "test"))

for row in rows:
    row["experiment"] = E8_NAME
    row["experiment_short"] = cfg["short"]
    row["pathway"] = cfg["pathway"]
    row["dose"] = cfg["dose"]

yaml_text = f"""path: {dataset_dir}
train: train/images
val: val/images
test: test/images
names:
  0: vape
"""
(dataset_dir / f"{E8_NAME}.yaml").write_text(yaml_text)

e8_manifest_df = pd.DataFrame(rows)
show_table(e8_manifest_df, "v3_yolo26n_E8_manifest", max_rows=30)

if "manifest_df" in globals() and len(manifest_df):
    manifest_df = manifest_df[manifest_df["experiment"] != E8_NAME]
    manifest_df = pd.concat([manifest_df, e8_manifest_df], ignore_index=True)
else:
    manifest_df = e8_manifest_df.copy()

manifest_df.to_csv(REPORTS / "v3_yolo26n_manifest_full.csv", index=False)

composition_df = (
    manifest_df
    .groupby(["experiment_short", "experiment", "pathway", "dose", "split", "source_pool"], as_index=False)
    .agg(
        images=("image", "count"),
        boxes=("boxes", "sum"),
        hard_negative_images=("hard_negative", "sum"),
    )
)
composition_df["positive_images"] = composition_df["images"] - composition_df["hard_negative_images"]
show_table(composition_df[composition_df["experiment"] == E8_NAME], "v3_yolo26n_E8_dataset_composition_by_source", max_rows=50)

scale_comp_df = (
    manifest_df
    .groupby(["experiment_short", "experiment", "pathway", "dose", "split", "scale_band"], as_index=False)
    .agg(images=("image", "count"), boxes=("boxes", "sum"))
)
split_totals = scale_comp_df.groupby(["experiment", "split"])["images"].transform("sum")
scale_comp_df["percent"] = scale_comp_df["images"] / split_totals * 100.0
show_table(scale_comp_df[scale_comp_df["experiment"] == E8_NAME], "v3_yolo26n_E8_scale_band_composition", max_rows=80)

backup_all("after E8 dataset materialisation")
print("V3-E8 ready. Now run Cell 6H.")
```

## Cell 6: Define Training And Evaluation Helpers

This cell does not train anything. Run it once, then run one experiment cell at a time.

```python
from ultralytics import YOLO
import torch

TRAIN_CONFIG = {
    "model_weights": "yolo26n.pt",
    "imgsz": 960,
    "epochs": 100,
    "patience": 100,
    "batch": 16,
    "device": 0,
    "workers": 2,
    "optimizer": "auto",
    "cos_lr": True,
    "close_mosaic": 10,
    "seed": 42,
}

def latest_run_dir(base_name):
    candidates = sorted(RUNS.glob(f"{base_name}*"), key=lambda p: p.stat().st_mtime)
    return candidates[-1] if candidates else RUNS / base_name

def extract_box_metrics(metrics):
    box = metrics.box
    return {
        "precision": float(box.mp),
        "recall": float(box.mr),
        "mAP50": float(box.map50),
        "mAP50_95": float(box.map),
        "fitness": float(box.map),
    }

def read_best_epoch_from_results(results_csv):
    if not results_csv.exists():
        return {}
    df = pd.read_csv(results_csv)
    df.columns = [c.strip() for c in df.columns]
    map_col = "metrics/mAP50-95(B)"
    if map_col not in df.columns:
        return {}
    best_idx = df[map_col].idxmax()
    best = df.loc[best_idx]
    last = df.iloc[-1]
    return {
        "best_epoch_by_val_mAP50_95": int(best["epoch"]),
        "best_val_precision": float(best.get("metrics/precision(B)", np.nan)),
        "best_val_recall": float(best.get("metrics/recall(B)", np.nan)),
        "best_val_mAP50": float(best.get("metrics/mAP50(B)", np.nan)),
        "best_val_mAP50_95": float(best.get("metrics/mAP50-95(B)", np.nan)),
        "last_epoch": int(last["epoch"]),
        "last_val_precision": float(last.get("metrics/precision(B)", np.nan)),
        "last_val_recall": float(last.get("metrics/recall(B)", np.nan)),
        "last_val_mAP50": float(last.get("metrics/mAP50(B)", np.nan)),
        "last_val_mAP50_95": float(last.get("metrics/mAP50-95(B)", np.nan)),
    }

def train_experiment(exp_name, cfg):
    yaml_path = DATASETS / exp_name / f"{exp_name}.yaml"
    run_name = f"{exp_name}_{Path(TRAIN_CONFIG['model_weights']).stem}_img{TRAIN_CONFIG['imgsz']}_b{TRAIN_CONFIG['batch']}"

    print(f"\nTraining {cfg['short']} | {exp_name}")
    print("YAML:", yaml_path)
    print("Run name:", run_name)

    start = time.time()
    model = YOLO(TRAIN_CONFIG["model_weights"])
    model.train(
        data=str(yaml_path),
        project=str(RUNS),
        name=run_name,
        imgsz=TRAIN_CONFIG["imgsz"],
        epochs=TRAIN_CONFIG["epochs"],
        patience=TRAIN_CONFIG["patience"],
        batch=TRAIN_CONFIG["batch"],
        device=TRAIN_CONFIG["device"],
        workers=TRAIN_CONFIG["workers"],
        optimizer=TRAIN_CONFIG["optimizer"],
        cos_lr=TRAIN_CONFIG["cos_lr"],
        close_mosaic=TRAIN_CONFIG["close_mosaic"],
        seed=TRAIN_CONFIG["seed"],
        deterministic=True,
        plots=True,
        val=True,
    )
    elapsed_minutes = (time.time() - start) / 60
    run_dir = latest_run_dir(run_name)
    row = {
        "experiment": exp_name,
        "experiment_short": cfg["short"],
        "pathway": cfg["pathway"],
        "dose": cfg["dose"],
        "model": TRAIN_CONFIG["model_weights"],
        "imgsz": TRAIN_CONFIG["imgsz"],
        "run_name": run_dir.name,
        "run_dir": str(run_dir),
        "best_weights": str(run_dir / "weights" / "best.pt"),
        "last_weights": str(run_dir / "weights" / "last.pt"),
        "elapsed_minutes": elapsed_minutes,
        "best_exists": (run_dir / "weights" / "best.pt").exists(),
        "last_exists": (run_dir / "weights" / "last.pt").exists(),
        **read_best_epoch_from_results(run_dir / "results.csv"),
    }
    show_table(pd.DataFrame([row]), f"v3_yolo26n_{cfg['short'].replace('-', '_')}_training_record")
    backup_all(f"after training {cfg['short']}")
    return row

def evaluate_experiment(exp_name, cfg, weights_path):
    yaml_path = DATASETS / exp_name / f"{exp_name}.yaml"
    eval_name = f"v3_yolo26n_{cfg['short'].replace('-', '_')}_official_test_eval"
    print(f"\nEvaluating {cfg['short']} on fixed test set")

    model = YOLO(str(weights_path))
    metrics = model.val(
        data=str(yaml_path),
        split="test",
        imgsz=TRAIN_CONFIG["imgsz"],
        batch=TRAIN_CONFIG["batch"],
        device=TRAIN_CONFIG["device"],
        project=str(RUNS),
        name=eval_name,
        plots=True,
    )
    row = {
        "experiment": exp_name,
        "experiment_short": cfg["short"],
        "pathway": cfg["pathway"],
        "dose": cfg["dose"],
        "model": TRAIN_CONFIG["model_weights"],
        "imgsz": TRAIN_CONFIG["imgsz"],
        "checkpoint": "best.pt",
        "weights": str(weights_path),
        "eval_dir": str(RUNS / eval_name),
        "test_images": len(cfg["test"]),
        "test_positive_images": len(positive_test_items),
        "test_hard_negative_images": len(hardneg_test_items),
        "test_boxes": sum(len([x for x in item["label"].read_text().splitlines() if x.strip()]) for item in positive_test_items),
        **extract_box_metrics(metrics),
    }
    show_table(pd.DataFrame([row]), f"v3_yolo26n_{cfg['short'].replace('-', '_')}_official_test_metrics")
    backup_all(f"after evaluating {cfg['short']}")
    return row

def append_or_replace_csv(row, csv_path, key_cols):
    csv_path = Path(csv_path)
    new_df = pd.DataFrame([row])
    if csv_path.exists():
        old_df = pd.read_csv(csv_path)
        for col in new_df.columns:
            if col not in old_df.columns:
                old_df[col] = np.nan
        for col in old_df.columns:
            if col not in new_df.columns:
                new_df[col] = np.nan
        old_key = old_df[key_cols].astype(str).agg("|".join, axis=1)
        new_key = new_df[key_cols].astype(str).agg("|".join, axis=1).iloc[0]
        combined = old_df[old_key != new_key]
        combined = pd.concat([combined, new_df[old_df.columns]], ignore_index=True)
    else:
        combined = new_df
    combined.to_csv(csv_path, index=False)
    return combined

def run_one_v3_experiment(exp_name):
    cfg = V3_EXPERIMENTS[exp_name]
    train_row = train_experiment(exp_name, cfg)
    training_summary_df = append_or_replace_csv(
        train_row,
        REPORTS / "v3_yolo26n_training_checkpoint_summary_incremental.csv",
        ["experiment"],
    )

    best_weights = Path(train_row["best_weights"])
    if not best_weights.exists():
        raise FileNotFoundError(f"Missing best weights after training: {best_weights}")

    test_row = evaluate_experiment(exp_name, cfg, best_weights)
    official_test_df = append_or_replace_csv(
        test_row,
        REPORTS / "v3_yolo26n_official_test_metrics_incremental.csv",
        ["experiment"],
    )

    show_table(training_summary_df.sort_values("experiment_short"), "v3_yolo26n_training_checkpoint_summary_incremental", max_rows=50)
    show_table(official_test_df.sort_values("experiment_short"), "v3_yolo26n_official_test_metrics_incremental", max_rows=50)
    backup_all(f"after complete train/eval cycle for {cfg['short']}")
    return train_row, test_row

def load_incremental_summaries():
    training_path = REPORTS / "v3_yolo26n_training_checkpoint_summary_incremental.csv"
    test_path = REPORTS / "v3_yolo26n_official_test_metrics_incremental.csv"
    training_summary_df = pd.read_csv(training_path) if training_path.exists() else pd.DataFrame()
    official_test_df = pd.read_csv(test_path) if test_path.exists() else pd.DataFrame()
    if len(training_summary_df):
        show_table(training_summary_df.sort_values("experiment_short"), "v3_yolo26n_training_checkpoint_summary_incremental", max_rows=50)
    else:
        print("No incremental training summary found yet.")
    if len(official_test_df):
        show_table(official_test_df.sort_values("experiment_short"), "v3_yolo26n_official_test_metrics_incremental", max_rows=50)
    else:
        print("No incremental official test summary found yet.")
    return training_summary_df, official_test_df

print("Training config:")
print(json.dumps(TRAIN_CONFIG, indent=2))
print("CUDA available:", torch.cuda.is_available())
if torch.cuda.is_available():
    print("GPU:", torch.cuda.get_device_name(0))
print("\nHelpers ready. Now run one V3 experiment cell at a time.")
```

## Cell 6A: Run V3-E1 Baseline Only

Run this cell, paste the outputs for interpretation, and confirm Drive backup before moving to V3-E2.

```python
train_row, test_row = run_one_v3_experiment("V3_E1_real_hardneg_baseline")
```

## Cell 6B: Run V3-E2 Scraped 88 Only

```python
train_row, test_row = run_one_v3_experiment("V3_E2_scraped_088")
```

## Cell 6C: Run V3-E3 Scraped 177 Only

```python
train_row, test_row = run_one_v3_experiment("V3_E3_scraped_177")
```

## Cell 6D: Run V3-E4 Scraped 355 Only

```python
train_row, test_row = run_one_v3_experiment("V3_E4_scraped_355")
```

## Cell 6E: Run V3-E5 Synthetic 88 Only

```python
train_row, test_row = run_one_v3_experiment("V3_E5_synthetic_088")
```

## Cell 6F: Run V3-E6 Synthetic 177 Only

```python
train_row, test_row = run_one_v3_experiment("V3_E6_synthetic_177")
```

## Cell 6G: Run V3-E7 Synthetic 355 Only

```python
train_row, test_row = run_one_v3_experiment("V3_E7_synthetic_355")
```

## Cell 6H: Run V3-E8 Combined Medium Scraped + Synthetic Only

Run this after E1-E7 if you want to test the medium-dose complementarity hypothesis.

```python
train_row, test_row = run_one_v3_experiment("V3_E8_scraped177_synthetic177")
```

## Cell 6I: Reload Incremental Summaries

Use this after any runtime interruption.

```python
training_summary_df, official_test_df = load_incremental_summaries()
```

## Cell 7: Essential Backup Check

Run after any completed train/eval cycle if you want to verify that the minimum files are on Drive.

```python
essential_paths = [
    DRIVE_OUTPUTS / "reports" / "v3_yolo26n_training_checkpoint_summary_incremental.csv",
    DRIVE_OUTPUTS / "reports" / "v3_yolo26n_official_test_metrics_incremental.csv",
]

for exp_name, cfg in V3_EXPERIMENTS.items():
    run_prefix = f"{exp_name}_{Path(TRAIN_CONFIG['model_weights']).stem}_img{TRAIN_CONFIG['imgsz']}_b{TRAIN_CONFIG['batch']}"
    matching = sorted((DRIVE_OUTPUTS / "runs").glob(f"{run_prefix}*")) if (DRIVE_OUTPUTS / "runs").exists() else []
    if matching:
        run_dir = matching[-1]
        essential_paths.extend([
            run_dir / "weights" / "best.pt",
            run_dir / "weights" / "last.pt",
        ])
    yaml_path = DRIVE_OUTPUTS / "datasets" / exp_name / f"{exp_name}.yaml"
    if yaml_path.exists():
        essential_paths.append(yaml_path)

check_rows = []
for p in essential_paths:
    check_rows.append({
        "exists": p.exists(),
        "size_mb": round(p.stat().st_size / (1024 * 1024), 3) if p.exists() else np.nan,
        "path": str(p),
    })

backup_check_df = pd.DataFrame(check_rows)
show_table(backup_check_df, "v3_yolo26n_essential_backup_check", max_rows=200)

if backup_check_df["exists"].all():
    print("Essential YOLO26n backup files are present in Google Drive.")
else:
    print("Some essential files are missing. Check paths above before closing the runtime.")
```

## Cell 8: Prediction-Level Diagnostics

Run this only after at least one experiment has completed. It can be rerun after each new experiment.

```python
training_summary_df, official_test_df = load_incremental_summaries()

def xywhn_to_xyxy(box, img_w, img_h):
    x, y, w, h = box
    x1 = (x - w / 2) * img_w
    y1 = (y - h / 2) * img_h
    x2 = (x + w / 2) * img_w
    y2 = (y + h / 2) * img_h
    return [x1, y1, x2, y2]

def box_iou(a, b):
    ax1, ay1, ax2, ay2 = a
    bx1, by1, bx2, by2 = b
    ix1, iy1 = max(ax1, bx1), max(ay1, by1)
    ix2, iy2 = min(ax2, bx2), min(ay2, by2)
    iw, ih = max(0, ix2 - ix1), max(0, iy2 - iy1)
    inter = iw * ih
    area_a = max(0, ax2 - ax1) * max(0, ay2 - ay1)
    area_b = max(0, bx2 - bx1) * max(0, by2 - by1)
    denom = area_a + area_b - inter
    return inter / denom if denom else 0.0

def read_gt_boxes(label_path, image_path):
    with Image.open(image_path) as im:
        img_w, img_h = im.size
    boxes = []
    for line in label_path.read_text().splitlines():
        parts = line.strip().split()
        if len(parts) >= 5:
            boxes.append(xywhn_to_xyxy([float(x) for x in parts[1:5]], img_w, img_h))
    return boxes

def prediction_diagnostics_for_experiment(row, conf=0.001, iou_match=0.5):
    exp_name = row["experiment"]
    cfg = V3_EXPERIMENTS[exp_name]
    weights = Path(row["best_weights"])
    model = YOLO(str(weights))

    test_images_dir = DATASETS / exp_name / "test" / "images"
    test_labels_dir = DATASETS / exp_name / "test" / "labels"
    image_paths = sorted([p for p in test_images_dir.iterdir() if p.suffix.lower() in IMG_EXTS])

    pred_rows = []
    image_rows = []

    results = model.predict(
        source=[str(p) for p in image_paths],
        imgsz=TRAIN_CONFIG["imgsz"],
        conf=conf,
        iou=0.7,
        device=TRAIN_CONFIG["device"],
        verbose=False,
    )

    for img_path, result in zip(image_paths, results):
        label_path = test_labels_dir / f"{img_path.stem}.txt"
        gt_boxes = read_gt_boxes(label_path, img_path)
        matched_gt = set()
        is_hard_negative = len(gt_boxes) == 0

        preds = []
        if result.boxes is not None and len(result.boxes):
            xyxy = result.boxes.xyxy.cpu().numpy()
            confs = result.boxes.conf.cpu().numpy()
            for pred_idx, (bbox, score) in enumerate(zip(xyxy, confs)):
                best_iou = 0.0
                best_gt = None
                for gt_idx, gt in enumerate(gt_boxes):
                    if gt_idx in matched_gt:
                        continue
                    iou = box_iou(bbox.tolist(), gt)
                    if iou > best_iou:
                        best_iou = iou
                        best_gt = gt_idx
                is_tp = best_iou >= iou_match and best_gt is not None
                if is_tp:
                    matched_gt.add(best_gt)
                preds.append({
                    "experiment": exp_name,
                    "experiment_short": cfg["short"],
                    "pathway": cfg["pathway"],
                    "dose": cfg["dose"],
                    "image": img_path.name,
                    "hard_negative_image": is_hard_negative,
                    "prediction_index": pred_idx,
                    "confidence": float(score),
                    "iou_to_best_gt": float(best_iou),
                    "prediction_type": "TP" if is_tp else "FP",
                    "x1": float(bbox[0]),
                    "y1": float(bbox[1]),
                    "x2": float(bbox[2]),
                    "y2": float(bbox[3]),
                })
        pred_rows.extend(preds)

        scale_matches = manifest_df[
            (manifest_df["experiment"] == exp_name)
            & (manifest_df["split"] == "test")
            & (manifest_df["image"] == img_path.name)
        ]
        image_rows.append({
            "experiment": exp_name,
            "experiment_short": cfg["short"],
            "pathway": cfg["pathway"],
            "dose": cfg["dose"],
            "image": img_path.name,
            "gt_boxes": len(gt_boxes),
            "hard_negative_image": is_hard_negative,
            "predictions": len(preds),
            "tp_predictions": sum(p["prediction_type"] == "TP" for p in preds),
            "fp_predictions": sum(p["prediction_type"] == "FP" for p in preds),
            "false_negatives": len(gt_boxes) - len(matched_gt),
            "scale_band": scale_matches["scale_band"].iloc[0] if len(scale_matches) else "unknown",
        })

    return pd.DataFrame(pred_rows), pd.DataFrame(image_rows)

all_pred_dfs = []
all_image_dfs = []

for _, row in training_summary_df.iterrows():
    pred_df, image_df = prediction_diagnostics_for_experiment(row)
    all_pred_dfs.append(pred_df)
    all_image_dfs.append(image_df)

prediction_df = pd.concat(all_pred_dfs, ignore_index=True) if all_pred_dfs else pd.DataFrame()
image_eval_df = pd.concat(all_image_dfs, ignore_index=True) if all_image_dfs else pd.DataFrame()

show_table(prediction_df, "v3_yolo26n_prediction_level_table", max_rows=40)
prediction_df.to_csv(REPORTS / "v3_yolo26n_prediction_level_table_full.csv", index=False)
show_table(image_eval_df, "v3_yolo26n_image_level_eval_table", max_rows=40)
image_eval_df.to_csv(REPORTS / "v3_yolo26n_image_level_eval_table_full.csv", index=False)

hardneg_fp_df = (
    image_eval_df[image_eval_df["hard_negative_image"]]
    .groupby(["experiment_short", "experiment", "pathway", "dose"], as_index=False)
    .agg(
        hard_negative_test_images=("image", "count"),
        false_positive_predictions=("fp_predictions", "sum"),
        hard_negative_images_with_at_least_one_fp=("fp_predictions", lambda s: int((s > 0).sum())),
    )
)
hardneg_fp_df["fp_per_hard_negative_image"] = hardneg_fp_df["false_positive_predictions"] / hardneg_fp_df["hard_negative_test_images"]

if len(prediction_df):
    fp_conf = (
        prediction_df[prediction_df["hard_negative_image"] & (prediction_df["prediction_type"] == "FP")]
        .groupby("experiment")["confidence"]
        .max()
        .rename("max_fp_confidence")
        .reset_index()
    )
    hardneg_fp_df = hardneg_fp_df.merge(fp_conf, on="experiment", how="left")

show_table(hardneg_fp_df, "v3_yolo26n_hard_negative_false_positive_summary", max_rows=50)
hardneg_fp_df.to_csv(REPORTS / "v3_yolo26n_hard_negative_false_positive_summary.csv", index=False)

scale_perf_df = (
    image_eval_df[~image_eval_df["hard_negative_image"]]
    .groupby(["experiment_short", "experiment", "pathway", "dose", "scale_band"], as_index=False)
    .agg(
        images=("image", "count"),
        gt_boxes=("gt_boxes", "sum"),
        tp_predictions=("tp_predictions", "sum"),
        false_negatives=("false_negatives", "sum"),
        fp_predictions=("fp_predictions", "sum"),
    )
)
scale_perf_df["recall"] = scale_perf_df["tp_predictions"] / scale_perf_df["gt_boxes"].replace(0, np.nan)
scale_perf_df["precision"] = scale_perf_df["tp_predictions"] / (scale_perf_df["tp_predictions"] + scale_perf_df["fp_predictions"]).replace(0, np.nan)
scale_perf_df["f1"] = 2 * scale_perf_df["precision"] * scale_perf_df["recall"] / (scale_perf_df["precision"] + scale_perf_df["recall"]).replace(0, np.nan)
show_table(scale_perf_df, "v3_yolo26n_scale_band_performance", max_rows=100)
scale_perf_df.to_csv(REPORTS / "v3_yolo26n_scale_band_performance.csv", index=False)

confidence_summary_df = (
    prediction_df
    .groupby(["experiment_short", "experiment", "pathway", "dose", "prediction_type"], as_index=False)
    .agg(
        predictions=("confidence", "count"),
        mean_confidence=("confidence", "mean"),
        median_confidence=("confidence", "median"),
        p10_confidence=("confidence", lambda s: s.quantile(0.10)),
        p90_confidence=("confidence", lambda s: s.quantile(0.90)),
    )
)
show_table(confidence_summary_df, "v3_yolo26n_tp_fp_confidence_summary", max_rows=100)
confidence_summary_df.to_csv(REPORTS / "v3_yolo26n_tp_fp_confidence_summary.csv", index=False)
backup_all("after prediction diagnostics")
```

## Cell 9: Threshold Diagnostics, Source-Dose Tables, Excel Workbook

```python
THRESHOLDS = [0.05, 0.10, 0.15, 0.20, 0.25, 0.30, 0.40, 0.50, 0.60, 0.70]

threshold_rows = []
for exp_name, cfg in V3_EXPERIMENTS.items():
    exp_preds = prediction_df[prediction_df["experiment"] == exp_name].copy()
    exp_images = image_eval_df[image_eval_df["experiment"] == exp_name].copy()
    if not len(exp_images):
        continue
    total_gt = exp_images["gt_boxes"].sum()
    hardneg_images = exp_images["hard_negative_image"].sum()

    for t in THRESHOLDS:
        kept = exp_preds[exp_preds["confidence"] >= t]
        tp = int((kept["prediction_type"] == "TP").sum()) if len(kept) else 0
        fp = int((kept["prediction_type"] == "FP").sum()) if len(kept) else 0
        precision = tp / (tp + fp) if (tp + fp) else np.nan
        recall = tp / total_gt if total_gt else np.nan
        f1 = 2 * precision * recall / (precision + recall) if precision == precision and recall == recall and (precision + recall) else np.nan
        hardneg_fp = int((kept["hard_negative_image"] & (kept["prediction_type"] == "FP")).sum()) if len(kept) else 0
        threshold_rows.append({
            "experiment": exp_name,
            "experiment_short": cfg["short"],
            "pathway": cfg["pathway"],
            "dose": cfg["dose"],
            "threshold": t,
            "precision": precision,
            "recall": recall,
            "f1": f1,
            "false_positive_predictions": fp,
            "false_positives_per_hard_negative_image": hardneg_fp / hardneg_images if hardneg_images else np.nan,
            "false_negatives": total_gt - tp,
        })

threshold_df = pd.DataFrame(threshold_rows)
show_table(threshold_df, "v3_yolo26n_threshold_diagnostics", max_rows=200)

source_dose_df = official_test_df.copy()
if len(source_dose_df) and (source_dose_df["pathway"] == "baseline").any():
    baseline = source_dose_df[source_dose_df["pathway"] == "baseline"].iloc[0]
    for metric in ["precision", "recall", "mAP50", "mAP50_95"]:
        source_dose_df[f"{metric}_delta_vs_baseline"] = source_dose_df[metric] - baseline[metric]
    source_dose_df["dose_percent_of_baseline_train_images"] = source_dose_df["dose"] / len(V3_EXPERIMENTS["V3_E1_real_hardneg_baseline"]["train"]) * 100.0
show_table(source_dose_df, "v3_yolo26n_source_dose_effect", max_rows=50)

val_test_gap_df = training_summary_df.merge(
    official_test_df[["experiment", "precision", "recall", "mAP50", "mAP50_95"]],
    on="experiment",
    how="left",
) if len(training_summary_df) and len(official_test_df) else pd.DataFrame()

if len(val_test_gap_df):
    for metric, val_col in [
        ("precision", "best_val_precision"),
        ("recall", "best_val_recall"),
        ("mAP50", "best_val_mAP50"),
        ("mAP50_95", "best_val_mAP50_95"),
    ]:
        if val_col in val_test_gap_df.columns:
            val_test_gap_df[f"{metric}_gap_test_minus_val"] = val_test_gap_df[metric] - val_test_gap_df[val_col]
show_table(val_test_gap_df, "v3_yolo26n_validation_test_gap_summary", max_rows=50)

EXCEL_PATH = REPORTS / "v3_yolo26n_post_analysis_workbook.xlsx"
with pd.ExcelWriter(EXCEL_PATH, engine="xlsxwriter") as writer:
    experiment_plan_df.to_excel(writer, sheet_name="experiment_plan", index=False)
    composition_df.to_excel(writer, sheet_name="dataset_composition", index=False)
    scale_comp_df.to_excel(writer, sheet_name="scale_composition", index=False)
    training_summary_df.to_excel(writer, sheet_name="training_summary", index=False)
    official_test_df.to_excel(writer, sheet_name="official_test_metrics", index=False)
    source_dose_df.to_excel(writer, sheet_name="source_dose_effect", index=False)
    val_test_gap_df.to_excel(writer, sheet_name="val_test_gaps", index=False)
    hardneg_fp_df.to_excel(writer, sheet_name="hardneg_fp", index=False)
    scale_perf_df.to_excel(writer, sheet_name="scale_performance", index=False)
    confidence_summary_df.to_excel(writer, sheet_name="confidence_summary", index=False)
    threshold_df.to_excel(writer, sheet_name="threshold_diagnostics", index=False)
    image_eval_df.to_excel(writer, sheet_name="image_eval", index=False)
    prediction_df.head(5000).to_excel(writer, sheet_name="predictions_sample", index=False)

print("Saved workbook:", EXCEL_PATH)
backup_all("after final workbook")
```

## Recommended Run Order

1. Run Cell 0 if you want a clean YOLO26n restart.
2. Run Cells 1-6.
3. Run Cell 6A and paste the E1 output for interpretation.
4. After backup completes, run Cell 7 to verify essentials.
5. Continue one experiment at a time from Cell 6B onward.
6. Run Cell 8 and Cell 9 after enough experiments exist for diagnostics.
