# Dockerfile.slurmctld
# Slurm controller daemon on UBI 9.
# Requires slurm-base image to be built first.

FROM slurm-base:latest

LABEL org.opencontainers.image.title="slurmctld" \
      org.opencontainers.image.description="Slurm controller daemon on UBI 9"

USER root

RUN mkdir -p /var/spool/slurm/ctld \
    && chown slurm:slurm /var/spool/slurm/ctld

USER 1001

EXPOSE 6817

ENTRYPOINT ["/usr/sbin/slurmctld"]
CMD ["-D", "-vv"]
