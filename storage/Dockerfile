# ref: https://github.com/kubeflow/mpi-operator/blob/release-0.7/examples/v2beta1/pi/openmpi.Dockerfile
FROM mpioperator/openmpi:v0.7.0

ENV DEBIAN_FRONTEND=noninteractive \
    VENV_PATH=/workspace/venv

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

# Create venv, upgrade pip, install Python deps
# (no-cache-dir to avoid pip cache bloat)
RUN python3 -m venv "${VENV_PATH}" && \
    "${VENV_PATH}/bin/pip" install --upgrade pip && \
    "${VENV_PATH}/bin/pip" install --no-cache-dir pybind11

ENV PATH="${VENV_PATH}/bin:${PATH}"
WORKDIR /workspace

# Shallow clone the storage repository, install, and clean up.
RUN git clone --depth 1 https://github.com/mlcommons/storage.git && \
    cd storage && \
    pip install --no-cache-dir . && \
    cd .. && \
    rm -rf storage && \
    # optional: remove Python bytecode caches to trim a bit more
    find "${VENV_PATH}" -type d -name '__pycache__' -prune -exec rm -rf {} +
