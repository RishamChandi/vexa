FROM nvidia/cuda:12.3.2-cudnn9-runtime-ubuntu22.04

# uv bootstraps itself (no OS python needed) and provisions the managed CPython 3.12 venv the
# service runs on; the CUDA base provides the CUDA/cuDNN libraries CTranslate2 uses directly.
COPY --from=ghcr.io/astral-sh/uv:0.9.22 /uv /uvx /usr/local/bin/

ENV UV_LINK_MODE=copy \
    UV_PROJECT_ENVIRONMENT=/opt/venv \
    UV_PYTHON=3.12 \
    PYTHONUNBUFFERED=1 \
    PATH=/opt/venv/bin:$PATH

RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    libsndfile1 \
    ffmpeg \
    && rm -rf /var/lib/apt/lists/* \
 && uv python install 3.12 \
 && uv venv /opt/venv

WORKDIR /app

# Copy requirements and install
# faster-whisper uses CTranslate2 which has built-in CUDA support (no PyTorch needed)
COPY requirements.txt ./requirements.txt
RUN uv pip install --no-cache -r requirements.txt

# Copy application
COPY main.py .

# Create models directory
RUN mkdir -p /app/models

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

EXPOSE 8000

CMD ["python", "main.py"]
