# Permanently Disabling the Built-in Keyboard on an ASUS ROG G513IH (Windows)

**Result:** The built-in keyboard is disabled, the external USB keyboard (HP) works normally, and the setup survives restarts.

**Final working method:** The Interception driver plus an AutoHotInterception (AutoHotkey v1) script that finds the built-in keyboard by its VID/PID, started automatically at every login through Task Scheduler.

**Status of the later fixes:** the ghost mic-toggle workaround (section 6) and the keyboard-ID fix for sleep (section 7) were applied. The sleep fix was not yet confirmed with a long sleep test when this was written.

---

## 1. Goal

- Completely disable the laptop's built-in keyboard.
- It must not re-enable itself, even after a restart or sleep.
- The external keyboard must keep working.

## 2. Key discovery: how the keyboard is actually connected

This was the most important finding, and it explains why the early methods failed.

```powershell
Get-PnpDevice -Class Keyboard | Format-List FriendlyName, InstanceId, Status, Present
```

| Device | Instance ID | Meaning |
|---|---|---|
| PC/AT Enhanced PS/2 Keyboard | `ACPI\ATK3001\...` | Status `Error`, so it was already dead. Not the real keyboard. |
| HID Keyboard Device | `HID\VID_0B05&PID_1866&COL03\...` | **The real built-in keyboard.** `0B05` is ASUS's USB vendor ID. |
| HID Keyboard Device | `HID\VID_03F0&PID_0067&MI_00\...` | The external HP keyboard, which must not be touched. |

The built-in keyboard is **not on PS/2. It is an internal USB/HID device (`USB\VID_0B05&PID_1866`)** split into several collections:

- `COL01`, `COL04`, `COL05`: vendor-defined (ASUS)
- `COL02`: ASUS firmware update
- `COL03`: **the keyboard**
- `COL06`: consumer control (media keys)
- `COL07`: system controller

---

## 3. What was tried and did not work

### 3.1 Disabling the `i8042prt` service

```
sc config i8042prt start= disabled
```

- **Why it failed:** the keyboard was never on PS/2.
- Use `shutdown /r /t 0` (a real restart) when testing, because Fast Startup restores the old kernel session after a normal shutdown.

### 3.2 Disabling the device in Device Manager

- The disable option did not work, or Windows re-enabled the device.

### 3.3 Blocking hardware IDs through Group Policy / Registry

- First the PS/2 IDs were blocked, which were the wrong device.
- Blocking the COL03 IDs afterwards also did not help. The policy prevents **new** installations, and COL03 is a child device already loaded through its parent USB device.

### 3.4 Disabling COL03 with PowerShell

```powershell
Disable-PnpDevice -InstanceId 'HID\VID_0B05&PID_1866&COL03\...' -Confirm:$false
```

- **Error:** `Not supported (0x8004100c)`. Windows does not allow a child collection to be disabled on its own.

### 3.5 Disabling the parent USB device

```powershell
pnputil /disable-device 'USB\VID_0B05&PID_1866\6&5280D8E&0&3'
```

- **Error:** `Cannot disable critical system device.`
- It would also have killed the ASUS Fn and hotkey functions.

### 3.6 Replacing the driver in Device Manager

- Tried, but it did not work.

---

## 4. What worked: Interception + AutoHotInterception

A keyboard filter at the driver level drops keystrokes from **one specific keyboard** before they reach Windows, which avoids Windows' restrictions on disabling devices.

### 4.1 Install the Interception driver

1. Download `Interception.zip` from `github.com/oblitum/Interception`.
2. In an administrator Command Prompt, go to the `command line installer` folder and run:
   ```
   install-interception.exe /install
   ```
3. Do a **real restart**.

### 4.2 Install AutoHotkey v1 and AutoHotInterception

1. Install AutoHotkey **v1.1** (not v2).
2. Download AutoHotInterception from `github.com/evilC/AutoHotInterception`.
3. Use this folder structure:

```
C:\AHI\
 ├─ Monitor.ahk
 ├─ blockkb.ahk
 └─ Lib\
     ├─ AutoHotInterception.ahk
     ├─ AutoHotInterception.dll
     ├─ CLR.ahk
     ├─ Unblocker.ps1
     ├─ x64\
     └─ x86\
```

### 4.3 Problem: the `Monitor.ahk` include error

- **Error:** `#Include file "Lib\AutoHotInterception.ahk" cannot be opened.`
- **Cause:** `Monitor.ahk` was inside the `Lib` folder, so the script looked for `Lib\Lib\...`.
- **Fix:** move `Monitor.ahk` one level up, outside `Lib`.

If the DLL is blocked because it was downloaded, run `Lib\Unblocker.ps1` or tick **Unblock** in the DLL's Properties.

### 4.4 Script bugs and fixes

**Bug 1: `SubscribeKeyboard` was not supported**

- **Error:** `0x80004002 - No such interface supported. Specifically: SubscribeKeyboard`
- **Fix:** use a loop that calls `SubscribeKey` for each key.

**Bug 2: the script exited immediately after starting**

- **Cause:** in AutoHotkey v1, a script with no hotkeys, GUI, or `#Persistent` runs its startup code and exits.
- **Fix:** add `#Persistent`.

**Bug 3: Windows key, right Ctrl/Alt and similar keys still worked**

- **Cause:** the loop only covered key codes 1 to 132. Extended keys use higher codes.
- **Fix:** extend the loop to 512.

### 4.5 Final script: `C:\AHI\blockkb.ahk`

```autohotkey
#SingleInstance Force
#Persistent
#include %A_ScriptDir%\Lib\AutoHotInterception.ahk

AHI := new AutoHotInterception()
kbId := AHI.GetKeyboardId(0x0B05, 0x1866)   ; finds the built-in keyboard by VID/PID

Loop 512
    AHI.SubscribeKey(kbId, A_Index, true, Func("Ignore"))
return

Ignore(code, state) {
}
```

The HP keyboard has a different VID (`03F0`), so it is not affected.

---

## 5. Auto-start with Task Scheduler

### 5.1 First attempt failed

```
Register-ScheduledTask : The task XML contains a value which is incorrectly formatted or out of range.
```

The cause was a principal without a `LogonType`, together with an `ExecutionTimeLimit` value that was not written to the task XML in a valid form.

### 5.2 Working version (administrator PowerShell)

```powershell
$ahk = 'C:\Program Files\AutoHotkey\AutoHotkey.exe'
$user = "$env:USERDOMAIN\$env:USERNAME"

$action    = New-ScheduledTaskAction -Execute $ahk -Argument '"C:\AHI\blockkb.ahk"' -WorkingDirectory 'C:\AHI'
$trigger   = New-ScheduledTaskTrigger -AtLogOn -User $user
$principal = New-ScheduledTaskPrincipal -UserId $user -LogonType Interactive -RunLevel Highest
$settings  = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries -StartWhenAvailable
$settings.ExecutionTimeLimit = 'PT0S'

Register-ScheduledTask -TaskName 'BlockInternalKeyboard' -Action $action -Trigger $trigger -Principal $principal -Settings $settings -Force
```

- `-AllowStartIfOnBatteries` and `-DontStopIfGoingOnBatteries`: the task also runs on battery.
- `ExecutionTimeLimit = 'PT0S'`: no time limit. Otherwise Task Scheduler kills the script after 72 hours.
- `RunLevel Highest`: runs with administrator rights.

### 5.3 Verify

1. Close any running copy of the script (tray: green **H** icon > Exit).
2. `Start-ScheduledTask -TaskName 'BlockInternalKeyboard'`
3. Restart with `shutdown /r /t 0`, log in, and confirm the built-in keyboard is still disabled.

---

## 6. Later issue: ghost key events and mic toggling

After the block worked, the keys that bypass the keyboard collection (Fn combinations, volume, mic, ROG key, fan profile) were still active, and pressing almost any key toggled the microphone.

### What was found

- Those special keys do not arrive through COL03. They come through the vendor-defined and consumer-control collections and the ASUS ACPI/WMI interface, so Interception cannot see them.
- A WMI watcher showed **ATK event 124** firing on every key press and also with no key pressed:

  ```powershell
  Register-WmiEvent -Namespace root\wmi -Class AsusAtkWmiEvent -SourceIdentifier AsusWatch -Action {
      Write-Host ("{0}  ATK event: {1}" -f (Get-Date -Format HH:mm:ss), $Event.SourceEventArgs.NewEvent.EventID)
  }
  # stop it with: Unregister-Event AsusWatch
  ```

- External keyboard presses did not trigger it, only the laptop keyboard.
- It continued with the script off, with the PS/2 service and the block policy reverted, and after a restart. So the script, the policy and the PS/2 change were not the cause.
- The most likely explanation is a fault on the laptop keyboard or its ribbon cable that makes the embedded controller send phantom events. This is a best guess and was not confirmed.

### Workaround that worked

Stopping the ASUS services stopped the mic toggling. This was done with G-Helper, and it can also be done by hand:

```powershell
'AsusAppService','ASUSOptimization','ASUSSoftwareManager','ASUSSwitch','ASUSSystemAnalysis','ASUSSystemDiagnosis' | ForEach-Object { Set-Service $_ -StartupType Disabled; Stop-Service $_ -Force -ErrorAction SilentlyContinue }
```

- Trade-off: the ROG key, fan profile switching and OSD popups stop working.
- To undo, use `Automatic` instead of `Disabled` and start the services again.
- A service stopped from G-Helper may come back after a restart. The command above keeps it disabled.

### Hardware option

Unplugging the keyboard ribbon (FPC) cable from the motherboard would stop both typing and the phantom events, but may affect the warranty.

---

## 7. Later issue: the block stopped working after sleep

- **Symptom:** after the laptop slept for a while, the built-in keyboard came back and ghost typing started. The script still showed as running in Task Manager, and restarting it by hand did not help.
- **Cause:** Interception keyboard IDs can change after resume. The built-in keyboard was ID 1 at first, then showed as ID 3 in `Monitor.ahk` (VID/PID `0x0B05, 0x1866`, handle `HID\VID_0B05&PID_1866&REV_4023&Col03`). The script was still blocking the old ID 1.
- **Fix:** look the keyboard up by VID/PID instead of a fixed number, as in the final script in section 4.5:

  ```autohotkey
  kbId := AHI.GetKeyboardId(0x0B05, 0x1866)
  ```

- **Limitation:** the lookup happens only when the script starts. If the ID changes while the script is already running, it keeps the old ID.

### Optional: restart the script when the laptop wakes up

```powershell
$ahk = 'C:\Program Files\AutoHotkey\AutoHotkey.exe'
$cmd = "Stop-Process -Name AutoHotkey* -Force -ErrorAction SilentlyContinue; Start-Sleep 3; Start-Process '$ahk' -ArgumentList '\`"C:\AHI\blockkb.ahk\`"' -WorkingDirectory 'C:\AHI'"
$action = New-ScheduledTaskAction -Execute 'powershell.exe' -Argument "-WindowStyle Hidden -Command `"$cmd`""

$class = Get-CimClass -ClassName MSFT_TaskEventTrigger -Namespace Root/Microsoft/Windows/TaskScheduler
$trigger = New-CimInstance -CimClass $class -ClientOnly
$trigger.Enabled = $true
$trigger.Subscription = '<QueryList><Query Id="0" Path="System"><Select Path="System">*[System[Provider[@Name=''Microsoft-Windows-Power-Troubleshooter''] and EventID=1]]</Select></Query></QueryList>'

$principal = New-ScheduledTaskPrincipal -UserId "$env:USERDOMAIN\$env:USERNAME" -LogonType Interactive -RunLevel Highest
$settings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries
Register-ScheduledTask -TaskName 'BlockKeyboardOnResume' -Action $action -Trigger $trigger -Principal $principal -Settings $settings -Force
```

This restarts the script after every wake from sleep. Use it only if the keyboard still comes back after the VID/PID fix.

---

## 8. Limitations

- **The built-in keyboard works on the login screen** and in the BIOS and boot menu. The script only starts after login.
- Do not move or delete `C:\AHI`, or the script will stop running.
- After a Windows or driver update, Interception can be reset. If the keyboard suddenly works again, run `install-interception.exe /install` again and restart.
- This is a software-level block inside Windows. The only hardware-level option is unplugging the keyboard's ribbon cable.

---

## 9. Undo (using the external keyboard)

```powershell
Unregister-ScheduledTask -TaskName 'BlockInternalKeyboard' -Confirm:$false
Unregister-ScheduledTask -TaskName 'BlockKeyboardOnResume' -Confirm:$false -ErrorAction SilentlyContinue
Stop-Process -Name AutoHotkey* -Force
```

To remove the Interception driver completely:

```
uninstall-interception.exe /uninstall
```

then restart.

To re-enable the ASUS services, set them back to `Automatic` and start them.

Leftovers from the earlier experiments can be cleaned up with:

```powershell
Remove-Item 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\DeviceInstall\Restrictions' -Recurse -ErrorAction SilentlyContinue
sc config i8042prt start= auto
```

---

## 10. Quick summary

| Method or issue | Result | Reason |
|---|---|---|
| Disable `i8042prt` service | Failed | The keyboard was never on PS/2 |
| Disable in Device Manager | Failed | HID child device that Windows reloads |
| Hardware ID block policy | Failed | Wrong IDs first, then no effect on COL03 |
| `Disable-PnpDevice` on COL03 | Failed | "Not supported" for a child collection |
| `pnputil` disable on parent | Failed | "Critical system device" |
| Driver replacement | Failed | Did not take effect |
| **Interception + AHI script + Task Scheduler** | **Worked** | Driver-level filter that drops keys from just that keyboard |
| Win/Ctrl/Alt still working | Fixed | Key range extended from 132 to 512 |
| Ghost event 124 / mic toggle | Worked around | ASUS services stopped; likely keyboard or EC hardware fault |
| Block lost after sleep | Fixed (pending a long sleep test) | Keyboard ID changed; now found by VID/PID |