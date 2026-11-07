# VMware Fusion HID Passthrough Debugging

### VMware Fusion USB HID 直通故障排查｜Apple Silicon Mac 实测

**[中文](#问题与结论) · [English](#english-summary) · [完整实验报告](docs/HID-Debug-Report.md) · [《湾岸5DX》实战教程](docs/WMMT5DX-Guide.md)**

> **项目定位**：记录 VMware Fusion 在 Apple Silicon Mac 上直通 Xbox 兼容 USB HID 手柄时遇到的保护性拦截，并提供一项真实环境中成功的临时设备辅助连接方法。这里不是 VMware 漏洞披露，也不是针对所有版本的通用修复补丁。

## 问题与结论

当尝试把 Xbox 兼容手柄连接至 Windows ARM 虚拟机时，VMware Fusion 可能提示：

> **The host requires this device for input.**

在我们的环境中，手柄已经能在 macOS 中识别、Windows 也曾成功使用过它，但在拔出并重新连接后遇到了上述提示。即使为 USB 设备设置了 **Connect to this virtual machine**，问题仍然存在。

### 实测成功的临时解决方法

1. 确保 **MacBook 内置键盘和触控板正常使用**，不要把宿主机最后一个可靠的输入设备直通到虚拟机。
2. 额外连接一只可供 **macOS 宿主机**使用的 USB 鼠标。本例为 Rapoo 鼠标；**不要把这只鼠标直通给 Windows**。
3. 在 VMware Fusion 的 **USB 与蓝牙** 菜单中，选择 Xbox 兼容手柄（本例显示为 `2.4G XBOX 360 For Windows`），将其连接到 Windows 虚拟机。
4. 在本次测试中，手柄此时**成功连接到 Windows**。
5. 之后我们拔掉了临时接入的 Rapoo 鼠标，**Windows 中已建立的手柄连接仍保持正常，MacBook 内置输入设备也正常**。

**重要限制**：以上是本机、本设备组合的一次成功排障记录，并非已验证的跨版本通用操作流程；没有证明下一次热插拔可以自动重连。

### 关键对照观察

| 操作／状态 | 观察结果 |
|---|---|
| 只连接 Xbox 兼容手柄，尝试直通 Windows | ❌ 提示 `The host requires this device for input` |
| 额外接入 Rapoo 鼠标，保留给 macOS，再直通手柄 | ✅ 手柄直通成功 |
| 手柄已直通后，尝试把 Rapoo 鼠标也交给 Windows | ❌ 鼠标直通受到保护性限制 |
| 手柄已直通后，直接拔掉 Rapoo 鼠标 | ✅ 手柄保持连接，Mac 内置键盘及触控板正常 |

**我们能合理推测**：Fusion 的 HID 保护规则参与了设备连接准入；它对新连接的判断与对已建立连接的维持可能不同。**我们尚不能证明**它使用简单的“USB HID 数量”算法，或为什么内置键盘／触控板没有直接解除这次限制。

## 与 `.vmx` 参数的关系

我们的可运行配置中包含：

```ini
usb.generic.allowHID = "TRUE"
```

它可能使 HID 设备出现在可供直通的列表中，但 **我们没有独立完成开启／关闭该项的对照实验**，不将其断言为所有环境的必要条件。

Broadcom 官方文档还提到 `usb.generic.allowLastHID = "TRUE"`，用于允许将剩余的最后一个 HID 设备从宿主机移走。**这不是本项目推荐的绕行方法**：在我们的虚拟机里，加入该行后曾出现 `VMX file is corrupt`，删除新增行后恢复启动。该现象的确切原因仍未查明，不能因此断言该参数本身有缺陷。更重要的是，允许移走最后一个 HID 设备可能让宿主机暂时失去输入手段。

- [Broadcom 知识库：Connecting a second mouse or keyboard directly to a Fusion virtual machine（文章 344342）](https://knowledge.broadcom.com/external/article/344342)
- [完整 Debug 实验报告与证据边界](docs/HID-Debug-Report.md)

## 适用环境

- **宿主机**：Apple Silicon Mac（本机具体 macOS 和 Fusion 构建号尚待补充）
- **虚拟机**：VMware Fusion + Windows 11 ARM64
- **手柄**：GameSir Nova 2 Lite（盖世小鸡启明星2），USB 数据线连接，绿色 Xbox／XInput 模式
- **辅助设备**：Rapoo USB 鼠标，始终保留在 macOS；直通设备列表中曾显示 `Shenzhen Rapoo Rapoo Gaming Device`
- **验证工具**：Windows `joy.cpl`、JConfig 1.8、《湾岸5DX》游戏内 I/O 测试

我们尚未覆盖 Intel Mac、其他游戏手柄、其他 VMware Fusion 版本或 Windows x64 虚拟机。

## 实战案例：《湾岸5DX》

本项目起源于 **Apple Silicon Mac + VMware Fusion + Windows ARM + JConfig** 运行《湾岸5DX》的手柄排查。原教程完整保留，包括 USB 识别、Windows `joy.cpl`、XInput 按键绑定、JVS 油门和刹车处理。

**[查看完整《湾岸5DX》教程](docs/WMMT5DX-Guide.md)**

## 贡献与复现

欢迎报告 **Fusion 版本、macOS 版本、输入设备型号、是否热插拔、是否成功、错误提示**，帮助进一步验证。目前的结果属于单一环境中的观察，不应据此将其定性为可利用的安全漏洞。

## 许可与声明

原创文档与本项目拥有权利的原创截图遵循 [CC BY 4.0](LICENSE)；第三方软件、游戏、品牌及界面素材仍归原权利人。本文不是 Broadcom／VMware／Bandai Namco／GameSir 官方说明。

---

<a id="english-summary"></a>

## English summary

**Observed HID passthrough workaround on Apple Silicon VMware Fusion:** when an Xbox-compatible USB gamepad was rejected with `The host requires this device for input`, temporarily attaching an additional USB mouse **to the macOS host** allowed the gamepad to be passed through to a Windows 11 ARM VM. After the passthrough was established, the temporary mouse could be unplugged without disrupting the existing gamepad connection; the MacBook keyboard and trackpad remained functional.

This is a **single-environment empirical observation**, not proof of a VMware vulnerability, a universal fix, or any specific internal HID counting algorithm. Read the [full test report](docs/HID-Debug-Report.md) and [WMMT5DX game setup](docs/WMMT5DX-Guide.md).
