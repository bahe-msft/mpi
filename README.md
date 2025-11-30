# MPI

This repository hosts MPI-related Dockerfiles and Docker images.

## Available Images

### dlio_benchmark

A Docker image for running DLIO benchmarks with MPICH.

**Image:** `ghcr.io/bahe-msft/mpi/dlio_benchmark:v2.0.1-mpich`

Based on `mpioperator/mpich:v0.7.0` with DLIO benchmark v2.0.1 installed.

#### Pull the image

```bash
docker pull ghcr.io/bahe-msft/mpi/dlio_benchmark:v2.0.1-mpich
```

## Building Images

Images are automatically built and pushed to GitHub Container Registry (ghcr.io) when changes are pushed to the `main` branch.

To build locally:

```bash
cd dlio_benchmark
docker build -t dlio_benchmark:v2.0.1-mpich .
```

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.