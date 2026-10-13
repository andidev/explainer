# Pins: named places to jump to

Date: 2026-10-13. Status: approved in chat (design and mocks).

## Goal

On a big board, let people mark places and jump back to them with one click. A pin is a named,
coloured location on the board. You can have as many as you want. They are listed as chips in a
new bar at the bottom of the window.

## Behaviour

### 1. Pin bar
- A new floating island at the bottom centre (`#islPins`), styled like the other islands. It holds
  a **📍+** button (`#pinAdd`), a separator, and one chip per pin in creation order. A chip is a
  coloured dot plus the pin's name.
- The bar is never wider than `min(560px, 100% - 2×(left island width) - gaps)` at desktop widths.
  When the chips don't fit, it scrolls sideways (no visible scrollbar).
- At ≤640px it sits just above the bottom-left island, stretching between the left and right
  insets (12px), and still scrolls sideways.
- With no pins, the bar shows only 📍+ and the tooltip "Drag onto the board to add a pin".
- Nothing may overlap the other islands, the selection toolbar's clearance, the toast or the
  shortcuts panel button at 375, 800 or 1440px. The toast moves up if needed.

### 2. Creating a pin
- Press on 📍+ and drag onto the board. A marker preview follows the pointer, and the cursor is
  `copy`. Release over the board to drop a pin at that world position. Releasing over any island
  or the bar cancels it.
- Clicking 📍+ without dragging drops a pin at the centre of the view, so it works on touch and
  with the keyboard.
- The new pin is named "Pin N", where N is the lowest number not used by another pin named
  "Pin <number>". Its colour is the first of the 8 pin colours not used by any pin; if all 8 are
  in use, it cycles by count (`pins.length % 8`).
- Its chip opens straight into rename mode with the text selected: Enter or blur saves, Esc keeps
  the current name, and an empty name keeps the old name.
- Creating a pin is one undo step. The rename is a second step only if the name changed.

### 3. Markers on the board
- Each pin draws a marker at its world position: a map-pin shape in the pin's colour with its
  point exactly on the position, plus the name in a small label to its right. Markers keep a
  constant on-screen size at every zoom (counter-scaled by `1/view.z`). They sit above boxes and
  lines, but below the islands.
- Clicking a marker jumps to it (as in §4).
- Dragging a marker moves the pin, and the drop is one undo step. Snap to grid does not apply.

### 4. Jumping
- Clicking a chip or a marker glides the view (`animateView`) so the pin is centred, keeping the
  current zoom. The marker then flashes briefly. A press that moves less than 4px counts as a click.

### 5. Moving from the bar
- Dragging a chip out of the bar onto the board moves that pin to the drop position, as one undo
  step. While dragging, the old marker fades and a preview marker follows the pointer. Releasing
  over the bar or an island cancels.

### 6. Rename and delete
- Right-clicking a chip or a marker opens a small menu (`#pinMenu`) with **Rename** and
  **Delete pin**, the latter in a danger colour. Esc or a click outside closes it.
- Double-clicking a chip renames it too.
- Delete is one undo step, with the toast "Pin deleted. Undo brings it back."
- Right-click on the board elsewhere keeps panning as today.

### 7. Board, undo, links
- State gains `pins: [{ id, x, y, name, c }]`, where `c` is the colour index 0–7 and ids come
  from `state.nextId`. Boards saved before pins existed load with `pins: []`.
- Undo and redo cover every pin change. After an undo or redo that changed a pin, the view
  reveals that pin, through `diff()`.
- **Share links:** the packed array keeps its layout `[1, nextId, boxes, links, settings]`, and
  pins are appended as an optional 6th element `[[id, x, y, name, c], …]`, written only when
  there are pins. Old links load with no pins. Older app versions ignore the extra element. The
  format version stays `1`.
- `sameBoard` also compares pins.
- **Clear board** removes pins too (one undo step). Open the example board has no pins.
- **Copy, paste and Share → Selected items** leave pins out.

### 8. Colours
- Eight tokens `--pin-0` … `--pin-7` on `:root`, with dark-mode values, distinct from each other
  and from `--accent`, and readable on `--paper` and `--box`. A pin stores the index, so colours
  follow the theme.

### 9. Shortcuts panel and docs
- Shortcuts panel rows:
  - "Drag 📍 onto the board": Add a pin.
  - "Click a pin": Jump there.
  - "Drag a pin chip onto the board": Move the pin.
  - "Right-click a pin": Rename or delete.
- README gets a short **Pins** paragraph plus the same rows in the shortcut table.
- `DECISIONS.md` gets D-063 (pins). `CLAUDE.md`'s code map gets a line on pins.

## Non-goals

- Pins are not attached to boxes, and moving a box does not move a pin.
- There is no keyboard shortcut to cycle pins, pins are not reordered, and pins can't be picked
  manually by colour.

## Testing (manual, in the browser)

- **Pin actions:** create by drag and by click; rename (Enter, Esc, empty); jump from a chip and
  from a marker; move from a chip and by dragging a marker; delete from the menu on a chip and on
  a marker. Each one has an undo/redo check.
- **Saving:** refresh keeps pins; a share link carries pins into a fresh load (headless Chrome);
  a link made before pins (from v1.1.0) loads with no pins and no errors.
- **Board actions:** Clear board removes pins, and undo restores them.
- **Display:**
  - 8+ pins: colours cycle and the bar scrolls.
  - Widths: 1440, 800 and 375px with no overlaps.
  - Dark mode: colours readable.
  - The usual checks: box and type, connect, undo/redo, refresh.

## Release

After implementation and review, and with the user's go-ahead (pushing `main` deploys the live
site): merge to `main`, push, check GitHub Pages, and publish v1.2.0.
