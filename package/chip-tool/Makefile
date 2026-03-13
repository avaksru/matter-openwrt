include $(TOPDIR)/rules.mk

PKG_NAME:=chip-tool
PKG_VERSION:=1.5.0.1
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/project-chip/connectedhomeip.git
PKG_SOURCE_VERSION:=v1.5.0.1
PKG_MIRROR_HASH:=sha256:6646e275772cd61d84ee895a4bb36685f4c2529baab7146bc1c85e4512cdacc2

PKG_BUILD_DEPENDS:=gn-host ninja-host python3-host python-yaml-host pkgconf-host bash git
PKG_BUILD_FLAGS:=no-mips16

include $(INCLUDE_DIR)/package.mk

define Package/chip-tool
  SECTION:=net
  CATEGORY:=Network
  TITLE:=Matter chip-tool
  DEPENDS:=+libpthread +librt +libstdcpp +libatomic +libcurl +libjson-c +libopenssl +zlib +libev +libsqlite3 +libnl +libusb-1.0 +bluez-libs
endef

define Build/Prepare
	# Пропускаем стандартную prepare, если нужно
endef

define Build/Configure
	# Временно закомментируй activate.sh, если проблема в нём
	# cd $(PKG_BUILD_DIR) && ./scripts/activate.sh || true
	echo "Configure stub"
endef

define Build/Compile
	echo "Compile stub - add real commands later"
endef

define Package/chip-tool/install
	echo "Install stub"
endef

$(eval $(call BuildPackage,chip-tool))