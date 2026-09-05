# Diwali Live Stream — India Shines ✨

A glowing, animated India — decorated with flickering diyas, fireworks, and a
rangoli — streamed live to YouTube straight from a GitHub Actions workflow.

## What's in this repo

- `index.html` — the animated scene (pure HTML5 canvas, no dependencies, no
  internet connection needed to render). Twinkling stars, a real India map
  (traced from actual coastline data, with the Andaman & Nicobar Islands and
  Sri Lanka in their true positions) lined with ~110 flickering diyas, a
  rotating rangoli, a live countdown to Diwali, and six different types of
  fireworks (peony, chrysanthemum, willow, ring, crackle, and palm bursts)
  plus a ground fountain and spinning wheel.
- `.github/workflows/stream.yml` — launches a virtual display on the Actions
  runner, opens the page fullscreen in Chrome, and pipes the screen to
  YouTube Live via `ffmpeg`. Chrome itself is installed with the
  `browser-actions/setup-chrome` action (not `apt`), since the `chromium-browser`
  apt package is an unreliable snap-redirect stub on current Ubuntu runners.

## Setup

1. **Get your YouTube stream key.**
   Go to [YouTube Studio → Go Live](https://studio.youtube.com/) → "Stream"
   tab → copy your **Stream key**. (New channels sometimes need to wait ~24
   hours after enabling live streaming for the first time before they can go
   live.)

2. **Add it as a repo secret.**
   In your GitHub repo: **Settings → Secrets and variables → Actions → New
   repository secret**.
   - Name: `YOUTUBE_STREAM_KEY`
   - Value: (paste the key you copied)

3. **Push this repo to GitHub**, then go to the **Actions** tab and run
   "Diwali Live Stream" manually (`Run workflow`), or just wait for the
   scheduled trigger.

4. Open your YouTube Studio "Go Live" page — once ffmpeg connects
   (usually 20–40 seconds after the job starts), you'll see the preview and
   can click **Go Live**.

## Important things to know before you rely on this

- **GitHub Actions jobs have a hard 6-hour limit.** There's no way around
  this on standard runners. This workflow sets the job timeout to ~4h50m and
  re-triggers itself every 5 hours via `cron`, so the stream restarts
  automatically — but expect a **gap of a minute or two** between each
  segment while the new job spins up and reconnects to YouTube. It is not
  truly seamless 24/7.
- **Actions minutes are limited/billed.** On a free personal GitHub account
  you get a limited number of free minutes per month (and none on private
  repos beyond the free tier without billing set up). A near-continuous
  stream will burn through that budget fast. Public repos get unlimited
  minutes on standard runners, so consider making the repo public if you
  intend to stream long stretches.
- **This is a workload GitHub's compute wasn't really designed for.**
  GitHub Actions is intended for CI/CD (building, testing, deploying code),
  and GitHub's Acceptable Use Policies discourage using Actions runners for
  unrelated persistent workloads. Using it to run a continuous livestream
  is unlikely to get you individually flagged for a one-off event, but
  running it as a permanent 24/7 service is the kind of sustained,
  non-CI use that has led GitHub to restrict accounts in the past. If you
  want genuine 24/7 uptime, a small always-on VPS (e.g. running the same
  Chromium + ffmpeg pipeline) is the more appropriate and reliable tool —
  happy to help you set that up instead if you'd like.
- **YouTube needs live streaming enabled on your channel** and, for new
  channels, verified + waited through YouTube's first-time 24-hour hold.

## Customizing the visual

Everything is in `index.html`:
- `INDIA_MAINLAND` / `INDIA_ANDAMAN` / `SRI_LANKA` — real coastline coordinates
  (projected from actual lat/long data), not a stylized shape.
- `MAP_SCALE` / `MAP_OFFX` / `MAP_OFFY` — resize or reposition the whole map.
- `diyaSpots` — density of diyas around the mainland border
  (`perimeterPoints(mainlandPts, 110)`, change `110` to add/remove diyas).
- `crackerTypes` — the list of firework styles in rotation (peony,
  chrysanthemum, willow, ring, crackle, palm). Remove/reorder to change the mix.
- `spawnFirework` interval — how often fireworks launch (currently every 0.85s).
- `DIWALI_TARGET` — the countdown target date/time (currently Nov 8, 2026,
  midnight IST).
- Title/subtitle/Hindi text and colors — in the `<style>` block and the
  `<div>`s near the top of `<body>`.

## Testing locally before you stream

You can preview the scene in any browser without touching GitHub Actions —
just open `index.html` directly, or serve it locally:

```bash
python3 -m http.server 8080
# then visit http://localhost:8080/index.html
```

To test the full capture+stream pipeline locally on Linux (with Xvfb,
Chromium, and ffmpeg installed), run the same commands as in
`stream.yml` on your own machine — this lets you verify your stream key
and bitrate before relying on GitHub Actions.
