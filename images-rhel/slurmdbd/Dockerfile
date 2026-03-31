# Dockerfile.slurmdbd
# Slurm accounting daemon on UBI 9.
# Requires slurm-base image to be built first.

FROM slurm-base:latest

LABEL org.opencontainers.image.title="slurmdbd" \
      org.opencontainers.image.description="Slurm accounting daemon on UBI 9"

USER 1001

EXPOSE 6819

ENTRYPOINT ["/usr/sbin/slurmdbd"]
CMD ["-D", "-vv"]
