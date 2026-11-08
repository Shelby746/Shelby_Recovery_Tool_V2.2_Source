# Setup and build

Everything below is for the source folder. The app itself is a Windows desktop
tool; there is no server, database or installer.

## 1. Run the portable EXE

If you already built the tool, run:

```text
dist\Shelby_Recovery_Tool_v2.2.exe
```

That is the whole setup. The EXE creates a `data` folder beside itself on first
launch for settings, logs and history. If the folder is read-only it uses
`%APPDATA%\ShelbyRecoveryTool` instead.

You still need the phone's USB driver. No drivers are bundled.

## 2. Run from source

Requirements:

- Windows 10 or 11, 64-bit
- Python 3.11 or newer, 64-bit
- Internet access for the first `pip install`

From the source folder:

```bat
python -m pip install -r requirements.txt
python main.py
```

That starts the same window without building an EXE. Use this while editing the
code; changes show up on the next launch.

## 3. Build the EXE

The normal way:

```bat
build.bat
```

The script does five things:

1. installs `requirements.txt`
2. regenerates the icon files from `ui/brand.py`
3. runs the source self-test
4. builds `dist\Shelby_Recovery_Tool_v2.2.exe`
5. runs the self-test inside the frozen EXE and removes the temporary
   `dist\data` folder

If the build fails, read the last lines in the console. The usual causes are
Python not on `PATH`, a missing package, or a running copy of the EXE locking
`dist`.

Manual build:

```bat
python -m pip install -r requirements.txt
python assets\make_icon.py
python main.py --selftest
python -m PyInstaller --clean --noconfirm Shelby_Recovery_Tool.spec
dist\Shelby_Recovery_Tool_v2.2.exe --selftest
```

Build output is one file:

```text
dist\
    Shelby_Recovery_Tool_v2.2.exe
```

Copy it anywhere and run it. Python and Qt do not need to be installed on the
target machine.

## 4. Run the self-test

```bat
python main.py --selftest
```

The test builds the real window, parses a sample scatter file, runs a real
subprocess through the command runner, checks the settings round-trip and the
bundled tools, and confirms the five pages. It does not touch a phone. There
are 24 checks.

A source run writes the report to
`%APPDATA%\ShelbyRecoveryTool\selftest_report.txt`. A frozen run writes it to
`data\selftest_report.txt` beside the EXE.

## 5. Data folder

```text
data\
    config\settings.json
    logs\app.log
    logs\session_YYYYMMDD_HHMMSS.log
    history.json
    selftest_report.txt
    screenshots\
```

Set `SHELBY_DATA_DIR` to use a different folder. No registry keys are created.

## 6. Drivers

Install these yourself:

- **ADB / Fastboot**: Google USB driver or the vendor driver from the phone's
  support page.
- **MediaTek**: vendor USB VCOM / preloader driver for the chipset.

The model row only shows ADB and Fastboot devices. A phone in BROM or preloader
mode will not appear there; that is normal. Press **FLASH** and the engine
attaches it through libusb.

If detection fails, open **MTK Flash → Service → Check USB link**. It prints the
real ADB, Fastboot and Windows USB results. **Restart ADB** recycles the bundled
ADB server.

## 7. Common problems

**`python` is not recognized**

Install Python 3.11+ 64-bit and enable "Add Python to PATH".

**`pip install` fails**

Check the internet connection and make sure you are using 64-bit Python. A
virtual environment is fine:

```bat
python -m venv .venv
.venv\Scripts\activate
python -m pip install -r requirements.txt
```

**Build says it cannot clear `dist`**

Close the running `Shelby_Recovery_Tool_v2.2.exe` and run `build.bat` again.

**SmartScreen warns about the EXE**

The EXE is unsigned. PyInstaller one-file builds and flashing tools are common
false positives. Compare the SHA-256 with the one you published, then allow the
file if it matches.

**No device in the model row**

Check the cable, enable USB debugging, accept the RSA prompt, set USB mode to
File transfer, and run **Check USB link**. A BROM/preloader phone stays out of
the row by design.
