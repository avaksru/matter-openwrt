ARG OPENWRT_VERSION=25.12.0
ARG TARGET=imx/cortexa7

FROM ubuntu:24.04

RUN apt-get update && apt-get install -y \
    build-essential clang flex bison g++ gawk \
    gcc-multilib g++-multilib gettext git libncurses5-dev \
    libssl-dev python3-distutils rsync unzip zlib1g-dev \
    file wget curl vim subversion \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /openwrt

RUN git clone https://git.openwrt.org/openwrt/openwrt.git . \
 && git checkout v${OPENWRT_VERSION} || git checkout openwrt-${OPENWRT_VERSION}

# Добавьте ваш feed matter
COPY matter-feed /openwrt/feeds/matter

RUN ./scripts/feeds update -a \
 && ./scripts/feeds install -a -p matter \
 && echo "CONFIG_TARGET_imx=y" >> .config \
 && echo "CONFIG_TARGET_imx_cortexa7=y" >> .config \
 && echo "CONFIG_PACKAGE_matter=y" >> .config \
 && make defconfig \
 && make download -j$(nproc) \
 && make -j$(nproc) V=s || make -j1 V=s

# Артефакты будут в /openwrt/bin/