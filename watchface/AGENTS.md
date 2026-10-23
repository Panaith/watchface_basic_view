# watchface — AI Agent Instructions

This file contains rules specific to the **watchface** application, the Wear OS watch face
(what it is and how it is built: [documentation/overview.md](documentation/overview.md),
[documentation/design.md](documentation/design.md)).
The root [AGENTS.md](../AGENTS.md) and [PERMISSIONS.md](../ai_tooling/PERMISSIONS.md) apply here too.

**If your workspace is this folder and not the repository root** (the IDE was opened on `watchface` alone):

1. Read `../AGENTS.md` and `../ai_tooling/PERMISSIONS.md` now, unless they are already in your context, and follow them.
2. Tell the user once, at the start: *opening the repository root in the IDE is recommended. The project's skills,
   MCP servers and agent rules are published only there, so some of them may be missing in this window.*

## Critical Version Constraints

| Constraint | Why |
|-----------|-----|
| Watch Face Format **version 2** (`com.google.wear.watchface.format.version` in `AndroidManifest.xml`); validate with `wff-validator.jar 2` | Tags and attributes must exist in WFF v2; the validator checks against that version |
| `android:hasCode="false"` | A WFF watch face carries no code; Wear OS renders it from the XML alone |
| `minSdk` **36**, `targetSdk` **37**, `compileSdk` **37** (`app/build.gradle.kts`) | Wear OS 6+; the Android SDK needs platforms 36 and 37 |
| Gradle **9.6.0** (wrapper), AGP **9.4.1** (`gradle/libs.versions.toml`), daemon JVM toolchain **25** (`gradle/gradle-daemon-jvm.properties`) | Always build through `gradlew.bat`, never a system Gradle |
| Canvas **450 x 450**, circular (`clipShape="CIRCLE"` in `watchface.xml`, `shape="CIRCLE"` in `watch_face_shapes.xml`) | Every coordinate in `watchface.xml` is on this canvas |

## Things AI Agents Must Never Do

- Never add Kotlin or Java classes, activities or services for watch face logic: express all logic with declarative WFF
  XML tags and expressions in `app/src/main/res/raw/watchface.xml`.
- Never commit a change to `watchface.xml` or its resources before the validator passes and `.\gradlew.bat assembleDebug`
  succeeds.
- Never commit `local.properties`, `build/` or `.gradle/` (gitignored in `watchface/.gitignore` and `watchface/app/.gitignore`).

## Skills

| When you ... | Load |
|---|---|
| change `watchface.xml`, `strings.xml`, a drawable or font, or add a setting, complication or component | [`custom-watchface-xml-edit`](../ai_tooling/skills/custom-watchface/custom-watchface-xml-edit/SKILL.md) |
| install the watch face on a watch, or start a Wear OS emulator | [`custom-watchface-device-deploy`](../ai_tooling/skills/custom-watchface/custom-watchface-device-deploy/SKILL.md) |
| test or debug a change | `diagnosing-bugs` applies to rendering bugs (the loop is validator, build, install, look); `tdd` does not apply: there is no code and no test framework |

## Commands

Run from `watchface/`:

- **Prepare**: none (Gradle downloads its dependencies on the first build)
- **Run**: `.\gradlew.bat installDebug` on a connected watch or emulator (`custom-watchface-device-deploy`)
- **Unit tests**: none
- **E2E tests**: none
- **Lint**: `java -jar "tools\wff-validator.jar" 2 "app/src/main/res/raw/watchface.xml"`
- **Build**: `.\gradlew.bat assembleDebug` (release: `.\gradlew.bat assembleRelease`)

## Adding or Updating Dependencies

- Plugin versions live in `gradle/libs.versions.toml`; the Gradle version in `gradle/wrapper/gradle-wrapper.properties`
  (update with `.\gradlew.bat wrapper --gradle-version {version}`).
- **Notes**: the app has no library dependencies (`dependencies {}` is empty, `enableKotlin = false`); keep it that way.

## Agent Instruction Imports

The coding principles apply to every change in this app.

[@../ai_tooling/instructions/coding-principles.md](../ai_tooling/instructions/coding-principles.md)
