# Floating UI Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace Explainer's header bar with floating islands and a ☰ menu, add a quiet Share
button, a setting for new lines' arrowheads, a selection toolbar for existing lines and boxes,
and a welcome screen on an empty board.

**Architecture:** The whole app is the single file `index.html` (inline CSS and JS, no build
step, no dependencies). The existing controls keep their element IDs and handlers and move
into fixed-position islands. New behaviour is added as small functions next to related code.
State and share-link compatibility are preserved.

**Tech Stack:** Plain HTML/CSS/JS in one file. Google Fonts (Atkinson Hyperlegible, Bricolage
Grotesque, plus Caveat in Task 4). No test framework: verification is by running JavaScript
checks and taking screenshots in a browser.

**Spec:** `docs/superpowers/specs/2026-10-13-floating-ui-design.md` (read it before starting).
Also read `CLAUDE.md` (code map, rules) and `DECISIONS.md` (the Active decisions are the
app's required behaviour).

## Global Constraints

- One file: all code stays in `index.html`. No build step, no new dependencies, no frameworks.
- Colours are CSS tokens on `:root` (`--paper --dot --ink --box --line --accent --accent-soft
  --muted --chrome --edge`), with dark values already defined. Never write colour literals in
  new CSS, except the existing `rgba(20,30,60,…)` shadow colour, which may be reused.
- Old share links and stored boards must keep loading; `pack`/`unpack` keep the format version `1`.
- Call `commit()` after every user-visible board change (one undo step per user action).
- Keyboard shortcuts keep working: T (theme), G (grid), Ctrl/⌘ Z/Y/A/C/X/V, Alt+←/→, Esc,
  Enter, Backspace/Delete, Space+drag.
- No horizontal page scroll at 375px width; nothing overlaps at 375, 800 or 1440px width.
- Copy is sentence case with no exclamation marks, matching the existing UI text.
- Do **not** edit `DECISIONS.md`, `README.md` or the shortcuts panel text until Task 5, except
  where a task says so.

## Review Focus

1. **Pressing a control must not also act on the board.** Clicking an island, the menu, the
   selection toolbar or a welcome item must not start a marquee, deselect, create a box on
   double-click, or close/open other panels by accident. Each task includes a double-click and
   press check over its new UI.
2. **Editing text must survive.** Using an island or toolbar button while typing in a box must not
   lose the typed text or break the undo step. Buttons use `pointerdown` `preventDefault`,
   as `undoBtn` does.
3. **Phone width.** At 375px the islands, menu and toolbar must fit on screen, with no
   overlaps. Tasks 1, 3 and 4 check this.
4. **Dark mode.** Every new surface must use tokens. Each task checks with
   `document.documentElement.setAttribute('data-theme','dark')`.
5. **Keeping state and links stable.** Toggling settings, then refreshing or opening an older link,
   must keep the board and settings. Task 2 checks share settings key `a` with it missing and
   present.

---

## How to verify (all tasks)

There is no test runner. Verify in a browser:

1. Open `file:///Users/anders/Workspace/andidev/explainer/index.html` in the browser pane
   (`mcp__Claude_Browser__navigate`), or any Chromium browser.
2. Run each task's check snippets with `mcp__Claude_Browser__javascript_tool`. Each snippet
   returns an object; compare it with the **Expected** given.
3. Take screenshots where the task says (`mcp__Claude_Browser__computer` with action
   `screenshot`; use `mcp__Claude_Browser__resize_window` for widths, then reset with preset `desktop`).
4. The preview pane serves the file as a static snapshot: localStorage is unavailable there, so
   every load starts with the example board. That is expected.
5. If you have no browser tools, say so in your report with the snippets unrun. Do not claim
   they pass; the controller will run them.

Helper used by several snippets (paste at the top of a snippet when needed):

```js
const $q = s => document.querySelector(s), $$q = s => [...document.querySelectorAll(s)];
const rect = el => { const r = el.getBoundingClientRect(); return { l: Math.round(r.left), t: Math.round(r.top), r: Math.round(r.right), b: Math.round(r.bottom) }; };
const overlap = (a, b) => !(a.r <= b.l || b.r <= a.l || a.b <= b.t || b.b <= a.t);
const fire = (el, type, o = {}) => el.dispatchEvent(new PointerEvent(type, { bubbles: true, cancelable: true, composed: true, pointerId: 1, pointerType: 'mouse', button: 0, buttons: type === 'pointerup' ? 0 : 1, isPrimary: true, ...o }));
```

Commit messages reference the decision IDs this work will record in Task 5 (D-057 … D-062), and
end with:

```
Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>
```

---

### Task 1: Floating islands, ☰ menu and quiet Share (replace the header)

**Files:**
- Modify: `index.html` CSS (lines ~35–67 header/tools rules, ~110–139 toast/share/help rules),
  the `<header>…</header>` markup (~line 144), `let view = …` (~242), `fitBoard` (~525),
  `setTheme` (~639), `setCap` (~620), `setSnap` (~640), the button wiring block (~1272–1300),
  and the help close listener (~1316).

**Interfaces:**
- Consumes: the existing functions `setTheme(t)`, `setCap(m)`, `setSnap(on)`, `fitBoard()`,
  `zoomAt(z, sx, sy)`, `goPlace(step)`, `undo()`, `redo()`, `setHelp(open)`, `openShareMenu(open)`,
  `toast(msg)`, `sample()`, `onGrid(s)`, `renderAll()`, `commit()`, `setSel()`, `endEdit()`,
  and the globals `state`, `editing`, `armed`, `view`, `viewport`.
- Produces, for later tasks:
  - `#islTL`, `#islTC`, `#islTR`, `#islBL`: island elements, all with class `isl`.
  - `#islTC` contains `<span class="seg" id="segStyle">` with `#lsAngled` and `#lsCurved`.
    Task 2 appends a second `.seg` with id `segHeads`.
  - `openMenu(open: boolean)`: shows or hides `#menu`.
  - `loadExample()`: replaces the board with the example board as one undo step. Used by Task 4.
  - CSS classes `.isl`, `.ib` (an icon button in an island), `.isep` (a separator) and
    `.phone-only`. Task 3 reuses `.isl`, `.ib` and `.isep`.

- [ ] **Step 1: Record the baseline**

Open the app and run:

```js
({ header: !!document.querySelector('header'), ids: ['themeBtn','capBtn','snapBtn','shareBtn','undoBtn','redoBtn','zoomBtn','homeBtn','clearBtn','placeBack','placeFwd','lsAngled','lsCurved','exampleTag'].filter(id => !document.getElementById(id)) })
```

Expected: `{ header: true, ids: [] }`.

- [ ] **Step 2: Replace the header markup**

Replace everything from `<header>` to `</header>` with the following. Keep the existing inner
SVGs of `.mark`, `#shareBtn`, `#lsAngled`, `#lsCurved` and `.gh`; copy them across unchanged
where the comments say so.

```html
<div class="isl" id="islTL">
  <button id="menuBtn" class="ib" type="button" aria-haspopup="true" aria-expanded="false" aria-label="Menu" title="Menu"><svg viewBox="0 0 16 16" aria-hidden="true"><path d="M2.5 4h11M2.5 8h11M2.5 12h11" stroke="currentColor" stroke-width="1.6" stroke-linecap="round"/></svg></button>
  <div class="mark"><!-- existing .mark svg, unchanged -->Explainer</div>
  <span class="example" id="exampleTag" hidden>Example board</span>
  <div id="menu" role="menu" hidden>
    <button id="themeBtn" class="mrow" type="button" role="menuitem"></button>
    <button id="capBtn" class="mrow" type="button" role="menuitem"></button>
    <button id="snapBtn" class="mrow" type="button" role="menuitemcheckbox" aria-checked="false"><span class="mr-l">Snap to grid</span><span class="track" aria-hidden="true"></span></button>
    <hr>
    <button id="exampleBtn" class="mrow" type="button" role="menuitem"><span class="mr-l">Open the example board</span></button>
    <button id="clearBtn" class="mrow" type="button" role="menuitem"><span class="mr-l">Clear board</span></button>
    <hr class="phone-only">
    <button id="mFit" class="mrow phone-only" type="button" role="menuitem"><span class="mr-l">Fit board</span></button>
    <button id="mBack" class="mrow phone-only" type="button" role="menuitem"><span class="mr-l">Previous edit place</span><span class="mr-v">Alt ←</span></button>
    <button id="mFwd" class="mrow phone-only" type="button" role="menuitem"><span class="mr-l">Next edit place</span><span class="mr-v">Alt →</span></button>
    <hr>
    <button id="mHelp" class="mrow" type="button" role="menuitem"><span class="mr-l">Shortcuts</span></button>
    <a id="mGit" class="mrow" role="menuitem" href="https://github.com/andidev/explainer" target="_blank" rel="noopener"><span class="mr-l">Source on GitHub</span></a>
  </div>
</div>
<div class="isl" id="islTC" role="group" aria-label="Style for new lines" title="Style for new lines">
  <span class="seg" id="segStyle">
    <button id="lsAngled" type="button" aria-pressed="true" title="Angled lines that follow the grid"><!-- existing svg -->Angled</button>
    <button id="lsCurved" type="button" aria-pressed="false" title="Curved lines"><!-- existing svg -->Curved</button>
  </span>
</div>
<div class="isl" id="islTR">
  <a class="gh ib" href="https://github.com/andidev/explainer" target="_blank" rel="noopener" title="Explainer on GitHub" aria-label="Explainer on GitHub"><!-- existing GitHub svg --></a>
  <span class="share-wrap">
    <button id="shareBtn" type="button" aria-haspopup="true" aria-expanded="false" title="Share a link to this board or to the selected items"><!-- existing share svg -->Share</button>
    <div id="shareMenu" role="menu" hidden>
      <button id="shareAll" type="button" role="menuitem"><b>Whole board</b><span id="shareAllNote"></span></button>
      <button id="shareSel" type="button" role="menuitem"><b>Selected items</b><span id="shareSelNote"></span></button>
    </div>
  </span>
</div>
<div class="isl" id="islBL">
  <button id="zoomOut" class="ib" type="button" aria-label="Zoom out" title="Zoom out (Ctrl/⌘ −)"><svg viewBox="0 0 16 16" aria-hidden="true"><path d="M3.5 8h9" stroke="currentColor" stroke-width="1.6" stroke-linecap="round"/></svg></button>
  <button id="zoomBtn" type="button" title="Reset zoom to 100%">100%</button>
  <button id="zoomIn" class="ib" type="button" aria-label="Zoom in" title="Zoom in (Ctrl/⌘ +)"><svg viewBox="0 0 16 16" aria-hidden="true"><path d="M3.5 8h9M8 3.5v9" stroke="currentColor" stroke-width="1.6" stroke-linecap="round"/></svg></button>
  <button id="homeBtn" class="no-phone" type="button" title="Show the whole board">Fit board</button>
  <span class="isep"></span>
  <button id="undoBtn" class="ib" type="button" aria-label="Undo" title="Undo (Ctrl/⌘ Z)"><svg viewBox="0 0 16 16" aria-hidden="true"><path d="M5.5 3.5L2.5 6.5l3 3M2.8 6.5H10a3.5 3.5 0 0 1 0 7H7" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg></button>
  <button id="redoBtn" class="ib" type="button" aria-label="Redo" title="Redo (Ctrl/⌘ Shift Z)"><svg viewBox="0 0 16 16" aria-hidden="true"><path d="M10.5 3.5l3 3-3 3M13.2 6.5H6a3.5 3.5 0 0 0 0 7h3" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg></button>
  <span class="isep no-phone"></span>
  <span class="seg no-phone" role="group" aria-label="Edit places">
    <button id="placeBack" type="button" title="Previous edit place (Alt ←)" aria-label="Previous edit place">←</button>
    <span class="seg-label">Edits</span>
    <button id="placeFwd" type="button" title="Next edit place (Alt →)" aria-label="Next edit place">→</button>
  </span>
</div>
```

- [ ] **Step 3: Replace the header CSS with island CSS**

Delete these rules: `header{…}`, `.hint{…}` (both occurrences), `.hint kbd{…}`, `.tools{…}`,
`#snapBtn,#capBtn{…}` / `#snapBtn{…}` / `#capBtn{…}` / `#capBtn .cap-ico{…}`, `#themeBtn{…}`,
`#themeBtn svg{…}`, `.gh{…}`, `.gh:hover{…}`, `.gh:focus-visible{…}`, `.gh svg{…}`. Keep `.mark`,
`.mark svg`, `.track…`, `.seg…`, `button…`, `#zoomBtn`, `.example`, `#shareMenu…` and `#help…`.
Change `body{…display:flex;flex-direction:column…}` so `#viewport` fills the window: add
`#viewport{position:fixed;inset:0}` (keep its other properties). Then add:

```css
/* Floating islands: the board fills the window and the controls float at its edges */
.isl{position:fixed;z-index:5;display:flex;align-items:center;gap:4px;padding:4px;background:var(--chrome);backdrop-filter:blur(8px);
  border:1px solid var(--edge);border-radius:10px;box-shadow:0 6px 18px -12px rgba(20,30,60,.45)}
#islTL{top:calc(12px + env(safe-area-inset-top,0px));left:12px;padding-right:10px;gap:8px}
#islTC{top:calc(12px + env(safe-area-inset-top,0px));left:50%;transform:translateX(-50%)}
#islTR{top:calc(12px + env(safe-area-inset-top,0px));right:12px}
#islBL{bottom:calc(12px + env(safe-area-inset-bottom,0px));left:12px}
.isl .mark{font-size:17px;gap:7px} .isl .mark svg{width:19px;height:19px}
.ib{width:30px;height:30px;padding:0;display:inline-flex;align-items:center;justify-content:center;border:0;background:transparent;color:var(--muted);border-radius:7px}
.ib:hover:not(:disabled){background:var(--accent-soft);color:var(--ink)}
.ib svg{width:17px;height:17px}
.isl > button:not(.ib){border:0;background:transparent}
.isep{width:1px;height:20px;background:var(--edge);margin:0 2px}
#islBL .seg{border:0;background:transparent}
.gh svg{width:19px;height:19px}
/* Share: quiet — island surface, accent icon, soft accent on hover */
#shareBtn{display:inline-flex;align-items:center;gap:6px;background:var(--box);border:1px solid var(--edge);color:var(--ink);font-weight:700;padding:5px 12px}
#shareBtn svg{width:14px;height:14px;color:var(--accent)}
#shareBtn:hover{background:var(--accent-soft);border-color:var(--accent);color:var(--ink)}
#shareMenu{left:auto;right:0}
/* ☰ menu */
#menu{position:absolute;top:calc(100% + 8px);left:0;z-index:8;min-width:244px;padding:4px;background:var(--box);border:1px solid var(--edge);border-radius:10px;
  box-shadow:0 10px 28px -12px rgba(20,30,60,.45);display:flex;flex-direction:column;gap:1px}
#menu[hidden]{display:none}
#menu hr{border:0;border-top:1px solid var(--edge);margin:4px 2px;width:auto}
.mrow{display:flex;align-items:center;justify-content:space-between;gap:12px;width:100%;border:0;background:transparent;padding:7px 10px;border-radius:6px;
  font:inherit;font-size:13px;color:var(--ink);text-decoration:none;text-align:left;cursor:pointer}
.mrow:hover{background:var(--accent-soft);color:var(--ink)}
.mr-v{display:inline-flex;align-items:center;gap:6px;color:var(--muted);font-size:12.5px}
.mr-v svg{width:14px;height:14px}
.phone-only{display:none!important}
@media (max-width:640px){
  .phone-only{display:flex!important} hr.phone-only{display:block!important}
  .no-phone{display:none!important}
  #islTC{top:calc(62px + env(safe-area-inset-top,0px))}
  #islTL .example{display:none}
}
```

Move `#toast` above the bottom islands: in `#toast{…}` change
`bottom:calc(20px + env(safe-area-inset-bottom,0px))` to `bottom:calc(64px + env(safe-area-inset-bottom,0px))`.
Restyle `#helpBtn` as an island: replace its `background:var(--chrome);backdrop-filter:blur(8px)`
with `background:var(--chrome);backdrop-filter:blur(8px);border-radius:10px;width:40px;height:40px;right:12px;bottom:calc(12px + env(safe-area-inset-bottom,0px));box-shadow:0 6px 18px -12px rgba(20,30,60,.45)`,
and remove `border-radius:50%` from it. In `#help{…}` change `right:16px` to `right:12px` and
`bottom:calc(16px + …)` to `bottom:calc(60px + env(safe-area-inset-bottom,0px))` so the panel
opens above the button. Then add `body.help-open #helpBtn{display:inline-flex}` after the existing
`body.help-open #helpBtn{display:none}`, so the button stays visible.

- [ ] **Step 4: Fill the menu rows from the setters**

In `setTheme(t)`, replace the two lines that set `b.innerHTML` and `b.title` with:

```js
  const b = $('themeBtn'); if (b){ b.innerHTML = '<span class="mr-l">Theme</span><span class="mr-v">' + THEME_ICON[t] + { auto:'Auto', light:'Light', dark:'Dark' }[t] + '</span>';
    b.title = 'Theme: ' + { auto:'follows your system', light:'light', dark:'dark' }[t] + ' (T)'; }
```

In `setCap(m)`, replace the line that sets `b.innerHTML` with:

```js
  const b = $('capBtn'); b.innerHTML = `<span class="mr-l">Capitals</span><span class="mr-v">${CAPS[m]}</span>`;
```

(Keep its `b.title` line.) Delete the now-unused `CAP_ICO` constant. `setSnap` already sets
`aria-checked` on `#snapBtn`, so the switch track keeps working; leave it unchanged.

- [ ] **Step 5: Wire the new controls**

After `$('homeBtn').onclick = fitBoard;`, add:

```js
// ☰ menu: settings rows stay open so you can cycle them; actions close it
function openMenu(open){
  $('menu').hidden = !open; $('menuBtn').setAttribute('aria-expanded', open);
  if (open) openShareMenu(false);
}
$('menuBtn').addEventListener('pointerdown', e => e.preventDefault());
$('menuBtn').onclick = () => openMenu($('menu').hidden);
$('menu').addEventListener('pointerdown', e => { if (!e.target.closest('a')) e.preventDefault(); }); // keep focus (and any edit) where it is
document.addEventListener('pointerdown', e => { if (!$('menu').hidden && !e.target.closest('#menu, #menuBtn')) openMenu(false); }, true);
window.addEventListener('keydown', e => { if (e.key === 'Escape' && !$('menu').hidden){ e.stopPropagation(); openMenu(false); } }, true);
['exampleBtn','clearBtn','mFit','mBack','mFwd','mHelp','mGit'].forEach(id => $(id).addEventListener('click', () => openMenu(false)));
// Replace the board with the example board, as one undo step
function loadExample(){
  if (editing) endEdit();
  const ex = onGrid(sample()); ex.nextId = Math.max(ex.nextId, state.nextId);
  state = ex; setSel(); armed = null; renderAll(); commit(); fitBoard();
}
$('exampleBtn').onclick = loadExample;
$('mFit').onclick = fitBoard;
$('mBack').onclick = () => goPlace(-1);
$('mFwd').onclick = () => goPlace(1);
$('mHelp').onclick = () => setHelp(true);
['zoomIn','zoomOut','zoomBtn','homeBtn'].forEach(id => $(id).addEventListener('pointerdown', e => e.preventDefault()));
$('zoomIn').onclick = () => zoomAt(view.z*1.2, viewport.clientWidth/2, viewport.clientHeight/2);
$('zoomOut').onclick = () => zoomAt(view.z/1.2, viewport.clientWidth/2, viewport.clientHeight/2);
```

In `openShareMenu(open)`, add `if (open) openMenu(false);` as its second line (after the
`m.hidden` line). `openMenu` is a function declaration, so it is hoisted. Add
`$('snapBtn').addEventListener('pointerdown', e => e.preventDefault());` next to the existing
`capBtn` `pointerdown` line.

In the help close listener (`if (!helpPinned && … !e.target.closest('#help, #helpBtn')) setHelp(false);`),
change the selector to `'#help, #helpBtn, #mHelp'`, so choosing Shortcuts in the menu doesn't
close the panel straight away.

- [ ] **Step 6: Leave room for the islands**

Change `let view = { x: 40, y: 30, z: 1 };` to `let view = { x: 40, y: 80, z: 1 };`. In
`fitBoard()`, change `animateView({ x:40, y:30, z:1 })` to `animateView({ x:40, y:80, z:1 })` and
change `viewport.clientHeight/(r.h + 120)` to `viewport.clientHeight/(r.h + 200)`. That is
100px top and bottom: room for the islands.

In the viewport `pointerdown` handler, change `if (e.target.closest('header')) return;` to
`if (e.target.closest('.isl, #menu, #help, #helpBtn')) return;`. The islands sit outside
`#viewport`, so this is a safety net. In the `dblclick` handler, change
`under.closest('header, #help, #helpBtn')` to `under.closest('.isl, #menu, #help, #helpBtn')`.

- [ ] **Step 7: Verify**

Reload and run:

```js
const $q = s => document.querySelector(s);
const rect = el => { const r = el.getBoundingClientRect(); return { l: Math.round(r.left), t: Math.round(r.top), r: Math.round(r.right), b: Math.round(r.bottom) }; };
const overlap = (a, b) => !(a.r <= b.l || b.r <= a.l || a.b <= b.t || b.b <= a.t);
const isl = ['#islTL','#islTC','#islTR','#islBL','#helpBtn'].map(s => rect($q(s)));
const pairs = []; for (let i = 0; i < isl.length; i++) for (let j = i + 1; j < isl.length; j++) if (overlap(isl[i], isl[j])) pairs.push([i, j]);
$q('#menuBtn').click(); const menuOpen = !$q('#menu').hidden;
$q('#capBtn').click(); const capStill = !$q('#menu').hidden, capText = $q('#capBtn').textContent;
document.body.dispatchEvent(new PointerEvent('pointerdown', { bubbles: true })); const closed = $q('#menu').hidden;
({ header: !!$q('header'), pairs, menuOpen, capStill, capText, closed, noHScroll: document.documentElement.scrollWidth <= innerWidth,
   shareBg: getComputedStyle($q('#shareBtn')).backgroundColor, boxes: document.querySelectorAll('.box').length })
```

Expected: `header: false`, `pairs: []`, `menuOpen: true`, `capStill: true`, `capText` is
`"CapitalsSentence case"`, `closed: true`, `noHScroll: true`, `shareBg` is the `--box` colour
(`rgb(255, 255, 255)` in light mode), and `boxes: 3`.

Then check Clear board, Open the example board, undo and zoom:

```js
const $q = s => document.querySelector(s); const n = () => document.querySelectorAll('.box').length;
$q('#clearBtn').click(); const a = n(); $q('#exampleBtn').click(); const b = n();
$q('#undoBtn').click(); const c = n(); const z0 = $q('#zoomBtn').textContent; $q('#zoomIn').click();
({ a, b, c, z0, z1: $q('#zoomBtn').textContent })
```

Expected: `a: 0`, `b: 3`, `c: 0`, and `z1` is larger than `z0` (for example `"100%"` → `"120%"`).

Screenshots: take one at the pane's normal size, one at 1440×800, and one at 375×812
(`resize_window` preset `mobile`). Then run the first snippet's `pairs` and `noHScroll` checks
at 375px. Expected: no overlaps and no horizontal scroll. In dark mode
(`document.documentElement.setAttribute('data-theme','dark')`), check that the islands and the
menu are dark. Reset the viewport with preset `desktop`.

Review Focus check (pressing a control must not act on the board): double-click the middle of
`#islBL` (`computer` `double_click` on its coordinates) and confirm the box count didn't change.

- [ ] **Step 8: Commit**

```bash
git add index.html
git commit -m "Floating islands, ☰ menu and quiet Share replace the header (D-057, D-058, D-059)

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---

### Task 2: Arrowheads for new lines (top-centre island)

**Files:**
- Modify: `index.html`: `#islTC` markup, `heads()` (~line 868), `setLineStyle()` and
  `updateStyleButtons()` (~473–485), `currentSettings`/`applySettings` (~271, ~665), and the
  wiring block.

**Interfaces:**
- Consumes: `#islTC`/`#segStyle` (Task 1), `heads(e, click, noArrow)`, `linkProps(h)`, `isMac`,
  `settingChanged()`, `toast()`.
- Produces:
  - Global `let newHeads = 'end'` with values `'end' | 'none' | 'both'`.
  - `setNewHeads(v)`, which stores the setting, updates the buttons and marks settings changed.
  - `headsFrom(v)`, which returns `{arrow:true}`, `{arrow:false}` or `{arrow:true, back:true}`.
    Task 3 uses it.
  - `setLineStyle(style)` now only sets the style for **new** lines.

- [ ] **Step 1: Check it fails first**

```js
({ seg: !!document.getElementById('segHeads'), fn: typeof setNewHeads })
```

Expected: `{ seg: false, fn: "undefined" }`. (`setNewHeads` is a top-level function in a
classic script, so it will be visible once added. If the script is wrapped in an IIFE, use the
DOM checks in Step 5 instead.)

- [ ] **Step 2: Add the buttons**

Inside `#islTC`, after `</span>` of `#segStyle`, add:

```html
  <span class="seg" id="segHeads" role="group" aria-label="Arrowheads for new lines">
    <button id="hdEnd" type="button" aria-pressed="true" title="Arrow at the end"><svg viewBox="0 0 16 16" aria-hidden="true"><path d="M2 8h11M9.5 4.5L13 8l-3.5 3.5" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg></button>
    <button id="hdNone" type="button" aria-pressed="false" title="Plain line, no arrow (or hold Shift while drawing)"><svg viewBox="0 0 16 16" aria-hidden="true"><path d="M2 8h12" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round"/></svg></button>
    <button id="hdBoth" type="button" aria-pressed="false" title="Arrows at both ends (or hold Shift+Ctrl while drawing)"><svg viewBox="0 0 16 16" aria-hidden="true"><path d="M2.5 8h11M5.5 4.5L2 8l3.5 3.5M10.5 4.5L14 8l-3.5 3.5" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg></button>
  </span>
```

Add the CSS `#islTC{gap:6px}` and `#segHeads button{padding:5px 8px}`. The `.seg` styles from
the existing CSS already apply (pressed = accent). At ≤640px, hide the "Angled"/"Curved" words
to save width: wrap each label word in `<span class="lbl-txt">…</span>` inside `#lsAngled` and
`#lsCurved`, and add `@media (max-width:640px){#islTC .lbl-txt{display:none}}` inside the
existing 640px media block.

- [ ] **Step 3: Add the setting and use it when drawing**

After `let lineStyle = 'angled';` and its `try` line, add:

```js
// Arrowheads for new lines: 'end' (default), 'none' or 'both'. Shift / Shift+Ctrl still override while drawing.
let newHeads = 'end';
try { const v = localStorage.getItem('explainer.newheads'); if (v === 'none' || v === 'both') newHeads = v; } catch(e){}
const headsFrom = v => v === 'both' ? { arrow:true, back:true } : v === 'none' ? { arrow:false } : { arrow:true };
function setNewHeads(v){
  newHeads = v; try { localStorage.setItem('explainer.newheads', v); } catch(e){}
  updateStyleButtons(); settingChanged();
}
```

Replace `setLineStyle` and `updateStyleButtons` with:

```js
function setLineStyle(style){
  lineStyle = style; try { localStorage.setItem('explainer.linestyle', style); } catch(e){} settingChanged();
  updateStyleButtons();
}
// The top-centre island shows the style for new lines (selected lines are changed from the selection toolbar)
function updateStyleButtons(){
  const a = $('lsAngled'), c = $('lsCurved'); if (!a) return;
  a.setAttribute('aria-pressed', lineStyle === 'angled'); c.setAttribute('aria-pressed', lineStyle === 'curved');
  const hb = { end:$('hdEnd'), none:$('hdNone'), both:$('hdBoth') };
  for (const k in hb) if (hb[k]) hb[k].setAttribute('aria-pressed', newHeads === k);
}
```

In `heads(e, click, noArrow)`, replace its first line
`if (!e.shiftKey) return noArrow ? { arrow:false } : { arrow:true };` with:

```js
  if (!e.shiftKey) return noArrow ? { arrow:false } : headsFrom(newHeads);
```

Wire the buttons next to the `lsAngled`/`lsCurved` wiring:

```js
['hdEnd','hdNone','hdBoth'].forEach(id => $(id).addEventListener('pointerdown', e => e.preventDefault()));
$('hdEnd').onclick = () => { setNewHeads('end'); toast('New lines get an arrow'); };
$('hdNone').onclick = () => { setNewHeads('none'); toast('New lines have no arrow'); };
$('hdBoth').onclick = () => { setNewHeads('both'); toast('New lines get arrows at both ends'); };
```

and in the `lsAngled`/`lsCurved` handlers, show a toast after `setLineStyle(...)`:
`$('lsAngled').onclick = () => { setLineStyle('angled'); toast('New lines will be angled'); };`
(and the same for curved with `'curved'`).

- [ ] **Step 4: Put it in share-link settings**

In `currentSettings`, add `a: { none:0, end:1, both:2 }[newHeads],` after the `l:` entry. In
`applySettings(st)`, after the `st.l` line, add:

```js
  if (st.a != null) setNewHeads(st.a === 2 ? 'both' : st.a === 0 ? 'none' : 'end');
```

(An old link without `a` leaves `newHeads` unchanged.)

- [ ] **Step 5: Verify**

Paste the `fire` helper from "How to verify". Then run this drag check: it draws from box 1's
east dot to box 2 and reports the new line's markers.

```js
const fire = (el, type, o = {}) => el.dispatchEvent(new PointerEvent(type, { bubbles: true, cancelable: true, composed: true, pointerId: 1, pointerType: 'mouse', button: 0, buttons: type === 'pointerup' ? 0 : 1, isPrimary: true, ...o }));
const boxes = [...document.querySelectorAll('.box')]; boxes.forEach(b => b.querySelectorAll('.h').forEach(h => {}));
const b1 = boxes[0].getBoundingClientRect(), b2 = boxes[1].getBoundingClientRect();
async function draw(mods = {}){
  const sx = b1.left + b1.width/2, sy = b1.bottom - 4, ex = b2.left + b2.width/2, ey = b2.top + b2.height/2;
  const at = (x, y) => document.elementFromPoint(x, y);
  fire(at(sx, sy), 'pointermove', { clientX: sx, clientY: sy, buttons: 0 }); fire(at(sx, sy), 'pointerdown', { clientX: sx, clientY: sy });
  for (let i = 1; i <= 6; i++){ const x = sx + (ex - sx)*i/6, y = sy + (ey - sy)*i/6; fire(at(x, y), 'pointermove', { clientX: x, clientY: y, ...mods }); await new Promise(r => setTimeout(r, 20)); }
  fire(at(ex, ey), 'pointerup', { clientX: ex, clientY: ey, ...mods }); await new Promise(r => setTimeout(r, 50));
  const p = [...document.querySelectorAll('#links .ln')].pop(); const m = (p.getAttribute('marker-start') ? 'S' : '-') + (p.getAttribute('marker-end') ? 'E' : '-');
  document.getElementById('undoBtn').click(); return m;
}
const out = {};
document.getElementById('hdNone').click(); out.none = await draw();
out.noneShiftCtrl = await draw({ shiftKey: true, ctrlKey: true });
document.getElementById('hdBoth').click(); out.both = await draw();
out.bothShift = await draw({ shiftKey: true });
document.getElementById('hdEnd').click(); out.end = await draw();
out.pressed = ['hdEnd','hdNone','hdBoth'].map(id => document.getElementById(id).getAttribute('aria-pressed'));
out
```

Expected: `none: "--"`, `noneShiftCtrl: "SE"`, `both: "SE"`, `bothShift: "--"`, `end: "-E"`,
`pressed: ["true","false","false"]`.

Check that the style island no longer changes the selection. Select a line by clicking it, click
`#lsCurved`, and confirm that line's path is unchanged:
`document.querySelector('#links .ln.sel').getAttribute('d')` returns the same before and after.
Then click `#lsAngled` to restore.

Share-settings check (Review Focus 5): run
`JSON.stringify(currentSettings())` if it is reachable; otherwise skip this and say so. With
`hdBoth` pressed, expect `"a":2` in it.

Take a screenshot of the top-centre island at the normal width and at 375px.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "Arrowheads for new lines in the top island; style island sets new lines only (D-062)

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---

### Task 3: Selection toolbar

**Files:**
- Modify: `index.html`: markup (add `#selbar` right after `#islBL`), CSS (after the island CSS),
  `renderSel()` (~602), `applyView()` (~489), the `pointermove` listener (~857), `cleanupGesture()`
  (~934), `startEdit`/`endEdit` (they already call `renderSel()`), and the wiring block.

**Interfaces:**
- Consumes: `.isl`, `.ib`, `.isep` CSS (Task 1); `headsFrom(v)` (Task 2); `linkGeom(a, fa, b, ta, curve)`
  (returns `{ d, pts:[{x,y},…] }`); `bounds(rects)`; `box(id)`; `sel`; `editing`; `gesture`;
  `view`; `viewport`; `copySel(cut)`; `deleteSel()`; `commit()`; `renderLinks()`; `hasSel()`.
- Produces: `positionSelbar()`, `selLines()` (the selected link objects),
  `setSelStyle(curve: boolean)`, `setSelHeads(v: 'end'|'none'|'both')`, `reverseSel()`.

- [ ] **Step 1: Check it fails first**

```js
({ selbar: !!document.getElementById('selbar') })
```

Expected: `{ selbar: false }`.

- [ ] **Step 2: Add the markup**

After the `#islBL` element, add:

```html
<div class="isl" id="selbar" role="toolbar" aria-label="Selection" hidden>
  <span class="sb-lines">
    <button id="sbAngled" class="ib" type="button" title="Angled" aria-label="Angled"><svg viewBox="0 0 16 16" aria-hidden="true"><path d="M2 13h5a2 2 0 0 0 2-2V5a2 2 0 0 1 2-2h3" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round"/></svg></button>
    <button id="sbCurved" class="ib" type="button" title="Curved" aria-label="Curved"><svg viewBox="0 0 16 16" aria-hidden="true"><path d="M2 13C7 13 9 3 14 3" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round"/></svg></button>
    <span class="isep"></span>
    <button id="sbEnd" class="ib" type="button" title="Arrow at the end" aria-label="Arrow at the end"><svg viewBox="0 0 16 16" aria-hidden="true"><path d="M2 8h11M9.5 4.5L13 8l-3.5 3.5" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg></button>
    <button id="sbNone" class="ib" type="button" title="Plain line" aria-label="Plain line"><svg viewBox="0 0 16 16" aria-hidden="true"><path d="M2 8h12" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round"/></svg></button>
    <button id="sbBoth" class="ib" type="button" title="Arrows at both ends" aria-label="Arrows at both ends"><svg viewBox="0 0 16 16" aria-hidden="true"><path d="M2.5 8h11M5.5 4.5L2 8l3.5 3.5M10.5 4.5L14 8l-3.5 3.5" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg></button>
    <button id="sbReverse" class="ib" type="button" title="Reverse direction" aria-label="Reverse direction"><svg viewBox="0 0 16 16" aria-hidden="true"><path d="M2.5 5.5h10M10 3l2.5 2.5L10 8M13.5 10.5h-10M6 8l-2.5 2.5L6 13" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></svg></button>
    <span class="isep"></span>
  </span>
  <button id="sbCopy" class="ib" type="button" title="Copy (Ctrl/⌘ C)" aria-label="Copy"><svg viewBox="0 0 16 16" aria-hidden="true"><rect x="5.5" y="5.5" width="8" height="8" rx="1.5" fill="none" stroke="currentColor" stroke-width="1.5"/><path d="M10.5 3.5v-.5A1.5 1.5 0 0 0 9 1.5H4A1.5 1.5 0 0 0 2.5 3v5A1.5 1.5 0 0 0 4 9.5h.5" fill="none" stroke="currentColor" stroke-width="1.5"/></svg></button>
  <button id="sbDelete" class="ib" type="button" title="Delete (Backspace)" aria-label="Delete"><svg viewBox="0 0 16 16" aria-hidden="true"><path d="M3 4.5h10M6.5 4.5V3h3v1.5M4.5 4.5l.7 9h5.6l.7-9" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></svg></button>
</div>
```

CSS:

```css
/* Selection toolbar: floats just above the selection (below it if there is no room) */
#selbar{position:fixed;z-index:6;transform:translateX(-50%);gap:2px}
#selbar[hidden]{display:none}
#selbar .sb-lines{display:contents}
#selbar.no-lines .sb-lines{display:none}
#selbar .ib[aria-pressed="true"]{background:var(--accent-soft);color:var(--accent)}
body.gesturing #selbar{display:none}
```

- [ ] **Step 3: Implement**

Add next to `renderSel()`:

```js
/* ---------- selection toolbar ---------- */
const selLines = () => state ? state.links.filter(l => sel.links.has(l.id)) : [];
// Screen rectangle around the selection (boxes and the paths of selected lines)
function selScreenRect(){
  const rects = [];
  for (const id of sel.boxes){ const b = box(id); if (b) rects.push(b); }
  for (const l of selLines()){ const a = box(l.from), b = box(l.to); if (!a || !b) continue;
    for (const p of linkGeom(a, l.fa, b, l.ta, l.curve).pts) rects.push({ x:p.x, y:p.y, w:0, h:0 }); }
  const r = bounds(rects); if (!r) return null;
  const vr = viewport.getBoundingClientRect();
  return { l: vr.left + view.x + r.x*view.z, t: vr.top + view.y + r.y*view.z, r: vr.left + view.x + (r.x + r.w)*view.z, b: vr.top + view.y + (r.y + r.h)*view.z };
}
function positionSelbar(){
  const bar = $('selbar'); if (!bar) return;
  const r = (hasSel() && editing == null) ? selScreenRect() : null;
  bar.hidden = !r; if (!r) return;
  const ls = selLines();
  bar.classList.toggle('no-lines', !ls.length);
  if (ls.length){
    const all = f => ls.every(f);
    $('sbAngled').setAttribute('aria-pressed', all(l => !l.curve)); $('sbCurved').setAttribute('aria-pressed', all(l => l.curve));
    $('sbEnd').setAttribute('aria-pressed', all(l => l.arrow && !l.back)); $('sbNone').setAttribute('aria-pressed', all(l => !l.arrow && !l.back));
    $('sbBoth').setAttribute('aria-pressed', all(l => l.arrow && l.back));
    $('sbReverse').hidden = !ls.some(l => !!l.arrow !== !!l.back); // only one-way arrows can be reversed
  }
  const h = bar.offsetHeight || 40, w = bar.offsetWidth || 200, gap = 12, top = 64; // keep clear of the top islands
  let y = r.t - h - gap; if (y < top) y = r.b + gap;                          // no room above: go below
  y = Math.min(y, innerHeight - h - 64);
  const x = Math.max(w/2 + 8, Math.min(innerWidth - w/2 - 8, (r.l + r.r)/2));
  bar.style.left = x + 'px'; bar.style.top = y + 'px';
}
function setSelStyle(curve){ const ls = selLines(); if (!ls.length) return; ls.forEach(l => l.curve = curve); renderLinks(); commit(); positionSelbar(); }
function setSelHeads(v){ const ls = selLines(); if (!ls.length) return; const h = headsFrom(v);
  ls.forEach(l => { l.arrow = h.arrow; if (h.back) l.back = true; else delete l.back; }); renderLinks(); commit(); positionSelbar(); }
// Reverse a one-way arrow: swap its ends (lines with no arrow or arrows at both ends look the same either way)
function reverseSel(){ const ls = selLines().filter(l => !!l.arrow !== !!l.back); if (!ls.length) return;
  ls.forEach(l => { [l.from, l.to] = [l.to, l.from]; [l.fa, l.ta] = [l.ta, l.fa]; }); renderLinks(); commit(); positionSelbar(); }
```

Call `positionSelbar();` at the end of `renderSel()` (after `renderLinks();`) and at the end of
`applyView()` (after `saveView();`). `applyView` runs before `#selbar` exists on start-up, so
`positionSelbar` returns early when `$('selbar')` is null; that is why the first line checks it.
`state` may be undefined then too, which `selLines` handles.

Hide it during gestures: in the `pointermove` listener, change
`if (gesture) gesture.move(e); else nearDot(e);` to

```js
  if (gesture){ gesture.move(e); if (gesture && gesture.active) document.body.classList.add('gesturing'); } else nearDot(e);
```

and at the end of `cleanupGesture()` (before its closing brace) add
`document.body.classList.remove('gesturing'); positionSelbar();`.

Wire it (near the other button wiring):

```js
document.querySelectorAll('#selbar button').forEach(b => b.addEventListener('pointerdown', e => { e.preventDefault(); e.stopPropagation(); }));
$('sbAngled').onclick = () => setSelStyle(false);
$('sbCurved').onclick = () => setSelStyle(true);
$('sbEnd').onclick = () => setSelHeads('end');
$('sbNone').onclick = () => setSelHeads('none');
$('sbBoth').onclick = () => setSelHeads('both');
$('sbReverse').onclick = reverseSel;
$('sbCopy').onclick = () => copySel(false);
$('sbDelete').onclick = () => { if (hasSel()) deleteSel(); };
window.addEventListener('resize', positionSelbar);
```

`copySel(false)` puts the copy in the in-memory clipboard only. That is acceptable: ⌘/Ctrl+V on the
board pastes it, through the fallback path in `keyCopy`.

- [ ] **Step 4: Verify**

```js
const $q = s => document.querySelector(s);
const hit = $q('#links .hit'); const r = hit.getBoundingClientRect();
const fire = (el, type, o = {}) => el.dispatchEvent(new PointerEvent(type, { bubbles: true, cancelable: true, composed: true, pointerId: 1, pointerType: 'mouse', button: 0, buttons: type === 'pointerup' ? 0 : 1, isPrimary: true, ...o }));
// pick a point on the line: walk its path
const path = $q('#links .ln'), L = path.getTotalLength(), mid = path.getPointAtLength(L/2), ctm = path.getScreenCTM();
const sx = ctm.a*mid.x + ctm.e, sy = ctm.d*mid.y + ctm.f;
fire(document.elementFromPoint(sx, sy), 'pointerdown', { clientX: sx, clientY: sy }); fire(document.elementFromPoint(sx, sy), 'pointerup', { clientX: sx, clientY: sy });
await new Promise(r => setTimeout(r, 50));
const bar = $q('#selbar'), shown = !bar.hidden, lines = !bar.classList.contains('no-lines');
const before = $q('#links .ln.sel').getAttribute('d');
$q('#sbCurved').click(); const curved = $q('#links .ln.sel').getAttribute('d') !== before;
$q('#sbBoth').click(); const both = !!$q('#links .ln.sel').getAttribute('marker-start');
$q('#sbEnd').click(); const d1 = $q('#links .ln.sel').getAttribute('d'); $q('#sbReverse').click(); const reversed = $q('#links .ln.sel').getAttribute('d') !== d1;
$q('#undoBtn').click(); const undone = $q('#links .ln.sel') ? $q('#links .ln.sel').getAttribute('d') === d1 : 'deselected';
const br = bar.getBoundingClientRect();
({ shown, lines, curved, both, reversed, undone, aboveOrBelow: br.bottom <= sy || br.top >= sy })
```

Expected: `shown: true`, `lines: true`, `curved: true`, `both: true`, `reversed: true`,
`undone: true` (or `"deselected"`, because undo selects what it changed and may leave no line
selected), and `aboveOrBelow: true`. The toolbar must not cover the point you clicked.

Box-only selection: click inside the first box away from its text, for example 10px below its
top edge in the middle (by coordinates in a screenshot). Then check
`[!$q('#selbar').hidden, $q('#selbar').classList.contains('no-lines')]`. Expected: `[true, true]`.
Click `#sbDelete`; the box and its lines go. Undo brings them back.

While editing: click a box's text so it enters editing. `#selbar` must be hidden. Press Esc, and
the bar comes back with the box selected.

During a drag: start dragging a box (`left_click_drag`) and take a screenshot mid-drag if
possible; otherwise check that `document.body.classList.contains('gesturing')` is true during a
synthetic `pointermove` with buttons:1 after `pointerdown` on a box. The bar is hidden while the
class is set.

Near the top edge: drag the board (or a box) so the selected box is under the top islands. The
bar must appear *below* the box, not under the islands.

Screenshots: a selected line at the normal width and at 375px, and in dark mode.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Selection toolbar: line style, arrowheads, reverse, copy, delete (D-060)

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---

### Task 4: Welcome on an empty board

**Files:**
- Modify: `index.html`: the fonts `<link>` (add Caveat), CSS, markup (add `#welcome` before
  `#islTL`), `renderAll()` (~579), and the wiring block.

**Interfaces:**
- Consumes: `loadExample()` (Task 1), `setHelp(open)`, `isMac`, `state`.
- Produces: `updateWelcome()`, which shows `#welcome` when `state.boxes.length === 0`.

- [ ] **Step 1: Check it fails first**

```js
document.getElementById('clearBtn').click(); ({ welcome: !!document.getElementById('welcome') })
```

Expected: `{ welcome: false }`. Then undo (`document.getElementById('undoBtn').click()`).

- [ ] **Step 2: Load the handwritten font**

In the Google Fonts `<link>`, add `&family=Caveat:wght@500` before `&display=swap`, so the URL
contains `family=Atkinson+Hyperlegible:wght@400;700&family=Bricolage+Grotesque:opsz,wght@12..96,700&family=Caveat:wght@500&display=swap`.
Add the token `--f-hand:"Caveat", "Bradley Hand", cursive;` to `:root` (next to `--f-text`).

- [ ] **Step 3: Add the markup**

Before `<div class="isl" id="islTL">`, add:

```html
<div id="welcome" hidden>
  <div class="wl-center">
    <div class="wl-logo"><!-- copy of the .mark svg -->Explainer</div>
    <p class="wl-tag">Boxes, lines, and nothing in the way.</p>
    <ul class="wl-hints">
      <li><svg viewBox="0 0 16 16" aria-hidden="true"><rect x="2.5" y="3.5" width="11" height="9" rx="2" fill="none" stroke="currentColor" stroke-width="1.5"/><path d="M8 6v4M6 8h4" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/></svg>Double-click anywhere to add a box</li>
      <li><svg viewBox="0 0 16 16" aria-hidden="true"><path d="M2 8h11M9.5 4.5L13 8l-3.5 3.5" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg><span>Select a box, <kbd class="wl-mod">⌘</kbd>-click another to connect</span></li>
    </ul>
    <div class="wl-items">
      <button id="wlExample" type="button">Open the example board</button>
      <button id="wlHelp" type="button">Shortcuts</button>
      <a id="wlGit" href="https://github.com/andidev/explainer" target="_blank" rel="noopener">Source on GitHub</a>
    </div>
  </div>
  <div class="wl-note wl-menu">theme, capitals, grid…<svg viewBox="0 0 60 50" aria-hidden="true"><path d="M52 44 C 30 44 14 34 10 8 M4 15 L10 6 L16 14" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg></div>
  <div class="wl-note wl-help">shortcuts and help<svg viewBox="0 0 60 50" aria-hidden="true"><path d="M6 8 C 34 8 50 18 52 42 M45 36 L52 45 L58 36" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg></div>
</div>
```

- [ ] **Step 4: Add the CSS**

```css
/* Welcome on an empty board: it lets presses through to the board, except on its items */
#welcome{position:fixed;inset:0;z-index:1;pointer-events:none;color:var(--muted)}
#welcome[hidden]{display:none}
.wl-center{position:absolute;left:50%;top:46%;transform:translate(-50%,-50%);display:flex;flex-direction:column;align-items:center;gap:10px;width:min(360px,calc(100% - 32px));text-align:center}
.wl-logo{display:flex;align-items:center;gap:10px;font-family:var(--f-display);font-weight:700;font-size:34px;letter-spacing:-.02em;color:var(--ink)}
.wl-logo svg{width:36px;height:36px}
.wl-tag{margin:0 0 6px;font-family:var(--f-hand);font-size:24px;color:var(--muted)}
.wl-hints{list-style:none;margin:0;padding:0;display:flex;flex-direction:column;gap:6px;font-size:14px;align-self:stretch}
.wl-hints li{display:flex;align-items:center;gap:10px}
.wl-hints svg{width:16px;height:16px;flex:none}
.wl-hints kbd{font-family:var(--f-text);font-size:11.5px;border:1px solid var(--edge);border-bottom-width:2px;border-radius:4px;padding:0 4px;background:var(--box);color:var(--ink)}
.wl-items{align-self:stretch;display:flex;flex-direction:column;border-top:1px solid var(--edge);margin-top:6px;padding-top:6px}
.wl-items button,.wl-items a{pointer-events:auto;border:0;background:transparent;text-align:left;padding:7px 8px;border-radius:6px;font:inherit;font-size:14px;color:var(--ink);text-decoration:none;cursor:pointer}
.wl-items button:hover,.wl-items a:hover{background:var(--accent-soft);color:var(--ink)}
.wl-note{position:absolute;font-family:var(--f-hand);font-size:20px;line-height:1.1;display:flex;align-items:flex-end;gap:4px;color:var(--muted)}
.wl-note svg{width:52px;height:44px;flex:none}
.wl-menu{top:calc(66px + env(safe-area-inset-top,0px));left:22px;flex-direction:row-reverse}
.wl-help{bottom:calc(64px + env(safe-area-inset-bottom,0px));right:58px}
@media (max-width:640px){.wl-note{display:none}}
```

- [ ] **Step 5: Implement and wire**

Add after `renderAll()`:

```js
// The welcome shows whenever the board has no boxes
function updateWelcome(){ const w = $('welcome'); if (w) w.hidden = !!(state && state.boxes.length); }
```

Call `updateWelcome();` at the end of `renderAll()`, at the end of `deleteSel()`, and after a box
is created. Find where a new box is pushed (`newBoxAt` and the drag-to-draw `up` handler: search
for `state.boxes.push`) and call it after each push. It is cheap and idempotent. Wire it:

```js
document.querySelectorAll('#welcome .wl-mod').forEach(k => k.textContent = isMac ? '⌘' : 'Ctrl');
['wlExample','wlHelp'].forEach(id => $(id).addEventListener('pointerdown', e => e.stopPropagation()));
$('wlExample').onclick = loadExample;
$('wlHelp').onclick = () => setHelp(true);
```

Put this after `isMac` is defined. In the help close listener, extend the selector to
`'#help, #helpBtn, #mHelp, #wlHelp'`.

- [ ] **Step 6: Verify**

```js
const $q = s => document.querySelector(s);
$q('#clearBtn').click(); await new Promise(r => setTimeout(r, 50));
const shown = !$q('#welcome').hidden;
const c = $q('.wl-center').getBoundingClientRect(); const x = c.left + 20, y = c.top + 10; // over the logo, not an item
const passes = document.elementFromPoint(x, y).closest('#welcome') === null;
$q('#wlExample').click(); await new Promise(r => setTimeout(r, 50));
({ shown, passes, afterExample: document.querySelectorAll('.box').length, hidden: $q('#welcome').hidden })
```

Expected: `shown: true`, `passes: true` (the logo area lets presses through to the board),
`afterExample: 3`, `hidden: true`.

Create a box through the welcome screen: Clear board, then double-click the middle of the window
(`computer` `double_click` over the tagline). A box appears with the caret in it, and the welcome
hides. Press Backspace: the empty box is deleted (D-049) and the welcome comes back.

Screenshots: the welcome at the normal width, at 1440×800, at 375×812 (no hint notes, nothing
overlapping the islands), and in dark mode.

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "Welcome screen on an empty board (D-061)

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---

### Task 5: Decisions, README, shortcuts panel and screenshots

**Files:**
- Modify: `DECISIONS.md`, `README.md`, `index.html` (`#help` list only), `CLAUDE.md` (the code
  map line about header/gestures if it is stale), `docs/screenshot-light.png`, `docs/screenshot-dark.png`.

**Interfaces:**
- Consumes: everything above. The text describes behaviour exactly as built; read the code
  before writing.

- [ ] **Step 1: Append decisions D-057 … D-062 to `DECISIONS.md`**

Use the template at the top of the file. Each entry has Status Active, Date 2026-10-13, Decision,
Why ("User chose the floating layout inspired by Excalidraw, …"), and Code. Write:

- **D-057, "Floating islands replace the header"**, Amends D-055. Islands top-left (☰, mark,
  wordmark, Example board tag), top-centre (style for new lines), top-right (GitHub, Share),
  bottom-left (zoom −/%/+, Fit board, Undo, Redo, Edits), and **?** bottom-right. On phones
  the style island sits under the top row, and Fit board and Edits move into the menu. The board
  fills the window.
- **D-058, "Quiet Share button"**: island surface, accent icon, soft accent on hover; not a
  solid colour block.
- **D-059, "☰ menu"**: Theme, Capitals, Snap to grid (these rows stay open to cycle); Open the
  example board; Clear board; Shortcuts; Source on GitHub; phone-only Fit board and edit places.
  Closes on Esc or a click outside.
- **D-060, "Selection toolbar"**, Amends D-036. Above the selection (below it near the top);
  lines: Angled/Curved, arrow/plain/both, Reverse (one-way arrows only); always Copy and Delete;
  each change is one undo step; hidden while dragging or editing. The top-centre island no longer
  changes selected lines.
- **D-061, "Welcome on an empty board"**: what it shows, that presses pass through, and that
  the first visit still opens the example board (D-053).
- **D-062, "Arrowheads for new lines"**, Amends D-052. Setting `newHeads`, share key `a`
  (0/1/2, absent = arrow), and that Shift/Shift+Ctrl override it.

Then update the Status lines: D-055 → `Amended by D-057`; D-036 → `Amended by D-060`; D-052 →
`Amended by D-062`. Check each one's current Status first and append to an existing
"Amended by" list rather than replacing it.

- [ ] **Step 2: Update the README**

- "How it works": rewrite **Arrange** or add a short **Change a line** paragraph about the
  selection toolbar. Mention the ☰ menu (theme, capitals, grid) and the top-centre style for new
  lines. Replace the sentence that says "the **All caps** button" with "Capitals, in the ☰ menu".
- Shortcuts table: no key changes. Add one row: `| Change a selected line (style, arrowheads, reverse) | Use the toolbar above it |`.
- "Project files" table: unchanged.

- [ ] **Step 3: Shortcuts panel (`#help`)**

Add one row after "Hold ⇧ ⌃ while dropping": `<dt>Select a line</dt><dd>Toolbar: style, arrows, reverse</dd>`.
Change `<dt>Angled / Curved</dt><dd>Style of selected + new lines</dd>` to
`<dt>Top island</dt><dd>Style of new lines</dd>`.

- [ ] **Step 4: `CLAUDE.md`**

In the "Code map", add one line:
`- Chrome: floating islands (#islTL/#islTC/#islTR/#islBL), ☰ #menu, #selbar (positionSelbar), #welcome (updateWelcome).`
Nothing else changes.

- [ ] **Step 5: Screenshots**

Replace `docs/screenshot-light.png` and `docs/screenshot-dark.png` with the new UI: a board with
4–6 connected boxes explaining something (as in the current screenshot), one line selected so
the toolbar shows, at 1440×900. If you cannot capture screenshots with your tools, leave the files
unchanged and say so in your report; the controller will capture them.

- [ ] **Step 6: Verify**

```bash
grep -c '^### D-0' DECISIONS.md
grep -n 'Status: Amended by' DECISIONS.md | grep -E 'D-055|D-036|D-052' -n
grep -n 'All caps\*\* button' README.md
```

Expected: the count rises by 6 (from 56 to 62). The three amended entries show their new
"Amended by" values when you read them. The last grep prints nothing.

Open the app, open **?**, and confirm the two changed rows read correctly.

- [ ] **Step 7: Commit**

```bash
git add DECISIONS.md README.md CLAUDE.md index.html docs/screenshot-light.png docs/screenshot-dark.png
git commit -m "Record D-057..D-062; update README, shortcuts panel and screenshots

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---

## After all tasks (controller)

Run the full manual check from `CLAUDE.md` (draw a box and type, connect two boxes, undo/redo,
refresh, dark mode). Also open a share link made with the version live before this work: copy one
from https://andidev.github.io/explainer/ (Share → Whole board), then open the local `index.html`
with that `#…` hash appended. The board must load. Then push `main`, wait for the Pages build, and
publish release v1.1.0 (spec §10).
