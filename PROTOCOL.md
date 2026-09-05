# Tracker-Vault protocol

Vault: https://github.com/Stackocakes-Hub/Tracker-Vault
Branch: `main`

This repository is the shared log. It is **not** application source and **not** the Tracker website source.

- **Tracker** (the website) reads and writes this vault.
- **Design Grok** (Tracker-XML chat) owns `PROTOCOL.md`, files bugs/sets/decisions, and sets `verified` / `closed` / `wontfix` / `done`.
- **Impl Grok** reads the vault and writes status (`confirmed`, `in-progress`, `fixed`).
- Other apps follow **this GitHub file**. They do not keep a private fork of the rules.
- Both use GitHub tools on `main`.

Each project is `Logs-<ProjectName>/`. FrameField subject is **FrameField** (not Framefield). Tracker subject is **Tracker**.

See also the PROTOCOL.md inside each project folder (same rules, with that folder filled in).

## Files that matter

| Path | Role |
|---|---|
| `PROTOCOL.md` | This document. Read it first. Design owns edits. |
| `Logs-<Project>/HEAD.xml` | Newest delta. Always read. Always overwrite on write. Revision clock. |
| `Logs-<Project>/MANIFEST.txt` | One filename per line, oldest first, `HEAD.xml` last. |
| `Logs-<Project>/YYYY-MM-DDTHHmmssZ-<hash>.xml` | Immutable dated entries. Never edit. Never delete. |
| `Logs-<Project>/ID-Discussion/` | One XML file per message. |

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

Legacy compact names (`20260905T072846Z-…`) exist on Logs-Tracker. **Do not mint new files that way. Do not rename old files.**

`written=` on the XML root is ISO-8601 (colons allowed). Only the **filename** strips colons.

Discussion filenames: `TARGET-YYYY-MM-DDTHHmmssZ-<hash>.xml`

## Three-file commit (status logs)

GitHub `push_files` on `main` with **three** files in **one** commit (protocol/README may ride in the same commit):

1. `Logs-<Project>/<dated>.xml` — the new body
2. `Logs-<Project>/HEAD.xml` — **same** body
3. `Logs-<Project>/MANIFEST.txt` — previous dated names + any restored missing names + the new dated name + `HEAD.xml` last

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

`<app>Tracker</app>` always. `<subject>` is the project id exact: `FrameField` or `Tracker`.

## nextIds

HEAD (and any minting delta) **must** include:

```xml
<nextIds bug="4" feat="21" comp="10"/>
```

Those numbers are the **next unused** id for **that project**. Always read HEAD. Protocol snapshots drift; HEAD wins.

Last-known snapshots (2026-09-05):

| Project | bug | feat | comp |
|---|---|---|---|
| FrameField | 4 | 21 | 10 |
| Tracker | 2 | 2 | 1 |

FrameField issued BUG-001, BUG-002, BUG-003 so the next bug id is **4**. Tracker issued FEAT-001 so the next feat id is **2**. After minting a new id, bump the matching number in the same delta.

## How to read (every turn, before you code)

1. `github___get_file_contents` owner `Stackocakes-Hub` repo `Tracker-Vault` path `Logs-<Project>/MANIFEST.txt`
2. Same for `Logs-<Project>/HEAD.xml`
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
  <revision>19</revision>
  <nextIds bug="4" feat="21" comp="10"/>
  <bug id="BUG-001" status="fixed">
    <notes>File: src/foo.ts. How to verify: zoom in, draw, sizes match.</notes>
  </bug>
</trackerLog>
```

Then the three-file commit. `writer="impl"`. Bump revision from current HEAD.

## Design: verify or close

Same three-file commit, `writer="design"`.

## Discussion (not a status change)

One new XML per message in `Logs-<Project>/ID-Discussion/`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<discussion schema="1" target="BUG-001" writer="impl" written="2026-09-05T12:00:00-05:00">
  <app>Tracker</app>
  <body>Question or reply. Plain text.</body>
</discussion>
```

Commit **two** files:

1. `Logs-<Project>/ID-Discussion/<TARGET-dated-hash>.xml`
2. Append that name to `Logs-<Project>/ID-Discussion/MANIFEST.txt`

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
