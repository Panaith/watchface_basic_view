# Agent setup changelog

What each agent setup version (`ai_tooling/version`) changed for developers, newest first. When your
agent offers to re-run `ai_tooling/setup.ps1` at the start of a session, it shows the entries between
your `ai_tooling/localversion` and `ai_tooling/version`; after the run, `setup.ps1` prints them in full
as "What's new", and the agent shows you that section.

Every raise of `ai_tooling/version` adds an entry here, headed `## {version} — {date} — {issue}`.
Say what changes for developers and anything they must do beyond the re-run
(`custom-aitooling-maintain`, section 6).

## 1 — 2026-10-23 — initial setup

- The agent tooling is set up from the `ai_tooling` blueprint: skills, `PERMISSIONS.md`, subagent
  roles, MCP servers and the setup for Claude Code, Antigravity, GitHub Copilot and Cursor.
- The watch face rules from the old `watchface/AGENTS.md` are now the skills
  `custom-watchface-xml-edit` and `custom-watchface-device-deploy`; its design record is
  `watchface/documentation/design.md`.
- Run `.\ai_tooling\setup.ps1` once from the repository root, then start a new agent session.
