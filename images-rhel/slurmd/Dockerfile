# Dockerfile.slurmd
# Slurm compute daemon on UBI 9 with GPU support.
# Requires slurm-base image to be built first.
# Note: slurmd typically needs privileged or SYS_ADMIN capabilities for cgroup management.

FROM slurm-base:latest

LABEL org.opencontainers.image.title="slurmd" \
      org.opencontainers.image.description="Slurm compute daemon on UBI 9 with GPU support"

USER root

RUN mkdir -p /var/spool/slurm/d \
    && chown slurm:slurm /var/spool/slurm/d

# Install NVIDIA container toolkit repo and runtime for GPU auto-detection
RUN dnf install -y \
        https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm \
    && dnf config-manager --add-repo \
        https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo \
    && dnf install -y nvidia-container-toolkit \
    && dnf clean all

EXPOSE 6818

# slurmd needs root or elevated capabilities for cgroup management
ENTRYPOINT ["/usr/sbin/slurmd"]
CMD ["-D", "-vv"]
