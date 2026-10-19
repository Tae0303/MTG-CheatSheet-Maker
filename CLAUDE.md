# Deck Keyword Booklet

Single web page (`index.html`): paste a Magic: The Gathering decklist, and it builds a printable cheat sheet of every keyword, ability word, token and counter in the deck. Every printed panel is a standard card, **63 × 88 mm**, styled like a Magic card. Hosted on GitHub Pages from the `main` branch root. The owner is not a programmer and updates the site through the GitHub web uploader.

## How it works

1. `parseDeck()` reads Moxfield, Archidekt, Arena and MTGO exports. Lines under a `Commander` heading are commanders.
2. `fetchCards()` gets the cards from Scryfall. `slim()` keeps only the fields the page uses, cached in localStorage.
3. `analyze()` finds keywords:
   - The glossary `G` (made with `k(name, cat, text, opts)`) is matched against oracle text, with the card's own name replaced by `~`.
   - Options: `act: true` allows verb endings (fights, scries); `re` is a custom pattern (token names are case-sensitive); `type` matches the type line (Saga, Adventure, Class…).
   - Italic labels before " — " that are in `AW` (ability words) get their own entries. Other labels are merged into one "Flavor words" entry.
   - Scryfall keywords missing from the glossary fall back to the card's own reminder text in parentheses.
4. `flow()` lays entries onto panels. `impose()` places panels on paper for the chosen layout.

## Constraints

- **Static file only.** All CSS and JS are inline, with no build step.
- **Not a claude.ai Artifact.** Its CSP blocks the Scryfall API and images, and `window.print()` does nothing there.
- **Panels never overflow.** `flow()` measures each panel in the hidden `#measure` box and starts a new panel when `.textbox` overflows (`scrollHeight > clientHeight`).
  - After any change to text, CSS or fonts, check every panel in all four layouts (Cards, Accordion, Mini booklet, Image) on both A4 and US Letter.
  - An earlier check measured a flex child without `min-height: 0` and missed real overflow.
- **Grouping:**
  - Each group starts on a new card, and a long group continues on the next card under the same title.
  - In **A–Z** order, keywords form one alphabetical run titled by letter range ("Keywords A–F"); Tokens and Counters still get their own cards.
  - Custom groups (rename, reorder, delete, add, move a keyword) exist only in **By group** order, stored in `kb-groups-v1`. Reset restores `CATS`.
- **Layouts:** `@page` size is rewritten in `#page-style` with `margin: 0`, and each `.sheet` is the full paper size.
  - **Cards:** 3 × 3 per page, cut apart.
  - **Accordion:** strips of 4 panels with glue tabs, printed one-sided.
  - **Mini booklet:** saddle-stitch page order, printed double-sided with flip on long edge. The page count is padded to a multiple of 4 with Notes pages.
  - **Image:** every panel in one near-square grid (`.imgsheet`, paper is ignored). The Print button becomes **Save image**: `saveImage()` draws the sheet through an SVG `<foreignObject>` onto a canvas at `IMG_DPI` (400), stepping down to 300/200/150 if the canvas is too big, and downloads a PNG. Fonts (Google Fonts CSS, latin subsets), art and mana symbols are inlined as data URLs first, fetched with `cache: 'no-store'` because the copies cached by `<img>` lack CORS headers.
- **Scryfall:**
  - Plain `GET` requests only (`/cards/search` with `!"name" or …` batches of 30, then `/cards/named?fuzzy=`). The earlier `POST /cards/collection` failed for the owner with "Failed to fetch".
  - Look up `A // B` cards by their front-face name.
  - Mana and tap symbols are `svgs.scryfall.io` SVGs, with a drawn-circle fallback offline.
- **Glossary and ability-word text** are short paraphrases written for this tool. Keep new entries original and about one line long, rather than copying Wizards' reminder text.
- **Look:**
  - The frame color comes from the commanders' color identity (gold for multicolor).
  - The cover shows the **first** commander's `art_crop`, with "Illus. <artist>" in the collector line.
  - The owner asked for these to be removed; don't add them back: the keywords/cards count box on the cover, the "63×88" label, and the deck-name footer labels.
- **localStorage keys:**
  - `kb-cache-v2` (card data; bump the version when `slim()` gains fields)
  - `kb-edits-v1` (edited keyword text)
  - `kb-hidden-v1` (unticked keywords)
  - `kb-prefs-v1` (settings)
  - `kb-deck-v1` (pasted decklist)
  - `kb-groups-v1` (custom groups)

## Running locally

Run `python -m http.server 8000` in this folder and open `http://localhost:8000`. On first load it uses the Paradox Power sample list (needs internet).
