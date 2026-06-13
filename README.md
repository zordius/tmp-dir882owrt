# tmp-dir882owrt

GitHub Actions build of **OpenWrt v24.10.7** for the **D-Link DIR-882 A1**
(MT7621 + 2× MT7615N, 16 MB flash), patched to boot under a third-party
**Breed r1266** bootloader and to fix squashfs persistence.

## What this build does

**4 patches** (`.github/workflows/build.yml`):

1. **lzma-loader** (`$(Device/uimage-lzma-loader)`) — makes the kernel
   self-extracting (uImage `comp=none`) so the old Breed, which can't decompress
   newer LZMA kernels, can boot it.
2. **Flash layout → firmware @ `0x50000`** (`mt7621_dlink_flash-16m-a1.dtsi`):
   Breed reads/writes the image at `0x50000` (not the stock `0x60000`); `factory`
   shrunk to 64 KB (cal data fits below `0x10000`).
3. **Remove `openwrt,padding = <96>`** — the squashfs-bootloop fix. The A1
   normally carries a 96-byte D-Link SGE header (`uimage-sgehdr`) and the DTS
   tells `mtdsplit` to skip those bytes. The lzma-loader replaces the kernel
   wrapping, so the uImage sits at offset 0 — but the leftover padding made
   `mtdsplit` look 96 bytes too far, never found the squashfs → no rootfs →
   boot loop. (initramfs boots regardless: it needs no rootfs split.) Removing
   the padding makes `mtdsplit` read the uImage at offset 0 and find the
   squashfs → boots. This matches the upstream **R1** variant's layout.
4. **Baked LAN IP** `192.168.199.1` (`files/etc/uci-defaults/`).

**Trims** (leaner squashfs, ~7.7 MB): no `luci-ssl` (LuCI over plain HTTP),
no `ppp`/`ppp-mod-pppoe` (WAN = DHCP), no USB storage, no IPv6 service.

Pinned to tag **`v24.10.7`** → reproducible source + a stable download-cache key.

## Build

Push to `main`, or Actions → **build-dir882-lzma** → *Run workflow*. ~50 min.
Download artifact **`dir882-padfix-0x50000`**:

- `…-dir-882-a1-initramfs-kernel.bin` — RAM test image (does not touch flash)
- `…-dir-882-a1-squashfs-sysupgrade.bin` — permanent install

## Flash (Breed web recovery, http://192.168.1.1)

1. *(optional)* boot the **initramfs** first (RAM only) to sanity-check.
2. Flash the **squashfs-sysupgrade.bin**. Boots persistently; LAN = `192.168.199.1`.
3. Verify: `cat /proc/mtd` shows `firmware` split into `kernel` / `rootfs` /
   `rootfs_data` (= `mtdsplit` found the squashfs). With country **JP** the 5 GHz
   radio reaches **DFS ch100–140** (the reason for leaving Padavan).

## Post-flash device config — NOT in this repo

Per-device customization (static DHCP leases, Wi-Fi SSIDs + keys, SSH authorized
key, the TW SSID + its `192.168.200.0/24` subnet, firewall port-forwards to the
Pi) is applied by a separate **`adopt-config.sh`** kept **outside this repo**
(personal data, backed up locally). Run it once after flashing, then set the
root password (`passwd`).
