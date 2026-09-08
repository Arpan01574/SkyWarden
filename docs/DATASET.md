# SkyWarden Dataset Guide

This document provides complete documentation for the dataset utilized in training and evaluating the SkyWarden aircraft detection and identification system.

---

## 1. Dataset Overview

The project utilizes the [Military Aircraft Detection Dataset](https://www.kaggle.com/datasets/a2015003713/militaryaircraftdetectiondataset/data) hosted on Kaggle. The dataset contains high-resolution aerial and ground-based imagery capturing a wide diversity of military aircraft in varied lighting conditions, angles, and operational environments.

- **Total Classes:** 77 distinct aircraft types
- **Domain:** Military fixed-wing aircraft, bombers, transports, reconnaissance UAVs, and rotorcraft
- **Primary Task:** Multi-class object detection and categorization
- **Secondary Task:** Friend / Enemy identification based on allegiance mapping

---

## 2. Aircraft Classes & Categorization

The 77 aircraft classes recognized by the model include:

### Combat & Fighter Aircraft
- `A10`, `A29`, `AV8B`, `B1`, `B2`, `B52`
- `F117`, `F14`, `F15`, `F16`, `F18` (FA18), `F22`, `F35`, `F4`
- `J10`, `J20`, `JH7`, `JAS39`, `JF17`
- `Mirage2000`, `Mig29`, `Mig31`, `Su24`, `Su25`, `Su27`, `Su34`, `Su35`, `Su57`
- `Rafale`, `Tornado`, `Typhoon`, `Vulcan`

### Transport, Patrol & Airborne Early Warning (AEW)
- `C130`, `C17`, `C2`, `C5`, `E2`, `E7`, `KC135`, `P3`, `Tu160`, `Tu95`, `V22`, `Y20`

### Trainers & Light Attack
- `Alpha_Jet`, `Hawk`, `L15`, `L39`, `T45`, `T6`, `Yak130`

### Helicopters & Rotary Wing
- `AH64`, `CH47`, `Ka52`, `Mi24`, `Mi28`, `Mi8`, `UH60`, `Z10`, `Z19`

### Strategic Reconnaissance, UAVs & Support
- `MQ9`, `RQ4`, `SR71`, `U2`, `Wing_Loong`

---

## 3. Annotation Formats

### Original CSV Format
The source dataset provides bounding box annotations in tabular CSV format with the following schema:

```csv
filename,width,height,class,xmin,ymin,xmax,ymax
image_0001.jpg,1920,1080,F16,450,210,890,560
image_0002.jpg,1280,720,Su35,120,80,600,410
```

### YOLOv8 Label Format
During preprocessing via `preprocessing/dataset_split_preprocessing.py`, annotations are normalized and mapped into the standard YOLO format:

$$\text{box} = [ \text{class\_id}, x_{\text{center}}, y_{\text{center}}, \text{width}, \text{height} ]$$

Where:
- `class_id`: Integer index corresponding to the class name ($0 \dots 76$)
- $x_{\text{center}}, y_{\text{center}}$: Center coordinates normalized by image width and height $[0.0, 1.0]$
- $\text{width}, \text{height}$: Bounding box dimensions normalized by image width and height $[0.0, 1.0]$

Example label file (`image_0001.txt`):
```text
14 0.348958 0.356481 0.229167 0.324074
```

---

## 4. Dataset Partitioning

The preprocessor partitions labeled imagery using a stratified split:
- **Training Set (70%):** Utilized for deep feature representation and bounding box regression.
- **Validation Set (20%):** Monitored per epoch for early stopping and hyperparameter tuning.
- **Test Set (10%):** Held-out partition for unbiased final benchmark evaluation.

```
dataset/
├── images/
│   ├── train/
│   ├── val/
│   └── test/
└── labels/
    ├── train/
    ├── val/
    └── test/
```

---

## 5. Preprocessing Scripts

1. **`dataset_split_preprocessing.py`**:
   - Parses the primary annotation CSV.
   - Converts coordinates from absolute pixel offsets $(x_{\min}, y_{\min}, x_{\max}, y_{\max})$ to YOLO normalized format.
   - Generates directory hierarchies and outputs `data.yaml`.

2. **`SAM_Bounding_Box.py`**:
   - Leverages Meta's Segment Anything Model (SAM) for pseudo-labeling and boundary refinement on unannotated aerial frames.

3. **`crop_preprocessing.py`**:
   - Isolates bounding box crops per aircraft class to allow secondary fine-grained classifier training.
