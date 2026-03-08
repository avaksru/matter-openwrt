ARG OPENWRT_VERSION=24.10
ARG TARGET=imx/cortexa7

FROM openwrt/sdk:${OPENWRT_VERSION}-${TARGET// /-}   # или openwrtorg/sdk:${OPENWRT_VERSION}-${TARGET// /-}

# или если вы используете полностью кастомный билд:

# RUN git clone ... openwrt.git && cd openwrt \
#     && ./scripts/feeds update -a \
#     && echo "src-link matter $(pwd)/../matter-feed" >> feeds.conf.default \
#     && ./scripts/feeds install -a -p matter \
#     && echo "CONFIG_PACKAGE_matter=y" >> .config \
#     && make defconfig \
#     && echo "CONFIG_TARGET_${TARGET//\//_}=y" >> .config \
#     && make -j$(nproc)