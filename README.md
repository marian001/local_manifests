# LineageOS 23.2 for Samsung Galaxy S21+ (SM-G996B, t2s) – unofficial

Local manifests and release notes for an **unofficial** LineageOS 23.2 (Android 16) build for the Exynos Galaxy S21+ (`t2s`, SM-G996B).

> **No support.** One-time community release. No monthly security updates, no maintainer. Use at your own risk.

## What is different from upstream
- `device/samsung/universal2100-common`, `vendor/samsung/universal2100-common`, `device/samsung/t2s`: forks of the Pandaispro trees (branch `esim`) with:
  - build fix: `TARGET_2ND_ARCH_VARIANT := armv8-a` (current Soong rejects the old variant)
  - 32-bit `libteecl.so` added to the vendor tree and `proprietary-files.txt` (byte-identical to stock G996BXXSJHZC2)
  - eSIM: RIL shim `libsec-ril` -> `libsec-ril-impl`, `libsec-ril-slotswitch` interposer (port of exy2100 PRs #5, #9 and vendor PR #1 by Macktherealonereally), non-removable eUICC overlay
  - RIL shim retries `GET_IMSI` after an eSIM profile is enabled (mobile data comes up without a reboot)
  - OpenEUICC as privileged LPA + `android.hardware.telephony.euicc` (port of exy2100 o1s PR #3)
  - open-source IMS stack (VoLTE) with a CarrierConfig RRO for Slovak operators
  - sensors: the multihal loads `sensors.sensorhub` through `sensors.sensorhub_wait_for_mcu`, so it waits for the sensor hub MCU (light, proximity, accelerometer, gyroscope and compass were missing; adaptive brightness did not work)
  - fingerprint: SELinux policy for the Samsung sysinput HAL (`sec-sysinput-aidl` could not start) and `ro.vendor.fingerprint.request_touch_event=false` (enrollment failed at the first touch)
  - always-on display is off by default (it stays available in Settings): with it on, the display driver holds a wakeup source and the phone never suspends
  - sysinput HAL: three more SELinux rules (`input_device` write, `/mnt/vendor` and `/efs` search)
- `hardware/samsung`: fork of exy2100 (branch `esim`) with one commit: SamsungBiometrics uses the framework default for "fingerprint unlock with the screen off" (it treated a never-touched setting as off, so the fingerprint did nothing on the always-on display / with the screen off after a clean install)
- `frameworks/base` pinned to LineageOS `1ef04f0354` (upstream fix: the expanded volume panel did not open)
- `device/lineage/sepolicy` from exy2100 (labels `IWirelessFastCharge`; without it the build bootloops)
- `packages/apps/OpenEUICC`: fork with BPP 255-byte segments (#359 port) and a menu item to switch SIM 2 between the tray and the built-in eSIM
- `packages/modules/ImsMedia`, `packages/services/Telephony`: VoLTE uplink audio fixes (krazey/ImsMedia + Mack's PR, LineageOS Gerrit 505200)
- `vendor/apn`: IMS/XCAP APNs for Slovak operators

All manifests are pinned to exact commits. `pinned.xml` (in the GitHub release) is the full `repo manifest -r` of the tree the release was built from.

## Build
```bash
mkdir lineage && cd lineage
repo init -u https://github.com/LineageOS/android.git -b lineage-23.2 --git-lfs
git clone https://github.com/marian001/local_manifests .repo/local_manifests
repo sync -c -j8 --no-clone-bundle --no-tags
(cd packages/apps/OpenEUICC && git submodule update --init --recursive)
source build/envsetup.sh
lunch lineage_t2s-bp4a-userdebug
mka bacon
```
- `--git-lfs` matters: without it `external/chromium-webview` prebuilts stay LFS pointers and the build fails late (`aapt2 ... Invalid file`).
- Soong analysis needs about 40 GB RAM; the R8/dex phase needs more RAM or lower `-j`.
- To reproduce the release tree exactly, use `pinned.xml` from the release instead of the files here.
- The proprietary blobs come from the vendor forks (extracted from stock firmware). Sign your own builds with your own keys (LineageOS wiki: "Signing builds").

## Install (clean)
Tested on stock base firmware **G996BXXSJHZC2** (One UI 7 / Android 15).
1. Unlock the bootloader (wipes the phone and trips Knox permanently).
2. Download mode: flash `lineage_recovery-signed.tar` (contains `recovery.img` + `vbmeta.img`) with Odin/odin4 in the AP slot.
3. Boot to recovery: Factory reset -> **Format data**.
4. Apply update -> Apply from ADB: `adb sideload lineage-23.2-20261125-UNOFFICIAL-t2s-signed.zip`.
5. Reboot.

Coming from any other build (including test-keys builds) requires **Format data**: the signing keys differ.

Updating from our `20261116` or `20261118` signed build: reboot to recovery, Apply update -> Apply from ADB, sideload the new zip. No wipe needed (same keys). Coming from `20261116`, enrolled fingerprints are gone after the update; add them again.

## Signing keys
- Build type `userdebug`, `release-keys` (adb root is off by default).
- OTA / release key SHA-256: `91:C8:F0:74:21:41:A7:58:E9:9E:C3:67:3D:1C:9B:63:D2:1D:CE:83:AD:F6:36:DB:B5:93:F2:D1:E1:25:54:3C`
- AVB key: RSA-4096, public key SHA-1 `bf698d5ca8ec8e524504e61e7290a3507a409323` (`avb_pkmd.bin` in the release). The bootloader stays unlocked (orange state).

## Credits
LineageOS · Pandaispro (t2s device, vendor and kernel trees) · exy2100 (hardware, sepolicy, interfaces) · Macktherealonereally (eSIM/VoLTE fixes for o1s) · krazey (ImsStack, ImsMedia, CarrierSettings) · PeterCxy / estkme-group (OpenEUICC).

Kernel source (GPL-2.0): `Pandaispro/android_kernel_samsung_universal2100`, commit `dce077ec2b5a9990a3de6016a77a25637aab9f2c`.
