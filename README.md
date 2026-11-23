# uwuhook

A 64-bit Windows 10/11 usermode application written in C++ that acts as a KeyAuth-authenticated loader and panel launcher, wrapped in a modern ImGui-based GUI with multi-layer anti-analysis protection.

> [!NOTE]
> **Project Origin & Attribution**: This source code was self-leaked by the original author (ego / @humbleness on Discord) for educational purposes. Please do not redistribute without giving credit. Original project site: https://e-z.bio/%C2%A3

> [!WARNING]
> This application interacts with sensitive system areas and embeds bypass and driver binaries as inline byte arrays. It is intended for educational and security research purposes only.

---

## Features
* **KeyAuth License Login**: Key entry field on the login page authenticates via the [KeyAuth](https://keyauth.win) API (`KeyAuthApp.init()` + `KeyAuthApp.license()`). Supports a server selector (United Kingdom / Russia) with simulated per-server connection delays.
* **Credential Saving**: A **Save credentials** checkbox writes the entered license key to `C:\license.uwu` (or `%USERPROFILE%\Documents\license.uwu` as fallback).
* **Panel Launcher**: Post-login page presents a four-entry product list — Trace Cleaner, FiveM Internal, Github, and Discord — each with live status indicators (Undetected / Detected), relative last-updated timestamps, and expiry countdowns.
* **Simulated Loading Bar**: The Load button triggers a multi-stage progress bar with randomised jitter, easing curves, and occasional artificial stalls to mimic real loading behaviour.
* **Trace Cleaner Stub**: The Trace Cleaner entry calls a commented-out `modules::c_modules` cleaning pipeline. The full PC-cleaning module is available separately (contact the author).
* **Embedded Bypass Binary**: `headers/bypass.h` stores a 33,792-byte PE executable as `bypass_data[]` for future use.
* **Embedded Driver Binary**: `headers/driver.h` stores a 9,216-byte PE (targeting `\Device\{EpomakerKernel}`) as `detected_driverlol[]`, described by the author as a UD kernel driver.
* **Process Rename**: `LegitRename::Rename()` in `headers/rename.h` changes the running process name at runtime for evasion.
* **Embedded Startup Sound**: A WAV file is stored inline in `headers/intro.h` as `wavData[]` and played via `PlaySoundA` on launch.
* **Streamer-Safe Window**: The window is created with an obfuscated, randomised class/title string via `xorstr` and positioned centred on screen using `WS_POPUP` style with no titlebar.
* **External Link Shortcuts**: The Github entry opens `https://github.com/disbuted`; the Discord entry opens `https://e-z.bio/%C2%A3` via `ShellExecuteA`.

---

## Multi-Layer Protection System

Protection is applied at startup via VMProtect and static obfuscation libraries. The `main()` entry point is wrapped in `VMProtectBeginUltra("main")` / `VMProtectEnd()`.

### Obfuscation Libraries (`protection/`)

| Module | Files | Description |
| :--- | :--- | :--- |
| **VMProtect** | `VMProtect/VMProtectSDK.h` + `VMProtectSDK64.dll/.lib` | Wraps sensitive routines in `VMProtectBeginUltra` / `VMProtectBegin` / `VMProtectEnd` blocks |
| **Lazy Importer** | `lazy importer/lazy_importer.hpp` | Resolves Windows API imports at runtime to avoid static import table analysis |
| **ObfusHeader** | `obfusheader/obfusheader.h` | Compile-time obfuscation macros applied to constants and control flow throughout the codebase |
| **XOR String** | `libs/xor/xorstr.hpp` | Compile-time XOR encryption for string literals (`xorstr_`, `xorstr`) |

---

## Authentication System (`libs/keyauth/`)

Login is handled through the [KeyAuth](https://keyauth.win) API using the files in `libs/keyauth/`:

| File | Role |
| :--- | :--- |
| `auth.hpp` | Main KeyAuth API wrapper (`api` class, `license()`, `init()`) |
| `json.hpp` | Bundled JSON parser for API response handling |
| `skStr.h` | `skCrypt` compile-time string encryption for API credentials |
| `utils.hpp` | Utility helpers for the auth flow |

The login page presents a key input field, a server selector (United Kingdom / Russia), and a **Save credentials** checkbox that writes the license key to `C:\license.uwu` (or `%USERPROFILE%\Documents\license.uwu` as fallback). Successful authentication advances to the panel selection page.

---

## User Interface

Built on **Dear ImGui** (DirectX 11 backend) with a FreeType font renderer using `tahoma.ttf`. Window dimensions are fixed at **460 × 400** pixels. The UI is split into two pages:

### Page 0 — Register
Key entry, server selection (United Kingdom / Russia), Login / Exit buttons, and a credential save checkbox.

### Page 1 — Panel Selection
A four-entry list with per-entry status indicators, relative timestamps, and a simulated human-like loading progress bar (randomised jitter, easing, and occasional stalls):

| Entry | Description | Default Status |
| :--- | :--- | :--- |
| Trace Cleaner | PC trace cleaning stub (full module sold separately) | Undetected |
| FiveM Internal | FiveM cheat loader placeholder | Detected |
| Github | Opens `github.com/disbuted` in the default browser | Undetected |
| Discord | Opens author's bio link | Undetected |

---

## Project Structure

```
uwuhook/
├── build/Release/          # Compiled output: VMProtectSDK64.dll, license.bin, msutil.pdb
├── headers/
│   ├── bypass.h            # Embedded 33KB PE executable as bypass_data[] byte array
│   ├── driver.h            # Embedded 9KB kernel driver PE as detected_driverlol[] byte array
│   ├── images.h            # Embedded image resources
│   ├── intro.h             # Embedded WAV startup sound as wavData[] byte array
│   └── rename.h            # LegitRename: runtime process name spoofing
├── imgui/                  # Dear ImGui source + FreeType integration
│   └── backend/            # imgui_impl_dx11 + imgui_impl_win32
├── libs/
│   ├── keyauth/            # KeyAuth API: auth.hpp, json.hpp, skStr.h, utils.hpp
│   └── xor/xorstr.hpp      # Compile-time XOR string encryption
├── main/
│   ├── includes.hpp        # All standard + Windows + external headers
│   ├── main.cpp            # Entry point, Win32 window creation, DX11 render loop
│   ├── main.hpp            # c_main class, KeyAuth init, UI page definitions, GameInfo struct
│   └── rename.cpp          # LegitRename implementation
├── protection/
│   ├── lazy importer/      # lazy_importer.hpp: runtime API resolution
│   ├── obfusheader/        # obfusheader.h: compile-time code obfuscation
│   └── VMProtect/          # VMProtectSDK.h + VMProtectSDK64.dll/lib
├── resource/
│   └── app.ico             # Application icon
├── ui/
│   ├── fonts.hpp           # Font atlas definitions (c_font class, fonts_ enum, FreeType config)
│   └── ui.cpp/.hpp         # Render loop helpers, page system, child window wrappers
├── catpaw client loader.vcxproj   # Visual Studio project file
├── library_x64.lib                # Precompiled support library
├── resource.h                     # Windows resource definitions
├── uwuhook.rc                     # Resource script (icon, version info)
└── uwuhook.sln                    # Visual Studio solution
```

---

## Quick Setup

### 1. Prerequisites
* Visual Studio 2022 with the **Desktop development with C++** workload
* Windows SDK 10.0+
* DirectX SDK (or use the Windows SDK bundled version)

### 2. Compile
1. Open [uwuhook.sln](uwuhook/uwuhook.sln) in Visual Studio.
2. Build under **Release | x64** to produce the loader executable.
3. The `build/Release/` directory must contain `VMProtectSDK64.dll` alongside the compiled binary at runtime.

### 3. Configure
Before building, update the KeyAuth credentials in `main/main.hpp`:
```cpp
std::string name    = skCrypt("your_app_name").decrypt();
std::string ownerid = skCrypt("your_owner_id").decrypt();
```
The `version` and `url` fields are pre-filled with `"1.0"` and the KeyAuth v1.3 endpoint. The `path` field is left empty by default.

---

## Disclaimer
This software is provided for educational, security research, and testing purposes only. Portions of this codebase were modified, updated, or developed with the assistance of Artificial Intelligence (AI) models. The authors and contributors assume no liability for misuse, system damage, or issues arising from the deployment of this application in production or untrusted environments. The full PC trace-cleaning module is not included in this release — contact the author via Discord if you are interested.
