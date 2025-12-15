<p align="center">
  <a href="https://github.com/revoxxi/">
    <img src="https://img.shields.io/badge/Firmware-Marlin%202.1.2-blue?style=for-the-badge" alt="Marlin 2.1.2">
  </a>
  <img src="https://img.shields.io/badge/Printer-Ender%203%20V3%20SE-orange?style=for-the-badge" alt="Ender 3 V3 SE">
  <img src="https://img.shields.io/badge/Profile-Custom%20Tuned-success?style=for-the-badge" alt="Custom tuned profile">
</p>
This version of marlin is modified to make serial connection (print over OctoPrint as an example) more stable without resends, checksum errors, buffer overflows or other issues present in other custom firmware. For complicated G-code I included Arc. To enable this in Cura go to Marketplace, search Arc Welder, restart Cura and select your model and check the Arc Weld on the right sides settings. Other slicers have other means to enable arcs.

> **IMPORTANT — Hardware compatibility**
>
> This firmware targets the Ender 3 V3 SE with the Creality C14 motherboard (for example: `CR4NS200320C14`). Do NOT flash this firmware on C13 or other board revisions — flashing the wrong hardware revision can brick the board. See [How to check whether your board is C13 or C14](docs/C14_check.md) for details.

## PlatformIO Inspection

RAM:   [==        ]  23.5% (used 15416 bytes from 65536 bytes)
Flash: [====      ]  41.9% (used 215416 bytes from 514288 bytes)

## Table of Contents
- [Overview](#overview)
- [Version](#version)
- [Installation](#installation)
- [Included Binary](#included-binary)
- [Memory Usage](#memory-usage)
- [Compatibility](#compatibility)
- [Building](#building)
- [Features](#features)
- [Contributing](#contributing)
- [Support](#support)
- [License](#license)


## Overview

This repository contains a customized Marlin 2.1.2 firmware build tuned for the Creality Ender‑3 V3 SE (C14 motherboard). The project focuses on a modular, testable configuration that stays within the memory limits of the stock C14 board while providing commonly requested UX and motion features.


## Version

- Firmware base: Marlin bugfix-2.1.x (navaismo https://github.com/navaismo/Marlin_bugfix_2.1_E3V3SE)
- Project release: 2.1.2 (this repository)
- Maintainer: Revoxxi (custom modifications and configuration)


## Installation

1. Verify your board is a Creality C14. Do NOT flash this firmware on C13 or other revisions; doing so can brick the board. See `docs/C14_check.md` for identification steps.
2. Use the prebuilt binary in `.pio/build/STM32F401RE_creality/firmware.bin` or build your own (instructions below).
3. Follow the flashing instructions in `docs/Flashing.md` or the project's wiki pages. Always keep a working backup of your original firmware and configuration.


## Included Binary

The distributed binary is a conservative feature set chosen to balance functionality and memory usage. It includes:
- DWIN thumbnail rendering
- D routine auto Z offset
- Input shaping (optional via UI)
- Linear Advance
- LCD dim/brightness menu and extra UI tweaks
- CRTouch helpers and test functions

If you need a different feature set, build a custom binary from the provided `Marlin` configuration files.


## Memory Usage

Measured from a representative build (PlatformIO environment `STM32F401RE_creality`):

```
RAM:   15,416 bytes (23.5% of 65536 bytes)
Flash: 215,448 bytes (41.9% of ~514288 bytes)
```

These values are from a local build and may change when enabling/disabling features. Use `PlatformIO Home → Project Inspect` or `platformio run -t size` to re-check memory usage after changes.


## Compatibility

This firmware targets the Ender‑3 V3 SE with the Creality C14 motherboard (for example: `CR4NS200320C14`). Do NOT flash on C13 or other revisions — check `docs/C14_check.md` before proceeding.


## Building

Recommended: Visual Studio Code with the PlatformIO extension.

To build from the repository root with the included virtual environment:

```powershell
# From repo root
.venv\Scripts\python.exe -m platformio run -e STM32F401RE_creality
```

To inspect sizes:

```powershell
.venv\Scripts\python.exe -m platformio run -e STM32F401RE_creality -t size
```

To build a custom configuration, edit `Marlin/Configuration.h` and `Marlin/Configuration_adv.h` then rebuild.


## Features

The project includes a number of user-facing and motion-related features; highlights:
- Input Shaping (configurable per-axis)
- FT_MOTION / FTM_SMOOTHING for trajectory smoothing
- Linear Advance
- DWIN UI additions: thumbnail, dim/brightness, custom extrude, preheat labels
- CRTouch utilities and enhanced Z offset routines

If you want to further reduce memory/serial pressure, consider disabling Input Shaping, reducing `FTM_BUFFER_SIZE`, or lowering `TX_BUFFER_SIZE`/`BLOCK_BUFFER_SIZE` in `Marlin/Configuration_adv.h`.


## Contributing

Contributions are welcome. Please open pull requests against the `bugfix-2.1.x` branch and include unit or build tests where applicable. See project contribution notes and use `make tests-config-single-local` or PlatformIO test targets to validate changes.


## Support

For documentation and community help, see the Marlin docs and community channels. For project-specific questions, open an issue in this repository describing firmware version `2.1.2`, platform `STM32F401RE_creality`, and the exact behavior observed.


## License

The project follows Marlin's GPL licensing. The `## License` section below contains the full GPL statement.

---

  /**
   * Advanced configuration
   */
  #define FTM_BUFFER_SIZE             128   // Window size for trajectory generation, must be a power of 2 (e.g 64, 128, 256, ...)
                                            // The total buffered time in seconds is (FTM_BUFFER_SIZE/FTM_FS)
  #define FTM_FS                     1000   // (Hz) Frequency for trajectory generation.
  #define FTM_STEPPER_FS        2'000'000   // (Hz) Time resolution of stepper I/O update. Shouldn't affect CPU much (slower board testing needed)
  #define FTM_MIN_SHAPE_FREQ           20   // (Hz) Minimum shaping frequency, lower consumes more RAM

#endif // FT_MOTION
``` 


> [!TIP]
> 
> Check the [Octoprint Pinput-Shaping plugin](https://github.com/revoxxi/Octoprint-Pinput_Shaping) to get the frequencies.
>
> Check the [comparison between Marlin & Klipper Input Shaping](https://github.com/revoxxi/Octoprint-Pinput_Shaping/discussions/27).
>

<br>

### * **AXIS TWIST COMPENSATION** 
```c++
 // Add calibration in the Probe Offsets menu to compensate for X-axis twist.
    #define X_AXIS_TWIST_COMPENSATION
    #if ENABLED(X_AXIS_TWIST_COMPENSATION)
      /**
       * Enable to init the Probe Z-Offset when starting the Wizard.
       * Use a height slightly above the estimated nozzle-to-probe Z offset.
       * For example, with an offset of -5, consider a starting height of -4.
       */
      #define XATC_START_Z 0.0
      #define XATC_MAX_POINTS 3             // Number of points to probe in the wizard
      #define XATC_Y_POSITION Y_CENTER      // (mm) Y position to probe
      #define XATC_Z_OFFSETS { 0, 0, 0 }    // Z offsets for X axis sample points
    #endif
```    


### * Marlin 2.1.2 is also compatible with the [CacomixtlePad for Android](https://github.com/revoxxi/cacomixtlePad)

![demo](./media/demo_fast.gif)



And so on...


<br>
<br>
<hr>
<br>
<p align="center"><img src="buildroot/share/pixmaps/logo/marlin-outrun-nf-500.png" height="250" alt="MarlinFirmware's logo" /></p>

<h1 align="center">Marlin 3D Printer Firmware</h1>

<p align="center">
    <a href="/LICENSE"><img alt="GPL-V3.0 License" src="https://img.shields.io/github/license/marlinfirmware/marlin.svg"></a>
    <a href="//github.com/MarlinFirmware/Marlin/graphs/contributors"><img alt="Contributors" src="https://img.shields.io/github/contributors/marlinfirmware/marlin.svg"></a>
    <a href="//github.com/MarlinFirmware/Marlin/releases"><img alt="Last Release Date" src="https://img.shields.io/github/release-date/MarlinFirmware/Marlin"></a>
    <a href="//github.com/MarlinFirmware/Marlin/actions/workflows/ci-build-tests.yml"><img alt="CI Status" src="https://github.com/MarlinFirmware/Marlin/actions/workflows/ci-build-tests.yml/badge.svg"></a>
    <a href="//github.com/sponsors/thinkyhead"><img alt="GitHub Sponsors" src="https://img.shields.io/github/sponsors/thinkyhead?color=db61a2"></a>
    <br />
    <a href="//bsky.app/profile/marlinfw.org"><img alt="Follow marlinfw.org on Bluesky" src="https://img.shields.io/badge/Follow%20@marlinfw.org-0085ff?logo=bluesky&logoColor=white"></a>
    <a href="//fosstodon.org/@marlinfirmware"><img alt="Follow MarlinFirmware on Mastodon" src="https://img.shields.io/mastodon/follow/109450200866020466?domain=https%3A%2F%2Ffosstodon.org&logoColor=%2300B&style=social"></a>
</p>

### 🌍 Translations

<table>
<tr>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=an">Aragonés</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=bg">Български</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=ca">Català</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=cs">Čeština</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=da">Dansk</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=de">Deutsch</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=el">Ελληνικά</a></td>
</tr>
<tr>
  <td><a href="//github.com/MarlinFirmware/Marlin">English</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=es">Español</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=eu">Euskara</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=fi">Suomi</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=fr">Français</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=gl">Galego</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=hr">Hrvatski</a></td>
</tr>
<tr>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=hu">Magyar</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=it">Italiano</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=ja">にほんご</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=ko">한국어</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=nl">Nederlands</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=pl">Polski</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=pt">Português</a></td>
</tr>
<tr>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=pt-BR">Português (Brasil)</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=ro">Română</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=ru">Русский</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=sk">Slovenčina</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=sv">Svenska</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=tr">Türkçe</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=uk">Українська</a></td>
</tr>
<tr>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=vi">Tiếng Việt</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=zh-CN">简体中文</a></td>
  <td><a href="//translate.google.com/translate?u=github.com/MarlinFirmware/Marlin&sl=auto&tl=zh-TW">繁體中文</a></td>
  <td></td>
  <td></td>
  <td></td>
  <td></td>
</tr>
</table>

Official documentation can be found at the [Marlin Home Page](//marlinfw.org/).

Please test this firmware and let us know if it misbehaves in any way. Volunteers are standing by!

---

## Marlin 2.1 Bugfix Branch

**Not for production use. Use with caution!**

Marlin 2.1 supports both 32-bit ARM and 8-bit AVR boards while adding support for up to 9 coordinated axes and to up to 8 extruders.

This branch is for patches to the latest 2.1.2 release version. Periodically this branch will form the basis for the next minor 2.1.2 release.

Download earlier versions of Marlin on the [Releases page](//github.com/MarlinFirmware/Marlin/releases).

## Example Configurations

Before you can build Marlin for your machine you'll need a configuration for your specific hardware. Upon request, your vendor will be happy to provide you with the complete source code and configurations for your machine, but you'll need to get updated configuration files if you want to install a newer version of Marlin. Fortunately, Marlin users have contributed hundreds of tested configurations to get you started. Visit the [MarlinFirmware/Configurations](//github.com/MarlinFirmware/Configurations) repository to find the right configuration for your hardware. Make sure to select a compatible branch! [The Marlin Download Page](//marlinfw.org/meta/download/) matches compatible software and configuration packages.

## Building Marlin 2.1

To build and upload Marlin you will use one of these tools:

- The free [Visual Studio Code](//code.visualstudio.com/download) using the [Auto Build Marlin](//marlinfw.org/docs/basics/auto_build_marlin.html) extension.
- Marlin is optimized to build with the [PlatformIO IDE](//platformio.org/) extension for Visual Studio Code.
- You can also use VSCode with devcontainer : See [Installing Marlin (VSCode devcontainer)](http://marlinfw.org/docs/basics/install_devcontainer_vscode.html).
- You can still build Marlin with [Arduino IDE](//www.arduino.cc/en/main/software) : See [Building Marlin with Arduino](//marlinfw.org/docs/basics/install_arduino.html). We hope to improve the Arduino build experience, but at this time, PlatformIO is the preferred choice.

## 32-bit ARM boards

Marlin is compatible with a plethora of 32-bit ARM boards, which offer ample computational power and memory and allows Marlin to deliver state-of-the-art performance and features we like to see in modern 3d printers. Some of the newer features in Marlin will require use of a 32-bit ARM board.

## 8-Bit AVR Boards

Marlin originates from the era of Arduino based 8-bit boards, and we aim to support 8-bit AVR boards in perpetuity. Both 32-bit and 8-bit boards are covered by a single code base that can apply to all machines. Our goal is to support casual hobbyists, tinkerers, and owners of older machines and boards, striving to allow them to benefit from the community's innovations just as much as those with fancier machines and newer baords. In addition, these venerable AVR-based machines are often the best for testing and feedback!

## Hardware Abstraction Layer (HAL)

Marlin's Hardware Abstraction Layer provides a common API for all the platforms it targets. This allows Marlin code to address the details of motion and user interface tasks at the lowest and highest levels with no system overhead, tying all events directly to the hardware clock.

Every new HAL opens up a world of hardware. Marlin currently has HALs for more than a dozen platforms. While AVR and STM32 are the most well known and popular ones, others like ESP32 and LPC1768 support a variety of less common boards. At this time, an HAL for RP2040 is available in beta; we would like to add one for the Duet3D family of boards. A HAL that wraps an RTOS is an interesting concept that could be explored.

Did you know that Marlin includes a Simulator that can run on Windows, macOS, and Linux? Join the Discord to help move these sub-projects forward!

### Supported Platforms

| Platform                                                                                                                                                                                         | MCU                              | Example Boards                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------- | ---------------------------------------------------------- |
| [Arduino AVR](//www.arduino.cc/)                                                                                                                                                                 | ATmega                           | RAMPS, Melzi, RAMBo                                        |
| [Teensy++ 2.0](//www.microchip.com/en-us/product/AT90USB1286)                                                                                                                                    | AT90USB1286                      | Printrboard                                                |
| [Arduino Due](//www.arduino.cc/en/Guide/ArduinoDue)                                                                                                                                              | SAM3X8E                          | RAMPS-FD, RADDS, RAMPS4DUE                                 |
| [ESP32](//github.com/espressif/arduino-esp32)                                                                                                                                                    | ESP32                            | FYSETC E4, E4d@BOX, MRR                                    |
| [GD32](//www.gigadevice.com/)                                                                                                                                                                    | GD32 ARM Cortex-M4               | Creality MFL GD32 V4.2.2                                   |
| [HC32](//www.huazhoucn.com/)                                                                                                                                                                     | HC32                             | Ender-2 Pro, Voxelab Aquila                                |
| [LPC1768](//www.nxp.com/products/processors-and-microcontrollers/arm-microcontrollers/general-purpose-mcus/lpc1700-cortex-m3/512-kb-flash-64-kb-sram-ethernet-usb-lqfp100-package:LPC1768FBD100) | ARM® Cortex-M3                  | MKS SBASE, Re-ARM, Selena Compact                          |
| [LPC1769](//www.nxp.com/products/processors-and-microcontrollers/arm-microcontrollers/general-purpose-mcus/lpc1700-cortex-m3/512-kb-flash-64-kb-sram-ethernet-usb-lqfp100-package:LPC1769FBD100) | ARM® Cortex-M3                  | Smoothieboard, Azteeg X5 mini, TH3D EZBoard                |
| [Pico RP2040](//www.raspberrypi.com/documentation/microcontrollers/pico-series.html)                                                                                                             | Dual Cortex M0+                  | BigTreeTech SKR Pico                                       |
| [STM32F103](//www.st.com/en/microcontrollers-microprocessors/stm32f103.html)                                                                                                                     | ARM® Cortex-M3                  | Malyan M200, GTM32 Pro, MKS Robin, BTT SKR Mini            |
| [STM32F401](//www.st.com/en/microcontrollers-microprocessors/stm32f401.html)                                                                                                                     | ARM® Cortex-M4                  | ARMED, Rumba32, SKR Pro, Lerdge, FYSETC S6, Artillery Ruby |
| [STM32F7x6](//www.st.com/en/microcontrollers-microprocessors/stm32f7x6.html)                                                                                                                     | ARM® Cortex-M7                  | The Borg, RemRam V1                                        |
| [STM32G0B1RET6](//www.st.com/en/microcontrollers-microprocessors/stm32g0x1.html)                                                                                                                 | ARM® Cortex-M0+                 | BigTreeTech SKR mini E3 V3.0                               |
| [STM32H743xIT6](//www.st.com/en/microcontrollers-microprocessors/stm32h743-753.html)                                                                                                             | ARM® Cortex-M7                  | BigTreeTech SKR V3.0, SKR EZ V3.0, SKR SE BX V2.0/V3.0     |
| [SAMD21P20A](//www.adafruit.com/product/4064)                                                                                                                                                    | ARM® Cortex-M0+                 | Adafruit Grand Central M4                                  |
| [SAMD51P20A](//www.adafruit.com/product/4064)                                                                                                                                                    | ARM® Cortex-M4                  | Adafruit Grand Central M4                                  |
| [Teensy 3.2/3.1](//www.pjrc.com/teensy/teensy31.html)                                                                                                                                            | MK20DX256VLH7 ARM® Cortex-M4    |
| [Teensy 3.5](//www.pjrc.com/store/teensy35.html)                                                                                                                                                 | MK64FX512-VMD12 ARM® Cortex-M4  |
| [Teensy 3.6](//www.pjrc.com/store/teensy36.html)                                                                                                                                                 | MK66FX1MB-VMD18 ARM® Cortex-M4  |
| [Teensy 4.0](//www.pjrc.com/store/teensy40.html)                                                                                                                                                 | MIMXRT1062-DVL6B ARM® Cortex-M7 |
| [Teensy 4.1](//www.pjrc.com/store/teensy41.html)                                                                                                                                                 | MIMXRT1062-DVJ6B ARM® Cortex-M7 |
| Linux Native                                                                                                                                                                                     | x86 / ARM / RISC-V               | Raspberry Pi GPIO                                          |
| Simulator                                                                                                                                                                                        | Windows, macOS, Linux            | Desktop OS                                                 |
| [All supported boards](//marlinfw.org/docs/hardware/boards.html#boards-list)                                                                                                                     | All platforms                    | All boards                                                 |

## Marlin Support

The Issue Queue is reserved for Bug Reports and Feature Requests. Please use the following resources for help with configuration and troubleshooting:

- [Marlin Documentation](//marlinfw.org) - Official Marlin documentation
- [Marlin Discord](//discord.com/servers/marlin-firmware-461605380783472640) - Discuss issues with Marlin users and developers
- Facebook Group ["Marlin Firmware"](//www.facebook.com/groups/1049718498464482/)
- RepRap.org [Marlin Forum](//forums.reprap.org/list.php?415)
- Facebook Group ["Marlin Firmware for 3D Printers"](//www.facebook.com/groups/3Dtechtalk/)
- [Marlin Configuration](//www.youtube.com/results?search_query=marlin+configuration) on YouTube

## Contributing Patches

You can contribute patches by submitting a Pull Request to the ([bugfix-2.1.2](//github.com/MarlinFirmware/Marlin/tree/bugfix-2.1.2)) branch.

- We use branches named with a "bugfix" or "dev" prefix to fix bugs and integrate new features.
- Follow the [Coding Standards](//marlinfw.org/docs/development/coding_standards.html) to gain points with the maintainers.
- Please submit Feature Requests and Bug Reports to the [Issue Queue](//github.com/MarlinFirmware/Marlin/issues/new/choose). See above for user support.
- Whenever you add new features, be sure to add one or more build tests to `buildroot/tests`. Any tests added to a PR will be run within that PR on GitHub servers as soon as they are pushed. To minimize iteration be sure to run your new tests locally, if possible.
  - Local build tests:
    - All: `make tests-config-all-local`
    - Single: `make tests-config-single-local TEST_TARGET=...`
  - Local build tests in Docker:
    - All: `make tests-config-all-local-docker`
    - Single: `make tests-config-all-local-docker TEST_TARGET=...`
  - To run all unit test suites:
    - Using PIO: `platformio run -t test-marlin`
    - Using Make: `make unit-test-all-local`
    - Using Docker + make: `maker unit-test-all-local-docker`
  - To run a single unit test suite:
    - Using PIO: `platformio run -t marlin_<test-suite-name>`
    - Using make: `make unit-test-single-local TEST_TARGET=<test-suite-name>`
    - Using Docker + make: `maker unit-test-single-local-docker TEST_TARGET=<test-suite-name>`
- If your feature can be unit tested, add one or more unit tests. For more information see our documentation on [Unit Tests](test).

## Contributors

Marlin is constantly improving thanks to a huge number of contributors from all over the world bringing their specialties and talents. Huge thanks are due to [all the contributors](//github.com/MarlinFirmware/Marlin/graphs/contributors) who regularly patch up bugs, help direct traffic, and basically keep Marlin from falling apart. Marlin's continued existence would not be possible without them.

Marlin Firmware original logo design by Ahmet Cem TURAN [@ahmetcemturan](//github.com/ahmetcemturan).

## Project Leadership

| Name                 | Role         | Link                                         | Donate                                                                |
| -------------------- | ------------ | -------------------------------------------- | --------------------------------------------------------------------- |
| 🇺🇸 Scott Lahteine    | Project Lead | [[@thinkyhead](//github.com/thinkyhead)]     | [💸 Donate](//marlinfw.org/docs/development/contributing.html#donate) |
| 🇺🇸 Roxanne Neufeld   | Admin        | [[@Roxy-3D](//github.com/Roxy-3D)]           |
| 🇺🇸 Keith Bennett     | Admin        | [[@thisiskeithb](//github.com/thisiskeithb)] | [💸 Donate](//github.com/sponsors/thisiskeithb)                       |
| 🇺🇸 Jason Smith       | Admin        | [[@sjasonsmith](//github.com/sjasonsmith)]   |
| 🇧🇷 Victor Oliveira   | Admin        | [[@rhapsodyv](//github.com/rhapsodyv)]       |
| 🇬🇧 Chris Pepper      | Admin        | [[@p3p](//github.com/p3p)]                   |
| 🇳🇿 Peter Ellens      | Admin        | [[@ellensp](//github.com/ellensp)]           | [💸 Donate](//ko-fi.com/ellensp)                                      |
| 🇺🇸 Bob Kuhn          | Admin        | [[@Bob-the-Kuhn](//github.com/Bob-the-Kuhn)] |
| 🇳🇱 Erik van der Zalm | Founder      | [[@ErikZalm](//github.com/ErikZalm)]         |

## Star History

<a id="starchart" href="//star-history.com/#MarlinFirmware/Marlin&Date">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=MarlinFirmware/Marlin&type=Date&theme=dark" />
    <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=MarlinFirmware/Marlin&type=Date" />
    <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=MarlinFirmware/Marlin&type=Date" />
  </picture>
</a>

## License

Marlin is published under the [GPL license](/LICENSE) because we believe in open development. The GPL comes with both rights and obligations. Whether you use Marlin firmware as the driver for your open or closed-source product, you must keep Marlin open, and you must provide your compatible Marlin source code to end users upon request. The most straightforward way to comply with the Marlin license is to make a fork of Marlin on Github, perform your modifications, and direct users to your modified fork.
