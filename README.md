Qpid-proton-c Migration to OpenWRT
----------------------------------------------
[Apache Qpid™](http://qpid.apache.org/) makes messaging tools that speak AMQP and support many languages and platforms.

AMQP is an open internet protocol for reliably sending and receiving messages. It makes it possible for everyone to build a diverse, coherent messaging ecosystem.

This tutorial will guide you to build a OpenWRT migration for Qpid-proton-c core library.

## Prerequisite
###Setup OpenWRT BuildRoot environment:

In your linux host, setup the OpenWRT cross-compile environment 
With [This](http://wiki.openwrt.org/doc/howto/buildroot.exigence) tutorial.


###Install OpenWRT cross-compile SDK and required libraries for qpid-proton:
Config building options:

	$cd openwrt/trunk
	$make menuconfig

![menuconfig] (./menuconfig.png)

In the menuconfig, choose your build platform, and select *build the OpenWrt SDK*,
also select 2 modules needed by qpid-proton: *library->SSL, library->libuuid* 

Save and exit.

Begin making your SDK:

	$make V=s 

This may take about 2 hours to compile a SDK evnironment.

After compiling finished, add short url for convenience:

	$ln -s ~/openwrt/trunk/bin/ramips/OpenWrt-SDK-ramips-for-linux-x86_64-gcc-4.8-linaro_uClibc-0.9.33.2 ~/sdk
	$ln -s ~/sdk/staging_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2 ~/tc
	$ln -s ~/sdk/staging_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2 ~/target

> Note: The directory name above may various by the building platform you selected. In this tutorial, we use [WRTNode](http://wrtnode.com/) as an example.

Copy required library to toolchain directory:

	$cd ~/target/usr/lib
	$cp libuuid* libcrypto* libssl* ~/tc/usr/lib/
	$cd ~/target/usr/include
	$cp -r uuid openssl crypto ~/tc/usr/include/ 


## Setup package building file structure

Setup file structure:

	cd ~/sdk/package
	mkdir openwrt-qpid-proton
	cd openwrt-qpid-proton
	cp ~/qpid-proton-0.8.tar.gz ./
	tar xvf qpid-proton-0.8.tar.gz
	mv qpid-proton-0.8 src


Change Makefile:

	vi Makefile

to the following content:

	Makefile:
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


Add contents to CMakefile:

	vi src/proton-c/CMakeLists.txt

Section added:

	SET(IN_OPENWRT $ENV{IN_OPENWRT})
	
	#IF (IN_OPENWRT)
	        ADD_DEFINITIONS("$ENV{TARGET_LDFLAGS}" "$ENV{TARGET_CPPFLAGS}" "$ENV{TARGET_CFLAGS}")
	        INCLUDE_DIRECTORIES("$ENV{TOOLCHAIN_DIR}/usr/include" "$ENV{TARGET_LDFLAGS}" "$ENV{TARGET_CPPFLAGS}" "$ENV{TARGET_CFLAGS}")
	        INCLUDE_DIRECTORIES("$ENV{TOOLCHAIN_DIR}/usr/include/openssl")
	        INCLUDE_DIRECTORIES("$ENV{TOOLCHAIN_DIR}/usr/include/uuid")
	#ENDIF(IN_OPENWRT)



## Build package

Build package:

	cd ~/sdk
	make V=s

An ipk package will be generated at: ~/sdk/bin/ramips/packages

## Deploy in Target host

install in your OpenWRT node:

	#opkg install openwrt-qpid-proton_0.8-1_ramips_24kec.ipk


## Test your package deployed

Test connection to your AMQP broker on your OpenWRT node:
	
	#cd /usr/share/proton/examples/
	-- send message:
	#./send -a amqps://username:password@brokerUrl
	
	-- receive message:
	#./recv amqps://username:password@brokerUrl

