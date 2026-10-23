---
name: custom-watchface-device-deploy
description: Installs the watch face on a Wear OS watch over wireless debugging, or starts a local Wear OS emulator on Windows and installs it there. Use when the user wants to see, run or try a change on a watch or emulator, or asks to start, launch or list emulators (AVDs), or to run adb or installDebug.
---

# Running the watch face on a watch or an emulator

**Owner:** project-owned, edited directly (see `custom-aitooling-maintain`).

The steps for people are in
[watchface/documentation/overview.md, How to Develop](../../../../watchface/documentation/overview.md#how-to-develop);
this skill adds what an agent must do differently. `PERMISSIONS.md` applies: installing on a
device and starting an emulator act outside the repository, so ask first. Run every command from
`watchface/`, after the validator and `.\gradlew.bat assembleDebug` pass
(`custom-watchface-xml-edit`).

## 1. On a watch (wireless debugging)

1. Ask the user for the IP address and port the watch shows under **Developer options** →
   **Wireless debugging**. The port is dynamic: ask for the current one, never reuse an old one.
2. `adb connect <WATCH_IP>:<PORT>`, then `adb devices`: the watch must be listed as `device`.
3. `.\gradlew.bat installDebug`.
4. Tell the user to select the watch face in the watch face picker or the companion app (Galaxy
   Wearable / Pixel Watch app).

## 2. In a local emulator (Windows)

Always in this order:

1. **List the emulators:**

   ```powershell
   & "$env:LOCALAPPDATA\Android\Sdk\emulator\emulator.exe" -list-avds
   ```

2. **Ask the user which one to start**, as one question with the AVDs found as options (your
   agent's question tool, or a direct prompt), and wait for the answer. Never pick one yourself.
3. **Start it through WMI (`Win32_Process`)**, so it runs as an independent desktop process:

   ```powershell
   Invoke-CimMethod -ClassName Win32_Process -MethodName Create -Arguments @{ CommandLine = "$env:LOCALAPPDATA\Android\Sdk\emulator\emulator.exe -avd <SELECTED_AVD_NAME>" }
   ```

   Never start `emulator.exe` with `Start-Process` or as a foreground command of your shell tool:
   Windows ends the child processes of the agent's temporary PowerShell job when the command
   returns, and the emulator closes with it.

4. When `adb devices` lists the emulator as `device`, run `.\gradlew.bat installDebug` and tell the
   user to select the watch face in the emulator.
