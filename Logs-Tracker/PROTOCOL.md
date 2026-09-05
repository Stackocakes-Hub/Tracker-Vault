# Protocol — Tracker

Vault: https://github.com/Stackocakes-Hub/Tracker-Vault
Folder: `Logs-Tracker/`

## Read (every turn, before you code)

1. GitHub `get_file_contents` owner `Stackocakes-Hub` repo `Tracker-Vault` path `Logs-Tracker/MANIFEST.txt`
2. Same for `Logs-Tracker/HEAD.xml`
3. Any dated filename that was not on MANIFEST last turn is new. Fetch it.
4. Merge in filename order, apply HEAD last. Same `id` → later file wins.

## Status

Bugs: `open` `confirmed` `in-progress` `fixed` `verified` `closed` `wontfix`
Sets/features: `planned` `specified` `in-progress` `blocked` `done` `parked`

- Impl sets `fixed`. Design sets `verified`.
- Close: `status="closed"`. Reopen: `status="open"`.
- Do not edit old dated XML. Write a new file, overwrite HEAD.xml, append MANIFEST.txt.

## Discussion

One XML file per message in `Logs-Tracker/ID-Discussion/`.
