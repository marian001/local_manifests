# Changelog

## 2026-11-25 – lineage-23.2-20261125-UNOFFICIAL-t2s (signed)
Bug-fix build, same keys as 20261116 / 20261118 (can be sideloaded over them without a wipe).

**Not tested on a phone yet.** The causes below were found and confirmed on our phone running 20261118; the fixes were checked only in the built and signed package. Reports are welcome.

### Fixed
- **Fingerprint on the always-on display / with the screen off after a clean install:** SamsungBiometrics treated a never-touched "fingerprint unlock with the screen off" setting as off, while the system and Settings treat it as on. The touch panel was told to stop watching the sensor area at screen off, so nothing happened. On 20261118 it works once the setting is set explicitly (`adb shell settings put secure screen_off_udfps_enabled 1`); this build uses the same default as the system.
- **Volume panel did not expand** (three dots in the volume slider): upstream LineageOS fix, `frameworks/base` moved to `1ef04f0354`.
- **Phone never sleeps with always-on display on:** in the AOD power state the display driver holds a kernel wakeup source (`19f00000.decon_0`) all the time. Measured on 20261118: held 97 % of 6.5 days of uptime. The driver is not changed; always-on display is now **off by default**. You can still turn it on in Settings, at the cost of battery.
- **sysinput HAL:** added the three missing SELinux rules (`input_device` write, `/mnt/vendor` and `/efs` search).

### Notes
- Updating: if you never touched the always-on display switch, it will be off after the update. If you set it yourself, your choice is kept.
- Battery: idle drain is still higher than it should be even with always-on display off (our phone: about 3.9 %/h with Wi-Fi + LTE, about 2.0 %/h in airplane mode, awake about 70 % of the time). Seen in the logs: the NFC chip wakes the phone every 11-14 s even with nothing near it, GNSS and failed suspend attempts. Cause not found yet. Turning NFC off when you do not need it may help (not measured).
- A tap on the always-on display does not wake the screen (not investigated).
- `boot.img`, `vendor_boot.img` and `dtbo.img` are byte-identical to 20261118 (a Magisk-patched boot can be reused). `recovery.img` and `vbmeta.img` are rebuilt.
- Everything else as in 20261118 below.

## 2026-11-18 – lineage-23.2-20261118-UNOFFICIAL-t2s (signed)
Bug-fix build, same keys as 20261116 (can be sideloaded over it without a wipe).

### Fixed
- **Sensors / adaptive brightness:** the sensors HAL started about 3 s before the sensor hub MCU was ready, so only 25 of 39 sensors were registered (no light, proximity, accelerometer, gyroscope, compass). The HAL now waits for the MCU. Tested: 39 sensors after a cold boot, adaptive brightness reads the light sensor.
- **Fingerprint:** the Samsung sysinput HAL had no SELinux policy and never started, and `request_touch_event` made the ultrasonic sensor abort enrollment at the first touch. Tested: enrollment completes, unlock works with the screen on and off.

### Notes
- Fingerprints enrolled before the update are gone; enroll again.
- Tested as an update over 20261116 (no wipe), SELinux enforcing, with Magisk. A clean install of this build was not tested.
- Proximity during a call and screen rotation were not tested by hand (the sensors are listed).
- Three SELinux denials of the sysinput HAL remain in the log (`input_device` write, `/mnt/vendor` and `/efs` search). Fingerprint works with them.
- `boot.img`, `vendor_boot.img` and `dtbo.img` are byte-identical to 20261116 (a Magisk-patched boot from 20261116 can be reused). `recovery.img` and `vbmeta.img` are rebuilt.
- Everything else as in 20261116 below.

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
