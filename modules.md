# Root modules

Tested on **Infinity X 4.1 (Android 17) + ReSukiSU 4.2.0-rc3**. To install a module, use ReSukiSU → **Modules** → the install button (bottom right) → pick the `.zip`, then reboot. Install modules one at a time if you can, so a bootloop has an obvious cause.

| Module | Version | Status | What it's for |
|---|---|---|---|
| NeoZygisk | v2.4 | ✅ | Zygisk for KernelSU; required by Vector/LSPosed |
| Vector | v2.2 | ✅ | Xposed framework, the maintained continuation of LSPosed |
| systemless hosts KernelSU module | v1.2.2 + bind-mount fix | ✅ (with fix) | Writable `/system/etc/hosts` for AdAway. **Doesn't work unmodified on dubai**; see below |

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
