# Astroflux Redux

A community-run Astroflux project, with a Windows game launcher and an independently developed server backend. The goal is to preserve the game's feel while rebuilding the services needed to play it and maintain it.

This repository is the **public download and issue-tracking home for the launcher**. It does not contain the private server workspace, the original game client, game art, audio, or the Flash runtime.

**[Download the latest release](https://github.com/Torvail/AstrofluxRedux/releases/latest)**

> **Early public release.** The launcher is available, but gameplay and multiplayer compatibility are still being tested and corrected. Expect bugs, maintenance periods, and differences from the original game. A published installer is not a promise that the server is always online.

## Install and play

1. Open the [latest release](https://github.com/Torvail/AstrofluxRedux/releases/latest).
2. Download `AstrofluxRedux-Setup-<version>.exe`. You do not need the `.blockmap` or `latest.yml` files.
3. Run the installer. It installs for your Windows account, creates desktop and Start menu shortcuts, and opens the launcher.
4. Let the launcher check the server and download or verify the required game files.
5. Press **Play** when the launcher says **Ready to play**.
6. Use the game's built-in **Login** or **Register** screen to connect to this server. Steam is not required.

The public server address is included in the release installer. Players do not need to edit a hosts file, patch a SWF, or configure a server address themselves.

### Requirements

- Windows 10 or 11, 64-bit x64.
- Internet access for installation, updates, and online play.
- Disk space for the launcher, downloaded game client and runtime, and game asset downloads.
- A running public Astroflux Redux server and tunnel.

The installer is per-user and normally does not require administrator access. Use credentials created for this server, not the password for your Steam account or another service.

### Windows SmartScreen

The current installer is **unsigned**. Windows may show a SmartScreen warning because there is no publisher certificate.

Only proceed if you intended to install Astroflux Redux and downloaded the file from this repository's Releases page. Where Windows permits it, **More info > Run anyway** continues installation. If a device policy blocks unsigned software, follow that policy; do not disable security software to force the install.

## What the launcher actually does

### Before Play

- Checks the master server's health before starting the game.
- Reads the server's client-build manifest.
- Downloads a new client build when needed and verifies its size and SHA-256 checksum.
- Verifies the cached published client so a damaged download can be replaced.
- Falls back to the official client download if the server has no published build.
- Downloads the Flash Player projector separately if it is missing.
- Enables Play only after preparation succeeds.

If the launcher has been waiting at Ready for more than ten minutes, Play triggers another check so it does not blindly launch an old build.

### When Play is pressed

The launcher starts a local HTTP proxy, serves the client with its Player.io endpoint patched in memory, and forwards API requests to the configured master server. Flash Player opens the SWF from that local proxy; this is a desktop game window, not a browser tab.

The proxy listens on `127.0.0.1`, normally port `8080`, and picks a free port if that one is occupied. The game-room connection uses the server address supplied by the master.

The launcher minimizes while the game runs. **Keep it running:** closing the launcher also removes the proxy the game needs. A confirmation asks whether to keep playing or close both. When the game exits, the launcher returns and checks the game files again.

## Two separate update paths

| What changes | Where the update comes from | What players do |
| --- | --- | --- |
| Game client build | The Astroflux Redux master server | Let the launcher check and download before Play. |
| Launcher application | This repository's GitHub Releases | Let the launcher download the newer version, then choose **Restart to update**. |

A game-client update does not require a new launcher installer. A launcher update does not have to wait for the game server to be online, because its update feed is hosted on GitHub.

The installed launcher checks for a newer launcher version at startup. Updates download in the background, and a downloaded update also installs when the launcher quits. Restart to update is unavailable while the game is running; close the game first.

The fallback is always the [Releases page](https://github.com/Torvail/AstrofluxRedux/releases/latest): download and run the latest installer over the existing installation.

## Project status and development focus

The work is split between making the download-and-launch flow dependable and bringing the server's behavior into line with the client.

### Available now

- A dedicated Electron launcher window with progress and error states.
- A one-click, per-user Windows installer and shortcuts.
- A baked public server address for zero-configuration startup.
- Client-build discovery, download, and integrity checks.
- A loopback proxy and separate Flash game window.
- Retry and log-file access when preparation fails.
- Launcher self-update support through GitHub Releases.
- Server-side account login and game-room infrastructure.

These are implemented features, not a claim of complete gameplay parity.

### Current priorities

- **Weapon and projectile synchronization:** investigate client/server firing cadence, prediction, reconciliation, and visual jitter. Piraya is one of the reported cases under investigation.
- **Multiplayer behavior:** compare movement, combat, and room synchronization with the client rather than treating a successful login as proof that everything works.
- **Gameplay compatibility:** identify missing or incorrect progression, equipment, enemy, and interaction behavior.
- **Distribution reliability:** keep release metadata, downloaded builds, and launcher versions consistent.
- **Release hardening:** move the launcher to a supported Electron version and improve the signing/distribution process.

These are priorities, not dated delivery commitments. Fixes should be reproduced and tested before being described as resolved.

### Validation so far

The launcher release has been checked locally for installation, file preparation, Play opening the game login screen, and return to the launcher after the game closes. The release installer and update metadata were checksum-verified before publication.

The internal launcher test suite covers manifest parsing, size/hash verification, cached-file replacement, stalled downloads, malformed SWFs, endpoint patching, master health checks, proxy forwarding, and occupied-port handling.

This does **not** establish that every weapon, enemy, mission, or public multiplayer scenario is correct. Gameplay reports remain an important part of validation. This public repository does not currently contain the source workspace or CI workflows, so cloning it is not a way to build the launcher.

## Troubleshooting

| What you see | What to check |
| --- | --- |
| Server unavailable or offline | The host must have the public server and tunnel running. Wait and use **Try again**; repeatedly reinstalling cannot bring the server online. |
| Play remains disabled | Preparation has not finished, or it failed. Read the status message and use **Show log file** for details. |
| Download or verification fails | Check connectivity and available disk space, then retry. If it repeats, report the exact error; do not use an unverified client file to bypass the check. |
| Launcher update check fails | GitHub or the release metadata may be unavailable. Try a later launch or download the installer from Releases. Game-file updates are a separate path. |
| Flash displays `#2032` | This indicates a failed resource/network load. Include the full message and launcher log in a report; it is not enough to identify the cause by itself. |
| Game loses its connection after the launcher closes | The launcher owns the local proxy. Leave it running during play. |
| Projectiles jitter, shots disappear, or fire rates disagree | Report the weapon, upgrade level, equipment, and conditions. Client prediction and authoritative server behavior are still being investigated. |

### Local files and logs

The launcher keeps its downloaded files and diagnostic log under:

```text
%LOCALAPPDATA%\AstrofluxRedux
```

That folder contains the cached client, Flash projector, `launcher.json`, and `launcher.log`. On an error screen, **Show log file** opens Explorer with the log selected.

Do not delete the whole cache as your first troubleshooting step: the next run would need to download missing files again. Review logs before sharing them and redact personal information, credentials, tokens, or private configuration.

### Uninstalling

Use **Windows Settings > Apps** to uninstall Astroflux Redux. The installation and the download cache are separate; do not assume uninstalling removes every cached file. If you choose to remove the cache manually, close both the game and launcher first.

## Report a bug

Use the repository's [Issues](https://github.com/Torvail/AstrofluxRedux/issues). A reproducible report is more useful than a screenshot alone.

Include:

- Launcher version and game-build identifier from the launcher footer.
- Your Windows version.
- What you expected, what happened, and steps to reproduce it.
- Whether it happens consistently or only sometimes.
- Relevant log lines, reviewed and redacted before posting.
- A screenshot or short video when it helps explain the behavior.

For combat or synchronization problems, also include the weapon name and upgrade level, relevant artifacts or bonuses, whether you were moving or stationary, whether you held fire or clicked repeatedly, and whether the behavior occurs with other weapons.

Never post passwords, authentication tokens, or your full account data. Please check existing issues before opening a duplicate.

## Release contents

Each launcher release includes:

| File | Purpose |
| --- | --- |
| `AstrofluxRedux-Setup-<version>.exe` | The installer players download. |
| `AstrofluxRedux-Setup-<version>.exe.blockmap` | Metadata used for differential launcher updates. |
| `latest.yml` | The version, file name, size, and checksum used by the launcher update feed. |

The original client SWF, game assets, Flash runtime, private server code, operator tools, and account data are not release attachments. Required client/runtime files are fetched separately when the launcher prepares the game.

## Scope and limitations

- This is an independent community project, not an official Astroflux release.
- The current public build is an early release, not a complete recreation of every original feature.
- Server availability depends on the host and its public connectivity; there is no uptime guarantee.
- Flash Player is end-of-life. It is used here for compatibility with the existing client, not presented as a supported modern runtime.
- The current launcher uses Electron 33, which is also past end-of-life and needs upgrading.
- Download checksums help detect damaged files; they do not replace publisher code signing or make an unsigned installer trustworthy on their own.
- A public launcher release does not grant rights to redistribute Astroflux's original client, art, audio, or Adobe's runtime.

Astroflux and its original game content remain the property of their respective rights holders. Keeping the release limited to the launcher is deliberate; this repository is not a mirror of the original game files.
