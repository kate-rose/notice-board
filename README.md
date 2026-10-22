# Notice Board

A plain-language guide to public notices in Skamania and Klickitat counties:
what is coming up for a decision where you live, and when you can still speak up.
Built to sit in an iframe on skamaniadispatch.com (Ghost), hosted on GitHub Pages.

No build step. One HTML file plus three JSON files. No credentials anywhere.

**This is not an official notice.** Legal notices run in each county's official
newspaper (the Skamania County Pioneer; the Goldendale Sentinel and The Enterprise
in Klickitat). The page says so in its footer, on every view.

## Files

- `index.html`: the whole page. Vanilla JS, no dependencies.
- `data/notices.json`: the notices. Each one links its official source.
- `data/places.json`: "where do you live" → the governments whose notices apply there.
  Hand-mapped for now; every place is `boundary_verified: false` until it is
  rebuilt from county tax-code-area data (plan milestone M5).
- `data/statutes.json`: minimum notice periods from state law. A timing line appears inside "About this notice" only for entries marked `vouched` (read on app.leg.wa.gov and approved by Kate). `proposed` entries never render.
- `data/sources.json`: what the board checks and when. Rendered as the
  "Where this comes from" block, so an empty list never reads as "nothing planned."

The plan, the data rules and the milestones live in Ferric:
`~/Ferric/drafts/2026-09-24_notice-board-plan.md`.

## Two kinds of card

- **Curated cards** are written by hand: a plain-language title, how to weigh in,
  the places a notice concerns. They change only when a person edits them. **Their
  truth lives in Ferric** (`~/Ferric/reference/notice-board/notices.json`, with
  `places.json`, `sources.json` and `statutes.json` beside it); `data/` here is
  the published copy and is overwritten on every sweep. Edit the truth, not the copy.
- **"As printed" cards** are the legal notices as the two county newspapers printed
  them (the Skamania County Pioneer for Skamania County, the Goldendale Sentinel
  for Klickitat County), with the dates they ran. They carry an "As printed" chip
  and are not rewritten or checked; the agency's own posting is the record. They
  are produced by Ferric's `legals.py --board` and merged in by `notice_publish.py`
  after each twice-weekly sweep, which also attaches "Printed in …" to a curated
  card when the papers ran the same notice. `notices.ics` (subscribe in a
  calendar) and `feed.xml` (RSS) are written at the same time.

## Data rules that matter

- A date shows as scheduled only if it comes from the agency's own notice or a
  typed record. A date someone said out loud at a meeting shows under
  "On the horizon" with the hedge visible.
- Deadlines worked out from a rule ("20 days from the date of this notice") say
  so. If one lands on a day the office is closed, the card suggests the last open
  day before it ("to be safe") and still shows the county's own date.
- File numbers are shown as printed on the notice. Where the county's web page
  lists a different number, the card says so.

## How it updates

```bash
cd ~/Ferric && python3 legals.py --write        # capture the papers (scheduled Thu + Mon)
python3 notice_publish.py                      # dry run: what would change
python3 notice_publish.py --apply --push        # write data/, notices.ics, feed.xml; commit; push
```

GitHub Pages redeploys on push. The Ghost page needs no change.

## Preview

```bash
cd ~/notice-board && python3 -m http.server 5192
```

Then open http://localhost:5192/ (try `?place=underwood`, `?theme=amber`).

## Options

- `?place=<id>`: preselect a place (ids in `data/places.json`).
- `?theme=amber`: the amber CRT look. Light is the default for legibility.
- `?embed=1`: hides the page's own title (use inside Ghost, which has one).
- `?view=list`: open in Quick list (one line per notice, tap to open the full card). Full cards are the default.

## Ghost embed

On the Ghost page, add an **HTML card**. The script lets the frame grow to fit
the page, so readers never scroll inside a box:

```html
<iframe id="notice-board" src="https://kate-rose.github.io/notice-board/?embed=1"
        style="width:100%; height:1400px; border:0; background:#ffffff"
        loading="lazy" title="Notice Board"></iframe>
<script>
  window.addEventListener("message", function (e) {
    if (e.origin !== "https://kate-rose.github.io") return;
    var h = e.data && e.data.noticeBoardHeight;
    if (h) document.getElementById("notice-board").style.height = (h + 20) + "px";
  });
</script>
```

Keep the Ghost page text free of dates; the board computes "in 4 days" at view time.
