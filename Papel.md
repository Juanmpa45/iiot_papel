# Table of Contents

**DroneCAN: A Lightweight Protocol for Highly Reliable Intra-Vehicle Communication in Aerospace and Robotic Applications**

- **Abstract**
- **Keywords**

## 1. Introduction: What is DroneCAN
### 1.1. Background: From Single Controllers to Distributed System
A modern unmanned aerial vehicle (UAV) is no longer a single controller wired to a few actuators; it is a distributed embedded system in which several devices, each with its own processor, cooperate by exchanging data continuously during flight. A typical multirotor combines a flight controller. A typical multirotor combines a flight controller (the circuit board that runs the autopilot software) with one or more GNSS receivers (satellite-navigation receivers such as GPS), a magnetometer (compass), an airspeed sensor, a rangefinder, several electronic speed controllers (ESCs, circuits that regulate motor speed), a battery monitor and, optionally, a companion computer (an additional computer for high-level tasks such as image processing). The DroneCAN specification describes the processors in such systems as ranging from high-performance on-board computers to extremely resource-constrained microcontrollers [1], and the devices often come from different vendors, which is why the same specification stresses inter-vendor compatibility [11].

he conventional approach to connecting these devices is *point-to-point wiring*, in which each device is attached to the flight controller through a dedicated port and cable, using the interface that the device provides. This arrangement is simple to implement, but it has three limitations. First, the number of ports is determined by the hardware and therefore bounds the number of actuators and sensors that a vehicle can carry. Second, each interface type requires its own cabling and driver, which increases the weight, the number of connectors and the number of potential points of failure. Third, these interfaces do not provide common services such as device discovery, status monitoring or firmware update.

Communication faults can have severe consequences: the loss of the link between a sensor and the flight controller can result in the loss of the vehicle, damage to property or injury to people. The communication subsystem must therefore tolerate electrical noise and recover from transmission errors, properties that the Controller Area Network (CAN) provides through differential wiring and built-in error handling [19], [20]. The DroneCAN specification adds further requirements: a network without a master node, support for redundant interfaces and redundant nodes, and low-latency, high-throughput communication [1].

### 1.2. ArduPilot and PX4
An autopilot is an embedded computer that runs the software providing control and guidance of an unmanned vehicle. Its key tasks include stabilising attitude and heading, flying missions along waypoints, triggering failsafe actions (predefined reactions to faults, such as returning to the launch point) and connecting to ground control stations [18].

ArduPilot is open-source autopilot software that originated in 2007 in the DIY Drones community as an offshoot of the Arduino project. It was initially designed as monolithic software (a single program) and later moved to the ChibiOS real-time operating system (RTOS) [18].

PX4 is open-source autopilot software that originated in 2008 at ETH Zurich. It has a modular architecture and runs on the NuttX RTOS [18], [14].

Both projects use the MAVLink protocol as the primary protocol for communication with ground stations [18], whereas communication between onboard components relies on CAN: PX4 uses UAVCAN [18], and ArduPilot uses CAN for high-speed communication between components such as GNSS units, sensors and controllers [18]. UAVCAN version 0 has since been renamed DroneCAN, which is now the primary CAN protocol used by ArduPilot and PX4 to communicate with CAN peripherals [12], [13].

### 1.3. **From CAN to DroneCAN**
The Controller Area Network (CAN) was developed by Bosch for the automotive industry to replace complex wiring harnesses with a two-wire bus, that is, a shared communication line to which all devices are connected [19], [20]. It is a multi-master, message-broadcast system with a maximum signalling rate of about 1 Mbit/s that offers immunity to electrical noise and the ability to detect and recover from errors [19], [20]. CAN is standardised in the ISO 11898 series [1], which covers only the physical and data-link layers: it specifies how bits are signalled on the bus and how frames are transmitted, but not how applications name data, assign node addresses, split large data structures across frames or discover other devices. These functions are provided by a higher-layer protocol that runs on top of CAN. In industrial automation this role is filled by protocols such as CANopen, in heavy road vehicles by SAE J1939, in aircraft by ARINC 825, and in unmanned aerial vehicles by DroneCAN. Without such a protocol, each vendor would invent its own conventions and interoperability would be lost.

DroneCAN is an application-level protocol of this kind for aerospace and robotic applications. The DroneCAN specification describes it as a lightweight protocol designed to provide a highly reliable communication method for aerospace and robotic applications via the CAN bus [2]. Devices on the bus, called nodes, form a decentralised peer network in which every node has a unique numeric identifier [2]. Nodes exchange data either as broadcast messages that any interested node can receive (publish/subscribe) or as one-to-one requests and responses (services), and the formats of both are written in a text-based data structure description language [3]. On top of this, the protocol defines standard functions such as node status monitoring, node discovery, time synchronisation, parameter configuration, firmware update and dynamic node ID allocation [9]. DroneCAN is the primary CAN protocol used by the ArduPilot and PX4 projects to communicate with CAN peripherals, and it was created to continue the development of the widely used UAVCAN v0 protocol [12]. The PX4 documentation recommends DroneCAN for most common CAN setups [13].

### 1.4. **Related Protocols and Their Relationships**
DroneCAN is one of several communication technologies available to UAV designers. MAVLink (Micro Air Vehicle Link) is a lightweight messaging protocol for communication between unmanned systems and ground stations and between autopilots and companion computers. Cyphal is a transport-independent protocol whose primary transport is CAN FD, an extension of CAN for larger frames and higher data rates [15].

UAVCAN was the original open protocol for CAN-based communication between the components of a vehicle. In 2022 the project split in two: UAVCAN v0 was renamed DroneCAN, and UAVCAN v1 was renamed Cyphal [13]. The two descendants share design goals, such as a network without a master node and both publish/subscribe and request/response communication [1], [15], but they define different CAN frame formats [4], [15] and are therefore separate protocols, not versions of one another. Publications that predate the split and use the name UAVCAN, such as [19], refer to what is now DroneCAN. MAVLink is independent of this lineage: it connects the vehicle with ground stations and companion computers, not the components inside the vehicle [16], [18], and it meets DroneCAN only where DroneCAN frames are carried inside MAVLink 2 messages [6].

## 2. Communication in UAVs: Why we need DroneCAN
### 2.1. Architecture of UAV
The core of a UAV is the flight controller (FC), which communicates with the surrounding devices to exchange data and commands. The flight controller hardware runs the autopilot, that is, flight stack software on a real-time operating system (RTOS), which stabilises the vehicle and carries out commands and missions [14]. The surrounding devices fall into four groups: sensors, actuators, the power supply, and links to the pilot and to other computers.

The flight controller uses outputs to drive actuators: motors, flight surfaces such as ailerons and flaps, and payloads such as camera triggers or grippers [14]. A motor is driven through an *electronic speed controller* (ESC), which converts the signal from the flight controller into the appropriate level of power for the motor [14].

The vehicle is usually powered by a lithium-polymer battery connected through a power module, which supplies the flight controller and the ESCs separately [14].

The vehicle also communicates with the pilot and with other computers. A pilot can fly the vehicle with a radio control (RC) transmitter and a receiver on the vehicle [14]. A *ground control station* (GCS), such as QGroundControl, communicates with the vehicle through a *telemetry radio*, a bidirectional wireless data link [14].

Fig. 1 shows the main communication links of the flight controller. Actuators and sensor modules such as the GNSS module are attached directly to dedicated ports of the controller, and the outputs can also be DroneCAN nodes, for example DroneCAN motor controllers [14]. The ground station and the companion computer are reached through links such as the telemetry radio, a serial cable, an Ethernet cable or Wi-Fi [14].

Fig. 1. Typical communication links around a flight controller.

### 2.2. Point-to-Point Interfaces (PWM, UART, I2C, SPI)
The traditional way to connect devices to a flight controller is to give each device its own dedicated interface on the controller, chosen to suit the kind of data that the device exchanges. These interfaces are hardware interfaces and signalling methods rather than application-level protocols: they define how electrical signals on the wires represent data, but not what the data mean. Four are in common use. Pulse-width modulation (PWM) sends a setpoint to an actuator over a single wire. A UART (universal asynchronous receiver-transmitter) is a serial link between two devices. I2C and SPI are short-distance serial buses on which the controller communicates with several peripheral chips or modules over shared wires. PWM and UART therefore connect exactly two devices, whereas I2C and SPI share wires among several, but in all four cases the connection is local to the controller’s own ports, and no common addressing or services extend across the vehicle.

PWM is the simplest of them and is used for actuators. It encodes a setpoint, such as the desired motor speed, in the width of the pulses of a periodic signal, so one wire carries one setpoint in one direction, from the controller to one actuator. The output connectors of the controller are called ports, in PX4 they are MAIN and AUX[14]. The number of outputs is fixed by the hardware, typically six to eight [14].

The serial interfaces suit devices that exchange data rather than receive a single setpoint. UART links are used for telemetry radios and companion computers [14], whereas I2C and SPI are used for sensors and other peripherals located close to the controller, SPI being the faster of the two. All three are simple and widely supported. Their main limit is range: I2C and SPI are designed for short connections, and a UART connects only two devices.

Point-to-point wiring therefore has three limitations:

- The number of ports is fixed by the hardware [14], so each additional device needs another port and another cable.
- Each interface type needs its own driver and cabling.
- The interfaces define no common services such as device discovery, status monitoring or firmware update [9].

### 2.3. MAVLink: Communication Between the Vehicle and Other Computers
The interfaces described above connect individual devices to the flight controller. MAVLink addresses a different problem: communication between the vehicle and the other computers that supervise or assist it, namely the ground control station and the companion computer [16], [18]. The point-to-point interfaces thus serve communication between the flight controller and the devices inside the vehicle, whereas MAVLink serves communication between the vehicle and other computers. DroneCAN belongs to the first of these two problems: it connects the flight controller and the onboard devices over a shared CAN bus [12], [19].

### 2.4. Summary of Requirements
The limitations of point-to-point wiring lead directly to the following requirements for an intra-vehicle bus:

- a shared medium to which many devices connect through one interface, so that the number of devices is not limited by the number of controller ports and each additional device does not need another cable;
- one interface and one type of cabling for all device types, so that a single driver serves every device;
- standard definitions of the high-level functions common to all nodes, such as discovery, status monitoring, configuration and firmware update.

Using such a bus for the control of a vehicle on small embedded hardware adds further requirements:

- no single point of failure, and therefore no bus master (a node that controls access for all others);
- support for redundant interfaces and redundant nodes;
- low latency and high throughput for hard real-time control loops, that is, control loops that must meet strict timing deadlines;
- a simple way to exchange payloads that exceed a single frame;
- low computational requirements, down to resource-constrained microcontrollers;
- an open specification with open reference implementations.

These requirements lead to the design goals and philosophy of DroneCAN [1].

## 3. Overview: How to design DroneCAN
### 3.1. Design Goals
The CAN bus already removes the first two limitations of point-to-point wiring: the number of devices is no longer limited by the number of controller ports, and one interface and one type of cabling serve all of them. However, CAN defines no common services, and sharing a single medium among all devices brings problems of its own, such as single points of failure, payloads that exceed the size of a frame, and the demands for redundancy and low latency. The design goals of DroneCAN address these two groups of problems [1].

To fill the gap in common services of CAN, DroneCAN defines standard messages and services for the functions that every vehicle needs: discovering the nodes on the network, configuring a node, updating its firmware, monitoring its status, synchronising clocks and assigning node IDs automatically [1]. The protocol keeps simple and its computational requirements low, because the devices on the bus range from microcontrollers with very little memory and processing power to high-performance computers running Linux [1]. The specification is fully open, and its reference implementations are open source under the MIT licence [1].

The remaining goals address the problems of sharing one medium. A master node that controls access to the bus would be a single point of failure, so DroneCAN has no master and gives all nodes the same communication rights. A classical CAN frame carries at most 8 bytes of data [4], which is too little for the GNSS solutions and three-dimensional vectors that a vehicle needs, so DroneCAN splits such data over several frames and reassembles it at the receiver; the application hands over and receives complete data structures and does not have to handle this itself. A failure on the shared bus affects every device, so DroneCAN also supports redundant interfaces and redundant nodes. Finally, because all devices share the same bandwidth, the data of time-critical control loops must still arrive quickly, so DroneCAN aims at high throughput and low latency [1].

### 3.2. Network Model and Communication Methods
A DroneCAN network is a decentralised peer network in which every device on the bus has a unique numeric node ID, with valid values from 1 to 127 [2], [4]. Each message and service has a data type ID that identifies its data structure, and the node ID in a message identifies its source, so that messages of the same type sent by redundant nodes can be told apart [2].

Nodes communicate mainly by message broadcasting, which follows the publish/subscribe model and serves one-to-many and many-to-many communication [2]. In addition, service invocation provides a request/response exchange between exactly two nodes and is typically used for parameter updates, file transfer and firmware update [2].

### 3.3. Layer Structure
DroneCAN builds on the physical and data-link layers of CAN, which provide the bus and the frames, and adds a transport layer and an application layer (Fig. 2).

The transport layer sends each piece of data, that is, a message, a service request or a service response, in CAN frames, splitting it over several frames when it does not fit into one. The 29-bit identifier of each frame contains the priority, the data type ID and the node ID, and the receiver puts the frames back together into the complete data [4]. The transport layer can also run over CAN FD, UDP multicast and MAVLink 2 messages [5]–[7].

The application layer defines the data structures of messages and services in the Data Structure Description Language (DSDL) and, on top of them, provides standard high-level functions such as node discovery and health monitoring [2].

The following chapters describe these layers in detail, starting with CAN.

## IV. Layer 1: Physical Layer
- A. Controller Area Network (CAN) Fundamentals
  - 1. Differential Signalling and Bus Termination
  - 2. Dominant and Recessive Bits, Bit Stuffing
  - 3. Bit Rates, Sample Point and Bus Length
- B. Hardware Design Recommendations
  - 1. Connectors (D-Sub, M8, Micro)
  - 2. Bus Power and Overcurrent Protection
  - 3. Redundant CAN Interfaces and Bus Topology
- C. Automatic Bit-Rate Detection
- D. CAN FD at the Physical Level

## V. Layer 2: Data-Link and Transport Layer
- A. CAN Data-Link Mechanisms
  - 1. Frame Structure
  - 2. Arbitration and Priority
  - 3. Error Detection and Fault Confinement (Error-Passive and Bus-Off)
- B. Transfers: Message, Anonymous and Service
- C. 29-Bit CAN Identifier Layout
  - 1. Message Frames
  - 2. Anonymous Message Frames and the Discriminator
  - 3. Service Frames
  - 4. Worked Example
- D. Tail Byte and Transfer ID
- E. Multi-Frame Transfers
  - 1. Decomposition and Reassembly
  - 2. Transfer CRC and Toggle Bit
  - 3. Frame-Count Calculation
- F. Timing Rules and Timeouts
- G. Redundant Interfaces: Simultaneous Transmission and Deduplication
- H. Alternative Transports
  - 1. CAN FD
  - 2. UDP Multicast
  - 3. MAVLink Tunnel

## VI. Layer 3: Application Layer
- A. Data Structure Description Language (DSDL)
  - 1. Definition Files and Namespaces
  - 2. Type System and Serialisation
  - 3. Data Type Signatures and Compatibility
  - 4. Data Type IDs and Vendor-Specific Ranges
- B. Standard Data Types
- C. Standard Application-Level Functions
  - 1. Node Status (NodeStatus) and Health Monitoring
  - 2. Node Discovery (GetNodeInfo)
  - 3. Time Synchronisation
  - 4. Parameter Configuration
  - 5. File Transfer and Firmware Update
  - 6. Debugging and Emergency Functions (Panic)
  - 7. Dynamic Node ID Allocation (with Raft Consensus)

## VII. Reliability and Safety Mechanisms
- A. Overview: Mechanisms by Layer
- B. Physical Layer Mechanisms
- C. Data-Link and Transport Layer Mechanisms
- D. Application Layer Mechanisms
- E. Redundancy: Interfaces, Nodes and Services
- F. Limits of the Protection
  - 1. No Authentication or Encryption
  - 2. Attack Types on CAN-Based Networks
  - 3. Experimental Attacks on a DroneCAN Bus
  - 4. Dependence on Correct Implementation
  - 5. Dynamic Allocation and Mixed Networks

## VIII. Comparison with Related Technologies
- A. DroneCAN and Cyphal
- B. DroneCAN and MAVLink
- C. DroneCAN and Point-to-Point Interfaces

## IX. Open Challenges and Research Directions
- A. Security and Intrusion Detection
- B. Quantitative Evaluation
- C. Migration to CAN FD
- D. Coexistence with Cyphal
- E. Fault Isolation on a Shared Bus
- F. Evolution of the Specifications

## X. Conclusion

## References