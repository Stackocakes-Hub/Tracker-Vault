# Tracker-Vault protocol

Vault: https://github.com/Stackocakes-Hub/Tracker-Vault

This repository is the shared log. It is **not** Framefield source and **not** the Tracker website source.

- **Tracker** (published site) reads this vault and displays it.
- **Design Grok** (Tracker chat) files bugs, sets, decisions.
- **Impl Grok** (Framefield chat) reads the vault and writes status updates.
- Both use GitHub tools. Branch is always `main`.

Logs live under `logs/`.

## Files that matter

| Path | Role |
|---|---|
| `PROTOCOL.md` | This document. Read it first. |
| `logs/HEAD.xml` | Newest delta. Always read this. Always overwrite on write. |
| `logs/MANIFEST.txt` | One filename per line, oldest first, `HEAD.xml` last. |
| `logs/YYYY-MM-DDTHHmmssZ-<hash>.xml` | Immutable dated entries. Never edit. Never delete. |

## How to read (every turn, before you code)

1. GitHub `get_file_contents`
   - owner: `Stackocakes-Hub`
   - repo: `Tracker-Vault`
   - path: `logs/MANIFEST.txt`
2. GitHub `get_file_contents` path `logs/HEAD.xml`
3. Compare `MANIFEST.txt` to the list you saw last turn.
4. Any **new** dated name is a new log. Fetch that path under `logs/`.
5. Merge in filename order, then apply `HEAD.xml` last.
6. Same `id` → later file wins. Notes and decisions accumulate.

A log is “new” if its filename is not in your last-seen MANIFEST.

Raw fallback (no GitHub tool):

`https://raw.githubusercontent.com/Stackocakes-Hub/Tracker-Vault/main/logs/HEAD.xml`

`https://raw.githubusercontent.com/Stackocakes-Hub/Tracker-Vault/main/logs/MANIFEST.txt`

## Status words (use these exact strings)

### Bugs (`<bug>`)

| Status | Who sets it | Meaning | Complete? |
|---|---|---|---|
| `open` | design | Filed, not started | no |
| `confirmed` | impl | Reproduced | no |
| `in-progress` | impl | Being fixed | no |
| `fixed` | impl | Code changed; waiting for design to check | no |
| `verified` | design | Exit is true; bug is done | **yes** |
| `wontfix` | design | Will not do | parked |

Impl never sets `verified`. Design never sets `fixed`.

### Sets (`<set>`) and features (`<feature>`)

| Status | Meaning | Complete? |
|---|---|---|
| `planned` | Not started | no |
| `specified` | Spec exists, not building | no |
| `in-progress` | Building | no |
| `blocked` | Waiting on a gate | no |
| `done` | Exit is true | **yes** |
| `parked` | Not this phase | n/a |

A set is complete only when:

- `status="done"`, **and**
- every `<gate>` is complete (`verified` for bugs, `done` for sets), **and**
- the `<exit>` sentence is true in the running app.

Do not mark SET-1.0 `done` while BUG-001 or BUG-002 is still `open` / `fixed`.

## How to mark complete (impl)

Do **not** edit an old dated XML file.

Write a **new** delta:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<trackerLog schema="1" writer="impl" written="2026-09-05T12:00:00-05:00">
  <app>Tracker</app>
  <subject>Framefield</subject>
  <revision>8</revision>
  <bug id="BUG-001" status="fixed">
    <notes>screenToWorld used for ghost and commit. File: src/foo.ts. Verify: zoom in, draw, sizes match.</notes>
  </bug>
</trackerLog>
```

Then:

1. SHA-256 the body. First 8 hex chars = `hash`.
2. Dated name: `YYYY-MM-DDTHHmmssZ-<hash>.xml` (UTC).
3. GitHub `push_files` on `main` with **three** files in one commit:
   - `logs/<dated>.xml` — the new body
   - `logs/HEAD.xml` — same body
   - `logs/MANIFEST.txt` — previous lines + the dated name + `HEAD.xml` last
4. Bump `<revision>` by 1 from the last HEAD.
5. `writer="impl"`.

That is “not complete yet”: status `fixed`. Design later writes `verified`.

## How to mark complete (design)

Same three-file commit, `writer="design"`:

```xml
<bug id="BUG-001" status="verified">
  <notes>Checked on the live app. Ghost and commit match at deep zoom.</notes>
</bug>
<set id="SET-1.0" status="done">
  <notes>Exit true: draw, reload, export. Geometry survives.</notes>
</set>
```

## How to mark not complete / reopen

```xml
<bug id="BUG-001" status="open">
  <notes>Still wrong on a 4px square at default zoom. Reopened.</notes>
</bug>
<set id="SET-1.0" status="in-progress">
  <notes>Exit not true. Reopened.</notes>
</set>
```

## ID discussion (questions and replies)

Folder:

- Local: `Tracker-Log/ID-Discussion/`
- Vault: `logs/ID-Discussion/`

The scanner loads **every** `*.xml` in that folder and `ID-Discussion/MANIFEST.txt`.

One file = one message. `target` is the line id (`BUG-001`, `SET-1.0`, `FEAT-020`, `COMP-008`). Click that line on Tracker to expand the thread.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<discussion schema="1" target="BUG-001" writer="impl" written="2026-09-05T12:00:00-05:00">
  <app>Tracker</app>
  <body>Your question or reply. Plain text.</body>
</discussion>
```

Filename: `TARGET-YYYY-MM-DDTHHmmssZ-<hash>.xml`

Push on main:

1. `logs/ID-Discussion/<dated>.xml`
2. Append that name to `logs/ID-Discussion/MANIFEST.txt`

Do not edit old discussion files. Replies are new files with the same `target`. Impl asks. Design replies. Either side may post.

A discussion is **not** a status change. Completing a bug still uses a `trackerLog` delta with `status="fixed"` / `verified`.

## What not to write

- Framefield application source
- `Framefield_Tracker.json`
- Edits to old `logs/20*.xml` files
- `status="verified"` from impl
- `status="done"` on a set whose gates are still open

## GitHub tool names (Grok)

Read: `github___get_file_contents`  
Write several files: `github___push_files`  
owner `Stackocakes-Hub` repo `Tracker-Vault` branch `main`
