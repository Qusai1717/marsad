# Marsad Finance — Website (no framework, no build step)

Single-file app: `index.html`. Open it directly, or host the folder anywhere static.

## Run locally
Double-click `index.html`, or serve it (recommended, so `fetch` behaves normally):
```bash
python -m http.server 8080     # then open http://localhost:8080
```

## Deploy (any static host)
- **Cloudflare Pages** — connect the repo (or drag-drop the folder). Build command: *none*.
  Output directory: `/` (this folder).
- **Netlify / Vercel / GitHub Pages** — same: no build, publish this folder.
Before going live, replace `REPLACE-WITH-YOUR-DOMAIN` in `robots.txt` and `sitemap.xml`.

## Structure
- 3 tabs: **Home** (USD/LYD hero + chart, watchlist, quick convert, indicators, top news),
  **Convert** (keypad + recents), **News** (featured + grid + categories + bookmarks).
- Currency detail opens full-screen from any rate: interactive chart (crosshair, wheel/pinch
  zoom, drag pan, double-click reset), ranges 1W→MAX, event markers, compare (up to 5 series).
- Arabic default with full RTL; English one tap away. Both persist in `localStorage`.

## Data (all real, attributed)
| Purpose | Source |
|---|---|
| Latest ECB reference rates | Frankfurter API |
| Latest rates, ~160 currencies | open.er-api.com |
| Daily history, ~200 currencies | Currency-API (jsDelivr) |
| Libya indicators | World Bank open API |
| Parallel rate | the `PARALLEL` constant in `index.html` (set by you) |
| News | RSS ×13: Libya Observer, Libya Herald, BBC Arabic Business, BBC Business, France24 AR, Al Arabiya Aswaq, OilPrice, Kitco (gold), Investing Commodities, CNBC Economy, MarketWatch, Yahoo Finance, IMF |

Offline/first paint uses **real embedded snapshots** (labeled as snapshots in the UI).
Nothing is ever fabricated — missing data is stated plainly.

## News reliability (read this if sources look thin)
The news header shows **“x/13 sources”** — tap it to see exactly which publishers
answered and which failed. Feeds are fetched over 3 lanes (not all at once) and each
feed rotates through several mirrors, then a JSON bridge, then a direct attempt.
Last good articles are cached in `localStorage`, so a bad refresh never empties the page.
If a source is permanently red, delete it from `News.FEEDS` or swap the URL.

## News & CORS (important)
Browsers block direct RSS fetches (publishers send no CORS headers), so requests go through
public mirrors listed in `News.PROXIES`. For production, put your own proxy first in that
array — e.g. a Cloudflare Worker:

```js
export default {
  async fetch(req) {
    const url = new URL(req.url).searchParams.get('url');
    if (!url) return new Response('missing url', { status: 400 });
    const r = await fetch(url, { headers: { 'User-Agent': 'MarsadBot/1.0' } });
    return new Response(await r.text(), {
      headers: { 'content-type': 'application/xml',
                 'access-control-allow-origin': '*',
                 'cache-control': 'public, max-age=600' } });
  }
}
```
Then: `PROXIES:["https://your-worker.workers.dev/?url=", ...]`

## Updating the parallel (black) market rate — you, in code
Open `index.html`, search for **EDIT ME**, change two lines, redeploy:

```js
const PARALLEL = {
  usdLyd : 7.20,          // dinars per US dollar (null = section hidden)
  updated: "2026-07-30",  // the day you set it
  note   : "",            // optional context, e.g. "Tripoli street rate"
  staleDays: 3,
};
```

Visitors can't change it — there is no editor, no admin mode, no API for it.
While `usdLyd` is `null` the section stays hidden rather than showing a number
we can't stand behind. It's always badged *unofficial estimate*, shows the gap
vs the official rate, and after `staleDays` marks itself "may be outdated".

Keep it in sync with the app's `AppConfig.parallelUsdLyd`.

## Editing
Everything lives in `index.html`:
- `News.FEEDS` — add/remove sources
- `News.CATS` — category keywords
- `I18N` — all Arabic/English strings
- `:root` CSS variables — the Marsad gold/cream palette
- `ParallelSource.fetch()` — return `{v, src, at}` from a vetted parallel-market source and
  that dashboard section appears automatically (hidden until then, by design).
