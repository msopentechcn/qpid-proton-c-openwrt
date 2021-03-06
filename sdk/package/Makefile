#
# Copyright (C) 2006 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#
include $(TOPDIR)/rules.mk

PKG_NAME:=openwrt-qpid-proton
PKG_VERSION:=0.8
PKG_RELEASE:=1

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)

include $(INCLUDE_DIR)/package.mk

define Package/openwrt-qpid-proton
  SECTION:=libs
  CATEGORY:=Libraries
  DEPENDS:=+libopenssl +libuuid
  TITLE:=qpid-proton v0.8 c migration to OpenWRT
endef

define Build/Prepare
        mkdir -p $(PKG_BUILD_DIR)
        $(CP) ./src/* $(PKG_BUILD_DIR)/
        rm -f $(PKG_BUILD_DIR)/CMakeCache.txt
        rm -fR $(PKG_BUILD_DIR)/CMakeFiles
        rm -f $(PKG_BUILD_DIR)/Makefile ]
        rm -f $(PKG_BUILD_DIR)/cmake_install.cmake
endef

define Build/Configure
  IN_OPENWRT=1 \
  AR="$(TOOLCHAIN_DIR)/bin/$(TARGET_CROSS)ar" \
  AS="$(TOOLCHAIN_DIR)/bin/$(TARGET_CC) -c $(TARGET_CFLAGS)" \
  LD="$(TOOLCHAIN_DIR)/bin/$(TARGET_CROSS)ld" \
  NM="$(TOOLCHAIN_DIR)/bin/$(TARGET_CROSS)nm" \
  CC="$(TOOLCHAIN_DIR)/bin/$(TARGET_CC)" \
  GCC="$(TOOLCHAIN_DIR)/bin/$(TARGET_CC)" \
  CXX="$(TOOLCHAIN_DIR)/bin/$(TARGET_CROSS)g++" \
  RANLIB="$(TOOLCHAIN_DIR)/bin/$(TARGET_CROSS)ranlib" \
  STRIP="$(TOOLCHAIN_DIR)/bin/$(TARGET_CROSS)strip" \
  OBJCOPY="$(TOOLCHAIN_DIR)/bin/$(TARGET_CROSS)objcopy" \
        OBJDUMP="$(TOOLCHAIN_DIR)/bin/$(TARGET_CROSS)objdump" \
        TARGET_CPPFLAGS="$(TARGET_CPPFLAGS)" \
        TARGET_CFLAGS="$(TARGET_CFLAGS)" \
        TARGET_LDFLAGS="$(TARGET_LDFLAGS)" \
        cmake -DCMAKE_FIND_ROOT_PATH="$(TOOLCHAIN_DIR)" $(PKG_BUILD_DIR)/CMakeLists.txt
endef

define Build/InstallDev
        $(INSTALL_DIR) $(STAGING_DIR)/usr/include/proton
        $(CP) $(PKG_BUILD_DIR)/proton-c/include/* $(STAGING_DIR)/usr/include/
        $(INSTALL_DIR) $(STAGING_DIR)/usr/lib
        $(CP) $(PKG_BUILD_DIR)/proton-c/libqpid-proton.so.2 $(STAGING_DIR)/usr/lib/
endef

define Build/UninstallDev
        rm -rf \
        $(STAGING_DIR)/usr/include/* \
        $(STAGING_DIR)/usr/lib/*
endef

define Build/Compile
        $(MAKE) -C $(PKG_BUILD_DIR)
        $(STRIP) $(PKG_BUILD_DIR)/proton-c/libqpid-proton.so*
endef

define Package/openwrt-qpid-proton/install
        $(INSTALL_DIR) $(1)/usr/lib
        $(INSTALL_DIR) $(1)/usr/share/proton/examples
        $(INSTALL_BIN) $(PKG_BUILD_DIR)/proton-c/libqpid-proton.so.2 $(1)/usr/lib/
        $(INSTALL_BIN) $(PKG_BUILD_DIR)/examples/messenger/c/recv* $(1)/usr/share/proton/examples/
        $(INSTALL_BIN) $(PKG_BUILD_DIR)/examples/messenger/c/send* $(1)/usr/share/proton/examples/
endef

$(eval $(call BuildPackage,openwrt-qpid-proton))
