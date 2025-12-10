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

# Shallow clone the storage repository at v2.0 tag, install, and clean up.
# NOTE: We patch the pyproject.toml to use a specific commit hash so we can remove the outdated tensorflow_io dependency
# ref: https://github.com/argonne-lcf/dlio_benchmark/pull/317
RUN git clone --depth 1 --branch v2.0 https://github.com/mlcommons/storage.git && \
    cd storage && \
    sed -i 's/@mlperf_storage_v2.0/@695f3a3ca995e8f5d0c9f4b3a7b7400068a28afb/g' pyproject.toml && \
    pip install --no-cache-dir . && \
    cd .. && \
    rm -rf storage && \
    # optional: remove Python bytecode caches to trim a bit more
    find "${VENV_PATH}" -type d -name '__pycache__' -prune -exec rm -rf {} +
