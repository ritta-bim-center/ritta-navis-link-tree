# OTA update — how it works

The add-in checks for a newer version every time the Clash Tracking pane opens. It fetches one
small JSON file from your GitHub Pages site and compares it to the version built into the DLL
(`Config.CurrentVersion` / `AssemblyVersion`). If the JSON says a higher version, a yellow
**"Update available"** banner appears with an **Update** button that opens the download URL.

It only NOTIFIES — a DLL that Navisworks has loaded can't replace itself, so the user downloads and
runs the new installer after closing Navisworks.

```
Navisworks opens pane ──▶ GET rittatools-navisworks.json ──▶ version > mine? ──▶ show banner ──▶ open URL
```

## The manifest file
`rittatools-navisworks.json` (in this folder) is the manifest. Fields:

| field | meaning |
|---|---|
| `version` | latest released version, e.g. `"1.1.0"` (compared with the installed one) |
| `url` | where **Update** sends the user — a download page or a direct installer link |
| `notes` | changelog shown under the banner (`\n` for line breaks) |
| `mandatory` | reserved for a future force-update; keep `false` for now |

The add-in already points at:
`https://ritta-bim-center.github.io/ritta-navis-link-tree/rittatools-navisworks.json`
(set in `ClashSync/Config.cs → VersionManifestUrl`).

## One-time setup on the GitHub Pages repo
Your Pages site is the repo behind `ritta-bim-center.github.io/ritta-navis-link-tree`.
Copy **`rittatools-navisworks.json`** into that repo's **root** (next to the link-tree `index.html`)
and commit. It is a standalone file — it has nothing to do with the HTML page; GitHub Pages just
serves any file in the repo. After the commit it is reachable at:

`https://ritta-bim-center.github.io/ritta-navis-link-tree/rittatools-navisworks.json`

Open that URL in a browser once to confirm it loads.

## Publishing a new release (every time)
1. Bump the version in **all three** places so they match:
   - `Properties/AssemblyInfo.cs` → `AssemblyVersion` / `AssemblyFileVersion`
   - `ClashSync/Config.cs` → `Config.CurrentVersion`
   - `installer/RittaTools.iss` → `AppVersion`
2. Build the installer: `installer\build-installer.cmd obf`
3. Put the new `RittaTools-Navisworks2026-Setup-<ver>.exe` somewhere users can download it:
   - **GitHub Release** (recommended): create a release on the repo, attach the `.exe`; its asset
     URL is a stable direct-download link — use that as `url`.
   - Or a SharePoint share link (like the pyRevit downloads) — opens the page, user clicks download.
3. Edit `rittatools-navisworks.json` in the Pages repo: set `version` to the new number, `url` to
   the download link, and `notes` to the changelog. Commit.

That's it — the next time anyone opens the pane, they see the banner.

## Notes
- The add-in adds a `?t=<timestamp>` cache-buster to the request, so a fresh commit is picked up
  without waiting for the GitHub CDN cache (which is a few minutes otherwise).
- To test the banner without a real release: temporarily set `version` in the JSON higher than the
  installed one (e.g. `"9.9.9"`), commit, reopen the pane. Set it back afterwards.
- The check never blocks the pane — if the file is missing or the network is down, it silently
  shows no banner.
