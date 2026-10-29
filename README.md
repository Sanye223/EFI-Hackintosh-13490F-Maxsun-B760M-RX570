<div align="center">

# EFI-Hackintosh-i5-13490F-Maxsun-Terminator-B760M-D4-RX570

**OpenCore EFI for macOS Tahoe 26 on a Maxsun Terminator B760M D4 · i5-13490F · RX 570 · AX210 build**

![macOS](https://img.shields.io/badge/macOS-Tahoe%2026.7.1-000000?logo=apple&logoColor=white) ![OpenCore](https://img.shields.io/badge/OpenCore-1.0.8-9cf) ![CPU](https://img.shields.io/badge/CPU-i5--13490F-0071c5?logo=intel&logoColor=white) ![GPU](https://img.shields.io/badge/GPU-RX_570_4GB-ed1c24?logo=amd&logoColor=white) ![Wi-Fi](https://img.shields.io/badge/Wi--Fi%20%2B%20BT-AX210-0071c5?logo=intel&logoColor=white) ![Status](https://img.shields.io/badge/status-fully%20working-brightgreen)

**English** &nbsp;·&nbsp; [简体中文](README.zh-CN.md)

</div>

---

## Overview

A working OpenCore EFI for a budget Raptor Lake desktop with a Polaris GPU and an Intel AX210 wireless card. Every claim below was verified on the actual machine, not copied from a template.

What makes this repository worth reading is that three things which "just work" on older macOS releases quietly break on Tahoe, and none of them produces an error message:

| Symptom | Real cause |
| --- | --- |
| Some USB ports never appear | Tahoe ignores the port table entirely if it uses the old property names |
| Bluetooth is dead, macOS reports a bogus `BCM_4350C2 / v0` | The firmware upload step is blocked because its helper kext is gated off on macOS 26 |
| About This Mac says "Unknown" for the processor | The CPU family value in SMBIOS is `0`, so the name lookup misses |

Each one is documented with its root cause and the exact fix in [Deep dives](#deep-dives).

The `config.plist` in this repository is **sanitized**: serial number, MLB, UUID and ROM are blank. Generate your own with [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) before booting.

## Hardware

| Component | Model | Status under macOS |
| --- | --- | --- |
| Motherboard | Maxsun MS-Terminator B760M D4 (AMI BIOS) | — |
| CPU | Intel Core i5-13490F (Raptor Lake, 6P + 4E, 10 cores / 16 threads, **no iGPU**) | Native XCPM power management |
| GPU | AMD Radeon RX 570 4 GB (Polaris 10) | Native drivers, Metal works, no HEVC encoding |
| Memory | 32 GB DDR4-2666 | Recognized correctly |
| Wi-Fi / Bluetooth | Intel AX210 (Wi-Fi 6E + Bluetooth 5.3) | `AirportItlwm` for Wi-Fi, `-ibtcompatbeta` for Bluetooth |
| Ethernet | Realtek RTL8125 2.5 GbE | `LucyRTL8125Ethernet` |
| Audio | Realtek ALC897 | `AppleALC` with `alcid=11` |
| System disk | 2 TB, Tahoe inside a 161 GB APFS container | Native |
| Boot disk | 120 GB SSD, 209.7 MB ESP | — |

> The ESP lives on the 120 GB SSD; the 2 TB system disk has **no EFI partition**. The machine will not boot without that SSD installed — don't pull it out.

## What works

| Feature | Status | Notes |
| --- | --- | --- |
| macOS Tahoe 26.7.1 (25G241) install | Works | Clean install from USB, no workarounds |
| Graphical boot picker | Works | OpenCanopy + GoldenGate theme, official macOS 26 icon |
| GPU acceleration (RX 570) | Works | `WhateverGreen`; `agdpmod=pikera` is **not** needed (that flag is for Navi) |
| Ethernet 2.5 GbE | Works | `LucyRTL8125Ethernet` |
| Wi-Fi (AX210) | Works | Native System Settings UI, 5 GHz / 160 MHz / 802.11ac / 1732 Mbps |
| Bluetooth (AX210) | Works | Requires `-ibtcompatbeta` — see [Intel Bluetooth](#2-intel-bluetooth-ax210) |
| Audio (ALC897) | Works | `alcid=11`; USB sound card and HDMI audio work simultaneously |
| Processor name | Works | About This Mac shows `Intel Core i5-13490F` |
| USB ports | Works | Custom 15-port map; rear hub, front USB 3 and Bluetooth all live |
| AirDrop / Continuity / Sidecar | Not available | Requires a genuine Apple wireless card — the hard ceiling for Intel |
| HEVC hardware encoding | Not available | Polaris limitation; decoding works fine |

## Screenshots

All screenshots are from this machine. **Serial number, MAC addresses and network names have been redacted.**

| Wi-Fi — native UI, 5 GHz / 160 MHz / 802.11ac | About This Mac — Mac Pro (2019) · i5-13490F · RX 570 |
| --- | --- |
| ![Wi-Fi](docs/screenshots/01-wifi-connected.jpg) | ![About This Mac](docs/screenshots/02-about-this-mac.jpg) |

| Bluetooth — AirPods Pro + HECATE G1500 | Sound output — display audio + USB sound card |
| --- | --- |
| ![Bluetooth](docs/screenshots/03-bluetooth.jpg) | ![Sound](docs/screenshots/04-sound-output.jpg) |

## Getting started

### 1. BIOS settings (press Del at boot)

| Setting | Value |
| --- | --- |
| Secure Boot | Disabled (clear the Secure Boot keys while you're there) |
| CSM / Legacy Support | Disabled (pure UEFI) |
| Above 4G Decoding | **Enabled** |
| Re-Size BAR Support | **Disabled** |
| VT-d / IOMMU | Disabled |
| CFG Lock | Disabled if the option exists — optional, `AppleXcpmCfgLock` covers it |
| XHCI Hand-off / Legacy USB | Enabled |
| Serial / COM Port | Disabled |
| Boot Mode | UEFI only |

### 2. Build the installer

Use a USB drive of at least 16 GB, partitioned as GPT.

**From a Mac** — create the installer with `createinstallmedia`, then copy the whole `EFI` folder to the drive's EFI partition. The result must be `USB/EFI/BOOT/BOOTx64.efi`.

**From Windows** — format a FAT32 partition on the drive and copy `EFI` into it, then download a recovery image with the bundled script and place it at the root of the drive, next to `EFI`:

```bat
cd tools\macrecovery
python macrecovery.py -b Mac-E43C1C25D4880AD6 -m 00000000000000000 download
```

### 3. Install

1. Press **F11** at boot and pick `UEFI: <your USB drive>` to reach the graphical picker.
2. Choose `macOS Base System` / `Install macOS Tahoe`.
3. In Disk Utility, erase the target disk as **APFS / GUID**, then install normally. The machine reboots two or three times — each time pick **`macOS Installer`**.
4. When it's done, move the EFI folder to the system disk's ESP.

> **When copying EFI to the ESP**, delete the old `EFI\OC` first and check the byte size of `config.plist` afterwards. A truncated copy from a full ESP produces `OC: Failed to load configuration!`, which looks like a broken config but is really an incomplete file.

### 4. After installation

1. Generate a serial with GenSMBIOS and fill it into `PlatformInfo > Generic` — this repository ships them blank.
2. Change `revcpuname` under `NVRAM > Add > 4D1FDA02-…` to match your own CPU.
3. To debug, set `Misc > Debug > Target` to `19`. The log is written to the **root of the ESP**, not to `EFI/OC/`.

## Deep dives

### 1. USB port mapping on Tahoe

This is the part that cost the most time, and the one worth reading even if you skip everything else.

The board exposes 26 ports to the XHCI controller (`HS01`–`HS14`, `USR1`/`USR2`, `SS01`–`SS10`), but macOS 26 still caps a controller at **15 ports per boot**. On Tahoe the `XhciPortLimit` kernel quirk no longer works, so the limit cannot be lifted — you have to choose which 15 to keep.

Getting a port table accepted takes four things, and they all have to be true at once:

1. **Rebuild the kext from scratch.** Do not patch an existing `Info.plist`. Start from an empty bundle with a complete set of bundle keys.
2. **Use only the new property names** — `usb-port-number` and `usb-port-type`. The legacy `port` and `UsbConnector` keys must be **completely absent**. Tahoe does not ignore unknown legacy keys; it falls back to the old parsing path and discards the entire table without logging anything.
3. **Give the kext its own bundle identifier** (`com.wb.USBPorts`). Tahoe ships `AppleUSBHostMergeProperties.kext` inside `IOUSBHostFamily.kext/Contents/PlugIns/` with the identifier `com.apple.driver.AppleUSBHostMergeProperties`; reusing it makes the injector look like a duplicate of a system kext.
4. **Match the controller the way Apple's tooling does** — `IOProviderClass = AppleUSBHostController`, `IONameMatch = XHCI`, plus a `model` string — and clean the AppleDouble `._*` files that macOS leaves on FAT32, or the bundle will not parse.

The 15 ports kept in this build:

| Ports | Count | Purpose |
| --- | --- | --- |
| `HS03`–`HS11` | 9 | Rear USB 2.0; `HS07` currently hosts the keyboard, `HS09` the mouse |
| `HS14` | 1 | Bluetooth radio, declared internal (`usb-port-type = 255`) |
| `SS01`–`SS04` | 4 | Rear USB 3.0 |
| `SS09` | 1 | Front panel USB 3.0 |

That leaves `HS01`, `HS02` (motherboard ARGB controller), `HS12`, `HS13`, `USR1`, `USR2` and `SS05`–`SS10` unmapped. **The front panel USB 3 socket drops to USB 2 speeds if you remove `SS09`** — it was added back at the cost of `HS13`, which is why the port list looks the way it does.

To check whether your own table actually took effect:

```bash
ioreg -p IOService -r -n XHCI -d 20 -l -w 0 | grep -c '"ports"'
ioreg -p IOService -r -n XHCI -d 20 -l -w 0 | grep -oE '\+-o (HS|SS|USR)[0-9]+@'
```

If `"ports"` is missing, the merge never happened and the personality did not match — the port list is irrelevant at that point.

### 2. Intel Bluetooth (AX210)

The card is `0x8087:0x0032`, attached over USB. Wi-Fi works out of the box, Bluetooth does not, and the failure is easy to misread. macOS reports:

```
Chipset: BCM_4350C2    Firmware: v0    Address: NULL    State: Off
```

Those are placeholder values for "no firmware loaded" — not a Broadcom chip. `ioreg` shows why:

```
+-o IntelBluetoothFirmware  <class IntelBluetoothFirmware, !registered, !matched>
      "idVendor" = 32903 (0x8087)    "idProduct" = 50 (0x0032)
```

The firmware driver matched the device but `start()` failed, so the firmware was never uploaded.

The actual blocker sits one layer earlier. `IntelBTPatcher` carries a **maximum supported kernel version** in its plugin configuration, and in the 2.5.0 release that ceiling is macOS 15. Lilu refuses to load it on Tahoe, the firmware upload patch never gets applied, and the firmware driver fails as a consequence.

The fix is a flag the kext already exposes for exactly this case:

```
-ibtcompatbeta
```

Add it to `boot-args` together with `BlueToolFixup` **2.7.2** (2.6.9 predates Tahoe and patches the wrong `BlueTool` offsets). After a reboot the correct values appear: `THIRD_PARTY_DONGLE`, firmware `v13719`.

Related flags in the same kext, should you need them: `-ibtcompatdbg` for verbose logging and `-ibtcompatoff` to disable patching entirely.

### 3. Processor name showing "Unknown"

About This Mac does not read the CPU brand string directly. It builds a lookup key from the core count and the CPU family, then looks it up in

```
/System/Library/PrivateFrameworks/AppleSystemInfo.framework/Versions/A/Resources/Processors.loctable
```

which contains entries such as `"10-Core Intel Xeon W"`. If the key doesn't resolve, the field reads "Unknown". This build had `ProcessorType = 0` — family unknown — so the lookup never stood a chance.

The fix has two halves, and both are required:

1. **Make the lookup succeed.** Set `PlatformInfo > Generic > ProcessorType` **and** `PlatformInfo > SMBIOS > ProcessorType` to `3841` (`0x0F01`, Xeon W family). With 6P + 4E the machine reports 10 cores, which resolves to `10-Core Intel Xeon W`.
2. **Replace the resulting string.** `RestrictEvents` rewrites that localized string inside `AppleSystemInfo` at code-signature validation time. It needs `revcpu=1` — which is **off by default on Intel** — and takes the replacement text from NVRAM:

```
NVRAM > Add > 4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102 > revcpuname = Intel Core i5-13490F
```

`revpatch=auto` already implies `cpuname`, so nothing else is needed. Do not try to pass the name through `boot-args` — it contains spaces.

The `2.5 GHz` shown next to it is correct: that is the P-core base clock of the i5-13490F.

Skip step 1 and `revcpu=1` does nothing at all, because there is no Xeon string to replace. That is the trap.

### 4. Boot picker icon

The GoldenGate theme resolves icons by OS version, looking for `Apple<version>.icns`. The bundled set stops at `Apple15`, so Tahoe fell back to the generic `HardDrive.icns`.

The official artwork ships inside the installer — no need to hunt for it online:

```
/Volumes/Install macOS Tahoe/Install macOS Tahoe.app/Contents/Resources/ProductPageIcon.icns
```

Copy it verbatim (1.6 MB, 1024×1024, no re-encoding) to `Apple26.icns`, and additionally as `Apple16.icns` and `Apple.icns` as fallbacks in case the version-to-filename mapping differs between OpenCore builds. The theme is selected by `Misc > Boot > PickerVariant = Acidanthera\GoldenGate` with `PickerAttributes = 145`.

## Troubleshooting

| Symptom | What to check |
| --- | --- |
| Prohibited sign at boot | SMBIOS not in the OS whitelist; check `PlatformSupport.plist` inside the installer |
| `OC: Failed to load configuration!` | `config.plist` was copied incompletely — re-copy and compare file size |
| Boot hangs right after `[EB&#124;#LOG:EXITBS:START]` | Stale `-v` or a debug build mismatch; check `boot-args` |
| Some USB ports missing | Port table not merged — see [USB port mapping](#1-usb-port-mapping-on-tahoe) |
| Bluetooth off, `BCM_4350C2 / v0` | Missing `-ibtcompatbeta`, or `BlueToolFixup` too old |
| "Memory Modules Misconfigured" popup | `RestrictEvents` older than 1.1.6 on MacPro7,1; `revpatch=memtab` does **not** help this model |
| Processor shows "Unknown" | `ProcessorType` still `0` — see [Processor name](#3-processor-name-showing-unknown) |
| No log file despite `Target=3` | Release builds of OpenCore have logging compiled out; use a DEBUG `OpenCore.efi` |

## Pitfalls worth remembering

These are the ones that actually cost time on this build.

1. **`iMacPro1,1` is not accepted by Tahoe.** The installer's `PlatformSupport.plist` whitelists `MacBookPro16,x`, `MacPro7,1` and `iMac20,x` only. An `iMacPro1,1` config boots to a prohibited sign. Switch to `MacPro7,1` and generate a matching serial.
2. **Legacy USB keys poison the whole table.** Adding `usb-port-number` alongside `UsbConnector` looks like a safe compatibility move and silently fails. Remove the old keys completely.
3. **`USBInjectAll` is worse than useless on Tahoe.** It does not merely under-report ports — it tears down enumeration completely. With it enabled, `AppleUSBXHCIPCI` had **zero** child nodes.
4. **`XhciPortLimit` is dead on Tahoe.** The 15-port cap is real; plan your port selection accordingly.
5. **`revpatch=memtab` never applied to MacPro7,1.** That flag is for the memory tab on MacBookAir and MacBookPro10,x. What silences the warning on MacPro7,1 is `revblock` (on by default) inside RestrictEvents 1.1.6 or newer.
6. **`revcpu` defaults to off on Intel**, unlike on AMD. Setting `revcpuname` without `revcpu=1` changes nothing.
7. **Release OpenCore cannot write logs.** `Misc > Debug > Target` is inert without a DEBUG build, which sends people chasing a config problem that doesn't exist.
8. **`cp -R src dst` nests the EFI folder.** If the destination exists you end up with `EFI/EFI`. Use `cp -R src/. dst/`.
9. **FAT32 keeps AppleDouble files.** `._Info.plist` and friends survive `rm -rf` on the ESP. Run `dot_clean` and verify with `ls -a`; if the partition is beyond repair, recreate it with `newfs_msdos -v EFI -F 32`.
10. **Disk identifiers change between reboots.** The ESP may be `disk1s1` on one boot and `disk2s1` on the next. Resolve the mount point at runtime instead of hardcoding it.
11. **A backup made as root can be empty.** Copying into a user's home directory via `sudo` or `with administrator privileges` can fail silently and leave an empty folder behind. Always verify with a file count and an `md5`.
12. **`agdpmod=pikera` is for Navi only.** On Polaris it is unnecessary and can cause a black screen.

## Repository layout

```
EFI/
  BOOT/BOOTx64.efi
  OC/
    ACPI/          SSDT-AWAC, SSDT-EC-USBX-DESKTOP, SSDT-PLUG-ALT
    Drivers/       HfsPlus, OpenCanopy, OpenRuntime, ResetNvramEntry
    Kexts/         18 kexts (see below)
    Resources/     OpenCanopy theme assets, including Apple26.icns
    Tools/         OpenShell and friends
    config.plist   Sanitized — generate your own serial
docs/screenshots/  Screenshots used in this README
tools/macrecovery/ Recovery image downloader for Windows users
可选WiFi/          Alternative Wi-Fi setups from an earlier iteration
                   (A: native AirportItlwm, B: itlwm + HeliPort)
```

## Kext list

| Kext | Version | Notes |
| --- | --- | --- |
| Lilu | 1.7.2 | Patching framework, required by most others |
| VirtualSMC | 1.3.8 | SMC emulation |
| SMCProcessor | 1.3.8 | CPU temperature sensors |
| SMCSuperIO | 1.3.8 | Fan and voltage sensors |
| SMCRadeonSensors | 2.3.0 | GPU sensors |
| WhateverGreen | 1.7.1 | GPU fixes |
| AppleALC | 1.9.4 | Audio, `alcid=11` |
| AppleMCEReporterDisabler | — | Suppresses MCE panics on non-Xeon SMBIOS |
| RestrictEvents | 1.1.6 | Memory/slot notification blocking and CPU name patching |
| Innie | 1.3.1 | Makes PCIe devices appear internal |
| LucyRTL8125Ethernet | 1.2.2 | 2.5 GbE |
| USBPorts | 1.0 | Custom 15-port map, built for this repository |
| AirportItlwm | 2.4.0 | Intel Wi-Fi, Tahoe build |
| IntelBluetoothFirmware | 2.5.0 | Uploads AX210 firmware |
| IntelBTPatcher | 2.5.0 | Firmware upload patch — needs `-ibtcompatbeta` on Tahoe |
| BlueToolFixup | 2.7.2 | Fixes `BlueTool` offsets; 2.6.9 is too old for Tahoe |
| CPUFriend | 1.2.7 | **Disabled** — power data was generated for a 12490F, not this CPU |
| CPUFriendDataProvider | 1.0.1 | **Disabled** — same reason |

`CPUFriend` is deliberately disabled: the frequency vector shipped with it was generated for an i5-12490F, and applying it to a 13490F would be worse than leaving native XCPM in charge. `SSDT-PLUG-ALT` handles the CPU power management declarations instead.

## Changelog

**2026-10-29 — Tahoe migration**
- OpenCore 1.0.8, SMBIOS moved from `iMacPro1,1` to `MacPro7,1` (Tahoe whitelist)
- Custom `USBPorts.kext` replacing `USBInjectAll`: 15 ports, Tahoe-only property names, independent bundle identifier
- Bluetooth fixed with `-ibtcompatbeta` and `BlueToolFixup` 2.7.2
- Processor name fixed via `ProcessorType = 0x0F01`, `revcpu=1` and NVRAM `revcpuname`
- Boot picker switched to the official macOS 26 icon
- `RestrictEvents` 1.1.5 → 1.1.6, `AirportItlwm` replaced with the Tahoe build
- README rewritten; `config.plist` sanitized

**Earlier — Sequoia 15.7.1** — previous working configuration, `iMacPro1,1` with `USBInjectAll`.

## Credits

- [Acidanthera](https://github.com/acidanthera) — OpenCore, Lilu, VirtualSMC, WhateverGreen, AppleALC, RestrictEvents, BrcmPatchRAM
- [OpenIntelWireless](https://github.com/OpenIntelWireless) — AirportItlwm, IntelBluetoothFirmware
- [laobamac/itlwm](https://github.com/laobamac/itlwm) — the AirportItlwm build for macOS 26 used here (2.4.0-alpha, commit `738d4c6`); this fork is where Tahoe support currently lives
- [corpnewt](https://github.com/corpnewt) — GenSMBIOS, USBMap, macrecovery
- The InsanelyMac and r/hackintosh communities for their write-ups on Tahoe USB behaviour

## Disclaimer

This configuration is published for reference on hardware identical or very close to the list above. Applying it to different hardware can prevent your machine from booting. Back up your existing EFI before replacing anything, and keep a working USB installer within reach. Use at your own risk.

---

### Not affiliated with Apple

This project has **no connection to Apple Inc.** It is not authorized, endorsed, sponsored or supported by Apple in any way. macOS is a trademark of Apple Inc., and every other trademark or product name mentioned here belongs to its respective owner.

What this repository contains is an unofficial, community-built configuration for running macOS on hardware Apple never sold. It exists for educational and reference purposes only, and comes with no warranty of any kind. If you need a supported Mac, buy one from Apple.
