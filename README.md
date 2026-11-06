# 🏎️ WMMT5DX Apple Silicon Guide

### M 芯片 Mac 运行《湾岸5DX》及 Xbox 手柄适配指南

**中文教程 | English summary below**

本项目记录了一套经过实际测试的方案：在 **Apple Silicon Mac** 上，通过 **VMware Fusion + Windows 11 ARM + JConfig** 运行《湾岸5DX》，并使用 Xbox 兼容手柄进行控制。

特别适合遇到以下问题的玩家：

- VMware 虚拟机无法识别 Xbox 手柄
- Windows 11 ARM 中 `joy.cpl` 找不到控制器
- JConfig 识别到手柄，但游戏按键无效
- 游戏可以转向，但油门和刹车无法使用

## ✅ 已验证的测试环境

| 项目 | 测试配置 |
|---|---|
| 电脑 | Apple Silicon Mac |
| 虚拟机 | VMware Fusion |
| 操作系统 | Windows 11 ARM64 |
| 游戏 | Wangan Midnight Maximum Tune 5DX |
| 输入配置 | JConfig 1.8 |
| 手柄 | 盖世小鸡启明星 2（GameSir Nova 2 Lite） |
| 连接 | USB 有线，Xbox / XInput 模式 |

目前已经在游戏的 I/O 测试中验证手柄按钮、转向、油门和刹车能够响应。

*注意：模拟踏板是否保留完整线性输入、其他设备的兼容性仍有待进一步测试。*

## 🔧 核心解决步骤

### 1. 让 VMware 显示 Xbox 手柄

完全关闭 Windows 虚拟机及 VMware Fusion。

找到虚拟机的 `.vmwarevm` 文件，右键选择「显示包内容」，备份其中的 `.vmx` 配置文件。

在 `.vmx` 文件中添加：

```ini
usb.generic.allowHID = "TRUE"
```

保存后重新启动 VMware。

此设置可以让 VMware 显示此前可能被隐藏的 USB HID 输入设备。

**注意：不要误将 Mac 的键盘或鼠标直通给 Windows。**

### 2. 连接 Xbox 兼容手柄

以启明星 2 为例：

1. 使用 USB 数据线连接 Mac。
2. 将手柄切换至绿色 Xbox / XInput 模式。
3. 打开 VMware 的「USB 与蓝牙」设备列表。
4. 找到 `XBOX 360 For Windows` 对应设备。
5. 将它连接到 Windows 虚拟机。

在 Windows 中按 `Win + R`，输入：

```text
joy.cpl
```

确认控制器出现，并在「属性」中测试摇杆和扳机。

### 3. 设置 JConfig

打开 `JConfigWM5.exe`。

将 `Joystick Type` 选择为：

**XInput (Xbox Gamepad)**

推荐按键布局：

| 游戏操作 | 手柄按键 |
|---|---|
| START | A |
| TEST MODE | View |
| EXIT | Menu |
| MENU UP / DOWN | 十字键上 / 下 |
| ENTER SWITCH | X |
| PERSPECTIVE | Y |
| INTERRUPTION | B |
| GEAR SHIFT UP | RB |
| GEAR SHIFT DOWN | LB |
| 转向 | 左摇杆 |
| ACCEL（油门） | RT |
| BRAKE（刹车） | LT |

进入 `Settings`：

- `Fake Analog Steering wheel`：关闭
- `Fake Analog Pedals`：开启
- `Terminal Emulator`：保持开启
- `Terminal Mode`：保持关闭

注意：**RT 和 LT 还需要分别绑定到 ACCEL、BRAKE**，不能只开启 Fake Analog Pedals。

其他原本可以正常运行游戏的设置建议保持不变。

### 4. 游戏内验证

启动游戏，进入 `TEST MODE`。

依次进入：

`入出力テスト → スイッチテスト`

检查：

- 左摇杆左右移动，方向盘数值是否变化。
- RT 是否让油门数值变化。
- LT 是否让刹车数值变化。
- 手柄菜单、确认及换挡按钮是否正常。

通过后即可进一步测试实际驾驶。

## ⚠️ 已知问题

**Q：Mac 能识别手柄，VMware USB 列表却是空的？**

A：检查 `.vmx` 中的 `usb.generic.allowHID` 设置，并确保虚拟机完全重启。

**Q：蓝灯模式正常，绿灯 Xbox 模式却不被 Windows 识别？**

A：先检查 USB 设备是否真正直通给 Windows，而不是立即认定驱动不兼容。

**Q：切换 XInput 后，原来的 T 和 Esc 不工作？**

A：本次测试中出现过此问题。可以将 TEST MODE 和 EXIT 直接绑定到手柄。

**Q：方向盘正常，但油门、刹车没有反应？**

A：将 RT/LT 绑定到 ACCEL/BRAKE，并在 JVS Settings 中开启 `Fake Analog Pedals`。

## 🌍 English

This project documents a tested setup for running **Wangan Midnight Maximum Tune 5DX** with an Xbox-compatible controller on an Apple Silicon Mac using VMware Fusion, Windows 11 ARM64, and JConfig.

The key USB passthrough workaround is:

```ini
usb.generic.allowHID = "TRUE"
```

After enabling HID passthrough, connect the controller to the Windows VM, confirm it works in `joy.cpl`, and configure JConfig for XInput.

For our tested controller, steering works through the left analog stick. Accelerator and brake input required binding RT/LT and enabling `Fake Analog Pedals`.

A complete English guide will be added later.

## 📜 声明 / Disclaimer

本仓库仅分享原创技术教程和配置经验，不提供游戏本体、商业素材、未经授权的程序文件或游戏下载链接。

项目与 Bandai Namco、VMware、GameSir 均无官方关联。

欢迎通过 Issues 提出问题或分享其他设备的实测结果。

**Made for the community. 分享经验，帮助更多玩家。 ❤️**
