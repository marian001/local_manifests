# Changelog

## 2026-11-16 – lineage-23.2-20261116-UNOFFICIAL-t2s (signed)
First public build. Android 16 (LineageOS 23.2), security patch 2026-11-01, vendor firmware G996BXXSJHZC2. `userdebug`, release-keys.

### Works (tested on our phone)
- Boot, SELinux enforcing, 0 `avc: denied`, no crashes or tombstones
- Wi-Fi, mobile data (LTE) over eSIM
- eSIM: OpenEUICC reads the eUICC, installs and switches profiles; profile enable without a reboot
- Running HALs: health, lineage health, fingerprint, UWB, NFC, secure element, vibrator, Bluetooth, GNSS, camera
- Magisk works (not included)

### Not tested
Voice calls, SMS, VoLTE call (no SIM with a phone number), cameras, fingerprint enrollment, wireless charging, GPS fix, Bluetooth pairing.

### Known issues
- eSIM is switched in **OpenEUICC -> menu -> "Use built-in eSIM as SIM 2" / "Use SIM tray 2 instead of eSIM"**, not in Settings -> SIMs. The app closes after switching; open it again.
- eSIM shares the SIM 2 interface with the physical tray 2 (Samsung tsds2): you get either SIM tray 2 or eSIM, not both.
- VoLTE carrier config is included only for Slovak operators.
- After some reboots the phone does not show up on USB (`setCurrentUsbFunctionsCb failed`); toggling USB debugging off/on helps. Seen 2 of 4 boots, so far only with Magisk installed.
- Fingerprint enrollment and UWB are known weak spots of the upstream t2s tree.
- No support, no monthly updates.
