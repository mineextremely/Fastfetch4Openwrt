include $(TOPDIR)/rules.mk
-include $(dir $(lastword $(MAKEFILE_LIST)))version.mk

PKG_NAME:=fastfetch
PKG_VERSION:=$(or $(FASTFETCH_VERSION),2.63.0)
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://codeload.github.com/fastfetch-cli/fastfetch/tar.gz/$(PKG_VERSION)?
PKG_HASH:=8bf9de6c031aeca8afb85bca5aec2ee82034b339525e2862e0c1dcc6d470d9c0

PKG_LICENSE:=MIT
PKG_LICENSE_FILES:=LICENSE
PKG_MAINTAINER:=Nova

PKG_BUILD_PARALLEL:=1
PKG_BUILD_FLAGS:=no-mips16

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/cmake.mk

TARGET_CFLAGS += -ffunction-sections -fdata-sections
TARGET_LDFLAGS += -Wl,--gc-sections

CMAKE_OPTIONS += \
	-DBUILD_FLASHFETCH=OFF \
	-DENABLE_SYSTEMD=OFF \
	-DENABLE_X11=OFF \
	-DENABLE_WAYLAND=OFF \
	-DENABLE_DRM=OFF \
	-DENABLE_PULSE=OFF \
	-DENABLE_OPENGL=OFF \
	-DENABLE_VULKAN=OFF \
	-DENABLE_SQLITE3=OFF \
	-DENABLE_IMAGEMAGICK6=OFF \
	-DENABLE_IMAGEMAGICK7=OFF

define Package/fastfetch
  SECTION:=utils
  CATEGORY:=Utilities
  TITLE:=Fast system information tool
  URL:=https://github.com/fastfetch-cli/fastfetch
endef

define Package/fastfetch/description
 Fastfetch is a fast neofetch-like system information tool.
endef

define Package/fastfetch/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/fastfetch $(1)/usr/bin/
endef

$(eval $(call BuildPackage,fastfetch))