<div align="center">

# Maxsun B760M 13490F D4 Hackintosh — Sequoia 15

**OpenCore EFI for macOS Sequoia 15.7.9 on a Maxsun Terminator B760M D4 · i5-13490F · RX 570 · AX210 build**

![macOS](https://img.shields.io/badge/macOS-Sequoia%2015.7.9-000000?logo=apple&logoColor=white) ![OpenCore](https://img.shields.io/badge/OpenCore-1.0.8-9cf) ![CPU](https://img.shields.io/badge/CPU-i5--13490F-0071c5?logo=intel&logoColor=white) ![GPU](https://img.shields.io/badge/GPU-RX_570_4GB-ed1c24?logo=amd&logoColor=white) ![Wi-Fi](https://img.shields.io/badge/Wi--Fi%20%2B%20BT-AX210-0071c5?logo=intel&logoColor=white) ![Status](https://img.shields.io/badge/status-verified-brightgreen)

**English** &nbsp;·&nbsp; [简体中文](README.zh-CN.md)

</div>

---

> **You are on the `sequoia-15` branch.** The [`main`](../../tree/main) branch carries the **macOS Tahoe 26** build for the same machine. The two are not interchangeable — SMBIOS, the Wi-Fi driver's kernel gate and the USB strategy all differ. Pick the branch that matches the OS you are installing.

## Overview

This is the macOS **Sequoia 15** configuration for the same hardware documented in `main`. It was not produced by downgrading the Tahoe EFI — it was rebuilt from the machine's working Sonoma-era EFI, then carried forward to 15.7.9.

The guiding rule: change only what macOS 15 actually breaks, and keep every workaround that exists purely because of Tahoe out of this branch. That means no `-no_compat_check`, no OCLP root patches, no USB port-table rewrite, and no `revpatch` / `revcpu`.

## Differences from `main`

| Item | `main` (Tahoe 26) | This branch (Sequoia 15) | Why |
| --- | --- | --- | --- |
| SMBIOS | `MacPro7,1` | **`iMacPro1,1`** | Natively supported by 15; no compatibility bypass needed |
| `AirportItlwm` | Tahoe build, `MinKernel 25.0.0` | **2.4.0, kernel 24.0.0 – 24.99.99** | Loads on 15 only |
| `-ibtcompatbeta` | Required | **Still required** | `IntelBTPatcher` 2.5.0 declares a kernel ceiling; Lilu drops the plugin above it |
| `BlueToolFixup` | 2.7.2 | **2.7.2** | 2.6.9 patches the wrong `BlueTool` offsets |
| `-no_compat_check` | Required | **Not used** | — |
| `revpatch` / `revcpu` | `auto` / `1` | **Not used** | That is a Tahoe-only CPU-name workaround |
| USB | `USBPorts.kext`, custom 15-port map | **`USBInjectAll` + `XhciPortLimit`** | See [Known limitations](#known-limitations) |

## Hardware

| Component | Model |
| --- | --- |
| Motherboard | Maxsun MS-Terminator B760M D4 (AMI BIOS) |
| CPU | Intel Core i5-13490F (Raptor Lake, 6P + 4E, no iGPU) |
| GPU | AMD Radeon RX 570 4 GB (Polaris 10) |
| Memory | 32 GB DDR4-2666 |
| Wi-Fi / Bluetooth | Intel AX210 (Wi-Fi 6E + BT 5.3) |
| Ethernet | Realtek RTL8125 2.5 GbE |
| Audio | Realtek ALC897 |
| Boot disk | 120 GB SSD, 209.7 MB ESP |

> The ESP lives on the 120 GB SSD, not on the system disk. The machine will not boot without that SSD installed.

## Install

1. **Generate your own identity.** `config.plist` here is sanitized — `SystemSerialNumber`, `MLB`, `SystemUUID` and `ROM` are all blank. Use [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) with `iMacPro1,1` and fill in `PlatformInfo > Generic`. A blank serial boots, but iCloud, the App Store and iMessage will refuse to work.
2. Copy `EFI/` to the ESP (FAT32, 200 MB is enough).
3. Boot into OpenCore and **reset NVRAM twice**. The `boot-args` flags only take effect once NVRAM has been rewritten.
4. BIOS: disable Secure Boot, disable Resizable BAR, enable Above 4G Decoding, SATA mode AHCI.

This EFI boots the **macOS Sequoia 15.7.9 (24G830)** installer.

## What works

| Feature | Status | Notes |
| --- | --- | --- |
| Wi-Fi (AX210) | Verified | `AirportItlwm` 2.4.0, native menu bar, no HeliPort needed |
| Bluetooth (AX210) | Verified | Needs `-ibtcompatbeta` + `BlueToolFixup` 2.7.2 |
| USB keyboard / mouse | Verified | via `USBInjectAll` |
| Graphics acceleration | Expected | Same GPU path as `main`; WhateverGreen 1.7.1 |
| Ethernet, audio | Not separately re-verified | Same kexts and `alcid=11` as `main` |
| AirDrop / Continuity / Sidecar | Not available | Requires a genuine Apple card — the hard ceiling for Intel |

`ocvalidate` (OpenCore 1.0.8) reports **no issues**.

## Kext versions

| Kext | Version |
| --- | --- |
| Lilu | 1.7.2 |
| VirtualSMC / SMCProcessor / SMCSuperIO | 1.3.8 |
| WhateverGreen | 1.7.1 |
| AppleALC | 1.9.8 |
| AirportItlwm | 2.4.0 (kernel 24.x only) |
| IntelBluetoothFirmware / IntelBTPatcher | 2.5.0 |
| BlueToolFixup | 2.7.2 |
| CPUFriend / CPUFriendDataProvider | 1.2.7 / 1.0.1 |
| RestrictEvents | 1.1.5 |
| LucyRTL8125Ethernet | 1.2.2 |
| USBInjectAll | 0.8.0 |

## Known limitations

- **`USBInjectAll` is a stopgap.** 0.8.0 only officially supports up to macOS 13; it works here because the keyboard and mouse happen to fall inside the first 15 ports. If a port stops appearing, or sleep/wake misbehaves, build a proper `USBPorts.kext` — `main` already has one.
- **`IntelBluetoothFirmware` 2.5.0 is not an official release.** The last tag is v2.4.0; the upstream commit that raises the kernel ceiling lives only on `master`, so "install the latest release" would actually move you backwards.
- **`CPUFriendDataProvider` is enabled and carries a frequency vector from the original build.** If power management looks off, disable both `CPUFriend` and `CPUFriendDataProvider` and let native XCPM take over — that is what `main` does.
- The `可选WiFi/` directory is **Tahoe-only** (macOS 26 configs and `AirportItlwm` build). Do not use it on this branch.

## Credits

Same projects as `main` — Acidanthera (OpenCore, Lilu, VirtualSMC, WhateverGreen, AppleALC, BrcmPatchRAM), OpenIntelWireless (AirportItlwm, IntelBluetoothFirmware), [laobamac/itlwm](https://github.com/laobamac/itlwm) for the Sequoia-compatible AirportItlwm build, and corpnewt for GenSMBIOS. See the [`main` README](../../blob/main/README.md) for the full list and the Tahoe-specific deep dives.
