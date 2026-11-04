# Apps

## Root apps

| App | Version | Status | Notes |
|---|---|---|---|
| AdAway | 6.1.4 | ⚠️ | Installed; blocking isn't active until the hosts module works (see [modules.md](modules.md)) |

### AdAway 6.1.4: ⚠️

| | |
|---|---|
| Source | <https://github.com/AdAway/AdAway/releases/tag/v6.1.4> (official, developer-signed) |
| File | `AdAway-6.1.4-20241027.apk` |
| SHA-256 | `09f8e1528a53e5ffad59e57a174e90d4e10c5092bf4f6a60ab6594f046614417` |
| Signing cert SHA-256 | `d647fdac42961502ac78f99919b8e1901747e8da78fe13e1eaba688fecc4c99e`. v6.1.3 on GitHub is signed with the same key |
| Package | `org.adaway` |

- The F-Droid build is signed with **F-Droid's** key, not the developer's. Pick one source and stick with it, because Android won't let you update an app from one source with a build from the other.
- **Root mode** on KernelSU needs the systemless hosts module. **VPN mode** works without root but takes up your VPN slot.

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
