# Decisions

The product and design decisions for Explainer, in the order they were made.
This file is the source of truth for how the app should behave. Code that disagrees with an
Active decision is a bug. Read it before changing behaviour, and update it in the same change.

## Rules for keeping this file

- **IDs.** Every decision gets the next ID in the form `D-NNN` (zero-padded, e.g. `D-039`).
  IDs are permanent: never renumber, reuse or delete one. Find the next ID with
  `grep -o '^### D-[0-9]*' DECISIONS.md | tail -1`.
- **Append only.** Add new decisions at the bottom. Do not rewrite history.
- **Status** is one of:
  - `Active`: in force.
  - `Superseded by D-NNN`: fully replaced. Keep the text; the newer decision wins.
  - `Amended by D-NNN`: still in force except for the part the newer decision changes.
- **Overrides.** When a new decision replaces or changes an older one, the new entry says
  `Supersedes: D-NNN` (or `Amends: D-NNN`), and the old entry's Status line is updated to point
  forward. Both edits happen in the same commit.
- **One decision per entry.** Keep each entry short: what was decided, why (the user's request,
  in brief), and where it lives in the code.
- Mention the decision IDs in the commit message, e.g. `Add share menu (D-037, supersedes D-029 part)`.

## Entry template

```
### D-NNN: Short title
- Status: Active
- Date: YYYY-MM-DD
- Supersedes / Amends: D-NNN (omit if none)
- Decision: What the app does, stated as behaviour.
- Why: The request or reason behind it.
- Code: Functions, CSS classes or files involved.
```

---

### D-001: Name the product "Explainer"
- Status: Active
- Date: 2026-10-13
- Decision: The app is called Explainer. Domain is not chosen yet; unregistered candidates when
  checked were explain.how, explainthis.how, explainthis.page, explainer.how, boxsplain.com,
  whyboard.app, explainerboard.com.
- Why: User's chosen name.
- Code: `<title>`, header `.mark`.

### D-002: One self-contained HTML file, no build step
- Status: Active
- Date: 2026-10-13
- Decision: The whole app is `index.html` with inline CSS and JS. No frameworks or build tools.
  The only external resource is Google Fonts (Atkinson Hyperlegible for text, Bricolage Grotesque
  for the logo), with system-font fallbacks.
- Why: Very simple tool; easy to host on any static host or open from disk.
- Code: `index.html`.

### D-003: Purpose and scope: very simple, for explaining things
- Status: Active
- Date: 2026-10-13
- Decision: Explainer is a deliberately minimal Draw.io-like tool: boxes with text and lines
  between them. New features should keep the interaction this simple.
- Why: User's brief.

### D-004: Drag on empty canvas draws a box and starts typing
- Status: Amended by D-020
- Date: 2026-10-13
- Decision: Dragging on empty canvas draws a box (dashed preview). On release the box is created
  and immediately enters text editing. Very small drags (under 14px on screen) do nothing.
- Why: "If I drag somewhere I create the box, and immediately … enter some text."
- Code: `downCanvas`.

### D-005: Centred, wrapping text; the box grows by 50% when full
- Status: Active
- Date: 2026-10-13
- Decision: Box text is centred horizontally and vertically and word-wraps. When the text no
  longer fits, the box grows to 1.5x its width and height, around its centre, repeating as needed
  while you type. Boxes never shrink automatically.
- Why: "It should become bigger automatically, with 0.5x the size … and keep growing."
- Code: `fit`, `GROW`.

### D-006: Click inside a box edits its text
- Status: Superseded by D-019
- Date: 2026-10-13
- Decision: A click anywhere inside a box starts editing with the caret at the click point.

### D-007: Double-click a box selects all its text
- Status: Amended by D-026
- Date: 2026-10-13
- Decision: Double-clicking a box enters editing and selects all its text.
- Why: "If you double click it, it should select all the text."
- Code: `selectAllText`, viewport `dblclick` handler.

### D-008: Eight connection dots per box
- Status: Active
- Date: 2026-10-13
- Decision: Each box has eight anchors: the four corners and the midpoints of the four sides.
- Why: User's spec.
- Code: `A` (anchor table), `.h` elements, `anchorPt`.

### D-009: Dots appear on hover of the box
- Status: Superseded by D-024
- Date: 2026-10-13
- Decision: Hovering or selecting a box showed all eight dots.

### D-010: Drag a dot onto another box to connect with an arrow
- Status: Active
- Date: 2026-10-13
- Decision: Dragging from a dot and releasing over another box creates a line ending in an arrow
  at that box. The end snaps to the target's dot nearest the pointer, which is highlighted while
  dragging. Releasing over empty space creates nothing.
- Why: User's spec.
- Code: `downHandle`.

### D-011: Double-click a dot for a line without an arrow
- Status: Active
- Date: 2026-10-13
- Decision: Double-clicking exactly on a dot arms it (filled dot); the next line dragged from it
  has no arrowhead. Double-click-then-drag also works. Double-clicking again disarms.
- Why: "Double click on the circle and then connect … there should not be an arrow."
- Code: `armed`, `lastHandle`, `downHandle(…, exact)`.

### D-012: Drag a box from anywhere on it to move it; lines follow
- Status: Amended by D-019, D-033
- Date: 2026-10-13
- Decision: Pressing and dragging anywhere on a box moves it; connected lines re-route live.
- Why: User's spec.
- Code: `downBox`.

### D-013: Straight connector lines
- Status: Superseded by D-028
- Date: 2026-10-13
- Decision: Lines were straight segments between anchors.

### D-014: Basic keyboard: Delete, Enter, Esc
- Status: Active
- Date: 2026-10-13
- Decision: Delete/Backspace removes the selection (a deleted box takes its lines with it).
  Enter edits the single selected box. Esc stops editing, or clears the selection.
- Code: window `keydown` handler, `deleteSel`.

### D-015: Example board on first open; Clear board
- Status: Active
- Date: 2026-10-13
- Decision: A first-time visitor sees a small example board that explains the gestures, tagged
  "Example board". "Clear board" empties it and can be undone.
- Why: The first view should show what the tool does.
- Code: `sample`, `#clearBtn`, `#exampleTag`.

### D-016: Visual design
- Status: Active
- Date: 2026-10-13
- Decision: Cool grey dotted canvas (dots every 22px), white boxes with dark 1.5px borders and
  7px radius, one blue accent for selection, dots and previews. All colours are CSS tokens on
  `:root` with dark values (see D-040).
- Code: `:root` tokens in `<style>`.

### D-017: Infinite canvas with pan and zoom
- Status: Active
- Date: 2026-10-13
- Decision: The canvas has no edges. Pan with scroll/trackpad, Space+drag, or middle/right-drag.
  Ctrl/Cmd+scroll zooms around the pointer (20% to 300%); Ctrl/Cmd +/−/0 also zoom. While drawing,
  moving or connecting near the edge, the view auto-scrolls. Toolbar has "100%" and "Fit board".
  The view (pan and zoom) is remembered per browser.
- Why: "It should also support infinite scroll like Draw.io."
- Code: `view`, `applyView`, `zoomAt`, `autoPan`, `fitBoard`.

### D-018: Undo and redo cover everything, including typing
- Status: Active
- Date: 2026-10-13
- Decision: Every change is undoable. Typing is grouped into steps by pauses (700 ms). Undo/redo
  work while typing (Ctrl/Cmd+Z, Shift+Ctrl/Cmd+Z, Ctrl+Y, toolbar buttons that don't steal
  focus). After undo/redo the changed box is selected and flashes, and the view pans to it if it
  is off screen. History holds 300 steps.
- Why: "Undo should undo anything you have edited and jump to the previous thing you did."
- Code: `commit`, `restore`, `undo`, `redo`, `diff`, `TYPE_PAUSE`.

### D-019: Click the text to edit; click around it to select the box; Shift multi-select
- Status: Active
- Date: 2026-10-13
- Supersedes: D-006
- Amends: D-012
- Decision: Clicking on the text itself edits it. Clicking inside the box away from the text
  selects the box (blue tint). Shift+click adds or removes boxes and lines. Dragging any selected
  box moves the whole selection.
- Why: User asked for box selection separate from text editing, with Shift for multiple items.
- Code: `onText`, `downBox`, `.box.sel`.

### D-020: Drag-select: a drag that touches items selects them
- Status: Active
- Date: 2026-10-13
- Amends: D-004
- Decision: If the rectangle dragged on empty canvas touches any box or line, it becomes a
  selection (thin outline) instead of a new box. Shift+drag adds to the selection.
- Why: "It should not create the box. It should actually select them instead."
- Code: `downCanvas`, `inRect`, `.draft.marquee`.

### D-021: Ctrl/Cmd+A selects all; Backspace deletes it
- Status: Active
- Date: 2026-10-13
- Decision: Outside text editing, Ctrl/Cmd+A selects every box and line; Backspace/Delete then
  deletes them all (undoable).
- Code: keydown handler.

### D-022: Shortcuts panel in the bottom-right corner
- Status: Amended by D-037, D-038, D-047
- Date: 2026-10-13
- Decision: A small, translucent, minimisable panel lists the shortcuts. Minimised it becomes a
  "?" button. Modifier keys show as ⌘ on Mac and Ctrl elsewhere.
- Code: `#help`, `#helpBtn`, `setHelp`.

### D-023: Show the line being dragged
- Status: Active
- Date: 2026-10-13
- Decision: While dragging from a dot, a solid accent-coloured preview line (with arrowhead
  unless D-011 applies) follows the pointer and uses the same routing as a real line.
- Code: `#tmp` path in `downHandle`.

### D-024: Dots appear only near a dot, not for the whole selected box
- Status: Amended by D-031, D-033
- Supersedes: D-009
- Date: 2026-10-13
- Decision: A dot appears only when the pointer is within about 16px of it. On touch devices
  (no hover) a selected box shows its dots.
- Why: "I don't want it to show all the circles … only when hovering near the circle."
- Code: `nearDot`, `.h.near`.

### D-025: Backspace always deletes the selection
- Status: Active
- Date: 2026-10-13
- Decision: Every press on the board takes keyboard focus, so Backspace/Delete reaches the app
  even if something outside it had focus. Pressing near a dot inside a box counts as on the box
  unless it is right on the dot. A box being edited is styled differently from a selected box,
  so it is clear Backspace edits text there.
- Why: User reported Backspace not always deleting.
- Code: `focusBoard`, `onDot`, `.box.sel.editing`.

### D-026: Double-click anywhere in a box selects all its text
- Status: Active
- Date: 2026-10-13
- Amends: D-007
- Decision: Applies anywhere inside the box, including near its edges. Exception: double-click
  exactly on a dot (D-011). The pressed box is not re-inserted in the DOM if already on top,
  because that breaks native double-click detection.
- Code: `downBox` ("bring to front" guard), `dblclick` handler.

### D-027: Snap to grid, on by default
- Status: Active
- Date: 2026-10-13
- Decision: A "Snap to grid" switch (key G) snaps drawing, moving and auto-growing to the 22px
  dot grid. When moving several boxes, the grabbed box snaps and the others keep their relative
  positions. Turning it off does not move existing boxes.
- Why: User asked for a grid snap toggle, enabled by default.
- Code: `snap`, `sn`, `setSnap`, `G`.

### D-028: Lines route around boxes
- Status: Amended by D-036
- Supersedes: D-013
- Date: 2026-10-13
- Decision: A line leaves its dot pointing outward, travels with right-angle turns and rounded
  corners, avoids every nearby box (not only the two it connects), and enters the target dot
  head-on. Routing is a shortest-path search on a grid built from box edges, with a penalty per
  bend. A simpler route is the fallback when no path exists.
- Why: Lines were drawn over boxes; user asked for lines that go around.
- Code: `route`, `routeSimple`, `roundedPath`, `linkGeom`.

### D-029: Board saved in the URL, like Mermaid
- Status: Amended by D-041
- Date: 2026-10-13
- Decision: After every change the board is packed, deflate-compressed, base64url-encoded and put
  in the URL hash (`#x1.…`; `u1.` uncompressed fallback). Opening a URL with a board loads it,
  even in a fresh browser. The board is also kept in localStorage. The untouched example board is
  not written to the URL.
- Why: "Refresh should remember the state, and I can share it."
- Code: `pack`, `unpack`, `encodeBoard`, `decodeBoard`, `scheduleUrl`.

### D-030: Double-click empty canvas creates a default box
- Status: Amended by D-046
- Date: 2026-10-13
- Decision: Creates a box of 7×3 grid cells (154×66 px) centred on the pointer, snapped, in
  editing mode. Not triggered by double-clicking a line or with Shift held.
- Code: `newBoxAt`.

### D-031: A visible dot always wins a drag
- Status: Active
- Date: 2026-10-13
- Amends: D-024
- Decision: Whenever a dot is visible, press-and-drag starts a line from it, even if the pointer
  is slightly off the dot. A plain click there still selects the box.
- Code: viewport `pointerdown` (nearEl branch), `downHandle(…, exact=false)`.

### D-032: Hovering a line shows its nearer end dot
- Status: Active
- Date: 2026-10-13
- Decision: Hovering a line shows whichever of its two end dots is closer. Dragging from there
  starts a new line from that dot; clicking selects the line.
- Code: `nearDot` (`.hit` branch), `nearLink`.

### D-033: Near a box's outline the nearest dot always shows
- Status: Active
- Date: 2026-10-13
- Amends: D-012, D-024
- Decision: Within a 12px band on either side of a box's outline, the nearest of its eight dots
  always appears and a drag starts a line from it. Deeper inside the box, dragging moves the box.
- Code: `EDGE_ZONE`, `nearDot`.

### D-034: Auto-capitalise, on by default
- Status: Active
- Date: 2026-10-13
- Decision: A switch capitalises the first letter typed at the start of a box, of a new line, and
  after `.`, `!` or `?`. Also sets `autocapitalize` for on-screen keyboards.
- Code: `autoCap`, `setCap`, `beforeinput` handler.

### D-035: Jump between edit places
- Status: Active
- Date: 2026-10-13
- Decision: The toolbar "← Edits →" control and Alt+←/→ move between the places you last changed;
  each jump pans there, selects and flashes the box. The first "back" goes to the latest edit
  (or the one before it if you are already looking at it). Consecutive edits to the same box are
  one place. Up to 60 places, kept across refreshes, reset when a different shared board opens.
- Why: "Return to the last edited place and jump between edits."
- Code: `places`, `recordPlace`, `goPlace`.

### D-036: Line style: Angled or Curved, per line
- Status: Active
- Date: 2026-10-13
- Amends: D-028
- Decision: The "Angled | Curved" switch changes the selected lines, or with nothing selected sets
  the style for new lines. Curved draws one smooth arc when it is clear of other boxes, otherwise
  a smooth curve along the routed path. Each line stores its own style.
- Code: `lineStyle`, `setLineStyle`, `arcCurve`, link field `curve`.

### D-037: Shortcuts panel remembers its state; open on first visit
- Status: Amended by D-038
- Amends: D-022
- Date: 2026-10-13
- Decision: Open or minimised is remembered at every screen size. On a first visit it opens,
  except on phone-width screens (480px or less).
- Code: `setHelp`, `explainer.help`.

### D-038: Pin for the shortcuts panel, unpinned by default
- Status: Active
- Amends: D-022, D-037
- Date: 2026-10-13
- Decision: The panel has a pin button. Unpinned (default), any click outside the panel hides it.
  Pinned, it stays open. The pin state is remembered.
- Code: `helpPinned`, `setPin`, capture-phase `pointerdown` listener.

### D-039: Drags can never get stuck
- Status: Active
- Date: 2026-10-13
- Decision: Gestures use pointer capture; a pointer move with no button pressed ends the gesture;
  window blur, tab hide and Esc cancel it; a new press cancels any stale gesture; native text
  selection and drag-and-drop are blocked outside the box being edited. Cleanup always removes
  selection rectangles and preview lines.
- Why: User saw a selection rectangle stuck on screen.
- Code: `endGesture`, `cancelGesture`, `cleanupGesture`, `capId`.

### D-040: Light and dark mode
- Status: Active
- Date: 2026-10-13
- Decision: A theme button cycles Auto → Light → Dark (key T). Auto follows the system (or the
  host page's theme when embedded).
- Code: `theme`, `setTheme`, `[data-theme]` CSS blocks.

### D-041: Every setting is saved with the board
- Status: Active
- Amends: D-029
- Date: 2026-10-13
- Decision: Theme, snap, auto-capitalise, line style, and shortcuts panel open/pinned are stored in
  localStorage and inside the URL with the board, so a shared link reproduces them. Changing a
  setting is not an undo step.
- Code: `currentSettings`, `applySettings`, `settingChanged`.

### D-042: Share menu: whole board or selected items
- Status: Active
- Date: 2026-10-13
- Decision: "Share" replaces "Copy link". Its menu copies a link to the whole board, or to only
  the selection: selected boxes, lines between selected boxes, and selected lines with the two
  boxes they join. "Selected items" is disabled when nothing is selected.
- Code: `selectionBoard`, `shareBoard`, `#shareMenu`.

### D-043: Project lives in ~/Workspace/andidev/explainer under git
- Status: Active
- Date: 2026-10-13
- Decision: The project folder is `~/Workspace/andidev/explainer`, a git repository on `main`.
  Decisions are recorded in this file (see CLAUDE.md for the workflow).

### D-044: Decisions log and agent instructions
- Status: Active
- Date: 2026-10-13
- Decision: Every decision is recorded in `DECISIONS.md` with a permanent `D-NNN` ID. Each new
  feature request adds an entry; overridden entries are marked `Superseded by` / `Amended by` and
  the new entry names what it supersedes or amends. `CLAUDE.md` (and `AGENTS.md`, which points to
  it) instructs coding agents to follow this.
- Why: User asked for a decisions file that coding agents keep up to date, with overrides marked.
- Code: `DECISIONS.md`, `CLAUDE.md`, `AGENTS.md`.

### D-045: README and logo
- Status: Active
- Date: 2026-10-13
- Decision: The README explains what Explainer is for and how it works, with the logo (light and
  dark variants) and a screenshot at the top. It does not list candidate domains. The logo is two
  boxes joined by a routed arrow, beside the wordmark in Bricolage Grotesque with the tagline
  "Boxes, lines, and nothing in the way."
- Why: User asked for a polished GitHub README and a logo.
- Code: `README.md`, `docs/logo-*.png`, `docs/screenshot-*.png`.

### D-046: Double-click box size: last created, default, or last selected
- Status: Active
- Date: 2026-10-13
- Amends: D-030
- Decision: Double-clicking empty space creates a box the size of the last box you created (drawn
  or double-clicked; its size at creation, remembered across refreshes). Shift+double-click uses
  the default size (7×3 grid cells). Cmd+Shift (Mac) or Ctrl+Shift (Windows/Linux) +double-click
  uses the current size of the last box you selected by clicking or drag-selecting; if there is
  none, it falls back to the last created size and says so. With no box created yet, the default
  size is used.
- Why: User asked for quick repeat sizes when building diagrams.
- Code: `sizeFor`, `rememberCreated`, `lastCreatedSize`, `lastSelectedId`, `newBoxAt`.

### D-047: Shortcuts panel scrolls instead of growing off screen
- Status: Active
- Date: 2026-10-13
- Amends: D-022
- Decision: The panel's height is capped to the window; its list scrolls and its header (with the
  pin and minimise buttons) stays visible.
- Why: On short windows the panel's top, including minimise, was pushed off screen.
- Code: `#help` max-height, sticky `.help-head`.


### D-048: Public GitHub repository under the MIT license
- Status: Active
- Date: 2026-10-13
- Decision: Explainer is published as the public repository `andidev/explainer` under the MIT
  license, and the app is served from GitHub Pages at https://andidev.github.io/explainer/.
  Issues use bug report and feature request templates; the wiki and projects are off; pull
  requests are squash-merged and their branches deleted after merging.
- Why: User asked for a public GitHub repository with an MIT license and the usual configuration.
- Code: `LICENSE`, `.github/ISSUE_TEMPLATE/`, `README.md` (Try it, License).

### D-049: Backspace in an empty box deletes the box
- Status: Active
- Date: 2026-10-13
- Amends: D-014
- Decision: While editing a box whose text is empty (or only whitespace), pressing Backspace or
  Delete stops editing and deletes the box with its lines (undoable). A held key that has just
  erased the last character does not delete the box; a fresh press is needed.
- Why: User asked that Backspace in an empty box delete it.
- Code: window `keydown` handler (editing branch), `deleteSel`; shortcuts panel; README table.

### D-050: The cursor shows what a press will do
- Status: Active
- Date: 2026-10-13
- Decision: Over a box, the cursor is a hand (grab) wherever a press drags the box, including
  the space around the text of a box being edited. Over the text of a box being edited it is the
  text cursor. Whenever a connection dot is showing, so that a press starts a line (near a box's
  edge or a dot, or hovering a line near its end), the cursor is a crosshair everywhere it would
  be, overriding the hand and the text cursor. Shift (which selects instead) and panning keep
  their own cursors.
- Why: User reported no hand on the draggable part of a box being edited, and a hand where a
  press would create a line.
- Code: `nearDot` toggles `#viewport.linking`; CSS `.box`, `.box.editing .txt`, `#viewport.linking`.

### D-051: Cmd/Ctrl+click a box to draw an arrow to it
- Status: Amended by D-052
- Date: 2026-10-13
- Decision: With a box selected or being edited, Cmd+click (Mac) or Ctrl+click (Windows/Linux)
  on another box draws an arrow from the selected box to it (from every selected box, if several
  are selected; from the edited box while editing). The arrow joins the sides that face each
  other, uses the current line style, and is one undo step. Editing stops, and the clicked box
  becomes the selection, so repeated Cmd/Ctrl+clicks build a chain. A pair that is already
  connected in that direction is not doubled ("Already connected"). While Cmd/Ctrl is held over
  a box that would be connected, the cursor is the crosshair (D-050). Cmd/Ctrl+double-click on
  a box does not start editing.
- Why: User asked for a quick way to connect boxes from the keyboard and mouse.
- Code: `linkSources`, `facing`, `connectTo`, viewport `pointerdown`, `nearDot`, `modCursor`,
  `dblclick`; shortcuts panel; README table.

### D-052: Shift for a plain line, Shift+Ctrl for arrows at both ends
- Status: Active
- Date: 2026-10-13
- Amends: D-051
- Decision: When dropping a line dragged from a dot, holding Shift makes it a plain line (no
  arrowheads) and holding Shift+Ctrl gives arrowheads at both ends; the keys are read when the
  line is dropped, and the preview follows them during the drag. The same applies to
  Cmd/Ctrl+click (D-051): Cmd+Shift+click (Ctrl+Shift+click on Windows) draws a plain line, and
  Cmd+Shift+Ctrl+click on Mac draws arrows at both ends. On Windows that last one is
  Ctrl+Shift+Alt+click, because Ctrl already means "connect" there. Shift+pressing near a dot now
  starts a line if you drag; Shift+click without dragging still adds to or removes from the
  selection. Double-clicking a dot (D-011) still gives a line without an arrow.
- Why: User asked for modifier keys that choose a plain line or a two-way arrow, both when
  dragging and when Cmd/Ctrl+clicking; they chose these key combinations when asked about the
  Windows clash.
- Code: `heads`, `linkProps`, `connectTo`, `downHandle` (`shift0`), link field `back`
  (share format: arrow value 2 = both ends; older links still load, and older versions of the
  app read 2 as a normal arrow), `marker-start` with `orient="auto-start-reverse"`; shortcuts
  panel (`.both` key label); README.
