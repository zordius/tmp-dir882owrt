# tmp-dir882owrt

Throwaway repo: build OpenWrt 24.10 for **D-Link DIR-882 A1** with the
`uimage-lzma-loader` (self-extracting kernel) so it boots under an old
**Breed r1266** bootloader that can't decompress the stock LZMA kernel.

## Why
- DIR-882 A1 = MT7621 + 2x MT7615N. Bootloader was replaced by Breed r1266 (2018).
- Stock OpenWrt images for dir-882-a1 do **not** include the lzma-loader, so Breed's
  old LZMA decompressor fails → boot loops back to Breed.
- Adding `$(Device/uimage-lzma-loader)` makes the image self-extract → boots on any loader.

## How to run
1. Actions tab → **build-dir882-lzma** → **Run workflow** (or it runs on push).
2. Wait ~40–90 min.
3. Download artifact **dir882-lzma-images**, which contains:
   - `...-dir-882-a1-initramfs-kernel.bin`  ← RAM test image (does not touch flash)
   - `...-dir-882-a1-squashfs-sysupgrade.bin` ← permanent install

## Test order (safe)
1. Boot the **initramfs** image first (RAM only, Padavan untouched). Confirm it boots.
2. Only then flash the **sysupgrade** to install permanently.
