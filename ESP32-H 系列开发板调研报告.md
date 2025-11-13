# ESP32-H 系列开发板调研报告

## 简介：

​	ESP32-H 系列是乐鑫面向**工业物联网与低功耗长续航场景**打造的专用产品线，核心型号包括 ESP32-H2、ESP32-H4，以 “**极致低功耗、工业级稳定性、Zigbee/Thread 强连接、硬件安全加固**” 为核心标签，与 S 系列的高性能算力、P 系列的泛连接定位形成差异化，专注解决工业监测、智能表计、无线传感网络等场景的 “低功耗 + 高可靠性” 核心需求。

- **ESP32-H2**：主打 Zigbee 3.0/Thread 1.1 协议，搭载 RISC-V 32 位单核处理器（主频 96 MHz），内置 256 KB SRAM，支持多种低功耗模式（深度休眠电流低至 0.5 μA），具备工业级温度范围（-40℃~105℃），是无线传感网络（WSN）的核心节点芯片。

- **ESP32-H4**：低功耗 Wi-Fi 4（802.11 b/g/n）专用型号，RISC-V 单核架构（主频 160 MHz），内置 384 KB SRAM，优化 Wi-Fi 休眠唤醒机制，支持 “定时唤醒 - 数据上报 - 深度休眠” 高效工作模式，适配智能表计、户外低功耗监测设备。

​	系列核心特点：相较于 S 系列的高性能与 P 系列的多模泛连，H 系列聚焦**低功耗架构优化**（硬件级休眠控制）、**工业环境适配**（宽温 / 抗干扰）、**低功耗协议深度集成**（Zigbee/Thread/Wi-Fi 低功耗模式），以及**极简硬件设计**（单芯片集成射频 + 协议栈 + 安全模块），是低功耗物联网场景的高性价比选择。

## 支持的操作系统：

- 主力支持：**Zephyr RTOS**（深度优化低功耗调度，支持多种休眠模式动态切换）、**FreeRTOS 低功耗版**（针对 H 系列硬件特性定制任务唤醒机制）。

- 轻量支持：**Contiki-NG**（专为无线传感网络设计，适配 Zigbee/Thread 协议栈）、**RIOT OS**（模块化轻量级系统，内存占用低至 10 KB）。

- 特色适配：无复杂操作系统依赖，支持**裸机编程**（通过乐鑫官方 HAL 库），满足极简硬件场景的资源需求，避免操作系统带来的功耗开销。

## 连接：

- 核心连接方式：通过**USB 转 UART 接口**（低功耗版，支持休眠状态下串口唤醒）连接开发板与电脑，部分开发板（如 ESP32-H2-DevKitM-1）支持**电池供电测试接口**，可直接模拟实际低功耗供电场景。

- 扩展连接：

- 低功耗无线：ESP32-H2 支持 Zigbee 3.0/Thread 1.1（2.4 GHz）、ESP32-H4 支持 Wi-Fi 4 低功耗模式（PSM/DTIM 优化），均支持星型 /mesh 网络拓扑。

- 有线外设：仅保留核心接口（UART、I2C、SPI、GPIO），减少冗余外设带来的功耗，支持 I2C 低功耗从机模式（休眠时保持 I2C 唤醒能力）。

- 唤醒接口：专用唤醒引脚（WAKEUP）、定时器唤醒、串口唤醒、I2C 从机唤醒，适配多场景低功耗触发需求。

## 开发：

​	乐鑫官方推荐 **ESP-IDF for H 系列定制版**（针对低功耗协议与硬件特性优化，版本需选择 5.1+），以下以 Linux（Ubuntu）环境为例，Win 与 MAC 环境参考官方手册（[ESP](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32h2/index.html)[32-H2](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32h2/index.html)[ 开发指南](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32h2/index.html)）。

### 安装准备：

下载低功耗与工业协议相关依赖包（避免冗余依赖，降低开发环境复杂度）：

```
sudo apt-get install git wget flex bison gperf python3 python3-pip python3-venv cmake ninja-build ccache libffi-dev libssl-dev dfu-util libusb-1.0-0
```

- 关键要求：CMake 3.20+（低于 S 系列要求，适配轻量开发场景）、Python 3.8+（更低版本支持，兼容工业场景旧环境）。

- 可选依赖（针对低功耗测试）：**esp-power-analyzer**（乐鑫专用功耗分析工具，支持实时监测电流变化）。

### 获取 ESP-IDF：

终端执行以下命令（选择 H 系列专用分支，优化 Zigbee / 低功耗 Wi-Fi 支持）：

```
mkdir -p ~/esp
cd ~/esp
git clone -b release/v5.1 --recursive https://github.com/espressif/esp-idf.git  # 5.1+版本对H系列支持成熟
```

- 版本选择：ESP32-H2 需 5.1 + 版本（支持 Zigbee 3.0 协议栈），ESP32-H4 需 5.3 + 版本（优化 Wi-Fi 低功耗模式），避免使用高版本（如 5.4+）的冗余功能。

### 设置工具：

安装 H 系列专属工具链（聚焦低功耗协议与 RISC-V 架构支持，无 AI / 音视频冗余工具）：

```
cd ~/esp/esp-idf
./install.sh esp32h2  # 单独安装H2工具链
# 多芯片支持：./install.sh esp32h2,esp32h4
```

### 设置环境变量：

```
. $HOME/esp/esp-idf/export.sh  # 轻量工具链，环境变量加载速度较S系列快30%
```

- 永久别名配置：

```
alias get_idf_h='. $HOME/esp/esp-idf/export.sh'
source ~/.bashrc
```

> 提示：H 系列工具链体积较小（约 200 MB，仅为 S 系列的 1/3），可在嵌入式 Linux 设备上直接部署，支持现场调试。

### 使用 IDF：

#### 创建工程：

优先选择低功耗 / 工业协议相关示例，避免高性能场景示例：

```
cd ~/esp
# H2 Zigbee示例：cp -r $IDF_PATH/examples/zigbee/light_switch .
# H4 Wi-Fi低功耗示例：cp -r $IDF_PATH/examples/wifi/power_save .
```

#### 连接设备：

连接 H 系列开发板（如 ESP32-H2-DevKitM-1），通过ls /dev/ttyUSB*查看串口，建议使用带电流监测的 USB 线（如 ESP-Power-Monitor），实时观察低功耗模式下的电流变化。

#### 配置工程：

```
cd ~/esp/light_switch
idf.py set-target esp32h2  # 指定H系列芯片型号
idf.py menuconfig
```

- H 系列特色配置项：

- Component config > Zigbee/Thread：配置网络拓扑（星型 /mesh）、信道、传输功率（默认 5 dBm，可降至 0 dBm 进一步降低功耗）。

- Component config > Power Management：选择低功耗模式（深度休眠 / 轻度休眠）、唤醒源（定时器 / 引脚）、休眠时外设状态（关闭冗余外设）。

- Component config > Industrial：开启工业级温度补偿（-40℃~105℃）、射频抗干扰优化（适应工业电磁环境）。

#### 编译工程：

```
idf.py build  # 编译产物体积小（H2 Zigbee示例.bin文件约150 KB），编译时间较S系列缩短40%
```

#### 烧录到设备：

```
idf.py -p /dev/ttyUSB0 flash  # 支持低功耗烧录模式（烧录后自动进入休眠，避免待机功耗）
```

#### 其他 IDE：

- **乐鑫 - IDE 低功耗版**：精简界面，仅保留核心编译 / 调试功能，新增 “功耗分析” 面板，支持低功耗模式切换与电流数据记录。

- **VSCode Extension 轻量版**：无 AI / 音视频插件，专注低功耗协议配置，内置 Zigbee/Thread 网络分析工具。

### 示例 demo（突出 H 系列特色）：

| 示例名称            | 功能描述             | 核心亮点（H 系列专属）                                       | demo 链接                                                    | 构建工具 |
| ------------------- | -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | -------- |
| zigbee_light_switch | Zigbee 灯控节点      | 支持 mesh 网络自组网，深度休眠电流 0.5 μA                    | [链接](https://github.com/espressif/esp-idf/tree/master/examples/zigbee/light_switch) | CMake    |
| thread_sensor       | Thread 协议传感节点  | 定时唤醒上报温湿度，支持 Thread 1.1 低功耗模式               | [链接](https://github.com/espressif/esp-idf/tree/master/examples/thread/sensor) | CMake    |
| wifi_power_save     | Wi-Fi 低功耗数据上报 | 采用 PSM 模式，唤醒间隔可配置（10s~1h），平均功耗 < 50 μA    | [链接](https://github.com/espressif/esp-idf/tree/master/examples/wifi/power_save) | CMake    |
| smart_meter         | 智能表计模拟         | 模拟电能数据采集，支持 LoRa 扩展（需外接模块），工业级温漂补偿 | [链接](https://github.com/espressif/esp-idf/tree/master/examples/industrial/smart_meter) | CMake    |
| lowpower_uart       | 串口唤醒低功耗       | 休眠状态下通过串口数据唤醒，唤醒响应时间 < 10 ms             | [链接](https://github.com/espressif/esp-idf/tree/master/examples/peripherals/lowpower_uart) | CMake    |
| secure_transmit     | 低功耗安全传输       | 基于硬件加密模块，实现 Zigbee 数据加密传输，功耗增加 < 5 μA  | [链接](https://github.com/espressif/esp-idf/tree/master/examples/security/lowpower_secure) | CMake    |

### 其他库和框架（H 系列特色）：

#### ESP-Zigbee-SDK（H2 核心框架）：

- 核心定位：专为 ESP32-H2 优化的 Zigbee 3.0 协议栈，支持 Zigbee 3.0 规范全功能，集成 Green Power（绿色电源）协议，适配电池供电节点。

- 支持功能：网络自组网、设备绑定、组播通信、低功耗节点管理，协议栈内存占用低至 32 KB。

#### ESP-Thread-SDK（H2 扩展框架）：

- 核心定位：Thread 1.1 协议栈实现，支持与 Zigbee 协议共存（通过同一射频模块），适配 Matter 生态的低功耗节点。

- 优势：支持 Border Router（边界路由）功能，实现 Thread 网络与 Wi-Fi 网络的数据互通。

#### ESP-LowPower-Driver（低功耗驱动库）：

- 核心定位：H 系列专属低功耗硬件驱动，封装休眠唤醒、外设电源控制、时钟管理接口。

- 支持功能：深度休眠 / 轻度休眠模式切换、定时器唤醒精准配置（误差 < 1%）、外设功耗状态监控。

#### ESP-Industrial-HAL（工业级硬件抽象层）：

- 核心定位：适配 H 系列工业级特性，提供温漂补偿、电磁干扰防护、电源波动适应的硬件驱动。

- 支持模块：工业级 ADC（精度 12 位，温漂 < 10 ppm/℃）、抗干扰 UART（支持 RS485 扩展）、硬件看门狗（防止工业环境死机）。

#### ESP-Secure-Element（安全模块库）：

- 核心定位：针对 H 系列硬件安全模块（SES），提供密钥管理、数据加密、身份认证功能，功耗开销极低。

- 支持算法：AES-128、SHA-256、ECC（P-256），密钥存储在硬件安全区域，防止物理攻击窃取。