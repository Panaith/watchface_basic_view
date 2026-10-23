---
name: custom-watchface-xml-edit
description: Changes the Watch Face Format definition of the watch face safely. Use when you edit watchface/app/src/main/res/raw/watchface.xml, strings.xml, a drawable or the font, add or move a component, setting (UserConfigurations), flavor or complication slot, or when the watch face renders wrong on a watch.
---

# Editing the watch face (Watch Face Format v2)

**Owner:** project-owned, edited directly (see `custom-aitooling-maintain`).

The watch face is one declarative file, `watchface/app/src/main/res/raw/watchface.xml`, plus its
resources. What is where (settings, colour roles, components, coordinates, slot IDs):
[watchface/documentation/design.md](../../../../watchface/documentation/design.md). Read it before
a change and update it in the same commit. `PERMISSIONS.md` applies; nothing in this skill writes to
GitHub.

## 1. Procedure

1. Read the part of `design.md` that the change touches, and the matching part of `watchface.xml`.
2. Make the change in XML only: no Kotlin or Java (`watchface/AGENTS.md`).
3. Check it against every pitfall in section 2.
4. A new setting or option: add its label to `app/src/main/res/values/strings.xml` and reference
   it by name (pitfall 1). A new or changed complication slot: re-declare its default provider
   policy in **every** `<Flavor>` too, as the existing slots do.
5. Decide the element's ambient behaviour (pitfall 5).
6. From `watchface/`, run the validator and the build; both must pass before a commit:

   ```powershell
   java -jar "tools\wff-validator.jar" 2 "app/src/main/res/raw/watchface.xml"
   .\gradlew.bat assembleDebug
   ```

7. Update `watchface/documentation/design.md` (configuration table, components, layout, slot IDs).
8. If the user wants to see it, install it (`custom-watchface-device-deploy`).

## 2. Pitfalls found on real hardware

Each of these broke rendering on a Wear OS device.

1. **`displayName` is a string resource name, without `@string/`.** In `<UserConfigurations>` write
   `displayName="config_show_battery_bar"`. A literal (`displayName="Show Battery"`) shows as
   "1st setting" on the watch; `@string/config_show_battery_bar` does not work either.
2. **Never put a `<Transform>` inside `<TimeText>`.** A `<Transform target="alpha" ...>` directly in
   `<TimeText>` is invalid and hides the clock. Wrap the `<DigitalClock>` in a `<Group>` and put the
   `Transform` on the `Group`.
3. **`ListConfiguration` values are integers in expressions.** Write
   `[CONFIGURATION.clockWeight] == 0`. Quotes (`'0'`) break the conditional rendering.
4. **Weather complications send either a colour or a monochrome icon.** Handle both:

   ```xml
   <PartImage x="..." y="..." width="..." height="...">
       <Image resource="[COMPLICATION.ICON]" />
   </PartImage>
   <PartImage x="..." y="..." width="..." height="..." tintColor="[CONFIGURATION.themeColor.1]">
       <Image resource="[COMPLICATION.MONOCHROMATIC_IMAGE]" />
   </PartImage>
   ```

5. **Ambient mode: keep lit pixels to a minimum** (OLED burn-in, battery). Give every non-essential
   element `<Variant mode="AMBIENT" target="alpha" value="0" />`: seconds, AM/PM, battery, logo,
   background, shortcut slots, complications. The clock and the date stay; the dimming overlay, the
   last element of the `Scene`, dims them.
6. **`TimeText` formats are not Java patterns.** `format="HH:mm"` makes the clock fail silently. Use
   `format="hh:mm"` (two `h` give the leading zero) and choose 12 or 24 hours with the `hourFormat`
   attribute: `"24"`, `"12"` or `"SYNC_TO_DEVICE"` (the default).

## 3. Colours

Colour text, icons and bars through the theme roles, so they follow the user's choice:
`themeColor.0` background tint, `themeColor.1` accent (icons, monochrome images, progress bars,
secondary labels), `themeColor.2` primary text. A new `ColorOption` defines all three colours, the
first always black.
