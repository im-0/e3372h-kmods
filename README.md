[![Build Status](https://api.travis-ci.org/im-0/e3372h-kmods.svg?branch=master)](https://travis-ci.org/im-0/e3372h-kmods)
# Additional kernel modules for Huawei E3372h LTE modem

This repository contains source code and tools required
for building kernel modules for E3372h.

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

1. `mbcache.ko`
2. `ext2.ko`

### `ext3.ko` - EXT3 filesystem support

Dependencies/insmod order:

1. `mbcache.ko`
2. `jbd.ko`
3. `ext3.ko`

### `ext4.ko` - EXT4 filesystem support

Dependencies/insmod order:

1. `mbcache.ko`
2. `jbd2.ko`
3. `crc16.ko`
4. `ext4.ko`

### `tun.ko` - TUN/TAP network device

Dependencies/insmod order:

1. `tun.ko`

### `ip_gre.ko` - GRE tunnels over IP

Dependencies/insmod order:

1. `gre.ko`
2. `ip_gre.ko`

### PPP-related modules

Dependencies/insmod order (some modules are optional):

1. `slhc.ko`
2. `ppp_generic.ko`
3. `bsd_comp.ko`
4. `ppp_deflate.ko`
5. `ppp_mppe.ko`
6. `pppox.ko`
7. `pppopns.ko`
8. `gre.ko`
9. `pptp.ko`

## Network-related modules and "fastip"

As it appears, Balong chipset used in this modem has an hardware NAT
acceleration feature. Kernel code for this feature affects field layout
of `struct sk_buff`, witch is used inside other network-related modules
as well.

In older firmware versions this feature was disabled, but in newer firmwares
someone decided to enable it. And thus, network-related modules compiled for
old firmwares are not compatible with new firmwares.

Run following command on modem immediately after a reboot to check presence
of "fastip":

```
dmesg | grep -i fastip
```

Feature is enabled if you see simething like this:

```
<4>[000005479ms] fastip ver:2013-12-05 10:32 v1.0
<4>[000005479ms] fastip_module_init: fastip_kattach kfastip_handle:c0a4e244
<4>[000005480ms] rmnet2 fastip init
<4>[000005480ms] rmnet2 fastip devname rmnet2
<4>[000005480ms] _fastip_attach: name:rmnet2 fastip handle:c2ed3840
<4>[000005480ms]  rmnet2_fastip_handle attach ok !!!!!! cih = 0xc2ed3840
<4>[000005480ms] rmnet2 fastip register ok
<4>[000007252ms] _fastip_attach: name:usb0 fastip handle:c27ba240
<4>[000013491ms] wan fastip init
<4>[000013491ms] wan fastip devname eth_v
<4>[000013491ms] _fastip_attach: name:eth_v fastip handle:c24ade40
<4>[000013491ms]  rnic_fastip_handle attach ok !!!!!! cih = 0xc24ade40
<4>[000013491ms] wan fastip register ok
```

Modules in this repository are compiled with "fastip" enabled.
