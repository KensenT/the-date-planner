# Petal — a date route planner

A small web app that takes a handful of places you want to visit in one day and works out the fastest order to see them all, then hands the finished route off to Google Maps for turn-by-turn directions.

Built to solve a real problem: my girlfriend and I would spend ten or fifteen minutes manually rearranging stops in Google Maps before every multi-stop date. This does it in a second.

**Live at:** [kensent.github.io/petal-the-date-planner](https://kensent.github.io/petal-the-date-planner/)

## How it works

1. Add every place you want to visit, in any order.
2. Choose your starting point (the first stop you added, or your current location).
3. Pick a travel mode — driving, motorcycle, walking, or cycling.
4. Tap *Find the best route*. The app computes travel times between every pair of stops and solves for the shortest total route, then shows the reordered itinerary with a one-tap *Open in Google Maps* button.

Under the hood: the Traveling Salesman Problem is solved optimally via Held-Karp dynamic programming for up to 12 stops, with a nearest-neighbor + 2-opt heuristic as a fallback for larger inputs.

## Stack

Single `index.html`, no build step, no backend. Plain HTML/CSS/JS.

- **Geocoding & place search:** [Photon](https://photon.komoot.io/) (primary) and [Nominatim](https://nominatim.org/) (fallback), both powered by OpenStreetMap data. Optional upgrade to [Google Places](https://developers.google.com/maps/documentation/places/web-service) when a key is configured.
- **Travel-time matrix:** [Google Routes API](https://developers.google.com/maps/documentation/routes) (traffic-aware) when a key is set, [OSRM](https://project-osrm.org/) public demo server otherwise. Haversine straight-line distance as a last-resort fallback.
- **Navigation hand-off:** [Google Maps URLs API](https://developers.google.com/maps/documentation/urls/get-started) — no key required for this part.
- **Location bias:** browser timezone for country-level filtering; optional precise coordinates if the user grants geolocation.

## Running it yourself

```bash
# Clone and serve
git clone https://github.com/KensenT/petal-the-date-planner.git
cd petal-the-date-planner
python3 -m http.server  # or any static server
# Open http://localhost:8000
```

Or just drop `index.html` on GitHub Pages, Netlify, Cloudflare Pages, or any static host. There's no build step.

### Optional: Google API key

Petal works out of the box using only open-source data — no account, no key, no setup. But plugging in a Google Maps API key unlocks three meaningful upgrades:

- **Richer place search and exact pin accuracy.** The Google Places autocomplete covers small businesses that OpenStreetMap misses, and pins land on the actual venue when the route opens in Google Maps.
- **Traffic-aware travel times.** Time estimates match what Google Maps itself will show you, instead of free-flow times that undercount real driving time by 30–50% in dense cities.
- **Motorcycle mode.** Google's two-wheeler routing accounts for shortcuts that cars can't take — a real win for anywhere traffic-heavy with motorbike culture. This mode is disabled unless a key is configured, because no open-source routing profile produces sensible two-wheeler results.

To set one up: open Settings in the app and follow the guide. You'll need to enable three APIs on your Google Cloud project — **Places API (New)**, **Maps JavaScript API**, and **Routes API**. The key is stored only in your browser's localStorage and is restricted by HTTP referrer to your domain, so it can't be used from anywhere else.

**Set a daily quota cap in the Google Cloud Console** as an extra safety net — even if somehow your key leaks, the quota cap means the worst-case damage is bounded to whatever you chose.

### Honest labeling

Estimates are always tagged with their data source so you know what you're looking at:

- *with typical traffic* — Google Routes, traffic-aware (matches Google Maps)
- *Google estimate* — Google Routes for walking or cycling (no traffic concept)
- *no traffic — real time will be longer* — OSRM free-flow driving
- *rough straight-line estimate* — haversine fallback when nothing else is available

When Google can't route a mode in your area (e.g. cycling in Jakarta), Petal won't silently fall through to a made-up straight-line estimate — it'll tell you to pick a different mode instead.

### Data persistence

Your stops and settings persist across sessions — close the tab, reopen it tomorrow, and your itinerary is still there. Only the **Clear all** button wipes it. This is useful for building up a list of date ideas over a week and then optimizing the route the day of.

Specifically, Petal saves (in your browser's localStorage, never sent to a server):

- The list of stops — name, address, coordinates, Place ID
- Your travel mode selection
- The "Return to start at the end" checkbox
- Your start-from choice (first stop vs. current location)

A few things are intentionally *not* persisted:

- The resolved GPS coordinates of your current location — the browser asks again each session anyway
- The optimized route itself — it's recomputed on demand so traffic data stays fresh
- The search input's in-progress text

If you're in Incognito/Private mode, the persistence lives only for that window, which is the browser's behavior and probably what you want there.

## License

[MIT](LICENSE). Do what you like with it.

## A note on authorship

This is an honest collaboration: product direction, testing, and most of the debugging decisions are mine. The code itself was written iteratively with [Claude](https://claude.ai) as a pair programmer over many turns. Neither of us could have shipped it alone — Claude wouldn't have known what problem to solve or when its output was wrong; I wouldn't have written 3,000 lines of TSP solvers, Google Places integrations, and flex-layout edge-case handling in the same evening.

If you're curious how that process actually looked — including all the dead ends, course corrections, and "that's not what I meant" moments — the commit history tells the story.