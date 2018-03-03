ROOT_DIR := $(CURDIR)

V ?= 0

CROSS_TOOLCHAIN ?= arm-linux-gnu-
BUILD_DIR ?= $(ROOT_DIR)/build-dir

SOC ?= hi6921_v711
SOC_MODE ?= hilink

VENDOR_KERNEL_DIR ?= $(ROOT_DIR)/e3372h-vendor-src/vender/modem/system/android/android_4.2_r1/kernel
VENDOR_PLATFORM_DIR ?= $(ROOT_DIR)/e3372h-vendor-src/vender/platform/$(SOC)
VENDOR_PRODUCT_DIR ?= $(ROOT_DIR)/e3372h-vendor-src/vender/config/product/$(SOC)_$(SOC_MODE)

# Kernel config from vendor sources differs from config from /proc/config.gz
# on device.
#KERNEL_CONFIG ?= $(VENDOR_PRODUCT_DIR)/os/acore/balongv7r2_defconfig
KERNEL_CONFIG ?= $(ROOT_DIR)/kernel-config

MODULES_CONFIG ?= $(ROOT_DIR)/modules-config
MODULES ?= \
	drivers/net/tun.ko \
	net/ipv4/gre.ko \
	net/ipv4/ip_gre.ko

MODULES_FULL_PATH = $(addprefix $(BUILD_DIR)/kernel-build/,$(MODULES))

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

$(BUILD_DIR)/.mkdir-done:
	[ -e "$(BUILD_DIR)" ] && \
		rm -fr "$(BUILD_DIR)" || \
		true
	mkdir "$(BUILD_DIR)"
	touch "$(@)"

$(BUILD_DIR)/kernel-src/.copy-done: $(BUILD_DIR)/.mkdir-done
	[ -e "$(BUILD_DIR)/kernel-src" ] && \
		rm -fr "$(BUILD_DIR)/kernel-src" || \
		true
	cp -r "$(VENDOR_KERNEL_DIR)" "$(BUILD_DIR)/kernel-src"
	cd "$(BUILD_DIR)/kernel-src" && patch -p1 <"$(ROOT_DIR)/kernel.patch"
	for gcc_ver in 5 6 7; do \
		cp -v "$(BUILD_DIR)/kernel-src/include/linux/compiler-gcc4.h" \
			"$(BUILD_DIR)/kernel-src/include/linux/compiler-gcc$${gcc_ver}.h"; \
	done
	touch "$(@)"

COMMON_MAKE_OPTS := KERNELRELEASE="3.4.5" \
	ARCH="arm" \
	CROSS_COMPILE="$(CROSS_TOOLCHAIN)" \
	EXTRA_CFLAGS="-I$(VENDOR_PLATFORM_DIR)/soc -I$(VENDOR_PRODUCT_DIR)/config"

$(BUILD_DIR)/kernel-build/.build-done: $(BUILD_DIR)/kernel-src/.copy-done $(KERNEL_CONFIG) $(MODULES_CONFIG)
	[ -e "$(BUILD_DIR)/kernel-build" ] && \
		rm -fr "$(BUILD_DIR)/kernel-build" || \
		true
	mkdir "$(BUILD_DIR)/kernel-build"

	cp "$(KERNEL_CONFIG)" "$(BUILD_DIR)/kernel-build/.config"
	cat "$(MODULES_CONFIG)" >>"$(BUILD_DIR)/kernel-build/.config"

	cd "$(BUILD_DIR)/kernel-src" && make \
		V=$(V) \
		$(COMMON_MAKE_OPTS) \
		O="$(BUILD_DIR)/kernel-build" \
		silentoldconfig prepare headers_install scripts

	cd "$(BUILD_DIR)/kernel-src" && make \
		V=$(V) \
		$(COMMON_MAKE_OPTS) \
		O="$(BUILD_DIR)/kernel-build" \
		$(MODULES)

	$(CROSS_TOOLCHAIN)strip --strip-debug $(MODULES_FULL_PATH)

	ls -lh $(MODULES_FULL_PATH)

	touch "$(@)"
