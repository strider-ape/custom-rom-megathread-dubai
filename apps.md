# Apps

## Root apps

| App | Version | Status | Notes |
|---|---|---|---|
| AdAway | 6.1.4 | ✅ | Root mode, with the hosts module **plus its bind-mount fix** (see [modules.md](modules.md)) |
| Iconify | 8.0.0 | ✅ installed and active (customizations not yet tested) | SystemUI customization: volume panel, QS panel, clocks, lock screen, icons |

### AdAway 6.1.4: ✅

| | |
|---|---|
| Source | <https://github.com/AdAway/AdAway/releases/tag/v6.1.4> (official, developer-signed) |
| File | `AdAway-6.1.4-20241027.apk` |
| SHA-256 | `09f8e1528a53e5ffad59e57a174e90d4e10c5092bf4f6a60ab6594f046614417` |
| Signing cert SHA-256 | `d647fdac42961502ac78f99919b8e1901747e8da78fe13e1eaba688fecc4c99e`. v6.1.3 on GitHub is signed with the same key |
| Package | `org.adaway` |

- The F-Droid build is signed with **F-Droid's** key, not the developer's. Pick one source and stick with it, because Android won't let you update an app from one source with a build from the other.
- **Root mode** on KernelSU needs the systemless hosts module. On dubai that module only works after the bind-mount fix in [modules.md](modules.md). **Apply the fix before opening AdAway.** Otherwise AdAway writes to the full system partition and leaves `/system/etc/hosts` empty.
- After switching to root mode, **pause and resume** blocking once so AdAway writes its list into the bind-mounted file. The home screen should say **"Hosts file successfully updated"**.
- My result: 77,838 domains from 2 default sources. Ad domains resolve to `127.0.0.1`, and normal sites are unaffected. It still works after a reboot.
- **VPN mode** works without root or the module, but takes up your VPN slot.

### Iconify 8.0.0: ✅ installed and active (customizations not yet tested)

| | |
|---|---|
| Source | <https://github.com/Mahmud0808/Iconify/releases/tag/v8.0.0> |
| File | `Iconify.v8.0.0.apk`, package `com.drdisagree.iconify` |
| SHA-256 | `7cd95ac2355b156535b5ebf7646af60bfa9cf0e4d6050227d923747affcc8452` (matches GitHub) |
| Signing cert | `4af4ffa12ce90815a1775c4604ea16c19f5bed2a6e09ae7c3c92815982af052e` (same key as v7.3.0) |
| Android support | v8.0.0 is for **Android 16 QPR1 and later**; v7.3.0 is the last release for older versions. It came out in May 2026, **before Android 17**, so Android 17 isn't officially listed |
| Requirements | AOSP-based ROM ✅ · KernelSU/APatch/Magisk root ✅ · **Mountify** on KernelSU 3.0+ ✅ (see [modules.md](modules.md)) · LSPosed/Vector for its Xposed features ✅ |

Setup order that worked:

1. Install **Mountify**, reboot, set `MOUNT_DEVICE_NAME=KSU` in its web UI. You can skip the separate reboot if you install Iconify straight after; one reboot at the end covers both.
2. `adb install Iconify.v8.0.0.apk`
3. **Grant root manually** in ReSukiSU → Superuser. KernelSU managers don't show a root prompt.
4. Open Iconify, go through its setup, let it build its module, and reboot when asked.
5. **Scope it in Vector:** Vector → Modules → Iconify → Selection (☑︎≡) → **"Use what the module asks for"**. That adds **System Framework** and **System UI**. Tap **Apply**, then do a **full reboot** (not Vector's soft reboot), so Mountify also mounts at boot.

Gotchas I hit:

- **Iconify wasn't in ReSukiSU's Superuser list.** The manager caches its app list, so force-stop and reopen ReSukiSU after installing a new app.
- **Iconify said "root not accessible" after granting.** Tapping an app in the Superuser list only opens its page; the **Superuser switch** on that page must be on, and the list should show a pink **ROOT** badge. Iconify also keeps a "no root" result until restarted (`adb shell am force-stop com.drdisagree.iconify`), and then it worked.
- Iconify's README suggests ZygiskNext, but NeoZygisk does the same job. **Only one Zygisk implementation should be installed.**

- **"Module Not Activated: System service not running"** on Iconify's Xposed tab means it's enabled in Vector but **not scoped**. By default, Vector put only Iconify itself in scope. Step 5 above fixed it, and after a reboot the banner was gone.

Verified on 2026-11-04 after the reboot:
- **Mountify mounted Iconify's module:** 50+ `IconifyComponent*.apk` overlays are present in `/product/overlay`, and `cmd overlay list` shows them registered (one already enabled).
- **Xposed side active:** Iconify's Xposed tab shows Statusbar, Quick Settings, Lockscreen, Volume Panel and Launcher with no warning banner.
- **System UI loads the overlays without errors.**
- AdAway, NeoZygisk and Vector still work.

> ⚠️ **Known limitation:** regular apps marked `UMOUNT` in ReSukiSU (the default) can't see module files, so they log `Failed to open APK '/product/overlay/IconifyComponent….apk'`. I saw this in Launcher and Phone. System UI is unaffected, so volume panel, QS, status bar and lock screen tweaks should work. Overlays that target **other apps or the framework inside regular apps** (for example Launcher tweaks or icon packs in apps) may not apply there. Iconify's README suggests turning off "Umount modules by default" in that case, but that **weakens root hiding for every app**. The better option is turning off umount only for the specific apps you want themed (ReSukiSU → Superuser → app → App Profile).

*Still to test: an actual customization (for example the volume panel style) on Android 17.*

## Morphe-patched apps

Morphe (<https://morphe.software>) patches the official YouTube and Reddit apps (no ads, extra features). It downloads nothing itself: you give it the exact original APK version it asks for. I get those from APKMirror and check each one's signing certificate before patching.

| App | Status | Package after patching |
|---|---|---|
| Morphe Manager 2.3.0 | ✅ | `app.morphe.manager` |
| YouTube 21.30.216 (Morphe) | ✅ | `app.morphe.android.youtube` |
| Reddit 2026.40.0 (Morphe) | ✅ | `com.reddit.frontpage` |

### Morphe Manager 2.3.0

| | |
|---|---|
| Source | <https://github.com/MorpheApp/morphe-manager/releases/tag/v2.3.0> (linked from morphe.software) |
| SHA-256 | `d59f7bc835294a05814232e2cfd8bf22775434763e9a1ac26c88836b0be27dde` (matches GitHub) |
| Permissions it needs | notifications, all-files access, install unknown apps, unrestricted battery |

Granting those over adb:

```bash
adb shell pm grant app.morphe.manager android.permission.POST_NOTIFICATIONS
adb shell appops set app.morphe.manager MANAGE_EXTERNAL_STORAGE allow
adb shell appops set app.morphe.manager REQUEST_INSTALL_PACKAGES allow
adb shell dumpsys deviceidle whitelist +app.morphe.manager
```

### YouTube 21.30.216

| | |
|---|---|
| Original | APKMirror, **APK** (not bundle), universal nodpi, version code 1561187872 |
| SHA-256 | `3775a15129dcc4e5b80fc17b7e0841b7fdd1ca89a6e72baa798806f666c74d02` |
| Signing cert | `3d7a1223019aa39d9ea0e3436ab7c0896bfb4fb679f4de5fe7c23f326c8f994a` (Google's official YouTube key) |
| Result | 108/108 patches applied. It installs as a separate app, so Play Store updates don't touch it |
| Note | Signing into Google in the patched app may need MicroG |

### Reddit 2026.40.0

| | |
|---|---|
| Original | APKMirror, **bundle (.apkm)**, universal 120-640dpi, version code 2640091 (fits this phone's 400 dpi) |
| SHA-256 | `00396ce0f3746dc3687081c8c3d2f71d1d32b8958acd629ed8a34320518cfb9c` |
| Signing cert | `970b91143813b4c9d5f3634f672c9fcaa5621b4efaaedafd6c235cbbb869736f` (reddit Inc.) |
| Result | 29/29 patches applied. Morphe merges the bundle's split APKs itself |
| Note | Patched Reddit keeps the stock package name, so **turn off Play Store auto-update for Reddit**, or it will be replaced by the unpatched version. Uninstall the stock app before patching |

## Play Store apps (no root needed)

These all installed fine from the Play Store on Infinity X 4.1 GApps: Brave, WhatsApp, Google Drive, Photos, Messages, Maps, Slack, Bitwarden, Ente Auth, Obsidian, Claude, Gmail, Google Contacts, Google Phone.
