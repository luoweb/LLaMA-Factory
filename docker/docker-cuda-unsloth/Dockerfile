# Default use the NVIDIA official image with PyTorch 2.3.0
# https://docs.nvidia.com/deeplearning/frameworks/pytorch-release-notes/index.html
# cuda12.3
# ARG BASE_IMAGE=nvcr.io/nvidia/pytorch:24.02-py3 
ARG BASE_IMAGE=nvcr.io/nvidia/pytorch:23.01-py3
FROM ${BASE_IMAGE}

# Define environments
ENV MAX_JOBS=4
ENV FLASH_ATTENTION_FORCE_BUILD=TRUE
ENV VLLM_WORKER_MULTIPROC_METHOD=spawn

# Define installation arguments
ARG INSTALL_BNB=false
ARG INSTALL_VLLM=false
ARG INSTALL_DEEPSPEED=false
ARG INSTALL_FLASHATTN=false
ARG INSTALL_LIGER_KERNEL=false
ARG INSTALL_HQQ=false
ARG INSTALL_EETQ=false
ARG PIP_INDEX=https://pypi.org/simple
ARG HTTP_PROXY=

# Set the working directory
WORKDIR /app

# Set http proxy
RUN if [ -n "$HTTP_PROXY" ]; then \
        echo "Configuring proxy..."; \
        export http_proxy=$HTTP_PROXY; \
        export https_proxy=$HTTP_PROXY; \
    fi

# Install the requirements
# COPY requirements.txt /app
RUN curl -LsSf https://astral.sh/uv/install.sh | sh &&  \
    curl https://sh.rustup.rs -sSf | sh -s -- -y --no-modify-path --profile minimal && \
    sudo ln -s $HOME/.cargo/bin/cargo /usr/local/bin && \
    sudo ln -s $HOME/.cargo/bin/rustup /usr/local/bin && \
    sudo ln -s $HOME/.local/bin/uv /usr/local/bin && \
    rustup install 1.79.0 && \
    export PATH="$HOME/.local/bin:$PATH" && uv python install 3.11 && \
    uv venv && uv python pin 3.11 && \
    pip config set global.index-url "$PIP_INDEX" && \
    pip config set global.extra-index-url "$PIP_INDEX" && \
    python -m pip install --upgrade pip && \
    if [ -n "$HTTP_PROXY" ]; then \
    python -m pip install --proxy=$HTTP_PROXY "unsloth[cu118-torch240] @ git+https://github.com/unslothai/unsloth.git"; \
    else \
    python -m pip install "unsloth[cu118-torch240] @ git+https://github.com/unslothai/unsloth.git"; \
    fi

# Expose port 7860 for the LLaMA Board
ENV GRADIO_SERVER_PORT 7860
EXPOSE 7860

# Expose port 8000 for the API service
ENV API_PORT 8000
EXPOSE 8000
