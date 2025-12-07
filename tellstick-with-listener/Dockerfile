FROM debian:buster

ARG BUILD_ARCH
ARG S6_VER=3.2.0.2

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV LANG=C.UTF-8 \
    DEBIAN_FRONTEND=noninteractive

# Install S6
RUN case "$BUILD_ARCH" in \
      amd64)   S6_ARCH=x86_64 ;; \
      armhf)   S6_ARCH=armhf ;; \
      armv7)   S6_ARCH=armhf ;; \
      aarch64) S6_ARCH=aarch64 ;; \
      *) echo "Unsupported arch $BUILD_ARCH"; exit 1 ;; \
    esac && \
    wget -O /tmp/s6.tar.gz \
      https://github.com/just-containers/s6-overlay/releases/download/v${S6_VER}/s6-overlay-${S6_ARCH}.tar.gz && \
    tar xzf /tmp/s6.tar.gz -C / && \
    rm /tmp/s6.tar.gz

# ----------------------------------------------------------
# Basic system + build deps
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
# Install S6-overlay
# ----------------------------------------------------------

# required by HA
ARG BUILD_ARCH

# choose version
ARG S6_VER=3.2.0.2

# install overlay
ADD https://github.com/just-containers/s6-overlay/releases/download/v${S6_VER}/s6-overlay-${BUILD_ARCH}.tar.gz /tmp/s6.tar.gz
RUN tar xzf /tmp/s6.tar.gz -C / \
 && rm /tmp/s6.tar.gz \
 && mkdir -p /run /data /etc/services.d

# ----------------------------------------------------------
# Addon files
# ----------------------------------------------------------

COPY rootfs/ /

RUN chmod +x /usr/local/bin/td.py /usr/local/bin/tdtool-improved.py

# ----------------------------------------------------------
# Install old libconfuse required by telldus-core
# ----------------------------------------------------------

ADD http://ftp.bme.hu/debian-archive/debian-amd64/pool/main/c/confuse/libconfuse0_2.5-1_amd64.deb /tmp/libconfuse.deb
RUN dpkg -i /tmp/libconfuse.deb || apt-get -f install -y

# ----------------------------------------------------------
# Telldus apt repo
# ----------------------------------------------------------

RUN wget -O- http://download.telldus.com/debian/telldus-public.key \
    | gpg --dearmor \
    > /usr/share/keyrings/telldus.gpg

RUN echo "deb [signed-by=/usr/share/keyrings/telldus.gpg] http://download.telldus.com/debian/ stable main" \
    > /etc/apt/sources.list.d/telldus.list

RUN apt-get update && apt-get install -y \
    telldus-core libtelldus-core2 libtelldus-core-dev \
 && rm -rf /var/lib/apt/lists/*

# ----------------------------------------------------------
# Python libs
# ----------------------------------------------------------

RUN pip3 install --no-cache-dir tellcore-py

RUN cd /usr/src \
 && git clone --depth 1 https://github.com/erijo/tellive-py.git \
 && cd tellive-py \
 && python3 setup.py install

# ----------------------------------------------------------
# Python2 symlinks (som du hade)
# ----------------------------------------------------------

RUN ln -s /usr/local/bin/td.py /usr/local/lib/python2.7/dist-packages/td.py && \
    ln -s /usr/local/bin/tdtool-improved.py /usr/local/lib/python2.7/dist-packages/tdtool_improved.py

# ----------------------------------------------------------
# Required for HA addon
# ----------------------------------------------------------

ENTRYPOINT ["/init"]
