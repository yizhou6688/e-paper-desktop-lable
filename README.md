# 四色墨水屏桌面信息终端

基于 STM32G070CBT6 与 4.2 英寸 400 × 300 四色墨水屏，支持 NFC 配对、BLE 传图、日历与温湿度显示、七灯灯效及无线固件升级。墨水屏断电后仍可保留画面。

## 演示视频

[观看或下载 e-label-intro 介绍视频](refer_doc/e-label-intro.mp4)

## 主要功能

- **四色传图**：将图片转换为黑、白、黄、红四色，通过 BLE 传输并刷新。
- **NFC 配对**：近场读取设备信息和配对凭据；图片与控制数据通过 BLE 传送。
- **日历与环境信息**：显示日期、室内温湿度，支持本地跨日更新；天气和日程由客户端同步。
- **七灯灯效**：支持关闭、湿度呼吸和扩散呼吸，NFC 感应时临时切换同步呼吸。
- **电脑负载灯效**：Windows Pulse 通过 CH340 串口，将电脑负载映射为灯效。
- **BLE OTA**：通过 Bootloader 接收 `.epota` 升级包并更新固件。

## 功耗与电池

| 项目 | 参数 |
| --- | --- |
| 静态待机电流 | 10 mA |
| 刷新时最大电流 | 22 mA |
| 电池容量 | 1200 mAh |

按 10 mA 恒定待机电流估算，理论待机约 **120 小时（5 天）**。实际续航受刷新频率、灯效、无线通信、电池可用容量及供电损耗影响。上述功耗数据以电流表示，刷新峰值不代表平均电流。

## 硬件

| 模块 | 用途 |
| --- | --- |
| STM32G070CBT6 | 主控、屏幕刷新、RTC 与协议处理 |
| GDEM042F86 | 4.2 英寸 400 × 300 四色墨水屏 |
| ST25DV04K | NFC 识别与配对 |
| CH9140 | BLE 透明串口，连接 USART2 |
| AHT20 | 温湿度采样 |
| 74HC595、七颗 LED | 灯效输出 |
| CH340 | USB 串口，连接 USART1 |

硬件资料：[原理图](pcb/Board_Schematic.pdf) · [PCB 模型](pcb/PCB_model.pdf) · [嘉立创EDA工程](pcb/epaper-label.epro2) · [CubeMX 配置](refer_doc/墨水屏项目.ioc) · [外壳模型与预览](3dmodels/)。

## 仓库结构

| 目录 | 内容 |
| --- | --- |
| [e-paper/](e-paper) | STM32 固件与 Bootloader 子模块 |
| [yipengyishun/](yipengyishun) | Android 客户端子模块 |
| [pulse/](pulse) | Windows Pulse 客户端子模块 |
| [refer_doc/](refer_doc/) | 通信协议、开发文档、硬件参考与演示视频 |
| [pcb/](pcb/) | 原理图、PCB 工程与相关资料 |
| [3dmodels/](3dmodels/) | 外壳 STEP 模型、Blender 文件与预览图 |

在仓库根目录执行以下命令，拉取三个子模块的源码：

```bash
git submodule update --init --recursive
```

## 开发文档

| 开发方向 | 参考文档 |
| --- | --- |
| Android 应用 | [应用需求](refer_doc/ANDROID_APP_REQUIREMENTS.md)、[日历开发](refer_doc/ANDROID_CALENDAR_DEVELOPMENT.md) |
| NFC 配对与 BLE 传图 | [图片传输协议](refer_doc/IMAGE_TRANSFER_PROTOCOL.md) |
| 灯效与控制 | [灯效逻辑](refer_doc/LED_LOGIC.md)、[Android 灯效接口](refer_doc/ANDROID_LED_CONTROL.md) |
| 固件升级 | [BLE OTA 设计](refer_doc/BLE_OTA_DESIGN.md) |
| 硬件适配 | [硬件适配记录](refer_doc/HARDWARE_2026_09_11.md)、[参考原理图](refer_doc/SCH_Schematic3_2026-09-11.pdf) |

客户端的构建与使用说明见对应子模块。部分参考文档包含历史方案，接口与功能以所用版本的源码为准。

## 固件编译与烧录

初始化子模块并安装 Arm GNU Toolchain 与 Make 后，进入固件目录编译：

```bash
cd e-paper
make -j4 firmware
```

以下输出路径均相对于 `e-paper/`：

| 组件 | HEX 文件 | BIN 烧录地址 |
| --- | --- | --- |
| Bootloader | `build_bootloader/epaper_bootloader.hex` | `0x08000000` |
| Application | `build/epaper_project.hex` | `0x08006000` |

HEX 自带地址；首次烧录需写入 Bootloader 和 Application。Application 的 BIN 不可写入 `0x08000000`。升级前请核对 [OTA 布局与恢复说明](refer_doc/BLE_OTA_DESIGN.md)。

## 通信说明

- BLE 使用 `FFF0` 服务、`FFF1` Notify、`FFF2` Write，底层串口为 115200、8N1。
- 图片为 400 × 300、2bpp，共 30,000 字节；分包传输后需等待校验与屏幕刷新完成。
- NFC 配对凭据和 OTA 发布密钥应妥善保管；开发测试密钥不用于正式交付。
