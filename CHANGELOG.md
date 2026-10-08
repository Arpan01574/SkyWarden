# Changelog

All notable changes to the SkyWarden project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

### Planned
- Dynamic configuration for friend/enemy coalitions via YAML or JSON.
- Video stream inference and multi-frame tracking support with ByteTrack.
- ONNX and TensorRT export pipelines for accelerated inference.
- Automated unit and integration test suite for preprocessing scripts.
- Dockerfile and Docker Compose orchestration manifests.

---

## [0.2.0] - 2026-10-01

### Added
- Complete system architecture documentation (`docs/ARCHITECTURE.md`).
- Dataset documentation with 77-class catalog and YOLO schema (`docs/DATASET.md`).
- Community contribution guide with commit rules (`CONTRIBUTING.md`).
- Troubleshooting and diagnostics manual (`docs/TROUBLESHOOTING.md`).
- Deployment guide covering local, Docker, and cloud infrastructure (`docs/DEPLOYMENT.md`).
- GitHub issue templates for bug reporting and feature requests.
- GitHub pull request review template.

### Changed
- Refactored `README.md` to include comprehensive sections for prerequisites, getting started, known limitations, and roadmap.
- Updated Google Drive link in `model` pointer file to current trained checkpoint weights.
- Consolidated dependencies into `requirements.txt`.

---

## [0.1.0] - 2026-08-07

### Added
- Initial release of SkyWarden.
- Core Streamlit application (`app.py`) with real-time friend/enemy threat detection.
- YOLOv8 model checkpoint trained for 20 epochs across 77 military aircraft categories.
- Dataset preparation pipeline (`dataset_split_preprocessing.py`) supporting CSV-to-YOLO conversion.
- Automated bounding-box generation using Meta's Segment Anything Model (`SAM_Bounding_Box.py`).
- Two-stage training scripts (`Full_Frame_Train.py` and `full_frame_freeze_train.py`).
- Evaluation results and training curves in `results/`.
