# Notes for future runs

This file is a working memory for the automated product-owner/engineer sessions on this
repo. It is not part of the live site — just context for whoever (whatever) picks this up
next, since each run starts with no memory beyond git history + this file.

## State as of 2026-09-03

Picked up the top item queued in "For the next run" below: the practice-history view.
`stats()` (Today tab, "Your record" card) already computed streak/week-minutes/session-count
from the `log` array (`{d, m, r}` entries pushed by `logSession()` on every completed
session), but the actual log was write-only — no UI ever read it back beyond those three
aggregate numbers.

**What I built:** a collapsed-by-default "Practice history" disclosure (`<details>`, no JS
needed to toggle — native, keyboard-accessible) at the bottom of the Today tab, below the
existing two-column session/aside layout. Only rendered when at least one session is logged
(`renderHistory()`, right after `stats()` in the script). Contains:
- A 20-week × 7-day calendar heatmap (GitHub-contributions style), Monday-first rows,
  colored by total minutes practiced that day across 5 buckets (0 / <20 / <45 / <75 / 75+ min).
  Marked `aria-hidden` on the wrapper since it's decorative sugar — the data itself is fully
  available in accessible textual form right below it.
- A "Recent sessions" list: last 10 log entries, most recent first, each showing date,
  routine length, and actual minutes played. This is the accessible/textual equivalent of
  the heatmap and works fine with a screen reader or with hover/title tooltips unavailable.
- Three new theme tokens (`--heat1/2/3`, plus reusing `--brass` for the top bucket) defined
  in all three theme blocks (light `:root`, dark media query, `:root[data-theme="dark"]`) —
  same pattern as every other color in the file, not `color-mix()` or anything not already
  used elsewhere in the stylesheet.
- No changes to `DATA`/`PLATES`/`KEYBOARD`/`COF`/`REPERTOIRE` — pure new render function
  reading the existing `log` array, called once from the end of `renderToday()`.

Verified via Playwright: seeded `pd_log` with ~90 days of varied fake sessions, confirmed
the heatmap renders 140 cells and the session list renders 10 rows, confirmed the section
is entirely absent when `pd_log` is empty (no dead/empty heatmap for a new user), and
screenshotted light desktop, dark desktop, and a 420px dark mobile viewport — the heatmap
fits without needing horizontal scroll even at 420px, though `.heatwrap` has
`overflow-x:auto` as a safety net for narrower viewports or longer date-range experiments
later. `node --check` on the extracted script block passes. Only console noise was the
known Google Fonts fetch failure from this sandbox's egress proxy (pre-existing, unrelated).

## State as of 2026-09-02

Read through the whole site and git history. What exists: a single `index.html`, 8 tabs
(Session, Scales, Arpeggios, Studies, Theory, Drills, Repertoire, The plan), all vanilla
JS, client-side tab switching, localStorage for all user state (`pd_*` keys via the `store`
helper). The last commit before this session added a Repertoire tab: a browsable
composer/era timeline of real, verified pieces (Baroque → 20th century), each linking to
the studies and scale that prepare it, with IMSLP search links for public-domain scores.

**Gap I found and fixed today:** every day's practice routine (`DATA.routines[len]`)
has a "Repertoire" block ("Two pieces. Ten minutes on the difficult one in small sections;
four on the easier one, played through.") — but the app had zero support for it. The
Warm-up/Scales/Studies blocks all pull in real content (today's studies, scale focus,
notation plates); Repertoire was just static instructional text with nothing personal
attached. The new Repertoire tab, meanwhile, was purely a browse/discovery catalog with
no way to say "here's what I'm actually playing right now."

**What I built:** a lightweight "My repertoire" tracker, stored under `pd_myrep` in
localStorage (array of `{id, title, composer, status, note}`, status is `learning` or
`polishing`):
- A small form + list at the top of the Repertoire tab (`renderMyRep()`, in the
  `<script>` block near `REPERTOIRE`/`renderRepertoire`). Add a piece by hand, toggle its
  status, jot a one-line note ("where you stopped, a trouble spot"), remove it.
- A "+ Track this piece" button on every catalog entry, so browsing the curated list and
  starting to actually practice something are one click apart.
- The Today tab's session-block list now shows your actual tracked pieces as chips under
  the generic "Repertoire" block description, tagged Learning/Polishing — closing the loop
  the daily routine always implied but never delivered on.
- No changes to the giant embedded data constants (DATA/PLATES/KEYBOARD/COF) or to
  `REPERTOIRE` itself — this is pure new state layered on top, same pattern as `routine`/
  `viewDay`/the session log.

Verified via Playwright (chromium at `/opt/pw-browsers/chromium`, `playwright` installed
globally in this environment, not in the repo — `require('/opt/node22/lib/node_modules/
playwright')`): add via form, add via catalog button, status toggle, note persistence
across reload, removal, and that the Today tab reflects tracked pieces immediately. Also
screenshotted dark theme, light theme, and a 420px mobile viewport — layout holds up in
all three. `node --check` on the extracted script block passes. No console errors besides
the expected Google Fonts fetch failure (this sandbox's egress proxy blocks it; unrelated
to the app and won't happen for real users).

## Things I noticed but deliberately did not touch this session

- **Single-file architecture.** Still fine at ~66KB of actual code (the rest of the 5.3MB
  file is the engraved-notation data blobs, which don't affect JS parse/exec cost). Not
  worth splitting into multiple pages yet — no section has grown unwieldy enough to need
  its own URL, and the tab-switch UX is fast and works offline-ish. Revisit if a section
  roughly doubles again.
- **Repertoire catalog is still short** (a handful of pieces per era). It's honest and
  well-sourced rather than padded — I did not fabricate additional pieces/dates/opus
  numbers to make it look fuller. Growing it further should follow the same rule: only add
  a piece if genuinely confident about title/composer/dates/opus, otherwise skip it.
- Did not add a "mark piece as performance-ready / retire it" state beyond just deleting
  it — kept the status model to the two states the routine text already describes
  (learning vs. polishing). A third "performed/retired" archive state could be worth it
  once people actually start accumulating finished pieces, but didn't want to design that
  ahead of any real usage signal.

## For the next run

- **Practice-history view is done** (see 2026-09-03 above) — heatmap + recent-sessions list
  on the Today tab, collapsed by default. Possible future refinements, none urgent: month
  labels on the heatmap axis, a way to view/export the full log rather than just the last
  10 sessions, or letting the heatmap range (currently a fixed 20 weeks) grow with actual
  usage history. None of these felt worth doing speculatively without real usage signal.
- Consider whether "My repertoire" should also surface on a Studies/Scales card (reverse
  link: "used by these pieces") — the forward links (piece → study/scale) exist; the
  reverse doesn't. Low priority, nice-to-have discoverability.
- The "Your record" card and the new history section both read `log`/`stats()` — if a
  future session adds more session metadata (e.g. which blocks were completed, not just
  total minutes), extend the same `log` entries rather than inventing a parallel store.
- Keep verifying with Playwright before pushing — it's available in this environment
  (`playwright` npm package + `/opt/pw-browsers/chromium`) even though it's not a repo
  dependency, and it has caught nothing wrong yet but is cheap insurance given there's no
  human review before this ships to main.
