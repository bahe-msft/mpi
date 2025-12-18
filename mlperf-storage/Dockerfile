# ref: https://github.com/kubeflow/mpi-operator/blob/release-0.7/examples/v2beta1/pi/openmpi.Dockerfile
FROM mpioperator/openmpi:v0.7.0

ENV DEBIAN_FRONTEND=noninteractive \
    VENV_PATH=/workspace/venv \
    PYTHON_VERSION=3.12.8 \
    PYTHON_INSTALL_PREFIX=/usr/local

# Install only what we really need, with no recommends,
# and clean up apt cache in the same layer.
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        git \
        sysstat \
        python3 \
        python3-pip \
        python3-venv \
        cmake \
        libhwloc-dev \
    && rm -rf /var/lib/apt/lists/*

# Check if python3.12 is available, if not download and compile it
RUN if ! command -v python3.12 > /dev/null 2>&1; then \
        echo "python3.12 not found, downloading and compiling Python ${PYTHON_VERSION}..." && \
        apt-get update && \
        apt-get install -y --no-install-recommends \
            build-essential \
            zlib1g-dev \
            libncurses5-dev \
            libgdbm-dev \
            libnss3-dev \
            libssl-dev \
            libreadline-dev \
            libffi-dev \
            libsqlite3-dev \
            libbz2-dev \
            liblzma-dev \
            wget && \
        cd /tmp && \
        wget -q "https://www.python.org/ftp/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tgz" && \
        tar -xzf "Python-${PYTHON_VERSION}.tgz" && \
        cd "Python-${PYTHON_VERSION}" && \
        ./configure --prefix="${PYTHON_INSTALL_PREFIX}" --enable-optimizations --with-ensurepip=install && \
        make -j"$(nproc)" && \
        make install && \
        cd / && \
        rm -rf /tmp/Python-${PYTHON_VERSION} /tmp/Python-${PYTHON_VERSION}.tgz && \
        apt-get clean && \
        rm -rf /var/lib/apt/lists/* && \
        echo "Python ${PYTHON_VERSION} installed successfully at ${PYTHON_INSTALL_PREFIX}/bin/python3.12"; \
    else \
        echo "python3.12 is already available: $(command -v python3.12)"; \
    fi

# Create venv using python3.12, upgrade pip, install Python deps
# (no-cache-dir to avoid pip cache bloat)
RUN python3.12 -m venv "${VENV_PATH}" && \
    "${VENV_PATH}/bin/pip" install --upgrade pip && \
    "${VENV_PATH}/bin/pip" install --no-cache-dir pybind11

ENV PATH="${VENV_PATH}/bin:${PATH}"

# Create /workspace directory and set ownership to mpiuser
RUN mkdir -p /workspace && chown mpiuser:mpiuser /workspace
WORKDIR /workspace

# Clone the storage repository at specific commit, install, and clean up.
RUN git clone https://github.com/mlcommons/storage.git && \
    cd storage && \
    git checkout 5ded0d8f663e59450cb276f0ddfe91bd4bf3f595 && \
    pip install --no-cache-dir . && \
    cd .. && \
    rm -rf storage && \
    # optional: remove Python bytecode caches to trim a bit more
    find "${VENV_PATH}" -type d -name '__pycache__' -prune -exec rm -rf {} +
