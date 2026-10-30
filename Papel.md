# Table of Contents

**DroneCAN: A Lightweight Protocol for Highly Reliable Intra-Vehicle Communication in Aerospace and Robotic Applications**

- **Abstract**
- **Keywords**

## 1. Introduction
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

## II. Communication Interfaces in Aerial and Robotic Platforms
- A. Architecture of a Typical UAV
- B. Point-to-Point Interfaces (PWM, UART, I2C, SPI)
- C. Higher-Level Links: MAVLink
- D. Summary of Requirements

## III. Overview of the DroneCAN Protocol Stack
- A. Design Goals
- B. The Three Layers
  - 1. Layer 1: Physical Layer
  - 2. Layer 2: Data-Link and Transport Layer
  - 3. Layer 3: Application Layer
- C. Network Model: Peer Network, Node IDs, No Bus Master
- D. Communication Semantics: Publish/Subscribe and Services
- E. From UAVCAN v0 to DroneCAN: The 2022 Split

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