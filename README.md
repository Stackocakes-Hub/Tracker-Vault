# Tracker-Vault

Shared XML log vault for multiple software projects.

This repository is **not** application source. Tracker (the website) reads and writes it.

Each project is a folder named `Logs-<ProjectName>/`.

**Write contract:** [PROTOCOL.md](PROTOCOL.md). Every project folder has a copy.

| Path | Role |
|---|---|
| `Logs-<ProjectName>/` | Status XML, HEAD, MANIFEST |
| `Logs-<ProjectName>/ID-Discussion/` | One XML file per question or reply |
| `PROTOCOL.md` | Hash names, three-file commit, revision, writers, pictures |

Pictures are **not** stored here. See PROTOCOL.md (POST /api/picture, then `<image name>` in XML).

Repo: https://github.com/Stackocakes-Hub/Tracker-Vault
