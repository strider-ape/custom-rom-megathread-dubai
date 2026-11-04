# Troubleshooting

These are real problems I hit on this phone, and what fixed them.

## Recovery bootloops (restarts, turns off, restarts again)

**Cause:** on `dubai`, recovery is built from **boot + vendor_boot + dtbo**. I had flashed the ROM's recovery `vendor_boot` and `dtbo` but the **ReSukiSU root boot.img** instead of the recovery `boot.img`. With mismatched images, recovery couldn't start.

**Fix:** go to the bootloader (hold Power + Volume Down) and flash the matching recovery `boot.img`:

```bash
fastboot flash boot boot.img
fastboot reboot recovery
```

**Lesson:** flash only the three images from the ROM's `recovery/` folder before installing. Root comes after the ROM has booted (see [roms.md](roms.md)).

## The phone won't boot after a flash

The phone has two slots. A bad flash usually breaks only the active one, so switch to the other:

```bash
fastboot getvar current-slot
fastboot set_active a   # or b
fastboot reboot
```

This only helps if the other slot still has a complete, working system.

## A module bootloops the phone

Press **Volume Down several times while the phone boots**. ReSukiSU (and KernelSU in general) then starts in safe mode with every module disabled. Remove the bad module in the Modules tab and reboot.

## `adb devices` or `fastboot devices` shows nothing

1. **Check which tool fits the phone's current mode.** `adb` works in Android and in recovery; `fastboot` works only in the bootloader.
2. **Check whether Windows sees the phone at all** (in Device Manager, or with `Get-PnpDevice -PresentOnly | ? InstanceId -match 'VID_18D1|VID_22B8'`). If nothing Android-related shows up, it's the **cable, port or USB-C socket**, not a driver. For me, reseating the cable or changing ports fixed it every time. The connection also tends to drop during reboots.
3. **For sideloading**, the phone must be on **Apply update → Apply from ADB**, and `adb devices` must show `sideload`. "Enable ADB" in recovery's Advanced menu is a different feature.
4. **After a clean flash**, accept the "Allow USB debugging" prompt on the phone.

## Play Store apps vanish after installing

It happened twice: right after a fresh flash, a Play Store app showed as installed, and later it was gone. I haven't found the cause. Check with `adb shell pm list packages | grep <name>` and reinstall if needed.

## Patched app shows "you need the unpatched APK version"

Morphe needs the **exact** original version it names, not the newer one from the Play Store. Download that version from APKMirror, check its signing certificate (see the hashes in [apps.md](apps.md)), copy it to `Download/`, and choose **"No, I already have it"** in Morphe.
