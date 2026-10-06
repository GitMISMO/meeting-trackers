# MISMO Meeting Trackers

Attendance across every MISMO workgroup meeting and summit session — logging, drill-downs by
group, person and company, and the figures that feed the Member 360 Activity Snapshot.

Hosted the same way as the Sponsorship Prospectus and Summit HQ: GitHub Pages on
`resources.mismo.org`.

## What Is in Here

| File | What it is |
|---|---|
| `index.html` | The whole application, plus the block at the top that connects it to the save relay. No build step, nothing to install. |
| `vendor/xlsx.min.js` | SheetJS, used only to read TrackPod summit exports in the browser. |
| `.nojekyll` | Stops GitHub Pages running Jekyll over the files. Required. |

**This repository holds no attendance data.** The imported history — 48,338 records naming 2,300
people and the companies they come from — is member information, so it lives in the private
`meeting-trackers-data` repository and reaches the page through the relay only after sign-in.
That is the same rule Summit HQ follows: nothing in the public page repository identifies a member.

The history is loaded once, by whoever sets this up, from the `baseline.json` supplied with the
tool. Until it is, anyone opening the page sees a one-time screen offering to load it; afterwards
nobody sees that screen again.

## How It Saves

It uses **MISMO's save relay**, the same one as Summit HQ — and the same way. The page itself was
built inside Claude, where it reached its data through `window.claude.use('db' | 'user')`. The
block at the top of `index.html` rebuilds that door on top of the relay, so not a line of the tool
had to change.

- **Relay:** the shared Lambda URL, project key **`meeting-trackers`**
- **Data:** the private repository **`GitMISMO/meeting-trackers-data`**
- **Sign-in:** the shared resources.mismo.org sign-in (`/assets/session.js`), gated on
  `requireAccess('meeting-trackers')`

### Before It Can Save — One Setup Step

The relay has to be told about this tool. Until it is, every call comes back `UNKNOWN_PROJECT` and
the page says saving is not set up yet. Perry needs to:

1. Create the private repository `GitMISMO/meeting-trackers-data`.
2. Register the project key `meeting-trackers` on the relay, pointing at it.
3. Give the facilitators the role that lets them write to it.

Nothing in this repository changes when that happens — the page starts saving on the next reload.

### How Documents Are Filed

The relay has no "list a folder" route, so, exactly as in Summit HQ, related documents share one
file and the ones that grow without end get a file each.

| Relay file | Holds |
|---|---|
| `people`, `companies`, `groups`, `summits`, `aliases`, `sched` | one file per collection — these are a handful of bucket documents each and stay small |
| `m-<group>-<year>` | `meetings/<groupKey>__<year>` — one file per group-year, because meetings never stop accumulating |
| `own-index` | the list of those meeting files, so the collection can be read back |

A shared file is `{docs:{"<collection>/<id>": {...}}}`; a file of its own is `{doc:{...}}`.

Each meeting document is `{ "items": { "YYYY-MM-DD": {...} }, "updated": "<ISO date>" }`, so a
year of one workgroup is a few kilobytes — far inside the relay's 1 MB-a-save limit.

### Saving, Clashes and Catching Up

- Every save sends the version (`sha`) it last read. If someone else saved in between the relay
  answers **409**; the change is re-applied on top of theirs and sent again. Two facilitators
  logging different meetings never overwrite each other.
- Saves made within a moment of each other go out as one commit, and commits go out one at a time.
- The relay has no push, so loaded files are re-read every 30 seconds while the page is in use
  (every 2 minutes after ten idle minutes, not at all while the tab is hidden). Someone else's work
  appears without a reload.
- **Unreachable relay:** the change stays queued, the person is told it has not saved yet, and it
  goes out when the connection is back.
- **Misconfigured relay** (`TOKEN`, `GITHUB`, `UNKNOWN_PROJECT`): reported at once rather than
  retried forever, because retrying will not fix it.
- **Expired sign-in:** the save waits, the person is asked to sign in again, nothing is lost.

### Who Logged It

The audit trail — who saved a meeting and who first recorded it — comes from the signed-in person's
name via `ResourcesSession.current()`. No extra relay route is needed.

## Deploying

1. Put these files at the root of the repository.
2. **Settings → Pages**, source **Deploy from a branch**, branch `main`, folder `/ (root)`.
3. Add the custom domain if this repo is the one serving `resources.mismo.org`, and keep
   `.nojekyll` in place.

There is no build step. Editing `index.html` and pushing is the whole deployment.

## Refreshing the Baseline

`baseline.json` is generated from the fifteen tracker workbooks and only needs regenerating if
those are reimported from scratch — day-to-day logging never touches it. To replace it, delete the
`baseline` document from `meeting-trackers-data` and the one-time load screen comes back.

The Joseph Lamothe to Kathryn Williams handover is baked into the current file, so a regeneration
from the original Zoom workbook would reintroduce him unless that workbook is corrected first.

## Known Data Gaps

- 71 people have no company on file, which puts **147 attendance records outside every company
  total**. The Companies list says so on screen. Giving those people a company is the highest-value
  cleanup available.
- 4 companies have no category.
- One active workgroup (Private Label RMBS Valuation DWG) has no cadence set.

All three are listed under **Data Health** on the Stats page.
