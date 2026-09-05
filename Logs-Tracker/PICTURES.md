# Pictures — Tracker

Do **not** store image bytes in GitHub.

1. POST raw png/jpg/webp/gif (max 2 MB, no SVG) to
   `{TRACKER_ORIGIN}/api/picture?project=Tracker&scope=discussion&target=FEAT-001`
2. Put the returned `name` in XML: `<image name="…"/>`
3. Commit only the XML.

See PROTOCOL.md.
