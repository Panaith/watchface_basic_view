# watchface_basic_view Workspace

This repository is a multi-app workspace. Each app lives in its own top-level folder with its own build setup and its own `AGENTS.md` describing it in detail.

- **Repository:** [https://github.com/Panaith/watchface_basic_view](https://github.com/Panaith/watchface_basic_view)
- **Primary Branch:** `master`
- **IDE / Environment:** Visual Studio Code and Google Antigravity IDE. Do **not** generate or commit Android Studio / JetBrains `.idea/` directory files or cache artifacts.

## Apps

| Folder | Description | Guide |
|---|---|---|
| `watchface/` | Wear OS digital watch face built with Watch Face Format v2 (Gradle project) | [`watchface/AGENTS.md`](watchface/AGENTS.md) |

When working on an app, read its `AGENTS.md` first and run its commands from inside its folder.

## Adding a New App

1. Create a new top-level folder (e.g. `backend/`) containing everything the app needs to build on its own.
2. Put app-specific ignore rules in that folder's `.gitignore`; the root `.gitignore` only holds repository-wide rules.
3. Add an `AGENTS.md` inside the folder and list the app in the table above.

## Git & Contribution Guidelines

- Keep commits focused, clean, and atomic (Conventional Commits style, scoped by app where useful, e.g. `feat(watchface): ...`, `docs: ...`).
