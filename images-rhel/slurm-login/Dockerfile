# Dockerfile.slurm-login
# Slurm login/submit node on UBI 9.
# Requires slurm-base image to be built first.

FROM slurm-base:latest

LABEL org.opencontainers.image.title="slurm-login" \
      org.opencontainers.image.description="Slurm login and job submission node on UBI 9"

USER root

RUN dnf install -y \
        openssh-server \
        openssh-clients \
    && dnf clean all \
    && ssh-keygen -A \
    && mkdir -p /run/sshd \
    && sed -i 's/^#PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config \
    && sed -i 's/^#PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config

# Allow non-root to run sshd on an unprivileged port
RUN sed -i 's/^#Port 22/Port 2222/' /etc/ssh/sshd_config \
    && chown -R 1001:1001 /etc/ssh/ssh_host_* /run/sshd

USER 1001

EXPOSE 2222

ENTRYPOINT ["/usr/sbin/sshd"]
CMD ["-D", "-e"]
