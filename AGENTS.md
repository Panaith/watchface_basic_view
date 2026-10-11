# Watch Face Basic View (`watchface_basic_view`)

## 1. Project Overview & Philosophy

`watchface_basic_view` is a modern, minimalist, battery-efficient digital watch face for Wear OS (Wear OS 4 / 5+).
It is built strictly using Google's declarative **Watch Face Format (WFF)** Version 1.

- **Package / Application ID:** `com.pansar.watchface_basic_view`
- **Repository:** [https://github.com/Panaith/watchface_basic_view](https://github.com/Panaith/watchface_basic_view)
- **Target OS:** Wear OS 4+ (minSdk 36, targetSdk 37, compileSdk 37)
- **IDE / Environment:** The user works exclusively in **Visual Studio Code** and **Google Antigravity IDE**. Do **not** generate or commit Android Studio / JetBrains `.idea/` directory files or cache artifacts.

---

## 2. Declarative Architecture (`android:hasCode="false"`)

This watch face runs entirely on Wear OS system rendering services without any custom Kotlin/Java runtime code:
1. `AndroidManifest.xml` explicitly defines `android:hasCode="false"`.
2. All visual layouts, complications, user settings, animations, dynamic color bindings, and ambient modes are defined in XML under `watchface/src/main/res/raw/watchface.xml`.
3. Never attempt to add Kotlin or Java classes, activities, or services for watch face logic. All logic must be expressed using WFF expressions and XML tags.

---

## 3. Repository Directory Structure

```
watchface-minimal/
├── .gitignore                         # Configured to ignore Gradle caches, local.properties, GEMINI.md
├── AGENTS.md                          # AI agent guide and technical documentation (this file)
├── GEMINI.md                          # Antigravity context loader (imports @AGENTS.md)
├── build.gradle.kts                   # Root build script
├── settings.gradle.kts                # Root project settings
├── gradle/
│   ├── libs.versions.toml             # Version catalog (Android Gradle Plugin, etc.)
│   └── wrapper/                       # Gradle wrapper binaries
├── gradlew, gradlew.bat               # Gradle wrapper CLI scripts
└── watchface/                         # Core watch face module
    ├── build.gradle.kts               # Module build configuration (minSdk 36, targetSdk 37)
    └── src/main/
        ├── AndroidManifest.xml        # Declares standalone watch face with format version 1
        └── res/
            ├── drawable/              # Preview and static graphic assets
            │   └── preview.png        # Watch face preview thumbnail
            ├── font/
            │   └── corinthia.ttf      # Custom script font for branding logo ("PanSar")
            ├── raw/
            │   └── watchface.xml      # Core declarative Watch Face Format definition
            ├── values/
            │   └── strings.xml        # Localization, config titles, and option labels
            └── xml/
                └── watch_face_info.xml # Watch face metadata (preview drawable, editable flag)
```

---

## 4. Watch Face Layout & Feature Specification (`watchface.xml`)

Canvas dimensions: **450 x 450** circular display.

### A. Background & Patterns
- Background solid fill bound to `[CONFIGURATION.bgColor]`.
- Dynamic pattern overlays toggled via `[CONFIGURATION.bgPattern]`:
  - `0`: Solid Color (Default)
  - `1`: Vignette (radial dark border stroke)
  - `2`: Grid Pattern (geometric line lattice)
  - `3`: Circles Pattern (nested concentric rings)
  - `4`: Diagonals Pattern (angled cross-hatch lines)

### B. Top Battery Widget (x=125, y=25, w=200, h=50)
- Toggled by boolean setting `[CONFIGURATION.showBattery]`.
- Launches `BATTERY_STATUS` when tapped.
- Displays dynamic progress bar scaled horizontally using `[BATTERY_PERCENT] / 100`.
- Text readout displays percentage: `[BATTERY_PERCENT]%`.
- Colors bound to `[CONFIGURATION.batteryColor]` and `[CONFIGURATION.fontsColor]`.
- Hidden in ambient mode (`<Variant mode="AMBIENT" target="alpha" value="0" />`).

### C. Row 1: Left & Right Complications (y=85, h=60)
- **Slot 1 (Left, x=40, w=170):** Date
  - System default: `DAY_OF_WEEK_AND_DATE` (type: `SHORT_TEXT`).
  - Supports: `SHORT_TEXT`, `SMALL_IMAGE`, `MONOCHROMATIC_IMAGE`, `PHOTO_IMAGE`, `EMPTY`.
- **Slot 2 (Right, x=240, w=170):** Weather / Info
  - System default: `EMPTY`.
  - Supports: `SHORT_TEXT` (dual icon support for `[COMPLICATION.ICON]` and `[COMPLICATION.MONOCHROMATIC_IMAGE]`), `SMALL_IMAGE`, `MONOCHROMATIC_IMAGE`, `PHOTO_IMAGE`, `EMPTY`.

### D. Row 2: Digital Clock (x=0, y=145, w=450, h=120)
- Launches `ALARM` when tapped.
- Supports 24-hour (`HH:mm`) and 12-hour (`hh:mm`) time via `[CONFIGURATION.clockFormat]`.
- Supports 3 font weights (`NORMAL`, `BOLD`, `LIGHT`) via `[CONFIGURATION.clockWeight]`.
- Bound to `[CONFIGURATION.accentColor]`.
- **Seconds Display (x=380, y=25 relative to clock widget):**
  - Toggled by `[CONFIGURATION.showSeconds]`.
  - Hidden in ambient mode to conserve battery.

### E. Row 3: Next Event Complication (x=40, y=260, w=370, h=45)
- **Slot 3:**
  - System default: `NEXT_EVENT` (type: `LONG_TEXT`).
  - Supports: `LONG_TEXT` (title + text), `SHORT_TEXT`, and image types.

### F. Bottom Arc: App Shortcuts (Slots 4 to 8, size 42x42)
Arranged along a bottom curve to match round watch screens:
- **Slot 4:** x=78, y=310
- **Slot 5:** x=140, y=330
- **Slot 6:** x=204, y=340 (Center bottom)
- **Slot 7:** x=268, y=330
- **Slot 8:** x=330, y=310
- Supported types: `MONOCHROMATIC_IMAGE`, `SMALL_IMAGE`, `PHOTO_IMAGE`, `EMPTY`.
- Ambient mode: Hidden (`alpha=0`).

### G. Custom Logo / Branding (x=0, y=400, w=450, h=35)
- Displays "PanSar" in custom font `corinthia`.
- Launches Contacts app (`com.google.android.contacts`) when tapped.
- Hidden in ambient mode.

---

## 5. User Customization Schema (`<UserConfigurations>`)

All configuration options are defined in `watchface.xml` and mapped to `strings.xml`:

| Configuration ID | Type | Options / Values | Default | Description |
|---|---|---|---|---|
| `showBattery` | Boolean | `TRUE` / `FALSE` | `TRUE` | Toggle top battery widget visibility |
| `showSeconds` | Boolean | `TRUE` / `FALSE` | `TRUE` | Toggle seconds counter display |
| `bgColor` | Color | `0` (Black), `1` (Dark Blue), `2` (Dark Red), `3` (Dark Green) | `0` | Background canvas color |
| `batteryColor` | Color | `0` (White), `1` (Green), `2` (Orange) | `0` | Battery indicator bar color |
| `fontsColor` | Color | `0` (White), `1` (Light Grey), `2` (Cyan) | `0` | Color for complications and general text |
| `accentColor` | Color | `0` (Green), `1` (White), `2` (Orange), `3` (Blue) | `0` | Primary digital clock color |
| `clockWeight` | List | `0` (Normal), `1` (Bold), `2` (Light) | `1` (Bold) | Font weight for the digital clock |
| `clockFormat` | List | `0` (24 Hour `HH:mm`), `1` (12 Hour `hh:mm`) | `0` (24 Hour) | Time format |
| `bgPattern` | List | `0` (Solid), `1` (Vignette), `2` (Grid), `3` (Circles), `4` (Diagonals) | `0` (Solid) | Background graphic texture |

---

## 6. Critical Watch Face Format (WFF) Rules & Gotchas

> [!WARNING]
> These rules were discovered through real hardware testing. Violating any of them will break rendering on Wear OS devices.

1. **`displayName` Must Reference String Resource Names (No `@string/` prefix):**
   - In `<UserConfigurations>`, use `displayName="config_show_battery"`.
   - **Do NOT** use string literals like `displayName="Show Battery"` (Wear OS will display "1st setting" instead).
   - **Do NOT** use `@string/config_show_battery`. Use only the resource identifier name.

2. **Never Put `<Transform>` Inside `<TimeText>`:**
   - Placing `<Transform target="alpha" ...>` directly inside `<TimeText>` is invalid syntax in WFF v1 and completely disables/hides the clock.
   - **Correct Solution:** Wrap `<DigitalClock>` or `<TimeText>` inside an enclosing `<Group>` and apply the `<Transform target="alpha" ...>` to that `<Group>`.

3. **ListConfiguration Values in Expressions are Integers:**
   - In WFF expressions, evaluate list options as numbers: `[CONFIGURATION.clockWeight] == 0`.
   - **Do NOT** use quotes around numbers (e.g. `'0'` evaluates incorrectly and breaks conditional rendering).

4. **Weather Complication Support:**
   - Weather providers supply both colored and monochrome icons depending on the user's provider.
   - Always define both:
     ```xml
     <PartImage x="..." y="..." width="..." height="...">
         <Image resource="[COMPLICATION.ICON]" />
     </PartImage>
     <PartImage x="..." y="..." width="..." height="..." tintColor="[CONFIGURATION.fontsColor]">
         <Image resource="[COMPLICATION.MONOCHROMATIC_IMAGE]" />
     </PartImage>
     ```

5. **Ambient Mode Optimization:**
   - Ambient mode on Wear OS OLED panels requires minimizing lit pixels to prevent burn-in and save power.
   - Add `<Variant mode="AMBIENT" target="alpha" value="0" />` to non-essential elements (seconds, battery, logo, background patterns, shortcut slots).

6. **TimeText Formatting & 12h / 24h Notation (`hourFormat`):**
   - In Watch Face Format, `<TimeText>` does **not** support Java-style `HH:mm`. Using `format="HH:mm"` is invalid syntax in WFF and causes the clock element to fail silently and disappear.
   - Always use `format="hh:mm"` (use two `h` characters to enforce leading zero padding, e.g. `01` to `12` or `01` to `23`).
   - To control 12-hour vs 24-hour notation, use the `hourFormat` attribute on `<TimeText>`:
     - `hourFormat="24"` forces 24-hour format.
     - `hourFormat="12"` forces 12-hour format.
     - `hourFormat="SYNC_TO_DEVICE"` (default when omitted) follows the device system setting.

---

## 7. Build, Test & Deployment Guide

### A. Prerequisites
- Java JDK 17 or higher
- Android SDK with Platform 36 & 37 installed
- Android Debug Bridge (`adb`) in system `PATH`

### B. Building
From the project root:
```powershell
# Build debug APK
.\gradlew.bat assembleDebug

# Build release APK
.\gradlew.bat assembleRelease
```

### C. Wireless Debugging & Deployment on Wear OS Watch
1. On the watch: Enable **Developer options** -> enable **Wireless debugging**.
2. Note the IP address and dynamic port shown (e.g., `192.168.1.176:37805`).
3. Connect via ADB:
   ```powershell
   adb connect 192.168.1.176:<PORT>
   adb devices
   ```
4. Install debug build directly to the watch:
   ```powershell
   .\gradlew.bat installDebug
   ```
5. Apply/Switch to the watch face:
   Select `watchface_basic_view` directly on the watch screen picker or in the companion Wear OS mobile app (e.g., Galaxy Wearable or Pixel Watch app).

---

## 8. Git & Contribution Guidelines

- **Primary Branch:** `master`
- **Origin Remote:** `https://github.com/Panaith/watchface_basic_view.git`
- Always verify that the project builds with `.\gradlew.bat assembleDebug` before committing.
- Keep commits focused, clean, and atomic.
