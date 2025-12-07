# ----------------------------------------------------------
# Debian Buster base
# ----------------------------------------------------------

FROM debian:buster

ARG BUILD_ARCH
ARG S6_VER=3.2.0.9

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV LANG=C.UTF-8 \
    DEBIAN_FRONTEND=noninteractive

# map arch
RUN case "$BUILD_ARCH" in \
      amd64)   echo "x86_64" > /tmp/s6arch ;; \
      armhf)   echo "armhf"  > /tmp/s6arch ;; \
      armv7)   echo "armhf"  > /tmp/s6arch ;; \
      aarch64) echo "aarch64"> /tmp/s6arch ;; \
      *) echo "Unsupported arch"; exit 1 ;; \
    esac

# read computed arch
ARG S6_ARCH
RUN export S6_ARCH=$(cat /tmp/s6arch) && echo "S6 arch = $S6_ARCH"

# install S6
ADD https://github.com/just-containers/s6-overlay/releases/download/v${S6_VER}/s6-overlay-noarch.tar.xz /tmp/
ADD https://github.com/just-containers/s6-overlay/releases/download/v${S6_VER}/s6-overlay-$(cat /tmp/s6arch).tar.xz /tmp/

RUN tar -C / -Jxpf /tmp/s6-overlay-noarch.tar.xz && \
    tar -C / -Jxpf /tmp/s6-overlay-$(cat /tmp/s6arch).tar.xz && \
    rm /tmp/*.tar.xz && \
    mkdir -p /run /data /etc/services.d

# ----------------------------------------------------------
# Basic deps
# ----------------------------------------------------------

RUN apt-get update && apt-get install -y \
    curl ca-certificates gnupg \
    git build-essential cmake pkg-config \
    libftdi1 libftdi1-dev \
    python2 python2-minimal python2-dev \
    python3 python3-pip \
    wget socat \
 && rm -rf /var/lib/apt/lists/*

# ----------------------------------------------------------
# Addon files
# ----------------------------------------------------------

COPY rootfs/ /

RUN chmod +x /usr/local/bin/td.py \
 && chmod +x /usr/local/bin/tdtool-improved.py

# ----------------------------------------------------------
# Old libconfuse needed by telldus-core
# ----------------------------------------------------------

ADD http://ftp.bme.hu/debian-archive/debian-amd64/pool/main/c/confuse/libconfuse0_2.5-1_amd64.deb /tmp/libconfuse.deb
RUN dpkg -i /tmp/libconfuse.deb || apt-get -f install -y

# ----------------------------------------------------------
# Telldus repo + install
# ----------------------------------------------------------

RUN wget -O- http://download.telldus.com/debian/telldus-public.key \
    | gpg --dearmor \
    > /usr/share/keyrings/telldus.gpg

RUN echo "deb [signed-by=/usr/share/keyrings/telldus.gpg] \
http://download.telldus.com/debian/ stable main" \
    > /etc/apt/sources.list.d/telldus.list

RUN apt-get update && apt-get install -y \
    telldus-core libtelldus-core2 libtelldus-core-dev \
 && rm -rf /var/lib/apt/lists/*

# ----------------------------------------------------------
# Python libs
# ----------------------------------------------------------

RUN pip3 install --no-cache-dir tellcore-py

RUN cd /usr/src && \
    git clone --depth 1 https://github.com/erijo/tellive-py.git && \
    cd tellive-py && \
    python3 setup.py install

# ----------------------------------------------------------
# Python2 symlinks
# ----------------------------------------------------------

RUN ln -s /usr/local/bin/td.py \
        /usr/local/lib/python2.7/dist-packages/td.py && \
    ln -s /usr/local/bin/tdtool-improved.py \
        /usr/local/lib/python2.7/dist-packages/tdtool_improved.py

# ----------------------------------------------------------
# HA required
# ----------------------------------------------------------

ENTRYPOINT ["/init"]
