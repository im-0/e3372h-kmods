[![Build Status](https://api.travis-ci.org/im-0/e3372h-kmods.svg?branch=master)](https://travis-ci.org/im-0/e3372h-kmods)
# Additional kernel modules for Huawei E3372h LTE modem

This repository contains binaries as well as source code and tools required
for building.

## Supported firmware versions

Only HiLink firmware is supported.

Kernel modules should be built with the same configuration and toolchain that
was used to build kernel running on the device.

Currently we use following versions:

* Kernel configuration (`/proc/config.gz`) from firmware
version 22.328.62.00.143. It is a modded version downloaded from
[4pda.ru](https://4pda.ru/forum/index.php?showtopic=582284&st=20#entry39517088)
(in Russian), file: `E3372h-153_Update_22.328.62.00.143_M_AT_05.10.rar`.
* Toolchain from Android NDK r8b for x86. Version string matches with
dmesg: `gcc version 4.6.x-google 20120106 (prerelease) (GCC)`.

Also, there are "product configuration" files inside the vendor's sources.
Seems that these are also varies between firmware revisions, and we only have
outdated version included in the initial vendor source code distribution.
Product configuration was changed at least once, resulting in `tun.ko` breakage
(see `patches/0003-Enable-MBB_FEATURE_FASTIP.patch`).

## Kernel modules

### `ext2.ko` - EXT2 filesystem support

Dependencies/insmod order:

1. [mbcache.ko](binary/fs/mbcache.ko)
2. [ext2.ko](binary/fs/ext2/ext2.ko)

### `ext3.ko` - EXT3 filesystem support

Dependencies/insmod order:

1. [mbcache.ko](binary/fs/mbcache.ko)
2. [jbd.ko](binary/fs/jbd/jbd.ko)
3. [ext3.ko](binary/fs/ext3/ext3.ko)

### `ext4.ko` - EXT4 filesystem support

Dependencies/insmod order:

1. [mbcache.ko](binary/fs/mbcache.ko)
2. [jbd2.ko](binary/fs/jbd2/jbd2.ko)
3. [crc16.ko](binary/lib/crc16.ko)
4. [ext4.ko](binary/fs/ext4/ext4.ko)

### `tun.ko` - TUN/TAP network device

Dependencies/insmod order:

1. [tun.ko](binary/drivers/net/tun.ko)

### `ip_gre.ko` - GRE tunnels over IP

Dependencies/insmod order:

1. [gre.ko](binary/net/ipv4/gre.ko)
2. [ip_gre.ko](binary/net/ipv4/ip_gre.ko)
