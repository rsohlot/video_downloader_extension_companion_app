# Video Downloader Companion App

Companion app for the [Video Downloader Firefox extension](https://addons.mozilla.org/en-US/firefox/addon/video-grabber/) to download HLS (`.m3u8`) streams using ffmpeg.

## Features

- 🚀 **HLS stream downloads** via ffmpeg
- 🔐 **Authenticated streams** — automatically forwards cookies and headers from the browser
- 📂 **Open Folder** — click a completed download in the extension to reveal the file in your native file manager
- ✕ **Cancel** — stop an active or queued download at any time
- 🔒 **Localhost-only, token-authenticated** HTTP API (not reachable from the internet or other websites)

## Downloads

| Platform | Download |
|----------|----------|
| 🪟 Windows | [video-downloader-companion-windows.zip](installers/video-downloader-companion-windows.zip) |
| 🍎 macOS | [video-downloader-companion-macos.tar.gz](installers/video-downloader-companion-macos.tar.gz) |
| 🐧 Linux | [video-downloader-companion-linux.tar.gz](installers/video-downloader-companion-linux.tar.gz) |

## Requirements

- **ffmpeg** installed and available on `PATH`
- **Firefox** with the Video Downloader extension installed

## Installation

### Windows
```powershell
Expand-Archive video-downloader-companion-windows.zip -DestinationPath companion
cd companion
.\install-windows.ps1
```

### macOS
```bash
tar -xzf video-downloader-companion-macos.tar.gz
chmod +x install-macos.sh
./install-macos.sh
```

### Linux
```bash
tar -xzf video-downloader-companion-linux.tar.gz
chmod +x install-linux.sh
./install-linux.sh
```

## Auto-Start on Login (Windows, optional)

The companion runs as a small Node.js HTTP server and stops on reboot. Pick the option that fits you:

### Option A — Startup Folder (easiest)

1. Press `Win + R`, type `shell:startup`, press Enter.
2. In the Startup folder, right-click → **New → Shortcut**.
3. Point it at `start-server-silent.vbs` inside the extracted `companion` folder.
4. Name it `Video Downloader Companion`.

The server will launch silently on every login. To remove, delete the shortcut.

### Option B — Scheduled Task (most reliable)

Run in an **elevated** PowerShell (adjust the path to where you extracted the zip):

```powershell
$vbsPath = "C:\path\to\companion\start-server-silent.vbs"
$action    = New-ScheduledTaskAction -Execute "wscript.exe" -Argument "`"$vbsPath`""
$trigger   = New-ScheduledTaskTrigger -AtLogOn -User $env:USERNAME
$settings  = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries -StartWhenAvailable
$principal = New-ScheduledTaskPrincipal -UserId $env:USERNAME -LogonType Interactive
Register-ScheduledTask -TaskName "VideoDownloaderCompanion" `
  -Action $action -Trigger $trigger -Settings $settings -Principal $principal `
  -Description "Auto-starts Video Downloader HLS companion server on login"
```

To remove: `Unregister-ScheduledTask -TaskName "VideoDownloaderCompanion" -Confirm:$false`.

### Option C — Manual

Double-click `start-server.bat` (shows logs in a terminal) or `start-server-silent.vbs` (no window).

### Verify the server is running

```powershell
Invoke-RestMethod -Uri "http://127.0.0.1:9876/ping" -Method POST
```

Should return `success: True`.

## Security

- HTTP server binds to `127.0.0.1` only — not reachable from the LAN or the internet.
- Every protected endpoint requires a per-session Bearer token stored in the user's temp directory.
- `/openFolder` is restricted to files inside your configured downloads directory.

Please report security issues via a GitHub Security Advisory on this repo.

## License

MIT — see `LICENSE`.


---

**Version:** v1.2.1 | **Updated:** 2026-04-17
