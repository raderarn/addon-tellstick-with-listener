# ----------------------------------------------------------
# Debian bas, Python, Telldus + Tellive + Python2
# ----------------------------------------------------------

ARG BUILD_FROM=debian:buster-slim
FROM ${BUILD_FROM}

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION

ENV LANG=C.UTF-8 \
    DEBIAN_FRONTEND=noninteractive

RUN sed -i 's/deb.debian.org/archive.debian.org/g' /etc/apt/sources.list \
 && sed -i 's/security.debian.org/archive.debian.org/g' /etc/apt/sources.list \
 && echo "Acquire::Check-Valid-Until false;" > /etc/apt/apt.conf.d/99no-check

# ----------------------------------------------------------
# Install system packages
# ----------------------------------------------------------

RUN apt update && apt install -y \
        git \
        build-essential \
        cmake \
        pkg-config \
        libftdi1 \
        libftdi1-dev \
        python3 \
        python3-pip \
        python2 \
        python2-minimal \
        python2-dev \
        socat \
    && rm -rf /var/lib/apt/lists/*

# ----------------------------------------------------------
# Build telldus-core
# ----------------------------------------------------------

RUN set -eux; \
    git clone https://github.com/erik73/telldus.git /src/telldus; \
    cd /src/telldus/telldus-core; \
    git checkout v2.1.1; \
    cmake . \
        -DBUILD_LIBTELLDUS-CORE=ON \
        -DBUILD_TDADMIN=OFF \
        -DBUILD_TDTOOL=ON \
        -DGENERATE_MAN=OFF \
        -DBUILD_FROM_TRUNK=ON \
        -DWITH_X11=OFF \
        -DWITH_MACOS=OFF \
        -DWITH_WINDOWS=OFF ; \
    make -j$(nproc); \
    make install

# ----------------------------------------------------------
# Install Python libs
# ----------------------------------------------------------

RUN pip3 install --no-cache-dir tellcore-py

RUN cd /usr/src \
    && git clone --depth 1 https://github.com/erijo/tellive-py.git \
    && cd tellive-py \
    && python3 setup.py install

# ----------------------------------------------------------
# Copy addon data
# ----------------------------------------------------------

COPY rootfs /

# ----------------------------------------------------------
# Metadata
# ----------------------------------------------------------

LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.version=${BUILD_VERSION}

