##########################################################################################
#	prepare param
##########################################################################################
export OSDRV_DIR=$(shell pwd)
export OSDRV_CROSS
export CHIP
export ARCH=arm
export CROSS_COMPILE=$(OSDRV_CROSS)-
export OSDRV_CROSS_CFLAGS
export FLASH_TYPE

#if CHIP is support, we set SUPPORT_CHIP = "yes"
SUPPORT_CHIP =: "no"

ifeq ($(OSDRV_CROSS), )
$(error you must set OSDRV_CROSS first!)
endif 

ifeq ($(CHIP), )
$(error you must set CHIP first!)
endif 

ifeq ($(OSDRV_CROSS),arm-hisiv500-linux)
LIB_TYPE:=uclibc
TOOLCHAIN_DIR:=arm-hisiv500-linux
CROSS_SPECIFIED:=yp
endif

ifeq ($(OSDRV_CROSS),arm-hisiv600-linux)
LIB_TYPE:=glibc
TOOLCHAIN_DIR:=arm-hisiv600-linux
CROSS_SPECIFIED:=y
endif

BUSYBOX_VER:=busybox-1.20.2
TOOLCHAIN_RUNTIME_LIB_C:=lib.tgz

ifneq ($(CROSS_SPECIFIED),y)
all:
	@echo "---------------------------------------------------------------------"
	@echo "ERROR: you should have specified an OSDRV_CROSS!                     "
	@echo "e.g., make OSDRV_CROSS=arm-hisiv500-linux	all 	    	    "
	@echo "e.g., make OSDRV_CROSS=arm-hisiv600-linux 	all	    	    "
	@echo "---------------------------------------------------------------------"
endif

ifeq ($(CHIP),hi3516av200)
UIMAGE:=uImage_hi3516av200
UBOOT:=u-boot-hi3516av200.bin
JFFS2_IMAGE_BIN_64K:=rootfs_hi3516av200_64k.jffs2
JFFS2_IMAGE_BIN_128K:=rootfs_hi3516av200_128k.jffs2
JFFS2_IMAGE_BIN_256K:=rootfs_hi3516av200_256k.jffs2
YAFFS2_IMAGE_BIN_2K_4BIT:=rootfs_hi3516av200_2k_4bit.yaffs2
YAFFS2_IMAGE_BIN_2K_24BIT:=rootfs_hi3516av200_2k_24bit.yaffs2
EXT4_IMAGE_BIN:=rootfs_hi3516av200_32M.ext4
SUPPORT_CHIP:= "yes"
ifeq ($(FLASH_TYPE), nand)
export UBOOT_CONFIG:=hi3516av200_nand_config
export KERNEL_CFG:=hi3516av200_big_little_nand_defconfig
else
export UBOOT_CONFIG:=hi3516av200_config
export KERNEL_CFG:=hi3516av200_big_little_defconfig
endif
ifeq ($(TARGET_XLSM),)
TARGET_XLSM:=Hi3516AV200-DMEB-uboot-DDR3_1600M_32bit_1GB-A7_792M-A17_930M-BUS_200M.xlsm
endif
endif

ifeq ($(CHIP),hi3519v101)
UIMAGE:=uImage_hi3519v101
UBOOT:=u-boot-hi3519v101.bin
JFFS2_IMAGE_BIN_64K:=rootfs_hi3519v101_64k.jffs2
JFFS2_IMAGE_BIN_128K:=rootfs_hi3519v101_128k.jffs2
JFFS2_IMAGE_BIN_256K:=rootfs_hi3519v101_256k.jffs2
YAFFS2_IMAGE_BIN_2K_4BIT:=rootfs_hi3519v101_2k_4bit.yaffs2
YAFFS2_IMAGE_BIN_2K_24BIT:=rootfs_hi3519v101_2k_24bit.yaffs2
EXT4_IMAGE_BIN:=rootfs_hi3519v101_32M.ext4
SUPPORT_CHIP:= "yes"
ifeq ($(FLASH_TYPE), nand)
export UBOOT_CONFIG:=hi3519v101_nand_config
export KERNEL_CFG:=hi3519v101_big_little_nand_defconfig
else
export UBOOT_CONFIG:=hi3519v101_config
export KERNEL_CFG:=hi3519v101_big_little_defconfig
endif
ifeq ($(TARGET_XLSM),)
TARGET_XLSM:=Hi3519V101-DMEB-uboot-DDR3_1866M_32bit_1GB-A7_792M-A17_930M-BUS_200M.xlsm
endif
endif

UBOOT_VER:=u-boot-2010.06
UBOOT_TAR:=$(UBOOT_VER).tgz
KERNEL_VER:=linux-3.18.y
KERNEL_TAR:=$(KERNEL_VER).tgz

YAFFS_TOOL:=mkyaffs2image100

TOOLCHAIN_RUNTIME_LIB:=a17_a7_softfp_neon-vfpv4
OSDRV_CROSS_CFLAGS:=-mcpu=cortex-a17.cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4
ifeq ($(OSDRV_CROSS),arm-hisiv500-linux)
BUSYBOX_CFG:=config_v500_a17_a7_softfp_neon
endif
ifeq ($(OSDRV_CROSS),arm-hisiv600-linux)
BUSYBOX_CFG:=config_v600_a17_a7_softfp_neon
endif

EXT4_TOOL:=make_ext4fs

HIREGBING_PACKAGE_VER:=hiregbin-v5.0.0
UBOOT_REG_BIN:=reg_info.bin

export PUB_BOARD:=board_$(LIB_TYPE)_big-little
export PUB_IMAGE:=image_$(LIB_TYPE)_big-little
export PUB_ROOTFS:=rootfs_$(LIB_TYPE)_big-little

ifneq ($(SUPPORT_CHIP), "yes")
$(error $(CHIP) not support!)
endif
##########################################################################################
#	set task
##########################################################################################
ifeq ($(CROSS_SPECIFIED),y)
all: prepare hiboot hikernel hidrv hirootfs_prepare hibusybox hipctools hiboardtools hirootfs_build
notools: hiboot hikernel hidrv hinotools_prepare hirootfs_notools_build

clean: hiboot_clean hikernel_clean hidrv_clean \
       hibusybox_clean hipctools_clean hiboardtools_clean hirootfs_clean

distclean:clean pub_clean
endif

##########################################################################################
#task [0]	prepare
##########################################################################################
prepare:
	mkdir $(OSDRV_DIR)/pub/$(PUB_IMAGE) -p
	mkdir $(OSDRV_DIR)/pub/bin/$(PUB_BOARD) -p
	mkdir $(OSDRV_DIR)/pub/bin/pc -p

##########################################################################################
#	prepare hiregbin tools
##########################################################################################
check_hiregbin:
HIREGBING_PACKAGE = $(shell find $(OSDRV_DIR)/tools/pc/uboot_tools/ \
					-name "hiregbin-v5.0.0.tgz")
#$(warning $(HIREGBING_PACKAGE))

ifeq ($(HIREGBING_PACKAGE), )
$(warning "---------------------------------------------------------------------")
$(warning "     Cannot found the hiregbin-v5.0.0.tgz package !!                 ")
$(warning "---------------------------------------------------------------------")
$(error )
endif

##########################################################################################
#	hiregbin_prepare
##########################################################################################
hiregbin_prepare:
	@echo "---------task [0]    hiregbin_prepare"
	tar xzf $(HIREGBING_PACKAGE) -C $(OSDRV_DIR)/tools/pc/uboot_tools
	chmod 777 $(OSDRV_DIR)/tools/pc/uboot_tools/$(HIREGBING_PACKAGE_VER)/hiregbin
	cp $(OSDRV_DIR)/tools/pc/uboot_tools/$(TARGET_XLSM) $(OSDRV_DIR)/tools/pc/uboot_tools/$(HIREGBING_PACKAGE_VER)
	pushd $(OSDRV_DIR)/tools/pc/uboot_tools/$(HIREGBING_PACKAGE_VER);./hiregbin $(TARGET_XLSM) $(UBOOT_REG_BIN);popd
	mv $(OSDRV_DIR)/tools/pc/uboot_tools/$(HIREGBING_PACKAGE_VER)/$(UBOOT_REG_BIN) $(OSDRV_DIR)/tools/pc/uboot_tools
	rm $(OSDRV_DIR)/tools/pc/uboot_tools/$(HIREGBING_PACKAGE_VER) -rf

##########################################################################################
hinotools_prepare:
	pushd $(OSDRV_DIR)/pub;tar xzf $(PUB_ROOTFS).tgz;popd
##########################################################################################
#task [1]	build uboot
##########################################################################################
hiboot: prepare hiregbin_prepare
	@echo "---------task [1]	build boot"
	rm $(OSDRV_DIR)/opensource/uboot/$(UBOOT_VER) -rf
	tar xzf $(OSDRV_DIR)/opensource/uboot/$(UBOOT_TAR) -C $(OSDRV_DIR)/opensource/uboot
	find $(OSDRV_DIR)/opensource/uboot/$(UBOOT_VER) | xargs touch
	make -C $(OSDRV_DIR)/opensource/uboot/$(UBOOT_VER) ARCH=arm CROSS_COMPILE=$(OSDRV_CROSS)- $(UBOOT_CONFIG)
	pushd $(OSDRV_DIR)/opensource/uboot/$(UBOOT_VER);make ARCH=arm CROSS_COMPILE=$(OSDRV_CROSS)- -j 20 >/dev/null;popd
	cp $(OSDRV_DIR)/opensource/uboot/$(UBOOT_VER)/u-boot.bin $(OSDRV_DIR)/pub/$(PUB_IMAGE)
	cp $(OSDRV_DIR)/tools/pc/uboot_tools/mkboot.sh $(OSDRV_DIR)/pub/$(PUB_IMAGE)
	cp $(OSDRV_DIR)/tools/pc/uboot_tools/$(UBOOT_REG_BIN) $(OSDRV_DIR)/pub/$(PUB_IMAGE)
	chmod 777 $(OSDRV_DIR)/pub/$(PUB_IMAGE)/mkboot.sh
	pushd $(OSDRV_DIR)/pub/$(PUB_IMAGE);./mkboot.sh $(UBOOT_REG_BIN) $(UBOOT);popd
	rm $(OSDRV_DIR)/pub/$(PUB_IMAGE)/u-boot.bin
	rm $(OSDRV_DIR)/pub/$(PUB_IMAGE)/mkboot.sh
	rm $(OSDRV_DIR)/pub/$(PUB_IMAGE)/$(UBOOT_REG_BIN)

hiboot_clean:
	rm $(OSDRV_DIR)/opensource/uboot/$(UBOOT_VER) -rf

##########################################################################################
#task [2]	build kernel
##########################################################################################
hikernel: prepare
	@echo "---------task [2] build kernel"
	rm $(OSDRV_DIR)/opensource/kernel/$(KERNEL_VER) -rf
	tar xzf $(OSDRV_DIR)/opensource/kernel/$(KERNEL_TAR) -C $(OSDRV_DIR)/opensource/kernel/
	find $(OSDRV_DIR)/opensource/kernel/$(KERNEL_VER) | xargs touch
	make -C $(OSDRV_DIR)/opensource/kernel/$(KERNEL_VER) ARCH=arm CROSS_COMPILE=$(OSDRV_CROSS)- $(KERNEL_CFG)
	pushd $(OSDRV_DIR)/opensource/kernel/$(KERNEL_VER);make ARCH=arm CROSS_COMPILE=$(OSDRV_CROSS)- -j 20 >/dev/null;popd
	pushd $(OSDRV_DIR)/opensource/kernel/$(KERNEL_VER);\
		make ARCH=arm CROSS_COMPILE=$(OSDRV_CROSS)- uImage -j 20 >/dev/null;popd
	cp $(OSDRV_DIR)/opensource/kernel/$(KERNEL_VER)/arch/arm/boot/uImage $(OSDRV_DIR)/pub/$(PUB_IMAGE)/$(UIMAGE)

hikernel_clean:
	rm $(OSDRV_DIR)/opensource/kernel/$(KERNEL_VER) -rf

##########################################################################################
#task [3]       prepare drv
##########################################################################################
hidrv:
	@echo "---------task [3] build drv"
ifeq ($(PCI_MODE),slave)
	pushd $(OSDRV_DIR)/drv/pcie_mcc;make clean;popd
	pushd $(OSDRV_DIR)/drv/pcie_mcc;make VER=2 target=slave >/dev/null;popd
	mkdir $(OSDRV_DIR)/pub/bin/$(PUB_BOARD)/pci -p
	cp $(OSDRV_DIR)/drv/pcie_mcc/out/*.ko $(OSDRV_DIR)/pub/bin/$(PUB_BOARD)/pci/
	cp $(OSDRV_DIR)/drv/pcie_mcc/include $(OSDRV_DIR)/pub/bin/$(PUB_BOARD)/pci/ -af
endif
ifeq ($(PCI_MODE),master)
	pushd $(OSDRV_DIR)/drv/pcie_mcc;make clean;chmod 777 ./mkbooter.sh;popd
	pushd $(OSDRV_DIR)/drv/pcie_mcc;make VER=2 SOFT_DDRT=y >/dev/null;popd
	mkdir $(OSDRV_DIR)/pub/bin/$(PUB_BOARD)/pci -p
	cp $(OSDRV_DIR)/drv/pcie_mcc/out/*.ko $(OSDRV_DIR)/pub/bin/$(PUB_BOARD)/pci/
	cp $(OSDRV_DIR)/drv/pcie_mcc/out/booter $(OSDRV_DIR)/pub/bin/$(PUB_BOARD)/pci/
	cp $(OSDRV_DIR)/drv/pcie_mcc/include $(OSDRV_DIR)/pub/bin/$(PUB_BOARD)/pci/ -af
endif

hidrv_clean:
	make -C $(OSDRV_DIR)/drv/pcie_mcc clean

##########################################################################################
#task [4]	prepare rootfs 
##########################################################################################
hirootfs_prepare: prepare
	@echo "---------task [4] prepare rootfs "
	rm $(OSDRV_DIR)/pub/$(PUB_ROOTFS)* -rf
	tar xzf $(OSDRV_DIR)/rootfs_scripts/rootfs.tgz -C $(OSDRV_DIR)/pub
	mv $(OSDRV_DIR)/pub/rootfs $(OSDRV_DIR)/pub/$(PUB_ROOTFS)
	tar xzf $(OSDRV_DIR)/opensource/toolchain/$(TOOLCHAIN_DIR)/runtime_lib/$(TOOLCHAIN_RUNTIME_LIB)/$(TOOLCHAIN_RUNTIME_LIB_C) -C $(OSDRV_DIR)/pub/$(PUB_ROOTFS)

##########################################################################################
#task [5]	build busybox 
##########################################################################################
hibusybox: prepare
	@echo "---------task [5] build busybox "
	rm $(OSDRV_DIR)/opensource/busybox/$(BUSYBOX_VER) -rf
	tar xzf $(OSDRV_DIR)/opensource/busybox/$(BUSYBOX_VER).tgz -C $(OSDRV_DIR)/opensource/busybox
	find $(OSDRV_DIR)/opensource/busybox/$(BUSYBOX_VER) | xargs touch
	cp $(OSDRV_DIR)/opensource/busybox/$(BUSYBOX_VER)/$(BUSYBOX_CFG) \
		$(OSDRV_DIR)/opensource/busybox/$(BUSYBOX_VER)/.config
	pushd $(OSDRV_DIR)/opensource/busybox/$(BUSYBOX_VER)/;make -j 20 >/dev/null;popd
	make -C $(OSDRV_DIR)/opensource/busybox/$(BUSYBOX_VER) install
	cp -af $(OSDRV_DIR)/opensource/busybox/$(BUSYBOX_VER)/_install/* $(OSDRV_DIR)/pub/$(PUB_ROOTFS)

hibusybox_clean:
	rm $(OSDRV_DIR)/opensource/busybox/$(BUSYBOX_VER) -rf

##########################################################################################
#task [6]	build pc tools
##########################################################################################
hipctools: prepare
	@echo "---------task [6] build tools which run on pc"
	make -C $(OSDRV_DIR)/tools/pc/mkyaffs2image/
	cp $(OSDRV_DIR)/tools/pc/mkyaffs2image/bin/$(YAFFS_TOOL) $(OSDRV_DIR)/pub/bin/pc
	find $(OSDRV_DIR)/tools/pc/jffs2_tool -type f | xargs -I % sed -i 's/sys\/types/sys\/sysmacros/g' %
	make -C $(OSDRV_DIR)/tools/pc/jffs2_tool/
	cp $(OSDRV_DIR)/tools/pc/jffs2_tool/mkfs.jffs2 $(OSDRV_DIR)/pub/bin/pc
	cp $(OSDRV_DIR)/tools/pc/jffs2_tool/mkfs.ubifs $(OSDRV_DIR)/pub/bin/pc
	cp $(OSDRV_DIR)/tools/pc/jffs2_tool/ubinize $(OSDRV_DIR)/pub/bin/pc
#	
	make -C $(OSDRV_DIR)/tools/pc/cramfs_tool/
	cp $(OSDRV_DIR)/tools/pc/cramfs_tool/mkfs.cramfs $(OSDRV_DIR)/pub/bin/pc
#	find $(OSDRV_DIR)/tools/pc/mkimage_tool -type f | xargs -I % sed -i 's/sys\/types/sys\/sysmacros.h>\n#include<sys\/types/g' %
	make -C $(OSDRV_DIR)/tools/pc/mkimage_tool/
	cp $(OSDRV_DIR)/tools/pc/mkimage_tool/mkimage $(OSDRV_DIR)/pub/bin/pc
	make -C $(OSDRV_DIR)/tools/pc/squashfs4.3/
	cp $(OSDRV_DIR)/tools/pc/squashfs4.3/mksquashfs $(OSDRV_DIR)/pub/bin/pc
	make -C $(OSDRV_DIR)/tools/pc/lzma_tool/
	cp $(OSDRV_DIR)/tools/pc/lzma_tool/lzma $(OSDRV_DIR)/pub/bin/pc
	make -C $(OSDRV_DIR)/tools/pc/ext4_utils/
	cp $(OSDRV_DIR)/tools/pc/ext4_utils/bin/$(EXT4_TOOL) $(OSDRV_DIR)/pub/bin/pc

hipctools_clean:
	make -C $(OSDRV_DIR)/tools/pc/mkyaffs2image clean
	make -C $(OSDRV_DIR)/tools/pc/jffs2_tool distclean
	make -C $(OSDRV_DIR)/tools/pc/cramfs_tool distclean
	make -C $(OSDRV_DIR)/tools/pc/mkimage_tool distclean
	make -C $(OSDRV_DIR)/tools/pc/squashfs4.3 distclean
	make -C $(OSDRV_DIR)/tools/pc/lzma_tool distclean
	make -C $(OSDRV_DIR)/tools/pc/ext4_utils clean


##########################################################################################
#task [7]	build board tools
##########################################################################################
hiboardtools: hirootfs_prepare hiboardtools_clean
	@echo "---------task [7] build tools which run on board "
	make -C $(OSDRV_DIR)/tools/board/e2fsprogs
	cp -af $(OSDRV_DIR)/tools/board/e2fsprogs/bin/* $(OSDRV_DIR)/pub/$(PUB_ROOTFS)/bin
	make -C $(OSDRV_DIR)/tools/board/reg-tools-1.0.0
	cp -af $(OSDRV_DIR)/tools/board/reg-tools-1.0.0/bin/* $(OSDRV_DIR)/pub/$(PUB_ROOTFS)/bin
	make -C $(OSDRV_DIR)/tools/board/udev-167
	cp -af $(OSDRV_DIR)/tools/board/udev-167/rootfs/. $(OSDRV_DIR)/pub/$(PUB_ROOTFS)/
	make -C $(OSDRV_DIR)/tools/board/gdb
	cp $(OSDRV_DIR)/tools/board/gdb/gdb-$(OSDRV_CROSS) $(OSDRV_DIR)/pub/bin/$(PUB_BOARD)
	make -C $(OSDRV_DIR)/tools/board/mtd-utils/
	cp $(OSDRV_DIR)/tools/board/mtd-utils/bin/* $(OSDRV_DIR)/pub/bin/$(PUB_BOARD)
	make -C $(OSDRV_DIR)/tools/board/ethtools/
	cp $(OSDRV_DIR)/tools/board/ethtools/bin/sbin/* $(OSDRV_DIR)/pub/bin/$(PUB_BOARD)

hiboardtools_clean:
	make -C $(OSDRV_DIR)/tools/board/e2fsprogs distclean
	make -C $(OSDRV_DIR)/tools/board/reg-tools-1.0.0 clean
	make -C $(OSDRV_DIR)/tools/board/udev-167 clean
	make -C $(OSDRV_DIR)/tools/board/gdb distclean
	make -C $(OSDRV_DIR)/tools/board/mtd-utils distclean
	make -C $(OSDRV_DIR)/tools/board/ethtools distclean

##########################################################################################
#task [8]	build rootfs
##########################################################################################
hirootfs_build: hibusybox hiboardtools hirootfs_prepare hipctools hirootfs_notools_build
hirootfs_notools_build:
	@echo "---------task [8] build rootfs"
	chmod 777 $(OSDRV_DIR)/pub/bin/$(PUB_BOARD)/*
	chmod 777 $(OSDRV_DIR)/pub/bin/pc/*
	rm $(OSDRV_DIR)/pub/$(PUB_ROOTFS)/dev/* -rf
	pushd $(OSDRV_DIR)/pub/$(PUB_ROOTFS);ln -s sbin/init init;popd
ifneq ($(FLASH_TYPE),nand)
	pushd $(OSDRV_DIR)/pub/bin/pc;./mkfs.jffs2 -d $(OSDRV_DIR)/pub/$(PUB_ROOTFS) -l -e 0x40000 -o $(OSDRV_DIR)/pub/$(PUB_IMAGE)/$(JFFS2_IMAGE_BIN_256K);popd
	pushd $(OSDRV_DIR)/pub/bin/pc;./mkfs.jffs2 -d $(OSDRV_DIR)/pub/$(PUB_ROOTFS) -l -e 0x20000 -o $(OSDRV_DIR)/pub/$(PUB_IMAGE)/$(JFFS2_IMAGE_BIN_128K);popd
	pushd $(OSDRV_DIR)/pub/bin/pc;./mkfs.jffs2 -d $(OSDRV_DIR)/pub/$(PUB_ROOTFS) -l -e 0x10000 -o $(OSDRV_DIR)/pub/$(PUB_IMAGE)/$(JFFS2_IMAGE_BIN_64K);popd
endif
	pushd $(OSDRV_DIR)/pub/bin/pc;./$(YAFFS_TOOL) $(OSDRV_DIR)/pub/$(PUB_ROOTFS)/ $(OSDRV_DIR)/pub/$(PUB_IMAGE)/$(YAFFS2_IMAGE_BIN_2K_4BIT) 1 2;popd
	pushd $(OSDRV_DIR)/pub/bin/pc;./$(YAFFS_TOOL) $(OSDRV_DIR)/pub/$(PUB_ROOTFS)/ $(OSDRV_DIR)/pub/$(PUB_IMAGE)/$(YAFFS2_IMAGE_BIN_2K_24BIT) 1 4;popd
	chmod 644 $(OSDRV_DIR)/pub/$(PUB_IMAGE)/$(YAFFS2_IMAGE_BIN_2K_4BIT)
	chmod 644 $(OSDRV_DIR)/pub/$(PUB_IMAGE)/$(YAFFS2_IMAGE_BIN_2K_24BIT)
	pushd $(OSDRV_DIR)/pub/bin/pc;./$(EXT4_TOOL) -l 32M -s $(OSDRV_DIR)/pub/$(PUB_IMAGE)/$(EXT4_IMAGE_BIN) $(OSDRV_DIR)/pub/$(PUB_ROOTFS);popd
	find $(OSDRV_DIR)/pub/$(PUB_ROOTFS)/ -name '*svn' | xargs rm -rf
	pushd $(OSDRV_DIR)/pub;tar czf $(PUB_ROOTFS).tgz $(PUB_ROOTFS);rm $(PUB_ROOTFS) -rf;popd
	@echo "---------finish osdrv work"

hirootfs_clean:
	rm $(OSDRV_DIR)/pub/$(PUB_ROOTFS)/ -rf

##########################################################################################
#task [9]	clean pub
##########################################################################################
pub_clean:
	rm $(OSDRV_DIR)/pub/$(PUB_IMAGE)/ -rf
	rm $(OSDRV_DIR)/pub/bin/ -rf
	rm $(OSDRV_DIR)/pub/$(PUB_ROOTFS).tgz -rf
	rm $(OSDRV_DIR)/kernel/$(KERNEL_VER) -rf
