# Permanently Disabling the Built-in Keyboard on an ASUS ROG G513IH (Windows)

**Result:** The built-in keyboard is disabled, the external USB keyboard (HP) works normally, and the setup survives restarts.

**Final working method:** The Interception driver plus an AutoHotInterception (AutoHotkey v1) script, started automatically at every login through Task Scheduler.

---

## 1. Goal

- Completely disable the laptop's built-in keyboard.
- It must not re-enable itself, even after a restart.
- The external keyboard must keep working.

## 2. Key discovery: how the keyboard is actually connected

This was the most important finding, and it explains why the early methods failed.

Listing the keyboards in PowerShell:

```powershell
Get-PnpDevice -Class Keyboard | Format-List FriendlyName, InstanceId, Status, Present
```

What the output meant:

|Device|Instance ID|Meaning|
|---|---|---|
|PC/AT Enhanced PS/2 Keyboard|`ACPI\ATK3001\...`|Status `Error`, so it was already dead. This was not the real keyboard.|
|HID Keyboard Device|`HID\VID_0B05&PID_1866&COL03\...`|**The real built-in keyboard.** `0B05` is ASUS's USB vendor ID.|
|HID Keyboard Device|`HID\VID_03F0&PID_0067&MI_00\...`|The external HP keyboard, which must not be touched.|

The built-in keyboard is **not on PS/2. It is an internal USB/HID device (`USB\VID_0B05&PID_1866`).** That single device is split into several collections:

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

- This disables the PS/2 keyboard driver.
- **Why it failed:** the keyboard was never on PS/2, so the built-in keyboard kept working.
- Side note: use `shutdown /r /t 0` (a real restart) when testing, because Fast Startup restores the old kernel session after a normal shutdown.

### 3.2 Disabling the device in Device Manager

- The disable option did not work, or Windows re-enabled the device.

### 3.3 Blocking hardware IDs through Group Policy / Registry

The three PS/2 hardware IDs (such as `ACPI\PNP0303`) were added to `DenyDeviceIDs`.

- **Why it failed:** those were the IDs of the wrong device (PS/2). The real keyboard was on the HID path.

Blocking the COL03 IDs afterwards also did not help. This policy prevents **new** device installations. COL03 is a child device that is already loaded through its parent USB device, so the policy had no effect on it.

### 3.4 Disabling COL03 with PowerShell

```powershell
Disable-PnpDevice -InstanceId 'HID\VID_0B05&PID_1866&COL03\...' -Confirm:$false
```

- **Error:** `Not supported (0x8004100c)`
- **Why:** Windows does not allow a child collection to be disabled on its own.

### 3.5 Disabling the parent USB device

```powershell
pnputil /disable-device 'USB\VID_0B05&PID_1866\6&5280D8E&0&3'
```

- **Error:** `Cannot disable critical system device.`
- **Why:** Windows marks this device as critical.
- It would also have killed the ASUS Fn and hotkey functions.

### 3.6 Replacing the driver in Device Manager ("Update driver" > "Let me pick from a list")

- Tried, but it did not work.

---

## 4. What worked: Interception + AutoHotInterception

The idea is to place a keyboard filter at the driver level that drops keystrokes from **one specific keyboard** before they reach Windows. This avoids all of Windows' restrictions on disabling devices.

### 4.1 Install the Interception driver

1. Download `Interception.zip` from `github.com/oblitum/Interception`.
2. Open an administrator Command Prompt, go to the `command line installer` folder, and run:
    
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

### 4.3 First problem: the `Monitor.ahk` include error

- **Error:** `#Include file "Lib\AutoHotInterception.ahk" cannot be opened.`
- **Cause:** `Monitor.ahk` had been placed **inside** the `Lib` folder. The script resolves `Lib\...` relative to its own folder, so it was looking for `Lib\Lib\...`.
- **Fix:** move `Monitor.ahk` one level up, outside `Lib`.

If the DLL is blocked because it was downloaded from the internet, run `Lib\Unblocker.ps1`, or tick **Unblock** in the DLL's Properties.

### 4.4 Find the keyboard ID

Run `Monitor.ahk` and press keys on the built-in keyboard. The ID that shows the key events is the built-in keyboard's ID. The HP keyboard shows a different ID. This ID goes into `kbId` in the script.

### 4.5 Two script bugs and their fixes

**Bug 1: `SubscribeKeyboard` was not supported**

- **Error:** `0x80004002 - No such interface supported. Specifically: SubscribeKeyboard`
- **Fix:** replace it with a loop that calls `SubscribeKey` for each key.

**Bug 2: The script exited immediately after starting**

- **Symptom:** no error, the built-in keyboard still worked, and no AutoHotkey icon appeared in the tray.
- **Cause:** in AutoHotkey v1, a script with no hotkeys, GUI, or `#Persistent` runs its startup code and exits.
- **Fix:** add `#Persistent`. `#NoTrayIcon` was also removed during testing so the running script is visible.

### 4.6 Final script: `C:\AHI\blockkb.ahk`

```autohotkey
#SingleInstance Force
#Persistent
#include %A_ScriptDir%\Lib\AutoHotInterception.ahk

AHI := new AutoHotInterception()
kbId := 1   ; ID of the built-in keyboard from Monitor.ahk (confirm on your own system)

Loop 132
    AHI.SubscribeKey(kbId, A_Index, true, Func("Ignore"))
return

Ignore(code, state) {
}
```

Test result: typing on the built-in keyboard did nothing, and the HP keyboard worked normally.

---

## 5. Auto-start with Task Scheduler

### 5.1 First attempt failed

```
Register-ScheduledTask : The task XML contains a value which is incorrectly formatted or out of range.
```

The cause was a principal without a `LogonType`, together with an `ExecutionTimeLimit` value that was not being written to the task XML in a valid form.

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

Important settings:

- `-AllowStartIfOnBatteries` and `-DontStopIfGoingOnBatteries`: the task also runs on battery.
- `ExecutionTimeLimit = 'PT0S'`: no time limit. Otherwise Task Scheduler kills the script after 72 hours and the keyboard comes back.
- `RunLevel Highest`: runs with administrator rights.

### 5.3 Verify

1. Close any already-running copy of the script (tray: green **H** icon > Exit).
2. Run `Start-ScheduledTask -TaskName 'BlockInternalKeyboard'`.
3. Restart with `shutdown /r /t 0`, log in, and confirm the built-in keyboard is still disabled.

**Result:** the built-in keyboard stayed disabled after the restart.

---

## 6. Limitations

- **The built-in keyboard works on the login screen.** The script only starts after login. Enter the password with the external keyboard or the on-screen keyboard.
- **It also works in the BIOS and boot menu**, because those are firmware level.
- Do not move or delete `C:\AHI`, or the script will stop running.
- After a Windows or driver update, Interception can be reset. If the keyboard suddenly works again, run `install-interception.exe /install` again and restart.
- Only **one keyboard ID** is blocked. If that ID ever changes (new driver, different USB device order), check it again with `Monitor.ahk`.
- This is a software-level block inside Windows. The only hardware-level permanent option is unplugging the keyboard's ribbon (FPC) cable from the motherboard, which may affect the warranty.

---

## 7. Undo (using the external keyboard)

```powershell
# Remove the task and stop the running script
Unregister-ScheduledTask -TaskName 'BlockInternalKeyboard' -Confirm:$false
Stop-Process -Name AutoHotkey* -Force
```

To remove the Interception driver completely:

```
uninstall-interception.exe /uninstall
```

then restart.

Leftovers from the earlier experiments can be cleaned up too:

```powershell
Remove-Item 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\DeviceInstall\Restrictions\DenyDeviceIDs' -Recurse -ErrorAction SilentlyContinue
sc config i8042prt start= auto
```

---

## 8. Quick summary

|Method|Result|Reason|
|---|---|---|
|Disable `i8042prt` service|Failed|The keyboard was never on PS/2|
|Disable in Device Manager|Failed|HID child device that Windows reloads|
|Hardware ID block policy|Failed|First the wrong (PS/2) IDs, then no effect on COL03|
|`Disable-PnpDevice` on COL03|Failed|"Not supported" for a child collection|
|`pnputil` disable on parent|Failed|"Critical system device"|
|Driver replacement|Failed|Did not take effect|
|**Interception + AHI script + Task Scheduler**|**Worked**|A driver-level filter that drops keys from just that one keyboard|