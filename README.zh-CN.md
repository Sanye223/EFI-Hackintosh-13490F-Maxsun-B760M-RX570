<div align="center">

# Maxsun B760M 13490F D4 Hackintosh — Sequoia 15

**铭瑄终结者 B760M D4 · i5-13490F · RX 570 · AX210 的 macOS Sequoia 15 OpenCore EFI**

![macOS](https://img.shields.io/badge/macOS-Sequoia%2015.7.9-000000?logo=apple&logoColor=white) ![OpenCore](https://img.shields.io/badge/OpenCore-1.0.8-9cf) ![CPU](https://img.shields.io/badge/CPU-i5--13490F-0071c5?logo=intel&logoColor=white) ![GPU](https://img.shields.io/badge/GPU-RX_570_4GB-ed1c24?logo=amd&logoColor=white) ![Wi-Fi](https://img.shields.io/badge/Wi--Fi%20%2B%20BT-AX210-0071c5?logo=intel&logoColor=white) ![状态](https://img.shields.io/badge/状态-已验证-brightgreen)

[English](README.md) &nbsp;·&nbsp; **简体中文**

</div>

---

> **当前在 `sequoia-15` 分支。** [`main`](../../tree/main) 分支是**同一台机器**的 **macOS Tahoe 26** 配置。两者不能混用：机型、Wi-Fi 驱动的内核版本闸门、USB 方案都不一样。装哪个系统，就用哪个分支。

## 简介

这是同一套硬件（见 `main`）的 **macOS Sequoia 15** 配置。它不是把 Tahoe 那份改下来的，而是从这台机器当年跑 Sonoma 的 EFI 重新升级到 15.7.9。

原则只有一条：**只改 macOS 15 真正弄坏的东西**，凡是「因为要上 Tahoe 才存在」的补丁，一律不带到这个分支。所以这里没有 `-no_compat_check`、没有 OCLP 根补丁、没有重写 USB 端口表、也没有 `revpatch` / `revcpu`。

## 与 main 的差异

| 项目 | `main`（Tahoe 26） | 本分支（Sequoia 15） | 原因 |
| --- | --- | --- | --- |
| 机型 | `MacPro7,1` | **`iMacPro1,1`** | 15 原生支持，不需要兼容绕过 |
| `AirportItlwm` | Tahoe 版，`MinKernel 25.0.0` | **2.4.0，内核 24.0.0 – 24.99.99** | 只在 15 上加载 |
| `-ibtcompatbeta` | 需要 | **仍然需要** | `IntelBTPatcher` 2.5.0 声明了内核上限，超限时 Lilu 直接丢弃插件 |
| `BlueToolFixup` | 2.7.2 | **2.7.2** | 2.6.9 的 `BlueTool` 补丁偏移对不上 |
| `-no_compat_check` | 需要 | **不用** | — |
| `revpatch` / `revcpu` | `auto` / `1` | **不用** | 那是 Tahoe 上修 CPU 名称的补丁 |
| USB | `USBPorts.kext` 自定义 15 口 | **`USBInjectAll` + `XhciPortLimit`** | 见[已知限制](#已知限制) |

## 硬件

| 部件 | 型号 |
| --- | --- |
| 主板 | 铭瑄 MS-Terminator B760M D4（AMI BIOS） |
| CPU | Intel Core i5-13490F（Raptor Lake，6P + 4E，无核显） |
| 显卡 | AMD Radeon RX 570 4 GB（Polaris 10） |
| 内存 | 32 GB DDR4-2666 |
| 无线 | Intel AX210（Wi-Fi 6E + 蓝牙 5.3） |
| 有线 | Realtek RTL8125 2.5 GbE |
| 声卡 | Realtek ALC897 |
| 启动盘 | 120 GB SSD，209.7 MB ESP |

> ESP 在 120 GB 那块 SSD 上，不在系统盘上。把那块盘拔了机器就起不来。

## 安装

1. **生成你自己的身份**。本分支的 `config.plist` 已脱敏：`SystemSerialNumber`、`MLB`、`SystemUUID`、`ROM` 全为空。用 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) 按 `iMacPro1,1` 生成后填入 `PlatformInfo > Generic`。序列号留空能开机，但 iCloud、App Store、iMessage 会不正常。
2. 把 `EFI/` 复制到 ESP（FAT32，200 MB 足够）。
3. 开机进 OpenCore，**连按两次 Reset NVRAM**——`boot-args` 里的 flag 要等 NVRAM 被重写才生效。
4. BIOS：关 Secure Boot、关 Resizable BAR、开 Above 4G Decoding、SATA 设为 AHCI。

这套 EFI 引导的是 **macOS Sequoia 15.7.9（24G830）** 安装器。

## 已验证

| 功能 | 状态 | 说明 |
| --- | --- | --- |
| Wi-Fi（AX210） | 已验证 | `AirportItlwm` 2.4.0，系统原生菜单栏，不需要 HeliPort |
| 蓝牙（AX210） | 已验证 | 需要 `-ibtcompatbeta` + `BlueToolFixup` 2.7.2 |
| USB 键鼠 | 已验证 | 走 `USBInjectAll` |
| 显卡加速 | 预期可用 | 与 `main` 同一条 GPU 路径，WhateverGreen 1.7.1 |
| 有线、音频 | 未单独复验 | kext 与 `alcid=11` 与 `main` 相同 |
| AirDrop / 通用剪贴板 / Sidecar | 不可用 | 需要原厂网卡，Intel 网卡的硬上限 |

`ocvalidate`（OpenCore 1.0.8）校验结果：**无问题**。

## 驱动版本

| Kext | 版本 |
| --- | --- |
| Lilu | 1.7.2 |
| VirtualSMC / SMCProcessor / SMCSuperIO | 1.3.8 |
| WhateverGreen | 1.7.1 |
| AppleALC | 1.9.8 |
| AirportItlwm | 2.4.0（仅内核 24.x） |
| IntelBluetoothFirmware / IntelBTPatcher | 2.5.0 |
| BlueToolFixup | 2.7.2 |
| CPUFriend / CPUFriendDataProvider | 1.2.7 / 1.0.1 |
| RestrictEvents | 1.1.5 |
| LucyRTL8125Ethernet | 1.2.2 |
| USBInjectAll | 0.8.0 |

## 已知限制

- **`USBInjectAll` 是临时方案**。0.8.0 官方只支持到 macOS 13，这里能用是因为键鼠刚好落在前 15 个端口内。一旦某个口不认、或睡眠唤醒异常，应当制作 `USBPorts.kext`——`main` 里已经有一份。
- **`IntelBluetoothFirmware` 2.5.0 不是官方发布版本**。最后一个 tag 是 v2.4.0；把内核上限抬高的提交只在 `master` 上，所以「装最新 release」反而会让你退回旧版。
- **`CPUFriendDataProvider` 是启用的**，里面的频率向量来自最初的配置。如果发现电源管理表现不对，把 `CPUFriend` 和 `CPUFriendDataProvider` 一起停用，让原生 XCPM 接管——`main` 就是这么做的。
- `可选WiFi/` 目录**只适用于 Tahoe**（macOS 26 的配置和 `AirportItlwm` 构建），不要用在本分支。

## 致谢

项目与 `main` 相同——Acidanthera（OpenCore、Lilu、VirtualSMC、WhateverGreen、AppleALC、BrcmPatchRAM）、OpenIntelWireless（AirportItlwm、IntelBluetoothFirmware）、[laobamac/itlwm](https://github.com/laobamac/itlwm)（本分支可用的 Sequoia 版 AirportItlwm 来自这里），以及 corpnewt 的 GenSMBIOS。完整清单和 Tahoe 相关的深挖内容见 [`main` 的 README](../../blob/main/README.zh-CN.md)。
