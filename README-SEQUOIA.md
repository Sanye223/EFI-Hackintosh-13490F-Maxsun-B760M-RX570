# macOS Sequoia 15 —— 同平台 EFI 分支

**分支：`sequoia-15`** · OpenCore **1.0.8** · macOS **15.7.9 (24G830)** · 已在实机验证

---

## 这个分支是什么

`main` 分支是为 **macOS Tahoe 26** 调的。Tahoe 之后，Wi-Fi 驱动、蓝牙固件、USB 端口表、SMBIOS 的写法都变了，那套配置**不能**直接用在 Sequoia 上；反过来，Sequoia 也不需要 Tahoe 的那些补丁（`-no_compat_check`、OCLP 根补丁等）。

这个分支保留同一台机器、同一代平台，只把系统换成 **macOS 15 Sequoia**，从本机 Sonoma 时期的 EFI 升级而来。

## 与 main 的差异

| 项目 | main（Tahoe 26） | 本分支（Sequoia 15） | 原因 |
| --- | --- | --- | --- |
| SMBIOS 机型 | `MacPro7,1` | **`iMacPro1,1`** | Sequoia 原生支持 iMacPro1,1，不需要 `-no_compat_check` |
| `AirportItlwm` | MinKernel 25.0.0（Tahoe 版） | **2.4.0，MinKernel 24.0.0 / MaxKernel 24.99.99** | 只在内核 24.x 加载 |
| `-ibtcompatbeta` | 需要 | **同样需要** | `IntelBTPatcher` 2.5.0 的插件声明带内核上限，Lilu 会拒绝加载 |
| `BlueToolFixup` | 2.7.2 | **2.7.2** | 2.6.9 的 `BlueTool` 补丁偏移对不上新系统 |
| `-no_compat_check` | 需要 | **不需要** | SMBIOS 本身就受支持 |
| `revpatch` / `revcpu` | `auto` / `1` | **不用** | 那是为了让 Tahoe 显示正确的 CPU 名称 |
| USB | `USBPorts.kext` 自定义 15 端口表 | **`USBInjectAll` + `XhciPortLimit`** | 见下方「已知限制」 |

## 目录约定

- `EFI/` —— 本分支的完整 EFI，直接放到 ESP 的根目录即可
- `docs/`、`tools/` —— 与 main 相同，未改动
- `可选WiFi/` —— **这是 main（Tahoe）用的**，其中的 `AirportItlwm.kext` 与 `config-*.plist` 面向 macOS 26，**不要**用在本分支

## 使用

1. **生成你自己的序列号**：本仓库的 `config.plist` 已脱敏（`SystemSerialNumber` / `MLB` / `SystemUUID` / `ROM` 均为空），用 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) 按 `iMacPro1,1` 生成后填入 `PlatformInfo > Generic`。序列号留空可以开机，但 iCloud / App Store / iMessage 会异常。
2. 把 `EFI/` 复制到 ESP（FAT32，200 MB 足够）。
3. 开机进 OpenCore，**连按两次 Reset NVRAM**（`boot-args` 里的 flag 需要重新写入 NVRAM 才生效）。
4. BIOS：关 Secure Boot、关 Resizable BAR、开 Above 4G Decoding、SATA = AHCI。

## 已验证

- Wi-Fi（Intel，`AirportItlwm` 2.4.0）：正常，走系统原生菜单栏
- 蓝牙（AX210，`-ibtcompatbeta` + `BlueToolFixup` 2.7.2）：正常，可配对
- 键盘 / USB：正常
- `ocvalidate`（OpenCore 1.0.8）：**No issues found**

## 已知限制

- **`USBInjectAll` 是临时方案**。0.8.0 只支持到 macOS 13，现在能用是因为键鼠正好落在前 15 个端口内。一旦出现某个 USB 口不认、或睡眠唤醒异常，应改用 `USBPorts.kext`（main 分支里那份端口表就是为此做的）。
- `IntelBluetoothFirmware` / `IntelBTPatcher` 用的是 **2.5.0**，这不是官方发布版本（最后一个 tag 是 v2.4.0）。上游把内核上限抬高的提交只存在于 `master`，所以「装最新 release」反而会更旧。
- AirDrop / 通用剪贴板 / Sidecar 不可用 —— 这是 Intel 网卡的天花板，与配置无关。

---

<sub>英文版说明见 `README.md`（面向 main 分支的 Tahoe 配置）。本文件仅描述 `sequoia-15` 分支。</sub>
