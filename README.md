# Face Test

A single-page "wall of faces" magic app, live at **[faces.oddpaq.com](https://faces.oddpaq.com)**.

The spectator scrolls an endless Pinterest-style wall of faces — mostly strangers, with
recognisable celebrities sprinkled in. When the phone locks (or the tab goes inactive),
every celebrity in the landed view silently becomes a stranger **except one large, central
tile, which becomes the forced person**. Off-screen celebrities stay put, so scrolling away
still looks like a normal wall — they could have stopped on anyone.

The whole app is one file (`index.html`) hosted on GitHub Pages. No backend, no build step.

## Choosing the force

There are two ways to set who gets forced.

### 1. The classic three-way tap force

The welcome overlay ("Tap anywhere to begin") is secretly split into three full-width
horizontal bands. Where the screen is tapped sets the force:

| Tap zone | Forced person |
|---|---|
| Top third | Leonardo DiCaprio |
| Middle third | Barack Obama |
| Bottom third | Scarlett Johansson |

Nothing on screen hints at the bands. Either you tap to begin yourself before handing the
phone over, or you direct where the spectator taps ("tap the card to begin" → middle).

### 2. Name anyone in the URL

Append `?=` plus a name to the URL and the app fetches that person's **Wikipedia lead
photo** and uses it as the force, no matter where the welcome screen is tapped:

```
faces.oddpaq.com/?=dualipa
faces.oddpaq.com/?=dua-lipa
faces.oddpaq.com/?=pedro+pascal
```

Spaces, dashes, underscores, `+`, or no separator at all — all fine. The lookup is a fuzzy
Wikipedia search, so `dualipa` finds Dua Lipa and minor typos usually resolve. Anyone with
a Wikipedia article that has a photo works: actors, musicians, athletes, politicians,
historical figures.

You can also force a **specific photo** by passing a direct image URL:

```
faces.oddpaq.com/?=https://example.com/photo.jpg
```

**The parameter is scrubbed from the address bar immediately** — by the time anyone looks,
the URL reads plain `faces.oddpaq.com`.

#### The performer signal

The welcome card tells you whether the named photo is loaded, via one word:

> How many of these **people** do you recognise?

- **"people"** — photo found and fully downloaded. The force is armed. Hand the phone over.
- **"faces"** — not ready: still loading (give it a second) or the lookup failed (no
  such article, article has no photo, or no signal). If it stays on "faces", nothing
  breaks — the app silently falls back to the classic three-way tap force above.

A spectator sees nothing unusual either way; both sentences read naturally.

## Performing it

1. Ask the spectator to name (or think of — see below) a celebrity.
2. Out of their view, open `faces.oddpaq.com/?=theirname`.
3. Wait for the welcome card to say "**people**" (typically under a second).
4. Hand the phone over: "It's a face-recognition test — tap to begin, and just keep
   scrolling. Tell me roughly how many you recognise."
5. Let them scroll a good way (the trick arms after ~1.2 screen-heights of scrolling).
6. Have them lock the phone / put it face-down / switch away: "OK, lock it for a second
   so you're not tempted to keep counting."
7. Build the reveal however you like. When the screen comes back on, the view they
   stopped on now contains exactly one celebrity — theirs — big and central.

Tips:

- **Don't skip step 3.** "people" is your confirmation the exact photo is on the phone;
  after that, the reveal works even with no signal.
- The swap fires on phone lock, switching apps, or (desktop) clicking another window or
  switching tabs — whichever comes first after the big scroll.
- The force stays put afterwards. Scrolling up and down still shows other celebrities
  elsewhere on the wall, so it looks like they could have landed anywhere. The forced
  person never appears twice.
- Reloading the page resets everything — you get one reveal per load.
- Wikipedia's lead photo is usually a clean, recognisable headshot, but you don't choose
  it. If a particular photo matters, use the direct-image-URL form instead.

## How the URL force works (technical)

- `?=name` is parsed from the query string, then removed with `history.replaceState`
  (no reload, no history entry).
- One CORS request to the MediaWiki API (`generator=search` + `prop=pageimages`) resolves
  the fuzzy name to an article and returns its lead image at 900px. No API key needed.
- The image is fully preloaded — and its true aspect ratio recorded, so its tile is sized
  to show the photo uncropped — before the card's wording flips to "people".
- Lookup and download are capped by timeouts (8s + 10s); any failure quietly leaves the
  three-way tap force in charge.

## Development

Everything lives in `index.html`. Serve it locally with any static server:

```
python3 -m http.server 8000
```

then open `http://localhost:8000/?=dualipa`. A manual test hook is exposed at
`window.__faceTest` (`state()`, `arm(url)`, `doSwap()`) for driving the swap
deterministically from the console.
