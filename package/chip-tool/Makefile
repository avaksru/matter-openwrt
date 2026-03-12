include $(TOPDIR)/rules.mk

PKG_NAME:=chip-tool
PKG_VERSION:=1.5.0.1
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/project-chip/connectedhomeip.git
PKG_SOURCE_VERSION:=v1.5.0.1
PKG_SOURCE_SUBDIR:=$$   (PKG_NAME)-   $$(PKG_VERSION)
PKG_SOURCE:=$(PKG_SOURCE_SUBDIR).tar.zst
PKG_MIRROR_HASH:=sha256:6646e275772cd61d84ee895a4bb36685f4c2529baab7146bc1c85e4512cdacc2

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