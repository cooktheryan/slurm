# Dockerfile.gpu-job
# RHEL 9 + CUDA 12.9 base image for GPU workloads on Slurm.
# Use as a base for job containers submitted via Slurm.

FROM registry.redhat.io/rhai/base-image-cuda-12.9-rhel9:3.3

LABEL org.opencontainers.image.title="gpu-job" \
      org.opencontainers.image.description="RHEL 9 + CUDA 12.9 base for GPU job containers"

# Install Python and numpy; PyTorch/vLLM can be layered on top as needed
RUN dnf install -y \
        python3 \
        python3-pip \
    && dnf clean all \
    && pip3 install --no-cache-dir numpy

# Create non-root user for OpenShift compatibility
RUN groupadd -g 1001 jobuser \
    && useradd -u 1001 -g jobuser -s /bin/bash -d /home/jobuser -m jobuser

ENV NVIDIA_VISIBLE_DEVICES="all" \
    NVIDIA_DRIVER_CAPABILITIES="compute,utility"

USER 1001
WORKDIR /home/jobuser
