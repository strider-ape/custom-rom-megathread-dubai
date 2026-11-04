# Root

## ReSukiSU + SuSFS (KernelSU fork): ✅

**ReSukiSU** is a fork of KernelSU: root is built into the kernel instead of being patched into the boot ramdisk like Magisk does. **SuSFS** is built into the same kernel and hides root from apps that check for it.

| | |
|---|---|
| Boot image | `ReSukiSU-Dubai-boot.img` from the Infinity X SourceForge folder `dubai/17/recovery/ReSukiSU-Boot/`, MD5 `6fc508cced13401d881061cd1a2e6f8e` |
| Works with | **Only** the Infinity X build it was made for (4.1, 2026-10-18). Each ROM update needs a matching new image |
| Manager app | `ReSukiSU_v4.2.0-rc3_35171-arm64-v8a-release.apk` from <https://github.com/Baka-SU/BakaSU/releases/tag/v4.2.0-rc3> (a pre-release) |
| Manager SHA-256 | `25657bc449439687608fffa04b4b586de90fc405e3dc6217bd997fc71ba0a0a1` (matches GitHub's published digest) |
| Manager package | `com.resukisu.resukisu` |

### Installing

Only do this after the ROM is installed and has booted once.

```bash
adb reboot bootloader
fastboot flash boot ReSukiSU-Dubai-boot.img   # goes to the active slot
fastboot reboot
```

Then install the manager APK:

```bash
adb install ReSukiSU_v4.2.0-rc3_35171-arm64-v8a-release.apk
```

### Checking that root is actually active

Look at the kernel version string. The root kernel and the ROM's plain kernel have different build hashes:

```bash
adb shell uname -r
```

| Output | Meaning |
|---|---|
| `5.4.302-moto-ga116ec437f85` | ReSukiSU kernel (rooted) |
| `5.4.302-moto-gbee2048c3b90` | Infinity X's plain recovery `boot.img` (not rooted) |

The manager's **Home** tab should say it's working, and its **Superuser** tab lists apps with per-app root settings. That tab only works when root is active.

`adb shell su -c id` returns `su: not found` by default. That's normal: KernelSU only provides `su` to apps you allow, and the adb shell isn't allowed until you enable **Shell** in the manager's Superuser list.

### Removing root

Flash the ROM's plain `boot.img` back:

```bash
fastboot flash boot boot.img
```

### Safety notes

- Grant root only to apps you trust; an app with root can read everything on the phone.
- If a module bootloops the phone, press **Volume Down several times while it boots** to start in safe mode with all modules disabled.
- Banking apps and Google Wallet tap-to-pay may still detect root. SuSFS plus the ROM's Play Integrity guide improve the odds, but don't guarantee it.
