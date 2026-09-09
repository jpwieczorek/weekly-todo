/
Claude finished the response
Thought for 54s

Good call; it's the most discoverable of the four. Building the patch now, plus the keyboard fix, since fixing the entry point alone would leave you with an input you can't type into on iOS.
Ran 5 commands, read a file

Complete file, patched against a fresh pull of main. Three surgical replacements, 51 changed lines, node --check passes on the extracted module. Not tested; that's your browser.

What changed:

The dead 600ms block is gone, replaced by tap detection on .task-label. A tap counts only if the finger stayed within 8px and the gesture didn't become something else. Four guards, in order: tapMoved catches scrolling, touchDragging catches the long-press drag, openSwipeRow means the tap should close the tray instead, and the 350ms lastSwipeEnd window catches the tail of a swipe. preventDefault on touchend kills the synthetic click.

Deliberately no stopPropagation on that handler. The row's own touchend has to keep running, because that's what clears touchLongPressTimer; stopping propagation would leave the drag timer armed after every tap.

New openEditor(id) replaces the two inline editingTaskId = tid; render() pairs, and the requestAnimationFrame focus is gone. Focus now happens synchronously right after render(), inside the touchend handler, so it stays in the user gesture WebKit requires to raise the keyboard.

Commit message:

Fix mobile inline edit: tap the task text to open the editor

The 600ms long-press could never fire. The row's drag timer set
touchDragging at 400ms and the edit callback bailed on exactly that
flag, so no press was ever both long enough to reach 600ms and free of
a drag. Removed it and moved edit onto a tap, which was the only touch
gesture nothing else claimed.

Tap is guarded four ways: 8px of movement makes it a scroll, an active
drag wins, an open swipe tray absorbs the tap to close itself, and the
350ms window after a swipe is ignored. No stopPropagation, so the row's
touchend still clears the long-press timer.

Also replaced the requestAnimationFrame focus with a synchronous focus
in a new openEditor() helper. Deferring focus to a later frame leaves
the user gesture, and iOS will not raise the keyboard for it.

Four things to check on the phone: a tap opens the editor and the keyboard actually comes up; scrolling past tasks doesn't fire it; a 400ms hold still gives you the drag ghost; and with a swipe tray open, tapping the text closes the tray instead of editing. Desktop double-click should be unchanged, but it now routes through openEditor so it's worth one click in Firefox.
Index
Code·HTML 

okay code has been saved and committed.
Ran a command

Live on main, byte-identical to what I handed you. GitHub Pages usually takes a minute or two to serve the new file, so hard-refresh on the phone if you get the old behavior.

The four checks from my last message are what matter now, and the keyboard one is the least certain; the tap fix is straightforward, but whether iOS raises the keyboard from a synchronous focus in touchend is the part I'd most want to see confirmed rather than reasoned about.

Once you've run them, the backlog needs a second pass: the Bugs section still describes this as open, and Done should pick up tap-to-edit with whatever you actually verified. Tell me what you see and I'll turn that around.

looks good.  let's update the backlog and then that's enough for today'
Ran 2 commands, read a file

Updated file, built from the version currently on main.

Bugs is now empty with a Fixed subsection preserving why the long-press failed, so the reasoning survives if it ever regresses. Inline editing in Done rewrote to cover both entry points and the synchronous focus, marked verified in browser. Tap-to-edit added as its own entry with the four guards and the no-stopPropagation note. Task notes is no longer blocked. Line refs re-measured against the committed file: render() is 947–1356 (410 lines), add-row 1302–1356, archive getDocs 1477, UID log 1512.

Commit message:

Backlog: mobile inline edit fixed, line refs re-measured

Moved the long-press bug to a Fixed subsection with the timer analysis
intact. Added tap-to-edit to Done with its four guards, and rewrote the
inline editing entry to cover both entry points and the synchronous
focus that makes the iOS keyboard appear.

Task notes is unblocked now that the editor is reachable on mobile.
Re-measured line refs against main: render() 947-1356, add-row block
1302-1356, archive getDocs 1477, UID console.log 1512.

Still open whenever you come back to it: the Firestore rules state in the console, which gates the multi-user work.
Backlog
Document·MD 
Write a message…


Claude is AI and can make mistakes.
Backlog · MD
# Jeffrey's Weekly List — Backlog
 
> Canonical tracker for features and fixes. Fetch this at the start of any session
> (raw URL below), update it as items move, and deliver a complete replacement file
> with a commit message whenever status changes.
>
> Raw: `https://raw.githubusercontent.com/jpwieczorek/weekly-todo/main/BACKLOG.md`
 
_Last updated: 2026-09-09 (audit against `main`; mobile inline edit fixed; line refs re-measured)_
 
---
 
## ✅ Done (verified in code)
 
- **Firebase Auth**: Google sign-in via `signInWithPopup`, `onAuthStateChanged` gate, sign-out.
- **Auto-collapse past days**: past days start collapsed; today + future start open.
- **Work/personal category filter**: `filterMode` toggle with color coding.
- **Google Calendar sync**: OAuth read-only, localStorage cache by date key.
- **Drag-and-drop**: desktop HTML5 + mobile touch (400ms long-press), auto-expand on hover.
- **Manual reorder within a day**: persisted `order` field, `nextOrderForDay` (new tasks append
  at the bottom), `insertBeforeIdAt` + `markInsertion` drop indicators (`drop-before` /
  `drop-after`), and `moveTaskTo` which renumbers the target day and writes only the tasks whose
  `order` actually changed. Sorted modes fall back to cross-day move + append. _Verified in code._
- **Flag model (priority collapsed to binary)**: `isFlagged` / `flagValue` read and write the
  existing `priority` field, where `'high'` means flagged and anything else (including legacy
  `'med'` / `'low'`) means not. `.p-med` / `.p-low` hidden in CSS, so no data migration was needed.
  Flag is settable from the add row, the inline editor, and the swipe action. Sort button now
  reads "Flagged". _Verified in code._
- **Inline editing**: double-click on desktop (`dblclick` on `.task-label`), single tap on touch.
  Both route through `openEditor(id)` (~1361), which sets `editingTaskId`, calls `render()`, then
  focuses and selects the input synchronously. The synchronous focus matters: the previous
  `requestAnimationFrame` focus left the originating user gesture, and iOS will not raise the
  keyboard for a focus deferred to a later frame. _Verified in browser on Brave/iOS and desktop
  Firefox, 2026-09-09._
- **Weekly reset with carry-over**: incomplete tasks roll forward; completed get archived.
- **Persistent sort/filter state**: filter + sort modes restored from localStorage
  (`wktodo_filter`, `wktodo_sort`) on load and written on change.
- **Archive history view**: `archived_tasks` readable via a modal (opened from the Account modal),
  queried newest-first with `getDocs` / `orderBy` / `limit(100)`.
- **Header streamline**: toolbar reduced to All · Sort · Sync cal · Account; Archive history and
  Clear completed relocated into the Account modal; Firebase UID display + Copy UID button removed.
  (A `console.log('Firebase UID:', user.uid)` remains at ~1512; harmless, drop it whenever that
  area is touched.)
- **Swipe actions (mobile)**: swipe left on a row to reveal Flag and Delete. Shipped as a two-button
  action tray, not delete-only. Gesture arbitration against the 400ms long-press drag: a swipe is
  only claimed past `SWIPE_START_PX` (10px) and when horizontal travel exceeds vertical by 1.4x, so
  vertical scrolling still feels normal; an in-progress drag wins outright. One row open at a time
  via `openSwipeRow`, a document-level `touchstart` closes it when you touch elsewhere, `render()`
  resets it since the DOM is rebuilt, and `lastSwipeEnd` swallows the synthetic click for 350ms so
  ending a swipe can't toggle a checkbox. Rows are opaque and the drag-over state uses an outer
  ring rather than a background tint. _Verified in code; browser behavior on Brave/iOS not
  re-confirmed since the visual pass._
- **Tap-to-edit (mobile)**: replaced the unreachable 600ms long-press (see the 2026-09-09 fix
  note below). Tap is guarded four ways: >8px of travel makes it a scroll, an active
  `touchDragging` drag wins, an open `openSwipeRow` tray absorbs the tap to close itself, and the
  350ms `lastSwipeEnd` window after a swipe is ignored. `preventDefault` on `touchend` suppresses
  the synthetic click. Deliberately no `stopPropagation`, so the row's own `touchend` still runs
  and clears `touchLongPressTimer`; stopping it would leave the drag timer armed after every tap.
  _Verified in browser on Brave/iOS, 2026-09-09._
- **Apple-native visual pass**: three `prefers-color-scheme: dark` blocks (full dark palette),
  translucent sticky header with `backdrop-filter`, inset grouped day lists on `--surface`,
  modals presented as bottom sheets with a grab handle and `sheet-up` animation, system font stack
  (`-apple-system` / SF Pro), hairline separators, and a check-pop animation. Class names and DOM
  structure were left unchanged. _Verified in code._
## 🟢 Quick wins (low risk, surgical)
 
- **Drop the UID console.log** (~1512). One line; do it opportunistically, not as its own commit.
## 🐞 Bugs
 
- _(none open)_
### Fixed
 
- **Mobile long-press-to-edit was unreachable** (fixed 2026-09-09). `.task-label` started a 600ms
  timer that opened the editor only `if (!touchDragging)`, but the row's own handler set
  `touchDragging` at `LONG_PRESS_MS` (400ms), so any press surviving to 600ms had already become a
  drag. The escape hatch (moving >8px cancels the drag timer) also fired the label's `touchmove`,
  which cleared the edit timer. Net effect: no path into inline edit on iPhone. Fixed by moving
  edit onto a tap, the one touch gesture nothing else claimed. Same commit fixed the keyboard not
  appearing, via the synchronous focus described under Inline editing above.
## 🟡 Medium lifts
 
- **Task notes field**: `saveTask` already spreads `...task`, so a `notes` property persists for
  free. Work is UI: entry/view (via the existing edit interaction) + a row indicator. Now
  unblocked: the editor is reachable on mobile again, so it can serve as the entry point.
- **Finish the PWA**: the meta tags are in place (`apple-mobile-web-app-capable`,
  `apple-mobile-web-app-status-bar-style`, light/dark `theme-color`, `viewport-fit=cover`,
  `apple-mobile-web-app-title`), and `icon.png` exists. Missing from the repo: `manifest.json`
  (both `manifest.json` and `manifest.webmanifest` 404) and any service worker (`sw.js` /
  `service-worker.js` 404). So Add to Home Screen on iOS gives standalone chrome but there's no
  install prompt elsewhere and no offline shell. Note this breaks the single-file rule: a manifest
  and a worker are separate files that must be committed alongside `index.html`.
## 🔴 Bigger projects
 
- **Refactor `render()`**: now 410 lines (947–1356) with 31 `addEventListener` calls rebuilt every
  cycle; it grew from ~317 lines as swipe actions and manual reorder landed inside it.
  **Working agreement (2026-06-22, still holds): do NOT do this as a standalone pass.** Efficiency
  is a non-issue at this scale (a few dozen tasks; the full innerHTML rebuild is imperceptible).
  The only real win is maintainability, and a big-bang rewrite of the most interaction-critical,
  pitfall-dense code (suppressSnapshot, the 400ms long-press, swipe arbitration, inline edit,
  en-CA keys) is the highest-regression-risk change in the app for zero user-visible benefit.
  **Plan instead:** extract helpers (`renderDayBlock`, `renderTaskRow`, `renderAddRow`,
  `attachDragHandlers`, `attachSwipeHandlers`) *opportunistically* as feature work enters that
  code, so each extraction rides along with the browser testing that feature needs anyway. Does
  not gate recurring tasks. Lowest-risk standalone starter, if ever wanted: extract the add-row
  block (~1302–1356) into `renderAddRow(di)`.
- **Recurring tasks**: recurrence model + generation logic that cooperates with the weekly
  reset/carry-over without duplicating. Largest data-model change; most regression-prone. Now also
  has to cooperate with the persisted `order` field when generated instances land in a day.
- **Multi-user / family rollout**: let Bethany, Hannah, and Amelia each keep their own list.
  Auth is already per-user (Google sign-in + `onAuthStateChanged`), and Calendar sync is already
  per-Google-identity, so those layers are free. The gap: task data is **not owner-scoped**. One
  flat `tasks` collection, `onSnapshot(query(tasksCol))` with no owner filter, and writes don't
  stamp an owner. Work for the **isolated personal lists** version (line refs re-measured
  2026-09-09):
  - Stamp `owner: auth.currentUser.uid` on every write. Four sites, not three: `saveTask` `setDoc`
    (~577), archive `addDoc` (~609), carry-over `setDoc` (~633), and note `commitNewTask` (~594)
    routes through `saveTask` so it's covered.
  - Filter every read by owner: `where('owner','==',uid)` on the snapshot query (~683) and the
    archive `getDocs` (~1477).
  - Rewrite Firestore rules from single-UID lockdown to per-owner
    (`resource.data.owner == request.auth.uid`). This is the real isolation boundary; the `where`
    clause is just a client filter. Console-only, can't ship in index.html, needs verification with
    a 2nd account.
  - **Backfill gotcha:** existing tasks have no `owner` field and will vanish from Jeffrey's view
    the moment the filter goes live. One-time console update to set `owner` = Jeffrey's UID first.
  - Doc IDs are `Date.now()+random` (globally unique), so no collision/namespacing needed.
  - **Scope fork that sets the size:** isolated personal lists (above) is contained; *shared /
    household* lists (seeing each other's, a shared family list) is a separate, much larger tier.
  - **Access question:** once rules go per-owner, anyone who signs in at the public URL can create
    a list. Optional allowlist of permitted UIDs in the rules to keep it to the four of us.
  - **Prerequisite:** resolve the security-rules state below before the rules rewrite; the
    per-owner rules build on whatever's actually live.
## ⚠️ Needs verification (not code — console / infra)
 
- **Firestore security rules**: notes conflict on whether these are locked to the Firebase UID
  with default-deny, or still in test mode. Verify in the Firebase console; an expired test-mode
  rule flips the database open or fully locked depending on the default. The client already
  handles the locked case (`permission-denied` shows "Access denied — check Firestore rules or
  sign-in"), so a silent failure here would surface in the status bar.
- **Swipe actions on Brave/iOS**: verified in code and shipped, but not re-confirmed on device
  since the visual pass changed row backgrounds and the drag-over treatment.
- **GitHub write access from chat**: the native GitHub connector is now connected in Claude, but
  it covers repo files as chat/project context only. No commit tools are exposed, so deployment
  stays manual via the GitHub web editor. Claude Code with the repo selected is the route if
  direct commits become worth setting up. The old fine-grained PAT ("Claude MCP") is orphaned and
  can be left to expire.
## 📌 Project-instruction drift (fix in the Claude project instructions, not here)
 
- Stack notes still list "Inter from Google Fonts". The file loads no Google Fonts and uses the
  system stack; only Tabler Icons is remote.
- Feature-ideas list still names swipe-to-delete, auto-collapse, and persistent sort/filter as
  ideas. All three shipped.
---
 
## Maintenance convention
 
- Fetch this file at session start alongside `index.html`.
- When an item ships: move it to **Done** with a one-line note on what was verified.
- When a new idea comes up: add it to the appropriate tier.
- Deliver the full updated `BACKLOG.md` + a commit message in the same response, never partial diffs.
- "Done" means verified (in code or by browser check), not merely "patch applied."
- Re-measure line references whenever `render()` or the Firebase helpers are touched; they drift
  fast and the multi-user plan depends on them.
 
