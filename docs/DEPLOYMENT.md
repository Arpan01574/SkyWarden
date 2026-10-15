# SkyWarden Deployment & Production Guide

This guide covers deployment procedures for SkyWarden across local workstations, Docker containers, and cloud environments.

---

## 1. Local Deployment

### 1.1 Virtual Environment
```bash
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 1.2 Configuration
Create a `.env` file in the project root:
```env
YOLO_MODEL_PATH=./best40.pt
STREAMLIT_SERVER_PORT=8501
STREAMLIT_SERVER_ADDRESS=0.0.0.0
```

### 1.3 Running the Server
```bash
streamlit run app.py --server.port 8501 --server.address 0.0.0.0
```

---

## 2. Docker Containerization

### 2.1 Example `Dockerfile`
A reference production `Dockerfile` utilizing Ubuntu and Python 3.10 with GPU/CPU support:

```dockerfile
FROM python:3.10-slim

WORKDIR /app

# Install system dependencies for OpenCV and image processing
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libgl1 \
    libglib2.0-0 \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Copy dependency requirements
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application files
COPY . .

# Expose Streamlit default port
EXPOSE 8501

HEALTHCHECK CMD curl --fail http://localhost:8501/_stcore/health || exit 1

ENTRYPOINT ["streamlit", "run", "app.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

### 2.2 Building and Running the Image
```bash
# Build Docker image
docker build -t skywarden:latest .

# Run container (CPU)
docker run -p 8501:8501 -v $(pwd)/model:/app/model skywarden:latest

# Run container with NVIDIA GPU acceleration
docker run --gpus all -p 8501:8501 -v $(pwd)/model:/app/model skywarden:latest
```

---

## 3. Cloud Deployment Options

### 3.1 Streamlit Community Cloud
1. Push your repository to GitHub.
2. Sign in to [Streamlit Cloud](https://share.streamlit.io/).
3. Select repository `ashishsps20/SkyWarden`, branch `main`, and main file `app.py`.
4. Under Advanced Settings, configure the Secrets management tab for any required environment tokens.

### 3.2 Hugging Face Spaces (Streamlit SDK)
1. Create a new Space on [Hugging Face](https://huggingface.co/spaces).
2. Choose **Streamlit** as the Space SDK.
3. Link your GitHub repository or push directly via Git.
4. Ensure `best40.pt` is downloaded via `git-lfs` or fetched dynamically during space initialization.

### 3.3 AWS EC2 / GCP Compute Engine
1. Launch an `Ubuntu 22.04 LTS` instance with an attached GPU (e.g., `g4dn.xlarge` with NVIDIA T4).
2. Install NVIDIA drivers and Docker NVIDIA Container Toolkit.
3. Run SkyWarden via Docker with restart policies enabled:
   ```bash
   docker run -d --restart always --gpus all -p 80:8501 skywarden:latest
   ```

---

## 4. Performance Tuning

- **Model Caching:** Ensure the YOLO model is cached via `@st.cache_resource` in `app.py` to prevent redundant weights reload on every UI refresh.
- **Image Downsampling:** For ultra-high-resolution satellite frames (>4K), consider resizing inputs to standard YOLO dimensions ($640 \times 640$ or $1024 \times 1024$) prior to forward pass to cap inference latency under 50ms.
