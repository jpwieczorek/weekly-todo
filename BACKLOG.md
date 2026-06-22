# Jeffrey's Weekly List — Backlog

> Canonical tracker for features and fixes. Fetch this at the start of any session
> (raw URL below), update it as items move, and deliver a complete replacement file
> with a commit message whenever status changes.
>
> Raw: `https://raw.githubusercontent.com/jpwieczorek/weekly-todo/main/BACKLOG.md`

_Last updated: 2026-06-22_

---

## ✅ Done (verified in code)

- **Firebase Auth** — Google sign-in via `signInWithPopup`, `onAuthStateChanged` gate, sign-out.
- **Auto-collapse past days** — past days start collapsed; today + future start open.
- **Work/personal category filter** — `filterMode` toggle with color coding.
- **Google Calendar sync** — OAuth read-only, localStorage cache by date key.
- **Drag-and-drop** — desktop HTML5 + mobile touch (400ms long-press), auto-expand on hover.
- **Inline editing** — double-click / double-tap.
- **Weekly reset with carry-over** — incomplete tasks roll forward; completed get archived.
- **Persistent sort/filter state** — filter + sort modes restored from localStorage (`wktodo_filter`,
  `wktodo_sort`) on load and written on change. _Patch + `node --check` verified; pending browser check._
- **Archive history view** — `archived_tasks` readable via a modal (opened from the Account modal),
  queried newest-first with `getDocs`/`orderBy`/`limit`. _Patch + `node --check` verified; pending browser check._
- **Header streamline** — toolbar reduced to All · Sort · Sync cal · Account; Archive history and
  Clear completed relocated into the Account modal; Firebase UID display + Copy UID button removed
  (security-rules setup complete). _Patch + `node --check` verified; pending browser check._

## 🟢 Quick wins (low risk, surgical)

- _(empty — both shipped 2026-06-22; pull new ideas up from below as needed)_

## 🟡 Medium lifts

- **Task notes field** — `saveTask` already spreads `...task`, so a `notes` property persists
  for free. Work is UI: entry/view (via the existing edit interaction) + a row indicator.
- **Swipe-to-delete (mobile)** — good payoff on the primary surface, but must share the touch
  surface with the 400ms long-press drag. Gesture arbitration is the risk area.

## 🔴 Bigger projects

- **Refactor `render()`** — ~290 lines (currently ~663–953). No behavior change, but it gates
  the risk of everything else. Do this *before* recurring tasks.
- **Recurring tasks** — recurrence model + generation logic that cooperates with the weekly
  reset/carry-over without duplicating. Largest data-model change; most regression-prone.

## ⚠️ Needs verification (not code — console / infra)

- **Firestore security rules** — notes conflict on whether these are locked to the Firebase UID
  with default-deny, or still in test mode. Verify in the Firebase console; an expired test-mode
  rule flips the database open or fully locked depending on the default.
- **GitHub MCP for direct commits** — blocked by integration type mismatch; deployment stays
  manual via the GitHub web editor for now.

---

## Maintenance convention

- Fetch this file at session start alongside `index.html`.
- When an item ships: move it to **Done** with a one-line note on what was verified.
- When a new idea comes up: add it to the appropriate tier.
- Deliver the full updated `BACKLOG.md` + a commit message in the same response — never partial diffs.
- "Done" means verified (in code or by browser check), not merely "patch applied."
