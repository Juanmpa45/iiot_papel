# DroneCAN: A Lightweight Protocol for Highly Reliable Intra-Vehicle Communication in Aerospace and Robotic Applications

**DroneCAN：面向航空与机器人应用的轻量级高可靠机内通信协议**

**Abstract—** Modern unmanned aerial vehicles (UAVs) and mobile robots are distributed embedded systems in which flight controllers, satellite-navigation receivers, compasses, airspeed sensors, rangefinders, electronic speed controllers (ESCs) and companion computers must exchange data continuously and dependably. Point-to-point interfaces such as PWM, UART, I2C and SPI scale poorly in this setting, whereas the Controller Area Network (CAN) offers differential signalling, priority-based arbitration and built-in error handling, but defines only the two lowest layers of the communication stack. DroneCAN is the open, lightweight application-level protocol that the open-source autopilots ArduPilot and PX4 use on top of CAN; it continues the development of the UAVCAN v0 protocol under a new name. The review proceeds from the bottom up. It summarises the relevant properties of CAN, traces the development from UAVCAN to DroneCAN and Cyphal, and describes the DroneCAN protocol in detail: its peer-to-peer network model without a master node, publish/subscribe and service semantics, the Data Structure Description Language (DSDL), the 29-bit CAN identifier layout, multi-frame transfers, the alternative CAN FD, UDP and MAVLink transports, and the standard application-level functions for node health monitoring, discovery, time synchronisation, configuration, firmware update and dynamic node-ID allocation. The reliability mechanisms are then analysed layer by layer, together with their limits, and DroneCAN is compared with Cyphal, MAVLink and conventional point-to-point interfaces. The presentation is based on the DroneCAN and Cyphal specifications, the PX4 documentation and peer-reviewed literature on CAN, MAVLink and UAV communication. Authentication, quantitative performance evaluation and migration to CAN FD are discussed as open directions.

> **摘要：** 现代无人机（UAV）和移动机器人是分布式嵌入式系统：飞控、卫星导航接收机、罗盘、空速计、测距仪、电子调速器（ESC）和机载计算机之间，需要持续、可靠地交换数据。PWM、UART、I2C、SPI 这类点对点接口在这种场景下扩展性差；控制器局域网（CAN）提供差分信号、基于优先级的仲裁和内置的错误处理，但只定义了通信栈的最低两层。DroneCAN 是开源自驾仪 ArduPilot 和 PX4 在 CAN 之上使用的、开放且轻量的应用层协议，由 UAVCAN v0 协议更名延续而来。综述自底向上展开：先总结 CAN 的相关特性，梳理从 UAVCAN 到 DroneCAN 与 Cyphal 的演变，再详细描述 DroneCAN 协议本身——无主节点的对等网络模型、发布/订阅与服务语义、数据结构描述语言（DSDL）、29 位 CAN 标识符布局、多帧传输、可选的 CAN FD / UDP / MAVLink 传输，以及用于节点健康监测、发现、时间同步、配置、固件更新和动态节点 ID 分配的标准应用层功能。随后逐层分析可靠性机制及其边界，并把 DroneCAN 与 Cyphal、MAVLink 和传统点对点接口比较。内容依据 DroneCAN 与 Cyphal 规范、PX4 文档，以及关于 CAN、MAVLink 和无人机通信的同行评议文献。认证、定量性能评估和向 CAN FD 迁移作为开放方向加以讨论。

**Keywords—** DroneCAN, UAVCAN, Cyphal, Controller Area Network, CAN FD, UAV, ArduPilot, PX4, MAVLink, publish/subscribe, reliability, intra-vehicle networks

> **关键词：** DroneCAN、UAVCAN、Cyphal、控制器局域网（CAN）、CAN FD、无人机、ArduPilot、PX4、MAVLink、发布/订阅、可靠性、机内网络

## I. INTRODUCTION / 引言

### A. Background: From Single Controllers to Distributed Systems / 背景：从单一控制器到分布式系统

A modern unmanned aerial vehicle (UAV, commonly called a drone) is no longer a single controller wired to a few actuators. It is a *distributed embedded system*: several devices, each containing its own processor, cooperate by exchanging data continuously during flight. A typical multirotor combines a *flight controller* (the circuit board that runs the autopilot software) with one or more *GNSS receivers* (satellite-navigation receivers such as GPS), a *magnetometer* (compass), an airspeed sensor, a rangefinder, several *electronic speed controllers* (ESCs, circuits that regulate motor speed), a battery monitor and, optionally, a *companion computer* (an additional computer for high-level tasks such as image processing). The DroneCAN specification describes the processors in such systems as ranging from high-performance on-board computers to extremely resource-constrained microcontrollers [1], and the devices often come from different vendors, which is why the same specification stresses inter-vendor compatibility [11].

> 现代无人机（UAV，俗称"无人机"）早已不是"一个控制器连几个执行器"。它是一个分布式嵌入式系统：若干设备各自含有处理器，在飞行中持续交换数据。典型的多旋翼包括：飞控（运行自驾仪软件的电路板）、一个或多个 GNSS 接收机（GPS 等卫星导航接收机）、磁力计（罗盘）、空速传感器、测距仪、若干电子调速器（ESC，调节电机转速的电路）、电池监测器，以及可选的机载计算机（用于图像处理等高层任务的附加计算机）。DroneCAN 规范指出，这类系统中的处理器从高性能机载计算机到资源极其受限的微控制器都有 [1]，且设备常来自不同厂商，因此规范强调跨厂商兼容性 [11]。

The PX4 autopilot illustrates how many components are involved. PX4 requires, as a minimum, a gyroscope, an accelerometer, a magnetometer and a barometer, all of which are integrated in Pixhawk-series flight controllers. External sensors extend this set: a GNSS receiver, or another source of global position, is needed to enable all automatic flight modes, airspeed sensors are highly recommended for fixed-wing and VTOL (vertical take-off and landing) vehicles, rangefinders for all vehicle types, and optical-flow sensors can support navigation without GNSS on multicopters and VTOL vehicles [14]. Each external device therefore adds a power supply, a physical interface and a software driver.

> PX4 自驾仪说明了涉及的部件之多。PX4 至少需要陀螺仪、加速度计、磁力计和气压计，这些都集成在 Pixhawk 系列飞控中。外部传感器进一步扩展这一集合：要启用全部自动飞行模式，需要 GNSS 接收机或其他全局位置来源；固定翼和 VTOL（垂直起降）飞行器强烈建议配备空速传感器；所有机型都建议配备测距仪；光流传感器可支持多旋翼和 VTOL 在无 GNSS 时导航 [14]。每增加一个外部设备，就多出一路供电、一个物理接口和一个软件驱动。

The conventional approach to connecting these devices is *point-to-point wiring*, in which each device is attached to the flight controller through a dedicated port and cable, using the interface that the device provides. In PX4, actuators are driven through PWM, DShot or OneShot outputs (signalling protocols by which a flight controller sends setpoints to motor controllers), which a flight controller typically provides in numbers of six to eight, divided into MAIN and AUX groups [14]. GNSS receivers, safety switches, buzzers and status LEDs are commonly integrated into a single module that is connected to a dedicated port [14]. This arrangement is simple to implement, but it has three limitations. First, the number of ports is determined by the hardware and therefore bounds the number of actuators and sensors that a vehicle can carry; according to the PX4 documentation, the limit of six to eight outputs results from the PWM, DShot and OneShot hardware, whereas a UAVCAN bus is not restricted to such a small number of nodes [14]. Second, each interface type requires its own cabling and driver, which increases the weight, the number of connectors and the number of potential points of failure; reduced wiring and simpler integration of additional hardware are likewise the motivation given by Ravi and El-Sharkawy for using UAVCAN on a UAV [19]. Third, these interfaces do not provide common services such as device discovery, status monitoring or firmware update, which DroneCAN standardises on the bus [9].

> 连接这些设备的传统做法是点对点接线：每个设备经专用端口和线缆连到飞控，使用设备自带的接口。在 PX4 中，执行器通过 PWM、DShot 或 OneShot 输出驱动（这些是飞控向电机控制器发送设定值的信号协议），飞控通常提供六到八路，分为 MAIN 和 AUX 两组 [14]。GNSS 接收机、安全开关、蜂鸣器和状态 LED 通常集成在一个模块中，接到专用端口 [14]。这种做法实现简单，但有三点限制。第一，端口数量由硬件决定，因此限制了飞行器可携带的执行器和传感器数量；按 PX4 文档，六到八路输出的上限来自 PWM、DShot 和 OneShot 硬件，而 UAVCAN 总线上的节点数不受如此小的限制 [14]。第二，每种接口都要各自的线缆和驱动，增加重量、连接器数量和潜在故障点；Ravi 和 El-Sharkawy 在无人机上使用 UAVCAN 的动机同样是减少接线、简化硬件集成 [19]。第三，这些接口不提供设备发现、状态监测、固件更新等通用服务，而 DroneCAN 在总线上把它们标准化了 [9]。

Communication faults can have severe consequences: the loss of the link between a sensor and the flight controller can result in the loss of the vehicle, damage to property or injury to people. The communication subsystem must therefore tolerate electrical noise and recover from transmission errors, properties that the Controller Area Network (CAN) provides through differential wiring and built-in error handling [19], [20]. The DroneCAN specification adds further requirements: a network without a master node, support for redundant interfaces and redundant nodes, and low-latency, high-throughput communication [1].

> 通信故障后果严重：传感器与飞控之间链路中断，可能导致飞行器损失、财产损坏或人员受伤。因此通信子系统必须容忍电气噪声并能从传输错误中恢复，控制器局域网（CAN）通过差分接线和内置错误处理提供了这些性质 [19]、[20]。DroneCAN 规范还提出更多要求：无主节点的网络、冗余接口和冗余节点的支持，以及低延迟、高吞吐的通信 [1]。

### B. ArduPilot and PX4 / ArduPilot 与 PX4

An *autopilot* is an embedded computer that runs the software providing control and guidance of an unmanned vehicle. Its key tasks include stabilising attitude and heading, flying missions along waypoints, triggering failsafe actions (predefined reactions to faults, such as returning to the launch point) and connecting to ground control stations [18]. An autopilot is *open source* when its hardware and software designs are publicly accessible and can be modified, which gives such projects flexibility, low cost and the support of a developer community [18].

> 自驾仪（autopilot）是运行软件、为无人载具提供控制与导航的嵌入式计算机，主要任务包括稳定姿态与航向、沿航点执行任务、触发失效保护动作（针对故障预先设定的反应，如返航），以及连接地面控制站 [18]。开源自驾仪的硬件和软件设计公开、可修改，因此项目灵活、成本低，并有开发者社区支持 [18]。

*ArduPilot* (originally APM) was developed by Jordi Muñoz and Chris Anderson from 2007 within the DIY Drones community, as an offshoot of the Arduino project, and was one of the first open-source autopilots to gain widespread use. It supports multicopters, fixed-wing aircraft, rovers and aquatic vehicles through the vehicle-specific variants ArduCopter, ArduPlane and ArduRover, and it is released under the GNU GPL licence [18]. ArduPilot was initially designed as monolithic software and later moved to the ChibiOS real-time operating system (RTOS), an operating system whose scheduler is designed to meet timing deadlines [18].

> ArduPilot（早期称 APM）由 Jordi Muñoz 和 Chris Anderson 自 2007 年起在 DIY Drones 社区开发，是 Arduino 项目的分支，也是最早获得广泛使用的开源自驾仪之一。它通过 ArduCopter、ArduPlane、ArduRover 等机型版本支持多旋翼、固定翼、地面车和水上载具，采用 GNU GPL 许可证 [18]。ArduPilot 最初是单体式软件，后来迁移到 ChibiOS 实时操作系统（RTOS，调度器按时限要求设计的操作系统）[18]。

*PX4* was initiated by Lorenz Meier in 2008 at ETH Zurich. Its hardware platform, Pixhawk, was released in cooperation with 3D Robotics, and the project is now managed by the Dronecode Foundation. The PX4 software is a modular flight control system that supports multicopters, fixed-wing aircraft, VTOL vehicles, rovers and boats, and it is released under a BSD licence [18]. PX4 runs on the NuttX RTOS [14] and is part of a broader platform that includes the QGroundControl ground station and the Pixhawk hardware [14].

> PX4 由 Lorenz Meier 于 2008 年在苏黎世联邦理工学院发起，其硬件平台 Pixhawk 与 3D Robotics 合作发布，项目现由 Dronecode 基金会管理。PX4 软件是模块化的飞控系统，支持多旋翼、固定翼、VTOL、地面车和船，采用 BSD 许可证 [18]。PX4 运行在 NuttX RTOS 之上 [14]，是更大平台的一部分，该平台还包括 QGroundControl 地面站和 Pixhawk 硬件 [14]。

Both projects use the MAVLink protocol (Section I-D) as the primary protocol for communication with ground stations [18]. PX4 additionally uses UAVCAN for communication between onboard components [18]. ArduPilot likewise uses CAN for high-speed communication between onboard components such as GNSS units, sensors and controllers [18]. This role of CAN-based communication is the subject of the following sections.

> 两个项目都把 MAVLink 协议（见 I-D 节）用作与地面站通信的主协议 [18]。PX4 还使用 UAVCAN 实现机载部件之间的通信 [18]；ArduPilot 同样用 CAN 在 GNSS、传感器和控制器等机载部件之间做高速通信 [18]。这种基于 CAN 的通信正是后文的主题。

### C. From CAN to DroneCAN / 从 CAN 到 DroneCAN

The Controller Area Network (CAN) was developed by Bosch for the automotive industry to replace complex wiring harnesses with a two-wire *bus*, that is, a shared communication line to which all devices are connected [19], [20]. It is a multi-master, message-broadcast system with a maximum signalling rate of about 1 Mbit/s that offers immunity to electrical noise and the ability to detect and recover from errors [19], [20]. CAN is standardised in ISO 11898, whose part 1 defines the data-link layer and physical signalling and whose part 2 defines the high-speed medium access unit [1]. CAN defines only the data-link and physical layers: it specifies how frames are transmitted, but not how applications name data, assign node addresses, split large data structures across frames or discover other devices. Without an *application-level protocol* that defines these aspects, each vendor would invent its own conventions and interoperability would be lost.

> 控制器局域网（CAN）由博世（Bosch）为汽车行业开发，用两线总线（所有设备接入的共享通信线）取代复杂的线束 [19]、[20]。它是多主、消息广播式的系统，最高信号速率约 1 Mbit/s，具有抗电气噪声及检测并恢复错误的能力 [19]、[20]。CAN 的标准是 ISO 11898，其中第 1 部分定义数据链路层与物理信号，第 2 部分定义高速介质访问单元 [1]。CAN 只定义数据链路层和物理层：它规定帧如何传输，但不规定应用如何命名数据、分配节点地址、把大数据结构拆到多帧或发现其他设备。没有定义这些的应用层协议，各厂商会各搞一套约定，互操作性就丧失了。

DroneCAN is such an application-level protocol for aerospace and robotic applications. The DroneCAN specification describes it as a lightweight protocol designed to provide a highly reliable communication method for aerospace and robotic applications via the CAN bus, organised as a decentralised peer network in which every node (every device on the bus) has a unique numeric identifier [2]. It is the primary CAN protocol used by the ArduPilot and PX4 projects to communicate with CAN peripherals, and it was created to continue the development of the widely used UAVCAN v0 protocol [12]. The PX4 documentation recommends DroneCAN for most common CAN setups [13].

> DroneCAN 就是面向航空与机器人应用的这样一种应用层协议。DroneCAN 规范把它描述为：通过 CAN 总线为航空与机器人应用提供高可靠通信的轻量协议，组织为去中心化的对等网络，其中每个节点（总线上的每个设备）都有唯一的数字标识 [2]。它是 ArduPilot 和 PX4 与 CAN 外设通信所用的主要 CAN 协议，为延续广泛使用的 UAVCAN v0 协议的开发而创建 [12]。PX4 文档对大多数常见 CAN 配置推荐 DroneCAN [13]。

### D. Related Protocols and Their Relationships / 相关协议及其关系

DroneCAN is one of several communication technologies available to UAV designers. *MAVLink* (Micro Air Vehicle Link) is a lightweight messaging protocol for communication between unmanned systems and ground stations and between autopilots and companion computers, and it is used by ArduPilot and PX4 [16], [18]. *Cyphal* is a transport-independent protocol whose primary transport is CAN FD, an extension of CAN for larger frames and higher data rates; the 2025 specification also describes experimental UDP/IP and serial transports [15]. Surveys of UAV communications and of open-source autopilots list these protocols side by side [17], [18].

> DroneCAN 只是无人机设计者可用的通信技术之一。MAVLink（Micro Air Vehicle Link）是无人系统与地面站之间、自驾仪与机载计算机之间通信的轻量消息协议，ArduPilot 和 PX4 都在用 [16]、[18]。Cyphal 是与传输层无关的协议，主传输是 CAN FD（CAN 的扩展，支持更大帧和更高速率）；2025 版规范还描述了实验性的 UDP/IP 和串行传输 [15]。关于无人机通信和开源自驾仪的综述把这些协议并列介绍 [17]、[18]。

The relationship between DroneCAN, Cyphal, UAVCAN and MAVLink is easily confused. UAVCAN was the name of the original open protocol for CAN-based communication between the components of a vehicle. In 2022 the project split in two: UAVCAN v0 was renamed DroneCAN, and UAVCAN v1 was renamed Cyphal [13]. DroneCAN is therefore the continuation of UAVCAN v0 and is the CAN protocol used by ArduPilot and PX4 [12], whereas Cyphal is the continuation of UAVCAN v1. The two descendants share design goals, such as a network without a master node and both publish/subscribe and request/response communication [1], [15], but they define different CAN frame formats [4], [15] and are therefore separate protocols, not versions of one another. Publications that predate the split and use the name UAVCAN, such as [19], refer to the protocol that is now called DroneCAN. MAVLink is independent of this lineage: it was released by Lorenz Meier in 2009 [16] and connects the vehicle with ground stations and companion computers, not the components inside the vehicle [16], [18]. The two protocols meet only where DroneCAN frames are carried inside MAVLink 2 messages [6]. Sections IV and VII treat these relationships in more detail.

> DroneCAN、Cyphal、UAVCAN 和 MAVLink 之间的关系容易混淆。UAVCAN 曾是基于 CAN 的飞行器部件间通信的原始开放协议。2022 年该项目一分为二：UAVCAN v0 更名为 DroneCAN，UAVCAN v1 更名为 Cyphal [13]。所以 DroneCAN 是 UAVCAN v0 的延续，是 ArduPilot 和 PX4 所用的 CAN 协议 [12]；Cyphal 是 UAVCAN v1 的延续。两个后继者有共同的设计目标，例如无主节点的网络、发布/订阅与请求/响应通信 [1]、[15]，但定义了不同的 CAN 帧格式 [4]、[15]，因此是彼此独立的协议，而不是同一协议的不同版本。分裂之前、使用 UAVCAN 名称的文献（如 [19]）指的是现在称为 DroneCAN 的协议。MAVLink 独立于这条血统：它由 Lorenz Meier 于 2009 年发布 [16]，连接飞行器与地面站和机载计算机，而不是飞行器内部的部件 [16]、[18]。两者只在 DroneCAN 帧被封装进 MAVLink 2 消息时交汇 [6]。第 IV 章和第 VII 章对这些关系有更详细的讨论。

### E. Organisation / 结构

Section II reviews the communication interfaces used on aerial and robotic platforms, and Section III summarises the properties of CAN on which DroneCAN builds. Section IV traces the development from UAVCAN to DroneCAN and Cyphal. Section V describes the design of DroneCAN, and Section VI analyses its reliability and safety mechanisms. Section VII compares DroneCAN with related technologies. Section VIII discusses open challenges, and Section IX concludes.

> 第 II 章回顾空中与机器人平台上的通信接口，第 III 章总结 DroneCAN 所建立在其上的 CAN 的特性。第 IV 章梳理从 UAVCAN 到 DroneCAN 和 Cyphal 的演变，第 V 章描述 DroneCAN 的设计，第 VI 章分析其可靠性与安全机制。第 VII 章把 DroneCAN 与相关技术比较，第 VIII 章讨论开放挑战，第 IX 章总结。

## II. COMMUNICATION INTERFACES IN AERIAL AND ROBOTIC PLATFORMS / 空中与机器人平台上的通信接口

### A. Architecture of a Typical UAV / 典型无人机的体系结构

The autopilot described in Section I-B is realised as flight stack software running on an RTOS on flight controller hardware. Some autopilots also include a general-purpose computer, or a separate companion computer, for higher-level command and control [14]. A *ground control station* (GCS) is the software, such as PX4's QGroundControl, that communicates with the vehicle, typically through a *telemetry radio*, a bidirectional wireless data link [14].

> I-B 节所述的自驾仪实现为：运行在 RTOS 上的飞行栈软件，跑在飞控硬件上。有些自驾仪还包含通用计算机，或另配机载计算机，负责更高层的指挥控制 [14]。地面控制站（GCS）是与飞行器通信的软件，例如 PX4 的 QGroundControl，通常经遥测电台（双向无线数据链路）通信 [14]。

Fig. 1 shows the resulting logical architecture. The flight controller reads sensors, computes actuator commands and communicates with the outside world through several different links.

> 图 1 给出由此得到的逻辑体系结构：飞控读取传感器、计算执行器指令，并通过多条不同链路与外界通信。

![Fig. 1](Figuras/Figura_1.svg)

*Fig. 1. Typical communication links around a flight controller. MAVLink serves the ground station and companion computer; DroneCAN serves intra-vehicle peripherals.*

> *图 1. 飞控周围的典型通信链路。MAVLink 服务于地面站和机载计算机；DroneCAN 服务于飞行器内部外设。*

### B. Point-to-Point Interfaces / 点对点接口

**PWM and related actuator protocols.** *Pulse-width modulation* (PWM) encodes a setpoint in the width of the pulses of a periodic signal. In PX4, outputs may be PWM ports or DroneCAN nodes, for example DroneCAN motor controllers [14]. The output ports (MAIN and AUX) are typically six to eight in number because most controllers have only that many PWM, DShot or OneShot outputs [14]. The output ports can use DShot or OneShot in addition to PWM, which offer much lower latency and are relevant for racing and other demanding airframes [14]. A DroneCAN ESC, by contrast, can report its own state: the standard ESC status message carries an error count, voltage, current, temperature and rotational speed [10].

> **PWM 及相关执行器协议。** 脉宽调制（PWM）把设定值编码在周期信号的脉冲宽度中。在 PX4 中，输出可以是 PWM 端口，也可以是 DroneCAN 节点，例如 DroneCAN 电机控制器 [14]。输出端口（MAIN 和 AUX）通常是六到八路，因为多数控制器只有这么多路 PWM、DShot 或 OneShot 输出 [14]。输出端口除 PWM 外还可使用 DShot 或 OneShot，它们的延迟低得多，与竞速等苛刻机型相关 [14]。相比之下，DroneCAN 电调可以回报自身状态：标准的电调状态消息携带错误计数、电压、电流、温度和转速 [10]。

**UART, I2C and SPI.** These are serial interfaces: a UART (universal asynchronous receiver-transmitter) provides a point-to-point serial link, while I2C and SPI are short-distance buses for connecting peripheral chips and modules. In the PX4 documentation, a GNSS module that combines a receiver and a compass is attached to the controller's GPS port, together with the safety switch and the buzzer, and the user-interface LED on recent controllers is typically an I2C peripheral integrated into the same module [14]. An external compass is recommended because it can be made less susceptible to electromagnetic interference than the internal compass of the flight controller [14]. Autopilots also use serial links for telemetry radios and for companion computers, which are connected by a serial cable, an Ethernet cable or Wi-Fi [14].

> **UART、I2C 和 SPI。** 这些是串行接口：UART（通用异步收发器）提供点对点串行链路，I2C 和 SPI 是连接外围芯片和模块的短距离总线。在 PX4 文档中，集成接收机和罗盘的 GNSS 模块连到控制器的 GPS 端口，安全开关和蜂鸣器也在其中，较新控制器上的用户界面 LED 通常是集成在同一模块里的 I2C 外设 [14]。建议使用外置罗盘，因为它比飞控内置罗盘更容易做到对电磁干扰不敏感 [14]。自驾仪还用串行链路连接遥测电台和机载计算机，机载计算机通过串口线、以太网线或 Wi-Fi 连接 [14]。

**Constraints.** Section I-A identified three constraints of point-to-point wiring: the number of ports is fixed by hardware [14], each interface type needs its own driver and cabling, and common services such as discovery, status monitoring and firmware update are not defined by the interfaces themselves. DroneCAN standardises these services on the bus [9].

> **约束。** I-A 节指出点对点接线的三点约束：端口数由硬件固定 [14]；每种接口需要自己的驱动和线缆；发现、状态监测、固件更新等通用服务不是由接口本身定义的。DroneCAN 在总线上把这些服务标准化 [9]。

### C. Higher-Level Links: MAVLink / 更高层链路：MAVLink

The Micro Air Vehicle Link (MAVLink) is a communication protocol for unmanned systems that specifies a comprehensive set of messages exchanged between unmanned systems and ground stations [16]. It is a lightweight messaging protocol, released by Lorenz Meier in 2009 under the LGPL, which is designed as a *marshalling library*: it serialises (converts into a sequence of bytes for transmission) messages describing the state of the system and the commands sent to it [16]. ArduPilot and PX4 rely on MAVLink as their primary protocol for telemetry and control [18]. A MAVLink packet consists of a header carrying information about the message and a payload carrying the data [16]. MAVLink is therefore a vehicle-to-ground and autopilot-to-companion protocol, and it is not intended to replace the real-time bus between sensors and actuators inside the airframe. DroneCAN is positioned for that role, and, as Section V-E shows, it can itself be tunnelled over MAVLink [6].

> 微型飞行器链路（MAVLink）是无人系统的通信协议，规定了无人系统与地面站之间交换的一整套消息 [16]。它是轻量消息协议，由 Lorenz Meier 于 2009 年以 LGPL 发布，设计为一个编组库：把描述系统状态的消息和发给系统的指令序列化（转换为字节序列以便传输）[16]。ArduPilot 和 PX4 把 MAVLink 用作遥测与控制的主协议 [18]。MAVLink 数据包由携带消息信息的包头和携带数据的载荷组成 [16]。因此 MAVLink 是飞行器到地面、自驾仪到机载计算机的协议，并不打算取代机体内传感器与执行器之间的实时总线。DroneCAN 定位于后一角色，而且如 V-E 节所示，它本身也可以通过 MAVLink 隧道传输 [6]。

### D. Summary of Requirements / 需求小结

From the discussion above, an intra-vehicle bus for UAVs and robots has to meet the following requirements, which closely match the design goals stated in the DroneCAN specification [1]:

> 由上述讨论可知，无人机和机器人的机内总线需要满足下列需求，它们与 DroneCAN 规范所述的设计目标高度吻合 [1]：

- no single point of failure, and therefore no bus master (a node that controls access for all others);  
  无单点故障，因而没有总线主节点（控制所有其他节点访问权的节点）；
- a simple way to exchange payloads that exceed a single frame;  
  能简单地交换超过单帧的载荷；
- support for redundant interfaces and redundant nodes;  
  支持冗余接口和冗余节点；
- low latency and high throughput for hard real-time control loops, that is, control loops that must meet strict timing deadlines;  
  面向硬实时控制回路（必须满足严格时限的控制回路）的低延迟、高吞吐；
- low computational requirements, down to resource-constrained microcontrollers;  
  计算要求低，可低至资源受限的微控制器；
- standard definitions of the high-level functions common to all nodes, such as discovery, configuration and firmware update;  
  对所有节点通用的高层功能（发现、配置、固件更新）有标准定义；
- an open specification with open reference implementations.  
  开放的规范和开放的参考实现。

## III. THE CONTROLLER AREA NETWORK / 控制器局域网（CAN）

DroneCAN inherits most of its physical and link-layer behaviour from CAN. This section summarises the properties that are relevant for the analysis of reliability in Section VI.

> DroneCAN 的大部分物理层与链路层行为继承自 CAN。本章总结与第 VI 章可靠性分析相关的特性。

### A. Physical Layer and Frame Structure / 物理层与帧结构

CAN was developed by Bosch as a multi-master, broadcast communication interface for in-vehicle networks [20]. *Multi-master* means that every node may start a transmission, and *broadcast* means that every transmitted frame can be received by all nodes. CAN uses differential wiring, that is, information is carried by the voltage difference between two wires so that interference acting on both wires cancels out, and two voltage levels: a dominant logic "0" and a recessive logic "1". No dedicated clock line is used; synchronisation is provided through signal edges and *bit stuffing* [20]. The bit-stuffing rule limits the number of repeated bits: after five consecutive bits of the same logic level, the next bit must be the complement, and the transmitter inserts such a complement bit that the receiver removes [20]. The two bus wires, CAN-High and CAN-Low, are terminated with a resistance of about 120 Ω, which prevents signal reflections [19].

> CAN 由博世开发，是用于车内网络的多主广播通信接口 [20]。多主指每个节点都可发起传输，广播指每个发出的帧都能被所有节点接收。CAN 采用差分接线（信息由两根线之间的电压差承载，使同时作用于两线的干扰相互抵消）和两种电压电平：显性逻辑"0"和隐性逻辑"1"。没有专用时钟线；同步通过信号边沿和位填充实现 [20]。位填充规则限制重复位的数量：连续五个相同电平位之后，下一位必须取反，发送方插入这样的补位，接收方再将其去掉 [20]。两根总线线 CAN-High 和 CAN-Low 两端用约 120 Ω 的电阻终接，防止信号反射 [19]。

In road vehicles, the intra-vehicle network connects approximately 70 electronic control units (ECUs) that exchange around 2500 signals [22]. Five network types are in wide use there: LIN, CAN, FlexRay, Ethernet and MOST. Among them, CAN is the most popular because of its relatively low cost, mature tool chains, acceptable noise resistance and fault tolerance [22]. A typical CAN installation has a bus topology with a baud rate above 125 kbit/s and a 120 Ω terminating resistor at each end of the cable [22]. These properties explain why the same bus has also been adopted for UAV intra-vehicle communication.

> 在道路车辆中，机内网络连接约 70 个电子控制单元（ECU），内部交换约 2500 个信号 [22]。那里广泛使用五种网络：LIN、CAN、FlexRay、以太网和 MOST。其中 CAN 最流行，原因是成本较低、工具链成熟、抗噪声能力和容错性可接受 [22]。典型 CAN 安装采用总线拓扑，波特率高于 125 kbit/s，线缆两端各有一个 120 Ω 终端电阻 [22]。这些特性说明了为什么同一种总线也被用于无人机的机内通信。

A CAN frame carries a start-of-frame bit, an identifier field that also encodes the message priority, the data, a CRC checksum (cyclic redundancy check, a code that allows errors in the data to be detected), acknowledgement bits and end-of-frame bits [19], [20]. Two identifier formats exist: standard CAN uses an 11-bit identifier, which provides 2048 distinct identifiers, and extended CAN uses a 29-bit identifier, which provides about 537 million identifiers; both formats are compatible and can coexist on the same bus [19]. DroneCAN uses only the CAN 2.0B format with 29-bit identifiers, while remaining able to share a bus with protocols based on CAN 2.0A (11-bit identifiers) [4].

> CAN 帧包含起始位、同时编码消息优先级的标识符字段、数据、CRC 校验（循环冗余校验，一种可检测数据错误的编码）、应答位和结束位 [19]、[20]。有两种标识符格式：标准 CAN 用 11 位标识符，共 2048 个不同标识符；扩展 CAN 用 29 位标识符，约 5.37 亿个；两种格式兼容，可在同一总线上共存 [19]。DroneCAN 只使用 CAN 2.0B 格式的 29 位标识符，同时仍能与基于 CAN 2.0A（11 位标识符）的协议共用一条总线 [4]。

### B. Arbitration and Priority / 仲裁与优先级

Collisions are resolved by a bit-wise *arbitration* based on the priority encoded in the identifier field: when two nodes start to transmit at the same time, the node with the higher-priority identifier continues and the other node stops [19], [20]. The dominant bit wins, so the numerically lowest identifier has the highest priority [19]. For a safety-oriented protocol, this has an important consequence: the placement of fields in the 29-bit identifier determines which messages pre-empt others. DroneCAN uses this property explicitly by making the 5-bit priority field the first, highest-order field of the identifier (Section V-D) [4].

> 冲突通过基于标识符字段所编码优先级的逐位仲裁解决：两个节点同时开始发送时，标识符优先级更高的节点继续发送，另一个停止 [19]、[20]。显性位获胜，所以数值最小的标识符优先级最高 [19]。对面向安全的协议而言这有重要后果：29 位标识符中各字段的位置决定了哪些消息可以抢占其他消息。DroneCAN 明确利用了这一点，把 5 位优先级字段放在标识符最高位（见 V-D 节）[4]。

### C. Error Detection and Fault Confinement / 错误检测与故障隔离

CAN has several built-in mechanisms that make it robust. The start-of-frame bit, the cyclic redundancy check for data integrity, the acknowledgement bits for successful transmission and the end-of-frame bits are the error-checking methods of CAN, and the bit-stuffing rule supports synchronisation [20].

> CAN 有若干内置机制使其健壮。起始位、保证数据完整性的循环冗余校验、确认成功传输的应答位和结束位，是 CAN 的错误检查方法，位填充规则则辅助同步 [20]。

To prevent a faulty node from disturbing the whole network, CAN includes an error-confinement mechanism. Each node maintains a transmit error counter and a receive error counter: the counter increases by eight when an error occurs during transmission and by one when an error occurs during reception. When the counter exceeds 127 the node enters the *error-passive* state, in which its error frames no longer affect bus traffic, and when it exceeds 255 the node enters the *bus-off* state and no longer takes part in bus traffic [20]. In this way a node with a persistent fault eliminates itself from the bus.

> 为防止故障节点干扰整个网络，CAN 含有错误限制机制。每个节点维护发送错误计数器和接收错误计数器：发送中出错计数加 8，接收中出错计数加 1。计数超过 127 时节点进入错误被动状态，其错误帧不再影响总线通信；超过 255 时进入总线关闭状态，不再参与总线通信 [20]。这样，持续故障的节点会把自己从总线上剔除。

### D. CAN FD

Classical CAN carries at most 8 bytes of data in a frame [7] and is limited to about 1 Mbit/s [19], [20]. CAN with flexible data rate (CAN FD) extends the frame to up to 64 bytes and allows the data phase to be transmitted at a higher bit rate than the arbitration phase [7]. DroneCAN defines a CAN FD transport in which the nominal bit rate is usually 1 Mbit/s, with bit-rate switching used to run the data phase at 1, 2, 4, 5 or 8 Mbit/s depending on hardware capabilities [7]. Because only 4 bits are available for the data-length code (DLC), lengths above 8 bytes are quantised to 12, 16, 20, 24, 32, 48 and 64 bytes [7]. Cyphal treats CAN FD as its primary CAN transport, with a single-frame payload (the application data that fits into one frame) of up to 63 bytes, compared with 7 bytes for classical CAN [15].

> 经典 CAN 每帧最多携带 8 字节数据 [7]，速率上限约 1 Mbit/s [19]、[20]。灵活数据速率 CAN（CAN FD）把帧扩展到最多 64 字节，并允许数据段以高于仲裁段的比特率传输 [7]。DroneCAN 定义了 CAN FD 传输，其标称比特率通常为 1 Mbit/s，用比特率切换使数据段按硬件能力运行在 1、2、4、5 或 8 Mbit/s [7]。由于数据长度码（DLC）只有 4 位，超过 8 字节的长度被量化为 12、16、20、24、32、48 和 64 字节 [7]。Cyphal 把 CAN FD 作为其首选 CAN 传输，单帧载荷（一帧能装下的应用数据）最多 63 字节，经典 CAN 则是 7 字节 [15]。

### E. Security Limitations / 安全局限

CAN was designed in a time when security was not a primary concern; it was used to connect a small number of electronic control units (ECUs) that were not accessible to the end user [20]. The principal weaknesses identified in the literature are the lack of encryption and authentication. Al-Jarrah et al. give the same two root causes: CAN has no message authentication and uses broadcast transmission, so an intruder who gains access to the bus can send messages because nothing verifies their origin [22]. They also note that conventional proactive countermeasures such as encryption and access control may not be applicable to vehicles because of their high connectivity, time sensitivity, resource constraints and complexity, which has directed attention to reactive measures such as intrusion detection systems (IDSs) [22]. Because CAN is a broadcast network, there are no source and destination addresses, every node can listen to every message, and any node can inject frames; CAN is also vulnerable to denial of service, since a malicious node that holds the highest priority and transmits all the time prevents the other nodes from communicating, and the error-confinement mechanism can be abused to force a node into the bus-off state [20]. In an analysis based on the confidentiality–integrity–availability triad, Bozdal et al. conclude that CAN fails all three criteria, and that the proposed countermeasures (network segmentation, encryption, authentication and intrusion detection) each have drawbacks such as bandwidth overhead, latency or incompatibility with standard CAN controllers, so that no solution has been approved by industry and academia [20]. They also point out that protocols implemented on top of the CAN physical layer, such as SAE J1939, can themselves be attacked [20]. These conclusions were reached in the automotive context, but they apply to protocols layered on CAN, including DroneCAN, whose transport layer defines no authentication or encryption fields [4]. Section VI-G returns to this point.

> CAN 设计于安全并非首要考虑的年代；它用来连接少量最终用户无法接触的电子控制单元（ECU）[20]。文献指出的主要弱点是缺少加密和认证。Al-Jarrah 等人给出了同样的两个根本原因：CAN 没有消息认证并采用广播传输，因此一旦入侵者接入总线，就能发送消息，因为没有任何机制验证其来源 [22]。他们还指出，加密和访问控制等传统的主动防护措施，由于车辆的高连接性、时间敏感性、资源受限和复杂性，可能并不适用，这使得人们把注意力转向入侵检测系统（IDS）等被动式措施 [22]。因为 CAN 是广播网络，没有源地址和目的地址，每个节点都能监听所有消息，任何节点都能注入帧；CAN 还易受拒绝服务攻击：持续以最高优先级发送的恶意节点会使其他节点无法通信，错误限制机制也可能被滥用，迫使某节点进入总线关闭状态 [20]。Bozdal 等人基于机密性—完整性—可用性三要素的分析认为，CAN 三项均不满足；所提出的对策（网络分段、加密、认证和入侵检测）各有代价，如带宽开销、延迟或与标准 CAN 控制器不兼容，因此尚无被产业界和学术界认可的方案 [20]。他们还指出，实现在 CAN 物理层之上的协议（如 SAE J1939）本身也可能被攻击 [20]。这些结论是在汽车语境下得出的，但适用于构建在 CAN 之上的协议，包括 DroneCAN，其传输层没有定义认证或加密字段 [4]。VI-G 节将回到这一点。

| Property | Classical CAN | CAN FD |
|---|---|---|
| Identifier | 11-bit (2.0A) or 29-bit (2.0B) [19] | 11-bit or 29-bit |
| Data per frame | up to 8 bytes [7] | up to 64 bytes [7] |
| Bit rate | up to about 1 Mbit/s [19], [20] | nominal 1 Mbit/s; data phase up to 8 Mbit/s depending on hardware [7] |
| Arbitration | bit-wise, lowest identifier wins [19], [20] | same |
| Error handling | error counters, error-passive and bus-off states [20] | same |
| Authentication / encryption | none [20] | none |

> | 属性 | 经典 CAN | CAN FD |
> |---|---|---|
> | 标识符 | 11 位（2.0A）或 29 位（2.0B）[19] | 11 位或 29 位 |
> | 每帧数据 | 最多 8 字节 [7] | 最多 64 字节 [7] |
> | 比特率 | 最高约 1 Mbit/s [19]、[20] | 标称 1 Mbit/s；数据段视硬件最高 8 Mbit/s [7] |
> | 仲裁 | 逐位，最小标识符获胜 [19]、[20] | 相同 |
> | 错误处理 | 错误计数器、错误被动与总线关闭 [20] | 相同 |
> | 认证/加密 | 无 [20] | 无 |

*Table I. Selected properties of classical CAN and CAN FD.*

> *表 I. 经典 CAN 与 CAN FD 的部分属性。*

## IV. FROM UAVCAN TO DRONECAN AND CYPHAL / 从 UAVCAN 到 DroneCAN 与 Cyphal

### A. Origins / 起源

DroneCAN is based on earlier work by Pavel Kirienko and the UAVCAN Development Team, as the DroneCAN specification states [1]. The inheritance is visible in the artefacts of the protocol itself: the root namespace of every standard data type is still `uavcan` (for example `uavcan.protocol.NodeStatus`), the files that define data structures carry the extension `.uavcan`, and the standard configuration parameters share the prefix `uavcan.` [3], [9]. It is also visible in the wording of the definition. Ravi and El-Sharkawy describe UAVCAN in 2019 as a protocol "designed for reliable communication for aerospace and robotic applications via CAN bus" and as a decentralised network in which each node has a unique node identifier [19]; the DroneCAN specification uses the same description for DroneCAN today [2].

> DroneCAN 以 Pavel Kirienko 和 UAVCAN 开发团队此前的工作为基础，DroneCAN 规范对此有明确说明 [1]。这种传承在协议本身的产物中清晰可见：每个标准数据类型的根命名空间仍是 uavcan（例如 uavcan.protocol.NodeStatus），定义数据结构的文件扩展名是 .uavcan，标准配置参数共用前缀 uavcan. [3]、[9]。定义的措辞也体现了这一点。Ravi 和 El-Sharkawy 在 2019 年把 UAVCAN 描述为"通过 CAN 总线为航空与机器人应用设计的可靠通信协议"，是每个节点都有唯一节点标识的去中心化网络 [19]；DroneCAN 规范今天对 DroneCAN 用的是同样的描述 [2]。

Ravi and El-Sharkawy report a UAV integration experiment with UAVCAN. A Pixhawk flight controller running PX4 was connected over a CAN bus to an NXP Rapid IoT board running FreeRTOS (a small real-time operating system for microcontrollers). Two additional UAVCAN messages were defined in the PX4 firmware to carry data in each direction, and a Zubax Babel adapter (a USB-to-CAN interface) together with the UAVCAN GUI tool was used to observe the CAN frames [19]. The motivation was to attach a Thread-based wireless module (Thread is a low-power wireless networking standard) to the flight controller with fewer wires and easier hardware integration. The results are qualitative: frames sent by the autopilot were visible in a terminal, and frames sent by the RTOS board were visible in the GUI tool [19]. The study demonstrates that a second embedded platform can be attached to a PX4 flight controller through UAVCAN.

> Ravi 和 El-Sharkawy 报告了一次用 UAVCAN 做无人机集成的实验。运行 PX4 的 Pixhawk 飞控经 CAN 总线连接到运行 FreeRTOS（面向微控制器的小型实时操作系统）的 NXP Rapid IoT 板；在 PX4 固件中新增两条 UAVCAN 消息，分别承载两个方向的数据，并用 Zubax Babel 适配器（USB 转 CAN 接口）配合 UAVCAN GUI 工具观察 CAN 帧 [19]。动机是以更少的线缆、更简便的硬件集成，把一个基于 Thread 的无线模块（Thread 是一种低功耗无线组网标准）接到飞控上。结果是定性的：自驾仪发出的帧能在终端看到，RTOS 板发出的帧能在 GUI 工具中看到 [19]。该研究表明，可以通过 UAVCAN 把第二个嵌入式平台接到 PX4 飞控上。

### B. The 2022 Split / 2022 年的分裂

The PX4 documentation records that in 2022 the original UAVCAN project was divided into two separate projects: UAVCAN v0 was renamed DroneCAN, while UAVCAN v1 was renamed Cyphal [13]. The same page recommends DroneCAN for most common CAN setups [13], and the DroneCAN website describes it as the primary CAN protocol used by ArduPilot and PX4 to communicate with CAN peripherals, created to continue the development of the widely used UAVCAN v0 protocol [12]. The name "UAVCAN" in older publications, forum posts and firmware settings therefore usually refers to what is now DroneCAN, whereas the newer Cyphal is a separate protocol. For this reason the name "UAVCAN" is used below only for the historical protocol or when a cited source uses it.

> PX4 文档记载，2022 年原 UAVCAN 项目被分为两个独立项目：UAVCAN v0 更名为 DroneCAN，UAVCAN v1 更名为 Cyphal [13]。同一页面对大多数常见 CAN 配置推荐 DroneCAN [13]，DroneCAN 网站则把它描述为 ArduPilot 和 PX4 与 CAN 外设通信所用的主要 CAN 协议，为延续广泛使用的 UAVCAN v0 协议的开发而创建 [12]。因此，旧文献、论坛帖和固件设置中的"UAVCAN"通常指现在的 DroneCAN，而较新的 Cyphal 是另一种协议。出于这一原因，下文仅在指历史协议或所引来源使用该名称时才用"UAVCAN"。

### C. Cyphal

Cyphal is described in its own specification as an open technology for real-time intravehicular distributed computing and communication, based on modern networking standards such as Ethernet and CAN FD, and aimed at manned and unmanned aircraft, spacecraft, robots and cars [15]. Its listed features resemble the DroneCAN design goals: a democratic network without a bus master, publish/subscribe and request/response semantics, automatic decomposition and reassembly of large data structures, support for dual and triply modular redundant transports, network-wide time synchronisation and an interface description language [1], [15]. Version 1.0 of the specification (16 May 2025) records that a physical-layer specification was removed in v1.0-beta (September 2020), that the subject-ID range was reduced from [0, 32767] to [0, 8191], and that Cyphal/UDP and Cyphal/serial transports were introduced in v1.0 [15]. Cyphal/CAN treats CAN FD as the primary transport protocol, supports a maximum node ID of 127, uses eight transfer priority levels, and carries up to 7 bytes in a single classic-CAN frame or up to 63 bytes in a single CAN FD frame [15]. Section VII-A compares these properties with those of DroneCAN in detail.

> Cyphal 自己的规范把它描述为面向实时机内分布式计算与通信的开放技术，基于以太网和 CAN FD 等现代网络标准，面向有人与无人飞机、航天器、机器人和汽车 [15]。其列出的特性与 DroneCAN 的设计目标相似：没有总线主节点的民主网络、发布/订阅与请求/响应语义、大数据结构的自动拆分与重组、对双重和三重模块冗余传输的支持、全网时间同步，以及接口描述语言 [1]、[15]。规范 v1.0（2025 年 5 月 16 日）记载：物理层规范在 v1.0-beta（2020 年 9 月）中被移除，subject-ID 范围从 [0, 32767] 缩小到 [0, 8191]，Cyphal/UDP 和 Cyphal/serial 传输在 v1.0 中引入 [15]。Cyphal/CAN 把 CAN FD 作为首选传输协议，支持的最大节点 ID 为 127，使用八个传输优先级，单个经典 CAN 帧最多携带 7 字节、单个 CAN FD 帧最多携带 63 字节 [15]。VII-A 节将详细对比这些属性与 DroneCAN 的差异。

| Generation | Name (until 2022) | Name (since 2022) | Transports defined in the cited specification | Source |
|---|---|---|---|---|
| Version 0 | UAVCAN v0 | DroneCAN | CAN 2.0B, CAN FD, UDP multicast, MAVLink tunnel | [1], [4]–[7], [13] |
| Version 1 | UAVCAN v1 | Cyphal | CAN (CAN FD primary), UDP (experimental), serial | [13], [15] |

> | 代际 | 名称（2022 年前） | 名称（2022 年起） | 所引规范中定义的传输 | 来源 |
> |---|---|---|---|---|
> | 版本 0 | UAVCAN v0 | DroneCAN | CAN 2.0B、CAN FD、UDP 组播、MAVLink 隧道 | [1]、[4]–[7]、[13] |
> | 版本 1 | UAVCAN v1 | Cyphal | CAN（CAN FD 为主）、UDP（实验性）、串行 | [13]、[15] |

*Table II. Naming and transports of the two protocol generations.*

> *表 II. 两代协议的命名与传输。*

## V. DESIGN OF DRONECAN / DroneCAN 的设计

### A. Design Goals and Layered Structure / 设计目标与分层结构

The DroneCAN specification lists seven core design goals: a democratic network without a master node; automatic transfer decomposition and reassembly for payloads that do not fit into a single CAN frame; support for redundant interfaces and redundant nodes; high-throughput, low-latency communication; simple logic and low computational requirements; clearly defined standard high-level functions; and an open specification with reference implementations under the MIT licence [1]. The same chapter specifies how the standard evolves within a major version: new data types may be added, existing types may be modified only if backward compatibility is preserved, and a type may be declared deprecated, after which it is maintained for at least two more years before its default ID may be reused [1].

> DroneCAN 规范列出七条核心设计目标：没有主节点的民主网络；对单个 CAN 帧装不下的载荷自动拆分与重组；支持冗余接口和冗余节点；高吞吐、低延迟通信；逻辑简单、计算要求低；清晰定义的标准高层功能；以及开放规范，参考实现采用 MIT 许可证 [1]。同一章还规定了标准在同一主版本内如何演进：可以新增数据类型；已有类型只有在保持向后兼容时才可修改；类型可被声明为废弃，此后至少再维护两年，其默认 ID 才可被复用 [1]。

The protocol can be understood as the layered structure shown in Fig. 2. Serialised data structures (data converted into a sequence of bytes), defined in the Data Structure Description Language (DSDL), are exchanged by a transport layer that decomposes long transfers into frames [2]. Standard application-level functions are built on top of the standard data types [2].

> 协议可以理解为图 2 所示的分层结构。用数据结构描述语言（DSDL）定义的序列化数据结构（转换成字节序列的数据）由传输层交换，传输层把长传输拆成帧 [2]。标准应用层功能建立在标准数据类型之上 [2]。

![Fig. 2](Figuras/Figura_2.svg)

*Fig. 2. Layered structure of DroneCAN. The alternative transports are described in Section V-E.*

> *图 2. DroneCAN 的分层结构。备选传输见 V-E 节。*

### B. Network Model and Communication Semantics / 网络模型与通信语义

A DroneCAN network is a decentralised peer network. Every peer (node) has a unique numeric node ID, and nodes communicate by one of two methods [2].

> DroneCAN 网络是去中心化的对等网络。每个对等体（节点）都有唯一的数字节点 ID，节点通过两种方式之一通信 [2]。

**Message broadcasting** is the primary method and follows *publish/subscribe* semantics: a node publishes data without addressing a particular receiver, and every node that has subscribed to that kind of data receives it. A broadcast message carries the serialised payload, the data type ID, the source node ID and a transfer ID, and typical uses are sensor measurements, actuator commands and equipment status [2]. Nodes that do not yet have a node ID may publish *anonymous messages*, which have no source node ID and are used mainly for initial configuration and dynamic node ID allocation [2]. An anonymous transfer can only be a single-frame transfer [4].

> 消息广播是主要方式，遵循发布/订阅语义：节点发布数据而不指定接收方，订阅了该类数据的每个节点都会收到。广播消息携带序列化载荷、数据类型 ID、源节点 ID 和传输 ID，典型用途是传感器测量、执行器指令和设备状态 [2]。尚无节点 ID 的节点可以发布匿名消息，匿名消息没有源节点 ID，主要用于初始配置和动态节点 ID 分配 [2]。匿名传输只能是单帧传输 [4]。

**Service invocation** is a two-step exchange (request/response) between exactly two nodes. The client sends a request, the server acts and sends a response, and typical uses are parameter updates, firmware update, file transfer and ad-hoc action requests [2]. Request and response contain the same values in all fields except the payload, so the client matches a response to its request using the data type ID, the client node ID, the server node ID and the transfer ID [2].

> 服务调用是恰好两个节点之间的两步交换（请求/响应）。客户端发送请求，服务器执行并回送响应，典型用途是参数更新、固件更新、文件传输和临时动作请求 [2]。请求和响应除载荷外所有字段取值相同，因此客户端用数据类型 ID、客户端节点 ID、服务器节点 ID 和传输 ID 把响应与请求匹配起来 [2]。

Valid node IDs range from 1 to 127, and the value 0 is reserved for an unknown node or all nodes, depending on the context [4]. The values 126 and 127 are reserved for debugging tools and should not be used for real nodes [8]. Apart from the anonymous case, a message must be sent by a node with a network-unique node ID [4].

> 有效节点 ID 范围为 1 到 127，值 0 按上下文保留表示未知节点或所有节点 [4]。126 和 127 保留给调试工具，不应用于真实节点 [8]。除匿名情形外，消息必须由具有网络内唯一节点 ID 的节点发送 [4]。

Redundant functionality is supported without a special mechanism. Because each published message type has its own data type ID and each node its own node ID, the pair of data type ID and node ID can be used to support redundant nodes with identical functionality inside the same network [2]. Two GNSS receivers publishing the same message type, for example, are distinguished by their node IDs, and a consumer can select between them.

> 冗余功能不需要特殊机制来支持。由于每种发布的消息类型有自己的数据类型 ID，每个节点有自己的节点 ID，"数据类型 ID + 节点 ID"这一对就可用来支持同一网络内功能相同的冗余节点 [2]。例如，两个发布相同消息类型的 GNSS 接收机靠节点 ID 区分，消费者可以在它们之间选择。

### C. Data Structure Description Language / 数据结构描述语言

Message and service structures are defined in the Data Structure Description Language (DSDL). A DSDL compiler generates serialisation and deserialisation code for each target programming language from the definition files. Because the data structure size can be determined statically, the generated code can avoid dynamic memory allocation, which is important for deeply embedded systems with little memory [2], [3].

> 消息和服务的结构用数据结构描述语言（DSDL）定义。DSDL 编译器根据定义文件为每种目标编程语言生成序列化与反序列化代码。因为数据结构的大小可以静态确定，生成的代码可以避免动态内存分配，这对内存很小的深度嵌入式系统很重要 [2]、[3]。

**Definition files.** Each file defines one data structure for message broadcasting or a request/response pair for a service, and the file name consists of the optional default data type ID and the data type name. The structure lives in a namespace (a named group of definitions that prevents name clashes), and the namespace hierarchy maps onto the file-system directory structure. The standard data types are in the root namespace `uavcan`, and an example is the file `341.NodeStatus.uavcan` in the directory `uavcan/protocol` [3]. The full data type name may not exceed 80 characters [3]. Services use a marker line of three hyphens to separate the request from the response part [3].

> **定义文件。** 每个文件定义一个用于消息广播的数据结构，或一个服务的请求/响应对，文件名由可选的默认数据类型 ID 和数据类型名组成。结构位于命名空间（防止重名的一组命名定义）中，命名空间层次映射到文件系统目录结构。标准数据类型在根命名空间 uavcan 下，例如目录 uavcan/protocol 中的文件 341.NodeStatus.uavcan [3]。完整数据类型名不得超过 80 个字符 [3]。服务用一行三个连字符的标记行把请求部分与响应部分分开 [3]。

**Type system.** Fields may be `bool`, signed (`intX`) and unsigned (`uintX`) integers of 2 to 64 bits, `float16`, `float32`, `float64`, nested structures, and arrays that are either static (`[X]`) or dynamic (`[<X]` or `[<=X]`). Special `voidX` types insert 1 to 64 padding bits for alignment, and a `@union` directive declares a tagged union in which only one field is encoded at a time [3]. Two cast modes control the conversion of a value to a narrower representation: *saturated*, the default, clamps a value that does not fit, while *truncated* drops excess most significant bits for integers [3]. Constants are not transmitted [3].

> **类型系统。** 字段可以是 bool、2 到 64 位的有符号（intX）和无符号（uintX）整数、float16、float32、float64、嵌套结构，以及静态（[X]）或动态（[<X] 或 [<=X]）数组。特殊的 voidX 类型插入 1 到 64 位填充位用于对齐，@union 指令声明带标签的联合体，同一时刻只编码一个字段 [3]。两种转换模式控制值转为更窄表示时的处理：saturated（默认）把装不下的值钳位，truncated 则对整数丢弃多余的高位 [3]。常量不会被传输 [3]。

**Serialisation.** Serialised data are an ordered sequence of bit fields with no header or footer and no implicit alignment. The byte order is little-endian (least significant byte first), and bits are filled from the most significant to the least significant [3]. A dynamic array is normally preceded by a length field of ⌈log₂(X+1)⌉ bits, where X is the maximum number of items. If the array is the last field of the top-level structure and its items are at least 8 bits long, the length field is omitted and inferred from the transfer length, a technique called tail array optimisation (TAO) [3]. TAO is never used on CAN FD [7].

> **序列化。** 序列化数据是有序的位字段序列，没有头部或尾部，没有隐式对齐。字节序为小端（低字节在前），位从最高有效位到最低有效位填充 [3]。动态数组前通常有长度字段，宽度为 ⌈log₂(X+1)⌉ 位，其中 X 是最大元素数。如果数组是顶层结构的最后一个字段且元素至少 8 位，则省略长度字段，由传输长度推断，这一技术称为尾数组优化（TAO）[3]。CAN FD 上从不使用 TAO [7]。

**Compatibility and signatures.** Different nodes can only exchange a structure correctly if their definitions agree. DroneCAN therefore computes a 64-bit *data type signature* from a normalised definition (comments and constants removed, default cast modes made explicit, full names for nested types) with the CRC-64-WE function, a cyclic redundancy check that produces a 64-bit checksum; the computation is extended over the signatures of nested structures. Two data types are compatible if their names and signatures are equal [3]. Constant definitions are deliberately excluded from the signature, because including them would make types non-extensible [3]. An aggregate signature allows two whole sets of types to be compared at once [3].

> **兼容性与签名。** 不同节点只有定义一致才能正确交换结构。因此 DroneCAN 用 CRC-64-WE 函数（产生 64 位校验值的循环冗余校验），对规范化后的定义（去掉注释和常量、显式写出默认转换模式、嵌套类型使用全名）计算 64 位数据类型签名，计算会延伸覆盖嵌套结构的签名。两个数据类型名称和签名都相等时才兼容 [3]。常量定义被有意排除在签名之外，因为纳入会使类型无法扩展 [3]。聚合签名可一次比较两整套类型 [3]。

**Data type IDs.** Message and service types do not share a data type ID space, so a message type and a service type can have the same ID without conflict, and changing an ID does not affect compatibility [3]. The ID space is partitioned by convention. Message type IDs 0–19999 are for standard messages, 20000–20999 for vendor-specific messages and 21000–65535 are reserved. Service type IDs 0–99 are for standard services, 100–199 are reserved and 200–255 are for vendor-specific services [8]. The vendor-specific range is used in practice by projects that extend the standard; for example, the ArduPilot project defines messages in its own `ardupilot.*` namespace, such as `ardupilot.indication.SafetyState` (ID 20000) [10]. Because vendor ranges are prone to collisions, end users must be able to change the ID of any non-standard type on a node [3].

> **数据类型 ID。** 消息类型和服务类型不共享数据类型 ID 空间，因此消息类型和服务类型可以有相同的 ID 而不冲突，更改 ID 不影响兼容性 [3]。ID 空间按约定划分。消息类型 ID 0–19999 用于标准消息，20000–20999 用于厂商专用消息，21000–65535 保留。服务类型 ID 0–99 用于标准服务，100–199 保留，200–255 用于厂商专用服务 [8]。扩展标准的项目在实践中使用厂商专用范围；例如 ArduPilot 项目在自己的 ardupilot.* 命名空间中定义消息，如 ardupilot.indication.SafetyState（ID 20000）[10]。由于厂商范围容易冲突，最终用户必须能在节点上更改任何非标准类型的 ID [3]。

Table III lists a selection of the standard data types that are relevant to a typical UAV. The standard definitions use SI units and follow the right-handed X-forward, Y-right, Z-down body convention, and quaternions are ordered X, Y, Z, W [8].

> 表 III 列出一部分与典型无人机相关的标准数据类型。标准定义使用 SI 单位，遵循右手系的 X 向前、Y 向右、Z 向下的机体坐标约定，四元数顺序为 X、Y、Z、W [8]。

| Data type | Default ID | Purpose |
|---|---|---|
| `uavcan.protocol.NodeStatus` | 341 | Periodic status and presence of every node [9], [10] |
| `uavcan.protocol.GetNodeInfo` (service) | 1 | Node identification [9], [10] |
| `uavcan.equipment.esc.RawCommand` | 1030 | Up to 20 ESC setpoints, each normalised to [−8192, 8191] [10] |
| `uavcan.equipment.esc.Status` | 1034 | ESC error count, voltage, current, temperature, speed and index [10] |
| `uavcan.equipment.gnss.Fix2` | 1063 | GNSS navigation solution [10] |
| `uavcan.equipment.gnss.RTCMStream` | 1062 | Correction data for high-precision positioning [10] |
| `uavcan.equipment.range_sensor.Measurement` | 1050 | Rangefinder measurement [10] |
| `uavcan.equipment.air_data.TrueAirspeed` | 1020 | Airspeed sensor data [10] |
| `uavcan.equipment.power.BatteryInfo` | 1092 | Battery information [10] |

> | 数据类型 | 默认 ID | 用途 |
> |---|---|---|
> | `uavcan.protocol.NodeStatus` | 341 | 每个节点的周期性状态与存在性 [9]、[10] |
> | `uavcan.protocol.GetNodeInfo`（服务） | 1 | 节点标识信息 [9]、[10] |
> | `uavcan.equipment.esc.RawCommand` | 1030 | 最多 20 个电调设定值，各自归一化到 [−8192, 8191] [10] |
> | `uavcan.equipment.esc.Status` | 1034 | 电调错误计数、电压、电流、温度、转速和序号 [10] |
> | `uavcan.equipment.gnss.Fix2` | 1063 | GNSS 导航解 [10] |
> | `uavcan.equipment.gnss.RTCMStream` | 1062 | 高精度定位的差分修正数据 [10] |
> | `uavcan.equipment.range_sensor.Measurement` | 1050 | 测距仪测量值 [10] |
> | `uavcan.equipment.air_data.TrueAirspeed` | 1020 | 空速传感器数据 [10] |
> | `uavcan.equipment.power.BatteryInfo` | 1092 | 电池信息 [10] |

*Table III. Examples of standard data types and their default data type IDs.*

> *表 III. 标准数据类型及其默认数据类型 ID 示例。*

The ESC messages illustrate the difference from point-to-point wiring. A single `RawCommand` message carries up to 20 setpoints, and each ESC answers with its own `Status` message [10]. An autopilot can thus both command and monitor many ESCs over the same two wires, whereas a PWM output carries a command only (Section II-B).

> 电调消息说明了与点对点接线的差别。一条 RawCommand 消息最多携带 20 个设定值，每个电调用自己的 Status 消息回应 [10]。自驾仪因此可以在同样两根线上既指挥又监测许多电调，而 PWM 输出只能携带指令（见 II-B 节）。

### D. Transport Layer on CAN / CAN 上的传输层

**Transfers.** A *transfer* is an act of data transmission between nodes. A broadcast transfer is addressed to all nodes except the source, and a unicast transfer is addressed to one node. A message transfer is a broadcast transfer that carries a serialised message, and a service transfer is a unicast transfer that carries a request or a response; both can be single-frame or multi-frame [4]. All transfers have a payload (the application data), a data type ID, a priority and a transfer ID [4].

> **传输（transfer）。** 传输是节点之间的一次数据发送行为。广播传输发给除源以外的所有节点，单播传输发给一个节点。消息传输是携带序列化消息的广播传输，服务传输是携带请求或响应的单播传输；两者都可以是单帧或多帧 [4]。所有传输都有载荷（应用数据）、数据类型 ID、优先级和传输 ID [4]。

**Timing.** A message transmission should be aborted if it cannot be completed within 1 second. For services, the transmission should be aborted after 1 second, the client should stop waiting for a response after 1 second, and the server should be able to process any request in under 0.5 seconds, unless the application strongly requires different documented values [4].

> **时序。** 消息发送如果不能在 1 秒内完成，应中止。对服务，发送应在 1 秒后中止，客户端应在 1 秒后停止等待响应，服务器应能在 0.5 秒内处理任何请求，除非应用强烈要求使用另行记录的其他值 [4]。

**CAN identifier layout.** DroneCAN uses only the CAN 2.0B frame format with 29-bit identifiers, and it can share a bus with other protocols based on CAN 2.0A [4]. The 29 bits are used differently for the three transfer kinds, as summarised in Table IV [4].

> **CAN 标识符布局。** DroneCAN 只使用 CAN 2.0B 帧格式的 29 位标识符，并可与其他基于 CAN 2.0A 的协议共用总线 [4]。三类传输对这 29 位的用法不同，汇总于表 IV [4]。

| Transfer kind | Fields from bit 28 down to bit 0 (width in bits) |
|---|---|
| Message | priority (5), message type ID (16), service-not-message = 0 (1), source node ID (7) |
| Anonymous message | priority (5), discriminator (14), lower bits of message type ID (2), service-not-message = 0 (1), source node ID = 0 (7) |
| Service | priority (5), service type ID (8), request-not-response (1), destination node ID (7), service-not-message = 1 (1), source node ID (7) |

> | 传输类别 | 从第 28 位到第 0 位的字段（括号内为位宽） |
> |---|---|
> | 消息 | 优先级（5），消息类型 ID（16），service-not-message = 0（1），源节点 ID（7） |
> | 匿名消息 | 优先级（5），判别符 discriminator（14），消息类型 ID 低位（2），service-not-message = 0（1），源节点 ID = 0（7） |
> | 服务 | 优先级（5），服务类型 ID（8），request-not-response（1），目的节点 ID（7），service-not-message = 1（1），源节点 ID（7） |

*Table IV. Layout of the 29-bit CAN identifier for the three transfer kinds [4]. In each row the widths add up to 29 bits, so the priority occupies bits 28–24, the service-not-message bit is bit 7 and the source node ID is bits 6–0 (bit positions derived from the field order and widths in [4]).*

> *表 IV. 三类传输的 29 位 CAN 标识符布局 [4]。每行位宽之和为 29 位，因此优先级占第 28–24 位，service-not-message 位是第 7 位，源节点 ID 是第 6–0 位（位位置由 [4] 中的字段顺序和位宽推导）。*

The priority is the most significant field. Valid values range from 0 to 31 and 0 is the highest priority, which follows from CAN arbitration, where the lowest identifier wins (Section III-B). The priority value must be identical for all frames of a multi-frame transfer [4]. Among transfers with the same priority, the next field decides arbitration, which for messages is the message type ID. A lower message type ID therefore takes precedence over a higher one (a consequence of the layout and the arbitration rule).

> 优先级是最高位字段。有效值范围 0 到 31，0 为最高优先级，这源自 CAN 仲裁"标识符最小者获胜"（见 III-B 节）。多帧传输的所有帧优先级必须相同 [4]。优先级相同的传输之间，由下一个字段决定仲裁，对消息而言就是消息类型 ID。因此较小的消息类型 ID 优先于较大的（这是布局与仲裁规则的推论）。

*Worked example (illustrative values).* A node with node ID 42 publishes `NodeStatus` (type ID 341 = 0x155) at an illustrative priority of 16. The CAN ID is (16 << 24) | (341 << 8) | (0 << 7) | 42 = 0x1001552A. For a single-frame transfer with transfer ID 3, the tail byte described below is 1 1 0 00011 = 0xC3. The number 341 is 101010101 in binary, an alternating bit pattern that the specification points to as facilitating automatic bit-rate detection [9].

> *算例（示意数值）。* 节点 ID 为 42 的节点以示意优先级 16 发布 NodeStatus（类型 ID 341 = 0x155）。CAN ID 为 (16 << 24) | (341 << 8) | (0 << 7) | 42 = 0x1001552A。对于传输 ID 为 3 的单帧传输，下文所述的尾字节是 1 1 0 00011 = 0xC3。341 的二进制是 101010101，是规范指出的有利于自动比特率检测的交替位型 [9]。

**Anonymous messages and the discriminator.** CAN does not allow different nodes to transmit frames with different data under the same CAN ID. Regular transfers satisfy this because the identifier contains the source node ID, but anonymous messages all share source node ID 0. DroneCAN therefore inserts a 14-bit *discriminator* filled with random data, for instance 14 bits of the transfer CRC of the message content. Because the discriminator is short, the probability that several nodes emit frames with the same ID but different data is higher than 0.006 %, which the specification calls significant. The protocol therefore requires nodes to abort the transmission of all anonymous transfers immediately when an error is detected on the bus, to avoid a bus deadlock caused by automatic retransmission [4]. Anonymous message type IDs are limited to 0–3, because only the 2 lower bits of the type ID are available [4].

> **匿名消息与判别符。** CAN 不允许不同节点用同一个 CAN ID 发送不同数据的帧。普通传输满足这一点，因为标识符里含有源节点 ID；但匿名消息的源节点 ID 都是 0。因此 DroneCAN 插入一个填充随机数据的 14 位判别符，例如取消息内容传输 CRC 的 14 位。由于判别符很短，多个节点发出相同 ID 但数据不同的帧的概率高于 0.006%，规范称其为显著。协议因此要求节点在检测到总线错误时立即中止所有匿名传输，以避免自动重发造成总线死锁 [4]。匿名消息类型 ID 限制在 0–3，因为只有类型 ID 的低 2 位可用 [4]。

**Tail byte.** The last byte of the data field of every frame is a *tail byte*, which has the fields (from the most significant bit) start of transfer (1 bit), end of transfer (1 bit), toggle bit (1 bit) and transfer ID (5 bits) [4]. For single-frame transfers, start and end of transfer are 1 and the toggle bit is 0. For multi-frame transfers, start of transfer is 1 only in the first frame, end of transfer is 1 only in the last frame, and the toggle bit starts at 0 and alternates [4]. The transfer ID is 5 bits wide, so it takes values from 0 to 31 [4].

> **尾字节。** 每帧数据域的最后一个字节是尾字节，从最高位起各字段为：传输起始（1 位）、传输结束（1 位）、切换位（1 位）和传输 ID（5 位）[4]。单帧传输中，起始和结束都为 1，切换位为 0。多帧传输中，起始仅在第一帧为 1，结束仅在最后一帧为 1，切换位从 0 开始交替翻转 [4]。传输 ID 宽 5 位，取值 0 到 31 [4]。

**Transfer ID.** Every node keeps a transfer ID map from *transfer descriptors* (transfer type, data type ID, source node ID and, for unicast transfers, destination node ID) to a counter. The counter starts at zero for a new entry, is incremented for every transfer and rolls over to zero after its maximum. Entries unused for at least 2 seconds may be removed. A service response copies the transfer ID of its request [4].

> **传输 ID。** 每个节点维护一张传输 ID 映射表，从传输描述符（传输类型、数据类型 ID、源节点 ID，单播时还有目的节点 ID）映射到计数器。新条目计数器从零开始，每次传输加一，到最大值后回绕为零。至少 2 秒未用的条目可以删除。服务响应复制其请求的传输 ID [4]。

**Multi-frame transfers.** If the payload exceeds the space available in one frame, the sender computes a transfer CRC, prepends it to the payload and sends the data as a sequence of frames, all but the last of which must be completely filled. The frames should be pushed to the bus at once, in order [4]. The CRC algorithm is CRC-16-CCITT-FALSE (polynomial 0x1021, initial value 0xFFFF), and it is computed over the payload prepended with the data type signature in little-endian byte order [4]. The toggle bit alternates so that duplicated frames can be detected [4]. Single-frame transfers are more efficient in throughput and latency, and the specification advises avoiding multi-frame transfers where possible [4].

> **多帧传输。** 如果载荷超出一帧可用空间，发送方计算传输 CRC，将其置于载荷之前，并把数据分成一串帧发送，除最后一帧外都必须填满。这些帧应一次性按顺序推入总线 [4]。CRC 算法为 CRC-16-CCITT-FALSE（多项式 0x1021，初值 0xFFFF），是对前面加上数据类型签名（小端字节序）的载荷计算的 [4]。切换位交替翻转，以便检测重复帧 [4]。单帧传输在吞吐和延迟上更高效，规范建议尽量避免多帧传输 [4]。

With 8-byte classic frames, one byte is used by the tail byte, so a single frame carries up to 7 bytes of payload. A multi-frame transfer with an N-byte payload therefore needs ⌈(N + 2) / 7⌉ frames, because the 2-byte CRC is prepended. For example, a 40-byte payload needs ⌈42/7⌉ = 6 frames, whereas a 7-byte payload, such as the 56-bit `NodeStatus`, fits a single frame. These figures are derived from the tail-byte and CRC rules in [3], [4] and [9]; the payload sizes are examples.

> 8 字节的经典帧中，一个字节被尾字节占用，因此单帧最多携带 7 字节载荷。N 字节载荷的多帧传输因而需要 ⌈(N + 2) / 7⌉ 帧，因为要前置 2 字节 CRC。例如 40 字节载荷需要 ⌈42/7⌉ = 6 帧，而 7 字节载荷（如 56 位的 NodeStatus）装进单帧即可。这些数字由 [3]、[4] 和 [9] 中的尾字节与 CRC 规则推导而来，载荷大小只是示例。

**Reception and redundancy.** A receiver keeps a state for every transfer descriptor. It restarts reassembly when it sees a first frame with a suitable transfer ID, or when a transfer ID timeout of 2 seconds has elapsed, and it uses the *transfer ID forward distance* (in the range 0–31) and a half range of 16 to decide whether a new transfer ID is later or earlier than the last one [4]. In configurations with redundant CAN interfaces (several physical connections to separate buses), a node must transmit every outgoing transfer on all available interfaces simultaneously; an exception is a transfer whose payload depends on the interface, such as time synchronisation. The receiver accepts a transfer from another interface only after the previous transfer is more than T<sub>switch</sub> old, where T<sub>switch</sub> must not exceed 2 seconds [4]. Duplicates of the same transfer arriving through different interfaces are thus suppressed, and a failed interface is tolerated without an explicit failover protocol.

> **接收与冗余。** 接收方为每个传输描述符保持一个状态。看到带合适传输 ID 的首帧，或 2 秒的传输 ID 超时已过时，就重新开始重组；并用传输 ID 前向距离（范围 0–31）和半范围 16 来判断新传输 ID 比上一个更晚还是更早 [4]。在有冗余 CAN 接口（连到独立总线的多个物理连接）的配置中，节点必须在所有可用接口上同时发送每一个外发传输；载荷与接口相关的传输（如时间同步）除外。接收方只有在上一个传输已超过 T<sub>switch</sub> 之后才接受来自另一个接口的传输，T<sub>switch</sub> 不得超过 2 秒 [4]。这样，经不同接口到达的同一传输的重复副本被抑制，失效的接口无需显式的故障切换协议就能被容忍。

**Driver requirement.** Because all frames of a multi-frame transfer share one CAN ID, the CAN driver (the software that controls the CAN hardware) must transmit frames of identical ID in the order in which they entered the transmit queue. The specification warns that some hardware does not do this by default [4].

> **驱动要求。** 因为多帧传输的所有帧共享同一个 CAN ID，CAN 驱动（控制 CAN 硬件的软件）必须按相同 ID 的帧进入发送队列的顺序发送它们。规范警告说，有些硬件默认并不这样做 [4]。

### E. Alternative Transports / 备选传输

Three further transports are defined for the same transfer-level data model (Table V).

> 同一传输层数据模型还定义了另外三种传输（表 V）。

**CAN FD.** DroneCAN operates on CAN FD with a small number of changes from classical CAN: frames can have up to 64 bytes, TAO is never used, and the data length code is quantised [7]. The usual configuration is bit-rate switching with a nominal rate of 1 Mbit/s and a data phase of 1, 2, 4, 5 or 8 Mbit/s depending on hardware capability, and DLC values 9 to 15 map to 12, 16, 20, 24, 32, 48 and 64 bytes [7]. By the same tail-byte arithmetic as above, a single CAN FD frame carries up to 63 bytes of transfer payload (derived value). The specification recommends the `dronecan_dsdlc` compiler for bindings that work with both classical CAN and CAN FD [7].

> **CAN FD。** DroneCAN 在 CAN FD 上运行，与经典 CAN 相比有少量改动：帧最多 64 字节，从不使用 TAO，数据长度码被量化 [7]。通常配置为比特率切换，标称速率 1 Mbit/s，数据段按硬件能力取 1、2、4、5 或 8 Mbit/s，DLC 值 9 到 15 对应 12、16、20、24、32、48 和 64 字节 [7]。按上述同样的尾字节算法，单个 CAN FD 帧最多携带 63 字节传输载荷（推导值）。规范推荐用 dronecan_dsdlc 编译器生成同时适用于经典 CAN 和 CAN FD 的绑定 [7]。

**UDP.** DroneCAN can be carried over UDP (User Datagram Protocol, a lightweight connectionless internet protocol) using multicast packets, which are delivered to all members of a group of receivers. The default multicast address is 239.65.82.B, where B is the DroneCAN bus number, and the default port is 57732. A packet contains a 16-bit magic number 0x2934 (a constant that identifies the packet type), a CRC-16-CCITT, 16 bits of flags, a 32-bit message ID and the data, all little-endian. The URI (uniform resource identifier) form is `mcast:B` [5].

> **UDP。** DroneCAN 可通过 UDP（用户数据报协议，一种轻量的无连接互联网协议）以组播包传输，组播包会送达一组接收方的所有成员。默认组播地址是 239.65.82.B，其中 B 为 DroneCAN 总线号，默认端口 57732。数据包含 16 位魔数 0x2934（标识包类型的常量）、CRC-16-CCITT、16 位标志、32 位消息 ID 和数据，均为小端。URI（统一资源标识符）形式为 mcast:B [5]。

**MAVLink.** DroneCAN packets can be carried over MAVLink 2 using `CAN_FRAME` or `CANFD_FRAME` messages, whose data are identical to the corresponding CAN or CAN FD transport packets. The URI prefix is `mavcan:` followed by a standard MAVLink URI [6]. This *tunnelling* allows a ground station that speaks MAVLink to reach DroneCAN nodes through the flight controller, for example for configuration and firmware update.

> **MAVLink。** DroneCAN 数据包可用 CAN_FRAME 或 CANFD_FRAME 消息通过 MAVLink 2 传输，其数据与相应 CAN 或 CAN FD 传输的数据包相同。URI 前缀为 mavcan:，后接标准 MAVLink URI [6]。这种隧道传输使说 MAVLink 的地面站可以通过飞控访问 DroneCAN 节点，例如用于配置和固件更新。

| Transport | Frame / packet size | Addressing and framing | Source |
|---|---|---|---|
| CAN 2.0B | 8 bytes per frame | 29-bit CAN ID, tail byte | [4] |
| CAN FD | up to 64 bytes per frame | same fields, quantised DLC, no TAO | [7] |
| UDP | datagram | multicast 239.65.82.B:57732, magic number 0x2934, CRC-16 | [5] |
| MAVLink 2 | `CAN_FRAME` / `CANFD_FRAME` messages | prefix `mavcan:` | [6] |

> | 传输 | 帧/包大小 | 寻址与封装 | 来源 |
> |---|---|---|---|
> | CAN 2.0B | 每帧 8 字节 | 29 位 CAN ID，尾字节 | [4] |
> | CAN FD | 每帧最多 64 字节 | 字段相同，量化 DLC，无 TAO | [7] |
> | UDP | 数据报 | 组播 239.65.82.B:57732，魔数 0x2934，CRC-16 | [5] |
> | MAVLink 2 | `CAN_FRAME` / `CANFD_FRAME` 消息 | 前缀 `mavcan:` | [6] |

*Table V. Transports defined for DroneCAN.*

> *表 V. DroneCAN 定义的传输。*

### F. Standard Application-Level Functions / 标准应用层功能

DroneCAN does not require any specific initialisation: a node may begin working as soon as it is powered up. The only mandatory application-level function is the periodic broadcasting of the node status message, and everything else is optional [9].

> DroneCAN 不要求任何特定初始化：节点上电后即可开始工作。唯一强制的应用层功能是周期性广播节点状态消息，其余都是可选的 [9]。

**Node status.** Every node broadcasts `uavcan.protocol.NodeStatus` (ID 341), the only data structure that all nodes are required to support [9]. Its fields are the uptime in seconds (32 bits), a health value (2 bits), a mode (3 bits), a reserved sub-mode (3 bits) and a vendor-specific status code (16 bits), which makes 56 bits or 7 bytes in total (the sum of the field widths in [9]). Health takes the values OK (0), WARNING (1), ERROR (2) and CRITICAL (3). Modes include OPERATIONAL, INITIALIZATION, MAINTENANCE, SOFTWARE_UPDATE and OFFLINE. A node may declare itself OFFLINE before it shuts down, so that others need not wait for a timeout [9]. The publication period may vary between 2 ms and 1000 ms, and a node that fails to publish within 3000 ms is considered offline [9]. Because every working node publishes this message at least once per second, the status stream also provides the vehicle-wide health overview that the specification mentions [1], [9].

> **节点状态。** 每个节点广播 uavcan.protocol.NodeStatus（ID 341），这是唯一要求所有节点都支持的数据结构 [9]。其字段为：运行时间（秒，32 位）、健康值（2 位）、模式（3 位）、保留的子模式（3 位）和厂商专用状态码（16 位），合计 56 位即 7 字节（字段位宽之和，见 [9]）。健康值取 OK（0）、WARNING（1）、ERROR（2）和 CRITICAL（3）。模式包括 OPERATIONAL、INITIALIZATION、MAINTENANCE、SOFTWARE_UPDATE 和 OFFLINE。节点可在关机前自行声明 OFFLINE，使其他节点无需等待超时 [9]。发布周期可在 2 ms 到 1000 ms 之间变化，3000 ms 内未发布的节点被视为离线 [9]。由于每个正常工作的节点每秒至少发布一次该消息，状态流也就提供了规范所提到的全飞行器健康概览 [1]、[9]。

**Node discovery.** A list of nodes can be built by listening for `NodeStatus`, and detailed information is available through the `GetNodeInfo` service, whose response contains the node status, the software version, the hardware version and a node name. Support for `GetNodeInfo` is highly recommended in every node [9]. The name is a reversed domain name, similar to a Java package name [9]. A `GetDataTypeInfo` service reveals whether a node knows a data type, and whether it publishes, subscribes to or serves it [9].

> **节点发现。** 监听 NodeStatus 即可建立节点列表，详细信息则通过 GetNodeInfo 服务获得，其响应包含节点状态、软件版本、硬件版本和节点名称。强烈建议每个节点都支持 GetNodeInfo [9]。名称是反向域名，类似 Java 包名 [9]。GetDataTypeInfo 服务可揭示节点是否认识某数据类型，以及它发布、订阅还是提供该类型 [9]。

**Time synchronisation.** DroneCAN supports network-wide time synchronisation with a resolution of up to one CAN bit period (1 µs at 1 Mbit/s) if the hardware timestamps CAN frames, and it still works, with degraded performance, without this support. The algorithm follows the work of Gergeleit and Streich: a *master* periodically broadcasts `GlobalTimeSync`, which contains the exact timestamp of its *previous* transmission, and *slaves* use it to estimate their phase difference to the master. The specification leaves the clock adjustment algorithm to the implementation. If several masters exist, only the master with the lowest node ID is active, and the slaves move to the next master if the active one is silent for T<sub>timeout</sub> [9]. The broadcast period is 40 ms to 1100 ms and the recommended broadcaster timeout is 2200 ms [9].

> **时间同步。** 若硬件对 CAN 帧打时间戳，DroneCAN 支持分辨率达一个 CAN 位周期（1 Mbit/s 时为 1 µs）的全网时间同步；没有这一支持也能工作，只是性能下降。算法沿用 Gergeleit 和 Streich 的工作：主节点周期性广播 GlobalTimeSync，其中含有它上一次发送的精确时间戳，从节点据此估算自己与主节点的相位差。时钟调整算法留给实现自行决定。若存在多个主节点，只有节点 ID 最小者处于活动状态，当前活动主节点沉默 T<sub>timeout</sub> 之后，从节点切换到下一个主节点 [9]。广播周期为 40 ms 到 1100 ms，推荐的广播方超时为 2200 ms [9]。

**Configuration.** The services `param.GetSet`, `param.ExecuteOpcode` (save to non-volatile memory, or erase) and `RestartNode` manage remote configuration. They are recommended but not mandatory [9]. A naming convention with the prefix `uavcan.` covers common parameters: `uavcan.node_id` (zero means unconfigured, which may prompt dynamic allocation), `uavcan.bit_rate` (zero should trigger automatic bit-rate detection), and `uavcan.dtid-X`, `uavcan.pubp-X`, `uavcan.prio-X` and `uavcan.id-X-Y` for data type IDs, publication periods, priorities and instance IDs [9]. A node with more complex configuration needs should use configuration files through the file services [9].

> **配置。** param.GetSet、param.ExecuteOpcode（保存到非易失存储或擦除）和 RestartNode 服务管理远程配置，建议实现但不强制 [9]。以 uavcan. 为前缀的命名约定涵盖常用参数：uavcan.node_id（零表示未配置，可能触发动态分配）、uavcan.bit_rate（零应触发自动比特率检测），以及 uavcan.dtid-X、uavcan.pubp-X、uavcan.prio-X 和 uavcan.id-X-Y，分别用于数据类型 ID、发布周期、优先级和实例 ID [9]。配置需求更复杂的节点应通过文件服务使用配置文件 [9]。

**File transfer and firmware update.** A set of services (`GetInfo`, `GetDirectoryEntryInfo`, `Delete`, `Read`, `Write`) gives access to the file system of a remote node. Firmware update is a special case: the *updater* calls `BeginFirmwareUpdate` on the *updatee*, which, if it accepts, retrieves the firmware image from a file server, usually the updater itself, through `file.Read`, sets its mode to SOFTWARE_UPDATE, and restarts (Fig. 3). The updatee typically then performs dynamic node ID allocation again [9].

> **文件传输与固件更新。** 一组服务（GetInfo、GetDirectoryEntryInfo、Delete、Read、Write）提供对远程节点文件系统的访问。固件更新是一个特例：更新方对被更新方调用 BeginFirmwareUpdate，被更新方如接受，就通过 file.Read 从文件服务器（通常就是更新方自己）取回固件镜像，把模式设为 SOFTWARE_UPDATE 并重启（图 3）。被更新方随后通常会再次执行动态节点 ID 分配 [9]。

![Fig. 3](Figuras/Figura_3.svg)

*Fig. 3. Firmware update over DroneCAN [9].*

> *图 3. DroneCAN 上的固件更新 [9]。*

**Debugging and emergency functions.** `debug.LogMessage` and `debug.KeyValue` report events and named values. `AccessCommandShell` offers a remote command shell, and its definition is marked in the specification as subject to change. The `Panic` message lets a node announce an unrecoverable fault. Receivers should wait for at least 3 messages sent at intervals no longer than 500 ms before they take emergency action, and the reason is a text of at most 7 bytes that fits a single frame [9].

> **调试与紧急功能。** debug.LogMessage 和 debug.KeyValue 报告事件和命名值。AccessCommandShell 提供远程命令行，规范中标注其定义可能变动。Panic 消息使节点能宣告不可恢复故障。接收方应至少等待 3 条间隔不超过 500 ms 的消息之后再采取紧急措施，原因文本最多 7 字节，装得进单帧 [9]。

**Dynamic node ID allocation.** Because a node needs a unique node ID, DroneCAN defines an allocation procedure that, combined with automatic bit-rate detection, makes *plug-and-play* nodes possible (nodes that join the network without manual configuration). A dynamically allocated ID is not persistent: the node must allocate again at every start [9]. The procedure involves *allocators*, which serve requests and are normally static nodes, and *allocatees*, each of which has a globally unique 128-bit identifier, the same value that it reports in `GetNodeInfo`. The allocators keep an *allocation table* that maps unique IDs to node IDs; it can only grow, unless it is wiped completely [9].

> **动态节点 ID 分配。** 因为节点需要唯一的节点 ID，DroneCAN 定义了分配流程，与自动比特率检测结合后可实现即插即用节点（无需手动配置就能加入网络的节点）。动态分配的 ID 不是持久的：节点每次启动都必须重新分配 [9]。该流程涉及分配器（处理请求，通常是静态节点）和被分配方（每个都有全球唯一的 128 位标识，与它在 GetNodeInfo 中报告的值相同）。分配器维护一张分配表，把唯一 ID 映射到节点 ID；它只能增长，除非被整体清除 [9]。

Allocatees use only anonymous single-frame transfers of the message `uavcan.protocol.dynamic_node_id.Allocation`. On classic CAN they therefore send their unique ID in three stages, and the allocator echoes the part received so far in each reply, as shown in Fig. 4. CAN FD frames can deliver the unique ID in one stage [9]. Request intervals are randomised to resolve collisions, similar to CSMA/CD (carrier-sense multiple access with collision detection) [9]. The allocator can use multi-frame transfers, because it has a node ID [9].

> 被分配方只使用消息 uavcan.protocol.dynamic_node_id.Allocation 的匿名单帧传输。因此在经典 CAN 上，它们分三个阶段发送自己的唯一 ID，分配器在每次回复中回显到目前为止收到的部分，如图 4 所示。CAN FD 帧可以一次性送出唯一 ID [9]。请求间隔是随机化的以解决冲突，类似 CSMA/CD（载波侦听多路访问/冲突检测）[9]。分配器可以使用多帧传输，因为它有节点 ID [9]。

![Fig. 4](Figuras/Figura_4.svg)

*Fig. 4. Dynamic node ID allocation on classical CAN, simplified from [9].*

> *图 4. 经典 CAN 上的动态节点 ID 分配，据 [9] 简化。*

The network can have zero, one, three or five allocators. With one allocator, allocation stops if it fails. With three or five allocators, the allocators replicate the allocation table using the Raft consensus algorithm of Ongaro and Ousterhout (a method by which several computers agree on a shared state even if some of them fail), so that the network tolerates the loss of one or two allocators, respectively, and continues to serve requests [9]. Only the elected leader processes allocation requests. The Raft log is the allocation table itself, and the default election timeout is randomised between 2000 ms and 4000 ms [9]. The specification advises against mixing dynamic and static nodes, except for the allocators, and gives rules for safe coexistence [9].

> 网络可以有零个、一个、三个或五个分配器。只有一个分配器时，它一旦失效分配就停止。有三个或五个分配器时，它们用 Ongaro 和 Ousterhout 的 Raft 共识算法（让多台计算机即使部分失效也能就共享状态达成一致的方法）复制分配表，因此网络分别能容忍丢失一个或两个分配器并继续服务请求 [9]。只有选出的领导者处理分配请求。Raft 日志就是分配表本身，默认选举超时在 2000 ms 到 4000 ms 之间随机化 [9]。规范建议不要混用动态节点和静态节点（分配器除外），并给出了安全共存的规则 [9]。

## VI. RELIABILITY AND SAFETY MECHANISMS / 可靠性与安全机制

DroneCAN is described as providing "a highly reliable communication method" [2], yet the specification does not reduce this claim to a single feature. Reliability emerges from mechanisms at several layers, and each layer defends against a different kind of fault. This section analyses them from the wire upwards and then discusses where the protection ends. The analysis is qualitative and is based on the specification and the cited literature.

> DroneCAN 被描述为提供"高可靠的通信方法"[2]，但规范并没有把这一主张归结为某一项特性。可靠性来自多个层次的机制，每一层防御不同类型的故障。本章自下而上分析这些机制，再讨论保护止于何处。分析是定性的，依据规范和所引文献。

### A. Overview / 概览

Table VI maps the mechanisms to the faults they address.

> 表 VI 把机制与其所应对的故障对应起来。

| Layer | Mechanism | Fault addressed | Source |
|---|---|---|---|
| Physical | Differential signalling, twisted pair, termination | Electrical noise, signal reflections | [11], [19], [20] |
| Physical | Recommended bit rates, sample points, bus and stub lengths | Timing violations on long or stubbed buses | [11] |
| Physical | Overcurrent protection, power isolation between interfaces, parallel connectors | Short circuit, loss of bus power, connector faults | [11] |
| Link | Bit-wise arbitration with priority in the identifier | Collisions, starvation of urgent messages | [4], [19], [20] |
| Link | Frame CRC, acknowledgement, bit stuffing, error frames | Corrupted frames | [20] |
| Link | Error counters, error-passive and bus-off states | Persistently faulty node | [20] |
| Transport | Transfer CRC over signature and payload, toggle bit, transfer ID, timeouts | Corrupted or duplicated multi-frame transfers, incompatible types, lost frames | [3], [4] |
| Transport | Abort of anonymous transfers on bus error | Deadlock caused by identifier collisions | [4] |
| System | Redundant interfaces (up to three), simultaneous transmission | Failure of one bus | [4], [11] |
| System | Redundant nodes distinguished by node ID | Failure of one sensor or computer | [2] |
| Application | Mandatory `NodeStatus`, health code, offline timeout | Silent failure of a node | [9] |
| Application | Redundant allocators with Raft consensus | Failure of the node ID allocator | [9] |
| Application | `Panic` message, time-sync master fail-over | Unrecoverable system fault, loss of the time reference | [9] |

> | 层次 | 机制 | 应对的故障 | 来源 |
> |---|---|---|---|
> | 物理层 | 差分信号、双绞线、终端匹配 | 电气噪声、信号反射 | [11]、[19]、[20] |
> | 物理层 | 推荐的比特率、采样点、总线与支线长度 | 长总线或带支线总线上的时序违例 | [11] |
> | 物理层 | 过流保护、接口间电源隔离、并联连接器 | 短路、总线供电丢失、连接器故障 | [11] |
> | 链路层 | 逐位仲裁，优先级在标识符中 | 冲突、紧急消息被饿死 | [4]、[19]、[20] |
> | 链路层 | 帧 CRC、应答、位填充、错误帧 | 帧损坏 | [20] |
> | 链路层 | 错误计数器、错误被动与总线关闭状态 | 持续故障的节点 | [20] |
> | 传输层 | 覆盖签名与载荷的传输 CRC、切换位、传输 ID、超时 | 多帧传输损坏或重复、类型不兼容、丢帧 | [3]、[4] |
> | 传输层 | 总线错误时中止匿名传输 | 标识符冲突造成的死锁 | [4] |
> | 系统层 | 冗余接口（最多三个），同时发送 | 一条总线失效 | [4]、[11] |
> | 系统层 | 靠节点 ID 区分的冗余节点 | 一个传感器或计算机失效 | [2] |
> | 应用层 | 强制的 `NodeStatus`、健康码、离线超时 | 节点无声失效 | [9] |
> | 应用层 | 带 Raft 共识的冗余分配器 | 节点 ID 分配器失效 | [9] |
> | 应用层 | `Panic` 消息、时间同步主节点切换 | 不可恢复的系统故障、时间基准丢失 | [9] |

*Table VI. Reliability mechanisms of a DroneCAN system by layer.*

> *表 VI. DroneCAN 系统按层次划分的可靠性机制。*

### B. Physical Layer and Hardware Recommendations / 物理层与硬件建议

The CAN physical layer uses differential signalling on a twisted pair (two wires twisted around each other), which suppresses noise that affects both wires equally, and a bus that is terminated at both ends with about 120 Ω [19], [20]. DroneCAN builds on ISO 11898-2 high-speed CAN for all of its connector types [11]. Chapter 8 of the specification, titled "Hardware design recommendations", adds the following rules that directly target reliability [11].

> CAN 物理层在双绞线（两根相互缠绕的线）上使用差分信号，抑制同时影响两线的噪声，总线两端各用约 120 Ω 终接 [19]、[20]。DroneCAN 的所有连接器类型都建立在 ISO 11898-2 高速 CAN 之上 [11]。规范第 8 章"硬件设计建议"增加了下列直接面向可靠性的规则 [11]。

*Connectors.* Three connector types are defined: a D-Sub DE-9 connector for maximum compatibility with commercial off-the-shelf parts, a circular M8 B-coded 5-pin connector, and a JST GH 4-pin Micro connector. The M8 connector is the default choice. The Micro connector is intended for small and weight-sensitive systems, has a secure positive lock but no shielding, and is explicitly described as not suitable for safety-critical hardware [11]. It is highly recommended to provide two identical parallel connectors per CAN interface so that a device can be daisy-chained (connected in series) without T-connectors, because T-connectors add an extra point of failure, increase stub length (the length of a branch off the main bus) and weight, and complicate harnesses [11].

> *连接器。* 定义了三种连接器：为最大限度兼容商用现成部件的 D-Sub DE-9、圆形 M8 B 编码 5 针连接器，以及 JST GH 4 针 Micro 连接器。M8 是默认选择。Micro 连接器面向小型、对重量敏感的系统，有可靠的正向锁定但没有屏蔽，规范明确说它不适用于安全关键硬件 [11]。强烈建议每个 CAN 接口提供两个相同的并联连接器，使设备可以菊花链连接（串联）而不需要 T 型连接器，因为 T 型连接器增加一个故障点，增加支线长度（总线分出的分支长度）和重量，并使线束复杂化 [11]。

*Bus power.* Bus-powered devices must protect each power input with an overcurrent protection circuit such as a fuse, so that a short circuit in one device does not bring down the bus supply. Devices with redundant interfaces must prevent direct current flow between the power inputs of different interfaces, and a power source must shut down a failing bus while continuing to supply the remaining ones [11]. The nominal supply is 24 V on the D-Sub and M8 connectors, with at most 500 mA recommended per bus-powered device, and about 5 V on the Micro connector [11].

> *总线供电。* 由总线供电的设备必须用保险丝等过流保护电路保护每个电源输入，使一个设备的短路不会拖垮总线供电。带冗余接口的设备必须防止不同接口电源输入之间有直接电流流动，电源必须切断失效的总线并继续给其余总线供电 [11]。D-Sub 和 M8 连接器的标称供电为 24 V，每个总线供电设备建议最大 500 mA，Micro 连接器约 5 V [11]。

*Bit rates and bus length.* DroneCAN is bit-rate agnostic in principle, but only the recommended bit rates should be used for compatibility (Table VII). All entries use a stub length of at most 0.3 m, and the bus-length estimates assume that the propagation delay does not exceed 5 ns/m, excluding transceivers [11]. The frame durations in Table VII are derived values and are not taken from the specification: a classical CAN frame with an extended identifier and 8 data bytes has 131 bit times without stuffing, including the 3-bit inter-frame space. A worst-case count of stuff bits for the 118 stuffable bits gives 29 additional bits, so that at most 160 bit times are needed (118 + 29 + 13 trailing bits).

> *比特率与总线长度。* DroneCAN 原则上对比特率没有限定，但为兼容起见只应使用推荐比特率（表 VII）。所有条目的支线长度至多 0.3 m，总线长度估算假设传播延迟不超过 5 ns/m（不含收发器）[11]。表 VII 中的帧时长是推导值，并非取自规范：带扩展标识符和 8 字节数据的经典 CAN 帧，不填充时为 131 个位时间（含 3 位帧间间隔）。对 118 个可被填充的位按最坏情况统计填充位，多出 29 位，因此最多需要 160 个位时间（118 + 29 + 13 个尾部位）。

| Bit rate | Sample point (recommended) | Max. bus length [11] | Max. stub length [11] | 8-byte frame, no stuffing (derived) | 8-byte frame, worst-case stuffing (derived) |
|---|---|---|---|---|---|
| 1000 kbit/s | 75–90 % (87.5 %) | 40 m | 0.3 m | 131 µs | 160 µs |
| 500 kbit/s | 85–90 % (87.5 %) | 100 m | 0.3 m | 262 µs | 320 µs |
| 250 kbit/s | 85–90 % (87.5 %) | 250 m | 0.3 m | 524 µs | 640 µs |
| 125 kbit/s | 85–90 % (87.5 %) | 500 m | 0.3 m | 1048 µs | 1280 µs |

> | 比特率 | 采样点（推荐） | 最大总线长度 [11] | 最大支线长度 [11] | 8 字节帧，无填充（推导） | 8 字节帧，最坏填充（推导） |
> |---|---|---|---|---|---|
> | 1000 kbit/s | 75–90 %（87.5 %） | 40 m | 0.3 m | 131 µs | 160 µs |
> | 500 kbit/s | 85–90 %（87.5 %） | 100 m | 0.3 m | 262 µs | 320 µs |
> | 250 kbit/s | 85–90 %（87.5 %） | 250 m | 0.3 m | 524 µs | 640 µs |
> | 125 kbit/s | 85–90 %（87.5 %） | 500 m | 0.3 m | 1048 µs | 1280 µs |

*Table VII. Recommended bit rates and physical parameters [11], with derived frame durations for a 29-bit identifier frame with 8 data bytes. The sample point is the position within a bit period at which the receiver reads the bit value.*

> *表 VII. 推荐比特率与物理参数 [11]，以及 29 位标识符、8 字节数据帧的推导帧时长。采样点是接收方在一个位周期内读取位值的位置。*

These figures allow a rough capacity estimate. At 1 Mbit/s no more than 1 s / 160 µs = 6250 frames per second fit on the bus in the worst case, which carries at most 6250 × 7 bytes = 43.75 kB/s of transfer payload (an upper bound that ignores arbitration losses and is not a measured throughput). A `NodeStatus` message from each of 20 nodes at 1 Hz needs at most 20 × 160 = 3200 bit/s, or 0.32 % of the bus at 1 Mbit/s. Status traffic is therefore small compared with the bus capacity, which remains available for sensor and actuator data. The same calculation shows why DroneCAN recommends avoiding multi-frame transfers where possible [4], since a multi-frame transfer occupies the bus for several frame times.

> 这些数字可以做粗略的容量估算。在 1 Mbit/s 下，最坏情况每秒最多放得下 1 s / 160 µs = 6250 帧，最多携带 6250 × 7 字节 = 43.75 kB/s 的传输载荷（这是忽略仲裁损失的上界，不是实测吞吐）。20 个节点各以 1 Hz 发一条 NodeStatus，最多需要 20 × 160 = 3200 bit/s，占 1 Mbit/s 总线的 0.32 %。因此状态流量相对总线容量很小，容量可留给传感器和执行器数据。同样的计算也说明了为什么 DroneCAN 建议尽量避免多帧传输 [4]：一次多帧传输会占用总线好几个帧时间。

*Automatic bit-rate detection.* Designers are encouraged to implement automatic bit-rate detection according to CiA 801. DroneCAN makes the simple approach of measuring bit times feasible, because any functioning network exchanges `NodeStatus` at a rate of at least 1 Hz and the CAN ID of that message contains an alternating bit pattern [11]. Together with dynamic node ID allocation this permits plug-and-play nodes [9].

> *自动比特率检测。* 鼓励设计者按 CiA 801 实现自动比特率检测。DroneCAN 使"测量位时间"这种简单做法成为可行，因为任何正常运行的网络都会以至少 1 Hz 的速率交换 NodeStatus，且该消息的 CAN ID 含有交替位型 [11]。与动态节点 ID 分配结合，可实现即插即用节点 [9]。

### C. Link Layer / 链路层

At the link layer DroneCAN inherits the mechanisms of CAN (Section III). *Arbitration* gives deterministic precedence to the most urgent frames and, because DroneCAN places a 5-bit priority at the top of the identifier, the application can decide which transfers pre-empt others [4]. *Error detection* relies on the CRC, the acknowledgement bits and end-of-frame bits, and the bit-stuffing rule, which also keeps the receiver synchronised [20]. *Fault confinement* is provided by the transmit and receive error counters, which move a persistently failing node into the error-passive state and finally the bus-off state, so that it can no longer disturb the bus [20]. The error-confinement mechanism has a drawback: it protects the bus against a failing node, but Bozdal et al. point out that it can also be abused to force a healthy node into the bus-off state [20].

> 在链路层，DroneCAN 继承 CAN 的机制（第 III 章）。仲裁使最紧急的帧获得确定性的优先，且由于 DroneCAN 把 5 位优先级放在标识符顶部，应用可以决定哪些传输抢占其他传输 [4]。错误检测依赖 CRC、应答位和帧结束位，以及同时让接收方保持同步的位填充规则 [20]。故障隔离由发送和接收错误计数器提供，它们把持续失效的节点推入错误被动状态并最终进入总线关闭状态，使其不能再干扰总线 [20]。错误限制机制有一个缺点：它保护总线免受失效节点影响，但 Bozdal 等人指出它也可能被滥用，迫使正常节点进入总线关闭状态 [20]。

### D. Transport Layer / 传输层

Several rules in the transport layer complement the link layer.

> 传输层中有若干规则对链路层形成补充。

1. **Integrity of multi-frame transfers.** A CRC-16 protects the reassembled payload, computed over the payload prefixed with the data type signature [4]. As a consequence of this construction, a receiver whose data type definition differs from the sender's computes a different CRC for a multi-frame transfer and rejects it. The signature itself is not transmitted. Because the transfer CRC is prepended only to multi-frame transfers [4], the signature check does not apply to single-frame transfers, so that a type mismatch on a single-frame message is not caught by the transport layer. This follows from the specification text, and it makes careful management of data type IDs and versions more important, as discussed in Section V-C.  
   **多帧传输的完整性。** 一个 CRC-16 保护重组后的载荷，它是对前置了数据类型签名的载荷计算的 [4]。这种构造的结果是：数据类型定义与发送方不同的接收方，对多帧传输会算出不同的 CRC 而拒收。签名本身并不传输。由于传输 CRC 只前置在多帧传输上 [4]，签名检查不适用于单帧传输，因此单帧消息上的类型不匹配不会被传输层发现。这是从规范文字得出的结论，它使得对数据类型 ID 和版本的细致管理更加重要，见 V-C 节的讨论。
2. **Ordering and duplicates.** The toggle bit and the 5-bit transfer ID allow receivers to detect duplicated or missing frames and to tell old transfers from new ones [4]. A 2 s transfer ID timeout lets a receiver resynchronise after a long silence [4]. With only 32 transfer IDs, a very long burst of lost transfers could in principle make an old transfer ID look new, which explains the half-range comparison and the timeout.  
   **顺序与重复。** 切换位和 5 位传输 ID 使接收方能检测重复或缺失的帧，并区分新旧传输 [4]。2 秒的传输 ID 超时使接收方在长时间沉默后能重新同步 [4]。传输 ID 只有 32 个，一长串连续丢失的传输理论上会使旧传输 ID 看起来像新的，这就是采用半范围比较和超时的原因。
3. **Bounded waiting.** Transmissions are aborted after 1 s, clients stop waiting for a service response after 1 s, and servers should answer within 0.5 s [4]. These bounds prevent a failed server from blocking a client indefinitely.  
   **有限等待。** 发送在 1 秒后中止，客户端在 1 秒后停止等待服务响应，服务器应在 0.5 秒内回答 [4]。这些界限防止失效的服务器无限期阻塞客户端。
4. **Collision handling for anonymous transfers.** Nodes must abort anonymous transfers immediately when a bus error occurs, to avoid a deadlock caused by automatic retransmission of frames with the same ID but different data [4].  
   **匿名传输的冲突处理。** 节点必须在总线出错时立即中止匿名传输，以避免相同 ID、不同数据的帧被自动重发造成的死锁 [4]。
5. **Driver ordering.** The CAN controller driver must preserve the order of frames with identical IDs [4], which makes correct driver implementation part of the reliability argument.  
   **驱动顺序。** CAN 控制器驱动必须保持相同 ID 帧的顺序 [4]，这使正确的驱动实现成为可靠性论证的一部分。

### E. Redundancy / 冗余

Redundancy is a central design goal [1] and is addressed at three levels.

> 冗余是核心设计目标 [1]，在三个层面上实现。

*Redundant interfaces.* A node can have two or three CAN interfaces, and DroneCAN supports up to three redundant buses [11]. A node must transmit each outgoing transfer on all interfaces simultaneously, and the receiver removes duplicates as described in Section V-D [4]. Because mission-critical and non-critical devices often share a network, the specification gives a topology rule (Fig. 5): each bus is assigned a level of importance (primary or backup), all devices connect to the primary bus, and only devices with redundant interfaces additionally connect to the backup buses [11]. One redundant interface may also be left unused [11]. The PX4 documentation notes that both DroneCAN and Cyphal can use a second, redundant CAN interface [13].

> *冗余接口。* 节点可以有两个或三个 CAN 接口，DroneCAN 最多支持三条冗余总线 [11]。节点必须在所有接口上同时发送每个外发传输，接收方按 V-D 节所述去除重复 [4]。由于关键任务设备和非关键设备常共享一个网络，规范给出了拓扑规则（图 5）：为每条总线指定重要等级（主或备），所有设备连到主总线，只有带冗余接口的设备才额外连到备用总线 [11]。一个冗余接口也可以闲置不用 [11]。PX4 文档指出，DroneCAN 和 Cyphal 都可以使用第二个冗余 CAN 接口 [13]。

![Fig. 5](Figuras/Figura_5.svg)

*Fig. 5. Primary and backup bus topology with devices that have different numbers of interfaces, following the rules in [11].*

> *图 5. 设备接口数不同时的主/备总线拓扑，遵循 [11] 中的规则。*

*Redundant nodes.* Identical sensors can coexist, because every node has its own node ID, and the combination of data type ID and node ID identifies the publisher [2]. How the consumer selects or fuses redundant data is application logic that DroneCAN does not prescribe.

> *冗余节点。* 相同的传感器可以共存，因为每个节点有自己的节点 ID，"数据类型 ID + 节点 ID"的组合标识发布者 [2]。消费者如何选择或融合冗余数据是应用逻辑，DroneCAN 并不规定。

*Redundant services.* The dynamic node ID service can be run with three or five allocators using Raft, tolerating one or two allocator failures [9]. The time-synchronisation function allows several masters, with automatic switch-over to the next master after a timeout [9]. These features remove two potential single points of failure that exist in simple master–slave designs.

> *冗余服务。* 动态节点 ID 服务可以用 Raft 运行三个或五个分配器，容忍一个或两个分配器失效 [9]。时间同步功能允许多个主节点，超时后自动切换到下一个主节点 [9]。这些特性消除了简单主从设计中存在的两个潜在单点故障。

### F. Health Monitoring and Emergency Handling / 健康监测与紧急处理

Because every node must publish `NodeStatus`, any consumer can detect the loss of a node: a node that is silent for 3000 ms is considered offline, and a node that intends to shut down can announce this with the OFFLINE mode, which removes the delay [9]. The health field (OK, WARNING, ERROR, CRITICAL) and the mode (for example SOFTWARE_UPDATE or MAINTENANCE) allow consumers to decide whether to trust the data from a node [9]. An uptime counter that goes backwards signals a restart [9]. The `Panic` message provides a network-wide emergency signal, which receivers should act on only after several repeated messages, so that a single corrupted or spurious message does not trigger an emergency action [9]. These functions provide the *detection* of failures, while the reaction is left to the application, for example to the failsafe logic of the autopilot [18].

> 由于每个节点都必须发布 NodeStatus，任何消费者都能检测到节点丢失：沉默 3000 ms 的节点被视为离线，打算关机的节点可以用 OFFLINE 模式宣告，从而消除延迟 [9]。健康字段（OK、WARNING、ERROR、CRITICAL）和模式（例如 SOFTWARE_UPDATE 或 MAINTENANCE）使消费者能判断是否信任某个节点的数据 [9]。运行时间计数器倒退表示发生了重启 [9]。Panic 消息提供全网紧急信号，接收方应在收到数条重复消息之后才采取行动，这样单条损坏或虚假的消息不会触发紧急动作 [9]。这些功能提供故障的检测，而反应留给应用，例如自驾仪的失效保护逻辑 [18]。

### G. Limits of the Protection / 保护的边界

**No authentication or encryption.** The CAN weaknesses summarised in Section III-E apply to DroneCAN. Every node can listen to all traffic and transmit frames with any identifier. The transport-layer chapters of the specification [4] define no fields for authentication or encryption, and the node ID in the identifier is a claim that is not verified. The review of Ghamari et al. lists UAVCAN together with MAVLink among common UAV protocols that are vulnerable to eavesdropping, man-in-the-middle attacks (an attacker intercepting and possibly altering the communication between two parties), packet injection and denial-of-service attacks [17]. By comparison, MAVLink 2 offers an optional 13-byte signature intended to authenticate messages [16]. Kim et al. recently proposed a protocol-aware intrusion detection system (IDS) for UAVCAN networks [21]. It reassembles UAVCAN transfers and extracts ten features from payload data and from violations of the transfer rules in CAN IDs and tail bytes. On ten attack scenarios from a public dataset, the best configuration reached an average accuracy, precision and recall of 99.89 %, 99.96 % and 99.88 %, and the transfer-rule features were especially effective for replay attacks (re-sending previously recorded valid messages) [21]. These numbers come from a single study on one dataset, but they indicate that the transport-layer fields (CAN ID and tail byte) are a useful source of detection features and that detection, rather than prevention, is the approach pursued in this study.

> **没有认证或加密。** III-E 节总结的 CAN 弱点同样适用于 DroneCAN。每个节点都能监听所有流量，并用任意标识符发送帧。规范的传输层章节 [4] 没有定义认证或加密字段，标识符中的节点 ID 只是一个未经验证的声明。Ghamari 等人的综述把 UAVCAN 与 MAVLink 一起列为易受窃听、中间人攻击（攻击者截获并可能篡改双方之间的通信）、包注入和拒绝服务攻击的常见无人机协议 [17]。相比之下，MAVLink 2 提供可选的 13 字节签名，用于认证消息 [16]。Kim 等人最近提出了面向 UAVCAN 网络的协议感知入侵检测系统（IDS）[21]。它重组 UAVCAN 传输，并从载荷数据以及 CAN ID 和尾字节中对传输规则的违反中提取十个特征。在公开数据集的十种攻击场景上，最佳配置的平均准确率、精确率和召回率分别达到 99.89 %、99.96 % 和 99.88 %，其中传输规则特征对重放攻击（重新发送先前录下的合法消息）尤其有效 [21]。这些数字来自在一个数据集上的单项研究，但它们表明，传输层字段（CAN ID 和尾字节）是有用的检测特征来源，并且该研究所采用的是检测而非预防的思路。

**Experimental attacks on a DroneCAN bus.** A technical report from the Hacking and Countermeasure Research Lab at Korea University documents attacks on the internal network of a UAV that uses UAVCAN v0, that is, DroneCAN [23]. The testbed is a quadcopter with a Pixhawk 4 flight controller connected over a CAN bus to four ESCs (Holybro Kotleta20, which supports UAVCAN v0), with a 120 Ω terminator at the end of the bus. The terminator was replaced by a CAN shield attached to a Raspberry Pi 4, which could read all frames and transmit frames of the attacker's choice [23]. The injected frames use CAN ID 0x05040601, which identifies `RawCommand` (message type ID 1030), the message that commands the ESCs [23]. Decoded with the identifier layout of Section V-D, this identifier corresponds to priority 5, message type ID 1030 and source node ID 1 (derived). The attacker therefore transmits frames that carry the same identifier as the legitimate commands, which illustrates in practice that the source node ID in the CAN identifier is an unverified claim.

> **对 DroneCAN 总线的实验攻击。** 韩国大学 Hacking and Countermeasure Research Lab 的一份技术报告记录了对使用 UAVCAN v0（即 DroneCAN）的无人机内部网络的攻击 [23]。测试台是一架四旋翼，Pixhawk 4 飞控经 CAN 总线连接四个电调（Holybro Kotleta20，支持 UAVCAN v0），总线末端有一个 120 Ω 终端电阻。该终端电阻被换成一块连接树莓派 4 的 CAN shield，它可以读取所有帧，并发送攻击者选择的帧 [23]。注入的帧使用 CAN ID 0x05040601，它标识 RawCommand（消息类型 ID 1030），即指挥电调的消息 [23]。按 V-D 节的标识符布局解码，该标识符对应优先级 5、消息类型 ID 1030、源节点 ID 1（推导）。因此攻击者发送的帧与合法指令带有相同的标识符，这在实践中说明，CAN 标识符中的源节点 ID 只是一个未经验证的声明。

Three injection attacks were implemented [23]. In a *flooding* attack, frames with an all-zero payload are sent at fixed intervals while the transfer ID in the tail byte is incremented, so that the frames conform to the transfer rules. In a *fuzzy* attack, the payload is filled with random values, and the transfer CRC is recomputed for every frame from the data type signature and the payload, so that the frames remain well-formed. In a *replay* attack, previously recorded direction-control frames are retransmitted with their original timing [23]. Because the signature is derived from the publicly available data type definition (Section V-C), the fuzzy attack shows that the transfer CRC protects against accidental corruption only and does not authenticate the sender (derived from the description in [23]).

> 实现了三种注入攻击 [23]。*泛洪（flooding）*攻击以固定间隔发送载荷全为零的帧，同时递增尾字节中的传输 ID，使帧符合传输规则。*模糊（fuzzy）*攻击把载荷填成随机值，并对每一帧根据数据类型签名和载荷重新计算传输 CRC，使帧保持格式正确。*重放（replay）*攻击按原有时序重新发送此前录下的方向控制帧 [23]。由于签名来自公开的数据类型定义（V-C 节），模糊攻击表明传输 CRC 只能防意外损坏，并不能认证发送方（据 [23] 的描述推导）。

The reported effects depend on the attack type [23]. Flooding stopped the motors for the duration of the attack. Fuzzy injection disturbed flight intermittently and stopped the motors whenever critical values were injected. Replay caused a continuous movement to one side and made normal control impossible. Ten scenarios of 180–280 s were recorded, in which the attacks lasted 30–50 s each and were separated by periods of normal operation; the attacker's packet generation interval was 0.0015 s or 0.005 s, and the labelled dataset contains the CAN frames of the ten scenarios (for example 91,042 normal and 116,816 attack frames in scenario 1) [23].

> 报告的效果因攻击类型而异 [23]。泛洪使电机在攻击期间停转；模糊注入使飞行间歇受扰，注入关键值时电机停转；重放导致持续向一侧移动，使正常控制变得不可能。共记录了 10 个时长 180–280 秒的场景，其中每次攻击持续 30–50 秒，攻击之间有正常运行阶段；攻击者的数据包生成间隔为 0.0015 秒或 0.005 秒，带标签的数据集包含这 10 个场景的 CAN 帧（例如场景 1 有 91,042 个正常帧和 116,816 个攻击帧）[23]。

**Attack types on CAN-based networks.** Al-Jarrah et al. classify the cyberattacks that intrusion detection research addresses on intra-vehicle CAN networks into five groups [22]. In a *denial-of-service* (DoS) attack, the attacker exploits the arbitration mechanism by sending high-priority messages that force other nodes to stop transmitting; an attacker with limited knowledge can launch it, so its likelihood and impact are rated very high. In a *message injection* attack the attacker sends fabricated but valid messages, and in a *replay* attack it stores a valid message and sends it again later. *Message manipulation* alters or deletes messages, for example after compromising a gateway between two buses. In a *masquerade* attack the attacker compromises two nodes, learns the identifiers and the transmission frequency of one of them, silences it, and uses the other to send messages on its behalf. *Malware* can send malicious messages once it has been introduced through a vulnerable interface [22]. Among the 42 intrusion detection studies reviewed in [22], 69 % addressed injection and replay attacks, 26 % message manipulation, 17 % DoS, 10 % malware and 7 % masquerade attacks.

> **基于 CAN 的网络上的攻击类型。** Al-Jarrah 等人把入侵检测研究针对的机内 CAN 网络攻击分为五类 [22]。拒绝服务（DoS）攻击中，攻击者利用仲裁机制，发送高优先级消息迫使其他节点停止发送；知识有限的攻击者也能发动，因此其可能性和影响被评为很高。消息注入攻击中，攻击者发送伪造但有效的消息；重放攻击中，攻击者存下一条有效消息，稍后再次发送。消息篡改会修改或删除消息，例如在攻陷连接两条总线的网关之后。伪装攻击中，攻击者攻陷两个节点，学习其中一个节点的标识符和发送频率，让它沉默，再用另一个节点以它的名义发消息。恶意软件经由有漏洞的接口引入后，可发送恶意消息 [22]。在 [22] 所综述的 42 项入侵检测研究中，69% 针对注入与重放攻击，26% 针对消息篡改，17% 针对 DoS，10% 针对恶意软件，7% 针对伪装攻击。

**Dependence on correct implementation.** The mechanisms depend on correct hardware and firmware. The in-order requirement for the driver [4] and the rules for redundancy [4], [11] are examples where non-compliant hardware or firmware silently undermines the guarantees.

> **依赖正确实现。** 这些机制依赖正确的硬件和固件。驱动的顺序要求 [4] 和冗余规则 [4]、[11] 就是不合规的硬件或固件会悄悄破坏保证的例子。

**Dynamic allocation and mixed networks.** Dynamic node IDs are not persistent, so a node can receive a different ID after each restart [9]. The specification therefore advises against mixing static and dynamic nodes, and it gives conditions under which a mix is safe [9]. Consumers that select data by node ID must take this into account.

> **动态分配与混合网络。** 动态节点 ID 不持久，节点每次重启后可能得到不同的 ID [9]。因此规范建议不要混用静态和动态节点，并给出了混用时的安全条件 [9]。按节点 ID 选择数据的消费者必须考虑这一点。

**Nature of the reliability argument.** The specification chapters state requirements and recommendations, and they contain neither measurements of frame-loss rates nor failover times of redundant buses, and they do not address safety certification. The statements on reliability in this review therefore describe mechanisms by design. The quantitative performance of a particular installation depends on its implementation, wiring and load.

> **可靠性论证的性质。** 规范各章陈述的是要求和建议，其中既没有丢帧率的测量，也没有冗余总线的故障切换时间，也不涉及安全认证。因此本综述中关于可靠性的陈述描述的是设计上的机制。具体安装的定量性能取决于其实现、布线和负载。

## VII. COMPARISON WITH RELATED TECHNOLOGIES / 与相关技术的比较

DroneCAN is not the only way to connect components in an unmanned vehicle, and several of the alternatives operate at different layers. This section compares it with its sibling Cyphal, with MAVLink and with conventional point-to-point interfaces.

> DroneCAN 不是连接无人载具内部部件的唯一方式，而且好几种替代方案工作在不同层次。本章把它与同源的 Cyphal、与 MAVLink、以及与传统点对点接口比较。

### A. DroneCAN and Cyphal / DroneCAN 与 Cyphal

Cyphal and DroneCAN share a heritage and many design goals (Section IV), but they are different protocols with different wire formats. Table VIII compares them using the two specifications.

> Cyphal 和 DroneCAN 有共同的渊源和许多共同的设计目标（第 IV 章），但它们是线格式不同的不同协议。表 VIII 依据两份规范对它们进行比较。

| Property | DroneCAN | Cyphal (v1.0) |
|---|---|---|
| Origin | UAVCAN v0 [12], [13] | UAVCAN v1 [13], [15] |
| Transports | CAN 2.0B, CAN FD, UDP multicast, MAVLink tunnel [4]–[7] | Cyphal/CAN (CAN FD is the primary transport), Cyphal/UDP (experimental), Cyphal/serial [15] |
| CAN identifier | 29 bits [4] | 29 bits [15] |
| Priority levels | 32 (5 bits) [4] | 8 [15] |
| Node ID range on CAN | 1–127 [4] | up to 127 (7 bits) [15] |
| Message identifiers | 16-bit message type ID (0–65535), 8-bit service type ID [4] | 8192 subject IDs and 512 service IDs [15] |
| Single-frame payload | 7 bytes on classic CAN (derived); frames up to 64 bytes on CAN FD [4], [7] | 7 bytes classic CAN, up to 63 bytes CAN FD [15] |
| Transfer ID on CAN | 5 bits [4] | cyclic, modulo 32 [15] |
| Anonymous transfers | single-frame, 14-bit random discriminator, abort on bus error [4] | supported, non-deterministic collision resolution [15] |
| Type compatibility | 64-bit data type signature (CRC-64-WE) [3] | major and minor versions of data types, up to 256 each [15] |
| Redundancy | up to three redundant CAN interfaces [11] | non-redundant, doubly and triply redundant transports [15] |
| Time synchronisation, DSDL, pub/sub, RPC | yes [1], [2], [9] | yes [15] |

> | 属性 | DroneCAN | Cyphal（v1.0） |
> |---|---|---|
> | 起源 | UAVCAN v0 [12]、[13] | UAVCAN v1 [13]、[15] |
> | 传输 | CAN 2.0B、CAN FD、UDP 组播、MAVLink 隧道 [4]–[7] | Cyphal/CAN（CAN FD 为主传输）、Cyphal/UDP（实验性）、Cyphal/serial [15] |
> | CAN 标识符 | 29 位 [4] | 29 位 [15] |
> | 优先级数 | 32（5 位）[4] | 8 [15] |
> | CAN 上节点 ID 范围 | 1–127 [4] | 最多 127（7 位）[15] |
> | 消息标识 | 16 位消息类型 ID（0–65535），8 位服务类型 ID [4] | 8192 个 subject ID 和 512 个 service ID [15] |
> | 单帧载荷 | 经典 CAN 上 7 字节（推导）；CAN FD 帧最多 64 字节 [4]、[7] | 经典 CAN 7 字节，CAN FD 最多 63 字节 [15] |
> | CAN 上的传输 ID | 5 位 [4] | 循环，模 32 [15] |
> | 匿名传输 | 单帧，14 位随机判别符，总线错误时中止 [4] | 支持，非确定性的冲突解决 [15] |
> | 类型兼容性 | 64 位数据类型签名（CRC-64-WE）[3] | 数据类型的主、次版本，各最多 256 [15] |
> | 冗余 | 最多三个冗余 CAN 接口 [11] | 非冗余、双重和三重冗余传输 [15] |
> | 时间同步、DSDL、发布/订阅、RPC | 有 [1]、[2]、[9] | 有 [15] |

*Table VIII. Comparison of DroneCAN and Cyphal.*

> *表 VIII. DroneCAN 与 Cyphal 的比较。*

Several differences deserve comment. The identifier space of DroneCAN is larger (65,536 message types against 8192 subjects), and it offers a finer priority scale (32 against 8 levels), whereas Cyphal treats CAN FD as its primary transport and adds UDP and serial transports for non-CAN networks [4], [15]. Cyphal/CAN states that it can share a bus with other high-level CAN protocols only if they do not use CAN 2.0B frames [15]. Since DroneCAN uses CAN 2.0B frames exclusively [4], it follows that DroneCAN and Cyphal/CAN cannot be mixed on one bus and that a vehicle that uses both must separate them on different buses. The PX4 documentation supports using both families side by side but recommends DroneCAN for the most common setups [13]. In practice the choice follows the ecosystem: peripherals and firmware for ArduPilot and PX4 are built for DroneCAN [12], [13], [18], whereas Cyphal is the option when a design needs the higher throughput of CAN FD or a non-CAN transport [15].

> 有几处差异值得评论。DroneCAN 的标识符空间更大（65,536 种消息类型对 8192 个 subject），优先级刻度更细（32 级对 8 级），而 Cyphal 把 CAN FD 作为主传输，并为非 CAN 网络增加了 UDP 和串行传输 [4]、[15]。Cyphal/CAN 声明，只有当其他高层 CAN 协议不使用 CAN 2.0B 帧时，才能与之共用总线 [15]。由于 DroneCAN 只使用 CAN 2.0B 帧 [4]，由此可知 DroneCAN 与 Cyphal/CAN 不能混在同一条总线上，同时使用两者的飞行器必须把它们分在不同总线上。PX4 文档支持并行使用两个协议族，但对最常见的配置推荐 DroneCAN [13]。实际上选择取决于生态：ArduPilot 和 PX4 的外设与固件是为 DroneCAN 构建的 [12]、[13]、[18]，而当设计需要 CAN FD 更高的吞吐或非 CAN 传输时，Cyphal 才是选项 [15]。

### B. DroneCAN and MAVLink / DroneCAN 与 MAVLink

MAVLink and DroneCAN serve different purposes, and in most vehicles both are present. MAVLink is a message marshalling protocol for telemetry and control between unmanned systems, ground stations and companion computers [16]. A MAVLink 2 packet carries a header, a payload of up to 255 bytes, a checksum and an optional 13-byte signature, and its message ID is 24 bits wide instead of 8 bits in MAVLink 1 [16]. MAVLink 1 identifies the vehicle with a 1-byte system ID, which limits one ground station to 254 vehicles [16]. DroneCAN, in contrast, is a bus protocol for components inside the vehicle, with automatic decomposition of long transfers, a decentralised network and a standard set of functions for node monitoring and update [1], [2], [9]. Table IX contrasts them.

> MAVLink 和 DroneCAN 用途不同，大多数飞行器两者都有。MAVLink 是用于无人系统、地面站和机载计算机之间遥测与控制的消息编组协议 [16]。MAVLink 2 数据包携带包头、最多 255 字节的载荷、校验和以及可选的 13 字节签名，其消息 ID 宽 24 位，而 MAVLink 1 中是 8 位 [16]。MAVLink 1 用 1 字节的系统 ID 标识飞行器，使一个地面站最多能管 254 架飞行器 [16]。相比之下，DroneCAN 是用于飞行器内部部件的总线协议，具有长传输的自动拆分、去中心化的网络，以及一套用于节点监测和更新的标准功能 [1]、[2]、[9]。表 IX 对两者做了对照。

| Aspect | DroneCAN | MAVLink |
|---|---|---|
| Typical role | Intra-vehicle bus between flight controller, sensors, actuators [1], [14] | Vehicle–ground station and autopilot–companion computer link [16], [18] |
| Topology | Multi-drop bus, any node can publish, no master [1], [2] | Point-to-point links or telemetry networks with system and component IDs [16] |
| Large data | Automatic multi-frame transfers [2], [4] | Payload up to 255 bytes per packet [16] |
| Integrity | CAN frame CRC, transfer CRC for multi-frame transfers [4], [20] | Packet checksum [16] |
| Authentication | None defined in the transport chapters [4] | Optional message signature in MAVLink 2 [16] |
| Interplay | Can be tunnelled over MAVLink 2 (`mavcan:`) [6] | Carries `CAN_FRAME` / `CANFD_FRAME` messages [6] |

> | 方面 | DroneCAN | MAVLink |
> |---|---|---|
> | 典型角色 | 飞控、传感器、执行器之间的机内总线 [1]、[14] | 飞行器—地面站、自驾仪—机载计算机链路 [16]、[18] |
> | 拓扑 | 多点总线，任何节点都可发布，无主节点 [1]、[2] | 点对点链路或带系统/组件 ID 的遥测网络 [16] |
> | 大数据 | 自动多帧传输 [2]、[4] | 每包载荷最多 255 字节 [16] |
> | 完整性 | CAN 帧 CRC，多帧传输用传输 CRC [4]、[20] | 数据包校验和 [16] |
> | 认证 | 传输层章节中未定义 [4] | MAVLink 2 中有可选的消息签名 [16] |
> | 相互作用 | 可通过 MAVLink 2 隧道传输（`mavcan:`）[6] | 携带 `CAN_FRAME` / `CANFD_FRAME` 消息 [6] |

*Table IX. Comparison of DroneCAN and MAVLink.*

> *表 IX. DroneCAN 与 MAVLink 的比较。*

The two protocols are therefore complementary. The tunnelling transport in the DroneCAN specification [6] is the point where they meet: a ground station that speaks MAVLink can reach a DroneCAN node through the flight controller.

> 因此这两种协议是互补的。DroneCAN 规范中的隧道传输 [6] 是两者交汇的地方：说 MAVLink 的地面站可以通过飞控访问 DroneCAN 节点。

### C. DroneCAN and Point-to-Point Interfaces / DroneCAN 与点对点接口

The comparison with PWM, serial and I2C connections can be summarised along the dimensions raised in Section II. A PWM, DShot or OneShot output is a dedicated connection per actuator, so the number of actuators is bounded by the output hardware, typically six to eight on a flight controller [14]. A DroneCAN ESC is a node on a shared bus. The number of nodes is bounded by the 127 node IDs and by the available bandwidth (Section VI-B), not by the number of ports [14], and each ESC also reports its own status [10]. Dedicated interfaces also lack standard definitions for node discovery, firmware update and health monitoring, whereas DroneCAN provides them for all devices that implement it [9]. Point-to-point links, on the other hand, are simpler: they need no bus termination, no node ID configuration and no CAN controller, and a PWM output has a trivially bounded latency. DShot and OneShot are described in the PX4 documentation as lower-latency actuator protocols [14]. Whether a CAN-based actuator link meets the latency requirements of a given airframe has to be assessed for the application.

> 与 PWM、串行和 I2C 连接的比较，可以沿着第 II 章提出的维度来归纳。PWM、DShot 或 OneShot 输出是每个执行器一条专用连接，所以执行器数量受输出硬件限制，飞控上通常是六到八路 [14]。DroneCAN 电调是共享总线上的一个节点。节点数受 127 个节点 ID 和可用带宽（VI-B 节）限制，而不是受端口数限制 [14]，而且每个电调还会报告自己的状态 [10]。专用接口还缺少节点发现、固件更新和健康监测的标准定义，而 DroneCAN 为所有实现它的设备提供了这些 [9]。另一方面，点对点链路更简单：不需要总线终端、不需要节点 ID 配置、不需要 CAN 控制器，PWM 输出的延迟有简单明确的上界。PX4 文档把 DShot 和 OneShot 描述为更低延迟的执行器协议 [14]。基于 CAN 的执行器链路能否满足某个机型的延迟要求，需要针对应用单独评估。

## VIII. OPEN CHALLENGES AND RESEARCH DIRECTIONS / 开放挑战与研究方向

**Security.** The most evident open issue is the absence of authentication and encryption (Section VI-G). The CAN security literature surveyed by Bozdal et al. concludes that no countermeasure is agreed upon by industry and academia, because each one adds overhead, latency or incompatibility with standard CAN controllers [20]. DroneCAN has limited room for a MAVLink 2-style signature: the single-frame payload on classic CAN is only 7 bytes, so that a signature of the kind that MAVLink 2 appends (13 bytes [16]) would not fit into one frame. CAN FD, with up to 63 bytes of payload per frame, relaxes this constraint [7], although a larger frame uses more bus time. Intrusion detection, such as the protocol-aware approach of Kim et al. [21], does not require a change of the protocol and is therefore the most readily deployable direction. How authentication could be added in a backward-compatible way, for example in a vendor-specific message or at the application level, remains an open design question.

> **安全。** 最明显的开放问题是缺少认证与加密（VI-G 节）。Bozdal 等人综述的 CAN 安全文献认为，目前没有任何对策得到产业界和学术界的一致认可，因为每种对策都会增加开销、延迟，或与标准 CAN 控制器不兼容 [20]。DroneCAN 为 MAVLink 2 式的签名留出的空间有限：经典 CAN 上单帧载荷只有 7 字节，因此 MAVLink 2 所附加的那种签名（13 字节 [16]）装不进一帧。CAN FD 单帧最多 63 字节载荷，缓解了这一限制 [7]，不过更大的帧会占用更多总线时间。入侵检测，例如 Kim 等人的协议感知方法 [21]，不需要改动协议，因此是最容易部署的方向。如何以向后兼容的方式加入认证，例如放在厂商专用消息中或应用层，仍是开放的设计问题。

**Quantitative evaluation.** Questions that call for experimental treatment include the end-to-end latency distribution of a control loop (for example from `RawCommand` to the reaction of an ESC) for different bus loads and priorities, the loss and recovery behaviour under electromagnetic interference, the failover time when one of two redundant buses fails, and the bus utilisation of a realistic node mix. The analytic bounds of Section VI-B provide reference values against which such measurements can be compared. A test bench with a flight controller, several DroneCAN nodes and a CAN analyser is sufficient for a first study.

> **定量评估。** 适合通过实验研究的问题包括：不同总线负载和优先级下控制回路（例如从 RawCommand 到电调反应）的端到端延迟分布；电磁干扰下的丢失与恢复行为；两条冗余总线之一失效时的故障切换时间；以及真实节点组合下的总线利用率。VI-B 节的解析界限为这类测量提供了可比对的参考值。一套含飞控、若干 DroneCAN 节点和 CAN 分析仪的测试台，就足以开展第一项研究。

**CAN FD migration.** CAN FD raises the payload per frame to 64 bytes and the data-phase bit rate to as much as 8 Mbit/s [7]. DroneCAN already specifies the necessary changes (no TAO, quantised DLC) [7], but a migration requires transceivers and controllers that support CAN FD and bus layouts that tolerate higher data-phase rates. The operation of mixed networks of classic-CAN and CAN FD nodes is a further design question.

> **向 CAN FD 迁移。** CAN FD 把每帧载荷提高到 64 字节，数据段比特率最高可到 8 Mbit/s [7]。DroneCAN 已规定了所需的改动（无 TAO、量化 DLC）[7]，但迁移需要支持 CAN FD 的收发器和控制器，以及能承受更高数据段速率的总线布局。经典 CAN 节点与 CAN FD 节点混合网络如何运行，是又一个设计问题。

**Coexistence with Cyphal.** Both families use 29-bit identifiers and cannot share a bus (Section VII-A). Vehicles that integrate peripherals of both families need separate buses or gateways.

> **与 Cyphal 共存。** 两个协议族都用 29 位标识符，不能共用一条总线（VII-A 节）。集成了两个协议族外设的飞行器需要分开的总线或网关。

**Fault isolation on a shared bus.** A shared linear bus has the property that a node that is electrically faulty, or a short circuit in the harness, can affect all devices. The specification mitigates this through redundant buses and power protection [11]; active fault isolation, for example with bus guardians or star couplers, is not part of it.

> **共享总线上的故障隔离。** 共享线性总线的特点是：电气上有故障的节点，或线束中的短路，可能影响所有设备。规范通过冗余总线和电源保护缓解这一点 [11]；主动的故障隔离（例如总线守护器或星形耦合器）不在其内容之中。

**Evolution of the specifications.** Both protocols are under active development [1], [15]. The statements in this review refer to the DroneCAN specification as generated on 26 August 2025 and to version 1.0 of the Cyphal specification of 16 May 2025.

> **规范的演进。** 两种协议都在积极发展 [1]、[15]。本综述中的陈述所指的是 2025 年 8 月 26 日生成的 DroneCAN 规范，以及 2025 年 5 月 16 日的 Cyphal 规范 v1.0。

## IX. CONCLUSION / 结论

DroneCAN is the open, lightweight application-level protocol that ArduPilot and PX4 use on top of CAN to connect flight controllers with sensors, actuators and other peripherals. It descends from UAVCAN v0 and was renamed in 2022, when the UAVCAN v1 branch became Cyphal [13]. Its design combines a decentralised publish/subscribe and request/response model, a compact data description language with signatures, a transport layer that hides multi-frame transfers behind 29-bit identifiers with a priority field, and a standard set of functions for monitoring, discovery, time synchronisation, configuration, firmware update and dynamic node ID allocation [1]–[4], [9].

> DroneCAN 是开放、轻量的应用层协议，ArduPilot 和 PX4 在 CAN 之上用它把飞控与传感器、执行器及其他外设连接起来。它源自 UAVCAN v0，在 2022 年 UAVCAN v1 分支成为 Cyphal 时被更名 [13]。其设计结合了：去中心化的发布/订阅和请求/响应模型、带签名的紧凑数据描述语言、把多帧传输隐藏在带优先级字段的 29 位标识符之后的传输层，以及一套用于监测、发现、时间同步、配置、固件更新和动态节点 ID 分配的标准功能 [1]–[4]、[9]。

The reliability of the system results from the combination of several mechanisms: differential signalling, arbitration, error confinement and CRC inherited from CAN [20]; transfer-level checks, bounded waiting and redundant interfaces defined by DroneCAN [4]; hardware rules for connectors, power and bus length [11]; and application-level health reporting, redundant allocators and emergency signalling [9]. These mechanisms protect against noise, corrupted frames, failed nodes and failed buses. They do not protect against a malicious participant, because CAN and the DroneCAN transport define no authentication or encryption, which is the principal open problem [20], [21].

> 系统的可靠性来自多种机制的组合：继承自 CAN 的差分信号、仲裁、错误隔离和 CRC [20]；DroneCAN 定义的传输级检查、有限等待和冗余接口 [4]；关于连接器、电源和总线长度的硬件规则 [11]；以及应用层的健康报告、冗余分配器和紧急信号 [9]。这些机制可防护噪声、损坏的帧、失效的节点和失效的总线。它们不能防护恶意参与者，因为 CAN 和 DroneCAN 传输层都没有定义认证或加密，这是主要的开放问题 [20]、[21]。

Compared with Cyphal, DroneCAN offers a larger identifier space and a finer priority scale on classic CAN frames, whereas Cyphal emphasises CAN FD and additional transports [4], [15]. Compared with MAVLink, it addresses the communication inside the vehicle and not the link to the ground station [16]. Compared with point-to-point interfaces, it removes the dependence on the number of hardware ports and adds standard services for discovery, status monitoring and firmware update [9], [14]. Quantitative studies of latency, loss and failover behaviour on a test bench with redundant buses are a natural continuation of this analysis.

> 与 Cyphal 相比，DroneCAN 在经典 CAN 帧上提供更大的标识符空间和更细的优先级刻度，而 Cyphal 更强调 CAN FD 和额外的传输 [4]、[15]。与 MAVLink 相比，它面向飞行器内部的通信，而不是通往地面站的链路 [16]。与点对点接口相比，它消除了对硬件端口数量的依赖，并增加了发现、状态监测和固件更新的标准服务 [9]、[14]。在带冗余总线的测试台上对延迟、丢失和故障切换行为做定量研究，是这一分析自然的延续。

## REFERENCES

[1] DroneCAN Development Team, "1. Introduction," in *DroneCAN Specification*. Accessed: Oct. 2026. [Online]. Available: https://dronecan.github.io/Specification/ (page generated 26 Aug. 2025).

[2] DroneCAN Development Team, "2. Basic concepts," in *DroneCAN Specification*. [Online]. Available: https://dronecan.github.io/Specification/

[3] DroneCAN Development Team, "3. Data structure description language," in *DroneCAN Specification*. [Online]. Available: https://dronecan.github.io/Specification/

[4] DroneCAN Development Team, "4.1 CAN bus transport layer," in *DroneCAN Specification*. [Online]. Available: https://dronecan.github.io/Specification/

[5] DroneCAN Development Team, "4.2 UDP bus transport layer," in *DroneCAN Specification*. [Online]. Available: https://dronecan.github.io/Specification/

[6] DroneCAN Development Team, "4.3 MAVLink bus transport layer," in *DroneCAN Specification*. [Online]. Available: https://dronecan.github.io/Specification/

[7] DroneCAN Development Team, "4.4 CANFD bus transport layer," in *DroneCAN Specification*. [Online]. Available: https://dronecan.github.io/Specification/

[8] DroneCAN Development Team, "5. Application level conventions," in *DroneCAN Specification*. [Online]. Available: https://dronecan.github.io/Specification/

[9] DroneCAN Development Team, "6. Application level functions," in *DroneCAN Specification*. [Online]. Available: https://dronecan.github.io/Specification/

[10] DroneCAN Development Team, "7. List of standard data types," in *DroneCAN Specification*. [Online]. Available: https://dronecan.github.io/Specification/

[11] DroneCAN Development Team, "8. Hardware design recommendations," in *DroneCAN Specification*. [Online]. Available: https://dronecan.github.io/Specification/

[12] DroneCAN project, "DroneCAN: a lightweight protocol for aerospace and robotic applications," project website. Accessed: Oct. 2026. [Online]. Available: https://dronecan.github.io/

[13] PX4 Autopilot Project, "CAN (DroneCAN & Cyphal)," in *PX4 User Guide*. Accessed: Oct. 2026. [Online]. Available: https://docs.px4.io/

[14] PX4 Autopilot Project, "Basic concepts," in *PX4 User Guide* (main branch). [Online]. Available: https://docs.px4.io/

[15] OpenCyphal Development Team, *Cyphal Specification v1.0*, rev. 16 May 2025. [Online]. Available: https://opencyphal.org/specification

[16] A. Koubâa, A. Allouch, M. Alajlan, Y. Javed, A. Belghith, and M. Khalgui, "Micro Air Vehicle Link (MAVLink) in a nutshell: A survey," *IEEE Access*, vol. 7, pp. 87658–87680, 2019, arXiv:1906.10641.

[17] M. Ghamari, P. Rangel, M. Mehrubeoglu, G. S. Tewolde, and R. S. Sherratt, "Unmanned aerial vehicle communications for civil applications: A review," *IEEE Access*, vol. 10, pp. 102492–102531, 2022, doi: 10.1109/ACCESS.2022.3208571.

[18] N. Aliane, "A survey of open-source UAV autopilots," *Electronics*, vol. 13, no. 23, Art. no. 4785, 2024.

[19] N. Ravi and M. El-Sharkawy, "Integration of UAVs with real time operating systems using UAVCAN," in *Proc. IEEE 10th Annu. Ubiquitous Comput., Electron. Mobile Commun. Conf. (UEMCON)*, 2019, pp. 600–605, doi: 10.1109/UEMCON47517.2019.8993011.

[20] M. Bozdal, M. Samie, and I. Jennions, "A survey on CAN bus protocol: Attacks, challenges, and potential solutions," in *Proc. Int. Conf. Comput., Electron. Commun. Eng. (iCCECE)*, Southend, U.K., Aug. 2018, doi: 10.1109/iCCECOME.2018.8658720.

[21] S. Kim, H. Im, J. Choi, and S. Lee, "Protocol-aware feature-based intrusion detection system for UAVCAN networks," *Sensors*, vol. 26, no. 19, Art. no. 6061, 2026, doi: 10.3390/s26196061.

[22] O. Y. Al-Jarrah, C. Maple, M. Dianati, D. Oxtoby, and A. Mouzakitis, "Intrusion detection systems for intra-vehicle networks: A review," *IEEE Access*, vol. 7, pp. 21266–21289, 2019, doi: 10.1109/ACCESS.2019.2894183.

[23] D. Kim, Y. Song, S. Kwon, H. Kim, J. D. Yoo, and H. K. Kim, "UAVCAN dataset description," Hacking and Countermeasure Research Lab, Korea University, arXiv:2212.09268v2 [cs.CR], Apr. 2024. [Online]. Available: https://arxiv.org/abs/2212.09268