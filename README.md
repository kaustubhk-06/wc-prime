<p align="center">
  <img src="assets/brand/social-preview.png" width="640" alt="wc — Soft Pearl wordmark on charcoal">
</p>

# wc-prime

**wc-prime keeps a device online automatically.** It's a small background service (`wcprime`) that watches the Wi-Fi link. When the link breaks, it reconnects to the best network from your saved list. When a network shows a login page (campus, hotel or office), it signs in for you.

- **One static binary, standard library only.** No browser engine, runtime or daemon to install. It drives the Wi-Fi tools each OS already ships.
- **Lightweight.** It runs one goroutine loop and does one HTTP request per check while healthy (every 15 s by default). The heap is capped at 24 MB. Idle RSS is typically under 15 MB, and CPU use rounds to zero.
- **Linux, macOS, Windows and Android.**

Created in [T3 Code](https://t3.codes).

## What it fixes

| Problem | How it's detected | What wcprime does |
| --- | --- | --- |
| Wi-Fi off | OS radio state | Turns Wi-Fi on |
| Not connected to any network | OS association state | Scans, then joins the highest-priority saved network in range |
| Connected, stuck behind a login page | The plain-HTTP connectivity check is redirected or rewritten | Submits the saved username and password to the portal, or moves to another saved network |
| Connected, no internet | Every connectivity endpoint fails, twice in a row | Tries the saved portal login URL (for portals that block DNS), then switches network |
| Poor or unusable network | Weak signal, or slow or lossy checks confirmed by follow-up probes, three times in a row | Switches only when a saved network that's clearly stronger is in range |

Safety rails:

- wcprime only ever joins networks you saved.
- A network that just failed is skipped for 5 minutes.
- If no alternative is in range, it stays on the network it has; a weak link beats none.
- After a scan that finds nothing to join, it waits before rescanning, which saves CPU and battery.

## Supported networks

| | Linux (NetworkManager) | Windows | macOS | Android |
| --- | --- | --- | --- | --- |
| Open | ✅ | ✅ | ✅ | ✅ |
| WPA2 / WPA3 passphrase | ✅ | ✅ | ✅ | ✅ |
| WPA2/3-Enterprise (802.1X) | ✅ PEAP / TTLS | ✅ PEAP-MSCHAPv2 | ⚠️ install the institution's profile once* | ⚠️ add once in Settings* |
| Captive portal login | ✅ | ✅ | ✅ | ✅ |
| Hidden SSIDs | ✅ | ✅ | ✅ | ✅ |

\* macOS and Android don't let command-line tools create 802.1X configurations. After you add the network once through the OS, wcprime still handles that network's captive portal, priorities and fallback.

### Captive portals

Portal login is plain HTTP; no browser is involved. wcprime follows the redirect from the connectivity check, plus any meta-refresh or JavaScript `location` hops, and finds the login form. It keeps hidden fields such as CSRF tokens and session IDs, fills in the username and password, ticks "accept terms" boxes, and submits with the cookies the portal set. It then checks that the internet is really reachable.

If a portal builds its form with JavaScript, set these in the network's portal options:

- a fixed **login URL**
- the **username and password field names**
- any **extra fields**

wcprime then posts to the login URL directly.

## Install

Download a binary from the CI artifacts, or build it yourself (Go 1.22+):

```sh
scripts/build.sh            # cross-compiles every platform into dist/
# or just for this machine:
go build -trimpath -ldflags "-s -w" -o wcprime ./cmd/wcprime
```

Run it as a background service that starts at login:

| Platform | How |
| --- | --- |
| Linux | `cp wcprime ~/.local/bin/ && cp deploy/linux/wcprime.service ~/.config/systemd/user/ && systemctl --user enable --now wcprime` |
| macOS | Copy the binary to `/usr/local/bin`, then follow [`deploy/macos/com.wcprime.agent.plist`](deploy/macos/com.wcprime.agent.plist) (LaunchAgent) |
| Windows | `powershell -ExecutionPolicy Bypass -File deploy\windows\install.ps1` (registers a hidden Scheduled Task at logon) |
| Android | See [`deploy/android/README.md`](deploy/android/README.md). Needs root or `adb shell`, because only those users may control Wi-Fi. |

On Linux, run the service as your own user, not root. NetworkManager's polkit rules already let the logged-in user manage Wi-Fi.

## Manage networks

```sh
wcprime url      # prints http://127.0.0.1:7878/#t=<token>
```

Open that link in any browser. From there you can:

- add, edit, enable or disable saved networks
- reorder networks: the top entry is the most preferred
- save a network straight from a scan
- see the live status and recent events

The UI listens only on loopback. It needs the per-install token and rejects requests with a foreign `Host` header, which blocks DNS rebinding. Secrets are write-only: once saved, the UI can replace them but never read them back.

Other commands:

```sh
wcprime run [-dir DIR] [-listen 127.0.0.1:7878] [-interval 15s] [-fast-interval 5s]
            [-min-signal 20] [-max-rtt 1.5s] [-no-ui]
wcprime check     # one-off diagnosis: Wi-Fi state + internet / captive / offline
wcprime version
```

## Credential storage

Credentials live in `vault.bin` in the config directory:

| OS | Config directory |
| --- | --- |
| Linux | `~/.config/wc-prime` |
| macOS | `~/Library/Application Support/wc-prime` |
| Windows | `%APPDATA%\wc-prime` |

- **Encryption.** The vault is encrypted with AES-256-GCM using a random 256-bit key in `vault.key`. On Windows the key is also wrapped with DPAPI for your user account. On other systems the key file is readable only by you (`0600` in a `0700` directory).
- **Secrets stay with the service.** The UI and API never return passwords. On Linux, NetworkManager profiles are created with "not saved" secret flags, so the passphrase is handed over at connect time and not stored by the OS.
- **Threat model.** This protects the vault at rest, including copies in backups or sync folders. It does not protect against malware already running as your user, which could read the key just as the service does.

Platform caveats:

- **macOS:** `networksetup` takes the passphrase as a command-line argument, so it's briefly visible to local users through `ps`.
- **Windows:** the profile is handed to WLAN AutoConfig, which stores the key itself.

## Layout

```
cmd/wcprime        CLI and service wiring
internal/keeper    control loop: classify the link and repair it
internal/probe     connectivity / captive-portal / quality checks
internal/portal    tiny HTML form parser and portal login (no browser)
internal/wifi      per-OS backends: nmcli, netsh + wlanapi, networksetup, cmd wifi
internal/vault     encrypted credential store
internal/ui        embedded single-page UI and JSON API
deploy/            systemd, launchd, Task Scheduler, Termux:Boot
```

## Branding

**Soft Pearl** is the official wc wordmark: rounded lettering, a subtle opal finish, and a charcoal background.

The [brand assets](assets/brand) include the scalable wordmark, project icon, website icons, and social preview image. Use these shared assets for consistent branding across the project and website.
