# Default use the NVIDIA official image with PyTorch 2.3.0
# https://docs.nvidia.com/deeplearning/frameworks/pytorch-release-notes/index.html
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
COPY requirements.txt /app
RUN apt-get update -y && \
    DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y software-properties-common && \
    DEBIAN_FRONTEND=noninteractive add-apt-repository -y ppa:deadsnakes/ppa && \
    apt-get update -y && \
    DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y python3.12 python3.12-venv python3.12-dev python3-distutils python3-pip && \
    update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.12 1 && \
    update-alternatives --install /usr/bin/python python /usr/bin/python3.12 1 && \
    curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py && \
    # pip install --upgrade setuptools && \
    # pip install --upgrade pip wheel && \
    # pip uninstall -y setuptools && \
    # pip install setuptools && \
    python3.12 get-pip.py --break-system-packages && \
    # python3 get-pip.py --user
    echo "python upgrade finished"

RUN ls -l /usr/local && curl -LsSf https://astral.sh/uv/install.sh | sh &&  \
    # curl https://sh.rustup.rs -sSf | sh -s -- -y --no-modify-path --profile minimal && \
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y && \
    ln -s $HOME/.cargo/bin/cargo /usr/local/bin/cargo && export PATH=$HOME/.cargo/bin:$PATH && \
    ln -s $HOME/.cargo/bin/rustup /usr/local/bin/rustup && export PATH=$HOME/.cargo/bin:$PATH && \
    ln -s $HOME/.local/bin/uv /usr/local/bin/uv && \
    rustup install 1.79.0 && \
    export PATH=$HOME/.cargo/bin:$PATH && cargo -V && rustc -V && \
    # export PATH="$HOME/.local/bin:$PATH" && uv python install 3.11 && \
    # uv venv && uv python pin 3.11 && \
    pip config set global.index-url "$PIP_INDEX" && \
    pip config set global.extra-index-url "$PIP_INDEX" && \
    python -m pip install --upgrade pip && \
    if [ -n "$HTTP_PROXY" ]; then \
        python -m pip install --proxy=$HTTP_PROXY -r requirements.txt; \
    else \
        python -m pip install -r requirements.txt; \
    fi

# Copy the rest of the application into the image
COPY . /app

# Install the LLaMA Factory
RUN EXTRA_PACKAGES="metrics"; \
    if [ "$INSTALL_BNB" == "true" ]; then \
        EXTRA_PACKAGES="${EXTRA_PACKAGES},bitsandbytes"; \
    fi; \
    if [ "$INSTALL_VLLM" == "true" ]; then \
        EXTRA_PACKAGES="${EXTRA_PACKAGES},vllm"; \
    fi; \
    if [ "$INSTALL_DEEPSPEED" == "true" ]; then \
        EXTRA_PACKAGES="${EXTRA_PACKAGES},deepspeed"; \
    fi; \
    if [ "$INSTALL_LIGER_KERNEL" == "true" ]; then \
        EXTRA_PACKAGES="${EXTRA_PACKAGES},liger-kernel"; \
    fi; \
    if [ "$INSTALL_HQQ" == "true" ]; then \
        EXTRA_PACKAGES="${EXTRA_PACKAGES},hqq"; \
    fi; \
    if [ "$INSTALL_EETQ" == "true" ]; then \
        EXTRA_PACKAGES="${EXTRA_PACKAGES},eetq"; \
    fi; \
    if [ -n "$HTTP_PROXY" ]; then \
        pip install --proxy=$HTTP_PROXY -e ".[$EXTRA_PACKAGES]"; \
    else \
        pip install -e ".[$EXTRA_PACKAGES]"; \
    fi

# Rebuild flash attention
RUN pip uninstall -y transformer-engine flash-attn && \
    if [ "$INSTALL_FLASHATTN" == "true" ]; then \
        pip uninstall -y ninja && \
        if [ -n "$HTTP_PROXY" ]; then \
            pip install --proxy=$HTTP_PROXY ninja && \
            pip install --proxy=$HTTP_PROXY --no-cache-dir flash-attn --no-build-isolation; \
        else \
            pip install ninja && \
            pip install --no-cache-dir flash-attn --no-build-isolation; \
        fi; \
    fi


# Unset http proxy
RUN if [ -n "$HTTP_PROXY" ]; then \
        unset http_proxy; \
        unset https_proxy; \
    fi

# Set up volumes
VOLUME [ "/root/.cache/huggingface", "/root/.cache/modelscope", "/app/data", "/app/output" ]

# Expose port 7860 for the LLaMA Board
ENV GRADIO_SERVER_PORT 7860
EXPOSE 7860

# Expose port 8000 for the API service
ENV API_PORT 8000
EXPOSE 8000
