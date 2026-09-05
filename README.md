# Tracker-Vault

Shared XML log vault for multiple software projects.

This repository is **not** application source. Tracker (the website) reads and writes it.

Each project is a folder named `Logs-<ProjectName>/`.

**Write contract:** [PROTOCOL.md](PROTOCOL.md). Every project folder has a copy. Design Grok in the Tracker-XML chat owns that contract. Other apps follow it.

| Path | Role |
|---|---|
| `Logs-<ProjectName>/` | Status XML, HEAD, MANIFEST |
| `Logs-<ProjectName>/ID-Discussion/` | One XML file per question or reply |
| `PROTOCOL.md` | Hash names, three-file commit, revision, writers |

Repo: https://github.com/Stackocakes-Hub/Tracker-Vault
