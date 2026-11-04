# Root modules

Tested on **Infinity X 4.1 (Android 17) + ReSukiSU 4.2.0-rc3**. To install a module, use ReSukiSU → **Modules** → the install button (bottom right) → pick the `.zip`, then reboot. Install modules one at a time if you can, so a bootloop has an obvious cause.

| Module | Version | Status | What it's for |
|---|---|---|---|
| NeoZygisk | v2.4 | ✅ | Zygisk for KernelSU; required by Vector/LSPosed |
| Vector | v2.2 | ✅ | Xposed framework, the maintained continuation of LSPosed |
| systemless hosts KernelSU module | v1.2.2 | ⚠️ | Writable `/system/etc/hosts` for AdAway |

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

## systemless hosts KernelSU module v1.2.2: ⚠️

| | |
|---|---|
| Source | <https://github.com/symbuzzer/systemless-hosts-KernelSU-module/releases/tag/v1.2.2> |
| File | `systemless-hosts-KernelSU-module.zip` |
| SHA-256 | `a5195f6cacc1be26299fb566785f4bd1587e7d1ca10cace8581b6649f81d9b02` (matches GitHub) |
| What it does | Copies `/system/etc/hosts` into the module and mounts it over `/system/etc` at boot, so AdAway's root mode can edit it. It's about 10 lines of script, which I read before installing |
| Status | Installed and rebooted, but `/system/etc` **wasn't overlay-mounted** and the hosts file still had only 2 lines |
| Likely fix | The module only works when the manager uses **OverlayFS**, not magic mount. Turn on OverlayFS in ReSukiSU's settings, reboot, then apply AdAway again. *(Not verified yet.)* |

To check it's working:

```bash
adb shell "grep ' /system/etc ' /proc/mounts; wc -l /system/etc/hosts"
```

When it works, you should see an overlay mount and thousands of lines.
