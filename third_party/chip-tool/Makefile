# Copyright (c) 2023-2024 Project CHIP Authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

include $(TOPDIR)/rules.mk

PKG_NAME:=chip-tool
PKG_RELEASE:=1
PKG_SOURCE_URL:=https://github.com/project-chip/connectedhomeip.git
PKG_SOURCE_PROTO:=git
PKG_SOURCE_MIRROR:=0 # don't try OpenWrt mirror
PKG_SOURCE_SUBMODULES:=\
  third_party/pigweed/repo \
  third_party/jsoncpp/repo \
  third_party/nlassert/repo \
  third_party/nlio/repo

# Hash can be regenerated with make package/chip-tool/check FIXUP=1
PKG_SOURCE_DATE:=2025-07-08
PKG_SOURCE_VERSION:=dda2252eb28440305c883d0204737dce5b3abe62
PKG_MIRROR_HASH:=ad81137221dafc0302c70cb90a03ab2472a02ee3c03e8cef8edcba26fd34201d

# Use local source dir for development
# USE_SOURCE_DIR:=$(HOME)/workspace/connectedhomeip

PKG_LICENSE:=Apache-2.0
PKG_LICENSE_FILES:=LICENSE

# Python 3.11 is required for code generation; use the python3 host package if necessary.
# Because it is installed with a broken SSL configuration in some cases (including the
# OpenWrt SDK docker images), we use the python3-host-ssl package to work around this.
HAVE_PYTHON_311:=$(shell $(STAGING_DIR_HOST)/bin/$(PYTHON) -c 'import sys; print(sys.version_info>=(3,11) or "")')
PKG_BUILD_DEPENDS:=gn/host $(if $(HAVE_PYTHON_311),,python3-host-ssl/host)
PKG_BUILD_PARALLEL:=1

include $(INCLUDE_DIR)/package.mk

define Package/chip-tool/default
  SECTION:=net
  CATEGORY:=Network
  TITLE:=CHIP Tool
  URL:=https://github.com/project-chip/connectedhomeip
  DEPENDS:=+libstdcpp +libatomic
endef

define Package/chip-tool/default/description
  CHIP Tool
  A command-line tool for interacting with Matter devices.
endef

define Package/chip-tool-mbedtls
  $(Package/chip-tool/default)
  TITLE+= (mbedtls)
  DEPENDS+= +PACKAGE_chip-tool-mbedtls:libmbedtls
  VARIANT:=mbedtls
  DEFAULT_VARIANT:=1
endef

define Package/chip-tool-mbedtls/description
$(Package/chip-tool/default/description)

This variant of the package uses the mbedTLS crypto library,
which must have the MBEDTLS_CCM_C option enabled.
endef

define Package/chip-tool-openssl
  $(Package/chip-tool/default)
  TITLE+= (openssl)
  DEPENDS+= +PACKAGE_chip-tool-openssl:libopenssl
  CONFLICTS:=chip-tool-mbedtls
  VARIANT:=openssl
endef

define Package/chip-tool-openssl/description
$(Package/chip-tool/default/description)

This variant of the package uses the OpenSSL crypto library.
endef

# General options
CONFIGURE_OPTIONS:=\
	--enable-recommended=no \
	--enable-access-restrictions=no \
	--enable-wifi=no \
	--enable-openthread=no \
	--enable-network-layer-ble=no \
	--enable-tracing-support=no

# Logging backend
CONFIGURE_OPTIONS+= --logging-backend=syslog
TARGET_CFLAGS+= -DCHIP_SYSLOG_IDENT=\"chip-tool\"

# Crypto backend
ifeq ($(BUILD_VARIANT),mbedtls)
CONFIGURE_OPTIONS+= --crypto=mbedtls --enable-external-mbedtls
TARGET_LDFLAGS+= -lmbedcrypto -lmbedx509
endif
ifeq ($(BUILD_VARIANT),openssl)
CONFIGURE_OPTIONS+= --crypto=openssl
endif

# Configuration / persistence
CONF_DIR:=/etc/chip-tool
TARGET_CFLAGS+=\
	-DCHIP_CONFIG_KVS_PATH=\"$(CONF_DIR)/chip_tool_kvs.ini\" \
	-DFATCONFDIR=\"$(CONF_DIR)\" \
	-DSYSCONFDIR=\"$(CONF_DIR)\" \
	-DLOCALSTATEDIR=\"$(CONF_DIR)\"

# Device Information. Runtime information is configured by bootstrap.sh
include $(INCLUDE_DIR)/version.mk
OS_VERSION:=$(if $(filter-out SNAPSHOT,$(VERSION_NUMBER)),$(VERSION_NUMBER)-)$(VERSION_CODE)
TARGET_CFLAGS+=\
	-DCHIP_DEVICE_CONFIG_ENABLE_TEST_SETUP_PARAMS=0 \
	-DCHIP_DEVICE_CONFIG_DEVICE_SOFTWARE_VERSION_STRING=\"$(PKG_VERSION)@$(OS_VERSION)\" \
	-DCHIP_DEVICE_CONFIG_DEFAULT_DEVICE_HARDWARE_VERSION_STRING=\"\" \
	-DCHIP_DEVICE_CONFIG_DEVICE_VENDOR_NAME=\"$(VERSION_DIST)\" \
	-DCHIP_DEVICE_CONFIG_DEVICE_PRODUCT_NAME=\"\"

# https://github.com/openwrt/openwrt/issues/13016
TARGET_CFLAGS+= -Wno-format-nonliteral

# The build environment contains host tools that can be shared between targets
CHIP_BUILD_ENV_DIR:=$(STAGING_DIR_HOST)/share/chip-build-env
OUT_DIR:=$(PKG_BUILD_DIR)/out/openwrt

define Build/Configure
	mkdir -p $(OUT_DIR) && cd $(OUT_DIR) && \
		$(CONFIGURE_VARS) $(PKG_BUILD_DIR)/scripts/configure \
			--build-env-dir="$(CHIP_BUILD_ENV_DIR)" \
			--project=examples/chip-tool \
			--target=$(GNU_TARGET_NAME) \
			$(CONFIGURE_OPTIONS)
endef

define Build/Compile
	$(NINJA) -C $(OUT_DIR) chip-tool
endef

define Package/chip-tool/default/install
	$(INSTALL_DIR) $(1)/usr/bin $(1)/usr/share/chip-tool $(1)/etc/init.d
	$(INSTALL_BIN) $(OUT_DIR)/chip-tool $(1)/usr/bin
	$(INSTALL_DATA) ./files/bootstrap.sh $(1)/usr/share/chip-tool/
	$(INSTALL_BIN) ./files/chip-tool.init $(1)/etc/init.d/chip-tool
endef

Package/chip-tool-mbedtls/install=$(Package/chip-tool/default/install)
Package/chip-tool-openssl/install=$(Package/chip-tool/default/install)

define Package/chip-tool/default/conffiles
$(CONF_DIR)/
endef

Package/chip-tool-mbedtls/conffiles=$(Package/chip-tool/default/conffiles)
Package/chip-tool-openssl/conffiles=$(Package/chip-tool/default/conffiles)

$(eval $(call BuildPackage,chip-tool-mbedtls))
$(eval $(call BuildPackage,chip-tool-openssl))