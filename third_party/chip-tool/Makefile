include $(TOPDIR)/rules.mk

PKG_NAME:=chip-tool
PKG_RELEASE:=1

PKG_SOURCE_URL:=https://github.com/project-chip/connectedhomeip.git
PKG_SOURCE_PROTO:=git
PKG_SOURCE_MIRROR:=0 # don't try OpenWrt mirror
PKG_SOURCE_SUBMODULES:= \
  third_party/pigweed \
  third_party/jsoncpp \
  third_party/nlassert \
  third_party/editline \
  third_party/nlunit-test \
  third_party/nlio \
  third_party/perfetto \
  third_party/libwebsockets

# Hash can be regenerated with make package/chip-tool/check FIXUP=1
PKG_SOURCE_DATE:=2025-07-08
PKG_SOURCE_VERSION:=dda2252eb28440305c883d0204737dce5b3abe62
PKG_MIRROR_HASH:=ad81137221dafc0302c70cb90a03ab2472a02ee3c03e8cef8edcba26fd34201d

# Use local source dir for development
# USE_SOURCE_DIR:=$(HOME)/workspace/connectedhomeip

PKG_LICENSE:=Apache-2.0
PKG_LICENSE_FILES:=LICENSE
PKG_BUILD_PARALLEL:=1
PKG_BUILD_DEPENDS:=gn/host
PKG_CONFIG_DEPENDS:= CONFIG_CHIP_OTA_PROVIDER CONFIG_CHIP_BRIDGE CONFIG_CHIP_TOOL

include $(INCLUDE_DIR)/package.mk
TARGET_CXXFLAGS += -DFATCONFDIR=\"/root\" -DSYSCONFDIR=\"/root\" -DLOCALSTATEDIR=\"/root\" -Wno-format-nonliteral -Wno-unused -Wno-unknown-warning-option -fexceptions -latomic
TARGET_CFLAGS += -DFATCONFDIR=\"/root\" -DSYSCONFDIR=\"/root\" -DLOCALSTATEDIR=\"/root\" -Wno-format-nonliteral -Wno-unused -Wno-unknown-warning-option -fexceptions
TARGET_LDFLAGS += -latomic

# The build environment contains host tools that can be shared between targets
CHIP_BUILD_ENV_DIR:=$(STAGING_DIR_HOST)/share/chip-build-env
CHIP_BRIDGE_OUT_DIR:=$(PKG_BUILD_DIR)/out/chip-bridge
CHIP_TOOL_OUT_DIR:=$(PKG_BUILD_DIR)/out/chip-tool
CHIP_OTA_PROVIDER_OUT_DIR:=$(PKG_BUILD_DIR)/out/chip-ota-provider

# https://github.com/project-chip/matter-openwrt/issues/2
ifeq ($(DUMP)$(filter SUBMODULES:=,$(Download/Defaults)),)
$(error PKG_SOURCE_SUBMODULES is not supported, ensure https://github.com/openwrt/openwrt/pull/13000 is included in your OpenWrt buildroot)
endif

define Package/chip
	SECTION:=net
	CATEGORY:=Network
	TITLE:=AVAKS Matter Tool
	URL:=https://github.com/project-chip/connectedhomeip
	DEPENDS:=+libstdcpp +libopenssl +glib2 +libavahi-client
	MENU:=1
endef

define Package/chip-tool/description
  Matter Host Features
  Integrates a router / access point with the Matter IoT ecosystem.
endef

define Package/chip-tool/config
	source "$(SOURCE)/Config.in"
endef

define Build/Configure
	[ ! -d "$(CHIP_OTA_PROVIDER_OUT_DIR) " ] && mkdir -p $(CHIP_OTA_PROVIDER_OUT_DIR)
	[ ! -d "$(CHIP_BRIDGE_OUT_DIR)" ] && mkdir -p $(CHIP_BRIDGE_OUT_DIR)
	[ ! -d "$(CHIP_TOOL_OUT_DIR)" ] && mkdir -p $(CHIP_TOOL_OUT_DIR)

	@if [ "$(CONFIG_CHIP_OTA_PROVIDER)" = "y" ]; then \
		if [ ! -f $(CHIP_OTA_PROVIDER_OUT_DIR)/build.ninja ]; then \
			echo "Configuring OTA Provider"; \
			cd $(CHIP_OTA_PROVIDER_OUT_DIR) && \
			$(CONFIGURE_VARS) $(PKG_BUILD_DIR)/scripts/configure --build-env-dir="$(CHIP_BUILD_ENV_DIR)" \
				--project=examples/ota-provider-app/linux \
				--target=$(GNU_TARGET_NAME) \
				--enable-network-layer-ble=no \
				--enable-openthread=no \
				--enable-wifi=no \
				--mdns='platform' \
				--enable-tracing-support=no; \
		else echo "Configuring OTA Provider no update"; fi \
	else echo "Configuring OTA Provider is disable"; fi

	@if [ "$(CONFIG_CHIP_BRIDGE)" = "y" ]; then \
		if [ ! -f $(CHIP_BRIDGE_OUT_DIR)/build.ninja ]; then \
			echo "Configuring Bridge"; \
			cd $(CHIP_BRIDGE_OUT_DIR) && \
			$(CONFIGURE_VARS) $(PKG_BUILD_DIR)/scripts/configure \
				--build-env-dir="$(CHIP_BUILD_ENV_DIR)" \
				--project=examples/bridge-app/linux \
				--target=$(GNU_TARGET_NAME) \
				--enable-network-layer-ble=no \
				--enable-openthread=no \
				--enable-wifi=no \
				--mdns='platform' \
				--enable-tracing-support=no; \
		else echo "Configuring Bridge no update"; fi \
	else echo "Configuring Bridge is disable"; fi

	@if [ "$(CONFIG_CHIP_TOOL)" = "y" ]; then \
		if [ ! -f $(CHIP_TOOL_OUT_DIR)/build.ninja ]; then \
			echo "Configuring Chip Tool"; \
			cd $(CHIP_TOOL_OUT_DIR) && \
			$(CONFIGURE_VARS) $(PKG_BUILD_DIR)/scripts/configure \
				--build-env-dir="$(CHIP_BUILD_ENV_DIR)" \
				--project=examples/chip-tool \
				--target=$(GNU_TARGET_NAME) \
				--enable-tracing-support=no; \
		else echo "Configuring Chip Tool no update"; fi \
	else echo "Configuring Chip Tool is disable"; fi
endef

define Build/Compile
	@if [ "$(CONFIG_CHIP_OTA_PROVIDER)" = "y" ]; then \
		ninja -C $(CHIP_OTA_PROVIDER_OUT_DIR) chip-ota-provider-app; \
	fi
	@if [ "$(CONFIG_CHIP_BRIDGE)" = "y" ]; then \
		ninja -C $(CHIP_BRIDGE_OUT_DIR) chip-bridge-app; \
	fi
	@if [ "$(CONFIG_CHIP_TOOL)" = "y" ]; then \
		ninja -C $(CHIP_TOOL_OUT_DIR) chip-tool; \
	fi
endef

define Build/Clean
endef

define Package/chip/install
	$(INSTALL_DIR) $(1)/usr/sbin
	@if [ "$(CONFIG_CHIP_OTA_PROVIDER)" = "y" ]; then \
		$(INSTALL_BIN) $(CHIP_OTA_PROVIDER_OUT_DIR)/chip-ota-provider-app $(1)/usr/sbin/ota-provider-app; \
	fi
	@if [ "$(CONFIG_CHIP_BRIDGE)" = "y" ]; then \
		$(INSTALL_BIN) $(CHIP_BRIDGE_OUT_DIR)/chip-bridge-app $(1)/usr/sbin/bridge-app; \
	fi
	@if [ "$(CONFIG_CHIP_TOOL)" = "y" ]; then \
		$(INSTALL_BIN) $(CHIP_TOOL_OUT_DIR)/chip-tool $(1)/usr/sbin/chip-tool; \
	fi
endef

$(eval $(call BuildPackage,chip))