---
name: wiki-update-planning
description: "How to plan an update build of a repository's wiki: which existing pages to schedule, using each page's committed update window, the claims needing attention, and the owner's planning context. Pinned automatically in an update; ignore it in an init."
metadata:
  version: "1.0.0"
---

# Planning an update

This skill only covers an update build. An init has no existing wiki to evaluate.

The first message lists the current wiki pages (read one with read_wiki_page, or search them with search_wiki), the committed per-page update windows, and the claims requiring attention.

For update, evaluate each existing page inside its own committed update window.
A page already advanced by a merged partial update must not be regenerated for
changes at or before its baseline. Schedule it only when changes after that
baseline, current Claims issues, navigation changes, or cross-page consistency
require work.

When the user planning context names files or wiki pages, says to document
only specific files, or otherwise limits the requested documentation scope,
treat that as a hard scoped update mandate. Preserve unrelated page
framing/theme and the existing information architecture; do not use a narrow
request as permission for a broad wiki refresh. Schedule only directly requested
pages/files and genuinely affected cross-references or navigation. Do not refresh
quickstart.md merely because a scoped update touches other pages;
include it when the user explicitly requests it or when the page map, navigation,
or task-routing links actually change.
