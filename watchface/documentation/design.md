# watchface — Design

The design record of the watch face: how the files are organised, what the user can configure, the
components and where each one sits on the dial. The source of truth is
[`app/src/main/res/raw/watchface.xml`](../app/src/main/res/raw/watchface.xml); update this document
in the same commit as a change there. Back to the [overview](overview.md).

## Contents

- [Directory structure](#directory-structure)
- [Declarative architecture](#declarative-architecture)
- [User configuration](#user-configuration)
- [Components](#components)
- [Layout](#layout)
- [Complication slot IDs](#complication-slot-ids)

## Directory structure

```
watchface/                             # Watch face Gradle project
├── .gitignore                         # Gradle caches, build output, local.properties
├── AGENTS.md                          # Rules for AI agents working on this app
├── documentation/                     # overview.md (how to build and run), design.md (this file)
├── build.gradle.kts                   # Root build script
├── settings.gradle.kts                # rootProject.name = "watchface_basic_view", includes :app
├── gradle.properties                  # Gradle / AndroidX properties
├── gradle/
│   ├── gradle-daemon-jvm.properties   # Gradle daemon JVM toolchain
│   ├── libs.versions.toml             # Version catalog (AGP)
│   └── wrapper/                       # Gradle wrapper
├── gradlew, gradlew.bat               # Gradle wrapper scripts
├── tools/
│   └── wff-validator.jar              # Google's Watch Face Format validator
└── app/                               # The watch face module
    ├── .gitignore                     # The module's build output
    ├── build.gradle.kts               # Module build configuration (SDK levels)
    └── src/main/
        ├── AndroidManifest.xml        # Standalone watch face, WFF version 2, publisher PanSar
        ├── keepRules/rules.keep       # R8 keep rules (template, unused)
        └── res/
            ├── drawable/              # background_1 … background_9 textures, preview.png
            ├── font/corinthia.ttf     # Script font of the "PanSar" logo
            ├── raw/watchface.xml      # The Watch Face Format definition
            ├── values/strings.xml     # Configuration titles and option labels
            └── xml/
                ├── watch_face_info.xml    # Preview, editable, flavors, multiple instances
                └── watch_face_shapes.xml  # Circular 450 x 450 display shape
```

## Declarative architecture

The watch face has no Kotlin or Java code: `AndroidManifest.xml` declares
`android:hasCode="false"`, and Wear OS renders it from the Watch Face Format definition alone. All
layout, complications, user settings, colour bindings and ambient behaviour are XML tags and
expressions in `watchface.xml`.

Canvas: **450 x 450**, circular (`clipShape="CIRCLE"`), `CLOCK_TYPE` `DIGITAL`.

## User configuration

`<UserConfigurations>` in `watchface.xml`; the labels are string resources in `strings.xml`.

| Configuration ID | Type | Options / Values | Default | Description |
|---|---|---|---|---|
| `themeColor` | Color | 11 options (`0`–`10`), see below | `0` (White) | Three-colour theme palette |
| `bgImage` | List | `0`–`8` → `background_1` … `background_9` | `0` | Background texture image |
| `clockFormat` | List | `0` (24 Hour), `1` (12 Hour) | `0` | Time format |
| `clockWeight` | List | `0` (Normal), `1` (Bold), `2` (Light) | `0` | Font weight for clock, seconds, AM/PM |
| `showSeconds` | Boolean | `TRUE` / `FALSE` | `TRUE` | Toggle seconds display |
| `showBattery` | Boolean | `TRUE` / `FALSE` | `TRUE` | Toggle battery progress bar |

**`themeColor` colour roles** (every option defines three colours):

- `[CONFIGURATION.themeColor.0]`: background tint (always black).
- `[CONFIGURATION.themeColor.1]`: accent: icons, monochrome images, progress bars, secondary labels.
- `[CONFIGURATION.themeColor.2]`: primary text: clock, date, values.
- Options `0`–`5` are single-tone (White, Cyan, Orange, Green, Blue, Dark Red); options `6`–`10`
  are dual-tone (coloured accent, white primary text).

**Flavors (presets):** `1` = Default (`themeColor=0`, `bgImage=0`), `2` = Variant
(`themeColor=7`, `bgImage=7`). Each flavor also re-declares the default provider policy of every
complication slot.

## Components

- **Background:** a solid `PartDraw` tinted with `themeColor.0`, plus the `bgImage` texture
  (`ListConfiguration`). Both hidden in ambient.
- **Battery widget:** launches `BATTERY_STATUS`; `[BATTERY_PERCENT]%` text and a horizontal
  progress bar toggled by `showBattery`. The bar is red (`#ff0101`) at 0–9% and `themeColor.1` at
  10–100%.
- **Date:** a two-line `Group` launching `CALENDAR` (top line `[DAY_OF_WEEK_F]`, bottom line
  `%s %s %s` with `DAY`, `MONTH_S`, `YEAR`), Gregorian calendar. Stays visible in ambient (dimmed by
  the overlay).
- **Digital clock:** a `Group` launching `ALARM`. 2 formats × 3 weights = 6 `DigitalClock` variants,
  toggled by alpha `Transform`s.
- **Seconds:** `TimeText format="ss"` in 3 weights, toggled by `showSeconds`, hidden in ambient.
- **AM/PM:** shown only when `clockFormat == 1`, through `[AMPM_STATE]` (`0` = AM, `1` = PM), 3
  weights, hidden in ambient.
- **Weather complication:** `SHORT_TEXT` with both icon kinds (`ICON` / `MONOCHROMATIC_IMAGE`),
  plus the image types. Default `EMPTY`.
- **Next Event complication:** `LONG_TEXT` (title and text), default provider `NEXT_EVENT`.
- **Logo:** "PanSar" in the `corinthia` font, launches Contacts (`com.google.android.contacts`),
  hidden in ambient.
- **Ambient dimming overlay:** a full-screen `#85000000` rectangle, visible only in ambient (the
  last element in the `Scene`).

## Layout

Coordinates are in canvas pixels; "relative" means relative to the enclosing group.

### Top: battery widget (`x=125, y=25, w=200, h=50`)

- Progress bar `48 x 6` at `x=76, y=5` (relative), toggled by `showBattery`.
- Percentage text: size 20, `BOLD`, `themeColor.2`.
- Hidden in ambient.

### Row 1: date and weather (`y=85, h=60`)

- **Date (left, `x=45, w=185`):** two lines (top: full day of week `DAY_OF_WEEK_F` at `y=0, h=30`;
  bottom: `DAY MONTH_S YEAR` at `y=30, h=30`), size 23, `BOLD`, `themeColor.2`. Stays visible in
  ambient (dimmed by the overlay).
- **Slot 2 (right, `x=240, w=170`): weather / info complication.** Default `EMPTY`
  (`SHORT_TEXT`). Supports `SHORT_TEXT`, `SMALL_IMAGE`, `MONOCHROMATIC_IMAGE`, `PHOTO_IMAGE`,
  `EMPTY`. Hidden in ambient.

### Row 2: digital clock (`x=0, y=145, w=450, h=120`)

- Clock size 110, `themeColor.2`.
- **Seconds (`x=380, y=15` relative):** size 26, `themeColor.1`.
- **AM/PM (`x=380, y=65` relative):** size 26, `themeColor.1`, directly below the seconds.

### Row 3: next event (`x=40, y=260, w=370, h=45`)

- **Slot 3:** default `NEXT_EVENT` (`LONG_TEXT`), fallback text "Next event" when empty.
- Supports `LONG_TEXT`, `SHORT_TEXT`, `SMALL_IMAGE`, `MONOCHROMATIC_IMAGE`, `PHOTO_IMAGE`, `EMPTY`.
- Hidden in ambient.

### Bottom arc: app shortcuts (slots 4–8, `42 x 42`, `BoundingOval`)

- **Slot 4:** `x=78, y=310` · **Slot 5:** `x=140, y=330` · **Slot 6:** `x=204, y=340` (centre) ·
  **Slot 7:** `x=268, y=330` · **Slot 8:** `x=330, y=310`
- Supports `MONOCHROMATIC_IMAGE`, `SMALL_IMAGE`, `PHOTO_IMAGE`, `EMPTY`. Default `APP_SHORTCUT`
  (`SMALL_IMAGE`).
- Hidden in ambient.

### Logo (`x=0, y=400, w=450, h=35`)

- `corinthia` size 28, `themeColor.2`.

## Complication slot IDs

Every slot is customizable (`isCustomizable="TRUE"`).

| Slot ID | Purpose |
|---|---|
| `2` | Weather / Info |
| `3` | Next Event |
| `4`–`8` | App Shortcuts 1–5 |
