#
# Copyright (C) 2017-2019 Chen Minqiang <ptpt52@gmail.com>
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=natflow
PKG_VERSION:=20220401

PKG_SOURCE_URL:=https://codeload.github.com/ptpt52/natflow/tar.gz/$(PKG_VERSION)?
PKG_HASH:=skip
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz

PKG_MAINTAINER:=Chen Minqiang <ptpt52@gmail.com>
PKG_LICENSE:=GPL-2.0

include $(INCLUDE_DIR)/package.mk

define KernelPackage/natflow
  CATEGORY:=X
  SUBMENU:=Fast Forward Stacks
  TITLE:=natflow kernel driver
  KCONFIG:= \
	    CONFIG_NETFILTER_INGRESS=y
  FILES:=$(PKG_BUILD_DIR)/natflow.ko
  AUTOLOAD:=$(call AutoLoad,96,natflow)
  DEPENDS:= +kmod-ipt-conntrack +kmod-ipt-nat +kmod-ipt-ipset +kmod-br-netfilter
endef

define KernelPackage/natflow/description
  fast nat forward kmod
endef

include $(INCLUDE_DIR)/kernel-defaults.mk

EXTRA_CFLAGS += -DCONFIG_NATFLOW_PATH -DCONFIG_NATFLOW_URLLOGGER
ifneq ($(CONFIG_TARGET_mediatek_mt7622),)
EXTRA_CFLAGS += -DCONFIG_HWNAT_EXTDEV_USE_VLAN_HASH
endif

define Build/Compile/natflow
	+$(MAKE) $(PKG_JOBS) -C "$(LINUX_DIR)" \
		EXTRA_CFLAGS="$(EXTRA_CFLAGS)" \
		$(KERNEL_MAKE_FLAGS) \
		ARCH="$(LINUX_KARCH)" \
		CROSS_COMPILE="$(KERNEL_CROSS)" \
		M="$(PKG_BUILD_DIR)" \
		modules
endef

define Build/Compile
	$(call Build/Compile/natflow)
endef

define Package/natflow-boot
  CATEGORY:=X
  SUBMENU:=Fast Forward Stacks
  TITLE:=natflow boot init script
  DEPENDS:= +kmod-natflow
endef

define Package/natflow-boot/install
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/natflow-boot.init $(1)/etc/init.d/natflow-boot
endef

define Package/natflow-auth
  CATEGORY:=X
  SUBMENU:=Fast Forward Stacks
  TITLE:=natflow auth init script
  DEPENDS:= +kmod-natflow +ipset +lua-ipops
endef

define Package/natflow-auth/conffiles
/etc/config/natflow
endef

define Package/natflow-auth/install
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/natflow-user.init $(1)/etc/init.d/natflow-user
	$(INSTALL_BIN) ./files/natflow-zone.init $(1)/etc/init.d/natflow-zone
	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_DATA) ./files/natflow.config $(1)/etc/config/natflow
	$(INSTALL_DIR) $(1)/etc/hotplug.d/iface
	$(INSTALL_DATA) ./files/20-natflow-zone.hotplug $(1)/etc/hotplug.d/iface/20-natflow-zone
endef

$(eval $(call KernelPackage,natflow))
$(eval $(call BuildPackage,natflow-boot))
$(eval $(call BuildPackage,natflow-auth))
