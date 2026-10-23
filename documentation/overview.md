# Watch Face Basic View Repository — Overview

Watch Face Basic View is a minimalist, battery-efficient digital watch face for Wear OS, published
as PanSar. The repository is a multi-app workspace: each app lives in its own top-level folder with
its own build setup, and `ai_tooling` sets up the AI agents that work on all of them.

## Pipelines

None: the repository has no CI. Each app's `documentation/overview.md` says how to build and check
it locally.

---

## Apps

| App | Description |
|-----|-------------|
| [watchface](../watchface/documentation/overview.md) | The Wear OS watch face, built with Watch Face Format v2 as a Gradle project without app code |
| [ai_tooling](../ai_tooling/documentation/overview.md) | AI agent tooling: the skills, permissions, MCP servers and setup for every agent; not deployed |
