include $(TOPDIR)/rules.mk

PKG_NAME:=chip-tool
PKG_VERSION:=1.4.0
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/project-chip/connectedhomeip.git
PKG_SOURCE_VERSION:=main   # или конкретный тег, напр. release/v1.4.0.0
PKG_MIRROR_HASH:=skip      # потом замени на реальный после первой сборки

PKG_BUILD_DEPENDS:=gn-host ninja-host python3-host python-yaml-host pkgconf-host
PKG_BUILD_FLAGS:=no-mips16

include $(INCLUDE_DIR)/package.mk

define Package/chip-tool
  SECTION:=net
  CATEGORY:=Network
  TITLE:=Matter chip-tool
  DEPENDS:=+libpthread +librt +libstdcpp +libatomic +libcurl +libjson-c +libopenssl \
           +zlib +libev +libsqlite3 +libnl +libusb-1.0 +bluez-libs
  URL:=https://github.com/project-chip/connectedhomeip
endef

define Package/chip-tool/description
  Matter commissioning tool (chip-tool)
endef

define Build/Configure
	+cd $(PKG_BUILD_DIR) && \
		./scripts/activate.sh && \
		gn gen out/armv7 \
			--args=' \
				target_os="linux" \
				target_cpu="arm" \
				arm_version=7 \
				chip_build_libshell=true \
				chip_build_tests=false \
				chip_enable_openthread=false \
				chip_enable_ble=true \
				treat_warnings_as_errors=false \
				target_sysroot="$(STAGING_DIR)" \
			'
endef

define Build/Compile
	ninja -C $(PKG_BUILD_DIR)/out/armv7 chip-tool
endef

define Package/chip-tool/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/out/armv7/chip-tool $(1)/usr/bin/
	$(STRIP) $(1)/usr/bin/chip-tool
endef

$(eval $(call BuildPackage,chip-tool))