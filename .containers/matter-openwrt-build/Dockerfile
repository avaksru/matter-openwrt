# .containers/matter-openwrt-build/Dockerfile

ARG OPENWRT_VERSION=25.12.0   # или ваша дефолтная версия
ARG TARGET=imx/cortexa7

# Заменяем / на _ , потому что официальные теги используют подчёркивание
FROM openwrt/sdk:${OPENWRT_VERSION}-${TARGET//\//_}

# ────────────────────────────────────────────────────────────────
# Здесь ваша логика: добавление feeds, установка пакетов Matter и т.д.
# ────────────────────────────────────────────────────────────────

# Пример (адаптируйте под свой проект):
USER root
RUN mkdir -p /builder/feeds \
 && echo "src-link matter /matter-feed" >> /builder/feeds.conf.default \
 && /builder/scripts/feeds update -a \
 && /builder/scripts/feeds install -a -p matter \
 && echo "CONFIG_PACKAGE_matter=y" >> /builder/.config \
 && echo "CONFIG_PACKAGE_matter-examples=y" >> /builder/.config \
 && make defconfig \
 && make package/matter/compile V=s -j$(nproc) \
 && make package/index V=s

# Опционально: копируем артефакты в известное место
RUN mkdir -p /artifacts \
 && cp -r /builder/bin/packages/* /artifacts/ 2>/dev/null || true \
 && cp -r /builder/bin/targets/${TARGET//\//_}/* /artifacts/images/ 2>/dev/null || true