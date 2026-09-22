---
description: DMP COMMAND project restart, backlog bundling, and deployment discipline
---

# DMP COMMAND project instructions

When a user says "wir arbeiten an DMP_Command weiter" or similar, first read `Documentation/DMP_COMMAND_SESSION_RESTART_GUIDE.md`, then `Documentation/DMP COMMAND_Mission_und_KI_Arbeitsregeln.md`, `Documentation/Backlog/DMP_COMMAND_Backlog.md`, `Documentation/DMP_COMMAND_Release_Notes.md`, `Documentation/DMP_COMMAND_Operations_Manual.md`, `Documentation/DMP Command Configuration.csv`, and `Documentation/DMP Command Agent Status.csv` before making technical claims or changes.

Before any version bump, `pac solution pack`, `pac solution import`, `pac canvas pack`, or app publish handoff, search the active backlog for the same agent, app screen, or deployment target. Bundle already diagnosed, low-risk backlog items into the same deployment package unless the user explicitly requests an immediate single hotfix or the issue is a production emergency. If an emergency forces a single hotfix, explicitly state which backlog-bundling check was skipped or deferred.

After every deployment/import/publish handoff, summarize what was deployed, version numbers before/after, validation performed, required live tests, and remaining open backlog items. Then remind the user to continue in a fresh session to reduce token/credit usage.

Keep DMP COMMAND documentation synchronized between the Git repository under `C:\PowerAppWork\DMP_COMMAND_Solution\Documentation` and the OneDrive/team-file copy under `C:\Users\df843\OneDrive - Deutsche Börse AG\GO365_DMP Communication - Email Hotline\AI_Agent\Documentation`.
