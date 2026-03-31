# Dockerfile.slurm-base
# Multi-stage build: compile Slurm from source on UBI 9, produce a slim runtime image.
# This image serves as the base for slurmctld, slurmd, slurmdbd, and login containers.

ARG BASE_IMAGE=registry.access.redhat.com/ubi9:latest

# ---------------------------------------------------------------------------
# Stage 1 — build (CentOS Stream 9 for full -devel package availability)
# ---------------------------------------------------------------------------
FROM quay.io/centos/centos:stream9 AS build

ARG SLURM_VERSION=25.11.4

LABEL stage=build

# Enable CRB + EPEL for build dependencies
RUN dnf install -y epel-release \
    && /usr/bin/crb enable \
    && dnf install -y \
        gcc gcc-c++ make wget bzip2 \
        munge-devel \
        hwloc-devel \
        lua-devel \
        readline-devel \
        pam-devel \
        openssl-devel \
        numactl-devel \
        json-c-devel \
        dbus-devel \
        python3-devel \
        mariadb-connector-c-devel \
        rpm-build \
        perl perl-ExtUtils-MakeMaker \
        pkgconfig \
    && dnf clean all

WORKDIR /tmp

RUN wget -q "https://download.schedmd.com/slurm/slurm-${SLURM_VERSION}.tar.bz2" \
    && tar xjf "slurm-${SLURM_VERSION}.tar.bz2"

WORKDIR /tmp/slurm-${SLURM_VERSION}

RUN ./configure \
        --prefix=/usr \
        --sysconfdir=/etc/slurm \
        --with-hwloc=/usr \
        --enable-pam \
    && make -j"$(nproc)" \
    && make install DESTDIR=/tmp/slurm-install

# Install runtime packages that are not available in UBI repos
RUN dnf install -y munge hwloc numactl lua mariadb-connector-c \
    && dnf clean all

# ---------------------------------------------------------------------------
# Stage 2 — runtime
# ---------------------------------------------------------------------------
FROM ${BASE_IMAGE}

ARG SLURM_VERSION=25.11.4

LABEL org.opencontainers.image.title="slurm-base" \
      org.opencontainers.image.description="UBI 9 base image with Slurm ${SLURM_VERSION}" \
      org.opencontainers.image.version="${SLURM_VERSION}" \
      org.opencontainers.image.source="https://github.com/rcook/slurm"

# Install packages available in UBI repos
RUN dnf install -y \
        openssl \
        json-c \
        dbus-libs \
        python3 \
        shadow-utils \
        procps-ng \
        hostname \
    && dnf clean all

# Copy munge binaries and libs from build stage (not available in UBI repos)
COPY --from=build /usr/bin/munge /usr/bin/remunge /usr/bin/unmunge /usr/bin/
COPY --from=build /usr/sbin/munged /usr/sbin/create-munge-key /usr/sbin/
COPY --from=build /usr/lib64/libmunge.so* /usr/lib64/
COPY --from=build /usr/lib64/libhwloc.so* /usr/lib64/
COPY --from=build /usr/lib64/libnuma.so* /usr/lib64/
COPY --from=build /usr/lib64/liblua-5*.so /usr/lib64/
COPY --from=build /usr/lib64/libmariadb.so* /usr/lib64/

# Copy compiled Slurm binaries and libraries from build stage
COPY --from=build /tmp/slurm-install/ /

# Refresh shared library cache so all libs are found
RUN ldconfig

# Create munge user/group (needed for munge authentication)
RUN groupadd -g 981 munge \
    && useradd -u 981 -g munge -s /sbin/nologin -d /var/lib/munge -r munge

# Create slurm user/group (UID/GID 1001 for OpenShift restricted-v2 SCC)
RUN groupadd -g 1001 slurm \
    && useradd -u 1001 -g slurm -s /bin/bash -d /var/lib/slurm -m slurm

# Create required directories with correct ownership
RUN mkdir -p \
        /etc/slurm \
        /var/spool/slurm/ctld \
        /var/spool/slurm/d \
        /var/log/slurm \
        /var/run/munge \
        /var/log/munge \
        /var/lib/munge \
        /etc/munge \
    && chown -R slurm:slurm \
        /etc/slurm \
        /var/spool/slurm \
        /var/log/slurm \
    && chown -R munge:munge \
        /var/run/munge \
        /var/log/munge \
        /var/lib/munge \
        /etc/munge \
    && chmod 0700 /etc/munge \
    && chmod 0711 /var/lib/munge \
    && chmod 0700 /var/log/munge

# Slurm configs are mounted at runtime via ConfigMap / Secret
VOLUME ["/etc/slurm", "/etc/munge"]

USER 1001
