# ref: https://github.com/kubeflow/mpi-operator/blob/release-0.7/examples/v2beta1/pi/openmpi.Dockerfile
# Use a multi-stage build to get Python 3.11 (tensorflow_io doesn't support Python 3.13)
FROM python:3.11-slim-bookworm AS python-source

FROM mpioperator/openmpi:v0.7.0

ENV DEBIAN_FRONTEND=noninteractive \
    VENV_PATH=/workspace/venv \
    SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt

# Copy Python 3.11 from the python image
COPY --from=python-source /usr/local/bin/python3.11 /usr/local/bin/python3.11
COPY --from=python-source /usr/local/lib/python3.11 /usr/local/lib/python3.11
COPY --from=python-source /usr/local/lib/libpython3.11.* /usr/local/lib/
COPY --from=python-source /usr/local/include/python3.11 /usr/local/include/python3.11

# Install only what we really need, with no recommends,
# and clean up apt cache in the same layer.
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        git \
        sysstat \
        cmake \
        libhwloc-dev \
        ca-certificates \
    && rm -rf /var/lib/apt/lists/* && \
    update-ca-certificates

# Update ldconfig and create symlinks
RUN ldconfig && \
    ln -sf /usr/local/bin/python3.11 /usr/local/bin/python3

# Set environment variables for pip trusted hosts
ENV PIP_TRUSTED_HOST="pypi.org files.pythonhosted.org pypi.nvidia.com"

# Create venv with Python 3.11, upgrade pip, install Python deps
# (no-cache-dir to avoid pip cache bloat)
RUN python3.11 -m venv "${VENV_PATH}" && \
    "${VENV_PATH}/bin/pip" install --upgrade pip && \
    "${VENV_PATH}/bin/pip" install --no-cache-dir certifi pybind11

ENV PATH="${VENV_PATH}/bin:${PATH}"
WORKDIR /workspace

# Shallow clone the storage repository at v2.0 tag, install, and clean up.
# Note: Global git http.sslVerify=false is temporarily set for this RUN command only
# due to SSL certificate issues when using copied Python. Pip will use git to clone
# dependencies, so we need the global config.
RUN git config --global http.sslVerify false && \
    git clone --depth 1 --branch v2.0 https://github.com/mlcommons/storage.git && \
    cd storage && \
    pip install --extra-index-url https://pypi.nvidia.com --no-cache-dir . && \
    cd .. && \
    rm -rf storage && \
    git config --global --unset http.sslVerify && \
    # optional: remove Python bytecode caches to trim a bit more
    find "${VENV_PATH}" -type d -name '__pycache__' -prune -exec rm -rf {} +
