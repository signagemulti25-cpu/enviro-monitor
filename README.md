# Enviro Monitor for GitHub Pages

This React dashboard displays stations in a vertical list, with line and bar
history charts, daily summaries, and CSV downloads. The browser loads a
same-origin snapshot. A GitHub Actions runner fetches the logger API and
retains up to 30 days of readings, plus the last reading for a stale station.

## Set up your seven stations

Edit `stations.json`. Only logger 1340 is included because the remaining
numeric IDs weren't supplied. Add the other six entries, using numeric IDs
as strings. Set `"channels": 5` for each of the five stations with AQI on ch5.
Set `"channels": 8` for the two stations with AQI on ch8. Both layouts use
ch1–ch4 for temperature, humidity, PM2.5 and PM10.

## Preview locally

Use Node.js 24 LTS. From this folder, run:

```powershell
npm ci
npm run collect
npm run dev
```

Open the local URL printed by Vite. For a production preview, run
`npm run build` and then `npm run preview`. Generated snapshots are excluded
from Git. The browser retries its published snapshot every minute.

## Deploy to your existing GitHub repository

1. Clone your existing repository into a new folder.

   ```powershell
   git clone https://github.com/signagemulti25-cpu/enviro-monitor.git enviro-monitor-web
   ```

2. Copy the contents of **enviro_monitor_react** into the cloned folder.
   Include the hidden-by-name folder `.github` and `.gitignore`. Keep the
   existing Git metadata. Update `stations.json` with your real station list.
3. On GitHub, set **Settings → Pages → Source → GitHub Actions**.
4. From the cloned folder, commit and push.

   ```powershell
   cd enviro-monitor-web
   git add .github .gitignore index.html package.json package-lock.json vite.config.js stations.json scripts src README.md
   git commit -m "Deploy React Enviro Monitor"
   git push origin main
   ```

5. In **Actions**, open **Collect readings and publish dashboard**. After the
   build and deployment succeed, open
   https://signagemulti25-cpu.github.io/enviro-monitor/ . To collect or publish
   immediately, select **Run workflow**. If Chrome still shows the old Flutter
   app, test in an Incognito window or clear the site's data for the Pages
   origin to remove the previous Flutter service worker.

The workflow deploys `dist/`. Existing Flutter files in `docs/` are outside
the publishing source once GitHub Actions is selected. Vite builds with
relative asset paths so this works under the repository URL.

## Data behavior

- The workflow targets a ten-minute schedule. GitHub can delay or skip runs.
  Collection can continue while your Windows PC is off.
- Successful polls are deduplicated by reading ID per station. If a request
  fails, the last available readings are kept and the station shows a failure.
- The workflow writes its archive to the repository's `measurements` branch
  and includes a snapshot in the published site. This requires permission
  for the workflow token to push that branch. If a ruleset restricts the
  branch, check the **Save new readings** step for the exact error.
- The logger endpoint supplies the latest reading, so history starts with
  collection and can have gaps. Existing Windows reading files are not imported
  automatically. The website shows a rolling 30-day history. Old commits can
  still contain earlier data.
- On this public site, visitors can download the published data, including
  station names and IDs. Use public-safe labels and publish only stations
  whose readings you intend to share.
