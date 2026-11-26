---
name: team-account-migration
description: Backup made 2026-10-31 for moving from the personal Pro account to a Team account, and where the restore guide lives
metadata:
  type: project
---
On 2026-10-31 the user said they are moving from this personal Pro account to a Team account and asked how to keep rules, memory, scheduled routines and artifacts. A full backup is in /mnt/project-files/chuyen-tai-khoan-team/ with HUONG-DAN-KHOI-PHUC.md (restore steps), personal preferences copy, memory copies, skill SKILL.md and zip, routine prompts (page routine trig_01GwvKbnDm7jnovRY7rie5dq and HLB coordinator trig_01WH9a4VRDCSW6az6Hofrcvd), the Query Fan-Out Generator HTML and its 47 "runs" docs (runs.json). Nothing on the old account was deleted or disabled.

**Official docs (read 2026-10-31, support.claude.com/en/articles/9267400 and 9267247):** with the same email, joining a Team offers "Use your organization account only" > "Bring your data with you" (one-way, closes the personal account, cancels Pro; App Store subscribers can only keep both). Moves: chats, artifacts within chats, projects (instructions, files), Claude memory, Claude Code memory/settings. Does NOT move: published artifacts (Team has no publishing; links stop working), public chat share links, custom skills, custom connectors, Claude Code cloud sessions incl. scheduled tasks, app sign-ins. Docs say nothing about artifact database data or cloud environments/setup scripts.

**Why:** skills, routines, the environment and the public Query Fan-Out Generator page must be recreated from the backup either way.

**How to apply:** on the new account, the page's trigger ID (appears once in the HTML) and the old page link (twice in the routine prompt) must be replaced with the new ones. Other HLB artifacts (Bảng điều phối HLB etc.) were only listed by link, not backed up. Related: [[page-google-button-routine]].
