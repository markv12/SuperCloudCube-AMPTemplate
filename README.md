# SuperCloudCube — AMP Configuration Repository

This repo lets anyone run the SuperCloudCube dedicated server in [AMP](https://cubecoders.com/AMP)
with almost no setup: they add this repo once, pick **SuperCloudCube** from the app list, and hit
**Update** — AMP downloads the latest server build from this repo's GitHub Releases automatically.

## Repo contents

| File | Purpose |
|---|---|
| `manifest.json` | Marks this repo as an AMP `AppTemplates` configuration repository. |
| `supercloudcube.kvp` | Module definition (executable, args, env, ready/stop behaviour). |
| `supercloudcubeconfig.json` | User-editable settings: port, save name, player limit. |
| `supercloudcubemetaconfig.json` | Empty — the server is arg-driven, no config file. |
| `supercloudcubeports.json` | Single UDP game port (default 10000). |
| `supercloudcubeupdates.json` | Update logic: download the build zip from Google Drive → unzip → chmod +x. |
| `package-server.ps1` | (In `BuildServer/`) zips the Linux build into `SuperCloudCubeServer-Linux.zip`. |

The build zip itself is **not** committed (see `.gitignore`) — it's hosted on Google Drive.

The build zip is hosted on **Google Drive**; `supercloudcubeupdates.json` downloads it via a
direct-download URL (the `drive.usercontent.google.com/download?...&confirm=t` form, which
bypasses Drive's scan-warning interstitial).

## One-time setup (you, the author)

1. **Create a public GitHub repo**, e.g. `SuperCloudCube-AMPTemplate`.
2. **Replace `YOUR_GH_USER`** with your GitHub username in `manifest.json` (`origin`, `url`).
3. **Commit and push** these template files to the repo's `main` branch.
   (The Drive download URL is already baked into `supercloudcubeupdates.json` — no edit needed.)

## What your friend does (per server)

1. **Add the repo once**: AMP → *Configuration → Instance Deployment → Add → Configuration Repository*,
   enter `YOUR_GH_USER/SuperCloudCube-AMPTemplate:main`.
2. **Create an instance** and choose **SuperCloudCube** from the application list.
3. Click **Update** (downloads the latest release build + chmods it).
4. Confirm the **Game Port** (UDP 10000 — forward it on the network), set a world name, **Start**.

## Shipping an update

Build → `package-server.ps1` → in Google Drive, open the existing file → **Manage versions →
Upload new version**. This keeps the **same file ID / URL**, so your friend just clicks **Update**
on their instance — nothing to re-share. (If you instead upload a *new* file, the ID changes and
you'd have to update the URL in `supercloudcubeupdates.json` and re-push.)

> Note: Google Drive can throttle a file with a temporary "download quota exceeded" error if it's
> pulled very frequently. Fine for a few friends; if it becomes a problem, switch back to a GitHub
> Release asset (`UpdateSource: GithubRelease`).

## How it maps to the server

- **Launch**: `SuperCloudCubeServer.x86_64 -batchmode -nographics -logfile /dev/stdout -bind 0.0.0.0 -port <allocated> -save <name>`
- **Ready detection**: matches `[DedicatedServer] Listening on` from `DedicatedServerBootstrap`.
- **Graceful stop**: `OS_CLOSE` (SIGINT/SIGTERM) → Unity `OnApplicationQuit` → `WorldSaveManager.SaveWorld()`.
- **Saves**: `XDG_CONFIG_HOME`/`HOME` are pointed at `<instance>/SuperCloudCube/saves`, so worlds
  stay inside the instance dir (isolated + backed up), and `SmartExcludeExemptions` protects them on update.
