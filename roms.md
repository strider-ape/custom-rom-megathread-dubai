# ROMs

## Project Infinity X 4.1 OFFICIAL (Android 17): ✅

| | |
|---|---|
| Version | 4.1 OFFICIAL, Android 17 QPR0 |
| Build | `Project_Infinity-X-4.1-dubai-18.10.2026-GAPPS-OFFICIAL.zip` (about 2.6 GB; Vanilla builds are also available) |
| Downloads | ROM: <https://projectinfinity-x.com/downloads/dubai> · Recovery: <https://sourceforge.net/projects/infinity-x/files/dubai/17/recovery/> |
| Tested | 2026-11-03, clean install with data formatted |
| Result | Boots and runs fine; root, Zygisk and Vector all work on top of it |

### Recovery files and official MD5s

These come from the SourceForge `dubai/17/recovery/` folder. The MD5s are SourceForge's own (from the folder's RSS feed), and my downloads matched them.

| File | Size (bytes) | MD5 |
|---|---|---|
| `boot.img` | 100663296 | `15894020f532d1a455b53055a0ecee1d` |
| `vendor_boot.img` | 100663296 | `9150baecf607463d3f10e882ab43afd1` |
| `dtbo.img` | 25165824 | `9e5e3da00142ee19b129f1eaf90f3776` |
| `ReSukiSU-Boot/ReSukiSU-Dubai-boot.img` | 100663296 | `6fc508cced13401d881061cd1a2e6f8e` |

> **Important:** `ReSukiSU-Dubai-boot.img` is **not** part of the recovery set. It's the rooted kernel, flashed **after** the ROM is installed and has booted once. Flashing it as the recovery boot image made recovery bootloop for me (see [troubleshooting.md](troubleshooting.md)).

### Install procedure that worked (clean flash)

Prerequisites: unlocked bootloader, and Android Platform-Tools (`adb`/`fastboot`) on the PC.

```bash
# 1. Reboot to the bootloader (or hold Power + Volume Down)
adb reboot bootloader

# 2. Flash the recovery set; these go to the active slot
fastboot flash dtbo dtbo.img
fastboot flash vendor_boot vendor_boot.img
fastboot flash boot boot.img

# 3. Boot into recovery
fastboot reboot recovery
```

4. In recovery, go to **Factory reset → Format data / factory reset**. This is required when coming from another ROM, and it wipes internal storage.
5. Go to **Apply update → Apply from ADB**, then on the PC run:

```bash
adb sideload Project_Infinity-X-4.1-dubai-18.10.2026-GAPPS-OFFICIAL.zip
```

   The PC side stopping at about 47% with `Total xfer: 1.00x` is normal. Trust what the phone screen says.

6. Reboot to system and let it boot fully once.
7. Optional, for root: follow [root.md](root.md).

### Things I noticed

- **The sideload installs to the *other* slot and switches to it.** I flashed the recovery set to slot B, and after the sideload the phone was running slot A. Root then gets flashed to the new active slot.
- **The ROM spoofs a stock fingerprint** (`motorola/dubai_g/dubai:14/...`), so adb shows `product:dubai_g` even on the custom ROM. That doesn't tell you whether a phone is on stock.
- **For Play Integrity**, the maintainer links a guide in the release post (Telegram).
- **There's a built-in charge limit**, LineageOS's *Charging control* (Settings → Battery → Charging control), so you don't need a root module like ACC. I set it to 80% and checked over adb: on the charger at 100%, `dumpsys battery` switched from `status: 2` (charging) to `status: 3` (discharging) while still `AC powered: true`, and LineageSettings shows `charging_control_enabled=1`, `charging_control_mode=3`. Worth turning on if the phone stays plugged in a lot.
- **Checking battery health without root:** `adb shell dumpsys battery` shows `Maximum capacity` (what the fuel gauge measures now) and `Design capacity` (4,089 mAh on dubai). Divide the first by the second. The cycle count is in `/sys/class/power_supply/battery/cycle_count`, which needs root.
