include $(TOPDIR)/rules.mk

PKG_NAME:=glibc
PKG_VERSION:=2.38
PKG_RELEASE:=1


PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://ftp.gnu.org/gnu/glibc/
PKG_HASH:=16e51e0455e288f03380b436e41d5927c60945abd86d0c9852b84be57dd6ed5e

PKG_CPE_ID:=cpe:/a:gnu:glibc

PKG_LICENSE:=LGPL-2.1-or-later GPL-2.0-or-later
PKG_LICENSE_FILES:=COPYING COPYING.LIB
PKG_MAINTAINER:=Jordan Ngako <jordan.ngako@mc-technologies.com>

PKG_BUILD_PARALLEL:=1
PKG_BUILD_DEPENDS:=libstdcpp

include $(INCLUDE_DIR)/package.mk


define Package/glibc
  SECTION:=libs
  CATEGORY:=Libraries
  TITLE:=GNU C Library (runtime) under /opt
  URL:=https://www.gnu.org/software/libc/
  DEPENDS:=+libstdcpp
endef

define Package/glibc/description
GNU C Library (glibc) runtime libraries installed under /opt/usr/lib on the target.
endef


# Base directory for std C++ headers for the target
CXX_STL_BASE := $(firstword $(wildcard \
  $(TOOLCHAIN_DIR)/aarch64-openwrt-linux-musl/include/c++/*))

GLIBC_CFLAGS   := -O2 -g -fno-stack-protector -U_FORTIFY_SOURCE
GLIBC_CPPFLAGS := -I$(STAGING_DIR)/glibc-linux-headers/include
GLIBC_LDFLAGS  :=

GCC_VER      := $(shell $(TARGET_CC) -dumpversion)
GCC_TARGET   := $(REAL_GNU_TARGET_NAME)

GCC_BUILTIN_INC := $(TOOLCHAIN_DIR)/lib/gcc/$(GCC_TARGET)/$(GCC_VER)/include
GCC_FIXED_INC   := $(TOOLCHAIN_DIR)/lib/gcc/$(GCC_TARGET)/$(GCC_VER)/include-fixed


GLIBC_CONFIGURE_ARGS := \
	--build=$(GNU_HOST_NAME) \
	--host=$(REAL_GNU_TARGET_NAME) \
	--prefix= \
	--disable-profile \
	--disable-werror \
	--without-gd \
	--without-cvs \
	--enable-add-ons \
	--enable-crypt \
	--enable-kernel=5.15.0 \
	--with-headers=$(STAGING_DIR)/glibc-linux-headers/include

#--with-headers="$(STAGING_DIR)/bpf-headers/user_headers/include/" 
CONFIGURE_VARS += \
	BUILD_CC="$(HOSTCC)" \
	CC="$(TARGET_CC)" \
	CXX="$(TARGET_CXX)" \
	AR="$(TARGET_AR)" \
	RANLIB="$(TARGET_RANLIB)" \
	AS="$(TARGET_AS)" \
	CXX_STL_BASE="$(CXX_STL_BASE)" \
	CXX_SYSINCLUDES="-isystem $(CXX_STL_BASE) -isystem $(CXX_STL_BASE)/$(REAL_GNU_TARGET_NAME)" \
	CXX_CSTDLIB_HEADER="$(CXX_STL_BASE)/cstdlib" \
	CXX_CMATH_HEADER="$(CXX_STL_BASE)/cmath" \
	CXX_BITS_STD_ABS_H="$(CXX_STL_BASE)/bits/std_abs.h" \
	CFLAGS="$(GLIBC_CFLAGS)" \
	CPPFLAGS="$(GLIBC_CPPFLAGS)" \
	LDFLAGS="$(GLIBC_LDFLAGS)"


define Build/Prepare
	$(call Build/Prepare/Default)

	# Fresh kernel-headers-only tree for glibc
	rm -rf $(STAGING_DIR)/glibc-linux-headers
	mkdir -p $(STAGING_DIR)/glibc-linux-headers/include

	# Copy only kernel/UAPI-style headers (no musl libc headers)
	for d in linux asm asm-generic drm mtd rdma scsi sound video xen misc net netinet netpacket; do \
		if [ -d "$(TOOLCHAIN_DIR)/include/$$$$d" ]; then \
			$(CP) -a "$(TOOLCHAIN_DIR)/include/$$$$d" \
			       "$(STAGING_DIR)/glibc-linux-headers/include/"; \
		fi; \
	done
endef


define Build/Configure
	mkdir -p $(PKG_BUILD_DIR)/build
	( cd $(PKG_BUILD_DIR)/build; \
	  echo "slibdir=/opt/usr/lib" >  configparms; \
	  echo "rtlddir=/opt/usr/lib" >> configparms; \
	  echo "sysincludes=-isystem $(GCC_BUILTIN_INC) -isystem $(GCC_FIXED_INC) \
	    -isystem $(STAGING_DIR)/glibc-linux-headers/include \
	    -isystem $(STAGING_DIR)/bpf-headers/user_headers/include" >> configparms; \
	  $(CONFIGURE_VARS) \
	  ../configure $(GLIBC_CONFIGURE_ARGS) \
	)
endef


define Build/Compile
	$(MAKE) -C $(PKG_BUILD_DIR)/build \
		$(if $(findstring c,$(MAKEFLAGS)),-s) \
		$(MAKE_FLAGS)
endef


define Build/Install
	$(MAKE) -C $(PKG_BUILD_DIR)/build \
		install install_root=$(PKG_INSTALL_DIR)
endef


define Package/glibc/install
	# Create target dir under /opt
	$(INSTALL_DIR) $(1)/opt/usr/lib

	# Just copy the runtime shared objects from glibc's install_root.
	# We do NOT package headers, binaries, or anything under /lib.
	$(CP) -a \
		$(PKG_INSTALL_DIR)/opt/usr/lib/*.so* \
		$(1)/opt/usr/lib/ 2>/dev/null || true

	# Also copy the dynamic loader(s) if they exist
	$(CP) -a \
		$(PKG_INSTALL_DIR)/opt/usr/lib/ld-*.so* \
		$(1)/opt/usr/lib/ 2>/dev/null || true

	$(CP) -a \
		$(PKG_INSTALL_DIR)/opt/usr/lib/ld-linux*.so* \
		$(1)/opt/usr/lib/ 2>/dev/null || true
endef


$(eval $(call BuildPackage,glibc))
