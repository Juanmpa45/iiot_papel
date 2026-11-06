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

## 2. Interfaces de comunicación en los UAV
### 2.1. Arquitectura de un UAV
El núcleo de un UAV es el controlador de vuelo (FC), que se comunica con los dispositivos que lo rodean para intercambiar datos y órdenes. El hardware del controlador de vuelo ejecuta el piloto automático, es decir, el software de vuelo (flight stack) sobre un sistema operativo en tiempo real (RTOS), que estabiliza el vehículo y ejecuta las órdenes y las misiones [14]. Los dispositivos que lo rodean se dividen en cuatro grupos: sensores, actuadores, la fuente de alimentación, y los enlaces con el piloto y con otros ordenadores.

El controlador de vuelo usa salidas para accionar los actuadores: motores, superficies de vuelo como los alerones y los flaps, y cargas útiles como los disparadores de cámara o las pinzas [14]. Un motor se acciona mediante un variador electrónico de velocidad (ESC), que convierte la señal del controlador de vuelo en el nivel de potencia adecuado para el motor [14].

El vehículo suele alimentarse con una batería de polímero de litio conectada mediante un módulo de potencia, que alimenta por separado al controlador de vuelo y a los ESC [14].

El vehículo también se comunica con el piloto y con otros ordenadores. Un piloto puede volar el vehículo con un transmisor de radiocontrol (RC) y un receptor instalado en el vehículo [14]. Una estación de control en tierra (GCS), como QGroundControl, se comunica con el vehículo mediante una radio de telemetría, un enlace de datos inalámbrico bidireccional [14].

La Fig. 1 muestra los principales enlaces de comunicación del controlador de vuelo. Los actuadores y los módulos de sensores, como el módulo GNSS, se conectan directamente a puertos dedicados del controlador, y las salidas también pueden ser nodos DroneCAN, por ejemplo controladores de motor DroneCAN [14]. A la estación de tierra y al ordenador complementario se llega mediante enlaces como la radio de telemetría, un cable serie, un cable Ethernet o Wi-Fi [14].

Fig. 1. Enlaces de comunicación típicos alrededor de un controlador de vuelo.

### 2.2. Interfaces punto a punto (PWM, UART, I2C, SPI)
La forma tradicional de conectar dispositivos a un controlador de vuelo es dar a cada dispositivo su propia interfaz dedicada en el controlador, elegida según el tipo de datos que intercambia el dispositivo. Estas interfaces son interfaces de hardware y métodos de señalización, no protocolos de nivel de aplicación: definen cómo las señales eléctricas de los cables representan datos, pero no qué significan los datos. Hay cuatro de uso común. La modulación por ancho de pulso (PWM) envía una consigna a un actuador por un solo cable. Un UART (receptor-transmisor asíncrono universal) es un enlace serie entre dos dispositivos. I2C y SPI son buses serie de corta distancia, en los que el controlador se comunica con varios chips o módulos periféricos mediante cables compartidos. Por tanto, PWM y UART conectan exactamente dos dispositivos, mientras que I2C y SPI comparten los cables entre varios, pero en los cuatro casos la conexión es local, limitada a los puertos del propio controlador, y ningún direccionamiento ni servicio común se extiende por todo el vehículo.

PWM es la más simple y se usa para los actuadores. Codifica una consigna, como la velocidad deseada del motor, en el ancho de los pulsos de una señal periódica, de modo que un cable transporta una consigna en un solo sentido, desde el controlador hasta un actuador. Los conectores de salida del controlador se llaman puertos; en PX4 son MAIN y AUX [14]. El número de salidas lo fija el hardware, normalmente entre seis y ocho [14].

Las interfaces serie son adecuadas para dispositivos que intercambian datos, en lugar de recibir una sola consigna. Los enlaces UART se usan para las radios de telemetría y los ordenadores complementarios [14], mientras que I2C y SPI se usan para sensores y otros periféricos situados cerca del controlador, y SPI es la más rápida de las dos. Las tres son simples y tienen un amplio soporte. Su principal límite es el alcance: I2C y SPI están pensados para conexiones cortas, y un UART conecta solo dos dispositivos.

Por tanto, el cableado punto a punto tiene tres limitaciones:

El número de puertos lo fija el hardware [14], así que cada dispositivo adicional necesita otro puerto y otro cable.
Cada tipo de interfaz necesita su propio controlador (driver) y su propio cableado.
Las interfaces no definen servicios comunes, como el descubrimiento de dispositivos, la supervisión del estado o la actualización del firmware [9].

### 2.3. MAVLink: comunicación entre el vehículo y otros ordenadores
Las interfaces descritas antes conectan dispositivos individuales con el controlador de vuelo. MAVLink resuelve un problema distinto: la comunicación entre el vehículo y los otros ordenadores que lo supervisan o lo ayudan, es decir, la estación de control en tierra y el ordenador complementario [16], [18]. Así, las interfaces punto a punto sirven para la comunicación entre el controlador de vuelo y los dispositivos del interior del vehículo, mientras que MAVLink sirve para la comunicación entre el vehículo y otros ordenadores. DroneCAN pertenece al primero de estos dos problemas: conecta el controlador de vuelo y los dispositivos de a bordo mediante un bus CAN compartido [12], [19].

### 2.4. Resumen de requisitos
Las limitaciones del cableado punto a punto llevan directamente a los siguientes requisitos para un bus dentro del vehículo:

un medio compartido al que se conecten muchos dispositivos mediante una sola interfaz, de modo que el número de dispositivos no esté limitado por el número de puertos del controlador y cada dispositivo adicional no necesite otro cable;
una sola interfaz y un solo tipo de cableado para todos los tipos de dispositivos, de modo que un único controlador (driver) sirva para todos;
definiciones estándar de las funciones de alto nivel comunes a todos los nodos, como el descubrimiento, la supervisión del estado, la configuración y la actualización del firmware.

Usar un bus así para el control de un vehículo con hardware embebido pequeño añade otros requisitos:

ningún punto único de fallo y, por tanto, ningún maestro del bus (un nodo que controla el acceso de todos los demás);
soporte para interfaces y nodos redundantes;
baja latencia y alto caudal de datos para bucles de control de tiempo real estricto, es decir, bucles de control que deben cumplir plazos de tiempo estrictos;
una forma sencilla de intercambiar datos que superan el tamaño de una sola trama;
bajos requisitos de cómputo, hasta microcontroladores con recursos limitados;
una especificación abierta con implementaciones de referencia abiertas.

Estos requisitos llevan a los objetivos de diseño y a la arquitectura de DroneCAN [1].

## 3. Objetivos de diseño y arquitectura de DroneCAN
### 3.1. Objetivos de diseño
El bus CAN ya elimina las dos primeras limitaciones del cableado punto a punto: el número de dispositivos ya no está limitado por el número de puertos del controlador, y una sola interfaz y un solo tipo de cableado sirven para todos. Sin embargo, CAN no define servicios comunes, y compartir un único medio entre todos los dispositivos trae sus propios problemas, como los puntos únicos de fallo, los datos que superan el tamaño de una trama, y las exigencias de redundancia y baja latencia. Los objetivos de diseño de DroneCAN responden a estos dos grupos de problemas [1].

Para cubrir la falta de servicios comunes que CAN deja sin definir, DroneCAN define mensajes y servicios estándar para las funciones que necesita todo vehículo: descubrir los nodos de la red, configurar un nodo, actualizar su firmware, supervisar su estado, sincronizar relojes y asignar ID de nodo de forma automática [1]. El protocolo se mantiene simple y con bajos requisitos de cómputo, porque los dispositivos del bus van desde microcontroladores con muy poca memoria y capacidad de procesamiento hasta ordenadores de alto rendimiento con Linux [1]. La especificación es totalmente abierta, y sus implementaciones de referencia son de código abierto con licencia MIT [1].

Los demás objetivos responden a los problemas de compartir un solo medio. Un nodo maestro que controlara el acceso al bus sería un punto único de fallo, así que DroneCAN no tiene maestro y da a todos los nodos los mismos derechos de comunicación. Una trama CAN clásica lleva como máximo 8 bytes de datos [4], que son muy pocos para las soluciones GNSS y los vectores tridimensionales que necesita un vehículo, así que DroneCAN divide esos datos en varias tramas y los reconstruye en el receptor; la aplicación entrega y recibe estructuras de datos completas y no tiene que ocuparse de esto. Un fallo en el bus compartido afecta a todos los dispositivos, así que DroneCAN también admite interfaces y nodos redundantes. Por último, como todos los dispositivos comparten el mismo ancho de banda, los datos de los bucles de control críticos en el tiempo deben seguir llegando rápido, así que DroneCAN busca un alto caudal de datos y una baja latencia [1].

### 3.2. Modelo de red y métodos de comunicación
Una red DroneCAN es una red descentralizada de nodos iguales, en la que cada dispositivo del bus tiene un ID de nodo numérico único, con valores válidos de 1 a 127 [2], [4]. Cada mensaje y cada servicio tiene un ID de tipo de datos que identifica su estructura de datos, y el ID de nodo de un mensaje identifica su origen, de modo que se pueden distinguir los mensajes del mismo tipo enviados por nodos redundantes [2].

Los nodos se comunican principalmente mediante la difusión de mensajes, que sigue el modelo de publicación/suscripción y sirve para la comunicación de uno a muchos y de muchos a muchos [2]. Además, la invocación de servicios ofrece un intercambio de petición/respuesta entre exactamente dos nodos, y se usa normalmente para actualizar parámetros, transferir archivos y actualizar el firmware [2].

### 3.3. Estructura de capas
DroneCAN se construye sobre la capa física y la capa de enlace de datos de CAN, que proporcionan el bus y las tramas, y añade una capa de transporte y una capa de aplicación (Fig. 2).

La capa de transporte envía cada dato, es decir, un mensaje, una petición de servicio o una respuesta de servicio, en tramas CAN, y lo divide en varias tramas cuando no cabe en una sola. El identificador de 29 bits de cada trama contiene la prioridad, el ID de tipo de datos y el ID de nodo, y el receptor vuelve a unir las tramas para obtener el dato completo [4]. La capa de transporte también puede funcionar sobre CAN FD, multidifusión UDP y mensajes MAVLink 2 [5]–[7].

La capa de aplicación define las estructuras de datos de los mensajes y los servicios en el lenguaje de descripción de estructuras de datos (DSDL) y, sobre ellas, ofrece funciones estándar de alto nivel, como el descubrimiento de nodos y la supervisión del estado de salud [2].

Los capítulos siguientes describen estas capas en detalle, empezando por CAN.

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
