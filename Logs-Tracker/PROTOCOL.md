# Protocol — Tracker

Vault: https://github.com/Stackocakes-Hub/Tracker-Vault
Branch: `main`

This repository is the shared log. It is **not** application source and **not** the Tracker website source.

- **Tracker** (the website) reads and writes this vault.
- **Design Grok** files bugs, sets, decisions; sets `verified` / `closed`.
- **Impl Grok** reads the vault and writes status (`confirmed`, `in-progress`, `fixed`).
- Both use GitHub tools on `main`.

This folder is `Logs-Tracker/`. `<subject>` must be **Tracker**.

See also the PROTOCOL.md inside each project folder (same rules, with that folder filled in).

## Files that matter

| Path | Role |
|---|---|
| `PROTOCOL.md` | This document. Read it first. |
| `Logs-Tracker/HEAD.xml` | Newest delta. Always read. Always overwrite on write. |
| `Logs-Tracker/MANIFEST.txt` | One filename per line, oldest first, `HEAD.xml` last. |
| `Logs-Tracker/YYYY-MM-DDTHHmmssZ-<hash>.xml` | Immutable dated entries. Never edit. Never delete. |
| `Logs-Tracker/ID-Discussion/` | One XML file per message. |

Every dated `*.xml` that exists in the folder **must** be listed on MANIFEST. If you write a file, list it in the same commit.

## Canonical filename (hash naming)

Do **not** edit an old dated XML file.

1. Build the XML body (UTF-8).
2. SHA-256 the **exact body**. First 8 lowercase hex chars = `hash`.
3. Dated name: `YYYY-MM-DDTHHmmssZ-<hash>.xml`
   - UTC
   - Date has hyphens: `2026-09-05`
   - Time has **no colons**: `T074310Z`
   - Then a hyphen and the 8-char hash
4. Example: `2026-09-05T074310Z-ca1a383b.xml`

Legacy compact names (`20260905T072846Z-…`) exist on Logs-Tracker. **Do not mint new files that way. Do not rename old files.**

`written=` on the XML root is ISO-8601 (colons allowed). Only the **filename** strips colons.

Discussion filenames: `TARGET-YYYY-MM-DDTHHmmssZ-<hash>.xml`

## Three-file commit (status logs)

GitHub `push_files` on `main` with **three** files in **one** commit:

1. `Logs-Tracker/<dated>.xml` — the new body
2. `Logs-Tracker/HEAD.xml` — **same** body
3. `Logs-Tracker/MANIFEST.txt` — previous dated names + the new dated name + `HEAD.xml` last

Do not drop names from MANIFEST. Insert a missed file in filename order.

## Revision

Read `HEAD.xml`. New `<revision>` is that number **plus 1**. Never reuse a revision. Never go backwards.

## Writers

| `writer=` | Who | May set |
|---|---|---|
| `impl` | implementation Grok | `confirmed` `in-progress` `fixed` and discussion |
| `design` | Tracker / design Grok | `open` `verified` `closed` `wontfix` `done` and discussion |

Impl never sets `verified`. Design never pretends a code fix is `fixed` unless they actually changed the app.

Either side may `closed` or reopen (`open`).

`<app>Tracker</app>` always. `<subject>` is the project id exact: `FrameField` or `Tracker`.

## nextIds

HEAD (and any minting delta) **must** include:

```xml
<nextIds bug="3" feat="21" comp="10"/>
```

Those numbers are the **next unused** id. FrameField current values (2026-09-05, rev 17): **bug 3, feat 21, comp 10** (BUG-001..002, FEAT-001..020, COMP-001..009). Before minting a new id, read HEAD. After minting, bump the matching number in the same delta.

## How to read (every turn, before you code)

1. `github___get_file_contents` owner `Stackocakes-Hub` repo `Tracker-Vault` path `Logs-Tracker/MANIFEST.txt`
2. Same for `Logs-Tracker/HEAD.xml`
3. Any dated name that was not on MANIFEST last turn is new. Fetch it.
4. Merge in **filename order**, then apply `HEAD.xml` last.
5. Same `id` → later file wins. Notes and decisions accumulate.

A log is “new” if its filename is not in your last-seen MANIFEST.

## Status words (exact strings)

### Bugs

| Status | Who | Complete? |
|---|---|---|
| `open` | design or impl | no (also used to reopen) |
| `confirmed` | impl | no |
| `in-progress` | impl | no |
| `fixed` | impl | no — waiting for design |
| `verified` | design | **yes** |
| `closed` | design or impl | **yes** |
| `wontfix` | design | parked |

`fixed` is not closed.

### Sets and features

| Status | Complete? |
|---|---|
| `planned` | no |
| `specified` | no |
| `in-progress` | no |
| `blocked` | no |
| `done` | **yes** (exit true and gates complete) |
| `parked` | n/a |

## Impl: mark a bug fixed (not verified)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<trackerLog schema="1" writer="impl" written="2026-09-05T12:00:00-05:00">
  <app>Tracker</app>
  <subject>FrameField</subject>
  <revision>18</revision>
  <nextIds bug="3" feat="21" comp="10"/>
  <bug id="BUG-001" status="fixed">
    <notes>File: src/foo.ts. How to verify: zoom in, draw, sizes match.</notes>
  </bug>
</trackerLog>
```

Then the three-file commit. `writer="impl"`. Bump revision.

## Design: verify or close

Same three-file commit, `writer="design"`.

## Discussion (not a status change)

One new XML per message in `Logs-Tracker/ID-Discussion/`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<discussion schema="1" target="BUG-001" writer="impl" written="2026-09-05T12:00:00-05:00">
  <app>Tracker</app>
  <body>Question or reply. Plain text. Concept art attached.</body>
  <image name="BUG-001-2026-09-05T120000Z-ab12cd34.jpg"/>
</discussion>
```

Commit **two** files only (XML, never image bytes):

1. `Logs-Tracker/ID-Discussion/<TARGET-dated-hash>.xml`
2. Append that name to `Logs-Tracker/ID-Discussion/MANIFEST.txt`

Do not edit old discussion files.

## Pictures (Grok concept art, screenshots)

**Do not put image bytes in this GitHub vault.** No `Pictures/` folder, no base64 in XML, no SVG.

XML only stores the filename:

```xml
<image name="FEAT-006-2026-09-05T150000Z-ab12cd34.jpg"/>
```

### How Grok attaches a picture

1. Generate or capture **png, jpg, webp, or gif**. No SVG. Max **2 MB**. Max **4** images per message.
2. POST **raw file bytes** (not JSON, not base64) to the **live Tracker website**:

```
POST {TRACKER_ORIGIN}/api/picture?project={ProjectId}&scope=discussion&target={FEAT-006}
Content-Type: image/jpeg
<body = the file bytes>
```

- `project` = subject id, e.g. `Tracker` or `FrameField`
- `scope=discussion` for a thread; `scope=log` only for a ticket-log screenshot
- `target` = the ticket id (`FEAT-006`, `BUG-001`, …)

3. JSON response: `{ "ok": true, "name": "FEAT-006-….jpg" }`. Use that **exact** `name`.
4. Add `<image name="…"/>` on the discussion (or log) XML.
5. Commit the XML as usual. **Do not** `push_files` the image.

GET for humans/site: `{TRACKER_ORIGIN}/api/picture?project={ProjectId}&name={name}`

Bytes live on **Vercel Blob** at key `pictures/{project}/{scope}/{name}`. Preview without a Blob token may keep a local copy only; that does not survive publish.

If POST fails, still post the discussion **text** and say the picture could not be stored. Do not fall back to GitHub binaries.

`TRACKER_ORIGIN` is the published Tracker URL (the website). If you do not have it, ask design. Do not guess.

Allowed name charset: `[A-Za-z0-9._-]`. Missing or bad pictures are skipped; they do not break the scanner.

## What not to write

- Application source
- Image bytes (png/jpg/webp/gif) or base64 pictures
- Edits to old dated xml
- `status="verified"` from impl
- `status="done"` on a set whose gates are still open
- New files using the compact `YYYYMMDDThhmmssZ` stamp

## GitHub tools (Grok)

Read: `github___get_file_contents`
Write several files: `github___push_files`
owner `Stackocakes-Hub` repo `Tracker-Vault` branch `main`
