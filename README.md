# 🌿 Smart Watering System (FreeRTOS Version)

基于 STM32F103 和 FreeRTOS 的高实时性智能浇水系统。
本项目旨在将传统的“裸机（Super-loop）”架构升级为**多任务实时操作系统**架构，以解决多传感器并发读取延迟和人机交互卡顿问题。

## 📅 项目背景 (Background)
在 V1.0 版本（裸机开发）中，由于温湿度读取（DHT11时序）和 OLED 刷新需要占用大量 CPU 时间，导致按键响应迟钝，且难以扩展网络功能。
**V2.0 版本引入 FreeRTOS**，通过任务拆分和优先级抢占机制，实现了系统的解耦与实时响应。

## 🛠️ 技术栈 (Tech Stack)
- **MCU**: STM32F103C8T6
- **OS**: FreeRTOS V10.3.1 (CMSIS-V1 API)
- **开发工具**: STM32CubeMX, Keil MDK / STM32CubeIDE
- **版本管理**: Git
- **传感器/外设**:
  - DHT11 / AHT10 (温湿度)
  - 土壤湿度传感器 (ADC)
  - OLED 0.96寸 (I2C)
  - 继电器/水泵

## ⚙️ 系统架构 (Architecture)
本项目采用模块化任务设计，核心任务规划如下：

| 任务名称 (Task) | 优先级 | 职责描述 | 通信机制 |
| :--- | :--- | :--- | :--- |
| **Task_Sensor** | Normal | 周期性轮询环境数据，进行滤波处理 | `xQueueSend` (发送数据至队列) |
| **Task_Control** | High | 核心业务逻辑，根据阈值控制水泵 | `xQueueReceive` (从队列读数据) |
| **Task_Display** | Low | 负责 OLED 界面刷新，显示实时状态 | 互斥量 `Mutex` 保护 I2C 总线 |
| **Task_Input** | RealTime | 响应紧急按键（手动浇水/模式切换） | `Binary Semaphore` (中断同步) |

### 关键技术点 (Key Features)
1.  **架构迁移**：从 `while(1)` 轮询机制重构为基于优先级的抢占式调度。
2.  **数据安全**：使用 **Queue (队列)** 替代全局变量传输传感器数据，避免竞态条件。
3.  **资源保护**：使用 **Mutex (互斥量)** 保护 I2C 总线，防止 OLED 刷新与传感器读取冲突。
4.  **中断管理**：采用 "Defer Interrupt Processing"（中断延迟处理）机制，ISR 仅释放信号量，逻辑处理交由任务执行，保证中断快进快出。

## 🚀 开发进度 (Roadmap)
- [x] **阶段一：RTOS 基础移植**
    - [x] 完成 CubeMX 环境配置与 FreeRTOS 移植
    - [x] 实现 Sensor 与 Control 任务的队列通信
    - [x] 实现按键中断的信号量同步
- [ ] **阶段二：多传感器融合**
    - [ ] 接入光照传感器与 OLED 显示
    - [ ] 优化 I2C 总线互斥锁
- [ ] **阶段三：高级控制与网络**
    - [ ] 引入 PID 算法控制环境恒定
    - [ ] 接入 ESP8266 实现 MQTT 远程监控

## 📂 目录结构
```text
├── Core/
│   ├── Src/
│   │   ├── main.c       # 硬件初始化
│   │   ├── freertos.c   # 任务定义与调度逻辑 (核心代码)
│   │   └── ...
│   └── Inc/
├── Drivers/             # HAL库驱动
└── README.md# 🌿 Smart Watering System (FreeRTOS Version)

基于 STM32F103 和 FreeRTOS 的高实时性智能浇水系统。
本项目旨在将传统的“裸机（Super-loop）”架构升级为**多任务实时操作系统**架构，以解决多传感器并发读取延迟和人机交互卡顿问题。

## 📅 项目背景 (Background)
在 V1.0 版本（裸机开发）中，由于温湿度读取（DHT11时序）和 OLED 刷新需要占用大量 CPU 时间，导致按键响应迟钝，且难以扩展网络功能。
**V2.0 版本引入 FreeRTOS**，通过任务拆分和优先级抢占机制，实现了系统的解耦与实时响应。

## 🛠️ 技术栈 (Tech Stack)
- **MCU**: STM32F103C8T6
- **OS**: FreeRTOS V10.3.1 (CMSIS-V1 API)
- **开发工具**: STM32CubeMX, Keil MDK / STM32CubeIDE
- **版本管理**: Git
- **传感器/外设**:
  - DHT11 / AHT10 (温湿度)
  - 土壤湿度传感器 (ADC)
  - OLED 0.96寸 (I2C)
  - 继电器/水泵

## ⚙️ 系统架构 (Architecture)
本项目采用模块化任务设计，核心任务规划如下：

| 任务名称 (Task) | 优先级 | 职责描述 | 通信机制 |
| :--- | :--- | :--- | :--- |
| **Task_Sensor** | Normal | 周期性轮询环境数据，进行滤波处理 | `xQueueSend` (发送数据至队列) |
| **Task_Control** | High | 核心业务逻辑，根据阈值控制水泵 | `xQueueReceive` (从队列读数据) |
| **Task_Display** | Low | 负责 OLED 界面刷新，显示实时状态 | 互斥量 `Mutex` 保护 I2C 总线 |
| **Task_Input** | RealTime | 响应紧急按键（手动浇水/模式切换） | `Binary Semaphore` (中断同步) |

### 关键技术点 (Key Features)
1.  **架构迁移**：从 `while(1)` 轮询机制重构为基于优先级的抢占式调度。
2.  **数据安全**：使用 **Queue (队列)** 替代全局变量传输传感器数据，避免竞态条件。
3.  **资源保护**：使用 **Mutex (互斥量)** 保护 I2C 总线，防止 OLED 刷新与传感器读取冲突。
4.  **中断管理**：采用 "Defer Interrupt Processing"（中断延迟处理）机制，ISR 仅释放信号量，逻辑处理交由任务执行，保证中断快进快出。

## 🚀 开发进度 (Roadmap)
- [x] **阶段一：RTOS 基础移植**
    - [x] 完成 CubeMX 环境配置与 FreeRTOS 移植
    - [x] 实现 Sensor 与 Control 任务的队列通信
    - [x] 实现按键中断的信号量同步
- [ ] **阶段二：多传感器融合**
    - [ ] 接入光照传感器与 OLED 显示
    - [ ] 优化 I2C 总线互斥锁
- [ ] **阶段三：高级控制与网络**
    - [ ] 引入 PID 算法控制环境恒定
    - [ ] 接入 ESP8266 实现 MQTT 远程监控

## 📂 目录结构
```text
├── Core/
│   ├── Src/
│   │   ├── main.c       # 硬件初始化
│   │   ├── freertos.c   # 任务定义与调度逻辑 (核心代码)
│   │   └── ...
│   └── Inc/
├── Drivers/             # HAL库驱动
└── README.md