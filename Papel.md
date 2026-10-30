# Table of Contents

**DroneCAN: A Lightweight Protocol for Highly Reliable Intra-Vehicle Communication in Aerospace and Robotic Applications**

- **Abstract**
- **Keywords**

## I. Introduction
- A. Background: From Single Controllers to Distributed Systems
- B. ArduPilot and PX4
- C. From CAN to DroneCAN
- D. Related Protocols and Their Relationships (DroneCAN, UAVCAN, Cyphal, MAVLink)
- E. Organisation

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