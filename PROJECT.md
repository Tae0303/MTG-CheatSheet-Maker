# Deck Keyword Booklet: project summary and context

## What it is

Deck Keyword Booklet is a single web page: paste a Magic: The Gathering decklist, and it builds a printable cheat sheet of every keyword, ability word, token and counter in the deck, each with a short explanation. Every printed panel is exactly standard card size, **63 × 88 mm**, and styled like a Magic card.

- **Repository:** [`Tae0303/MTG-CheatSheet-Maker`](https://github.com/Tae0303/MTG-CheatSheet-Maker) (public) on GitHub
- **Hosting:** GitHub Pages, from the `main` branch root
- **Live page:** https://tae0303.github.io/MTG-CheatSheet-Maker/
- **Local folder:** `Desktop/MTGTools/MTG card note/MTG_Keywords_Cheat_Sheet/`
- **Owner:** not a programmer; updates the site with GitHub's web uploader, so explain steps in plain language.
- **Use:** open it on a computer to print (it isn't designed for phones).

## Features

- **Decklist input:** reads exports from Moxfield, Archidekt, MTG Arena and MTGO. Lines under a `Commander` heading are commanders. Opens with the Paradox Power (Doctor Who) list as a sample.
- **Four layouts, all 63 × 88 mm per panel (the first three on A4 or US Letter):**
  - **Cards:** separate cards, 3 × 3 per page, with grey cut lines.
  - **Accordion:** strips of 4 panels with striped glue tabs and fold ticks. Print one-sided, glue strips together, fold like a fan.
  - **Mini booklet:** saddle-stitch page order. Print double-sided (flip on long edge), fold, nest the sheets and staple. Blank Notes pages pad the count to a multiple of 4.
  - **Image:** all cards in one high-resolution PNG (400 dpi) to view and zoom on a phone, tablet or computer. The Print button becomes **Save image**, and the Print sheets tab becomes **Full image**.
- **Magic-card styling:**
  - black border; frame color from the commanders' colors (gold for multicolor)
  - name bar with real mana symbols; parchment text box
  - explanations in italic brackets, like reminder text
  - a small collector line at the bottom
- **Cover card:** the **first** commander's artwork (front face for two-sided cards), with "Illus. <artist>" in the collector line. If there's no art, it shows a colored design with the deck name.
- **Order:**
  - **By group**: every group starts a new card, and a long group continues on the next card with the same title.
    - Default groups: Deck Mechanics, Commander & Multiplayer, Casting, Combat, Creatures & Permanents, Library & Graveyard, Tokens, Counters.
    - Custom groups: rename, reorder, delete, add your own, move any keyword into any group, and **Reset to default groups**.
  - **A–Z**: one alphabetical run titled by letter range ("Keywords A–F"). Tokens and Counters still get their own cards.
- **Keyword list (side panel):** untick a keyword to leave it out, or edit its text.
- **Other options:** text size slider; option to list the cards that use each keyword; deck name.
- **Two views:**
  - **Folded panels:** each card on its own, in reading order.
  - **Print sheets:** exactly what prints.
  - **Print** always prints the sheets (in Image layout it's **Save image** instead).
- **Last panel:** a "How to Read" card explaining the ×N count tags, symbols and ability words.

## Files

| File | Purpose |
|---|---|
| `index.html` | The whole tool: HTML, CSS and JavaScript inline |
| `README.md` | Plain-language description for GitHub visitors |
| `CLAUDE.md` | Context for Claude Code (same rules as below) |
| `PROJECT.md` | This file: project summary and context |

## How it works

1. `parseDeck()` reads the list, skipping set codes, foil marks, tags and sideboard/maybeboard sections.
2. `fetchCards()` gets the cards from Scryfall. `slim()` keeps only the needed fields, cached in localStorage.
3. `analyze()` finds keywords:
   - The glossary `G` (built with `k(name, cat, text, opts)`) is matched against card text, with the card's own name replaced by `~`.
   - Options: `act: true` allows verb endings (fights, scries); `re` is a custom pattern (token names are case-sensitive); `type` matches the type line (Saga, Adventure, Class, Room, Case, Battle).
   - Italic labels before " — " that are in `AW` (ability words, such as Paradox or Landfall) get their own entries. Other labels (flavor words such as Spoilers or Bear Witness) merge into one "Flavor words" entry.
   - Keywords Scryfall reports that aren't in the glossary use the card's own reminder text in parentheses.
4. `flow()` lays entries onto panels and checks each one for overflow.
5. `impose()` places panels on paper for the chosen layout and sets the paper size.

## Technical decisions and constraints

- **Static file only.** No build step or dependencies.
- **It can't be a claude.ai Artifact.** Artifact pages block the Scryfall API and images, and their Print button does nothing.
- **Text must never overflow a panel.** `flow()` measures each panel in a hidden `#measure` box and starts a new panel when the text box overflows (`scrollHeight > clientHeight`).
  - After any change to text, styles or fonts, check every panel in all four layouts on both A4 and US Letter.
  - A past check measured the wrong element and missed real overflow.
- **Printing:** the `@page` size is rewritten in `#page-style` with `margin: 0`, and each sheet is the full paper size. The owner must print at **100% / Actual size**.
- **Scryfall:**
  - Plain `GET` requests only: `/cards/search` with `!"name" or …` batches of 30, then `/cards/named?fuzzy=`. The earlier `POST /cards/collection` failed for the owner with "Failed to fetch".
  - Look up `A // B` cards by their front-face name.
  - Mana and tap symbols are `svgs.scryfall.io` images, with drawn circles as an offline fallback.
- **Glossary and ability-word text** are short paraphrases written for this tool. Keep new entries original and about one line long, rather than copying Wizards' reminder text.
- **Removed at the owner's request** (keep them out):
  - the keywords/cards count box ("44/100") on the cover
  - the "63×88" label
  - the "Keywords from this deck only" and "Paradox Power" footer labels
- **Saved in the browser (localStorage):**
  - `kb-cache-v2`: card data. Bump the version when `slim()` gains fields.
  - `kb-edits-v1`: edited keyword text.
  - `kb-hidden-v1`: unticked keywords.
  - `kb-prefs-v1`: layout, paper, order, text size, card names option, deck name.
  - `kb-deck-v1`: the pasted decklist.
  - `kb-groups-v1`: custom groups.

## Testing

In the `MTG_Keywords_Cheat_Sheet` folder, run `python -m http.server 8000` and open `http://localhost:8000`. The Paradox Power sample loads automatically, which needs internet.

Last check: 47 keywords found in the 100-card sample. All three layouts and both paper sizes came out at the right size with every panel 63 × 88 mm, no text overflowed, and the commander art and mana symbols loaded. Custom groups were tested by adding, moving, renaming, deleting and resetting groups. Nothing has been test-printed on paper yet. On 2026-10-19 the Image layout was added and tested in Chrome: the 12-card sample saved as a 4300 × 4442 px PNG (about 12 MB) with sharp text, real mana symbols and the cover art, and no panel overflowed in any layout or paper size.

## Status and next steps

- [x] Create the `MTG-CheatSheet-Maker` repository and upload everything in the folder (2026-10-19).
- [x] Turn on GitHub Pages (branch `main`, folder `/ (root)`).
- [ ] Test-print one sheet at 100% scale and measure a card (should be 63 × 88 mm).
- [ ] Optional idea: rename the "Folded panels" / "Print sheets" tabs to "Preview" / "Print layout".

## Related files (not part of this repository)

In `Desktop/MTGTools/MTG card note/` (one level up from this folder):
- `paradox-power-cheat-cards.html` is the first hand-made version of the cheat cards for one deck.
- `keyword-cheat-cards.html` is an older, separate version of this tool.

## Credits

Card data, art and mana symbols come from [Scryfall](https://scryfall.com). Not affiliated with or endorsed by Wizards of the Coast.
