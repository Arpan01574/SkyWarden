# SkyWarden ✈️

**AI-powered aerial image analysis that detects, identifies, and flags military aircraft in real time and process it.**

[![Streamlit App](https://static.streamlit.io/badges/streamlit_badge_black_white.svg)](https://skywarden.streamlit.app/)
![Python](https://img.shields.io/badge/Python-3.9%2B-blue)
![YOLOv8](https://img.shields.io/badge/Model-YOLOv8-8A2BE2)
![Streamlit](https://img.shields.io/badge/App-Streamlit-FF4B4B)
![PyTorch](https://img.shields.io/badge/Backend-PyTorch-EE4C2C)
![Documentation](https://img.shields.io/badge/docs-available-informational)

> 🚀 **Live Interactive Demo:** Experience SkyWarden in real-time in your browser: **[skywarden.streamlit.app](https://skywarden.streamlit.app/)**

SkyWarden is a computer-vision system that detects military aircraft in aerial images, classifies each as friendly or enemy, and raises an alert the moment a hostile aircraft is spotted.

## Table of Contents

- [Documentation Index](#documentation-index)
- [Objective](#objective)
- [Architecture](#architecture)
- [Features](#features)
- [Demo](#demo)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [How It Works](#how-it-works)
- [Dataset](#dataset)
- [Results](#results)
- [Known Limitations](#known-limitations)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [Security](#security)
- [Changelog](#changelog)
- [License](#license)
- [Acknowledgments](#acknowledgments)
- [Citation](#citation)
- [Support](#support)
- [Contributors](#contributors)

## Documentation Index

Explore in-depth documentation guides for developers, operators, and researchers:

- 🏗️ **[System Architecture & Pipeline Specification](docs/ARCHITECTURE.md)** — Architectural components, YOLOv8 topology, and SAM auto-annotation.
- 📊 **[Dataset & Taxonomy Guide](docs/DATASET.md)** — Comprehensive breakdown of 77 military aircraft classes and annotation format conversions.
- 🚀 **[Deployment & Production Guide](docs/DEPLOYMENT.md)** — Local setups, production Docker containers, and cloud deployments.
- 🛠️ **[Troubleshooting & Diagnostics](docs/TROUBLESHOOTING.md)** — Fixes for PyTorch CUDA/MPS, OpenCV headless, and Streamlit issues.
- 🤝 **[Contributing Guidelines](CONTRIBUTING.md)** — Development workflow, branching conventions, and commit formatting.
- 🔒 **[Security Policy](SECURITY.md)** — Supported versions, vulnerability disclosures, and machine learning security practices.
- 📜 **[Project Changelog](CHANGELOG.md)** — Detailed history of releases and milestone tracking.

## Objective

SkyWarden demonstrates how modern computer vision can automatically detect and classify military aircraft from aerial imagery while providing an immediate friend-or-enemy assessment for situational awareness. It combines a YOLOv8 detector, a Segment-Anything-assisted labeling pipeline, and a Streamlit front end into a single end-to-end demo — from raw aerial imagery to an annotated, alert-triggering result.

## Architecture

```
Input Image
     │
     ▼
YOLOv8 Detection + Classification
     │
     ▼
Friend / Enemy Mapping
     │
     ▼
Annotated Output + Alert
```

Detection and classification happen in a single YOLOv8 pass (one model predicts both the box and the aircraft type); the friend/enemy step is a separate rule-based lookup against the static allegiance lists in `app.py`. See [How It Works](#how-it-works) for the full pipeline, including preprocessing and training.

## Features

- **Aircraft detection** — locates aircraft in an aerial image and draws bounding boxes with class name and confidence score.
- **77-class recognition** — recognizes 77 distinct aircraft types, from fighters and bombers to transports and UAVs.
- **Friend/enemy classification** — cross-references each detected type against a curated allegiance list and colors boxes accordingly (green = friend, red = enemy, white = unrecognized).
- **Real-time alerts** — surfaces a clear on-screen warning the moment any enemy aircraft is detected.
- **Web interface** — drag-and-drop image upload via Streamlit, no setup beyond installing dependencies.
- **Reproducible training pipeline** — scripts to go from raw labeled data to a trained YOLOv8 checkpoint, including SAM-assisted auto-labeling.

## Demo

| Sample input | Model validation predictions |
|---|---|
| ![Sample aircraft image](images/sample_f16.jpg) | ![Validation batch predictions](results/val_batch0_pred.jpg) |

More raw sample inputs are in `images/`. Full evaluation plots and validation prediction grids from training are in `results/`. Note the grid above is YOLO's standard per-class training visualization — the live app's actual output uses green/red boxes for the friend/enemy call instead.

> If these don't render wherever you're previewing this file, it's because `images/` and `results/` need to sit alongside the README — the paths are relative to the repo, not broken links.

## Tech Stack

**Deep Learning & Computer Vision**
- [Ultralytics YOLOv8](https://github.com/ultralytics/ultralytics) — object detection
- [PyTorch](https://pytorch.org/) — model backend
- [OpenCV](https://opencv.org/) — image processing & annotation
- [Segment Anything (SAM)](https://github.com/facebookresearch/segment-anything) — automated bounding-box labeling

**Data & Evaluation**
- [NumPy](https://numpy.org/) · [Pandas](https://pandas.pydata.org/) · [Matplotlib](https://matplotlib.org/) · [scikit-learn](https://scikit-learn.org/)

**Application**
- [Streamlit](https://streamlit.io/) — web interface
- [python-dotenv](https://pypi.org/project/python-dotenv/) — environment configuration

## Project Structure

```
SkyWarden/
├── app.py                              # Streamlit app: detection + friend/enemy alerting
├── requirements.txt                    # Python dependencies
├── model                               # Pointer file — link to trained weights (see Getting Started)
├── images/                             # Sample aircraft images used for docs/demo
├── preprocessing/
│   ├── dataset_split_preprocessing.py  # CSV → YOLO labels, train/val/test split
│   ├── crop_preprocessing.py           # Restructures cropped, per-class images for classification training
│   └── SAM_Bounding_Box.py             # Auto-generates bounding boxes with Segment Anything
├── training/
│   ├── Full_Frame_Train.py             # Trains YOLOv8 on full aerial frames
│   ├── full_frame_freeze_train.py      # Resumes training with backbone frozen
│   └── crop_classification_train.py    # Fine-tunes on cropped, single-aircraft images
├── results/                            # Training curves & evaluation plots
├── .gitignore
└── README.md
```
## Getting Started

### Prerequisites

- Python 3.9+
- pip (or conda)
- A CUDA-capable GPU is recommended for training, but not required for running the app with a pre-trained checkpoint

### Installation

```bash
git clone https://github.com/ashishsps20/SkyWarden.git
cd SkyWarden
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

`requirements.txt` covers both the app and the preprocessing/training scripts.

### Model Weights

Trained weights aren't checked into the repo (see `.gitignore`). Download them from the link in the `model` file and note the local path.

### Environment Variables

Create a `.env` file in the project root:

```env
YOLO_MODEL_PATH=/path/to/your/trained/model.pt
```

### Run the App

Click the ▶️ **Play** button on `app.py` in VS Code, or run via terminal:
```bash
streamlit run app.py
```

Open the local URL Streamlit prints (typically `http://localhost:8501`), upload an aerial image, and view the annotated result.

## How It Works

1. **Label preparation** — `dataset_split_preprocessing.py` converts the dataset's CSV bounding-box annotations into YOLO-format `.txt` labels and splits the data 70/20/10 into train/val/test.
2. **Auto-labeling (optional)** — `SAM_Bounding_Box.py` runs Meta's Segment Anything Model over unlabeled images and converts the resulting masks into YOLO-format boxes.
3. **Classification data prep** — `crop_preprocessing.py` restructures a folder of cropped, per-class aircraft images into the same YOLO layout (one full-image box per crop) for a classification-style fine-tuning pass.
4. **Training** — `Full_Frame_Train.py` trains YOLOv8 on full aerial frames for 20 epochs; `full_frame_freeze_train.py` resumes from a checkpoint with the first 20 layers frozen for 10 more epochs; `crop_classification_train.py` fine-tunes the same way on cropped images.
5. **Inference** — `app.py` loads the trained checkpoint, runs detection on the uploaded image, and checks each predicted class name against two static allegiance lists (`friend_classes`, `enemy_classes`) to color-code boxes and trigger the enemy-detected alert.

## Dataset

Trained on the [Military Aircraft Detection Dataset](https://www.kaggle.com/datasets/a2015003713/militaryaircraftdetectiondataset/data) from Kaggle — 77 labeled aircraft classes spanning fighters, bombers, transports, helicopters, and UAVs, used for both detection (full frame) and classification (cropped) training.

For complete class taxonomy, CSV and YOLO annotation schemas, and data partitioning details, refer to the [Dataset Documentation](docs/DATASET.md).

## Results

Training ran for 20 epochs on the full-frame detector. These are approximate values read from the training curves in `results/results.png`, not exact logged metrics — see `results/` for the full confusion matrix and PR/F1 curves.

| Metric | Value |
|---|---:|
| Precision | ~0.78 |
| Recall | ~0.69 |
| mAP@50 | ~0.76 |
| mAP@50–95 | ~0.70 |
| Classes | 77 |

## Known Limitations

- **Dependencies aren't version-pinned** — `requirements.txt` lists packages without specific versions; pin them once you've confirmed a working set.
- **No license file** — see [License](#license).
- **Hardcoded paths** — the preprocessing and training scripts use local Windows paths (`C:\arpan\...`) and Kaggle paths (`/kaggle/working/...`) that need updating before running elsewhere.
- **Class-name mismatch** — `app.py`'s `friend_classes` list includes `"FA18"`, while `dataset_split_preprocessing.py`'s `CLASS_NAMES` uses `"F18"` for the same aircraft. Worth confirming against your trained model's actual `model.names` so F/A-18 detections aren't miscategorized as "Unknown."
- **Static allegiance list** — friend/enemy status is a hardcoded lookup in `app.py`, not a learned or configurable property; relabeling an aircraft means editing source code.

## Roadmap

- [ ] Pin versions in `requirements.txt`
- [ ] Move `friend_classes` / `enemy_classes` into an external config (JSON/YAML)
- [ ] Add batch and video/stream inference
- [ ] Add automated tests for the preprocessing pipeline
- [ ] Containerize with Docker for easier setup
- [ ] Publish a versioned model release instead of a Drive link
- [ ] Expose detection through a REST API
- [ ] Export the model to ONNX for broader deployment options

## Contributing

Contributions are warmly welcomed! Please check out the full [Contributing Guide](CONTRIBUTING.md) for branch naming conventions, PR guidelines, and coding standards.

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m "feat: add your feature"`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a pull request

## Security

For vulnerability disclosures and reporting procedures, review our [Security Policy](SECURITY.md).

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for a complete history of changes, version milestones, and roadmap items.

## License

No license has been specified for this project yet, so all rights are reserved by the authors by default. Consider adding an [MIT](https://choosealicense.com/licenses/mit/), [Apache 2.0](https://choosealicense.com/licenses/apache-2.0/), or similar open-source license if you want others to use or build on this code.

## Acknowledgments

- [Ultralytics](https://github.com/ultralytics/ultralytics) for YOLOv8
- [Meta AI](https://github.com/facebookresearch/segment-anything) for Segment Anything (SAM)
- [a2015003713](https://www.kaggle.com/a2015003713) for the Military Aircraft Detection Dataset on Kaggle

## Citation

If you use SkyWarden in research or a derivative project, please cite this repository:

```
SkyWarden — Military Aircraft Detection & Friend/Enemy Classification
https://github.com/ashishsps20/SkyWarden
```

## Support

For questions, bug reports, or feature requests, please open an issue in this repository.

## Contributors

| Name | Role | GitHub |
|---|---|---|
| Ashish Gautam | Project Maintainer | [@ashishsps20](https://github.com/ashishsps20) |
| Arpan Pethkar | Core Contributor | [@Arpan01574](https://github.com/Arpan01574) |
| Alok Shukla | Contributor | [@alok027-glitch](https://github.com/alok027-glitch) |

