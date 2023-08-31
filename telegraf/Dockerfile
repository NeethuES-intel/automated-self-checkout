#
# Copyright (C) 2023 Intel Corporation.
#
# SPDX-License-Identifier: Apache-2.0
#

FROM ubuntu:22.04

ARG IS_FLEX

RUN if [ -n "$HTTP_PROXY" ] ; then  echo "Acquire::http::Proxy \"$HTTP_PROXY\";" >  /etc/apt/apt.conf; fi

# Build and install igt for Arc and iGPU. Note although igt works for Flex, xpumcli is officially supported for Flex monitoring
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    build-essential \
    wget \
    gpg-agent \
    ca-certificates \
    git \
    cmake  \
    libunwind-dev \
    libgsl-dev \
    libasound2-dev \
    libxmlrpc-core-c3-dev \
    libjson-c-dev \
    libcurl4-openssl-dev \
    python3-docutils \
    valgrind \
    peg \
    libdrm-intel1 \
    pkg-config libdrm-dev libkmod-dev libprocps-dev libdw-dev libpixman-1-dev libcairo-dev libudev-dev flex bison \
    meson intel-gpu-tools net-tools && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Build and install telegraf/influxdb: Reference: https://docs.influxdata.com/telegraf/v1.21/introduction/installation/
WORKDIR /opt/intel-gpu-telegraf

RUN  apt-get update -y || true; DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends gnupg2 \
 && \
 apt-get clean && \
 rm -rf /var/lib/apt/lists/*

RUN \
## flex gpu drivers
 apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 9B10C065DBB72B06; \
 wget -qO - https://repositories.intel.com/graphics/intel-graphics.key | gpg --dearmor --output /usr/share/keyrings/intel-graphics.gpg; \
 echo 'deb [arch=amd64 signed-by=/usr/share/keyrings/intel-graphics.gpg] https://repositories.intel.com/graphics/ubuntu jammy flex' | tee  /etc/apt/sources.list.d/intel.gpu.jammy.list

RUN apt-get update -y || true; DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends intel-gsc level-zero intel-level-zero-gpu pciutils \
 && \
 apt-get clean && \
 rm -rf /var/lib/apt/lists/*

RUN wget https://github.com/intel/xpumanager/releases/download/V1.2.6/xpu-smi_1.2.6_20230322.060757.98a08a04_u22.04_amd64.deb; dpkg -i ./xpu-smi_1.2.6_20230322.060757.98a08a04_u22.04_amd64.deb


RUN \
 wget -q https://repos.influxdata.com/influxdata-archive_compat.key; \
 echo '393e8779c89ac8d958f81f942f9ad7fb82a25e133faddaf92e15b16e6ac9ce4c influxdata-archive_compat.key' | sha256sum -c && cat influxdata-archive_compat.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg > /dev/null; \
echo 'deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg] https://repos.influxdata.com/debian stable main' | tee /etc/apt/sources.list.d/influxdata.list

RUN apt-get update -y || true; DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends telegraf \
 && \
 apt-get clean && \
 rm -rf /var/lib/apt/lists/*

COPY ./opt/intel-gpu-telegraf /opt/intel-gpu-telegraf
RUN if [ -n "$IS_FLEX" ] ; then cp /opt/intel-gpu-telegraf/telegraf-flex.conf /opt/intel-gpu-telegraf/telegraf.conf; fi

RUN telegraf --sample-config > telegraf-sample.conf

CMD ["/usr/bin/telegraf", "--config", "/opt/intel-gpu-telegraf/telegraf.conf"]
