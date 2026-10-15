# SkyWarden Troubleshooting Guide

This guide helps diagnose and resolve common errors encountered when configuring, running, or training SkyWarden.

---

## 1. Quick Diagnostics

Run this quick one-liner to verify your Python environment and hardware accelerator status:

```bash
python -c "import torch, ultralytics, cv2; print('PyTorch:', torch.__version__); print('CUDA Available:', torch.cuda.is_available()); print('Ultralytics:', ultralytics.__version__); print('OpenCV:', cv2.__version__)"
```

---

## 2. Common Errors and Resolutions

### 2.1 Model Weights File Not Found (`FileNotFoundError`)

**Symptom:**
```
FileNotFoundError: [Errno 2] No such file or directory: 'best40.pt'
```

**Cause:**
Trained weights (`best40.pt` or `best.pt`) are omitted from Git due to file size constraints.

**Resolution:**
1. Check the Google Drive link located in the `model` pointer file in the project root.
2. Download `best40.pt` and place it in the project root or specified models directory.
3. Verify your `.env` points to the correct location:
   ```env
   YOLO_MODEL_PATH=./best40.pt
   ```

---

### 2.2 OpenCV Headless Server Error (`libGL.so.1`)

**Symptom:**
```
ImportError: libGL.so.1: cannot open shared object file: No such file or directory
```

**Cause:**
Standard `opencv-python` looks for X11 / OpenGL shared libraries which are missing in headless Docker containers or Linux cloud instances (e.g., Ubuntu server).

**Resolution:**
Install the headless version of OpenCV or install the missing system library:
```bash
# Option A: Install system graphics library
sudo apt-get update && sudo apt-get install -y libgl1-mesa-glx libglib2.0-0

# Option B: Swap to headless OpenCV in requirements
pip uninstall -y opencv-python opencv-python-headless
pip install opencv-python-headless
```

---

### 2.3 CUDA Out of Memory (OOM) During Training

**Symptom:**
```
torch.cuda.OutOfMemoryError: CUDA out of memory. Tried to allocate 512.00 MiB
```

**Cause:**
Batch size or input resolution exceeds GPU VRAM capacity.

**Resolution:**
1. Decrease batch size in training scripts (`training/Full_Frame_Train.py`):
   ```python
   # Modify batch argument from 16 to 8 or 4
   model.train(data="data.yaml", epochs=20, imgsz=640, batch=4)
   ```
2. Enable mixed-precision (FP16) training: `half=True`.
3. If running on Apple Silicon, use MPS:
   ```python
   device = "mps" if torch.backends.mps.is_available() else "cpu"
   ```

---

### 2.4 Streamlit Port Conflict (`Port 8501 is already in use`)

**Symptom:**
```
Network error: Port 8501 is already in use.
```

**Cause:**
Another background instance of Streamlit is active.

**Resolution:**
Specify an alternative port via the CLI:
```bash
streamlit run app.py --server.port 8502
```
Or terminate existing processes on port 8501:
```bash
# macOS / Linux
lsof -ti:8501 | xargs kill -9
```

---

### 2.5 Segment Anything (SAM) Checkpoint Missing

**Symptom:**
```
FileNotFoundError: sam_vit_h_4b8939.pth not found
```

**Cause:**
`preprocessing/SAM_Bounding_Box.py` requires Meta's Vision Transformer checkpoint.

**Resolution:**
Download the ViT-H checkpoint directly from Meta's official repository:
```bash
wget https://dl.fbaipublicfiles.com/segment_anything/sam_vit_h_4b8939.pth
```
Update the path in `preprocessing/SAM_Bounding_Box.py` to match the downloaded checkpoint location.
