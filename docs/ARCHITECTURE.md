# SkyWarden System Architecture & Pipeline Specification

This document details the architectural design, algorithmic workflow, and system modules powering SkyWarden.

---

## 1. High-Level System Architecture

SkyWarden is designed around a multi-tier modular pipeline integrating computer vision models, rule-based classification heuristics, and a reactive user interface:

```
┌────────────────────────────────────────────────────────┐
│                   User Upload Layer                    │
│   (Streamlit Interface, Image Validation & Buffer)     │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│               Inference & Detection Core               │
│        Ultralytics YOLOv8 Architecture (PyTorch)       │
│  - Spatial feature extraction (CSPDarknet53 backbone)  │
│  - Multi-scale feature fusion (PANet neck)             │
│  - Anchor-free bounding box regression & class heads   │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│             Allegiance Evaluation Engine               │
│  - Heuristic mapping against designated coalitions     │
│  - Friend / Enemy / Neutral / Unknown status labeling  │
│  - Threat aggregation & highest alert level selection  │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│            Rendering & Notification Engine             │
│  - Bounding box rendering with OpenCV                  │
│  - Green (Friendly) / Red (Hostile) / White (Unknown)  │
│  - Visual alert banners, audio cues, and summary stats │
└────────────────────────────────────────────────────────┘
```

---

## 2. Core Modules

### 2.1 Web Application Layer (`app.py`)
- **Technology:** Streamlit
- **Responsibilities:**
  - Manages browser sessions and drag-and-drop file inputs (JPEG, PNG, WebP).
  - Handles image memory buffers with Python PIL and NumPy.
  - Dynamically renders annotated output arrays and real-time alert widgets.

### 2.2 Deep Learning Detector (`model/best40.pt`)
- **Framework:** PyTorch via Ultralytics YOLOv8.
- **Model Topology:**
  - Backbone: Modified CSPDarknet with Cross-Stage Partial connections for robust feature extraction at varied scales.
  - Neck: Path Aggregation Network (PANet) ensuring spatial details from lower layers reach classification heads.
  - Head: Decoupled anchor-free detection head predicting bounding box coordinates $(x, y, w, h)$, objectness confidence, and class distributions across 77 categories simultaneously.

### 2.3 Threat Evaluation Logic
- Incoming detections above confidence threshold (default $\tau = 0.40$) are compared against:
  - **`friend_classes`:** Friendly military aircraft list.
  - **`enemy_classes`:** Opposing military aircraft inventory.
- When any detected aircraft matches an entry in `enemy_classes`:
  - An emergency warning banner (`⚠️ HOSTILE AIRCRAFT DETECTED`) is raised immediately.
  - Bounding box coordinates are drawn in bright red (`(0, 0, 255)` in BGR).
- Friendly matches are drawn in green (`(0, 255, 0)` in BGR).
- Unclassified or unrecognized classes are outlined in neutral white (`(255, 255, 255)`).

---

## 3. Training & Labeling Pipeline

```
[Raw Aerial Images]
        │
        ├─────────────────────────────┐
        ▼                             ▼
[CSV Ground Truth]           [Unlabeled Images]
        │                             │
        ▼                             ▼
[dataset_split_preprocessing] [SAM_Bounding_Box] (Meta SAM)
        │                             │
        └──────────────┬──────────────┘
                       ▼
             [YOLO Formatted Data]
                       │
       ┌───────────────┴───────────────┐
       ▼                               ▼
[Full_Frame_Train]             [crop_preprocessing]
  (20 Epochs Full Train)               │
       │                               ▼
       ▼                    [crop_classification_train]
[full_frame_freeze_train]    (Fine-grained crop training)
  (10 Epochs Backbone Freeze)
       │
       ▼
[Final Weights: best40.pt]
```

### 3.1 Semi-Automated Annotation (SAM)
`preprocessing/SAM_Bounding_Box.py` uses Meta's Segment Anything Model (`sam_vit_h_4b8939.pth`) to evaluate low-contrast or partially occluded aircraft silhouettes, translating segmentation masks into tight bounding boxes to augment sparse ground-truth labels.

### 3.2 Two-Stage Detection Training
1. **Full-Frame Training (`training/Full_Frame_Train.py`):**
   - Base weights initialized from pre-trained YOLOv8 weights.
   - Standard augmentation: Mosaic, MixUp, HSV jitter, random affine transformations.
   - Optimization: SGD with Cosine Annealing learning rate schedule.
2. **Backbone Freeze Refinement (`training/full_frame_freeze_train.py`):**
   - Freezes initial 20 layers of the feature extractor.
   - Fine-tunes neck and detection heads with lower learning rate to stabilize gradients.

---

## 4. Latency & Resource Benchmarks

| Hardware Profile | Image Resolution | Inference Latency | FPS |
|---|---|---|---|
| NVIDIA RTX 3080 / 4080 | 1024 × 1024 | 18 – 24 ms | ~45 FPS |
| NVIDIA T4 (Cloud GPU) | 1024 × 1024 | 35 – 48 ms | ~22 FPS |
| Apple Silicon M-series (MPS) | 1024 × 1024 | 45 – 65 ms | ~18 FPS |
| Intel/AMD x86 CPU (AVX2) | 1024 × 1024 | 180 – 260 ms | ~4 FPS |
