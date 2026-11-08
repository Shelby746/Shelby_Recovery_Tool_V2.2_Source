# Shelby Recovery Tool 2.2

MediaTek flashing and Android recovery in one Windows window.

Windows 10 / 11 | Python 3.11+ | PySide6 | single portable EXE

[About](#about-the-tool) | [Home screen](#the-home-screen) | [Pages](#pages) |
[Developer](#developer) | [Setup](#setup) | [Drivers](#drivers-and-detection)

## About the tool

Shelby Recovery Tool puts the MediaTek download engine and Android
platform-tools behind one desktop interface. Load a scatter firmware, match a
Download Agent to the chipset, and write, read or format partitions without
typing the engine commands. The same window handles Fastboot, GSI installation,
Magisk boot images and Transsion debloat.

Nothing is simulated. Every action runs the bundled binary, streams stdout and
stderr into the log, keeps the real exit code, and stops the sequence when a
step fails. If the engine rejects a DA or an auth file, its message appears in
the log unchanged.

What it covers:

- Scatter firmware flashing in Firmware Upgrade and Download Only modes
- Auth file, preloader and DA slots that stay visible on every MTK tab
- Partition table with file size, location, region and download flag
- Read, write and erase for partitions in the scatter or entered by hand
- Fastboot flashing, erase, `getvar all`, reboot targets and raw commands
- Guided 11-step GSI install with vbmeta, unlock, relock and slot control
- Magisk-patched boot image deployment with a stock vbmeta
- Transsion package scan with remove, restore, disable and enable
- Operation history with a log file for every run
- AMOLED black and white UI and a trace-S mark drawn in code

The app has no account system, no telemetry and no network calls. The frozen
build keeps settings, logs and history in a portable `data` folder beside the
EXE.

## The home screen

The app opens on **MTK Flash**. The layout is fixed. There are no dashboard
cards and the page does not scroll at 720p. The left side is the log, the right
side is the work.

**Title bar** carries the trace-S tile, the product wordmark, the current page
name and the USB state. The window buttons sit on the right.

**Mode rail** runs down the left edge: MTK Flash, Fastboot, GSI, Root and
Transsion. History and Dev sit at the bottom. The active entry gets a white
edge and a lighter label.

**Function pane** is the permanent log. It has a function name, search, copy,
save and clear buttons, the live output, a faint trace-S watermark, and an
elapsed clock with an Idle or Running state at the bottom. Drag the splitter to
change its width; the width is remembered.

**Model row** is an ADB and Fastboot readout. It is not a COM port list. A phone
in MediaTek BROM or preloader mode stays invisible here on purpose, because the
engine attaches to it directly through libusb when a flash starts.

**Firmware row** takes the scatter XML. Drop a firmware folder on the field or
use **Browse**. The images are resolved next to the XML and checked for size
and existence.

**File slots** sit directly under the firmware row: **Auth file**,
**Preloader** and **DA file**. They stay visible on Flash, Service, AntiCrack
and Partition Manager, so a protected device never sends you hunting through a
menu. Click a slot to pick a file.

**Workspace tabs** split the MTK page into Flash, Service, AntiCrack / Trigger
P7 and Partition Manager.

**Partition table** lists what the scatter contains. Ticked rows have an image
present. The package strip above the table shows the scatter name, partition
count, how many will be flashed, and the chipset.

**Action bar** holds Only downloads, Custom partition, Select All, Auto Reboot,
Flash Mode, Region, Read, Write, Erase and the main FLASH button.

### A normal flash

1. Pick the scatter with the **Firmware** field and **Browse**, or drop the
   firmware folder on the field.
2. The DA is selected from `tools\da` using the chipset in the scatter name,
   folder or preloader name. `MT6789`, `K6789` and `MTK6789` all resolve to the
   same chip. If the pick is wrong, click the **DA file** slot.
3. Add an **Auth file** for DAA or SLA protected chips, and a **Preloader** for
   XFlash devices in bootrom mode.
4. Choose **Firmware Upgrade** to write the whole scatter, or **Download Only**
   to write the ticked partitions that have an image. **Region** filters the
   table.
5. Press **FLASH** and confirm. The dialog repeats the chipset, DA and
   preloader. Writes and erases ask again.

**Read all** runs `read-all --skip userdata` and only reads. Keep the phone
connected for the whole operation.

While a task runs, the log prints
`Still running - Ns elapsed, no engine output for Ns.` every five seconds. Long
DA handshakes and large partition writes can be quiet for a while; the
heartbeat makes it clear the app is still working.

## Pages

| Page | What it is for |
| --- | --- |
| **MTK Flash** | Scatter firmware, DA, auth, preloader, partition work and device service calls |
| **Fastboot** | Bootloader operations, image flashing, erase, getvar and raw commands |
| **GSI Installer** | The 11-step GSI flow with vbmeta and slot control |
| **Root Tools** | Magisk-patched boot image and stock vbmeta deployment |
| **Transsion Tools** | Package scan and batch debloat for Transsion firmware |

### Fastboot

The device list has its own refresh, separate from the model row. Device actions
cover reboot to system, bootloader and recovery. The flash card takes a
partition name and an image, with a dedicated **Flash boot** shortcut. Below
that are erase and a raw command field for things like `getvar all` or
`oem device-info`.

### GSI Installer

The installer keeps the same 11 steps as the batch workflow. It accepts ADB,
Fastboot or Fastbootd as the entry method. The progress list shows which step
is running, and the flow stops on the first failure. It handles the vbmeta
verity disable, the empty product and system_ext partitions, the GSI flash and
the slot selection.

Steps 10 and 11 happen on the phone: wipe data and reboot to system. The UI
says so and waits.

### Root Tools

This page flashes a boot image that was already patched with Magisk on the
phone. It does not patch images itself. Pick the patched boot image and the
stock vbmeta from the same firmware, then use **Flash boot image** or the
four-step **Automated root** sequence: reboot to bootloader, flash boot, flash
stock vbmeta, reboot.

The bootloader must already be unlocked. Unlocking wipes data.

### Transsion Tools

The package list has 80 known Transsion entries: 55 safe and 25 all-in-one.
**Scan device** reads the installed state through ADB. The table filters by
name, category or state. Selection helpers cover safe installed, all installed
and visible packages. Remove, restore, disable and enable run in batches
through the same `pm` and `cmd package` calls as the batch workflow.

### Service, AntiCrack and Partition Manager

These are tabs inside MTK Flash, not separate pages.

**Service** holds the backup folder, flash mode, engine and DA status checks,
partition check, storage info, device keys, active slot read and set, reboot,
full backup, and the USB link check. **Check USB link** prints the real
`adb devices -l` and `fastboot devices` output plus any matching Windows USB
interface, then says which part is missing. **Restart ADB** runs
`adb kill-server`, `start-server` and rescans.

**AntiCrack / Trigger P7** shows the selected auth file, verifies the DA
handshake, reads device keys and tracks the four handshake stages. A signed
auth file is required for DAA and SLA protected chips.

**Partition Manager** is a read-only view of every partition in the loaded
scatter, with size, region and download flag.

### History

Every task is stored in `history.json` with its result, elapsed time and
session log. Double-click a row to open the log for that run. History is
available from the clock icon at the bottom of the mode rail.

## Developer

**SHELBY 746** is the name on the About Developer dialog, opened from **Dev**
at the bottom of the mode rail.

The tool grew out of a batch script I used for MediaTek work. The script did
the job, but every option was a menu number, the log scrolled away, and a failed
step was easy to miss. This app keeps the command order and puts the state on
one screen.

It is written in Python with PySide6. The interface, icon and trace-S mark are
drawn in code. Bundled tools and fonts keep their own notices in the repository.

Bug reports are easier to act on with the session log from `data\logs` and the
scatter file name. Check the log for a serial number or account name before
posting it.

## Setup

The full guide is in [SETUP.md](SETUP.md). The short version follows.

### Requirements

- Windows 10 or 11, 64-bit
- Python 3.11 or newer for a source run (I build with 3.13.17)
- The vendor USB driver for the phone
- A data cable, not a charge-only one

No drivers are bundled and the app does not install any.

### Portable EXE

```text
dist\Shelby_Recovery_Tool_v2.2.exe
```

Copy that file anywhere and run it. On first launch it creates a `data` folder
beside itself for settings, logs and history. If the location is read-only, it
falls back to `%APPDATA%\ShelbyRecoveryTool`. Python and Qt do not need to be
installed on the target machine.

### Run from source

```bat
python -m pip install -r requirements.txt
python main.py
```

### Build the EXE

```bat
build.bat
```

The script installs the requirements, regenerates the icon from `ui/brand.py`,
runs the source self-test, builds
`dist\Shelby_Recovery_Tool_v2.2.exe`, then runs the self-test inside the frozen
copy. The result is one file. Python, Qt, ADB, Fastboot, antumbra and the DA
library are all inside it.

Manual build:

```bat
python -m pip install -r requirements.txt
python assets\make_icon.py
python main.py --selftest
python -m PyInstaller --clean --noconfirm Shelby_Recovery_Tool.spec
dist\Shelby_Recovery_Tool_v2.2.exe --selftest
```

### Self-test

```bat
python main.py --selftest
```

The test builds the real window, parses a synthetic scatter, runs a subprocess
through the command runner, checks settings persistence, the package counts,
the bundled tools, the five pages, the log pane, the mode rail, the palette and
the flash workspace. It never touches a phone. There are 24 checks.

A source run writes the report to
`%APPDATA%\ShelbyRecoveryTool\selftest_report.txt`. A frozen run writes it to
`data\selftest_report.txt` beside the EXE.

### Data folder

```text
data\
    config\settings.json
    logs\app.log
    logs\session_YYYYMMDD_HHMMSS.log
    history.json
    selftest_report.txt
    screenshots\
```

Set `SHELBY_DATA_DIR` to use a different location. No registry keys are
written.

## Drivers and detection

The model row only shows ADB and Fastboot devices. A phone in BROM or
preloader mode will not appear there, and that is normal. Load the scatter,
choose the DA and press FLASH; the engine attaches through libusb.

If the row says **No device connected**:

1. Check the cable and try another USB port.
2. Turn on Developer options and USB debugging. Accept the RSA prompt on the
   phone.
3. Set the USB mode to File transfer, not charging only.
4. Install the Google ADB driver or the vendor driver for the phone.
5. Open **MTK Flash > Service > Check USB link**. It prints the real ADB,
   Fastboot and Windows USB results, then says which part is missing.
6. If ADB is stuck, use **Restart ADB** on the same tab.

For MediaTek work, install the vendor VCOM or preloader driver. Windows Device
Manager is the place to confirm the port. The tool has no COM port picker by
design.

`Unauthorized` in the model row means the RSA prompt is still waiting on the
phone. `Offline` usually means the cable or the USB mode, and a replug fixes it.
F5 forces a rescan.

## Command shape

The engine calls keep the argument order from the batch file:

```text
antumbra scatter <scatter.xml> -d <da.bin> [-a <auth>] -p <preloader>
antumbra read-all --skip userdata <folder> -d <da.bin>
antumbra read <partition> <image> -d <da.bin>
antumbra write <partition> <image> -d <da.bin>
antumbra format <partition> -d <da.bin>
antumbra pgpt | storage-info | keys | get-active-slot | set-active-slot
antumbra reboot [normal|home-screen|fastboot|meta|test]
```

ADB and Fastboot call the bundled platform-tools. Reboot modes, partition
names, package lists, GSI steps and the root sequence come from the batch
workflow.

One bug from the batch file is fixed: its `:put` routine wrote lines like
`(FONTNAME=...`, so saved settings were never read back. The GUI stores JSON
instead. Command sequences are otherwise unchanged.

## Project layout

```text
main.py                     entry point, --version, --selftest
core/                       command builders, parsers, tasks, logging
ui/                         theme, icons, widgets, console, main window
ui/pages/                   the five pages
tools/                      adb, fastboot, antumbra, DA library
assets/                     icons and logo exports (source: ui/brand.py)
assets/fonts/               IBM Plex Sans and IBM Plex Mono (OFL)
docs/screenshots/           workspace captures
config/settings.default.json
SETUP.md                    setup, build and troubleshooting
Shelby_Recovery_Tool.spec   PyInstaller definition
version_info.txt            Windows file metadata
build.bat                   build script
requirements.txt
```

## Bundled tools

- `adb.exe`, `fastboot.exe` and the two AdbWin DLLs from Android
  platform-tools.
- `antumbra.exe` 2.0.0, the MediaTek DA engine.
- 15 DA files for MT6765, MT6768, MT6789, MT6833, MT6855, MT6877, MT6878,
  MT6893, MT6895, MT6897, MT6983 and the Transsion `DA_BR_X6871`.
- The platform-tools and antumbra notices.

Tool lookup order is: saved path in `settings.json`, `tools` beside the EXE,
the embedded copy, then `PATH`. The DA library defaults to `tools\da` and can
be overridden with `paths.da_dir`.

## Credits

- Android platform-tools from Google, Apache License 2.0. Notice:
  `tools/NOTICE-platform-tools.txt`.
- antumbra Download Agent engine. Notice: `tools/NOTICE.txt`.
- IBM Plex Sans and IBM Plex Mono under the SIL Open Font License. Notice:
  `assets/fonts/OFL.txt`.
- Download Agent files for the chipsets listed above.

## Notes and limits

- The EXE is unsigned. SmartScreen may warn the first time; PyInstaller
  one-file bundles and flashing tools are common false positives.
- GSI steps 10 and 11 are physical actions on the phone. The UI says so and
  waits.
- Bootloader unlock wipes data.
- Fastboot must be able to see the phone. If it cannot, install the Google or
  vendor USB driver first.
- A BROM or preloader phone is invisible in the model row by design. Load the
  scatter, choose the DA and press FLASH.

Built and tested on Windows 11 with Python 3.13.17, PySide6 6.12.0,
PyInstaller 6.22.3 and Pillow 13.0.0. The source and frozen self-tests both pass
all 24 checks.
