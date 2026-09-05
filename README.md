# Tracker-Vault

Shared XML log vault for multiple software projects.

This repository is **not** application source. Tracker (the website) reads and writes it.

Each project is a folder named `Logs-<ProjectName>/`.

| Path | Role |
|---|---|
| `Logs-<ProjectName>/` | Status XML, HEAD, MANIFEST |
| `Logs-<ProjectName>/ID-Discussion/` | One XML file per question or reply |
| `PROTOCOL.md` | How to file, close, reopen, and discuss |

Open Tracker and pick a project. Add project creates a new `Logs-*` folder, a PROTOCOL copy, and a closed creation log.

Repo: https://github.com/Stackocakes-Hub/Tracker-Vault
