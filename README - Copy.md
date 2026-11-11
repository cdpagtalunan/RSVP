# Christian & Ijane — Wedding Invitation

A single-page wedding invitation site for **January 19, 2027**. It uses pure HTML, CSS, and vanilla JavaScript, with no frameworks and no build step, and is ready for GitHub Pages.

## Project structure

```
index.html     ← everything (HTML + inline CSS + inline JS)
images/        ← put your photos here
music/         ← optional background song
.nojekyll      ← tells GitHub Pages to serve files as-is
```

## Personalize: search `TODO` in index.html

| What | Where |
|---|---|
| Hero background photo | `<!-- TODO: Insert Hero Background Image here -->` inside `.hero__media` |
| Story / venue / gallery photos | `<!-- TODO: Insert Photo Here -->`. Replace the `<span class="photo__placeholder">` with `<img src="images/..." alt="..." loading="lazy">` |
| RSVP Google Form | `<!-- TODO: Replace with your actual Google Form Embed URL -->` |
| Ceremony / reception times, schedule | Venue cards + `.timeline` list; also `CONFIG.weddingDate` and `CONFIG.calendar` in the script |
| Dress code, RSVP deadline, hashtag, story text | Marked with `TODO` comments |
| Background music (optional) | Add `music/our-song.mp3` and uncomment the `<source>` in `<audio id="bg-music">` |

### Getting the Google Form embed URL
1. Open your form → **Send** → the **`< >`** tab.
2. Copy only the `src="…"` value (ends in `viewform?embedded=true`).
3. Paste it into the iframe's `src` in the RSVP section.

Until you do, the page shows a styled "RSVP form coming soon" placeholder instead of a broken embed.

### Photo tips
- Hero: landscape, ~2000px wide, compressed to under 500 KB.
- Others: ~1200px on the long side. Export as `.jpg` or `.webp`.

## Preview locally
Open `index.html` directly in a browser, or run a tiny server:
```
python -m http.server 8000
```
The envelope is skipped on reloads within the same tab. Open a new tab to see it again.

## Deploy to GitHub Pages
1. Create a repo and push these files to the `main` branch.
2. Go to **Settings → Pages → Build and deployment**. Set Source to **Deploy from a branch**, Branch to **main / (root)**.
3. Your site will be live at `https://<username>.github.io/<repo>/` in a minute or two.
