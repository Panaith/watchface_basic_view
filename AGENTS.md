# Watch Face Basic View (`watchface_basic_view`)

## 1. Project Overview

`watchface_basic_view` is a modern, minimalist, battery-efficient digital watch face for Wear OS.
It is built strictly using Google's declarative **Watch Face Format (WFF)** Version 2.

- **Package / Application ID:** `com.pansar.watchface_basic_view`
- **Repository:** [https://github.com/Panaith/watchface_basic_view](https://github.com/Panaith/watchface_basic_view)
- **Target OS:** Wear OS 6+ (minSdk 36, targetSdk 37, compileSdk 37)
- **Format Version:** Watch Face Format Version 2 (`com.google.wear.watchface.format.version=2`)
- **Publisher:** `PanSar`
- **IDE / Environment:** Visual Studio Code and Google Antigravity IDE. Do **not** generate or commit Android Studio / JetBrains `.idea/` directory files or cache artifacts.

---

## 2. Declarative Architecture (`android:hasCode="false"`)

This watch face runs entirely on Wear OS system rendering services without any custom Kotlin/Java runtime code:
1. `AndroidManifest.xml` explicitly defines `android:hasCode="false"`.
2. All visual layouts, complications, user settings, dynamic color bindings, and ambient modes are defined in XML under `watchface/src/main/res/raw/watchface.xml`.
3. Never add Kotlin or Java classes, activities, or services for watch face logic. All logic must be expressed using declarative WFF XML tags and expressions.

---

## 3. Repository Directory Structure

```
watchface_basic_view/
├── .gitignore                         # Ignores Gradle caches, build output, local.properties, GEMINI.md, .idea
├── AGENTS.md                          # AI agent guide and technical documentation (this file)
├── GEMINI.md                          # Antigravity context loader (imports @AGENTS.md)
├── build.gradle.kts                   # Root build script
├── settings.gradle.kts                # Root project settings (rootProject.name = "watchface_basic_view")
├── gradle.properties                  # Gradle / AndroidX properties
├── gradle/
│   ├── gradle-daemon-jvm.properties   # Gradle daemon JVM toolchain
│   ├── libs.versions.toml             # Version catalog (AGP 9.4.1)
│   └── wrapper/                       # Gradle wrapper binaries
├── gradlew, gradlew.bat               # Gradle wrapper CLI scripts
├── tools/
│   └── wff-validator.jar              # Official Google WFF validator
└── watchface/                         # Core watch face module
    ├── build.gradle.kts               # Module build configuration (minSdk 36, targetSdk 37)
    └── src/main/
        ├── AndroidManifest.xml        # Standalone watch face, WFF version 2, publisher PanSar
        ├── keepRules/rules.keep       # R8 keep rules (template, unused)
        └── res/
            ├── drawable/              # background_*.png textures, preview.png
            ├── font/corinthia.ttf     # Custom script font for "PanSar" branding
            ├── raw/watchface.xml      # Core declarative Watch Face Format definition
            ├── values/strings.xml     # Config titles and option labels
            └── xml/
                ├── watch_face_info.xml   # Metadata (preview, editable, flavors, multiple instances)
                └── watch_face_shapes.xml # Circular 450x450 display shape binding
```

---

## 4. Core Implementation

Canvas: **450 x 450** circular display (`clipShape="CIRCLE"`, `CLOCK_TYPE=DIGITAL`).

### A. User Customization Schema (`<UserConfigurations>`)

| Configuration ID | Type | Options / Values | Default | Description |
|---|---|---|---|---|
| `themeColor` | Color | 11 options (`0`–`10`), see below | `0` (White) | Three-color theme palette |
| `bgImage` | List | `0`–`8` → `background_1` … `background_9` | `0` | Background texture image |
| `clockFormat` | List | `0` (24 Hour), `1` (12 Hour) | `0` | Time format |
| `clockWeight` | List | `0` (Normal), `1` (Bold), `2` (Light) | `0` | Font weight for clock, seconds, AM/PM |
| `showSeconds` | Boolean | `TRUE` / `FALSE` | `TRUE` | Toggle seconds display |
| `showBattery` | Boolean | `TRUE` / `FALSE` | `TRUE` | Toggle battery progress bar |

**`themeColor` color roles** (every option defines 3 colors):
- `[CONFIGURATION.themeColor.0]` — Background tint (always black).
- `[CONFIGURATION.themeColor.1]` — Accent: icons, monochrome images, progress bars, secondary labels.
- `[CONFIGURATION.themeColor.2]` — Primary text: clock, date, values.
- Options `0`–`5` are single-tone (White, Cyan, Orange, Green, Blue, Dark Red); options `6`–`10` are dual-tone (colored accent + white primary text).

**Flavors (presets):** `1` = Default (`themeColor=0`, `bgImage=0`), `2` = Variant (`themeColor=7`, `bgImage=7`). Each flavor also re-declares the default provider policy of every complication slot.

### B. Core Components
- **Background:** solid `PartDraw` tinted with `themeColor.0` + `bgImage` texture `ListConfiguration`. Both hidden in ambient.
- **Battery widget:** launches `BATTERY_STATUS`; `[BATTERY_PERCENT]%` text + horizontal progress bar toggled by `showBattery`. The bar is red (`#ff0101`) at 0–9% and `themeColor.1` at 10–100%.
- **Date:** `PartText` with `%s, %s %s %s` template (`DAY_OF_WEEK_S`, `DAY`, `MONTH_S`, `YEAR`), Gregorian calendar, launches `CALENDAR`. Stays visible in ambient (dimmed by overlay).
- **Digital clock:** `Group` launching `ALARM`. 2 formats × 3 weights = 6 `DigitalClock` variants toggled by alpha `Transform`s.
- **Seconds:** `TimeText format="ss"` in 3 weights, toggled by `showSeconds`, hidden in ambient.
- **AM/PM:** shown only when `clockFormat == 1`, via `[AMPM_STATE]` (`0` = AM, `1` = PM), 3 weights, hidden in ambient.
- **Weather complication:** `SHORT_TEXT` with dual icon support (`ICON` / `MONOCHROMATIC_IMAGE`), plus image types. Default `EMPTY`.
- **Next Event complication:** `LONG_TEXT` (title + text) default provider `NEXT_EVENT`.
- **Logo:** "PanSar" in `corinthia` font, launches Contacts (`com.google.android.contacts`), hidden in ambient.
- **Ambient dimming overlay:** full-screen `#85000000` rectangle, visible only in ambient (last element in the `Scene`).

---

## 5. Watch Face Layout Specification

### A. Top Battery Widget (`x=125, y=25, w=200, h=50`)
- Progress bar `48 x 6` at `x=76, y=5` (relative), toggled by `showBattery`.
- Percentage text: size 20, `BOLD`, `themeColor.2`.
- Hidden in ambient mode.

### B. Row 1: Date & Weather (`y=85, h=60`)
- **Date (Left, `x=45, w=180`):** size 20, `BOLD`, `themeColor.2`. Stays visible in ambient (dimmed by overlay).
- **Slot 2 (Right, `x=240, w=170`): Weather / Info complication.**
  - Default: `EMPTY` (`SHORT_TEXT`).
  - Supports: `SHORT_TEXT`, `SMALL_IMAGE`, `MONOCHROMATIC_IMAGE`, `PHOTO_IMAGE`, `EMPTY`.

### C. Row 2: Digital Clock (`x=0, y=145, w=450, h=120`)
- Clock size 110, `themeColor.2`.
- **Seconds (`x=380, y=15` relative):** size 26, `themeColor.1`.
- **AM/PM (`x=380, y=65` relative):** size 26, `themeColor.1`, directly below seconds.

### D. Row 3: Next Event (`x=40, y=260, w=370, h=45`)
- **Slot 3:** customizable (`isCustomizable="TRUE"`), default `NEXT_EVENT` (`LONG_TEXT`), fallback text "Next event" when empty.
- Supports: `LONG_TEXT`, `SHORT_TEXT`, `SMALL_IMAGE`, `MONOCHROMATIC_IMAGE`, `PHOTO_IMAGE`, `EMPTY`.

### E. Bottom Arc: App Shortcuts (Slots 4–8, `42 x 42`, `BoundingOval`)
- **Slot 4:** `x=78, y=310` · **Slot 5:** `x=140, y=330` · **Slot 6:** `x=204, y=340` (center) · **Slot 7:** `x=268, y=330` · **Slot 8:** `x=330, y=310`
- Supports: `MONOCHROMATIC_IMAGE`, `SMALL_IMAGE`, `PHOTO_IMAGE`, `EMPTY`. Default: `APP_SHORTCUT` (`SMALL_IMAGE`).
- Hidden in ambient mode.

### F. Logo (`x=0, y=400, w=450, h=35`)
- `corinthia` size 28, `themeColor.2`.

### G. Complication Slot IDs
| Slot ID | Purpose |
|---|---|
| `2` | Weather / Info |
| `3` | Next Event |
| `4`–`8` | App Shortcuts 1–5 |

---

## 6. Critical Watch Face Format (WFF) Rules & Gotchas

> [!WARNING]
> These rules were discovered through real hardware testing. Violating any of them will break rendering on Wear OS devices.

1. **`displayName` Must Reference String Resource Names (No `@string/` prefix):**
   - In `<UserConfigurations>`, use `displayName="config_show_battery_bar"`.
   - **Do NOT** use string literals like `displayName="Show Battery"` (Wear OS will display "1st setting" instead).
   - **Do NOT** use `@string/config_show_battery_bar`. Use only the resource identifier name.

2. **Never Put `<Transform>` Inside `<TimeText>`:**
   - Placing `<Transform target="alpha" ...>` directly inside `<TimeText>` is invalid and hides the clock.
   - **Correct Solution:** Wrap `<DigitalClock>` inside an enclosing `<Group>` and apply the `<Transform target="alpha" ...>` to that `<Group>`.

3. **ListConfiguration Values in Expressions are Integers:**
   - Evaluate list options as numbers: `[CONFIGURATION.clockWeight] == 0`.
   - **Do NOT** use quotes around numbers (e.g. `'0'` breaks conditional rendering).

4. **Weather Complication Support:**
   - Weather providers supply either colored or monochrome icons. Always handle both:
     ```xml
     <PartImage x="..." y="..." width="..." height="...">
         <Image resource="[COMPLICATION.ICON]" />
     </PartImage>
     <PartImage x="..." y="..." width="..." height="..." tintColor="[CONFIGURATION.themeColor.1]">
         <Image resource="[COMPLICATION.MONOCHROMATIC_IMAGE]" />
     </PartImage>
     ```

5. **Ambient Mode Optimization:**
   - Minimize lit pixels in ambient mode to prevent OLED burn-in and save power.
   - Add `<Variant mode="AMBIENT" target="alpha" value="0" />` to non-essential elements (seconds, AM/PM, battery, logo, background, shortcut slots, complications).

6. **TimeText Formatting & 12h / 24h Notation (`hourFormat`):**
   - `<TimeText>` does **not** support Java-style `HH:mm`. Using `format="HH:mm"` makes the clock fail silently.
   - Always use `format="hh:mm"` (two `h` characters enforce leading zero padding).
   - Control 12h / 24h with the `hourFormat` attribute: `"24"`, `"12"`, or `"SYNC_TO_DEVICE"` (default).

---

## 7. Build, Validation & Deployment Guide

### A. Prerequisites
- Java JDK 17 or higher
- Android SDK with Platform 36 & 37 installed
- Android Debug Bridge (`adb`) in system `PATH`

### B. Validation
Validate WFF syntax with Google's official validator:
```powershell
java -jar "tools\wff-validator.jar" 2 "watchface/src/main/res/raw/watchface.xml"
```

### C. Building
```powershell
# Build debug APK
.\gradlew.bat assembleDebug

# Build release APK
.\gradlew.bat assembleRelease
```

### D. Wireless Debugging & Deployment on Wear OS Watch
1. On the watch: enable **Developer options** → **Wireless debugging**.
2. Note the IP address and dynamic port shown (e.g., `192.168.1.176:37805`).
3. Connect and install:
   ```powershell
   adb connect <WATCH_IP>:<PORT>
   adb devices
   .\gradlew.bat installDebug
   ```
4. Select the watch face on the watch picker or in the companion app (Galaxy Wearable / Pixel Watch app).

---

## 8. Git & Contribution Guidelines

- **Primary Branch:** `master`
- **Origin Remote:** `https://github.com/Panaith/watchface_basic_view.git`
- Validate `watchface.xml` and verify `.\gradlew.bat assembleDebug` succeeds before committing.
- Keep commits focused, clean, and atomic (Conventional Commits style, e.g. `refactor(wff): ...`, `docs: ...`).
