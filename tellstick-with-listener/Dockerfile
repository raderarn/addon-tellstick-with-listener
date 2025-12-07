#-----------------------------------------------------------
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

# Copy add-on rootfs into container
COPY rootfs/ /
RUN chmod +x /usr/local/bin/td.py /usr/local/bin/tdtool-improved.py

RUN sed -i 's/deb.debian.org/archive.debian.org/g' /etc/apt/sources.list \
 && sed -i 's/security.debian.org/archive.debian.org/g' /etc/apt/sources.list \
 && echo "Acquire::Check-Valid-Until false;" > /etc/apt/apt.conf.d/99no-check

# Install libconfuse0 manually (required by telldus-core)
ADD http://ftp.bme.hu/debian-archive/debian-amd64/pool/main/c/confuse/libconfuse0_2.5-1_amd64.deb /tmp/libconfuse.deb
RUN dpkg -i /tmp/libconfuse.deb || apt-get -f install -y

# Fix EOL repo again (bug in Debian requires re-applying)
RUN sed -i 's|deb http://deb.debian.org/debian|deb http://archive.debian.org/debian|g' /etc/apt/sources.list \
 && echo 'Acquire::Check-Valid-Until "false";' > /etc/apt/apt.conf.d/99no-check-valid

# ----------------------------------------------------------
# Install system packages
# ----------------------------------------------------------
RUN apt update && apt upgrade -y

RUN apt update && \
        apt install -y \
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
	wget \
        socat \
    && rm -rf /var/lib/apt/lists/*

# ----------------------------------------------------------
# Build telldus-core
# ----------------------------------------------------------
RUN wget -O- http://download.telldus.com/debian/telldus-public.key \
    | gpg --dearmor \
    > /usr/share/keyrings/telldus.gpg

# Add Telldus repo
RUN echo "deb [signed-by=/usr/share/keyrings/telldus.gpg] \
http://download.telldus.com/debian/ stable main" \
    > /etc/apt/sources.list.d/telldus.list

# Install telldus-core via apt (not apt-get)
RUN apt update && \
    apt install -y \
        telldus-core \
        libtelldus-core2 \
        libtelldus-core-dev && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*

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
RUN ln -s /usr/local/bin/td.py /usr/local/lib/python2.7/dist-packages/td.py && \
    ln -s /usr/local/bin/tdtool-improved.py /usr/local/lib/python2.7/dist-packages/tdtool_improved.py

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

