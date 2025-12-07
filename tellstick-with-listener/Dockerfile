#-----------------------------------------------------------
# Home Assistant Debian base (Bullseye runtime)
# Force APT to use Debian Buster Archive
#-----------------------------------------------------------

FROM ghcr.io/home-assistant/amd64-base-debian:bullseye

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV LANG=C.UTF-8 \
    DEBIAN_FRONTEND=noninteractive

# Redirect Bullseye → Buster + archive.debian.org
RUN sed -i 's|bullseye|buster|g' /etc/apt/sources.list \
 && sed -i 's|deb.debian.org|archive.debian.org|g' /etc/apt/sources.list \
 && sed -i 's|security.debian.org|archive.debian.org|g' /etc/apt/sources.list \
 && echo "Acquire::Check-Valid-Until false;" > /etc/apt/apt.conf.d/99no-check

# Copy add-on rootfs into container
COPY rootfs/ /
RUN chmod +x /usr/local/bin/td.py /usr/local/bin/tdtool-improved.py

# Install libconfuse0 manually
ADD http://ftp.bme.hu/debian-archive/debian-amd64/pool/main/c/confuse/libconfuse0_2.5-1_amd64.deb /tmp/libconfuse.deb
RUN dpkg -i /tmp/libconfuse.deb || apt-get -f install -y

# Update and install dependencies
RUN apt update && \
    apt upgrade -y && \
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
        socat && \
    rm -rf /var/lib/apt/lists/*

# Telldus key
RUN wget -O- http://download.telldus.com/debian/telldus-public.key \
    | gpg --dearmor \
    > /usr/share/keyrings/telldus.gpg

# Add Telldus repo
RUN echo "deb [signed-by=/usr/share/keyrings/telldus.gpg] http://download.telldus.com/debian/ stable main" \
    > /etc/apt/sources.list.d/telldus.list

# Install Telldus-core
RUN apt update && \
    apt install -y \
        telldus-core \
        libtelldus-core2 \
        libtelldus-core-dev && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*

# Install Python libs
RUN pip3 install --no-cache-dir tellcore-py

RUN cd /usr/src \
    && git clone --depth 1 https://github.com/erijo/tellive-py.git \
    && cd tellive-py \
    && python3 setup.py install

# Copy addon data
COPY rootfs /
RUN ln -s /usr/local/bin/td.py /usr/local/lib/python2.7/dist-packages/td.py && \
    ln -s /usr/local/bin/tdtool-improved.py /usr/local/lib/python2.7/dist-packages/tdtool_improved.py
