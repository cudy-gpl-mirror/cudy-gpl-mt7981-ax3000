include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk
include $(INCLUDE_DIR)/image.mk

PKG_NAME:=uboot-atf-mtk
PKG_VERSION=1.0
PKG_RELEASE:=20220606

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)/$(BUILD_VARIANT)

include $(INCLUDE_DIR)/package.mk

ifdef BUILD_VARIANT
UBOOT_CONFIG:=$(BUILD_VARIANT)
UBOOT_IMAGE:=$(if $(IMAGE),$(IMAGE),$(IMG_PREFIX)-$(BUILD_VARIANT)-uboot-atf-mtk.bin)
endif

ifeq ($(CONFIG_UBOOT_NAND),y)
UBOOT_MAX_SIZE:=2096128
else
UBOOT_MAX_SIZE:=523264
endif

UBOOT_BINARY:=u-boot.bin
FIP_BINARY:=fip.bin
BL2_BINARY:=bl2.img
OEM_UBOOT_BINARY:=oem-uboot.bin
OEM_UBOOT_TMP_BINARY:=oem-tmp-uboot.bin
OEM_UBOOT_VERSION:=00000003
OEM_UBOOT_STRING:="OEM_BOOT"
OEM_UBOOT_VERSION_FILE:="oem-uboot-version"
ATF_RELEASE:=$(PKG_BUILD_DIR)/atf/build/$(CONFIG_TARGET_SUBTARGET)/release

OemUbootDTS:=$(shell find ./configs/$(UBOOT_CONFIG)/ -name *.dts)
OemUbootDefconfig:=./configs/$(UBOOT_CONFIG)/$(UBOOT_CONFIG)_defconfig
OemAtfDefconfig:=./configs/$(UBOOT_CONFIG)/$(UBOOT_CONFIG)_atf.config
OemUbootKconfig:=./configs/$(UBOOT_CONFIG)/uboot_Kconfig
CONFIGS_DIR:=./configs/$(UBOOT_CONFIG)
define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
	$(CP) src/* $(PKG_BUILD_DIR)
	$(Build/Patch)
	[ ! -e $(OemUbootDefconfig) ] || cp -f $(OemUbootDefconfig) $(PKG_BUILD_DIR)/uboot/configs
	[ ! -e $(OemUbootDTS) ] || cp -f $(OemUbootDTS) $(PKG_BUILD_DIR)/uboot/arch/arm/dts/
	[ ! -e $(OemAtfDefconfig) ] || cp -f $(OemAtfDefconfig) $(PKG_BUILD_DIR)/atf/.config
	[ ! -e $(OemUbootKconfig) ]  || cp -f $(OemUbootKconfig) $(PKG_BUILD_DIR)/uboot/arch/arm/mach-mediatek/Kconfig
endef

define Build/Configure
	$(MAKE) -C $(PKG_BUILD_DIR)/uboot $(UBOOT_CONFIG)_defconfig
endef

define Package/sysupgrade-tools
  SECTION:=mtk
  CATEGORY:=mtk
  SUBMENU:=sysupgrade
  DEPENDS:= +libopenssl
  TITLE:=U-boot images tools (dumpimage, mkimage)
endef

define Build/Compile
	$(MAKE) -C $(PKG_BUILD_DIR)/uboot CROSS_COMPILE=/usr/bin/aarch64-linux-gnu-
	$(CP) $(PKG_BUILD_DIR)/uboot/$(UBOOT_BINARY) $(PKG_BUILD_DIR)/atf/
	$(MAKE) -C $(PKG_BUILD_DIR)/atf
endef

define Package/uboot-atf-mtk/common
  define Package/uboot-atf-mtk-$(1)
    SECTION:=mtk
    CATEGORY:=mtk
    SUBMENU:=boot
    TITLE:=U-boot for $(1)
    DEPENDS:=@TARGET_mediatek
    VARIANT:=$(1)
  endef
endef

define AppendFiles
	if [ `stat -c%s "$(1)"` -gt $(2) ]; then \
		echo "Warning: $(1) is too big"; \
	else \
		( dd if=$(1) bs=$(2) conv=sync) > $(3); \
	fi
endef

define CatFiles
	if [ `stat -c%s "$(1)"` -gt $(2) ]; then \
		echo "Warning: $(1) is too big"; \
	else if [ `stat -c%s $(3)` -gt $(4) ]; then \
		echo "Warning: $(3) is too big"; \
	else \
		( dd if=$(1) bs=$(2) conv=sync; dd if=$(3) ) > $(5); \
	fi; fi
endef

define OemBootVersion
	echo "$(OEM_UBOOT_STRING) $(OEM_UBOOT_VERSION)" >$1
endef

define Package/uboot-atf-mtk
  $(call Package/uboot-atf-mtk/common,$(1),$(2))

  define Package/uboot-atf-mtk-$(1)/config
	source "$$(SOURCE)/Config.in"
  endef

  define Package/uboot-atf-mtk-$(1)/install
	$(call AppendFiles,$(ATF_RELEASE)/$(FIP_BINARY),$(UBOOT_MAX_SIZE),$(ATF_RELEASE)/$(OEM_UBOOT_TMP_BINARY))
	$(call OemBootVersion,$(ATF_RELEASE)/$(OEM_UBOOT_VERSION_FILE))
	$(call CatFiles,$(ATF_RELEASE)/$(OEM_UBOOT_TMP_BINARY),$(UBOOT_MAX_SIZE),$(ATF_RELEASE)/$(OEM_UBOOT_VERSION_FILE),64,$(ATF_RELEASE)/$(OEM_UBOOT_BINARY))
	$(INSTALL_DATA) $(ATF_RELEASE)/$(OEM_UBOOT_BINARY) $(BIN_DIR)/$(UBOOT_IMAGE)
	$(INSTALL_DATA) $(ATF_RELEASE)/$(BL2_BINARY) $(BIN_DIR)/$(BL2_BINARY)
  endef

  $$(eval $$(call BuildPackage,uboot-atf-mtk-$(1)))
endef

define Package/sysupgrade-tools/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(CONFIGS_DIR)/dumpimage $(1)/usr/bin
endef

$(eval $(call Package/uboot-atf-mtk,R31))
$(eval $(call Package/uboot-atf-mtk,R36))
$(eval $(call Package/uboot-atf-mtk,R37))
$(eval $(call Package/uboot-atf-mtk,R47))
$(eval $(call Package/uboot-atf-mtk,R49))
$(eval $(call Package/uboot-atf-mtk,R50))
$(eval $(call Package/uboot-atf-mtk,R51))
$(eval $(call Package/uboot-atf-mtk,R58))
$(eval $(call Package/uboot-atf-mtk,R59))
$(eval $(call BuildPackage,sysupgrade-tools))