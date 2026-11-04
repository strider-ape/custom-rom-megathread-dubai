# Custom ROM megathread: Motorola Edge 30 (`dubai`)

These are my own tested notes on custom ROMs, root, modules and apps for the **Motorola Edge 30 (codename `dubai`)**. Every entry says whether I verified it on my own phone. When an item is downloaded, I check it against its official source (checksums and signing certificates) and record the hashes here.

> Flashing can brick your phone or wipe your data. These notes describe what worked for me, on my unit and firmware; they're not a guarantee. Read the official ROM thread first.

## Status legend

| Mark | Meaning |
|---|---|
| ✅ | Verified working on my phone (checked over adb, not just "it installed") |
| ⚠️ | Installed, but not working yet or only partly verified |
| ❌ | Tried and didn't work |

## Device facts

| | |
|---|---|
| Model | Motorola Edge 30, codename `dubai` |
| SoC | Snapdragon 778G+ |
| Partition scheme | **A/B slots** (`boot_a` / `boot_b` etc.) with a dynamic `super` partition |
| Recovery | No separate recovery partition. Recovery is built from **boot + vendor_boot + dtbo** together, so mismatched images can make recovery bootloop |
| Kernel | 5.4 |

## My current setup (as of 2026-11-04)

| Layer | What | Status |
|---|---|---|
| ROM | Project Infinity X 4.1 OFFICIAL (Android 17), build 2026-10-18, GApps | ✅ |
| Root | ReSukiSU (KernelSU fork) + SuSFS, using the ROM maintainer's boot.img | ✅ |
| Zygisk | NeoZygisk v2.4 | ✅ |
| Xposed | Vector v2.2 (maintained continuation of LSPosed) | ✅ |
| Ad blocking | AdAway 6.1.4 + systemless hosts KernelSU module | ⚠️ |
| Patched apps | Morphe Manager 2.3.0, YouTube 21.30.216, Reddit 2026.40.0 | ✅ |

## Contents

- [roms.md](roms.md): tested ROMs and the exact install procedure
- [root.md](root.md): root solutions, how to check root is active, how to remove it
- [modules.md](modules.md): root modules with compatibility status
- [apps.md](apps.md): root apps and Morphe-patched apps
- [troubleshooting.md](troubleshooting.md): bootloops, recovery loops, USB problems, and how I got out of them
