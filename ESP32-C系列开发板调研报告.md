## ESP32-C系列开发板调研报告

## 简介：

​	ESP32-C系列一共拥有ESP32-C6、ESP32-C61、ESP32-C5、ESP32-C3、ESP32-C2五款主流芯片。其中C6、C61、C5都支持WIFI6协议，且各具特点。C6为低功耗WIFI6 SOC，具备增强连接的特点；C61是高性价比的WIFI6芯片；支持2.4 & 5G赫兹的双核WIFI6 MCU；C3适用于安全的IOT应用；C2是高性价比芯片，适用于一些简单的IOT操作。

## 支持的操作系统：

​	ESP32-C2：FreeRTOS、Zrphyr

​	ESP32-C3：FreeRTOS、Zephyr

​	ESP32-C6：FreeRTOS、Zephyr

​	ESP32-C61：FreeRTOS、Zephyr

​	ESP32-C5：FreeRTOS、Zephyr

## 连接：

​	一款 **ESP32-P4** 开发板

​	**USB 数据线** （A 转 Micro-B）

​	电脑（Windows、Linux 或 macOS）

## 开发：

- 设置 **工具链**，用于编译代码；
- **编译构建工具** —— CMake 和 Ninja 编译构建工具，用于编译应用程序；
- 获取 **ESP-IDF** 软件开发框架。该框架已经基本包含 ESP32使用的 API（软件库和源代码）和运行 **工具链** 的脚本；

### 安装：

​	建议使用集成环境安装ESP-IDF，如果想要手动安装请参考 “ESP32-S系列开发板相关信息调研中相关操作”

### Eclipse Plugin：

​	乐鑫-IDE 是一个基于 Eclipse CDT 的集成开发环境 （IDE），用于使用 [ESP-IDF](https://github.com/espressif/esp-idf) 开发物联网应用程序。它是专为 ESP-IDF 构建的独立定制 IDE。乐鑫-IDE 附带了 IDF Eclipse 插件、基本的 Eclipse CDT 插件以及 Eclipse 平台的其他第三方插件，以支持构建 ESP-IDF 应用程序。

​	**乐鑫-IDE 3.0 及更高版本支持 ESP-IDF 5.x 及更高版本。对于 ESP-IDF 4.x 版，请使用 Espressif-IDE [2.12.1](https://github.com/espressif/idf-eclipse-plugin/releases/tag/v2.12.1) 版**

#### 如何在本地构建：

​	安装先决条件：Java 17+ 和 Maven。

​	运行以下命令进行克隆和构建。

```
git clone https://github.com/espressif/idf-eclipse-plugin.git
cd idf-eclipse-plugin
mvn clean verify -Djarsigner.skip=true
```

​	这将生成 p2 更新站点项目：

- 名字：`com.espressif.idf.update-*`
- 位置：`releng/com.espressif.idf.update/target`

#### 从 Eclipse Marketplace 安装 IDF Eclipse 插件

1. 打开 Eclipse 并转到 Eclipse Marketplace >帮助...。
2. 在搜索框中，输入 **ESP-IDF Eclipse Plugin** 以找到该插件。
3. 单击**安装**并按照屏幕上的说明完成安装。
4. 安装后，重新启动 Eclipse 以激活插件。

#### 从本地存档安装 IDF Eclipse 插件

1. 从 [此处]（https://github.com/espressif/idf-eclipse-plugin/releases） 下载 IDF Eclipse 插件的最新更新站点存档。
2. 在 Eclipse 中，转至帮助>安装新软件。
3. 单击“添加...”按钮。
4. 在“**添加存储库**”对话框中，选择**“存档”**，然后选择文件com.espressif.idf.update-vxxxxxxx.zip。
5. 单击**添加。**
6. 从列表中选择**乐鑫 IDF** 并继续安装。
7. 安装完成后重新启动 Eclipse。

#### 如何升级我现有的 IDF Eclipse 插件	

1. 转到窗口>首选项>安装/更新>可用软件站点。
2. 单击添加。
3. 输入新存储库的 URL：https://dl.espressif.com/dl/idf-eclipse-plugin/updates/latest/。
4. 单击确定。

**如果已经使用更新站点 URL 安装了 IDF Eclipse 插件，则可以通过以下步骤升级到最新版本：**

1. 转到帮助>检查更新。
2. 如果找到更新，请选择 Espressif IDF Plugins for Eclipse 并取消选择所有其他项目。
3. 单击下一步继续安装。

### VSCode Extension

​	下载VSCode

​	下载 Visual Studio Code (VS Code) 后，需要在 VS Code 中安装 ESP-IDF 扩展。前往菜单栏：查看->命令面板；输入：ESP-IDF：配置ESP-IDF拓展，选中该命令以启动设置向导。用户界面将显示加载通知，随后出现设置向导。

​	***ESP-IDF 5.0 以下的版本不支持在配置路径中添加空格。***

​	点击 `Express` 并选择下载服务器（推荐使用乐鑫）

​	选择要下载的 ESP-IDF 版本，或使用Find ESP-IDF in your system选项搜索现有的 ESP-IDF 目录。选择保存 ESP-IDF 工具的位置 (`IDF_TOOLS_PATH`)，默认情况下在 Windows 系统中为 `%USERPROFILE%\.espressif`，在macOS/Linux 系统中为 `$HOME\.espressif`

​	***确保 `IDF_TOOLS_PATH` 中没有空格，避免出现构建问题。此外，要注意 `IDF_TOOLS_PATH` 与 `IDF_PATH` 不能在相同目录下。***

​	***macOS/Linux 用户需要选择用于创建 ESP-IDF Python 虚拟环境的 Python 可执行文件。***

​	点击 `Install` 开始下载和安装 ESP-IDF 和 ESP-IDF 工具。

​	用户界面中将显示设置的进度：ESP-IDF 下载进度；ESP-IDF 工具下载和安装进度；Python 虚拟环境创建进度及 ESP-IDF Python 依赖包安装进度；

​	若一切安装正确，页面将显示所有设置已完成配置的消息。

### 新建项目：

​	在 Visual Studio Code 中：查看->命令面板；输入ESP-IDF：新建项目，选择命令以启动新项目向导窗口：

- 输入项目名称
- 选择保存新项目的位置
- 选择当前使用的乐鑫开发板名称
- 选择设备的串口（下拉菜单中会显示当前连接的串行设备列表）

​	***如果不确定串口名称，可以查看 [创建串口连接](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32/get-started/establish-serial-connection.html)。***

​	***请查看 [根据目标芯片配置 OpenOCD](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32/api-guides/jtag-debugging/tips-and-quirks.html#jtag-debugging-tip-openocd-configure-target)，为你的硬件选择合适的 OpenOCD 配置文件。***

- 也可以将 ESP-IDF 组件目录 `component-dir` 导入到新项目中。该组件目录将被复制到新项目的 `components` 子目录中 (`<project-dir>/components/component-dir`)。
- 点击 `Choose Template` 按钮。
- 如果想使用例程模板，请在下拉菜单中选择 ESP-IDF。

​	***如果想创建一个空白项目，请选择 `sample_project` 或 `template-app`。***

- 选择想要使用的模板并点击 `Create Project Using Template <template-name>` 按钮，其中 `<template-name>` 是所选模板的名称。
- 成功创建项目后，将弹出一个通知窗口，询问是否打开新创建的项目。

## ESP-AT：

​	ESP-AT 是乐鑫开发的可直接用于量产的物联网应用固件，旨在降低客户开发成本，快速形成产品。通过 ESP-AT 命令，您可以快速加入无线网络、连接云平台、实现数据通信以及远程控制等功能，真正的通过无线通讯实现万物互联。

​	ESP-AT 是基于 ESP-IDF 实现的软件工程。它使 ESP32-C3 模组作为从机，MCU 作为主机。MCU 发送 AT 命令给 ESP32-C3 模组，控制 ESP32-C3 模组执行不同的操作，并接收 ESP32-C3 模组返回的 AT 响应。ESP-AT 提供了大量功能不同的 AT 命令，如 Wi-Fi 命令、TCP/IP 命令、Bluetooth LE 命令、MQTT 命令、HTTP 命令等。

​	具体的编译连接与命令集请参考官方文档，在此不过多叙述。
