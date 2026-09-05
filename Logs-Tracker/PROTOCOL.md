# Protocol — Tracker

Vault: https://github.com/Stackocakes-Hub/Tracker-Vault
Branch: `main`

This repository is the shared log. It is **not** application source and **not** the Tracker website source.

- **Tracker** (the website) reads and writes this vault.
- **Design Grok** (Tracker-XML chat) owns `PROTOCOL.md`, files bugs/sets/decisions, and sets `verified` / `closed` / `wontfix` / `done`.
- **Impl Grok** reads the vault and writes status (`confirmed`, `in-progress`, `fixed`).
- Other apps follow **this GitHub file**. They do not keep a private fork of the rules.

This folder is `Logs-Tracker/`. `<subject>` must be **Tracker**.

## Files that matter

| Path | Role |
|---|---|
| `PROTOCOL.md` | This document. Read it first. Design owns edits. |
| `Logs-Tracker/HEAD.xml` | Newest delta. Always read. Always overwrite on write. Revision clock. |
| `Logs-Tracker/MANIFEST.txt` | One filename per line, oldest first, `HEAD.xml` last. |
| `Logs-Tracker/YYYY-MM-DDTHHmmssZ-<hash>.xml` | Immutable dated entries. Never edit. Never delete. |
| `Logs-Tracker/ID-Discussion/` | One XML file per message. |

Every dated `*.xml` that exists in the folder **must** be listed on MANIFEST. If you write a file, list it in the same commit. If a file exists but is missing from MANIFEST, insert it in filename order on the next status commit. Do not rewrite old dated bodies.

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

Legacy compact names (`20260905T072846Z-…`) exist in this folder. **Do not mint new files that way. Do not rename old files.**

`written=` on the XML root is ISO-8601 (colons allowed). Only the **filename** strips colons.

Discussion filenames: `TARGET-YYYY-MM-DDTHHmmssZ-<hash>.xml`

## Three-file commit (status logs)

GitHub `push_files` on `main` with **three** files in **one** commit (protocol/README may ride in the same commit):

1. `Logs-Tracker/<dated>.xml` — the new body
2. `Logs-Tracker/HEAD.xml` — **same** body
3. `Logs-Tracker/MANIFEST.txt` — previous dated names + any restored missing names + the new dated name + `HEAD.xml` last

Do not drop names from MANIFEST. Insert a missed file in filename order.

## Revision

Read `HEAD.xml`. New `<revision>` is that number **plus 1**. Never reuse a revision on a new write. Never go backwards.

HEAD is the revision clock. If an older dated file reused a number, leave that file immutable and continue from **current HEAD + 1**.

## Writers

| `writer=` | Who | May set |
|---|---|---|
| `impl` | implementation Grok | `confirmed` `in-progress` `fixed` and discussion |
| `design` | Tracker-XML / design Grok | `open` `verified` `closed` `wontfix` `done` and discussion |

Impl never sets `verified`. Design never pretends a code fix is `fixed` unless they actually changed the app.

Either side may `closed` or reopen (`open`).

`<app>Tracker</app>` always. `<subject>` is **Tracker**.

## nextIds

HEAD (and any minting delta) **must** include `<nextIds>`. Always read HEAD. Protocol snapshots drift; HEAD wins.

Tracker last-known (2026-09-05, HEAD rev 3): **bug 2, feat 1, comp 1**.

These numbers are **not** FrameField’s nextIds. Do not copy FrameField counters into this folder.

After minting a new id, bump the matching number in the same delta.

## How to read (every turn, before you code)

1. `github___get_file_contents` owner `Stackocakes-Hub` repo `Tracker-Vault` path `Logs-Tracker/MANIFEST.txt`
2. Same for `Logs-Tracker/HEAD.xml`
3. Any dated name that was not on MANIFEST last turn is new. Fetch it.
4. Merge in **filename order**, then apply `HEAD.xml` last.
5. Same `id` → later file wins. Notes and decisions accumulate.

## Status words (exact strings)

Bugs: `open` `confirmed` `in-progress` `fixed` `verified` `closed` `wontfix`

Sets/features: `planned` `specified` `in-progress` `blocked` `done` `parked`

- Impl sets `fixed`. Design sets `verified`.
- Close: `status="closed"`. Reopen: `status="open"`.
- `fixed` is not closed.
- Do not mark a set `done` while its gates are still open.

## Impl: mark a bug fixed (not verified)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<trackerLog schema="1" writer="impl" written="2026-09-05T12:00:00-05:00">
  <app>Tracker</app>
  <subject>Tracker</subject>
  <revision>4</revision>
  <nextIds bug="2" feat="1" comp="1"/>
  <bug id="BUG-001" status="fixed">
    <notes>File and how to verify.</notes>
  </bug>
</trackerLog>
```

Then the three-file commit. `writer="impl"`. Bump revision from current HEAD.

## Design: verify or close

Same three-file commit, `writer="design"`.

## Discussion (not a status change)

One new XML per message in `Logs-Tracker/ID-Discussion/`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<discussion schema="1" target="BUG-001" writer="impl" written="2026-09-05T12:00:00-05:00">
  <app>Tracker</app>
  <body>Question or reply. Plain text.</body>
</discussion>
```

Commit **two** files:

1. `Logs-Tracker/ID-Discussion/<TARGET-dated-hash>.xml`
2. Append that name to `Logs-Tracker/ID-Discussion/MANIFEST.txt`

Do not edit old discussion files.

## What not to write

- Application source
- Edits to old dated xml
- `status="verified"` from impl
- `status="done"` on a set whose gates are still open
- New files using the compact `YYYYMMDDThhmmssZ` stamp
- PROTOCOL.md edits from impl unless design asked

## GitHub tools (Grok)

Read: `github___get_file_contents`
Write several files: `github___push_files`
owner `Stackocakes-Hub` repo `Tracker-Vault` branch `main`
