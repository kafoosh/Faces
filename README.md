# Face Test

Face Test is a one-page magic app. It shows a wall of faces. The app is live at
**[faces.oddpaq.com](https://faces.oddpaq.com)**.

The spectator scrolls a Pinterest-style wall of faces that has no end. Most faces are
strangers. Some faces are celebrities. When the phone locks (or the browser tab becomes
inactive), the app changes each celebrity in the visible view to a stranger. Only one
large, central tile is different: **that tile changes to the forced person**. Celebrities
that are not in the visible view do not change. Thus, if the spectator scrolls away, the
wall looks normal. It looks as if the spectator could stop on any face.

The app is one file (`index.html`) on GitHub Pages. The app has no backend and no build
step.

Note: "Force" is a magic term. The force is the item that the performer makes the
spectator get. The "forced person" is the celebrity that the trick shows at the end (the
"reveal").

## How to set the force

There are two methods to set the forced person.

### 1. The three-zone tap force

The welcome overlay ("Tap anywhere to begin") has three horizontal zones. Each zone has
the full width of the screen. The zones are not visible. The zone that gets the tap sets
the force:

| Tap zone | Forced person |
|---|---|
| Top third | Leonardo DiCaprio |
| Middle third | Barack Obama |
| Bottom third | Scarlett Johansson |

The screen shows no marks for the zones. You have two options:

- Tap the screen before you give the phone to the spectator.
- Tell the spectator where to tap. Example: "Tap the card to begin" points to the middle
  zone.

### 2. Set a name in the URL

Add `?=` and a name to the URL. The app then gets the **lead photo of the Wikipedia
article** for that person. The app uses that photo as the force. The tap position on the
welcome screen then has no effect.

```
faces.oddpaq.com/?=dualipa
faces.oddpaq.com/?=dua-lipa
faces.oddpaq.com/?=pedro+pascal
```

You can use spaces, dashes, underscores, dots, `+` signs, or no separator. The search
accepts small errors: names with no separator (`tomholland`, `dualipa`) and small
spelling errors usually give the correct result. If two persons have the same name, the
app selects the most probable person (`tomholland` gives Tom Holland the actor, not the
disambiguation page). The app can find each person who has a Wikipedia article with a
photo: actors, musicians, athletes, politicians, and historical persons. Names with
separators (`tom-holland`) are the most reliable. Use them if a name with no separator
does not give a result.

You can also force a **specified photo**. For this, use a direct image URL:

```
faces.oddpaq.com/?=https://example.com/photo.jpg
```

**The app immediately removes the parameter from the address bar.** When a person
examines the URL, the URL shows only `faces.oddpaq.com`.

#### The performer signal

The welcome card shows you if the photo is loaded. Look at the **circle behind the small
face icon** at the top of the card:

- **The grey circle is visible** (the usual condition). The app found the photo and
  downloaded it fully. The force is armed. You can give the phone to the spectator.
- **The circle is white or not visible** (only the icon shows on the card). The app is
  not ready. Possible causes: the download continues (wait one or two seconds), or the
  search failed (there is no article, the article has no photo, or there is no network
  signal). If the circle stays white, the app continues to operate correctly. The app
  then uses the three-zone tap force.

The spectator does not see a difference in the two conditions. An icon with no circle
looks like a part of the design.

## How to do the trick

1. Ask the spectator for the name of a celebrity. (The spectator can also only think of
   one. Refer to the tips below.)
2. Open `faces.oddpaq.com/?=theirname`. Make sure that the spectator cannot see the
   screen.
3. Wait until the grey circle shows behind the face icon on the welcome card. This
   usually takes one to two seconds.
4. Give the phone to the spectator. Say: "This is a face-recognition test. Tap to begin,
   then scroll. Tell me approximately how many faces you know."
5. Let the spectator scroll a sufficient distance. The trick becomes armed after
   approximately 1.2 screen-heights of scroll.
6. Tell the spectator to lock the phone, to put it face-down, or to go to a different
   app. Example: "Lock the phone for a moment, so you do not continue to count."
7. Do the reveal in the way that you prefer. When the screen comes on again, the view
   where the spectator stopped has only one celebrity: the forced person, large and
   central.

Tips:

- **Do not omit step 3.** The grey circle shows that the correct photo is on the phone.
  After that point, the reveal operates with no network signal.
- The swap occurs at the first of these events after the long scroll: the phone locks,
  the spectator goes to a different app, or (on a desktop) the spectator clicks a
  different window or tab.
- The force stays in its position after the swap. When the spectator scrolls up and down,
  other celebrities show in other areas of the wall. Thus it looks as if the spectator
  could stop on any face. The forced person does not show two times.
- When you load the page again, the app resets. Each page load gives one reveal.
- The lead photo from Wikipedia is usually a clear photo of the face, but you cannot
  select it. If you must have a specified photo, use the direct image URL.

## How the URL force operates (technical)

- The app reads `?=name` from the query string. The app then removes it with
  `history.replaceState` (no reload, no history entry).
- The app gets candidate articles from two MediaWiki API functions (no key, CORS with
  `origin=*`). The first function is the completion suggester (`action=opensearch`). This
  is the search-box dropdown. It accepts errors, ranks by popularity, and is good for
  names with no separator. The second function is the full-text search. If the full-text
  search gives a "did you mean" suggestion, the app sends the query again with that
  suggestion.
- The app then ranks the candidates on the client in one batched
  `pageimages`+`pageprops` request. The app removes disambiguation pages and articles
  with no photo. A title that is equal to the typed name, letter for letter, wins
  immediately (the comparison ignores spaces, accents, and suffixes such as "(actor)").
  If no title wins, the popularity order decides. The app gets the lead image of the
  winner at a width of 900 px.
- The app downloads the full image and records its true aspect ratio before the circle
  becomes grey again. The tile for the image then has the correct size, and the photo
  shows with no crop.
- Each request has a time limit. If a request fails, the app uses the three-zone tap
  force. The app shows no error.

## Development

All the code is in `index.html`. To start the app on your computer, use a static server:

```
python3 -m http.server 8000
```

Then open `http://localhost:8000/?=dualipa`. For manual tests, the console object
`window.__faceTest` has these functions: `state()`, `arm(url)`, and `doSwap()`. Use them
to do the swap in a controlled way.
