FROM debian:buster

ARG BUILD_ARCH
ARG S6_VER=3.2.0.9

SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV LANG=C.UTF-8 DEBIAN_FRONTEND=noninteractive

# ----------------------------------------------------------
# Peka om till Debian archive (efter EOL)
# ----------------------------------------------------------
RUN cat <<EOF > /etc/apt/sources.list
deb http://archive.debian.org/debian buster main contrib non-free
deb http://archive.debian.org/debian-security buster/updates main
EOF

RUN echo 'Acquire::Check-Valid-Until "false";' \
      > /etc/apt/apt.conf.d/99no-check

# ----------------------------------------------------------
# Installera grundläggande verktyg (så du får wget, ca-certificates etc.)
# ----------------------------------------------------------
RUN apt-get update && apt-get install -y \
      wget ca-certificates tar git xz-utils \
 && rm -rf /var/lib/apt/lists/*

# ----------------------------------------------------------
# Install Bashio (krävs för HA add-ons)
# ----------------------------------------------------------

# Hämta bashio till den katalog som HA förväntar sig
RUN git clone https://github.com/hassio-addons/bashio.git /usr/lib/bashio && \
    chmod -R +x /usr/lib/bashio

# ----------------------------------------------------------
# Bekvämlighets-symlinks (du hade detta och det är ok)
# ----------------------------------------------------------

# skapa symlinks för alla hjälpshellscripts
RUN mkdir -p /usr/local/bin && \
    for f in /usr/lib/bashio/lib/*.sh; do \
        ln -sf "$f" /usr/local/bin/$(basename "$f"); \
    done

# symlink för bashio själv
RUN ln -sf /usr/lib/bashio/bashio.sh /usr/local/bin/bashio

# ----------------------------------------------------------
# Installera S6-overlay (fungerande version)
# ----------------------------------------------------------
ARG S6_VER=3.2.1.0

RUN case "$BUILD_ARCH" in \
      amd64)   S6_ARCH=x86_64 ;; \
      armhf)   S6_ARCH=armhf ;; \
      armv7)   S6_ARCH=armhf ;; \
      aarch64) S6_ARCH=aarch64 ;; \
      *) echo "Unsupported arch: $BUILD_ARCH"; exit 1 ;; \
    esac && \
    wget -O /tmp/s6-noarch.tar.xz \
      https://github.com/just-containers/s6-overlay/releases/download/v${S6_VER}/s6-overlay-noarch.tar.xz && \
    wget -O /tmp/s6-arch.tar.xz \
      https://github.com/just-containers/s6-overlay/releases/download/v${S6_VER}/s6-overlay-${S6_ARCH}.tar.xz && \
    tar -C / -Jxpf /tmp/s6-noarch.tar.xz && \
    tar -C / -Jxpf /tmp/s6-arch.tar.xz && \
    rm /tmp/s6-*.tar.xz && \
    mkdir -p /run /data /etc/services.d

# ----------------------------------------------------------
# Installera allt annat du behöver (byggverktyg, Python, Telldus-beroenden etc.)
# ----------------------------------------------------------
RUN apt-get update && apt-get install -y \
      curl ca-certificates gnupg \
      git build-essential cmake pkg-config \
      libftdi1 libftdi1-dev \
      python2 python2-minimal python2-dev \
      python3 python3-pip \
      socat \
 && rm -rf /var/lib/apt/lists/*

# (OBS: wget + ca-certificates installerades tidigare, så inga fel)

# ----------------------------------------------------------
# Din övriga logik (libconfuse, Telldus repo, telldus-core, python-paket, symlinks etc.)
# ----------------------------------------------------------
ADD http://ftp.bme.hu/debian-archive/debian-amd64/pool/main/c/confuse/libconfuse0_2.5-1_amd64.deb /tmp/libconfuse.deb
RUN dpkg -i /tmp/libconfuse.deb || apt-get -f install -y

RUN wget -O- http://download.telldus.com/debian/telldus-public.key | gpg --dearmor > /usr/share/keyrings/telldus.gpg
RUN echo "deb [signed-by=/usr/share/keyrings/telldus.gpg] http://download.telldus.com/debian/ stable main" \
     > /etc/apt/sources.list.d/telldus.list

RUN apt-get update && apt-get install -y \
      telldus-core libtelldus-core2 libtelldus-core-dev \
 && rm -rf /var/lib/apt/lists/*

RUN pip3 install --no-cache-dir tellcore-py

RUN cd /usr/src && \
    git clone --depth 1 https://github.com/erijo/tellive-py.git && \
    cd tellive-py && \
    python3 setup.py install

COPY rootfs/ /
RUN chmod +x /usr/local/bin/td.py /usr/local/bin/tdtool-improved.py
RUN ln -s /usr/local/bin/td.py /usr/local/lib/python2.7/dist-packages/td.py && \
    ln -s /usr/local/bin/tdtool-improved.py /usr/local/lib/python2.7/dist-packages/tdtool_improved.py

ENTRYPOINT ["/init"]
