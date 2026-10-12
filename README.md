# ArtCraft Launcher: Pages demo

Hosts the web build of the launcher on GitHub Pages, with a copy of each Craft app's latest web zip
next to it, so **Open** can download and run the apps from a browser. (GitHub's own file servers don't
let web pages read downloads; files on your own Pages site are same-origin, so they can.)

```
site/                   the launcher's dist/web, committed here (index.html, *.js, *.wasm, sw.js)
site/mirror/            made by the workflow on each deploy, not committed
tools/mirror.sh         copies each app's latest *-web-*.zip and SHA256SUMS.txt into site/mirror
.github/workflows/      builds nothing: checks site/, runs mirror.sh, deploys to Pages
```

## Set up

1. Make a new public repo (for example `launcher-demo`) and push this folder to its `main` branch.
2. Settings > Pages > Build and deployment > Source: **GitHub Actions**.
3. Build the launcher and copy it in (from the `craft-launcher` checkout):

   ```powershell
   cd apps\artcraft-launcher-web
   trunk build --release --cargo-profile wasm-release      # writes ..\..\dist\web
   robocopy ..\..\dist\web ..\..\..\launcher-demo\site /MIR /XD mirror
   ```

   (`robocopy` reports 1 to 3 as success. `/XD mirror` keeps it away from the generated folder.)
4. Commit and push `site/`. The workflow runs, and the page is at `https://<you>.github.io/<repo>/`.

## Updating

- **New launcher build:** repeat steps 3 and 4.
- **New app release:** the workflow also runs every 6 hours and refreshes the mirror. Run it by hand from
  the Actions tab (Deploy demo to Pages > Run workflow) to pick one up at once. Until then, Open on an
  app whose newest release isn't mirrored yet fails, because the other download routes don't work in a browser.

## Checking it works

Open the page, click **Open** on an app, and look in the browser's network tab for
`.../mirror/storytold/<app>/<tag>/<app>-web-<version>.zip` answering 200. A 404 there means the app has
no `*-web-*.zip` in its latest release, or the mirror hasn't caught up.

## Notes

- Only each app's latest stable release is mirrored. Pre-releases aren't.
- Pages sites are limited to about 1 GB; the 12 web zips are far under that unless the apps grow a lot.
- Scheduled workflows are switched off after 60 days without repo activity: push or run it by hand to wake it.
- `tools/mirror.sh` is a copy of `apps/artcraft-launcher-web/mirror.sh`. Keep its app list in step with the suite.
- The zips are checked against the SHA-256 GitHub publishes for each release (or the mirrored `SHA256SUMS.txt`
  when GitHub gives no digest), so a corrupted copy is refused.
