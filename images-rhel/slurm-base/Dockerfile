# Dockerfile.slurm-base
# Multi-stage build: compile Slurm from source on UBI 9, produce a slim runtime image.
# This image serves as the base for slurmctld, slurmd, slurmdbd, and login containers.

ARG BASE_IMAGE=registry.access.redhat.com/ubi9:latest

# ---------------------------------------------------------------------------
# Stage 1 — build (UBI 9 + activation key to unlock full RHEL repos)
# Pass --build-arg RHEL_ORG=<org> --build-arg RHEL_ACTIVATION_KEY=<key>
# ---------------------------------------------------------------------------
FROM ${BASE_IMAGE} AS build

ARG SLURM_VERSION=25.11.4
ARG RHEL_ORG
ARG RHEL_ACTIVATION_KEY

LABEL stage=build

# Register with Red Hat, enable CRB, add EPEL, install build deps
RUN rm -rf /etc/rhsm-host \
    && subscription-manager register \
        --org="${RHEL_ORG}" \
        --activationkey="${RHEL_ACTIVATION_KEY}" \
    && subscription-manager repos \
        --enable codeready-builder-for-rhel-9-x86_64-rpms \
    && dnf install -y \
        https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm \
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
        libjwt-devel \
        jansson-devel \
        http-parser-devel \
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
RUN dnf install -y munge hwloc numactl lua mariadb-connector-c tini \
    && dnf clean all \
    && subscription-manager unregister || true

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
COPY --from=build /usr/bin/tini /usr/bin/tini
COPY --from=build /usr/lib64/libjansson.so* /usr/lib64/
COPY --from=build /usr/lib64/libjwt.so* /usr/lib64/
COPY --from=build /usr/lib64/libhttp_parser.so* /usr/lib64/

# Copy compiled Slurm binaries and libraries from build stage
COPY --from=build /tmp/slurm-install/ /

# Refresh shared library cache so all libs are found
RUN ldconfig

# Create munge user/group (needed for munge authentication)
RUN groupadd -g 981 munge \
    && useradd -u 981 -g munge -s /sbin/nologin -d /var/lib/munge -r munge

# Create slurm user/group matching Slinky chart expectations (UID/GID 401).
# OpenShift restricted-v2 SCC overrides the UID at runtime with one from
# the namespace range; directories must be group-writable (GID 0) so the
# assigned UID can still write.
RUN groupadd -g 401 slurm \
    && useradd -u 401 -g slurm -s /bin/bash -d /var/lib/slurm -m slurm \
    && usermod -aG 0 slurm

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
    && chown -R slurm:0 \
        /etc/slurm \
        /var/spool/slurm \
        /var/log/slurm \
    && chmod -R g+rwX \
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
