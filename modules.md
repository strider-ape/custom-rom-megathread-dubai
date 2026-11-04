# Root modules

Tested on **Infinity X 4.1 (Android 17) + ReSukiSU 4.2.0-rc3**. To install a module, use ReSukiSU → **Modules** → the install button (bottom right) → pick the `.zip`, then reboot. Install modules one at a time if you can, so a bootloop has an obvious cause.

| Module | Version | Status | What it's for |
|---|---|---|---|
| NeoZygisk | v2.4 | ✅ | Zygisk for KernelSU; required by Vector/LSPosed |
| Vector | v2.2 | ✅ | Xposed framework, the maintained continuation of LSPosed |
| systemless hosts KernelSU module | v1.2.2 + bind-mount fix | ✅ (with fix) | Writable `/system/etc/hosts` for AdAway. **Doesn't work unmodified on dubai**; see below |
| Mountify | v2.0.5 | ✅ installed and active | Metamodule that mounts other modules' `system/` files. Iconify needs it |

## NeoZygisk v2.4: ✅

| | |
|---|---|
| Source | <https://github.com/JingMatrix/NeoZygisk/releases/tag/v2.4> |
| File | `NeoZygisk-v2.4-289-release.zip` |
| SHA-256 | `93a1425c67bb89f58a0dcb9fc823ec93f6472214f221e667a70b67c6a6e061a1` (matches GitHub) |
| Why this one | Its v2.4 release notes explicitly add Android 17 support. It's made by the same developer as Vector, so the two are built to work together |
| How I checked it | After a reboot, the daemons `zygiskd64` and `zygisk-ptrace64` are running (`adb shell ps -A`) |

Alternatives I haven't tested: ZygiskNext (v1.5.x mentions Android 17 QPR2) and ReZygisk (v1.0.0 mentions Android 17 Beta 3). **Only install one Zygisk implementation at a time.**

## Vector v2.2 (LSPosed): ✅

| | |
|---|---|
| Source | <https://github.com/JingMatrix/LSPosed/releases/tag/v2.2> (the repo is still named LSPosed; the project is now called Vector) |
| File | `Vector-v2.2-3080-Release.zip` |
| SHA-256 | `9ee8323575d615f7b3f1076ff60b2a63a49390ef11881b52632311a37f6f79cc` (matches GitHub) |
| Needs | A Zygisk module (NeoZygisk above) |
| How I checked it | `vectord` is running. The manager's log shows `Root implementation: KernelSU (4.2.0-rc3 (uapi: 4)) via ksud` and that it loaded the module repository |

Notes:
- The original `LSPosed/LSPosed` stopped releasing in 2023 (v1.9.2) and doesn't support Android 17. Use Vector.
- By default the manager runs "parasitically" inside the system, so there's **no app icon**. Open it from the Vector notification after booting; the manager can add a home-screen shortcut.
- LSPosed modules are normal APKs: install one, enable it in Vector → Modules, tick the apps it should hook (its "scope"), then force-stop those apps or reboot.
- If Vector stops working after a ROM update, try the latest canary build from the same releases page.
- Vector asks for a **soft reboot** after you enable a framework module. If you dismiss that prompt, there's no soft-reboot button on its home screen, and `adb shell am restart` is refused (the shell lacks the `SHUTDOWN` permission). A normal reboot works.

## Mountify v2.0.5 (metamodule): ✅

Newer KernelSU builds (including ReSukiSU 4.x) no longer mount modules' `system/` folders themselves. A **metamodule** does that job, and Iconify's README says KernelSU 3.0+ users need Mountify.

| | |
|---|---|
| Source | <https://github.com/backslashxx/mountify/releases> (latest: 205 / v2.0.5) |
| File | `mountify.zip` |
| SHA-256 | `2b1850e73bfb0b3dcec8b250272ee9a172b7456f5e2cb53bb5b11d5a0b8848c9` (matches GitHub) |
| What I checked in the zip | The shell scripts make no network calls; the only URLs are in comments. The optional "nuke" kernel modules are off by default (`enable_lkm_nuke=0`) and none are built for kernel 5.4. It skips any module that ships `system/etc/hosts` and respects `skip_mount`, so the hosts module's bind-mount fix is left alone |
| How it handles dubai's f2fs | It copies module files into tmpfs, or an **ext4 image** if tmpfs lacks xattr support, and overlays from there, avoiding the f2fs limitation that breaks the plain hosts module. On dubai it picked the **ext4 fallback** |
| Install log (ksud) | `Installing metamodule: mountify` → `Creating metamodule symlink: /data/adb/metamodule -> /data/adb/modules/mountify` → `installed successfully` |
| After a reboot | ReSukiSU's module card shows **`mode: auto 🤖 \| fstype: ext4 🛠️ \| no modules mounted`** ("no modules mounted" is expected until a module with `system/` files is installed). AdAway, NeoZygisk and Vector all still worked |

Settings for Iconify, changed in ReSukiSU → Modules → mountify → 📄 (web UI):

| Setting | Value | Note |
|---|---|---|
| `mountify_mounts` | `2` (auto) | the default |
| `MOUNT_DEVICE_NAME` | **`KSU`** | default is `overlay`; Iconify asks for `KSU` |
| `mountify_custom_umount` | `0` | the default |

On the module card, **▶** runs its action script, **📄** opens the settings web UI and **🗑** uninstalls it. Settings take effect at the next reboot.

## LSPosed (Xposed) modules

These are normal APKs, enabled in Vector → Modules.

| Module | Version | Status | What it does |
|---|---|---|---|
| Enable Screenshot (DisableFlagSecure) | 5.0.1 | ⚠️ enabled, not yet tested in an app | Allows screenshots and screen recording in apps that block them |
| PixelXpert | 5.1.1 | ❌ skipped (incompatible) | SystemUI tweaks for Pixel ROMs |

### Enable Screenshot (DisableFlagSecure) 5.0.1: ⚠️

| | |
|---|---|
| Source | <https://github.com/LSPosed/DisableFlagSecure/releases/tag/v5.0.1> (official LSPosed organization) |
| File | `app-release.apk`, package `io.github.lsposed.disableflagsecure` |
| SHA-256 | `dcf2006ad130ed58e97cc2bc506e9548044349ebd651c07c7646dd51acb4e852` (matches GitHub) |
| Signing cert | `2651b5caf28b2f62be8cead5df2be82cc90f9c8a954464878bb5880b62db718f` (same key as v5.0.0) |
| API | Modern Xposed API 101, which Vector 2.2 supports (it reports API 102) |
| Scope | System Framework + System UI. The module sets these itself, so there's nothing to tick |
| Status | Vector enabled it automatically after `adb install`. After a reboot the phone boots fine, Vector is hooked into `system_server`, and AdAway still works. **Not yet confirmed by taking a screenshot in an app that blocks it** |

### PixelXpert: ❌ don't install on Android 17

- The repo (<https://github.com/siavash79/PixelXpert>) was **archived in June 2026**.
- The last release (v5.1.1) officially supports only **stock Pixel Android 16 QPR1/QPR2**. It doesn't support Android 17 or AOSP custom ROMs.
- Its install notes warn that on KernelSU you must turn off "unmount modules by default" **"or your device will bootloop"**. It also hooks SystemUI, so a version mismatch risks a SystemUI crash loop.

## systemless hosts KernelSU module v1.2.2: ✅ only with a bind-mount fix

| | |
|---|---|
| Source | <https://github.com/symbuzzer/systemless-hosts-KernelSU-module/releases/tag/v1.2.2> |
| File | `systemless-hosts-KernelSU-module.zip` |
| SHA-256 | `a5195f6cacc1be26299fb566785f4bd1587e7d1ca10cace8581b6649f81d9b02` (matches GitHub) |
| What it does | Copies `/system/etc/hosts` into the module and, at boot, mounts that copy over `/system/etc` with **overlayfs**, so AdAway's root mode can edit it |

### Why it fails on dubai

The kernel rejects the overlay at every boot. `dmesg` shows:

```
overlayfs: filesystem on '/data/adb/modules/systemless-hosts-KernelSU-module/system/etc' not supported
```

`/data` is f2fs, and this 5.4 kernel's overlayfs won't use it as an upper dir. The module installs without errors, but nothing gets mounted. ReSukiSU 4.2.0-rc3 has no OverlayFS or magic-mount setting that changes this.

> ⚠️ **Side effect:** with no writable hosts file, AdAway's root mode **remounts the system partition read-write and writes to it directly**. On dubai the system partition is 100% full, so the write fails partway and leaves `/system/etc/hosts` **empty (0 bytes)**. The file can't be restored, because even 56 bytes won't fit. It's harmless once the fix below is in place (and a ROM reflash rewrites it), but **apply the fix before opening AdAway**.

### The fix: bind-mount the file instead of overlaying the folder

Replace the module's `service.sh` (in `/data/adb/modules/systemless-hosts-KernelSU-module/`) with:

```sh
MODDIR="${0%/*}"
# Bind-mount the hosts file instead of overlayfs: this kernel's overlayfs
# rejects /data (f2fs) as an upper dir, so the original overlay never mounted.
mount -o bind "$MODDIR/system/etc/hosts" /system/etc/hosts
```

Then reboot. You can also run the `mount` line as root to activate it right away. Then, in AdAway, pause and resume blocking so it writes its list. AdAway then writes into the module's copy on `/data` and never touches `/system`.

Verified on 2026-11-04, **including after a reboot**:
- The bind mount is present in the global mount namespace.
- netd (the system DNS resolver) sees all 77,847 lines.
- Ad domains resolve to `127.0.0.1`, while normal sites and `localhost` resolve correctly.

### Checking it

Run these as root (allow Shell in ReSukiSU's Superuser list first):

```bash
adb shell su -c "grep hosts /proc/1/mountinfo"   # should list the bind mount
adb shell ping -c1 ad.doubleclick.net             # should say 127.0.0.1
```

Without root, the adb shell can't see module mounts (it's set to `UMOUNT` by default), so a plain `cat /system/etc/hosts` from adb is **not** a reliable check. The `ping` test is: DNS goes through netd, which does see the mount.
