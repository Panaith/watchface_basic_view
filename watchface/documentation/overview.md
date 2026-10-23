# watchface

**Purpose: a modern, minimalist, battery-efficient digital watch face for Wear OS, for anyone who
installs it on their watch.**

How it is built inside (files, configuration, components, layout): [design.md](design.md).

## Tech Stack

- **Watch Face Format (WFF) version 2** (`com.google.wear.watchface.format.version` in
  `AndroidManifest.xml`): the watch face is declarative XML, with no Kotlin or Java code
  (`android:hasCode="false"`).
- **Wear OS 6+**: `minSdk` 36, `targetSdk` 37, `compileSdk` 37 (`app/build.gradle.kts`).
- **Gradle 9.6.0** (`gradle/wrapper/gradle-wrapper.properties`) with the Android Gradle Plugin
  **9.4.1** (`gradle/libs.versions.toml`); the Gradle daemon runs on a Java 25 toolchain
  (`gradle/gradle-daemon-jvm.properties`).
- Package / application ID `com.pansar.watchface_basic_view`, publisher `PanSar`.
- It talks to no other app in the repository.

## Installations

- Java JDK 17 or higher.
- Android SDK with platforms 36 and 37 installed (Gradle finds it through `local.properties`,
  which is not committed).
- Android Debug Bridge (`adb`) on the system `PATH`.
- Editor: Visual Studio Code or Google Antigravity.

## How to Develop

All commands run from the `watchface/` folder. Edit
[`app/src/main/res/raw/watchface.xml`](../app/src/main/res/raw/watchface.xml) and the resources next
to it, validate, then install on a watch or an emulator.

**Install on a Wear OS watch over wireless debugging:**

1. On the watch: enable **Developer options** → **Wireless debugging**.
2. Note the IP address and the port it shows (for example `192.168.1.176:37805`); the port
   changes.
3. Connect and install:

   ```powershell
   adb connect <WATCH_IP>:<PORT>
   adb devices
   .\gradlew.bat installDebug
   ```

4. Select the watch face in the watch face picker on the watch, or in the companion app (Galaxy
   Wearable / Pixel Watch app).

**Run it in a local Wear OS emulator (Windows):**

```powershell
& "$env:LOCALAPPDATA\Android\Sdk\emulator\emulator.exe" -list-avds
& "$env:LOCALAPPDATA\Android\Sdk\emulator\emulator.exe" -avd <AVD_NAME>
```

Then `.\gradlew.bat installDebug` and select the watch face in the emulator.

## How to Debug

- Validate the XML first (below).
- A watch face that validates can still render wrong on a watch: some mistakes hide the clock or
  make it fail silently. The rendering pitfalls found on real hardware (a literal `displayName`, a
  `Transform` inside `TimeText`, quoted list values, `HH:mm`, ...) are listed in the skill
  [`custom-watchface-xml-edit`](../../ai_tooling/skills/custom-watchface/custom-watchface-xml-edit/SKILL.md).

## How to Build

```powershell
.\gradlew.bat assembleDebug     # debug APK
.\gradlew.bat assembleRelease   # release APK
```

The APKs go to `app/build/outputs/apk/`. There is no CI pipeline.

## How to Test and Lint

- **Validate** the Watch Face Format syntax with Google's validator (a green run prints
  `PASSED : app/src/main/res/raw/watchface.xml is valid against watch face format version #2`):

  ```powershell
  java -jar "tools\wff-validator.jar" 2 "app/src/main/res/raw/watchface.xml"
  ```

- **Unit and E2E tests:** none. Check the result on a watch or an emulator.
- Before committing: the validator passes and `.\gradlew.bat assembleDebug` succeeds.
