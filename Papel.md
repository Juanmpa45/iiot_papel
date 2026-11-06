**DroneCAN: un protocolo ligero de comunicación fiable para aeronaves**

- **Abstract**
- **Keywords**

## 1. Introducción
### 1.1.Contexto: de controladores únicos a sistemas distribuidos
Un vehículo aéreo no tripulado (UAV) moderno ya no es un único controlador conectado con cables a unos pocos actuadores. Es un sistema embebido distribuido, en el que varios dispositivos, cada uno con su propio procesador, cooperan intercambiando datos de forma continua durante el vuelo. Un multirrotor típico combina un controlador de vuelo (la placa de circuito que ejecuta el software del piloto automático) con uno o más receptores GNSS (receptores de navegación por satélite, como el GPS), un magnetómetro (brújula), un sensor de velocidad del aire, un telémetro, varios variadores electrónicos de velocidad (ESC, circuitos que regulan la velocidad de los motores), un monitor de batería y, de forma opcional, un ordenador complementario (un ordenador adicional para tareas de alto nivel, como el procesamiento de imágenes). La especificación de DroneCAN describe los procesadores de estos sistemas como muy variados, desde ordenadores de a bordo de alto rendimiento hasta microcontroladores con recursos muy limitados [1]. Además, los dispositivos suelen venir de fabricantes distintos, y por eso la misma especificación insiste en la compatibilidad entre fabricantes [11].

La forma convencional de conectar estos dispositivos es el cableado punto a punto: cada dispositivo se conecta al controlador de vuelo mediante un puerto y un cable propios, usando la interfaz que ofrece el dispositivo. Esta solución es fácil de implementar, pero tiene tres limitaciones. Primero, el número de puertos lo determina el hardware, y por tanto limita el número de actuadores y sensores que puede llevar un vehículo. Segundo, cada tipo de interfaz necesita su propio cableado y su propio controlador (driver), lo que aumenta el peso, el número de conectores y el número de posibles puntos de fallo. Tercero, estas interfaces no ofrecen servicios comunes, como el descubrimiento de dispositivos, la supervisión del estado o la actualización del firmware.

Los fallos de comunicación pueden tener consecuencias graves: la pérdida del enlace entre un sensor y el controlador de vuelo puede causar la pérdida del vehículo, daños materiales o lesiones a personas. Por eso, el subsistema de comunicación debe tolerar el ruido eléctrico y recuperarse de los errores de transmisión. Estas propiedades las ofrece la red Controller Area Network (CAN) mediante el cableado diferencial y la gestión de errores integrada [19], [20]. La especificación de DroneCAN añade otros requisitos: una red sin nodo maestro, soporte para interfaces y nodos redundantes, y comunicación de baja latencia y alto caudal de datos [1].

### 1.2. ArduPilot y PX4
Un piloto automático es un ordenador embebido que ejecuta el software que controla y guía un vehículo no tripulado. Entre sus tareas principales están estabilizar la actitud y el rumbo, volar misiones siguiendo puntos de ruta, activar acciones de seguridad (failsafe, reacciones predefinidas ante fallos, como volver al punto de despegue) y conectarse con las estaciones de control en tierra [18].

ArduPilot es un software de piloto automático de código abierto, que nació en 2007 en la comunidad DIY Drones como una derivación del proyecto Arduino. Al principio se diseñó como un software monolítico (un único programa) y más tarde pasó al sistema operativo en tiempo real (RTOS) ChibiOS [18].

PX4 es un software de piloto automático de código abierto, que nació en 2008 en la ETH de Zúrich. Tiene una arquitectura modular y funciona sobre el RTOS NuttX [18], [14].

Los dos proyectos usan el protocolo MAVLink como protocolo principal para comunicarse con las estaciones de tierra [18], mientras que la comunicación entre los componentes de a bordo se basa en CAN: PX4 usa UAVCAN [18], y ArduPilot usa CAN para la comunicación de alta velocidad entre componentes como las unidades GNSS, los sensores y los controladores [18]. La versión 0 de UAVCAN se ha renombrado DroneCAN, que hoy es el protocolo CAN principal que usan ArduPilot y PX4 para comunicarse con los periféricos CAN [12], [13].

### 1.3. De CAN a DroneCAN
La red Controller Area Network (CAN) fue desarrollada por Bosch para la industria del automóvil, con el fin de sustituir los complejos mazos de cables por un bus de dos hilos, es decir, una línea de comunicación compartida a la que se conectan todos los dispositivos [19], [20]. Es un sistema multimaestro y de difusión de mensajes, con una velocidad máxima de unos 1 Mbit/s, que ofrece inmunidad al ruido eléctrico y capacidad para detectar errores y recuperarse de ellos [19], [20]. CAN está normalizado en la serie ISO 11898 [1], que cubre solo la capa física y la capa de enlace de datos: especifica cómo se señalizan los bits en el bus y cómo se transmiten las tramas, pero no cómo las aplicaciones dan nombre a los datos, asignan direcciones a los nodos, dividen estructuras de datos grandes en varias tramas o descubren otros dispositivos. Estas funciones las proporciona un protocolo de capa superior que funciona sobre CAN. En la automatización industrial este papel lo cumplen protocolos como CANopen, en los vehículos pesados de carretera SAE J1939, en los aviones ARINC 825, y en los vehículos aéreos no tripulados DroneCAN. Sin un protocolo así, cada fabricante inventaría sus propias convenciones y se perdería la interoperabilidad.

DroneCAN es un protocolo de nivel de aplicación de este tipo, para aplicaciones aeroespaciales y robóticas. La especificación de DroneCAN lo describe como un protocolo ligero, diseñado para ofrecer un método de comunicación muy fiable en aplicaciones aeroespaciales y robóticas a través del bus CAN [2]. Los dispositivos del bus, llamados nodos, forman una red descentralizada de nodos iguales, en la que cada nodo tiene un identificador numérico único [2]. Los nodos intercambian datos como mensajes de difusión que puede recibir cualquier nodo interesado (publicación/suscripción), o como peticiones y respuestas uno a uno (servicios). Los formatos de ambos se escriben en un lenguaje de descripción de estructuras de datos basado en texto [3]. Además, el protocolo define funciones estándar, como la supervisión del estado de los nodos, el descubrimiento de nodos, la sincronización horaria, la configuración de parámetros, la actualización del firmware y la asignación dinámica de ID de nodo [9]. DroneCAN es el protocolo CAN principal que usan los proyectos ArduPilot y PX4 para comunicarse con los periféricos CAN, y se creó para continuar el desarrollo del protocolo UAVCAN v0, muy extendido [12]. La documentación de PX4 recomienda DroneCAN para la mayoría de las configuraciones CAN habituales [13].

### 1.4. Protocolos relacionados y sus relaciones
DroneCAN es una de las varias tecnologías de comunicación disponibles para quienes diseñan UAV. MAVLink (Micro Air Vehicle Link) es un protocolo de mensajería ligero para la comunicación entre sistemas no tripulados y estaciones de tierra, y entre pilotos automáticos y ordenadores complementarios. Cyphal es un protocolo independiente del transporte, cuyo transporte principal es CAN FD, una extensión de CAN para tramas más grandes y velocidades de datos más altas [15].

UAVCAN fue el protocolo abierto original para la comunicación basada en CAN entre los componentes de un vehículo. En 2022 el proyecto se dividió en dos: UAVCAN v0 pasó a llamarse DroneCAN, y UAVCAN v1 pasó a llamarse Cyphal [13]. Los dos descendientes comparten objetivos de diseño, como una red sin nodo maestro y la comunicación tanto de publicación/suscripción como de petición/respuesta [1], [15], pero definen formatos de trama CAN distintos [4], [15], y por eso son protocolos separados, no versiones uno del otro. Las publicaciones anteriores a la división que usan el nombre UAVCAN, como [19], se refieren a lo que hoy es DroneCAN. MAVLink es independiente de esta línea: conecta el vehículo con las estaciones de tierra y los ordenadores complementarios, no los componentes dentro del vehículo [16], [18], y solo coincide con DroneCAN cuando las tramas DroneCAN se transportan dentro de mensajes MAVLink 2 [6].

## 2. Communication Interfaces in UAVs
### 2.1. Architecture of UAV
The core of a UAV is the flight controller (FC), which communicates with the surrounding devices to exchange data and commands. The flight controller hardware runs the autopilot, that is, flight stack software on a real-time operating system (RTOS), which stabilises the vehicle and carries out commands and missions [14]. The surrounding devices fall into four groups: sensors, actuators, the power supply, and links to the pilot and to other computers.

The flight controller uses outputs to drive actuators: motors, flight surfaces such as ailerons and flaps, and payloads such as camera triggers or grippers [14]. A motor is driven through an *electronic speed controller* (ESC), which converts the signal from the flight controller into the appropriate level of power for the motor [14].

The vehicle is usually powered by a lithium-polymer battery connected through a power module, which supplies the flight controller and the ESCs separately [14].

The vehicle also communicates with the pilot and with other computers. A pilot can fly the vehicle with a radio control (RC) transmitter and a receiver on the vehicle [14]. A *ground control station* (GCS), such as QGroundControl, communicates with the vehicle through a *telemetry radio*, a bidirectional wireless data link [14].

Fig. 1 shows the main communication links of the flight controller. Actuators and sensor modules such as the GNSS module are attached directly to dedicated ports of the controller, and the outputs can also be DroneCAN nodes, for example DroneCAN motor controllers [14]. The ground station and the companion computer are reached through links such as the telemetry radio, a serial cable, an Ethernet cable or Wi-Fi [14].

Fig. 1. Typical communication links around a flight controller.

### 2.2. Point-to-Point Interfaces (PWM, UART, I2C, SPI)
The traditional way to connect devices to a flight controller is to give each device its own dedicated interface on the controller, chosen to suit the kind of data that the device exchanges. These interfaces are hardware interfaces and signalling methods rather than application-level protocols: they define how electrical signals on the wires represent data, but not what the data mean. Four are in common use. Pulse-width modulation (PWM) sends a setpoint to an actuator over a single wire. A UART (universal asynchronous receiver-transmitter) is a serial link between two devices. I2C and SPI are short-distance serial buses on which the controller communicates with several peripheral chips or modules over shared wires. PWM and UART therefore connect exactly two devices, whereas I2C and SPI share wires among several, but in all four cases the connection is local to the controller’s own ports, and no common addressing or services extend across the vehicle.

PWM is the simplest of them and is used for actuators. It encodes a setpoint, such as the desired motor speed, in the width of the pulses of a periodic signal, so one wire carries one setpoint in one direction, from the controller to one actuator. The output connectors of the controller are called ports, in PX4 they are MAIN and AUX [14]. The number of outputs is fixed by the hardware, typically six to eight [14].

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

These requirements lead to the design goals and architecture of DroneCAN [1].

## 3. Design Goals and Architecture of DroneCAN
### 3.1. Design Goals
The CAN bus already removes the first two limitations of point-to-point wiring: the number of devices is no longer limited by the number of controller ports, and one interface and one type of cabling serve all of them. However, CAN defines no common services, and sharing a single medium among all devices brings problems of its own, such as single points of failure, payloads that exceed the size of a frame, and the demands for redundancy and low latency. The design goals of DroneCAN address these two groups of problems [1].

To fill the gap in common services of that CAN leaves undefined, DroneCAN defines standard messages and services for the functions that every vehicle needs: discovering the nodes on the network, configuring a node, updating its firmware, monitoring its status, synchronising clocks and assigning node IDs automatically [1]. The protocol is kept simple and its computational requirements low, because the devices on the bus range from microcontrollers with very little memory and processing power to high-performance computers running Linux [1]. The specification is fully open, and its reference implementations are open source under the MIT licence [1].

The remaining goals address the problems of sharing one medium. A master node that controls access to the bus would be a single point of failure, so DroneCAN has no master and gives all nodes the same communication rights. A classical CAN frame carries at most 8 bytes of data [4], which is too little for the GNSS solutions and three-dimensional vectors that a vehicle needs, so DroneCAN splits such data over several frames and reassembles it at the receiver; the application hands over and receives complete data structures and does not have to handle this itself. A failure on the shared bus affects every device, so DroneCAN also supports redundant interfaces and redundant nodes. Finally, because all devices share the same bandwidth, the data of time-critical control loops must still arrive quickly, so DroneCAN aims at high throughput and low latency [1].

### 3.2. Network Model and Communication Methods
A DroneCAN network is a decentralised peer network in which every device on the bus has a unique numeric node ID, with valid values from 1 to 127 [2], [4]. Each message and service has a data type ID that identifies its data structure, and the node ID in a message identifies its source, so that messages of the same type sent by redundant nodes can be told apart [2].

Nodes communicate mainly by message broadcasting, which follows the publish/subscribe model and serves one-to-many and many-to-many communication [2]. In addition, service invocation provides a request/response exchange between exactly two nodes and is typically used for parameter updates, file transfer and firmware update [2].

### 3.3. Layer Structure
DroneCAN builds on the physical and data-link layers of CAN, which provide the bus and the frames, and adds a transport layer and an application layer (Fig. 2).

The transport layer sends each piece of data, that is, a message, a service request or a service response, in CAN frames, splitting it over several frames when it does not fit into one. The 29-bit identifier of each frame contains the priority, the data type ID and the node ID, and the receiver puts the frames back together into the complete data [4]. The transport layer can also run over CAN FD, UDP multicast and MAVLink 2 messages [5]–[7].

The application layer defines the data structures of messages and services in the Data Structure Description Language (DSDL) and, on top of them, provides standard high-level functions such as node discovery and health monitoring [2].

The following chapters describe these layers in detail, starting with CAN.

## 4. CAN: Physical and Data-Link Layers
### 4.1. Overview of CAN
CAN (Controller Area Network) es un bus serie en el que todos los nodos están conectados en paralelo a un mismo par de cables. Una trama de datos no contiene la dirección del emisor ni la del receptor: el identificador indica el contenido de la trama. Cada nodo del bus usa el identificador para decidir si la trama le interesa. Por eso, al conectar o desconectar un nodo no hace falta reconfigurar los demás.

CAN no tiene nodo maestro: cuando el bus está libre, cualquier nodo puede transmitir. Si varios nodos transmiten a la vez, el identificador también indica la prioridad, y el nodo con mayor prioridad sigue transmitiendo. El nodo que pierde el arbitraje espera y vuelve a enviar su trama más tarde.

Sin embargo, CAN solo define cómo se transmiten las tramas, no su significado. No define cómo se asignan los identificadores, cómo se envían datos de más de 8 bytes, ni cómo se identifica un nodo o se configura la red. Todo esto lo resuelven protocolos de capas superiores, por ejemplo una capa de aplicación.

### 4.2. Capa física
#### 4.2.1. Estructura del bus y señalización
CAN usa un bus lineal: todos los nodos están conectados en paralelo a una misma línea lógica. Esta línea tiene dos estados: dominante (bit 0) y recesivo (bit 1). Si al menos un nodo transmite un bit dominante, el bus queda en estado dominante. Por tanto, el valor del bus es el AND de los valores que los nodos quieren transmitir en ese instante. Por eso, cada nodo compara el bit que envía con el bit que hay en el bus y, según la prioridad, arbitra el orden de transmisión.

El medio físico más usado es un par trenzado de dos cables, llamados CAN_H y CAN_L. El receptor mide la diferencia de tensión entre los dos cables. Una interferencia electromagnética afecta casi por igual a los dos cables, así que la diferencia casi no cambia. Por eso la transmisión diferencial resiste bien las interferencias.

#### 4.2.2. Codificación y temporización del bit
CAN no tiene una línea de reloj: cada nodo usa su propio oscilador y mide por su cuenta el tiempo de cada bit. Como los osciladores no son idénticos, el reloj de un receptor se desvía poco a poco del reloj del emisor. Para corregir esta desviación, el receptor usa los cambios de nivel del bus como referencia.

CAN codifica los bits con NRZ: como una secuencia larga de bits iguales no produce ningún cambio de nivel, y el receptor no puede corregir su reloj. Para evitarlo, CAN usa relleno de bits: después de cinco bits iguales seguidos, el emisor inserta un bit de valor contrario, y el receptor lo elimina. Así hay al menos un cambio de nivel cada seis bits.

La sincronización se hace de dos maneras. Al comienzo de cada trama, todos los nodos se sincronizan con el primer cambio de recesivo a dominante (sincronización dura). Durante la trama, cada nuevo cambio de recesivo a dominante sirve para corregir el reloj otra vez (resincronización). Para ello, cada bit incluye un pequeño tiempo reservado para la propagación de la señal y para la corrección del reloj. Esta corrección continua evita que el error se acumule. Por eso CAN puede usar osciladores baratos, y esto hace que su coste sea bajo.

### 4.3. Capa de enlace de datos
#### 4.3.1. Acceso al bus y arbitraje
Un nodo solo empieza a transmitir cuando el bus está libre. Si varios nodos empiezan a la vez, cada uno envía su identificador bit a bit, empezando por el bit más significativo, y compara cada bit enviado con el bit que lee en el bus. Si un nodo envía un bit recesivo (1) y lee un bit dominante (0), sabe que otro nodo tiene mayor prioridad. Entonces deja de transmitir, pasa a recibir la trama y lo vuelve a intentar cuando el bus esté libre. Como el bit dominante es el 0, el identificador con menor valor numérico tiene mayor prioridad. La trama ganadora no se destruye ni se retrasa. Para que el arbitraje funcione, un mismo identificador no puede ser enviado a la vez por dos nodos.

#### 4.3.2. Formato de las tramas
La trama de datos es la trama que transporta la información. Está formada por los siguientes campos: un bit de inicio, el campo de arbitraje (el identificador), el campo de control (que indica cuántos bytes de datos hay), el campo de datos, el campo CRC, el campo de confirmación (ACK) y los bits de fin de trama. El campo de datos tiene como máximo 8 bytes. Además, CAN define otras tramas: la trama remota, que pide a otro nodo que envíe una trama de datos, la trama de error y la trama de sobrecarga.

El identificador puede tener 11 bits (formato estándar, CAN 2.0A) o 29 bits (formato extendido, CAN 2.0B). Un bit del campo de arbitraje indica cuál de los dos formatos se usa. DroneCAN usa el formato extendido de 29 bits.

#### 4.3.3. Detección y gestión de errores
CAN detecta cinco tipos de errores:

- Error de bit: un nodo lee en el bus un valor distinto del que ha enviado (salvo durante el arbitraje y en el campo de confirmación).
- Error de relleno: aparecen seis bits iguales seguidos.
- Error de CRC: el CRC de 15 bits calculado por el receptor no coincide con el recibido.
- Error de formato: un campo de formato fijo tiene un valor no permitido.
- Error de confirmación: el emisor lee un bit recesivo en el campo de confirmación, es decir, ningún receptor ha recibido la trama correctamente. Los receptores que la reciben bien envían un bit dominante en ese campo.

Cuando un nodo detecta un error, envía una trama de error, formada por seis bits dominantes. Esta secuencia rompe la regla del relleno de bits, por lo que todos los nodos detectan el error y descartan la trama. Después, el emisor la retransmite automáticamente.

Para evitar que un nodo averiado perturbe el bus enviando tramas de error sin parar, CAN usa contadores de errores. Si los errores superan un límite, el nodo se aísla y se desconecta del bus (estado bus off).

### 4.4. CAN FD y otras versiones del protocolo
CAN ha evolucionado en varias versiones. CAN 2.0A usa identificadores de 11 bits. En redes con muchos tipos de mensajes y de nodos, los identificadores de CAN 2.0A no son suficientes. Por eso, CAN 2.0B añade el formato extendido, con identificadores de 29 bits (los 11 bits originales más 18 bits adicionales). Un bit de la trama indica cuál de los dos formatos se usa. Un nodo CAN 2.0B entiende los dos formatos, pero un nodo CAN 2.0A no entiende las tramas extendidas y las trata como errores.

CAN FD (Flexible Data-rate) nace porque 8 bytes por trama y una velocidad máxima de 1 Mbit/s son pocos cuando aumenta la cantidad de datos. CAN FD permite hasta 64 bytes de datos por trama y transmite el campo de datos a mayor velocidad, mientras que el arbitraje mantiene la velocidad normal. DroneCAN usa CAN clásico y también puede usarse sobre CAN FD.

### 4.5. Velocidad y longitud del bus en DroneCAN
DroneCAN usa la capa física ISO 11898-2, también llamada CAN de alta velocidad. No impone una velocidad, pero recomienda usar solo cuatro, para garantizar la compatibilidad entre fabricantes. Cuanto mayor es la velocidad, más corto es el bit y menos tiempo tiene la señal para recorrer el bus, por lo que la longitud máxima disminuye [11]:

| Velocidad (kbit/s) | Longitud máxima del bus | Longitud máxima de la derivación |
| --- | --- | --- |
| 1000 | 40 m | 0,3 m |
| 500 | 100 m | 0,3 m |
| 250 | 250 m | 0,3 m |
| 125 | 500 m | 0,3 m |

Estas longitudes suponen un retardo de propagación de 5 ns por metro, sin contar el retardo de los transceptores.

### 4.6. Resumen: lo que DroneCAN hereda de CAN
DroneCAN hereda de CAN las siguientes propiedades:

- Varios maestros: cualquier nodo puede enviar sin esperar a un maestro, y la avería de un nodo no detiene la red.
- Arbitraje por prioridad: los mensajes más importantes se envían primero.
- Detección de errores y retransmisión automática: las tramas erróneas se descartan y se retransmiten, y un nodo averiado se desconecta solo del bus (bus off).
- Difusión: cada trama llega a todos los nodos, por lo que quien envía un mensaje no necesita saber quién lo recibe.

Sin embargo, CAN solo ofrece un transporte básico de tramas. Todo lo demás, como dividir los datos largos, asignar la prioridad y el significado a los identificadores y gestionar la red, lo aportan las capas superiores de DroneCAN, es decir, la capa de transporte y la capa de aplicación.

## V. Capa de transporte de DroneCAN
### A. Tipos de transferencia: mensaje, anónimo y servicio
### B. Estructura del identificador de 29 bits
#### 1. Tramas de mensaje
#### 2. Tramas de mensaje anónimo y el discriminador
#### 3. Tramas de servicio
#### 4. Ejemplo práctico
### C. Byte de cola e ID de transferencia
### D. Transferencias de múltiples tramas
#### 1. Descomposición y reensamblaje
#### 2. CRC de transferencia y bit de conmutación
#### 3. Cálculo del recuento de tramas
### E. Reglas de temporización y tiempos de espera (timeouts)
### F. Interfaces redundantes: transmisión simultánea y deduplicación
### G. Transportes alternativos (CAN FD, UDP Multicast, MAVLink Tunnel)

## VI. DroneCAN Application Layer
### A. Data Structure Description Language (DSDL)
#### 1. Definition Files and Namespaces
#### 2. Type System and Serialisation
#### 3. Data Type Signatures and Compatibility
#### 4. Data Type IDs and Vendor-Specific Ranges
### B. Standard Data Types
### C. Standard Application-Level Functions
#### 1. Dynamic Node ID Allocation (with Raft Consensus)
#### 2. Node Status and Discovery (NodeStatus, GetNodeInfo)
#### 3. Parameter Configuration
#### 4. File Transfer and Firmware Update
#### 5. Time Synchronisation
#### 6. Debugging and Emergency Functions (Panic)

## VII. Reliability and Safety Mechanisms
### A. Fault Model (noise, corrupted frames, node failure, bus failure, malicious node)
### B. Protection Against Each Fault (table: fault, mechanism, layer)
### C. Redundancy: Interfaces, Nodes and Services
### D. Limits of the Protection (authentication, attacks, experimental attacks, implementation dependence)

## VIII. Comparison with Related Technologies
### A. DroneCAN and Cyphal
### B. DroneCAN and MAVLink
### C. DroneCAN and Point-to-Point Interfaces

## IX. Open Challenges and Research Directions
### A. Security and Intrusion Detection
### B. Quantitative Evaluation
### C. Migration to CAN FD
### D. Coexistence with Cyphal
### E. Fault Isolation on a Shared Bus
### F. Evolution of the Specifications

## X. Conclusion

## References
