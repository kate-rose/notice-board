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

## Weekly email digest (feeds for an RSS-to-email service)

`digest.xml` (both counties), `digest-skamania.xml` and `digest-klickitat.xml` carry ONE item per week, built on the first sweep of the week (normally Monday) and never edited afterwards, so an RSS-to-email service sends each subscriber one email a week and what it sent stays what the feed says. Each item lists what is open for comment, hearings and meetings in the next two weeks, bids and board seats, elections, and the horizon, with absolute dates and a link to each card. Eight weeks are kept. Point Buttondown's RSS-to-email, follow.it, Blogtrottr or any feed reader at one of these URLs.

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
- `?view=list` / `?view=cards`: Quick list (one line per notice, tap to open the full card) or Full cards. Phones open in Quick list, wider screens in Full cards; a reader's own choice is remembered.
- `?topic=<budget|land|apply|vote|other>`: open on one kind of notice (the chip row under the summary).

## How the page sorts notices (2026-11-03)

- **Open for comment**: a comment window that is still open (not a bid, not an election).
- **Hearings and meetings coming up**: a dated hearing or meeting with no open window.
- **On the horizon**: talked about, not scheduled (curated cards only); hidden when empty.
- **Bids, contracts and board seats** and **Elections**: their own sections, so they never inflate "open for comment".
- **Choosing a place** shows the governments `places.json` binds to it, plus any district no place lists yet, in its county, tagged "Area not mapped yet" (nothing is silently dropped). **"Only my town and county-wide notices"** keeps the town's own governments, districts bound to it and state notices; from the county government it keeps county-wide notices and ones that name the place.

## Ghost embed

Live at https://skamania-dispatch.ghost.io/speak-up-public-comment-notice-board/ (2026-11-03). skamaniadispatch.com redirects every path to the ghost.io home page, so link the ghost.io address until the custom domain is connected in Ghost. On the Ghost page, an **HTML card** holds this: The frame
grows to fit the board, so readers never scroll inside a box, and `?place=`, `?topic=`
and `?view=` on the Ghost page's address are passed through, so
`…/speak-up-public-comment-notice-board/?place=bingen` opens on Bingen:

```html
<iframe id="notice-board" title="Notice Board" loading="lazy"
        style="width:100%; height:1600px; border:0; background:#ffffff"></iframe>
<noscript><p><a href="https://kate-rose.github.io/notice-board/">Open the Notice Board</a></p></noscript>
<script>
(function () {
  var frame = document.getElementById("notice-board");
  var src = new URL("https://kate-rose.github.io/notice-board/?embed=1");
  // Pass ?place=, ?topic= and ?view= from this page's address to the board,
  // so skamaniadispatch.com/notice-board/?place=bingen opens on Bingen.
  var here = new URLSearchParams(window.location.search);
  ["place", "topic", "view"].forEach(function (k) {
    if (here.get(k)) src.searchParams.set(k, here.get(k));
  });
  if (window.location.hash) src.hash = window.location.hash;
  frame.src = src.toString();
  // The board reports its height, so the frame grows to fit and readers never scroll inside a box.
  window.addEventListener("message", function (e) {
    if (e.origin !== "https://kate-rose.github.io") return;
    var h = e.data && e.data.noticeBoardHeight;
    if (h) frame.style.height = (h + 20) + "px";
  });
})();
</script>
```

Keep the Ghost page text free of dates; the board computes "in 4 days" at view time.
