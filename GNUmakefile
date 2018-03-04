ROOT_DIR := $(CURDIR)

CROSS_TOOLCHAIN ?= $(ROOT_DIR)/arm-linux-androideabi-4.6-from-ndk-r8b-x86/bin/arm-linux-androideabi-
BUILD_DIR ?= $(ROOT_DIR)/build-dir

SOC ?= hi6921_v711
SOC_MODE ?= hilink

VENDOR_PRODUCT_TOPDIR ?= $(ROOT_DIR)/e3372h-vendor-src/vender
VENDOR_PRODUCT_NAME ?= $(SOC)_$(SOC_MODE)
VENDOR_PRODUCT_CONF_DIR ?= $(VENDOR_PRODUCT_TOPDIR)/config/product/$(VENDOR_PRODUCT_NAME)

# Kernel config from vendor sources differs from config from /proc/config.gz
# on device.
#KERNEL_CONFIG ?= $(VENDOR_PRODUCT_CONF_DIR)/os/acore/balongv7r2_defconfig
KERNEL_CONFIG ?= $(ROOT_DIR)/kernel-config

MODULES_CONFIG ?= $(ROOT_DIR)/modules-config
MODULES ?= \
	drivers/net/tun.ko \
	net/ipv4/gre.ko \
	net/ipv4/ip_gre.ko \
	fs/mbcache.ko \
	fs/ext2/ext2.ko \
	fs/jbd2/jbd2.ko \
	fs/ext3/ext3.ko \
	lib/crc16.ko \
	fs/ext4/ext4.ko

MODULES_FULL_PATH = $(addprefix $(BUILD_DIR)/kernel-build/,$(MODULES))

VENDOR_COPY_DIR = $(BUILD_DIR)/vendor-src
VENDOR_COPY_KERNEL_DIR = $(VENDOR_COPY_DIR)/modem/system/android/android_4.2_r1/kernel

.PHONY: help
help:
	@echo "Usage:"
	@echo "    make help   # show this help message"
	@echo "    make build  # build modules"
	@echo "    make clean  # cleanup"

.PHONY: build
build: $(BUILD_DIR)/kernel-build/.build-done

.PHONY: clean
clean:
	rm -fr "$(BUILD_DIR)"

.PHONY: copy-binaries
copy-binaries: build
	for module in $(MODULES); do \
		dir_name=$$( dirname "$${module}" ); \
		mkdir -p "$(ROOT_DIR)/binary/$${dir_name}"; \
		cp -fv "$(BUILD_DIR)/kernel-build/$${module}" \
			"$(ROOT_DIR)/binary/$${dir_name}/"; \
	done

$(BUILD_DIR)/.mkdir-done:
	[ -e "$(BUILD_DIR)" ] && \
		rm -fr "$(BUILD_DIR)" || \
		true
	mkdir "$(BUILD_DIR)"
	touch "$(@)"

$(VENDOR_COPY_DIR)/.copy-done: $(BUILD_DIR)/.mkdir-done $(ROOT_DIR)/patches/*.patch
	[ -e "$(VENDOR_COPY_DIR)" ] && \
		rm -fr "$(VENDOR_COPY_DIR)" || \
		true
	cp -r "$(VENDOR_PRODUCT_TOPDIR)" "$(VENDOR_COPY_DIR)"

	cd "$(VENDOR_COPY_DIR)"; \
	ls -1 $(ROOT_DIR)/patches/*.patch | sort -n | while read patch_file; do \
		printf "Applying patch: %s\n" $$( basename "$${patch_file}" ); \
		patch -p2 <"$${patch_file}"; \
	done

	for gcc_ver in 5 6 7; do \
		cp -v "$(VENDOR_COPY_KERNEL_DIR)/include/linux/compiler-gcc4.h" \
			"$(VENDOR_COPY_KERNEL_DIR)/include/linux/compiler-gcc$${gcc_ver}.h"; \
	done
	touch "$(@)"

COMMON_MAKE_OPTS := KERNELRELEASE="3.4.5" \
	O="$(BUILD_DIR)/kernel-build" \
	ARCH="arm" \
	CROSS_COMPILE="$(CROSS_TOOLCHAIN)" \
	EXTRA_CFLAGS="-fno-pic" \
	BALONG_TOPDIR=$(VENDOR_PRODUCT_TOPDIR) \
	OBB_PRODUCT_NAME=$(VENDOR_PRODUCT_NAME) \
	CFG_PLATFORM=$(SOC)

$(BUILD_DIR)/kernel-build/.build-done: $(VENDOR_COPY_DIR)/.copy-done $(KERNEL_CONFIG) $(MODULES_CONFIG) GNUmakefile
	[ -e "$(BUILD_DIR)/kernel-build" ] && \
		rm -fr "$(BUILD_DIR)/kernel-build" || \
		true
	mkdir "$(BUILD_DIR)/kernel-build"

	cp "$(KERNEL_CONFIG)" "$(BUILD_DIR)/kernel-build/.config"
	cat "$(MODULES_CONFIG)" >>"$(BUILD_DIR)/kernel-build/.config"

	cd "$(VENDOR_COPY_KERNEL_DIR)" && yes "n" | $(MAKE) -j1 \
		$(COMMON_MAKE_OPTS) \
		oldconfig

	cd "$(VENDOR_COPY_KERNEL_DIR)" && $(MAKE) \
		$(COMMON_MAKE_OPTS) \
		prepare headers_install scripts

	cd "$(VENDOR_COPY_KERNEL_DIR)" && $(MAKE) \
		$(COMMON_MAKE_OPTS) \
		$(MODULES)

	$(CROSS_TOOLCHAIN)strip --strip-debug $(MODULES_FULL_PATH)

	ls -lh $(MODULES_FULL_PATH)

	touch "$(@)"
