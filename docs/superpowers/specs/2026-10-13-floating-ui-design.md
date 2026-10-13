# Floating UI, selection toolbar and welcome screen

Date: 2026-10-13. Status: approved in chat, awaiting spec review.

## Goal

Make Explainer feel calmer and closer to Excalidraw's layout without losing anything it does
today. The header bar goes, so the board fills the window. Controls float in small "islands" at
the edges. Changing an existing line no longer needs keyboard shortcuts. An empty board greets you.

Inspiration: Excalidraw's layout (mock A), the logo, tagline and list from mock B, and the
selection toolbar from mock C. The user rejected Excalidraw's solid purple Share button.

## Non-goals

- No drawing tools to pick: every action is still a gesture.
- No new shapes, colours or text styles.
- No change to share-link compatibility: old links must still load.
- Keyboard shortcuts stay as they are.

## 1. Layout: floating islands

The `<header>` is removed and `#viewport` fills the window. An island is a small floating
panel: `var(--box)` background, a 1px `var(--edge)` border, 10px radius, and a soft shadow that
matches the existing panels. It is positioned with `position:fixed` and safe-area insets.

| Place | Contents |
|---|---|
| Top-left | ☰ menu button, the mark (two boxes and an arrow), the "Explainer" wordmark in `--f-display`, and the "Example board" tag while it applies |
| Top-centre | Style for **new** lines: Angled or Curved, then arrow, plain line, or arrows both ways |
| Top-right | GitHub icon link, then Share |
| Bottom-left | − · zoom % (click resets to 100%) · + · Fit board · Undo · Redo · ← Edits → |
| Bottom-right | **?** button. The shortcuts panel opens above it, as today |

- **Phone width (≤ 640px):** the top-centre island moves to the bottom centre. The bottom-left
  island keeps zoom and undo/redo only, and Fit board and Edits move into the ☰ menu. Nothing
  may overlap, and there is no horizontal scroll at 375px.
- **Toast:** moves up so it never covers a bottom island (above the bottom islands' height).
- **Board start:** the board's visible area starts under the top islands. Fit board and the
  example-board fit (D-053) leave room for the islands.

## 2. Share button

It is quiet, not a solid accent block. At rest it has the island surface, a 1px `var(--edge)`
border, `var(--ink)` text and an accent-coloured share icon. On hover its background becomes
`var(--accent-soft)` and its border `var(--accent)`. The Share menu (whole board or selection)
behaves as today and opens below the button.

## 3. ☰ menu

A dropdown under the ☰ button. It closes on Esc, on a click outside, or after choosing an action.
Settings rows show their current value and stay open when clicked, so you can cycle through them.

- **Theme:** Auto, Light, Dark (cycles; T still works).
- **Capitals:** All caps, Sentence case, As typed (cycles).
- **Snap to grid:** a switch (G still works).
- A divider, then:
- **Open the example board:** replaces the board with the three-box example, as one undo step.
- **Clear board:** as today, undoable.
- A divider, then:
- **Shortcuts:** opens the shortcuts panel.
- **Source on GitHub:** opens the repo in a new tab.
- On phones it also holds Fit board and ← Edits →.

## 4. Selection toolbar

A small island floating just above the selection. It is centred on the selection's bounds and
flips below the selection if there is no room above. It is hidden while dragging, drawing a
line, or editing text, and appears again after.

- **When one or more lines are selected:** Angled or Curved · arrow / plain line / both ways ·
  Reverse direction · then Copy · Delete.
- **When only boxes are selected:** Copy · Delete.

Each change applies to every selected line and is one undo step. "Reverse direction" swaps
`from`/`to` and `fa`/`ta`, so a one-way arrow points the other way; it is only shown when a
selected line has an arrow at one end only. The top-centre island no longer changes selected
lines: it sets new lines only. This changes D-033's "Style of selected + new lines".

## 5. Welcome on an empty board

A first visit still opens the three-box example board (D-053).

When the board has **no boxes** (for example after Clear board), the centre of the window shows:

- the mark and the "Explainer" wordmark (as in `docs/logo-*.png`);
- the tagline "Boxes, lines, and nothing in the way." in a handwritten font (Caveat, loaded
  from Google Fonts with the existing fonts);
- two hints: "Double-click anywhere to add a box" and "Select a box, ⌘-click another to connect"
  (Ctrl off Mac);
- three items: Open the example board · Shortcuts (?) · Source on GitHub;
- handwritten hints with small drawn arrows pointing to ☰ ("theme, capitals, grid…") and to **?**
  ("shortcuts and help"). These are hidden on phone widths.

The welcome layer does not catch pointer events except on its three items, so double-click and
drag still create a box through it. It disappears as soon as a box exists and returns when the
last box is deleted. It uses theme tokens only.

## 6. Data and settings

- New setting: the arrowheads for new lines, `newHeads` = `end` | `none` | `both` (default `end`),
  stored in localStorage (`explainer.newheads`). It travels in share-link settings as key `a`
  (0 none, 1 end, 2 both). Absent means `end`, so old links load unchanged.
- Drawing: with no modifier the new line gets `newHeads`. Shift gives a plain line and
  Shift+Ctrl arrows at both ends (D-052), which override it. Double-clicking a dot first (D-011)
  still means no arrow.
- Cmd/Ctrl+click (D-051) also uses `newHeads` unless its modifiers say otherwise.
- The line-style setting (`lineStyle`) keeps its existing storage and share key `l`.
- No change to the board format (`pack`/`unpack`). Reversing only swaps fields that exist.

## 7. Code shape (index.html)

- Markup: replace `<header>` with the islands (`#islTL`, `#islTC`, `#islTR`, `#islBL`), a
  `#menu` dropdown, a `#selbar` toolbar and a `#welcome` layer. The existing button IDs
  (`themeBtn`, `capBtn`, `snapBtn`, `shareBtn`, `undoBtn`, `redoBtn`, `zoomBtn`, `homeBtn`,
  `clearBtn`, `placeBack`, `placeFwd`, `lsAngled`, `lsCurved`, `exampleTag`) are kept where the
  control survives, so the handlers keep working. Theme, capitals and snap move into menu rows
  with the same IDs.
- `renderSel()` also positions and fills `#selbar`; `renderAll()` toggles `#welcome`.
- Gestures: the viewport `pointerdown` ignores presses on islands, the menu, the toolbar and
  the welcome items (as it ignores `header` today).
- Every new colour is a token. Dark mode comes from the tokens.

## 8. Decisions

New entries: D-057 (floating islands; amends D-055, whose link moves from the header corner to
the top-right island, and the decisions that place controls in the toolbar), D-058 (quiet Share), D-059 (☰ menu), D-060 (selection toolbar and Reverse,
amending D-036), D-061 (welcome on an empty board), D-062 (arrowheads for new lines). Earlier
entries get their Status lines updated. The README is updated, with new screenshots
(`docs/screenshot-*.png`), and the shortcuts panel only where shortcuts change.

## 9. Testing (manual, in the browser)

- Every island control works.
- ☰ menu: open, cycle settings, actions, Esc and outside click.
- Selection toolbar: on one line, several lines, boxes only, and a mixed selection. Each
  action is one undo step. It is hidden while dragging or editing, and flips near the top edge.
- Welcome: shown after Clear board, a box is created through it, it is hidden once a box exists,
  and each of its items works.
- New-line arrowheads in all three settings, with the Shift and Shift+Ctrl overrides and
  Cmd/Ctrl+click.
- The usual checks: draw a box and type, connect two boxes, undo/redo, refresh (the board
  survives), dark mode.
- Widths: 1440px, 800px and 375px, with no overlaps and no horizontal scroll.
- A share link from before this change still opens.

## 10. Release

When development is done and verified: push `main`, check the GitHub Pages deploy, and publish
release v1.1.0 with notes. It also includes copy/cut/paste, the paste safeguard, the GitHub link
and the capitals modes.
