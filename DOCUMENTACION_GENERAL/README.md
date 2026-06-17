# TABLA DE CONTENIDO

# 1. RESUMEN DE MICROCONTROLADORES

En esta sección destallamos los principales microcontroladores.

Hay dos familias, los basados en chips ESP8266 y ESP32

A continuación la diferencia entre los dos tipos de familia de procesadores:

- ESP-01: Aunque comparte el chip interno con sus hermanos mayores, físicamente te limita muchísimo por tener solo 2 pines GPIO cómodos (GPIO0 y GPIO2). Se suele usar más como módulo Wi-Fi esclavo para Arduino o para relés muy simples.
- Wemos D1 (Mini): Es el rey de las placas compactas basadas en ESP8266. Tené en cuenta que comparte las mismas limitaciones de memoria y Wi-Fi que el NodeMCU, pero en un formato súper amigable para prototipos chicos.
- ESP32 C3 Supermini: Esta placa es un golazo reciente. Cambia la arquitectura tradicional de Tensilica por RISC-V. Es una opción excelente si buscás el tamaño diminuto de un Wemos D1 pero con la potencia, la seguridad y el Bluetooth Low Energy (BLE) de la generación ESP32.

## 1.2 PRECIOS DE MICROCONTROLADORES

En la siguiente tabla mencionamos los principales procesadores y sus precios estimados como referencia:

| **MICROCONTROLADOR**                     | **PRECIO**<br><br>**(AR\$)** | **PRECIO (u\$d)** |
| ---------------------------------------- | ---------------------------- | ----------------- |
| ESP8266 - ESP01                          | 5.718                        | 4,00              |
| ESP8266 - WEMOS D1 (NodeMCUMini)         | 6.500                        | 4,50              |
| ESP8266 - NODEMCU v3 (ESP12) CH 340      | 5.690                        | 4,00              |
| ESP32 - DEVKIT V1 - DOIT (30 o 38 pines) | 16.500                       | 11,50             |
| ESP32 - C3 Supermini                     | 5.999                        | 4,20              |

Análisis de Prestaciones / Precio

En términos de costo, la brecha se ha cerrado muchísimo. En 2026, un ESP8266 puede costar alrededor de 3 - 5 u\$d/unidad, mientras que un ESP32 estándar ronda los 6 - 9 u\$d/unidad.

1\. ¿Por qué el ESP32 suele ser la mejor inversión?

- Multitarea Real: Al tener dos núcleos, puedes dedicar uno exclusivamente a mantener la conexión Wi-Fi estable y el otro a leer sensores o controlar luces. En el ESP8266, si tu código es muy pesado, el Wi-Fi puede desconectarse.
- Bluetooth Low Energy (BLE): Es vital para configurar tus dispositivos desde el celular sin necesidad de conectarte primero a su red Wi-Fi (modo Access Point), lo que mejora mucho la experiencia de usuario.
- Más Pines: En domótica, pronto te quedas sin pines en un ESP8266 si quieres conectar una pantalla OLED, un sensor de temperatura y un par de relés. El ESP32 te sobra en este aspecto.

2\. ¿Cuándo elegir el ESP8266?

- Costo Extremo: Si vas a fabricar 50 sensores de temperatura simples (solo lectura y envío), ahorrar esos \$3 dólares por unidad hace la diferencia.
- Consumo de Batería: Para proyectos extremadamente simples que duermen la mayor parte del tiempo (Deep Sleep), el ESP8266 sigue siendo muy eficiente, aunque los nuevos modelos ESP32-C3 ya compiten directamente en esto.

Veredicto para tu casa: ¿Cuál conviene?

- Para proyectos de IoT en el hogar, el ESP32 es el claro ganador.

La facilidad de tener Bluetooth para la configuración inicial y la potencia extra para manejar protocolos modernos como Matter o librerías pesadas (como las de Home Assistant / ESPHome) harán que tu proyecto sea mucho más estable y "a prueba de futuro".

# 2. HOME ASSISTANT

## 2.1 NOMENCLATURA

Esta es una de las preguntas más importantes para mantener la cordura a medida que tu casa se vuelve más inteligente. Nombrar los dispositivos por el tipo de placa (ej. "ESP32-C3-01") es un error común al principio, pero a largo plazo se vuelve un caos porque al usuario final (o a tu familia) no le importa qué chip hay adentro, sino qué hace y dónde está.

Aquí tienes la mejor estrategia de nomenclatura basada en las mejores prácticas de Home Assistant y el estándar de "Nombres Funcionales":

### 2.1.1 Regla de Oro: Estructura Jerárquica

- Lo ideal es seguir este patrón para el Friendly Name (el nombre que ves en la pantalla):
- \`\[Habitación\] \[Función/Objeto\] \[Atributo\]\`
- Ejemplos:
  - \`Cuarto Nicolas Luz Techo\`
  - \`Cocina Sensor Movimiento\`
  - \`Patio Bomba Riego\`
- ¿Por qué no usar el tipo de placa?
- Si mañana se quema tu ESP32-C3 y lo reemplazas por un ESP32-S3 o una placa de otra marca, tendrías que renombrar todo en Home Assistant, lo que rompería tus automatizaciones y paneles. Si el dispositivo se llama "Sensor Temperatura Cuarto", podés cambiar el hardware mil veces y la automatización seguirá funcionando.

### 2.1.2 Diferencia entre "Name" y "Entity ID"

- Es fundamental que entiendas cómo maneja Home Assistant estos dos campos:

| CAMPO         | EJEMPLO                    | FUNCION                                                 |
| ------------- | -------------------------- | ------------------------------------------------------- |
| FRIENDLY NAME | Temperatura Cuarto Nicolas | Es lo que aparece en la app y en Google / home Alexa    |
| ENTITY ID     | sensor.temp_cuarto_nico    | Es el "nombre interno" que usas en las automatizaciones |
|               |                            |                                                         |

- Tip Pro: En el archivo YAML de ESPHome, tratá de que el \`name\` sea el nombre amigable. Home Assistant generará el ID automáticamente eliminando espacios y mayúsculas.

### 2.1.3 ¿Dónde guardo la info técnica (tipo de placa)?

- No la pongas en el nombre del sensor. Usá estas herramientas:
- Nombre del Nodo en ESPHome: Ahí sí podés ser un poco más técnico. Por ejemplo: \`nodo_c3_cuarto_01\`.
- Áreas en Home Assistant: Asigná el dispositivo al área "Cuarto Nicolas". Esto te permite decir "Apaga todo en el cuarto de Nicolás" y HA sabrá qué hacer.
- Atributos y Etiquetas (Tags): Home Assistant permite agregar etiquetas. Podés crear una etiqueta que diga "ESP32-C3" para filtrar todos los dispositivos que usan ese chip si necesitás hacer un mantenimiento masivo.

### 2.1.4 Estrategia Sugerida para tus proyectos

- Basado en lo que estás armando (el sensor de movimiento, temperatura y el caudalímetro), te sugiero este esquema:
- Para el Cuarto:
- Dispositivo: \`Cuarto Nicolas Multi\` (así se llama el ESP32-C3 completo).
- Sensores:
- \`Movimiento Cuarto Nicolas\`
- \`Temperatura Cuarto Nicolas\`
- \`Luz Estado Nodo Cuarto\` (para el LED azul).
- Para el Caudalímetro:
- Dispositivo: \`Control Agua Principal\`
- Sensores:
- \`Caudal Agua Total\`
- \`Consumo Agua Instantaneo\`

### 2.1.5 El "Prefijo de Seguridad"

- Si tenés muchos dispositivos similares, usá un prefijo corto para los Entity IDs.
- Ejemplo: \`sn\` (Sensor), \`sw\` (Switch), \`lt\_\` (Light).
- \`sensor.sn_temp_cuarto_nico\`

Resumen: Nombra por Ubicación + Función. Es la única forma de que, dentro de dos años, sepas qué es cada cosa sin tener que abrir la caja para ver qué placa tiene adentro.

# 3. MICROCONTROLADORES

Un **microcontrolador** es, en esencia, una **computadora completa contenida en un solo circuito integrado** (chip). A diferencia de un microprocesador (como el que tiene tu computadora, que solo ejecuta instrucciones), un microcontrolador está diseñado para realizar tareas específicas y repetitivas dentro de dispositivos electrónicos.

### ¿Qué compone a un microcontrolador?

Para que pueda funcionar de forma autónoma, el chip integra en su interior tres componentes fundamentales que en una computadora convencional se encuentran por separado:

* **CPU (Unidad Central de Procesamiento):** El cerebro que ejecuta las instrucciones del programa.
* **Memoria:** * **Flash:** Donde se almacena el código (programa) que el usuario escribe.
* **RAM:** Donde se guardan temporalmente los datos durante la ejecución.


* **Periféricos de entrada/salida (I/O):** Son los "puertos" que le permiten comunicarse con el mundo exterior, como sensores, pantallas, botones o motores.

---

### Diferencias clave: Microprocesador vs. Microcontrolador

| Característica | Microprocesador (CPU) | Microcontrolador (MCU) |
| --- | --- | --- |
| **Uso** | Propósito general (computadoras, servidores) | Propósito específico (electrodomésticos, sensores) |
| **Integración** | Requiere componentes externos (RAM, almacenamiento) | Todo está integrado en un solo chip |
| **Costo** | Alto | Bajo |
| **Consumo de energía** | Elevado | Muy bajo |

---

### Aplicaciones Comunes

Los microcontroladores están presentes en casi todo lo que tiene un botón o una pantalla:

* **Automotriz:** Control del motor, sistemas de frenado ABS, control de ventanas.
* **Electrodomésticos:** Microondas, lavadoras, cafeteras programables.
* **Industria:** Controladores lógicos programables (PLC), sensores de temperatura y presión.
* **IoT (Internet de las Cosas):** Dispositivos domésticos inteligentes (enchufes wifi, termostatos).

En resumen, **un microcontrolador es el componente "inteligente"** que le da autonomía y control a los dispositivos electrónicos modernos, funcionando con una eficiencia energética y un costo mucho menores que los procesadores de alto rendimiento.

### Familias de Microcontroladores
Las familias de microcontroladores son conjuntos de chips que comparten la misma arquitectura interna (el "idioma" que habla su procesador) y conjunto de instrucciones.

Elegir una familia depende principalmente de la potencia necesaria, el consumo de energía y la facilidad de desarrollo.Aquí te presento las familias más relevantes y utilizadas en la industria actual:

- AVR (Atmel / Microchip)Es una de las arquitecturas más populares, especialmente conocida por ser el "corazón" de la plataforma Arduino.
  - Características: Arquitectura de 8 bits, muy robusta y con una curva de aprendizaje sencilla.
  - Ideal para: Proyectos de electrónica básica, prototipado rápido y aplicaciones sencillas que no requieren gran velocidad
- ARM Cortex-M (Arquitectura ARM)Actualmente es el estándar de la industria. No es un fabricante en sí, sino una arquitectura que muchas empresas (como STMicroelectronics, NXP o Texas Instruments) licencian y fabrican.
  - Características: De 32 bits, muy alta eficiencia energética y potencia de procesamiento superior.
  - Subfamilias:
    - Cortex-M0/M0+: Para aplicaciones de ultra bajo consumo.
    - Cortex-M4/M7: Con capacidades de pr  ocesamiento de señales (DSP) y punto flotante, ideales para audio o control de motores complejo.
  - Ideal para: Productos comerciales, dispositivos médicos, wearables y sistemas industriales.
- ESP32 / ESP8266 (Espressif Systems)Se han convertido en el estándar "de facto" para proyectos conectados a internet gracias a su increíble relación costo-beneficio.
  - Características: Integran Wi-Fi y Bluetooth nativos en el mismo chip. Son extremadamente rápidos comparados con los AVR tradicionales.
  - Ideal para: Internet de las Cosas (IoT), domótica, control remoto y dispositivos conectados a la nube.
- PIC (Microchip)Una de las familias con más historia y presencia en la industria pesada y automotriz.
  - Características: Son famosos por su gran variedad: existen desde modelos diminutos de 6 pines hasta procesadores complejos de 32 bits. Son muy resistentes al ruido eléctrico.
  - Ideal para: Entornos industriales hostiles, sistemas de control automotriz y electrodomésticos de consumo masivo.

### Resumen Comparativo
  
  |Familia|Arquitectura|Fortalezas|Uso principal
  |:---|:---|:---|:---
  |AVR|8 bits|Sencillez, gran comunidad|Hobby, educaciónARM 
  |Cortex-M|32 bits|Potencia, escalabilidad|Productos profesionales
  |ESP32|32 bits|Conectividad (Wi-Fi/BT)|IoT, domótica
  |PIC|8/16/32 bits|Robustez, versatilidad|Industria, automotriz
  
### ¿Cómo elegir?
Para seleccionar una familia, debes hacerte tres preguntas:

- ¿Necesito conectividad inalámbrica?

  Si la respuesta es sí, el ESP32 suele ser la opción más económica y fácil de implementar.

- ¿Es un producto profesional para fabricar en serie?

  En ese caso, los ARM Cortex-M ofrecen la mejor relación rendimiento/costo a largo plazo.

- ¿Es para aprender o un prototipo sencillo?

  AVR (Arduino) te permitirá avanzar más rápido gracias a la enorme cantidad de librerías y tutoriales disponibles.

## 3.1 ESP8266 - ESP01

El Modulo WiFi ESP01 basado en el chip ESP8266 que permite conectar tus proyectos a Internet mediante una red Wifi de forma fácil y económica.

Pude servir en 2 modos: como WiFi Station o como Access Point. Al trabajar WiFi Station se conecta a una red, mientras que en en modo Access Point se usa para "crear" una red wifi propia y acceder a ella para controlar un motor, luces o incluso obtener lectura de sensores digitales.

### 3.1.1 Especificaciones

- Tensión Nominal: 3.3V DC
- Tensión Maxima: 3.7V DC
- Integrado: ESP8266EX
- Comunicación de tipo: Serial, UART
- CPU: 32 Bits
- RAM: 32KB
- Data RAM: 96KB
- Memoria Flash: 1MB
- Soporte 3 modos: AP, STA, AP + STA
- Protocolos: 802.11 b/g/n - TCP/IP
- Red Wif de tipo: 2.4Ghz
- Dimensiones: 24 mm x 14 mm
- Peso: 3g

### 3.1.2 Esquema de conexiones (PIN OUT)

![ESP01](../assets/PO_ESP01.png)

## 3.2 ESP8266 - WEMOS D1 (NodeMCUMini - ESP8266)

El NodeMCUMINI es una placa de desarrollo que está basada en el popular chip ESP8266. Con este sencillo modulo se puede realizar el prototipo de cualquier sistema para el loT.

![WEMOS D1](../assets/WEMOS-D1.png)

### 3.2.1 Especificaciones

- Utiliza el conversor USB a UART CH340G
- Open-source
- Procesador principal ESP8266 ESO-12E o ESO-12F con 4Mbytes
- Stack TCP/IP integrado
- Potencia de salida +25dBm en modo 802.11b
- Corriente en reposo: <10uA
- USB-TTL incluido, plug & play
- 1 ADC con maximo de 3.3V
- 9 GPIO, cada GPIO puede ser PWM 12C 1-Wire
- LDO onboard
- Hilera de pines de 2x 8 pines de 2.56 mm compatible con protoboard
- Boton de reset
- Conector micro USB
- Antena PCB

### 3.2.1 Esquema de conexiones (PIN OUT)

![WEMOS D1](../assets/PO_WEMOS-D1.png)

## 3.3 ESP8266 - NODEMCU V3 CH340

El NodeMcu V3 CH340 es una placa de desarrollo de código abierto basada en el chip ESP8266 (ESP-12E), que utiliza el lenguaje de programación Lua para crear un ambiente de desarrollo propicio para aplicaciones que requieran conectividad Wifi de manera rápida.

El ESP8266 es un chip altamente integrado diseñado para las necesidades de un nuevo mundo conectado. Ofrece una solución completa y autónoma de redes Wi-Fi, lo que le permite alojar la aplicación o servir como puente entre Internet y un microcontrolador.

El ESP8266 tiene potentes capacidades a bordo de procesamiento y almacenamiento que le permiten integrarse con sensores y dispositivos específicos de aplicación a través de sus GPIOs con un desarrollo mínimo y carga mínima durante el tiempo de ejecución. Su alto grado de integración en el chip permite una circuitería externa mínima, y la totalidad de la solución, incluyendo el módulo, está diseñado para ocupar el área mínima en un PCB.

Esta versión incluye un pin de salida (VU) de 5V proveniente directamente de la conexión USB, utiliza como conversor FTDI el chip CH340G (requiere driver, link de descarga en la documentación) y sus pines permiten que se monte fácilmente en un protoboard.

Se puede alimentar un NodeMCU ESP8266 con 5V, pero es fundamental conectarlo al pin correcto para no dañar el módulo. Aunque el chip ESP8266 funciona internamente a 3.3V, la placa NodeMCU incluye un regulador de voltaje (generalmente el AMS1117) que se encarga de reducir los 5V a los 3.3V necesarios.

Opciones para alimentar con 5V:

- Por el puerto Micro-USB: Es la forma más segura y común. Puedes usar un cargador de celular, un power bank o el puerto USB de una computadora. El regulador interno gestionará la conversión de energía.
- Por el pin VIN (o 5V): Si tienes una fuente de alimentación externa de 5V (como una fuente conmutada o un regulador externo), debes conectar el polo positivo al pin marcado como VIN (en algunas versiones figura como 5V) y el negativo al pin GND.

No usar el pin 3V3: Nunca conectes 5V directamente al pin marcado como 3.3V o 3V3. Este pin se salta el regulador y va directo al chip; si le aplicas 5V, quemarás el ESP8266 instantáneamente.

### Lógica de los Pines (I/O): 

Aunque alimentes la placa con 5V, los pines de entrada/salida (GPIO) siguen funcionando a 3.3V. No conectes sensores o señales de 5V directamente a los pines digitales sin un divisor de tensión o un conversor de niveles lógicos.

Corriente: Asegúrate de que tu fuente pueda entregar al menos 500mA. El ESP8266 tiene picos de consumo altos cuando utiliza la conexión Wi-Fi.

Una diferencia importante es que Arduino se programa en C++ y NodeMCU en LUA, un lenguaje script que permite cargar scripts al NodeMCU para que corran en el inicio y mandarle comandos directamente para ejecutar funciones mediante una consola o Telnet.

NodeMCU es una pequeña placa Wifi compatible con Arduino lista para usar en cualquier proyecto IoT. Está montada alrededor del ESP8266 y expone todos sus pines en los laterales. Además, ofrece más ventajas como la incorporación de un regulador de tensión integrado, así como un puerto USB de programación. Se puede programar con LUA o mediante el IDE de Arduino. Dispone de una extensa comunidad y documentación que permiten conectar el proyecto al mundo exterior mediante conexión Wifi.

Debido a que utiliza un conversor USB CH340, normalmente el sistema operativo lo instala automáticamente, aunque en algunos casos puede ser necesario instalar el driver específico.

![NODEMCU ESP8266](../assets/ESP8266_NODEMCU_v340.png)

### 3.3.1 Especificaciones

- Procesador: ESP8266 @ 80MHz (3.3V) (ESP-12E)
- 4MB de memoria FLASH (32 MBit)
- Wifi 802.11 b/g/n
- Alimentación: 5V por usbc o por pin VIN
- Regulador 3.3V integrado (500mA)
- Conversor USB-Serial CH340G / CH340G
- Función Auto-reset
- 9 pines GPIO con I2C y SPI
- 1 entrada analógica (1.0V Max)
- 4 agujeros de montaje (3mm)
- Pulsador de RESET
- Dimensiones: 57 x 31 mm
- Peso: 11 g

### 3.3.4 Esquema de conexiones (PIN OUT)


![ESP8266 NODEMCU](../assets/PO_ESP8266_Nodemcu.png)


## 3.4 ESP32 - DEVKIT V1 - DOIT

Placa de desarrollo ESP32 WIFI Bluetooth redes componentes inteligentes ESP-WROOM-32 ESP-32. El ESP32 se puede programar en diferentes entornos de programación. Puedes utilizarlo: en Arduino IDE, Espressif IDF (Marco de Desarrollo IoT), Micropitón, SIM, LUA

![ESP32 DEVKIT](../assets/ESP32-30.png)

### 3.4.1 Especificaciones

- El ESP32 es de doble núcleo, lo que significa que tiene 2 procesadores.
- Tiene Wi-Fi integrado y compatible con bluetooth.
- Ejecuta programas de 32 bits.
- La frecuencia del reloj puede llegar hasta 240MHz y tiene una RAM de 512 kB.
- Esta placa en particular tiene 30 pines, 15 en cada fila.
- También tiene una amplia variedad de periféricos disponibles, como: tactil capacitivo, ADCs, DACs, UART, SPI, I2C y mucho mas.
- Viene con sensor de efecto hall integrado y sensor de temperatura incorporado.

### 3.4.2 Periféricos

- 18 canales de convertidor analógico a Digital (ADC)
- 3 interfaces SPI
- 3 interfaces UART
- 2 interfaces I2C
- 16 canales de salida PWM
- 2 convertidores digitales a analógicos (DAC)
- 2 interfaces I2S
- 10 GPIOs de detección capacitiva

### 3.4.3 Esquema de Conexiones 30 GPIOs (PIN OUT)

![ESP32](../assets/PO_ESP32-30.png)

### 3.4.4 Esquema de conexiones 36 GPIOs (PIN OUT)

![ESP32](../assetS/PO_ESP32-36.png)


## 3.5 ESP32 - C3 Supermini

Este NodeMCU ESP32-C3 SuperMini de 16 pines es una solución compacta, potente y eficiente para proyectos de electrónica e IoT. Equipado con conectividad WiFi 2.4GHz y Bluetooth 5.0 LE, este módulo permite desarrollar dispositivos inalámbricos sin complicaciones, manteniendo un diseño ultra compacto y fácil de integrar.

![ESP32](../assets/ESP32_C3.png)

### 3.5.1 Características

- WiFi 2.4GHz y Bluetooth 5.0 LE para una conectividad inalámbrica avanzada.
- Microcontrolador ESP32-C3 con arquitectura RISC-V de 32 bits, ideal para tareas de alto rendimiento.
- Conectividad inalámbrica avanzada: WiFi 802.11 b/g/n y Bluetooth 5.0 LE.
- Conector USB Tipo-C integrado para programación directa.
- Utiliza chip CH340 como conversor USB-Serial
- Seguridad por hardware: soporta cifrado AES, SHA, RSA y ECC.
- Compatible con Arduino, Micro Python, ESP-IDF, y otras plataformas populares
- Pines GPIO multifunción: PWM, I2C, SPI, UART, ADC y más.

### 3.5.2 Especificaciones

- Microcontrolador:ESP32-C3 SuperMini
- Frecuencia del reloj: 160 MHz
- Memoria Flash: 4 MB
- SRAM: 400 KB
- Voltaje de funcionamiento: 3.3V
- Voltaje de entrada recomendado: 5V (Por USB C)
- Voltaje de entrada máximo: 5.5V
- Entradas analógicas (ADC): Hasta 5 Canales
- Conectividad: Bluetooth, WiFi
- Cantidad de Pines Físicos: 16
- Dimensiones Físicas: 22.5 mm x 18 mm
- Grosor de la PCB: 1.2 mm (sin contar los componentes soldados)
- Espaciado de pines (Pitch): 2.54 mm (estándar para protoboard

### 3.5.3 Esquema de conexiones (PIN OUT)

![ESP 32 C3](../assets/PO_ESP32-C3%20Supermini.png)

### 3.5.4 Montaje en Protoboard (Prototipado)

- Es la forma más sencilla. Debido a que tiene un espaciado estándar de 2.54 mm, puedes soldar los pines (headers) hacia abajo y pincharla en la protoboard.
- Ventaja: La placa queda firme y te permite conectar sensores rápidamente.
- Tip: Al ser tan corta (solo 8 pines por lado), deja mucho espacio libre en la protoboard para otros componentes.

### 3.5.5 Uso de Zócalos o "Headers" Hembra

- Si vas a montar la placa sobre otra PCB o una placa perforada, lo ideal es usar headers hembra.
- Soldas dos tiras de 8 pines hembra en tu base y simplemente "enchufas" el SuperMini.
- Esto te permite retirar la placa fácilmente si se quema o si necesitas reprogramarla fuera del circuito.

### 3.5.6 Fijación Mecánica (Gabinete 3D)

- Como el SuperMini no tiene orificios de montaje para tornillos (debido a su tamaño), se suele sujetar mediante presión o guías:
- Pestañas de presión: Diseñar un gabinete con pequeñas pestañas que "abracen" los bordes de la PCB.
- Rieles laterales: Crear ranuras de 1.3 mm de ancho donde la placa se deslice.
- Soporte por conectores: A veces se fija simplemente dejando que el conector USB-C asome por un hueco ajustado, lo que le da un punto de anclaje.

### 3.5.7 Soldadura Directa (Surface Mount)

- Si necesitas que el perfil sea lo más bajo posible (por ejemplo, para un sensor de bolsillo), puedes soldar la placa como si fuera un componente SMD.
- El SuperMini suele tener pads almenados (huecos en los bordes de los contactos). Esto permite apoyarlo plano sobre otra superficie y soldar los bordes directamente.

### 3.5.8 Notas Técnicas para el C3

- Alimentación: El regulador del SuperMini es pequeño. Si el módulo LoRa intenta transmitir a máxima potencia (+20dBm), podría causar caídas de tensión que reinicien el ESP32. Si esto ocurre, intenta bajar la potencia en la configuración o colocar un capacitor de 100µF entre 3.3V y GND.
- DIO0: Este pin es obligatorio en ESPHome para que el componente funcione correctamente, ya que gestiona las interrupciones de recepción.
- Pines SPI: He seleccionado los pines 4, 5 y 6 porque son los pines por defecto del hardware SPI en muchas placas C3, lo que asegura el mejor rendimiento.
  
# 4. COMUNICACION (Conectividad)

Los modulos de comunicación es la forma en la cual se "hablan" los dispositivos. 

Esta categoría es "el corazón" de tu proyecto:
Si un sensor falla, solo dejas de ver un dato. Si un módulo de comunicación falla, te quedas "a ciegas" con todo un sector de la casa. 

|Categoría |Módulo  Principal| Función|
|:---|:---|:---
Conectividad|ESP32 / ESP8266| Nodo principal (Wi-Fi/Bluetooth)
|Conectividad|NRF24L01|Comunicación inalámbrica de bajo consumo (2.4GHz)
|Conectividad|HC-05 / HC-06|Comunicación serie vía Bluetooth
|Conectividad|Módulos LoRa|Largo alcance para sensores distantes

ASi se comunican los módulos

- ESP32/ESP8266: Son tus nodos principales. Se encargan de leer los sensores y enviar la info a Home Assistant vía Wi-Fi.

- Protocolos de comunicación: Aquí es donde documentas si el dispositivo usa MQTT (muy recomendado para Home Assistant), HTTP, o API nativa.

Por eso, en esta sección te recomiendo anotar siempre:

- Dirección IP o Identificador: Si usas IP estáticas, anótalas aquí.
- Protocolo: (Ej: "Este dispositivo se comunica mediante el broker MQTT").
- Antena: (Si es antena integrada o externa, ya que eso afecta el alcance).

## 4.1 LORA 01 FSK 00/K

La placa LoRa-01, basada en el chip SX1278 de Semtech, es un módulo transceptor de radiofrecuencia diseñado para comunicaciones de largo alcance y bajo consumo. Aunque su nombre destaca "LoRa", su versatilidad radica en que soporta múltiples modulaciones, incluyendo FSK (Frequency Shift Keying).

![LORA](../assets/Comunicacion_LORA01FSK.png)

Aquí te detallo sus características principales:

### 4.1.1 Modulaciones Soportadas

A diferencia de otros módulos más simples, la LoRa-01 es altamente flexible:

- LoRa™: Su característica estrella para largo alcance con alta inmunidad al ruido.
- FSK / GFSK: Para transmisiones de datos más convencionales y rápidas a distancias menores.
- MSK / GMSK / OOK: Modulaciones adicionales para compatibilidad con sistemas legados.

### 4.1.2 Especificaciones de Radiofrecuencia

- Frecuencia: Opera generalmente en la banda de 433 MHz (ISM), común en dispositivos de baja potencia.
- Sensibilidad: Puede alcanzar hasta -148 dBm, lo que le permite recibir señales extremadamente débiles que quedarían enterradas en el ruido para otros chips.
- Potencia de Salida: Máximo de +20 dBm (100 mW), ajustable por software para optimizar el consumo de batería.

### 4.1.3 Interfaz y Consumo

- Comunicación: Utiliza una interfaz SPI estándar, lo que la hace compatible con casi cualquier microcontrolador (ESP32, Arduino, STM32).
- Bajo Consumo: En modo "Sleep" consume apenas 0.2 µA, ideal para sensores que funcionan con baterías por años.
- Voltaje de Operación: Funciona entre 1.8V y 3.7V.

### 4.1.4 Características de Datos

- Packet Engine: Incluye un motor de paquetes con CRC (verificación de errores) de hasta 256 bytes.
- Bit Rate:
  - En FSK: Hasta 300 kbps.
  - En LoRa: Velocidades mucho menores (aprox. 0.018 a 37.5 kbps), priorizando la distancia sobre la velocidad.

### 4.1.5 Diferencia clave: FSK vs LoRa

En el contexto de tu consulta sobre el modo FSK, es importante notar que al usar esta placa en FSK en lugar de LoRa:

- Aumentas la velocidad de transmisión (útil para enviar ráfagas de datos más grandes).
- Reduces el alcance significativamente comparado con el espectro ensanchado de LoRa.
- Pierdes la inmunidad al ruido característica del protocolo LoRa,

### 4.1.6 Esquema de Conexión (Pinout)

- Para conectar un módulo LoRa-01 (SX1278) a un ESP32, debes utilizar el bus de comunicación SPI. Es fundamental recordar que ambos dispositivos trabajan a 3.3V, por lo que no necesitas convertidores de nivel lógico, pero sí una fuente de alimentación estable.
- La mayoría de las librerías para ESP32 (como la de Sandeep Mistry) utilizan los pines SPI nativos por defecto. El SuperMini tiene un diseño de pines muy compacto. Un punto vital es evitar conflictos con los **pines de booteo** (strapping pins). Como ya sabrás por experiencia con esta placa, el **GPIO2** debe manejarse con cuidado, por lo que lo evitaremos para funciones de interrupción críticas si es posible.
- Aquí tienes el mapeo estándar:

![PO LORA](../assets/PO_LORA01_SX1278.png)

# 5. SENSORES
En esta seccion agrupamos los sensores por categoria. 
## 5.1 Sensores de Posición, Presencia y Movimiento
Son los que "sienten" si algo ha cambiado de lugar o si hay alguien presente.

- Sensores de Efecto Hall: Detectan campos magnéticos (como el que hablamos para el eje de la cortina).

- Sensores PIR (Pasivos de Infrarrojos): Detectan el movimiento de personas/animales por calor. Muy usados para luces automáticas.

- Sensores Ultrasónicos (HC-SR04): Miden distancia mediante el rebote de sonido. Útiles para medir nivel de agua en tanques o proximidad de un objeto.
  
- Finales de carrera (Endstops): Interruptores mecánicos (físicos). Ideales para saber si la cortina llegó al tope.- 

## 5.2 Sensores de Flujo y Nivel
Vitales para tu PROYECTO_TANQUE_AGUA.

- Sensores de Nivel por boya (Interruptor de flotador): El clásico "flotante" que abre o cierra un contacto eléctrico al subir el agua.

- Sensores de nivel capacitivos: Detectan el líquido a través de la pared del tanque (sin tocar el agua, ideal para evitar corrosión).

- Sensores de Flujo (Flow meters): Miden cuántos litros pasan por una tubería por minuto mediante una turbina interna.

## 5.3 Sensores Ambientales (Variables físicas)
Miden las condiciones del entorno.

- Temperatura y Humedad (DHT11/DHT22): Son el estándar en automatización hogareña.

- Sensores de Presión Atmosférica (BME280): Además de temperatura y humedad, miden presión, lo que sirve para predecir cambios climáticos.

- Sensores de Calidad de Aire (MQ-series): Detectan gases como GLP, humo, CO2 o alcohol.

## 5.4 Sensores de Luz e Intensidad

- Fotoresistencias (LDR): Miden la intensidad de luz ambiente. Útiles para cerrar las cortinas automáticamente cuando el sol pega directo o encender luces al anochecer.

- Sensores infrarrojos (IR): Detectan haces de luz invisibles (útiles para barreras de seguridad o control remoto).

## 5.5 Sensores de Medición Eléctrica (Energía)
Estos sensores miden los parámetros eléctricos de tus circuitos para monitorear el consumo de la casa:
- Transformadores de Corriente (Tipo SCT-013 / CT Sensor:
  - ¿Qué hacen?: Miden la intensidad de corriente que pasa por un cable sin necesidad de cortarlo (funcionan mediante inducción magnética).
  - Uso: Son ideales para medir el consumo total de la casa o de un electrodoméstico específico en el tablero eléctrico.
- Sensores de Voltaje (tipo ZMPT101B):
  - ¿Qué hacen?: Miden el voltaje de corriente alterna (AC). 
  - Uso: Permiten conocer la tensión exacta de la red eléctrica para calcular la potencia real ($P = V \cdot I$).
- Módulos de Medición de Energía (tipo PZEM-004T):
  - ¿Qué hacen?: Es una solución "todo en uno" muy popular en Home Assistant. Miden voltaje, corriente, potencia y energía acumulada.
  - Uso: Se conectan mediante comunicación serial (UART) a tu ESP32/ESP8266 y son extremadamente precisos.

## 5.6 Sensores de Fuerza y Peso (Celdas de Carga)
Estos sensores se utilizan para medir masa o presión. Si vas a documentar una balanza inteligente (por ejemplo, para saber cuánto gas queda en una garrafa o cuánto alimento queda en un comedero de mascotas), esta es su categoría:
- Celdas de Carga (Load Cells):
  - ¿Qué son?: Son bloques metálicos (generalmente aluminio o acero) que contienen un puente de Wheatstone formado por galgas extensiométricas (resistencias que cambian su valor al deformarse).
  - ¿Cómo funcionan?: Cuando el metal se dobla una fracción de milímetro por el peso, la resistencia eléctrica cambia. Un módulo convertidor (como el HX711) lee esa mínima variación y la convierte en un dato digital que tu microcontrolador puede entender.

- Nota Técnica sobre Celdas de Carga:
  - Sensibilidad: Son extremadamente sensibles. Requieren una estructura mecánica estable donde el peso se aplique de forma vertical y uniforme.
  - Amplificación: La señal analógica es tan pequeña que siempre requieren un amplificador dedicado (como el HX711) para poder ser procesada por un ESP32 o Arduino.

## 5.7 Sensores Opticos (camaras)

Las cámaras (especialmente las cámaras IP o los módulos de cámara para proyectos de electrónica) no son microcontroladores en sí mismos, sino periféricos de entrada de datos complejos.

La categoría técnica es "Sistemas Embebidos de Visión"
- Si estás diseñando un proyecto, las cámaras encajan en la categoría de Sistemas Embebidos de Visión (Embedded Vision Systems).

- Si estás usando algo como un ESP32-CAM, estás en la categoría de Microcontroladores con aceleración de video.Si estás usando algo más avanzado, como una Raspberry Pi o una placa tipo NVIDIA Jetson, estás en la categoría de Microprocesadores (SoC - System on a Chip) con capacidades de Visión Artificial.

Dependiendo de cómo las analicemos, caen en categorías distintas dentro de un sistema:

- Desde la perspectiva de la arquitectura (Hardware)

  Las cámaras se clasifican como Periféricos de Periféricos (o Sensores de Alta Densidad).

  A diferencia de un sensor simple (como un termómetro o un sensor PIR, que envían un valor numérico), una cámara envía una corriente de datos masiva (imágenes o video).

  Por eso, la mayoría de los microcontroladores pequeños (como un Arduino Nano o un ESP8266) no pueden "procesar" una cámara por sí solos; necesitan una interfaz dedicada o un chip intermedio.

- Desde la perspectiva de su "inteligencia"

  Aquí es donde se vuelve interesante, porque las cámaras modernas suelen dividirse en dos mundos:

  - Cámaras "Tontas" (Módulos de imagen): Como el módulo OV2640 que se usa a menudo con el ESP32-CAM. La cámara solo captura la luz y entrega los datos crudos al microcontrolador. En este caso, el ESP32 actúa como el "cerebro" que gestiona la captura y el envío.
  - Cámaras Inteligentes (Smart Cameras / IP): Tienen su propio sistema operativo (generalmente Linux embebido), su propio procesador de imagen (ISP) y memoria RAM. En esta categoría entran las cámaras de seguridad que conectas a tu red. Estas son, en esencia, computadoras completas dedicadas exclusivamente a la visión.

### ¿Cómo se integran en tu ecosistema?
Dado que tienes interés en la domótica y ESP32, es probable que las veas así:

- Como nodos IoT: El ESP32-CAM captura la imagen y la transmite por Wi-Fi a tu Home Assistant. Aquí, la cámara es un sensor de entrada de video.

- Como dispositivos de red: Las cámaras IP (como las que integras vía RTSP o integración ONVIF en Home Assistant) son dispositivos de red autónomos. No dependen del microcontrolador para funcionar, sino que el microcontrolador (o el servidor de Home Assistant) simplemente consume el flujo de datos que la cámara ya está procesando.

### Lectura de un medidor de gas con una ESP32-CAM, 

La clave no es solo la cámara en sí, sino asegurar que tenga la memoria suficiente para procesar las imágenes (OCR) y la capacidad de enfocar a corta distancia.

Aquí te detallo exactamente qué buscar para que el proyecto funcione, especialmente si piensas usar el popular firmware "AI on the Edge":

- El Modelo de ESP32-CAM. Debes comprar el modelo AI-Thinker (es el más común, generalmente de color negro). Lo más importante es la PSRAM: Requisito Crítico: Asegúrate de que tenga al menos 4 MB de PSRAM. Sin esto, el software de reconocimiento de imágenes no podrá cargar los modelos de IA para leer los números. Cámara Incluida: Normalmente vienen con el sensor OV2640 de 2 MP. Es suficiente para esta tarea.
- El Lente: El Problema del Enfoque. Las ESP32-CAM vienen de fábrica enfocadas al "infinito" (más de 40 cm). Como vas a colocar la cámara muy cerca del medidor (a unos 10-15 cm), la imagen saldrá borrosa y la IA no podrá leer nada. Modificación necesaria: El lente viene pegado con un punto de pegamento. Debes usar un bisturí o una pinza pequeña para romper ese sello y girar el lente en sentido antihorario (hacia afuera) aproximadamente un cuarto o media vuelta hasta que la imagen a corta distancia se vea nítida. Opción alternativa: Puedes comprar un lente "Wide Angle" (gran angular) si el espacio donde está el medidor es muy reducido, pero el lente estándar modificado suele ser mejor para evitar distorsiones en los números.
- Accesorios Imprescindibles. Para que el sistema sea estable, vas a necesitar:
  - Adaptador USB (ESP32-CAM-MB): Te recomiendo mucho comprar el kit que viene con la "plaquita" de abajo que tiene puerto Micro-USB. Facilita enormemente la programación y la alimentación.
  - Tarjeta MicroSD: El firmware "AI on the Edge" corre desde la SD. Una de 4 GB o 8 GB (Clase 10) es más que suficiente. Evita tarjetas muy grandes (64 GB+) porque a veces dan problemas de compatibilidad.
  - Antena Externa (Opcional): Si tu medidor de gas está en un gabinete metálico o lejos del router, compra la versión que trae conector IPEX para una antena externa, ya que la antena integrada es muy débil.

Resumen de Compra

Componente Especificación Recomendada

Módulo ESP32-CAM AI-Thinker con 4MB PSRAM

Cámara OV2640 (incluida)

Alimentación Módulo de programador USB (CP2102 o CH340)

Almacenamiento MicroSD 4GB/8GB (FAT32)


## 5.8 Sensores Biológicos (Biosensores)
Estos dispositivos combinan un componente biológico sensible con un transductor electrónico que convierte la señal biológica en un dato digital.

A diferencia de los sensores resistivos o capacitivos, los sensores biológicos tienen una vida útil limitada (porque el componente biológico se degrada) y requieren calibración frecuente y condiciones de temperatura estables.

- Biosensores Enzimáticos: Utilizan enzimas específicas para reaccionar ante una sustancia.
  - Uso: El ejemplo más común es el medidor de glucosa (glucómetro). En automatización de agua, se usan para detectar niveles de urea o contaminantes orgánicos específicos.
- Sensores de pH biológicos/químicos:
  - Uso: Vitales en sistemas de acuaponia o piscinas automatizadas. Miden el nivel de acidez/alcalinidad necesario para el equilibrio de vida.

- Sensores de Oxígeno Disuelto (DO):
  - Uso: Indispensables si automatizas un estanque con peces o un sistema de tratamiento de agua, ya que miden cuánto oxígeno tienen los organismos vivos para respirar.

- Sensores de Microbios (Microbial Sensors):
  - Uso: Utilizan bacterias vivas inmovilizadas en una membrana. Cuando las bacterias consumen un contaminante, generan una señal eléctrica (metabolismo).
  
## 5.9 Sensores Biomédicos / Biométricos
Estos sensores interactúan directamente con el cuerpo humano para obtener métricas fisiológicas. Son la base de los dispositivos wearables (como smartwatches) y sistemas de telemedicina.

- Sensores de Frecuencia Cardíaca (PPG - Fotopletismografía):
  - ¿Qué hacen?: Utilizan luz infrarroja o verde para detectar los cambios de volumen sanguíneo en los capilares de la piel cada vez que late el corazón.
  - Uso: Monitoreo de pulso en reposo o ejercicio.

- Sensores de Oximetría de Pulso (SpO2):
  - ¿Qué hacen?: Miden la saturación de oxígeno en la sangre basándose en la absorción de luz a diferentes longitudes de onda.

- Sensores de Actividad Eléctrica (ECG/EMG):
  - ¿Qué hacen?: Capturan las señales eléctricas generadas por el corazón (ECG) o por la contracción de los músculos (EMG).
  - Uso: Diagnóstico básico de ritmo cardíaco o control de prótesis y dispositivos con gestos musculares.

- Sensores de Respuesta Galvánica de la Piel (GSR):
  - ¿Qué hacen?: Miden la conductividad eléctrica de la piel, que varía según la sudoración.
  - Uso: Es el indicador técnico del nivel de estrés o excitación emocional.

# 6. ACTUADORES

Esta es la categoría donde estan todos los dispositivos que "hacen algo". 

|Categoría|Componente| Función| 
|:---|:---|:---
|Actuador Sonoro|Buzzer (Pasivo/Activo)|Generar alertas, alarmas o avisos
|Actuador Visual|LED / Pantalla OLED|Mostrar estados o notificaciones
|Actuador Físico|Relé / MOSFET|Encender/apagar luces o motores

Por qué es importante separar Sensores de Actuadores:

- Seguridad: Los sensores suelen ser de "baja potencia" (entrada de datos), mientras que los actuadores (especialmente si usas relés para encender luces de 220V) implican un riesgo eléctrico mayor.

- Lógica: En la programación de Home Assistant o ESPHome, los sensores se configuran como binary_sensor o sensor, mientras que el buzzer se configurará como switch o output. 
  
Mantenerlos separados en tu documentación te facilitará muchísimo el trabajo cuando estés escribiendo el código.

# 11. SENSORES DE POSICION Y MOVIMIENTO

## 11.1 DISTANCIA - ULTRASONICO HC-SR04

El sensor ultrasónico HC-SR04 proporciona mediciones de distancia desde 2cm hasta 500cm con una precisión cercana a los 3mm. Este módulo se diferencia al contar con pines separados para la señal de entrada y salida.

### 11.1.1 Especificaciones

- Voltaje de alimentación: 5V DC
- Corriente en reposo: <2mA
- Ángulo de cobertura: <15°
- Rango de distancia: 2cm - 500 cm
- Resolución: 0.3 cm
- Frecuencia ultrasónica: 40k Hz

### 11.1.2 Esquema de conexiones (PIN OUT)

- VCC: +5V DC)
- TRIG: Disparo del ultrasonido
- ECHO: Recepción del ultrasonido
- GND: 0V

## 11.2 DISTANCIA - ULTRASONICO JSN-SR04T

MODELO: Ultrasóncio Jsn-04t 2.0 Waterproof 5V

Sensor por ultrasonido de distancia, resistente al agua AJ-SR04M

El sensor SR04 es un sensor de distancia que utiliza ultrasonido (sonar) para determinar la distancia de un objeto en un rango de 25 a 600 cm. Destaca por su pequeño tamaño, bajo consumo energético, buena precisión y especialmente por su resistencia al Agua.

El sensor trabaja con ultrasonido y contiene toda la electrónica encargada de hacer la medición. El funcionamiento del sensor es el siguiente: se emite un pulso de sonido (TRIG), se mide la anchura del pulso de retorno (ECHO), se calcula la distancia a partir de las diferencias de tiempos entre el Trig y Echo. El funcionamiento no se ve afectado por la luz solar o material negro (aunque los materiales blandos acusticamente como tela o lana pueden ser difícil de detectar).

Perfecto para aplicaciones donde el sensor estará expuesto a la intemperie, utilizado en automóviles para medir distancia de colisión/parqueo.

### 11.2.1 Especificaciones

- Modelo: JSN-SR04/ AJ-SR04M
- Voltaje de Operación: 5V DC
- Corriente de trabajo: 30mA
- Rango de detección: 25cm- 4.5Mts
- Precisión: puede variar entre los 3mm o 0.3cm
- Frecuencia de emisión acústica: 40KHz
- Duración mínima del pulso de disparo (nivel TTL): 10 µS.
- Tiempo mínimo de espera entre una medida y el inicio de otra 20 mS.
- Ángulo de detección: menor a 50º
- A prueba de agua (parte delantera)
- Diámetro: 22mm - Longitud: 17mm
- Temperatura de trabajo: -10ºC hasta 70ºC

### 11.2.2 Esquema de conexiones (PIN OUT)

- VCC: +5V DC
- TRIG: Disparo del ultrasonido
- ECHO: Recepción del ultrasonido
- GND: 0V

## 11.3 MOVIMIENTO - Sensor Infrarrojo PIR Hc Sr501

Los Sensores PIR pueden detectar movimientos hasta a 7 metros de Distancia gracias a su lente Fresnel. En su interior contiene un sensor Infrarrojo. Ideal para proyectos de sistemas de Alarmas, detección de movimiento o inclusive iluminación activada por proximidad.

### 11.3.1 Especificaciones

- Voltaje de operación: 4.5V - 20V
- Consumo en reposo: <50uA
- Voltaje de salida: 3.3V (alto) / 0V (bajo)
- Rango de detección: 3 a 7 metros (ajustable)
- Angulo de detección: <100º
- Retardo: 5-200 s (Ajustable) por defecto 5S +-3%
- Tiempo de bloqueo: 2.5s (por defecto)
- Temperatura de trabajo: -20ºC hasta 80ºC
- Dimensiónes: 32mm x 24mm x 18mm
- Redisparo configurable mediante jumper (soldado)

### 11.3.2 Esquema de conexiones (PIN OUT)

- VCC: 5 VDC
- OUT: señal analógica
- GND: 0 VDC (tierra)


## 11.4 MAGNETICO - REED SWITCH

Un reed switch es esencialmente, un interruptor mecánico que se activa por la presencia de un campo magnético. En términos de funcionamiento, es "primo hermano" del Sensor de Efecto Hall, pero con una diferencia clave: el Reed Switch es un componente mecánico (tiene dos láminas metálicas que se tocan físicamente), mientras que el sensor Hall es electrónico (de estado sólido).

¿Cuáles son las ventajas de usar los Interruptores Reed?

Están herméticamente sellados en un ambiente de vidrio, libres de contaminación y son seguros para su uso en entornos industriales y explosivos difíciles. Los Interruptores Reed son inmunes a las descargas electrostáticas (ESD) y no requieren ningún circuito externo de protección contra ESD. La resistencia de aislamiento entre los contactos es tan alta como 10^15 ohmios, y la resistencia de contacto es tan baja como 50 mili-ohmios. Los interruptores Reed pueden cambiar directamente cargas de tan pocos microvatios sin necesidad de circuitos de amplificación externos.

![REED SWITCH](../assets/Reed_Switch.png)

### 11.4.1 Especificaciones

- Contacto Normalmente Abierto
- 10 W/VA Máximo.
- 100 Vac/dc Máximo
- 0.5 A Máximo
- Medidas: 14 x 2 mm
- Modelo: GC 2325


# 12. SENSORES DE NIVEL Y FLUJO
## 12.1 CAUDAL - FS300A G3/4"

Sensor Medidor De Caudal (caudalímetro) de ¾" para agua y líquidos por efecto hall.

- Tensión de alimentación: +5V a +24V (DC)
- Rango de Medición de caudal: 1-60 l/minuto
- Tipo de medición: 5 pulsos por litro
- Temperatura de operación: hasta 80°C.
- Carga de salida máxima: 10 mA a 5V

![FS300A](../assets/Caudal_FS300A.png)

### 12.1.1 Especificaciones

- Roscas externas: 3/4 pulgadas
- temperatura: -20 80 °C
- Presión permitida: 1,75 Mpa
- Rango de humedad operativa: 35-90% de humedad relativa (sin escarcha)
- Capacidad de carga: 10 mA (5 V CC)
- Rango de voltaje de trabajo: 5 -18 V DC
- Corriente máxima de funcionamiento: 15 mA (5 V DC)
- El voltaje de trabajo nominal más bajo: 4,5 V CC (5 V DC)
- Rangos de flujo: 3/4 pulgadas 1 a 60 l/min
- Solenoide de potencia estándar o no estándar, presión de solenoide, baja presión, material de estructura de plástico medidor de flujo, medidor de flujo, medidor de flujo de agua, medidor de flujo digital, sensor de flujo de agua, medidor de flujo rv

### 12.1.2 Esquema de conexiones (PIN OUT)

- VCC: rojo
- Señal: amarillo
- GND: negro

## 12.2 CAUDAL - Dn25 FS400A G1"

Este caudalimetro tiene el mismo principio que los anteriores, es por efecto hall

![Caudal ](../assets/Caudal_FS401.png)


### 12.2.1 Especificaciones

- Frecuencia: HZ = (6 x Q-2) x Q = (L/min)
- Voltaje de salida de primera calidad: 3,5-24 VDC un litro de agua después de la salida de pulsos 358
- Rango: 2-60 L/min
- Diámetro interior/diámetro exterior: 20 mm/32,7 mm
- Longitud del tornillo: 13 mm
- Se puede utilizar para: aplicar a alimentos, máquinas de tarjetas de crédito, máquinas expendedoras de agua y otros dispositivos de medición
- Voltaje de funcionamiento nominal más bajo: DC4.5
- La corriente de funcionamiento máxima: 15 mA (5 V)
- Rango de voltaje de funcionamiento: 5 24 V
- Capacidad de carga: inferior o igual a 10 mA (5 V)
- Rango de temperatura de funcionamiento: inferior o igual a 80
- Humedad de funcionamiento: entre el 35 y el 90% de humedad relativa (sin condensación)
- Permita una presión inferior: 1,75 Mpa
- Temperatura de almacenamiento: -25 + 80
- Humedad de almacenamiento: entre el 25 y el 95% de humedad relativa.
- Color: negro. Material: ABS. Contenido del paquete: 4 sensores de agua DN25.

### 12.2.2 Esquema de conexiones (PIN OUT)

- VCC: rojo
- Señal: amarillo
- GND: negro

## 12.3 CAUDAL - YF-S403

Sensor de Flujo de Agua 3/4 Rosca Externa 1-60L / min Control de Agua Caudalímetro

El sensor de flujo de agua consta principalmente de un cuerpo de válvula de plástico, un conjunto de rotor y un sensor de corriente Hall. Está instalado en el extremo de entrada de agua caliente para detectar el flujo de agua. Cuando el agua atraviesa el conjunto del rotor de flujo, el rotor magnético girará y la velocidad cambiará a medida que cambie el flujo. El sensor de corriente Hall emite la señal de pulso correspondiente y la retroalimentación al controlador, luego el controlador controlará el flujo.

Puede usarse para calentadores de agua, máquinas de tarjetas de crédito, máquinas expendedoras de agua, dispositivos de medición de flujo.

![Caudal YFS403](../assets/Caudal_YF-S403.png)

### 12.3.1 Características

- Peso ligero, tamaño pequeño, fácil de instalar.
- Con incrustaciones de eje impulsor de acero inoxidable, resistente al desgaste.
- El sello con la estructura de fuerza e inferior nunca tiene fugas.
- Precauciones:
- Choque no violento y erosión química.
- No lanzar ni golpear.
- Montado verticalmente, no debe exceder los 5 grados de inclinación.
- La temperatura del medio no debe exceder los 120 C
- Conexión del cable:
- Rojo: positivo (+)
- Amarillo: salida de señal.
- Negro: negativo (-)

### 12.3.2 Especificaciones

- El voltaje de trabajo nominal más bajo: DC4.5 5V-24V.
- Corriente máxima de funcionamiento: 15 mA (DC 5V).
- Rango de voltaje de trabajo: DC 5 18 v.
- Capacidad de carga: <= 10 mA (DC 5V).
- Temperatura de trabajo: <= 80 C
- Rango de humedad de trabajo: 35% 90% RH (sin escarcha)
- Permitiendo presión: presión 1.75Mpa
- Temperatura de almacenamiento: -25 + 80 C
- Ahorre humedad: 25% 95% RH
- Rango de flujo: 1-60L / min
- Hilos externos: 3/4
- Tamaño (largo x ancho x alto): aprox. 6.2 x 3.6 x 3.5 cm / 2.44 x 1.38 x 1.38 pulgadas
- Diámetro de la interfaz de salida: Aprox. exterior 26 mm, interior 12 mm, 14 mm
- El paquete incluye:
- 1 x Sensor de flujo de agua 1-60L / min

### 12.3.3 Esquema de conexiones (PIN OUT)

- VCC: rojo
- Señal: amarillo
- GND: negro

## 12.4 CAUDAL - CAUDALIMETRO COBRE G3/4"

Medidor De Caudalímetro De Interruptor De Sensor De Flujo De Agua Líquida De Efecto Hall De Cobre G3/4

Hecho de material de cobre de alta calidad.

Su presión máxima de agua es de 1.75MPa, el rango de caudal es de 2L/min a 45L/min.

Ligero y de tamaño compacto, rendimiento estable, arranque con baja presión de agua, muy fácil de instalar y seguro de usar

Se puede usar perfectamente para calentadores de agua, dispensadores de agua, purificadores de agua, máquinas expendedoras de agua, cafeteras, dispositivos de medición de flujo, máquinas de control de agua, etc.

Conexión de tres cables: rojo para polo positivo, negro para polo negativo, amarillo para señal de pulso.

![Caudalimetro cobre](../assets/Caudal_Cobre34.png)

### 12.4.1 Especificaciones

- Tipo: G3/4(Diámetro de rosca externo)
- Tamaño: DN20
- Material: Cobre
- Requisito de calidad del agua: <= 60 C
- Rango de flujo de inicio: 1,5 l/min.
- Rango de flujo: 2-45L/min
- Presión máxima de agua: 1.75MPa
- Longitud del tubo: aprox. 45 mm/ 1,77 pulgadas
- Diámetro de rosca externa: G3/4
- Color: como muestran las imágenes

### 12.4.2 Esquema de conexiones (PIN OUT)

- VCC: rojo
- Señal: amarillo
- GND: negro

# 13. SENSORES AMBIENTALES
## 13.1 TEMPERATURA - DS18B20

Sensor Digital Temperatura DS18B20 con Cable Sumergible de 50 cm de largo (IP67)

Ideal para control ambiental de HVAC, sensor de temperatura interior, equipamiento o maquinas. Ideal también para medición en sitios lejanos o en condiciones húmedas.

Cada sensor tiene un número de serie de 64 bits único que le permite conectar múltiples sensores en paralelo usando solo un cable como bus de datos.

![DS18B20](../assets/Temperatura_DS18B20.png)

### 13.1.1 Especificaciones

- Interfaz 1-Wire (Requiere un solo pin digital para comunicarse)
- Numero de Serie unico de 64 bits grabado en el chip (Multiples sensores pueden compartir misma conexion)
- Sistema de alarma por limite de temperatura
- Exactitud de entre -10°C a +85°C (±0.5°C)
- Rango de temperatura: -55°C a 125°C (-67°F a +257°F)
- Voltaje de Operacion: 3 a 5 VCC
- Resolución: seleccionable de 9 a 12 bits
- Tiempo de consulta menor a 750ms
- Conexionado con 3 cables: VCC (Rojo), GND (Negro), Datos (Amarillo)
- Diámetro: 6 mm
- Largo del Cable: 50 cm
- Sensor de Acero Inoxidable de 35 mm de largo
- Cable tipo taller de 4 mm

### 13.1.2 Esquema de conexiones (PIN OUT)

![PO DS18B20](../assets/PO_DS18B20.png)

## 13.2 TEMPERATURA - MODULO DHT11 KY-015

El módulo DHT11 / KY-015 es un sensor de temperatura y humedad relativa de media precisión. Este módulo usa un termistor NTC y un sensor capacitivo de humedad para determinar las condiciones del entorno, tiene un tamaño ultra compacto, es de bajo consumo de energía y tiene gran utilidad cuando se requiere detectar dos magnitudes al mismo tiempo, se utiliza en un gran número de aplicaciones básicas.

El módulo KY-015/DHT11 es fácil de utilizar con las tarjetas de Arduino, Raspberry Pi y Nodemcu, a nivel de software se dispone de librerías para Arduino con soporte para el protocolo "Single bus".

En cuanto al hardware, solo es necesario conectar el pin VCC de alimentación a 3 o 5V, el pin GND a Tierra (0V) y el pin de datos a un pin digital.

Este modelo de DHT11 dispone de 3 pines, la toma de tierra GND, para los datos DATA y para la alimentación VCC (de 3,5V a 5V).

![DHT11](../assets/Temperatura_DHT11.png)

### 13.2.1 Especificaciones

- Voltaje de funcionamiento: 3.5V - 5.5V
- Rango de medición de humedad: 20% a 90% RH (Error de medición de humedad: +-5%)
- Resolución de medición de humedad: 1% RH
- Rango de medición de temperatura: 0ºC - 50ºC (Error de medición de temperatura: +-2 grados)
- Resolución de medición de temperatura: 1ºC
- Rango de transmisión de señal: 20 metros
- Altura: 19 mm
- Largo: 17 mm
- Ancho: 19 mm

### 13.2.2 Esquema de conexiones (PIN OUT)

![PO DHT11](../assets/PO_DHT11.png)

## 13.3 TEMPERATURA - AMT1001

El sensor AMT1001 adopta un modo de salida de voltaje analógico; este módulo destaca por su alta precisión, confiabilidad, buena consistencia y compensación de temperatura, garantizando estabilidad a largo plazo. Es fácil de usar, tiene un precio bajo y es especialmente adecuado para aplicaciones en empresas con requisitos de calidad y costos más exigentes.

Aire acondicionado, humidificadores, deshumidificadores, comunicaciones, monitoreo del entorno atmosférico, control de procesos industriales, agricultura, instrumentos de medición y otros campos de aplicación.

![AMT1001](../assets/Temperatura_AMT1001.png)
s
### 13.3.1 Características Destacadas

- Bajo consumo de energía.
- Tamaño compacto.
- Compensación de temperatura.
- Salida lineal calibrada con un solo chip.
- Alta confiabilidad.
- Fácil de usar.
- Precio bajo.

### 13.3.2 Especificaciones

- Voltaje de alimentación (Vin): DC 4.7V - 5.25V
- Consumo de corriente: aproximadamente 2mA (MÁX. 3mA)
- Rango de temperatura de operación: 0 - 50 ºC
- Rango de humedad de operación: por debajo del 95% de HR (sin condensación)
- Rango de detección de humedad: 2 - 90% de HR
- Rango de temperatura de almacenamiento: 0 - 50 ºC
- Rango de humedad de almacenamiento: por debajo del 80% de HR (sin condensación)
- Precisión de detección de humedad: ±5% de HR (0 - 50 ºC, 30 - 80% de HR)
- Rango de salida de voltaje: 0.6-2.7V DC

### 13.3.3 Esquema de conexiones (PIN OUT)

![PO AM1001](../assets/PO_AMT1001.png)


## 13.4 TEMPERATURA - AHT10

El AHT10, una nueva generación de sensores de temperatura y humedad, establece un nuevo estandar en tamano e inteligencia: esta integrado en un paquete SMD sin plomo plano de doble fila para soldadura por reflujo con 4x5mm inferior y una altura de 1,6mm. El sensor emite una senal digital calibrada en formato I 2C estandar. El AHT10 esta equipado con un chip ASIC nuevo disenado, un elemento de detección de humedad capacitivo MEMS semiconductor mejorado y un elemento de detección de temperatura estandar en chip. Su rendimiento ha mejorado mucho mas alla del nivel de fiabilidad de los sensores de generación anterior. Se ha mejorado la primera generación de sensores de temperatura y humedad para que sean mas estables en entornos duros.

![AHT10](../assets/Temperatura_AHT10.png)

### 13.4.1 Especificaciones
1. Tamano del módulo: 16*11mm
2. tipo de interfaz: I2C
3. voltaje de funcionamiento: 1,8-6,0 V
4. tamano de la interfaz: Paso de 4x2,54mm
5. precisión de la humedad: típica mas menos 2%
6. Resolución de humedad: 0.024%
7. precisión de temperatura: típica mas menos 0,3 gradosC
8. Resolución de temperatura: típica de 0,01 gradosC
9. temperatura de funcionamiento:-40 gradosC-85 Deg.C

### 13.4.2 Esquema de conexiones

![PO AHT10](../assets/PO_AHT10.png)

## 13.4.3Consideraciones de Instalación

- **Opción A (Extensión):** Uso de cable de 4 hilos (I2C) para posicionar el sensor fuera del gabinete, manteniendo una longitud máxima de 80 cm para preservar la integridad de la señal.
- **Opción B (Integración con Aislamiento):** Diseño de cámara externa en el gabinete.
    - Incluir barrera física (material aislante) entre el sensor y la electrónica.
    - Implementar ventilación pasiva mediante rejillas de flujo cruzado para asegurar el equilibrio térmico con el ambiente exterior.

## 13.5 TEMPERATURA - HTU21D

El sensor de temperatura y humedad HTU21 es ideal para la detección del medio ambiente y del registro de datos, es un sensor digital que permite realizar tomas de datos directamente con la interfaz deseada, cada sensor es calibrado y puede ser utilizado donde sea necesario tomar datos del medio ambiente o en alguna otra aplicación, es de fácil instalación.

- Pequeño y fácil de usar
- Preciso y de bajo costo
- Detecta humedad y temperatura
- Compatible con I2C

Es un sensor digital de humedad y temperatura de bajo costo, fácil de usar y altamente preciso. Este sensor es ideal para detección ambiental y registro de datos, y es perfecto para estaciones meteorológicas o sistemas de control de humidificadores.

Este sensor de humedad digital I2C es una alternativa precisa e inteligente al sensor de humedad y temperatura más simple. Tiene una precisión típica de ± 2% con un rango de operación que está optimizado de 5% a 95% HR. La operación fuera de este rango es posible, aunque la precisión puede disminuir ligeramente. La temperatura de salida tiene una precisión de ± 1 °C de -30 ~ 90 °C.

La placa cuenta con un filtro de PTFE para mantener limpio el sensor, un regulador de 3.3 V y un circuito de cambios de nivel I2C. Esto permite su uso seguro con cualquier tipo de microcontrolador con potencia o lógica de 3.3V-5V.

![HTU21D](../assets/Temperatura_HTU21D.png)

### 13.5.1 Aplicaciones

- Se puede utilizar en varias aplicaciones como: HVAC/R, termostatos/humidistatos, terapia respiratoria, Electrodomésticos, estaciones meteorológicas interiores, microentornos/centros de datos, control del clima automotriz, seguimiento de activos y bienes, teléfonos móviles y tabletas

### 13.5.2 Especificaciones

- Fuente de alimentación 3-5 v
- Precisión Sensor de humedad relativa ± 3% RH (max) 0-80% RH
- Alta precisión de Sensor de temperatura ±0.4 °C (máx.), de -10 a 85 °C
- Rango de funcionamiento de 0 a 100% RH
- Rango de funcionamiento de hasta -40 A + 125 °C
- Gran voltaje de funcionamiento (1,9 a 3,6 V)
- Bajo consumo de energía 150 µA corriente activa 60 nA corriente de reposo
- Calibrado en fábrica
- I2C interfaz
- Calentador integrado en chip
- 3x3mm paquete DFN
- Excelente estabilidad a largo plazo
- Protección durante reflujo Excluye líquidos y partículas
- Tamaño: 3 x1 cm

### 13.5.3 Esquema de conexión

- VCC: 5 VDC
- GND: 0 VDC
- SCL: datos (comunicación I2C)
- SDA: datas (comunicación I2C)

Esta tabla de ruptura tiene resistencias de extracción de 4.7k para comunicaciones I2C. Si se conectan varios dispositivos I2C en el mismo bus, puede ser necesario desactivar estos resistores.

## 13.6	TEMPERATURA – Modulo Max6675 
Termocupla K 0°c A 1024°c 
Mide temperaturas con alta precisión usando el módulo MAX6675 y termocupla K. Compatible con Arduino, ESP32 y Raspberry Pi, ofrece un rango de hasta 1024°C con salida digital SPI. Incluye jumpers para conexión rápida. Ideal para proyectos industriales, científicos y de control térmico. Compacto, confiable y fácil de usar.

![MAX6675](../assets/Temperatura_MAX6675.png)

### 13.6.1 Características:
- Peso Total aproximado: 28g
### 13.6.2	Características: Módulo MAX6675:
-	Voltaje de operación: 3.3V - 5V
-	Salida digital: SPI (Serial Peripheral Interface)
-	Precisión: ±1.5°C
-	Resolución: 0.25°C
-	Conversión de temperatura: Cada 0.25 segundos
-	Consumo de corriente: Bajo (< 50mA)
-	Compatibilidad: Ard5uino, ESP8266, ESP32, Raspberry Pi
-	Medidas: 25mm x 15mm x 13mm
-	Peso: 6gr
### 13.6.3	Características: Termopar Tipo K:
-	Rango de temperatura: 0°C a 1024°C
-	Material: Acero inoxidable
-	Tipo de conexión: 2 cables (positivo y negativo)
-	Aislamiento: Revestimiento de fibra de vidrio o teflón
-	Precisión: ±2°C
-	Longitud Aproximada: 1m

## 13.6.4.	Esquema de conexión (PIN OUT

![PO_MAX6675](../assets/PO_MAX6675.png)

## 13.7 TEMPERATURA - Modulo SHT21D



# 14. SENSORES DE LUZ E INTENSIDAD
## 14.1 LUZ - MODULO SENSOR DE LUZ

Se trata de un módulo sensor de luz basado en un LDR que entrega una salida digital de nivel bajo cuando la luz supera el valor prefijado con el preset. Cuenta con una salida analógica proporcional a la intensidad lumínica detectada.

Este módulo está conformado por LDR o fotorresistor, el cual es sensible a la exposición de intensidad lumínica ambiental, para así determinar el brillo e intensidad lumínica del medio, este módulo a través de una salida digital, puedes programar un rango de luminosidad, proporcionando un nivel de tensión alto o bajo, dependiendo de la configuración preestablecida.

Adicionalmente, la salida analógica permite medir de manera continua la variación de la luz, proporcionando una lectura más precisa y adaptable para aplicaciones como medición de luz ambiental o sistemas de control automático.

Posee 2 leds, uno de los cuales indica que el módulo está alimentado y el otro enciende cuando la luz supera el nivel prefijado (muy útil para realizar el ajuste).

![SENSOR LUZ](../assets/Luz_LM933.png)

### 14.1.1 Especificaciones

- Detectar intensidad de luz del entorno
- Sensibilidad ajustable mediante potenciómetro
- Utiliza el comparador LM393 para mayor estabilidad
- Orificio de instalación para facilitar su uso
- Indicador de alimentación (PWR LED)
- Indicador de salida digital (DO-LED)
- Salida analógica proporcional a la intensidad de luz.
- Conexion de 3 hilos
- Dimensiones 30 x 14mm

### 14.1.2 Esquema de conexiones (PIN OUT)

- VCC: +5V DC
- GND: 0 VDC (tierra)
- DO: Señal analógica (conectar a un pin analógico)
- AO: Señal digital (conectar a un pin digital)


# 15. SENSORES DE MEDICION DE ELECTRICIDAD

## 15.1 CORRIENTE - SCT 013

El SCT-013 es un transformador de corriente (CT) de núcleo partido, muy popular en proyectos de monitoreo energético DIY. Su función es medir corriente alterna (AC) de forma no invasiva, lo que significa que no necesitas cortar cables ni interrumpir el circuito para instalarlo.

¿Cómo funciona?

Funciona bajo el principio de inducción electromagnética. Al "abrazar" un cable por el que circula corriente alterna, el sensor genera una corriente proporcional (o voltaje, dependiendo del modelo) en su salida.

![Corriente SCT 013](../assets/Corriente_SCT-013.png)

### 15.1.1 Características

- No invasivo: Tiene una pinza que se abre y se cierra sobre el cable conductor.
- Solo para AC: Al ser un transformador, no funciona con corriente continua (DC).
- Seguridad: Permite medir altos voltajes (110V o 220V) manteniendo el microcontrolador aislado eléctricamente.

### 15.1.2 Conexión a un Esp32

Para usarlo con un ESP32 o similares, te encontrarás con dos desafíos técnicos que mencionabas antes:

- 1\. La Resistencia de Carga (Burden)
  - Si el sensor es de salida de corriente, necesitas elegir la resistencia adecuada. Si el ADC lee hasta 3.3V, debes calcular una resistencia que, a la corriente máxima, genere un voltaje cercano a ese límite para no perder resolución (pero dejando margen para picos).
- 2\. El Circuito de Offset
  - La señal que sale del SCT-013 es una onda senoidal que oscila entre valores positivos y negativos. Sin embargo, los ADC de la mayoría de los microcontroladores solo pueden leer voltajes positivos (de 0V a 3.3V).
  - Solución: Se crea un "divisor de tensión" con dos resistencias y un capacitor para "subir" la señal.
  - Esto le da un offset de 1.65V (la mitad de 3.3V), permitiendo que la onda oscile por encima y por debajo de ese punto central sin entrar en valores negativos.

# 16. SENSORES DE FUERZA Y PESO
## 16.1 PESO - MODULO HX711 PARA CELDA DE CARGA

Módulo HX711 Transmisor de celda de carga

El HX711 es un transmisor para celdas de carga, permite obtener lectura confiables y con buena precisión.

Este módulo es una interface entre las celdas de carga y el microcontrolador, permitiendo poder leer el peso de manera sencilla.

Internamente se encarga de la lectura del puente wheatstone formado por la celda de carga, convirtiendo la lectura analógica a digital con su conversor A/A interno de 24 bits.

Se comunica con el microcontrolador mediante 2 pines (Clock y Data) de forma serial.

Utilizado en procesos industriales, sistemas de medición automatizada, industria médica.

![HXT711](../assets/Peso_HX711.png)

### 16.1.1 Especificaciones

- Voltaje de Operación: 5 V DC
- Consumo de corriente: menor a 10 mA
- Voltaje de entrada diferencial: ±40mV
- Resolución conversion A/D: 24 bit
- Frecuencia de refresco: 80 Hz
- Dimensiones: 38mm21mm10mm

### 16.1.2 Esquema de conexiones (PIN OUT)

Cada celda de carga sale con cuatro cables de estos colores: rojo, negro verde y blanco.

- Si el valor medido disminuye al aplicar peso: Simplemente invierte los cables de señal (A+ y A-) entre sí.
- Soldadura: Aunque algunos módulos incluyen bornes de tornillo, soldar los cables directamente al HX711 garantiza una conexión más estable y reduce el ruido eléctrico, lo cual es crítico para lecturas precisas de peso.
- Blindaje: Si tu celda tiene un quinto cable (más grueso o sin recubrimiento), ese es el cable de blindaje (shield). Puedes conectarlo a GND en el HX711 para reducir interferencias electromagnéticas.

## 16.2 PESO - CELDA DE CARGA Yzc-161e Z09 (50 kg)

Al medir, la fuerza correcta se aplica al lado exterior de la parte del haz en forma de E del sensor (es decir, un medidor de tensión pegado al centro, con un haz de brazo de cubierta de plástico blanco) y los bordes exteriores para formar una fuerza de cizallamiento en la dirección opuesta, es decir, En el medio de la flexión de la viga de tensión pueden producirse los cambios necesarios bajo estrés, el haz de tensión lateral por otra fuerza no puede tener una barrera.

El sensor utiliza los siguientes tres métodos:

- Usar un sensor con resistencias externas rango de medición de puente completo de un rango de sensor: 50 kg. Requisitos más altos para resistencias externas
- El uso de solo dos sensores de puente completo es el rango de los dos sensores y: 50kgx2 = 100 kg
- El uso de cuatro sensores de puente completo es el rango de cuatro sensores y: 50kgx4 = 200 kg

Tamaño: 3,40x3,40 cm/1,34 pulgadas \* 1,34 pulgadas

Modelo: Tensión del Sensor de pesaje del Sensor de medio puente

Rango: 50 kg

![CELDA CARGA](../assets/Peso_YZC-161E_Z09.png)

### 16.2.1 Especificaciones

- 1\. Carga clasificada 50 (kilogramos)
- 2\. Salida nominal 1.0±0.15 mV/V
- 3\. Equilibrio cero ±0.3mV/V
- 4\. Resistencia de entrada 1000±10 Ohms
- 5\. Resistencia de salida 1000±10 Ohms
- 6\. Voltaje de la excitación 5~10 VDC
- 7\. Ausencia de linealidad 0.4%F.S
- 8\. Histéresis 0.3%F.S
- 9\. Repetibilidad 0.2%F.S
- 10\. Arrastramiento (30min) 0.1%F.S
- 11\. Temperatura de funcionamiento -10ºC ~ +40ºC
- 12\. Efecto de temperatura sobre cero 0.2%F.S/10ºC
- 13\. Efecto de temperatura sobre palmo 0.2%F.S/10ºC
- 14\. Resistencia de aislamiento 2000 MOhms (50VDC)
- 15\. Sobrecarga segura 150%F.S
- 16\. Última sobrecarga 150%FS
- 17\. Cable 420mm
- 18\. Tamaño 34 x 34mm

### 16.2.2 Esquema de conexión (PIN OUT)

Conectar una celda de carga de \*\*tres cables\*\* (como la YZC-161E) al módulo HX711 es un desafío especial porque el HX711 está diseñado internamente para recibir un \*\*puente de Wheatstone completo\*\* (que utiliza 4 cables).

Una celda de 3 cables representa solo una "media parte" de ese puente. Para que funcione, necesitas completar el puente utilizando resistencias externas o una segunda celda de carga.

### 16.2.3 Esquema de conexión (PIN OUT): 1 celda y dos resistencias externas

- Usar resistencias externas (La más común para una sola celda). Para completar el puente, debes añadir dos resistencias de precisión (idealmente de 1kΩ) para simular los dos brazos faltantes del circuito.

### 16.2.4 Consideraciones

- Calibración: Al usar este método, la calibración mediante software (ajustando el "factor de escala" en el código) es "obligatoria". Como el puente no es perfecto, el factor de escala será muy distinto al que obtendrías con una celda de 4 cables profesional.
- Precisión: Esta configuración es ideal para proyectos educativos o básculas sencillas, pero no se recomienda para aplicaciones industriales de alta precisión debido a que las resistencias externas son sensibles a los cambios de temperatura.
- Identificación de colores: aunque YZC-161E suele seguir Rojo (Excitación+), Blanco (Señal), Negro (Excitación-), te sugiero medir con un multímetro la resistencia entre los cables:
  - La resistencia mayor suele estar entre los cables de alimentación (Exc+ y Exc-).
  - La resistencia entre señal y alimentación suele ser la mitad de la anterior

# 17. SENSORES OPTICOS
## 17.1 ESP32 CAM Ov2640 + PLACA BASE PROGRAMADOR

El ESP32-CAM es una tarjeta de desarrollo y Modulo WiFi con Bluetooth. La Camara OV2640 de 2MP integra un sensor de imagen CMOS UXGA (16321232) de 1/4 de pulgada.

La Placa Base ESP32-CAM-MB es una placa diseñada para funcionar con el ESP32-CAM. Alimentado por una entrada Micro USB, proporciona la energía para operar y programar la ESP32-CAM. Simplifica el conexionado!

![ESP32 CAM](../assets/ESP32-CAM.png)


### 17.1.1 Especificaciones

- Modelo: ESP32-CAM + Camara OV2640
- Tensión de Alimentacion: 5V
- WiFi 802.11b/g/n
- Camara: OV2640 2MP
- CPU 32 bits de doble nucleo
- Frecuencia principal 240 MHz
- Potencia informatica 600 DMIPS
- Velocidad de reloj 160 MHz
- SRAM 520Kb, 4MPSRAM externa
- Soporta interfaces: UART / SPI / I2C / PWM / ADC / DAC
- Soporta cámaras OV2640 y OV7670, Flash Incorporado
- Soporta tarjetas TF micro SD (4GB Max)
- Compatible con modos de operación STA / AP / STA+AP
- Con antena en su PCB
- Integra conectores u.FL y FPC

### 17.1.2 Especificaciones ESP32-CAM-MB

- Tension de entrada: 5V DC (Por puerto USB)
- Compatibilidad: ESP32-CAM
- Modelo: ESP32-CAM-MB USB-C
- Integrado USB: CH340G
- Boton IO0: Boot
- Boton RST: Reset




# 17. SENSORES BIOLOGICOS

# 18. SENSORES BIOMETRICOS

## PULSO - Elster GmbH Elster IN-Z61

Pulse Sensor for Elster Bellows Gas Meter

Los fabricantes los diseñan con un campo magnético muy específico para evitar que alguien use imanes externos potentes para "frenar" el medidor (un fraude común). Por eso, un reed switch genérico a veces no tiene la sensibilidad o la orientación correcta para captarlo. El "medidor de pulsos" oficial para tu modelo Elster BK-G4 se conoce técnicamente como Generador de Impulsos de Baja Frecuencia.

El modelo exacto que necesitas: para ese medidor Elster (ahora parte del grupo Honeywell), el sensor oficial es el modelo IN-Z61. Aquí tienes los detalles para conseguirlo:

- Nombre comercial: Generador de impulsos Elster IN-Z61 (o serie IN-Z).
- Características: Es un cartucho de plástico que encaja perfectamente en la ranura que ves justo debajo de los números rojos en tu foto. No necesita tornillos, entra a presión o con un clip.
- Compatibilidad: Está diseñado para trabajar con la serie BK (BK-G1.6 hasta BK-G100).

### Especificaciones

- Suitable for Elster (Kromschröder) and Pipersberg gas meters with BK-G pulse interface
- Can only be used for pulse picking from the mechanical counter - can be retrofitted to all bellows gas meters BK-G 2.5 to BK-G 25
- Can be mounted without impairing the calibration validity
- Attaches to the counter with enclosed mounting clip
- Cable with 1.0 m length, can be used via plug-in connection with pulse receiver, 4 open cable ends (green and brown for pulse, yellow and white = alarm)

# 30. ACTUADORES SONOROS
## 30.1 SONIDO - BUZZER PASIVO KSSG1203-42

Existen dos tipos de buzzer:

- Activos: solo producen un solo nivel de audio
- Pasivos: pueden reproducir melodías

Este zumbador es un zumbador pasivo, lo que significa que requiere una señal oscilante para producir sonido. (Si estas buscando un zumbador que produzca sonido automáticamente con tan solo ser alimentado podes encontrar en nuestras publicaciones el zumbador activo)

### 30.1.1 Especificaciones

- Tamaño: 9 x 11.8 mm
- Voltaje de alimentación: 3.5-5 .5v
- Frecuencia emitida: 2300 ± 500 Hz
- Corriente clasificada (MAX): 30 mA
- Salida de sonido mínima a 10 cm: 85 dB
- Temperatura de funcionamiento: -27 a +70 ° C
- Temperatura de almacenamiento: -30 a 105°C

# 31. ACTUADORES VISUALES

A continuación se presentan los tipos principales:

- **LCD (Liquid Crystal Display)**
  
  Son los más básicos y económicos. Usan una luz de fondo (backlight) y cristales líquidos para bloquearla o dejarla pasar.
- LCD Alfanuméricos (ej. 16x2):
  
  Solo muestran caracteres fijos (letras y números). Son muy robustos y baratos.
- LCD Gráficos (ej. Nokia 5110 o 128x64)
  
  Permiten dibujar puntos (píxeles), pero son monocromáticos (un solo color).Consumo: Medio. Lo que más gasta es la luz de fondo.
  
  Ideal para: Ver datos rápidos como la temperatura del tanque o el nivel de gas sin gastar mucho.

- **OLED (Organic Light Emitting Diode)**
  
  Aquí no hay luz de fondo; cada píxel brilla por sí mismo. El modelo más famoso es el SSD1306 (0.96 pulgadas).Contraste: Infinito (el negro es negro absoluto porque el píxel se apaga).
  
  Consumo: Muy bajo si usas fondos negros con pocas letras blancas.Ángulo de visión: Excelente, se ve perfecto desde cualquier lado.
  
  Punto débil: Se "queman" si dejan la misma imagen fija por meses y se ven mal bajo sol directo.
  
  Ideal para: Proyectos a batería donde necesitás ver mucha info en poco espacio.
- **TFT (Thin-Film Transistor)**
  
  Es una evolución del LCD de matriz activa. Son las pantallas a todo color como las de los celulares viejos.Color: Pueden mostrar fotos, iconos y gráficos complejos a todo color.
  Velocidad: Son rápidas, ideales para menús táctiles.
  Consumo: Alto. La luz de fondo debe ser potente para que se vea algo.
  Ideal para: Interfaces de usuario complejas (un panel de control central en casa, por ejemplo).

- **E-Ink o E-Paper (Papel Electrónico)**
  
  Son como las del Kindle. Solo consumen energía cuando la imagen cambia.
  
  Consumo: Casi cero. Podés mostrar el nivel de batería, desconectar la alimentación, y la imagen queda ahí grabada para siempre.
  
  Visibilidad: Se ven mejor cuanto más sol les da, igual que un papel real.
  
  Punto débil: Tardan mucho en refrescar (parpadean un par de segundos para cambiar de imagen).
  
  Ideal para: Sensores en el campo (AgTech) o etiquetas que se actualizan una vez por hora.

### Tabla Comparativa Rápida

| TIPO | COLOR | CONSUMO | VISIBILIDAD AL SOL| RECOMENDACION |
|:---|:---|:---|:---|:--- |
| LCD 16x2 | No | Medio |Pobre | Tableros eléctricos |
|OLED | No | Dual | Bajo / Media | Nodos a batería |
| TFT | Sí | Alto | Pobre | Pantallas táctiles fijas |
| E-Paper | Rojo | Minimo | Excelente | Sensores de campo |

### ¿Cuál te conviene para tu proyecto solar?

Si vas a usar el panel de 1W:

- OLED: Si necesitás ver datos en el lugar. Podés programar que se apague a los 10 segundos de tocar un botón.

- E-Paper: Si necesitás que el dato esté siempre visible sin gastar nada de la batería 18650.Sin Display: Si vas a mandar los datos por Wi-Fi a tu Home Assistant, no pongas pantalla. Cada mA cuenta cuando el panel es chico.

Las pantallas TFT (Thin-Film Transistor o Transistor de Película Delgada) son un tipo de pantalla de cristal líquido (LCD) que utiliza tecnología de "matriz activa". En términos sencillos para tus proyectos de electrónica: son las pantallas que te permiten mostrar color, gráficos complejos, iconos e incluso imágenes, a diferencia de las pantallas LCD clásicas (como la 16x2 de fondo azul) que solo muestran letras y números fijos.

La diferencia clave es que cada píxel de la pantalla tiene su propio transistor pequeño. Esto permite:

- Mayor Velocidad: La imagen se actualiza rápido (puedes ver animaciones o gráficos que se mueven sin que dejen "estela").
- Mejor Contraste: Los colores son más vivos y definidos.
- Resolución: Pueden tener muchos más píxeles en el mismo tamaño.

Si estás pensando en sumarle una pantalla a tu sistema de monitoreo (por ejemplo, para ver el estado de carga de la batería que estás armando), estas son las variables que vas a encontrar:

- Interfaz de comunicación: La mayoría usa SPI (pocos cables) o Paralelo (muchos cables). Para un ESP32, las SPI son las más recomendadas porque ahorran pines.
- Controladores comunes: Vas a leer nombres como ILI9341, ST7735 o ST7789. Estos son los "cerebros" de la pantalla. Cuando busques código o librerías, necesitás saber cuál tiene la tuya.
- Touch (Táctil): Muchas pantallas TFT vienen con una capa táctil (resistiva o capacitiva), lo que te permite crear menús con botones en la pantalla.

Comparación rápida: TFT vs. OLED

| CARACTERISTICA |TFT | OLED|
|:---|:---|:--- |
| Brillo | Necesita una luz de fondo (backlight) siempre encendida. | Cada píxel brilla por sí mismo.
| Consumo| Alto (por la luz de fondo) | Bajo (especialmente si el fondo es negro)|
| Visibilidad  | Difícil de ver bajo sol directo (ojo con esto para el campo)| Excelente contraste, pero se degrada más rápido al sol|
| Color | Miles o millones de colores | Colores muy vibrantes, negros perfectos|

### Principales Usos:

Si vas a usar el panel solar de 1W y la batería 18650, tené cuidado con las pantallas TFT:

- Consumo: La luz de fondo (backlight) de una TFT puede consumir entre 30mA y 100mA constantemente.
- Gestión de energía: Si la dejás siempre prendida, se va a comer la batería rápido. Lo ideal es que se apague sola después de unos segundos o que tenga un botón físico para "despertarla".

Para un proyecto donde necesitás ahorrar energía (como un sensor en el campo), a veces es mejor una pantalla OLED chiquita (como la SSD1306) o directamente no poner pantalla y ver los datos en el celular por Wi-Fi/Home Assistant.

¿Tenías pensado poner una pantalla para ver el voltaje de la batería en el lugar?

## 31.1 DISPLAY LCD 16x2 con I2C Incorporado - 1602

Módulo de pantalla LCD monocromática con retroiluminación LED. Puede mostrar texto, líneas y cualquier otra forma que se programe, incluso puede mostrar imágenes o logotipos. El controlador es el chip HD44780 con una resolución en pantalla de 16 x 2 caracteres. El módulo I2C adyacente al LCD permite conectar el LCD con 4 cables (2 de alimentación, 2 de datos), simplificando las conexiones y ahorrando puertos en el microcontrolador. El protocolo I2C permite conectar muchos dispositivos sobre un mismo bus de comunicación.

![LCD 16x2](../assets/Display_LCD16x2.png)

### 31.1.1 Especificaciones

- Dimensiones: 80 x 33 x 14 mm
- Pantalla LCD de 16 x 2
- Pantalla en color azul
- Incluye el módulo I2C pines soldados
- Voltaje de alimentación: 5V

## 31.2 DISPLAY LCD - 16x4 - 2004

Display LCD alfanumerico de 20×4. El LCD 2004 cuenta con 4 filas y 20 columnas de dígitos y funciona con el controlador interno HD44780. Para su conexión se necesitan 6 pines: 2 de control y 4 de datos.

![LCD 16x4](../assets/Display_LCD16x4.png)


### 31.2.1 Especificaciones

- Backlight tipo LED
- Color azul.
- Caracteres blancos.
- Interfaces paralela. 5V.
- Controlador compatible HD44780.
- Compatible con las librerías de Arduino.

Los sistemas embebidos como Arduino, Pic u otros, trabajan con lógica binaria (0 y 1), utilizando pantallas como este LCD para mostrar datos en formato alfanumérico, lo que permite un manejo más eficiente de sensores y procesamiento de datos.

## 31.3 DISPLAY OLED 0.96" - Blanco 128x64 I2C Ssd1306

Pantalla de 0.96" OLED. Matriz de 128x64 puntos. No necesita retroiluminación y tiene alto contraste incluso con luz dia.

El driver interno es un SSD1306 que se comunica por I2C, Funciomaniento interno a 3.3V pero adaptada para funcionar en 5V.

![OLED 096](../assets/Display_OLED096.png)

### 31.3.1 Especificaciones

- Tension Fucionamiento: 3V a 5V DC
- Consumo: 80mA (Max)
- Pantalla OLED de alto contraste
- Color Emision: Blanco
- Resolucion: 128x64 pixeles
- Controlador Interno: SSD1306
- Interfaz: I2C
- Angulo de vision: >160º
- Dimensiones PCB: 35.5mm x 33.7mm
- Peso: 6g
- Orden de Pines: GND,VCC
- Temepratura de funcionamiento: -30 a +70 ºC

## 31.4 DISPLAY OLED 1.3" - Azul 128x64 I2C Sh1106

Esta pantalla es muy pequeña (1,3 pulgadas de diagonal) pero muy visible dado su alto contraste OLED. Es una pantalla con una matriz de un color de 128x64 puntos. Dado que la pantalla está basada en la tecnología LED, no necesita retroiluminación y tiene un alto contraste incluso a plena luz del día.

El driver interno es un SH1106 que se comunica por I2C, un protocolo muy rápido ideal para éste tipo de pantallas. Internamente todo el conjunto funciona a 3,3V pero se han acoplado tanto la alimentación como los pines de entrada para funcionar perfectamente a 5V lo que lo hace ideal para utilizar con nuestro microcontrolador favorito de 5V!

El consumo general depende en gran medida de cuantos píxeles están encendidos, pero el consumo medio ronda los 80mA.

![OLED 130](../assets/Display_OLED130.png)

### 31.4.1 Especificaciones

- Pantalla OLED de alto contraste
- Resolución: 128x64 píxeles (Azul)
- Controlador: SH1106
- Interfaz I2C
- Angulo de visión: >160º
- Alimentación: 3 a 5V DC
- Dimensiones: 3.5 cm x 3.4 cm
- Temepratura de funcionamiento: -30 a +70 ºC

## 31.5 DISPLAY OLED 0.91" - 128x32 Ssd1306 I2c Blanco

Display Oled 0.91 pulgadas 128x32 Ssd1306 I2c Blanco Arduino

Pantalla de 0.91 pulgadas de diagonal, altamente visible dado su alto contraste OLED.

Es una pantalla con una matriz de un color de 128 x 32 puntos.

Dado que la pantalla está basada en la tecnología LED, no necesita retroiluminación ya que tiene un alto contraste, incluso a plena luz del día.

El driver interno es un SSD1306 que se comunica por I2C, un protocolo muy rápido ideal para éste tipo de pantallas.

Internamente todo el conjunto funciona a 3.3V, pero tanto la alimentación como los pines de entrada pueden trabajar a 5V, lo que lo hace ideal para utilizar en cualquier microcontrolador de 5V.

![OLED 091](../assets/Display_OLED091.png)

### 31.5.1 Especificaciones

- Tamaño de la pantalla diagonal: 0.91 "
- Cantidad de píxeles: 128 × 32
- Interfaz: I2C
- Controlador: SSD1306
- Profundidad del color: Monocromo (blanco)
- Paso de píxel (mm): 0.175 × 0.175
- Tamaño del píxel (mm): 0.159 × 0.159
- Dimensiones: 12.0 mm x 39 mm (0.47 "x 1.5")
- Área de visualización: 7 mm x 25 mm
- Espesor: 4 mm

## 31.6 DISPLAY TFT - REDONDO 1,28" 240x240 Rgb

El TFT Display 1.28 Inch TFT LCD Display Module es una pantalla LCD redonda de alta resolución diseñada para una amplia gama de aplicaciones electrónicas y proyectos DIY. Con su formato compacto y su capacidad de mostrar gráficos en color vibrantes, este módulo es ideal para desarrolladores y entusiastas que buscan una solución visual de calidad para sus proyectos con Arduino y otros microcontroladores.

![TFT 128](../assets/Display_TFT128.png)

### 31.6.1 Características

- Pantalla TFT LCD de 1.28 pulgadas:
- Formato Circular: La pantalla redonda añade un toque único y atractivo a tus proyectos, ideal para aplicaciones donde el diseño visual es importante.
- Alta Resolución: Con una resolución de 240x240 píxeles, proporciona imágenes nítidas y detalladas, perfectas para gráficos y texto.
- Tecnología RGB y IPS:
- Colores Vibrantes: La tecnología RGB asegura una reproducción precisa y vibrante de los colores.
- Amplios Ángulos de Visión: La tecnología IPS (In-Plane Switching) garantiza que los colores y la claridad de la imagen se mantengan constantes desde diferentes ángulos de visión.
- Controlador GC9A01:
- Interfaz SPI de 4 Hilos: Utiliza una interfaz de comunicación SPI de 4 hilos, lo que permite una transferencia de datos rápida y eficiente. Esto facilita la integración con microcontroladores como Arduino.
- Compatibilidad: El controlador GC9A01 es compatible con una amplia variedad de microcontroladores, proporcionando flexibilidad en el desarrollo de proyectos.
- Fácil Integración:
- Diseño de PCB: Viene con un PCB diseñado para facilitar la conexión y el montaje en proyectos electrónicos.
- Compatibilidad con Arduino: Totalmente compatible con Arduino y otras plataformas de desarrollo, lo que facilita su uso en una variedad de proyectos.

### 31.6.2 Especificaciones

- Tipo de Pantalla: TFT LCD
- Tamaño: 1.28 pulgadas
- Resolución: 240x240 píxeles
- Tecnología: RGB, IPS
- Controlador: GC9A01
- Interfaz: SPI de 4 hilos
- Voltaje de Operación: 3.3V a 5V

## 31.7 DISPLAY TFT 1,44" - ST7735

Módulo display TFT color 1.44 pulgadas con controlador ST7735 y resolución 128×128 píxeles.

Este display permite mostrar gráficos, texto, iconos e interfaces completas a color, ideal para proyectos con Arduino, ESP8266, ESP32 y Raspberry Pi.

Utiliza interfaz SPI, lo que permite una conexión simple utilizando pocos pines.

![TFT 144](../assets/Display_TFT144.png)

### 31.7.1 Características

- Pantalla: 1.44 pulgadas
- Resolución: 128 × 128 píxeles
- Controlador: ST7735
- Interfaz: SPI
- Pantalla TFT color
- Bajo consumo
- Compatible con Arduino y ESP32
- Tamaño de la pantalla: 1,44 "
- Color de la luz de fondo: RGB
- Dimensiones: 40 x 50 mm
- Altura total: 3 mm

## 31.8 DISPLAY TFT 2.2" SPI

El Módulo Display LCD TFT de 2.2 pulgadas es ideal para tus proyectos con Arduino y Raspberry Pi, gracias a su interfaz de 4 hilos SPI que facilita la conexión.

Con una resolución de 176 x 220 puntos, este dispositivo ofrece imágenes nítidas y vibrantes gracias a su profundidad de color de 262K/65K. Este módulo se alimenta con 5V y opera a una lógica de 3V3, siendo versátil para diversas aplicaciones. Su tamaño compacto de 67 mm de longitud, 40 mm de ancho y apenas 4 mm de grosor permite integrarlo fácilmente en proyectos con espacio limitado. El paquete del vendedor incluye un diseño cuidadoso, con dimensiones de 14 cm de altura, 7 cm de largo y 10 cm de ancho, pesando solo 80 g. Recuerda que la imagen es solo ilustrativa y puede variar según la partida.

![TFT 220](../assets/Display_TFT220.png)

### 31.8.1 Especificaciones

- Pantalla: 2.2 pulgadas
- Resolucion: 176 x 220 pixeles
- Profundidad de color: 262/65K
- Dimensiones: 67 x 40 mm
- Espesor: 4 mm
- Peso: 80 g

## 31.9 E-PAPER E-ink 2,66" Para Cfd

## 31.10 E-PAPER HAT E-INK 2,7"

Módulo eInk de 2,7 pulgadas ePaper HAT eInk de 2 colores para Raspberry Pi/Arduino Características: sin luz de fondo, sigue mostrando el último contenido durante mucho tiempo incluso cuando se apaga. Consumo de energía ultrabajo, básicamente solo se necesita energía para refrescar.
![EPAPER 270](../assets/Display_EINK270.png)
### 31.10.1 Descripción

Esta pantalla E-Ink HAT, 2,7 pulgadas, resolución 264x176, es especialmente diseñado para Raspberry Pi. Viene con un controlador integrado que se comunica a través de la interfaz periférica en serie. Debido a sus ventajas, como el consumo de energía ultrabajo, el amplio ángulo de visión y el gran efecto bajo la luz solar, es una opción ideal para aplicaciones como etiquetas de estanterías, instrumentos industriales, etc.

Características: sin luz de fondo, muestra el último contenido durante mucho tiempo incluso cuando está apagado. Consumo de energía ultrabajo, básicamente solo se necesita energía para actualizar. Viene con cuatro botones para la expansión de las aplicaciones

Adecuado para la interfaz periférica serie Raspberry Pi 2B/3B/Zero/Zero W. para conectarse con otras placas controladoras como para A-RDuino/Nucleo, etc. Viene con recursos de desarrollo y un manual (ejemplos para Raspberry/A-RDuino/STM32).

### 31.10.2 Especificaciones

- Voltaje de funcionamiento: 3,3 V
- Interfaz: interfaz periférica en serie de 3 hilos, interfaz periférica en serie de 4 hilos
- Dimensiones del contorno: aproximadamente 85 mm x 56 mm
- Tamaño de la pantalla: aproximadamente 57,288 mm x 38,192 mm
- Distancia entre puntos: 0,217 x 0,217
- Resolución: 264 x 176
- Color de la pantalla: aproximadamente 57,288 mm x 38,192 mm
- Distancia entre puntos: 0,217 x 0,217
- Resolución : 264 x 176
- Color de la pantalla: negro, blanco
- Nivel de gris: 2
- Tiempo de actualización completo: 6 segundos
- Potencia de actualización: 24 mW (típica)
- Potencia de espera: 170
- Peso del paquete: 42 g
- Interfaz: VCC: 3,3 V
- GND: GND
- DIN: interfaz periférica en serie
- PIN CLK: interfaz periférica en serie
- PIN SCK : selección de chips de interfaz periférica en serie,
- DC baja activa: selección de datos/comandos (alto para datos, bajo para comando)
- RST: reinicio externo, ocupado bajo: salida de estado ocupado, poco activo

# 32. ACTUADORES FISICOS

### Introducción

Un relay (o relé) es básicamente un interruptor automático que permite controlar una corriente eléctrica muy grande mediante una corriente muy pequeña.

En términos prácticos para tus proyectos con la ESP32, el relay actúa como un "traductor": permite que la placa (que maneja solo 3.3V o 5V) pueda encender o apagar algo que funciona a 220V (como una bomba de agua, una luz o un motor) sin que la placa se queme.

### ¿Cómo funciona por dentro?

Un relay tradicional (electromecánico) tiene dos partes principales:

- La Bobina (El Electroimán): Cuando le aplicas un poquito de electricidad (por ejemplo, desde un pin de tu ESP32), se genera un campo magnético.
- El Interruptor (Contactos): Ese magnetismo atrae físicamente una pieza de metal que "golpea" un contacto y cierra el circuito, dejando pasar la corriente de alta potencia.

Si mirás un módulo de relay estándar, vas a ver tres bornes para conectar la carga:

- COM (Común): Es donde entra el cable de fase que querés interrumpir.
- NO (Normalmente Abierto): El circuito está abierto (apagado) por defecto. Solo se activa cuando la ESP32 envía la señal. Es el que más vas a usar.
- NC (Normalmente Cerrado): El circuito está cerrado (encendido) por defecto. Se corta cuando la ESP32 envía la señal.

### Optoacoplado

Que algo sea optoacoplado significa que hay un "puente de luz" que separa dos partes de un circuito eléctrico para que no se toquen físicamente. Es la forma definitiva de proteger tu ESP32 de cualquier descarga o ruido eléctrico proveniente de motores, relays o la red de 220V.

¿Cómo funciona?

- Dentro de un componente llamado optoacoplador (que parece un pequeño chip negro de 4 patas), hay dos elementos:
- Un LED: Que recibe la señal de tu ESP32 y se enciende.
- Un Fototransistor: Que "ve" la luz del LED y deja pasar la corriente del otro lado.
- Como la señal viaja a través de la luz y no de cables, si del lado de "fuerza" (donde tenés los 220V o el motor) ocurre un cortocircuito, la electricidad no puede "saltar" hacia tu ESP32 porque no hay conexión física. El chip simplemente se quema por dentro, pero salva a tu microcontrolador.

### ¿Por qué es importante para vos?

- Como estás armando un sistema para el medidor de gas y mencionaste fuentes aisladas y relays, el optoacoplamiento es tu mejor aliado por estas razones:
- Protección Total: Evitás que un pico de tensión de la red eléctrica llegue a los pines de la ESP32.
- Aislamiento de Ruido: Los motores o bobinas de los relays generan "ruido" eléctrico que hace que las placas inteligentes se tilden o se reinicien. El optoacoplador limpia esa señal.
- Seguridad: En entornos como un nicho de gas, mantener la electrónica de control separada de la potencia es una regla de oro.

### ¿Cómo saber si tu módulo es optoacoplado?

- Si compras un módulo de relay, vas a ver un componente cuadrado muy chiquito (generalmente con el código PC817). Eso es el optoacoplador.
- Jumper
- Muchos de estos módulos traen un jumper (un puentecito de plástico) que dice JD-VCC.
- Si sacás el jumper: Podés alimentar la bobina del relay con una fuente y la ESP32 con otra, logrando un aislamiento real y total.
- Si dejás el jumper: Comparten la misma energía y el aislamiento es solo parcial.
- En resumen: es un seguro de vida para tu ESP32. Si vas a conectar la cámara a algo que no sea solo su fuente (como un relay o un sensor industrial), buscá siempre que el módulo sea optoacoplado.

## 32.1 MODULO RELAY OPTOACOPLADO 2 CANALES 5VDC / 10A

Relé de alta corriente, AC250V 10A, DC30V 10A

2 LED para indicar cuando los relés están encendidos

Funciona con señales de nivel lógico de dispositivos de 3.3V o 5V.

Circuito de aislamiento opto

Tamaño de PCB: 50 x 45 mm

![RELAY](../assets/Relay_Optoacoplado.png)


## 32.2 MOTOR NEMA 17

Un motor NEMA 17 es un tipo de motor paso a paso (stepper motor) que recibe su nombre por las dimensiones de su cara frontal, estandarizadas por la National Electrical Manufacturers Association. En este caso, el número 17 indica que la placa frontal del motor mide 1.7 x 1.7 pulgadas (aproximadamente 42 x 42 mm).

A diferencia de un motor de corriente continua (DC) común que gira continuamente cuando recibe energía, un motor paso a paso divide una rotación completa en un número determinado de "pasos" iguales.

La mayoría de los NEMA 17 tienen un ángulo de paso de 1.8°, lo que significa que necesitan 200 pasos para completar una vuelta de 360°. Esto permite un control de posición extremadamente preciso sin necesidad de sensores externos (encoders).

### 32.2 Componentes y Conexión

Internamente, tiene bobinas que se activan en una secuencia específica para mover el eje.

- Fases: Generalmente son bipolares y tienen 4 cables (dos para cada bobina interna).
- Controlador (Driver): No se pueden conectar directo a la batería o fuente; necesitan un controlador (como el TMC2209 o A4988) que traduzca las señales de un microcontrolador (como tu ESP32) en impulsos eléctricos para las bobinas.

Este motor es un NEMA 17 de la marca Usongshine, modelo 17HS4401S.

Es uno de los motores paso a paso más comunes y versátiles para proyectos de impresión 3D (como la Ender 3), CNC pequeños y automatización general.

Aquí tienes las especificaciones técnicas principales para este modelo:

Características Eléctricas y Mecánicas

- Tipo de motor: Bipolar Paso a Paso.
- Ángulo de paso: 1.8° (200 pasos por vuelta).
- Corriente nominal: 1.5 A a 1.7 A por fase.
- Torque de mantenimiento (Holding Torque): Aproximadamente 40 N.cm (4 kg.cm).
- Resistencia de fase: 1.5 \$\\Omega \\pm 10\\%\$.
- Inductancia de fase: 2.8 mH \$\\pm 20\\%\$.
- Voltaje recomendado: Funciona bien con controladores de 12V a 24V (como el A4988 o TMC2209).
- Dimensiones y Conexión:
- Tamaño del cuerpo: 42mm x 42mm (NEMA 17).
- Longitud del motor: 40mm (excluyendo el eje).
- Diámetro del eje: 5mm (tipo "D-cut" o circular, según la versión exacta).
- Conector: Generalmente utiliza un conector PH-6 de 6 pines en el motor, que sale a 4 cables para el controlador.
- Pinout Común (4 cables) Si el cable sigue el estándar de colores típico para este modelo, los pares de bobinas suelen ser:
- Fase A: Negro y Verde.
- Fase B: Rojo y Azul.

Nota de seguridad: nunca desconectes el motor mientras el controlador esté encendido, ya que el pico de inducción puede quemar el driver (como un A4988, DRV8825 o TMC2209).

### Identificación Típica de Colores

Aunque los colores pueden variar según el fabricante, el estándar más común en los Nema 17 suele ser:

|BOBINA |PAR DE COLORES
|:---|:---
|Bobina A |Negro y Verde
|Bobina B |Rojo y Azul

## 32.3 DRIVER Tmc2209 Spi c/Disipador Mks

Un driver (o controlador) de motor es el "traductor" entre el cerebro de tu proyecto (el ESP32-C3) y el músculo (el motor NEMA 17).

Como el microcontrolador solo puede entregar una señal de muy baja potencia (3.3V y pocos miliamperios), no tiene la fuerza suficiente para mover las bobinas de un motor que requiere 12V y más de 1 Amper. El driver soluciona esto actuando como un puente de potencia.

### ¿Por qué es necesario un driver?

- Manejo de Corriente: El driver recibe una señal digital débil del ESP32 y la convierte en una corriente fuerte (proveniente de tu fuente externa de 12V/24V) para alimentar el motor.
- Secuencia de Pasos: Para que un motor paso a paso gire, hay que encender y apagar sus bobinas internas en un orden muy preciso. El driver hace este trabajo sucio por ti; tú solo le dices "da un paso" y él sabe qué bobinas activar.
- Microstepping (Pasos fraccionados): Drivers avanzados como el TMC2209 pueden dividir un paso físico en partes más pequeñas (hasta 256 micropasos). Esto hace que el motor se mueva de forma mucho más suave, silenciosa y sin vibraciones, algo vital para que tu riel de cortina no haga ruido.
- Protección: Protege al ESP32 de posibles picos de voltaje que genera el motor al girar o detenerse.

El TMC2209: Un driver "inteligente"; el TMC2209 no es un driver común, tiene funciones especiales:

- StealthChop2: Es una tecnología que elimina casi por completo el ruido del motor. Es la razón por la que las impresoras 3D modernas son silenciosas.
- StallGuard: Puede detectar si la cortina se trabó o llegó al final del riel midiendo la resistencia eléctrica, lo que te permitiría hacer un "homing" (volver a cero) sin usar sensores físicos.
- Control de temperatura: Se ajusta para no quemarse, aunque siempre es bueno ponerle el disipador de aluminio.
- TMC2209 V3.0 es un chip de transmisión de motor paso a paso silencioso de dos fases, capacidad de accionamiento de hasta 1.7A (RMS) Corriente de bobina continua-2.8A pico
- La Unidad de interpolación de micropasos flexible puede proporcionar hasta 256 de subdivisión, incluso en un sistema con frecuencia de pulso limitada, el control senoidal se puede realizar perfectamente.
- Controlador de motor paso a paso TMC2209 V3.0 compatible con StepStick y Pololu A4988.
- TMC2209 V3.0 con interfaz estándar de paso/dir, configuración a través de pines CFG o interfaz UART, es simple y conveniente de usar.
- El motor controlado por el modo chopper PWM funciona más suavemente y evita perder pasos y Jittering.

### Esquema de Conexión (PIN OUT)

### Pines UART

En este esquema los pines UART son los marcados como RX y TX.

- RX (Receive): Es el pin por donde el driver recibe datos y comandos desde tu microcontrolador (MCU).
- TX (Transmit): Es el pin por donde el driver envía información hacia tu microcontrolador (por ejemplo, el estado de diagnóstico, la temperatura o si detectó un error).

Consideraciones importantes para usar UART:

- Conexión simple: En muchos módulos TMC2209 "stepstick" (los que tienen pines en fila), los pines TX y RX suelen estar separados. Para comunicarte con el driver, debes conectar el TX del MCU al RX del driver, y el RX del MCU al TX del driver.
- El "truco" del puente: En algunos módulos específicos (especialmente los diseñados para impresoras 3D), los pines TX y RX están conectados internamente mediante una resistencia, o requieren un puente de soldadura en la placa para habilitar la comunicación UART.
- Direccionamiento: Si vas a conectar más de un driver al mismo puerto UART de tu MCU, necesitarás configurar los pines MS1 y MS2 (usados como direcciones AD0/AD1) para que cada driver tenga una dirección única en el bus.

### Tres pines adicionales

Esos tres pines que ves formando un pequeño triángulo (generalmente situados cerca del borde, por encima de los pines de control como ENABLE o MS1/MS2) son pines de diagnóstico y configuración adicional.

Dependiendo de la versión específica de la placa (PCB) que tengas, suelen ser estos tres:

- DIAG (Diagnostic): Este pin sirve para reportar errores. Se activa (normalmente a nivel lógico alto) si el driver detecta un problema, como un sobrecalentamiento, un cortocircuito en las bobinas o si se usa la función de "StallGuard" (detección de atascos) para hacer "homing" sin necesidad de finales de carrera físicos.

- INDEX: Este pin emite un pulso cada vez que el motor completa una vuelta completa (o un ciclo específico de micro-pasos). Es muy útil si necesitas sincronizar la posición del motor con mucha precisión en aplicaciones industriales o de posicionamiento.

- VREF (o pin de testeo): A veces, uno de esos pines está conectado directamente al potenciómetro para que puedas medir el voltaje de referencia Vref con un multímetro de forma más cómoda. El Vref es lo que determina el límite de corriente que el motor puede consumir.

¿Para qué sirven en la práctica?
Si eres principiante: Lo más probable es que no necesites conectarlos. Para la mayoría de los proyectos (como mover un eje de una impresora 3D o un brazo robótico simple), el driver funciona perfectamente solo con los pines de los lados (STEP, DIR, ENABLE, alimentación y motor).

Si quieres funciones avanzadas:

Puedes usar DIAG para configurar el "Sensorless Homing" (que el motor sepa dónde está el inicio del recorrido sin usar interruptores).

Puedes usar el pin de VREF para ajustar la potencia del motor con precisión mientras lo mides con un multímetro para evitar que el motor se caliente demasiado o pierda pasos por falta de fuerza.

Un consejo importante: Como esos pines están físicamente muy cerca, si estás usando una protoboard (breadboard), ten mucho cuidado de no hacer un puente accidental entre ellos y los pines de al lado, ya que podrías enviar un voltaje de potencia (VMOT) a un pin lógico y dañar el driver o tu microcontrolador.

### Configurar funciones avanzadas - Sensorless Homing

Normalmente, una impresora 3D o máquina CNC usa un interruptor físico (endstop) para saber dónde empieza el recorrido. Con el TMC2209, el driver puede "sentir" cuando el motor se detiene porque ha llegado al tope mecánico, gracias a que detecta un aumento en la carga (Back-EMF).

Pasos para implementarlo:

- Conexión Física: Debes conectar el pin DIAG del driver a un pin de interrupción digital de tu microcontrolador (MCU).
- Configuración UART: Como esto requiere parámetros precisos, es obligatorio usar el modo UART. No se puede configurar de forma fiable solo con pines físicos.

Ajuste por Software:

- A través del protocolo UART, debes enviar comandos al registro TMC2209_SGTHRS. Este valor define la "sensibilidad" de la detección de choque.
- Si el valor es muy bajo, el driver creerá que el motor chocó apenas empiece a moverse.
- Si el valor es muy alto, el motor chocará contra el tope mecánico y no se detendrá, lo cual puede ser peligroso para la mecánica.

### Consideración Técnica sobre los "3 pines"

Dependiendo de la marca de tu módulo (como BIGTREETECH o Makerbase), a veces el pin DIAG no está conectado internamente al pin que sale hacia afuera. En muchas placas verás un pequeño pad o "jumper" (una gota de estaño) que debes unir para habilitar la señal de diagnóstico hacia el pin físico de la placa.

## CONEXION Motor NEMA 17 a DRIVER Tmc2209

Para conectar un controlador TMC2209 (que es mucho más silencioso y eficiente que los básicos), la conexión dependerá de si lo vas a usar en modo Step/Dir (manual) o mediante UART (control por software).

Para conectar un driver TMC2209 a un NodeMCU ESP8266, debes tener en cuenta que el driver requiere dos tipos de alimentación: una para la lógica (3.3V) y otra para el motor (usualmente 12V o 24V).

Como estás usando un ESP8266, es fundamental que la parte lógica se mantenga en 3.3V.

### Esquema de Conexión (PIN OUT): Modo Step/Dir

- Esta es la configuración básica para controlar el motor mediante pulsos de pasos y dirección:


### Esquema de conexión (PIN OUT): Modo UART

Conexión de Potencia (Motor)

- VM / VMOT: Conectar al positivo de tu fuente externa (ej. 12V).
- GND (Potencia): Conectar al negativo de la fuente externa. Importante: Une esta tierra con el GND del NodeMCU.
- A1, A2, B1, B2: Conectar a las cuatro bobinas del motor paso a paso (Nema 17).

Consideraciones Críticas:

- Capacitor de protección: Es obligatorio colocar un capacitor electrolítico (de unos 100 uF) entre VMOT y GND lo más cerca posible del driver para protegerlo de picos de tensión.
- Modo UART: Si quieres configurar la corriente o el microstepping por software, deberás conectar el pin PDN/UART del TMC2209 a un pin TX/RX del ESP8266 usando una resistencia de 1k Ohm en el medio para la comunicación half-duplex.
- Disipación de calor: Los TMC2209 se calientan considerablemente. Asegúrate de pegar el disipador de aluminio sobre el chip y, si es posible, usar un pequeño ventilador, especialmente si vas a mover cortinas o mecanismos pesados.
- Configuración de Pines: Recuerda que en ESPHome o Arduino IDE, debes usar el número de GPIO o la constante de la placa (ej: D1 o 5).

Aquí tienes la guía para la conexión estándar (Step/Dir), que es la más común para integrarlo con microcontroladores:

### Esquema de Pines (Pinout)

El TMC2209 suele venir en formato "StepStick". Debes prestar mucha atención a la orientación para no quemarlo.

- VCC / VIO: Alimentación lógica (3.3V o 5V, según tu microcontrolador).
- GND: Tierra lógica y tierra de potencia (conéctalas ambas).
- VMOT: Alimentación de los motores (8V - 28V). Importante: Coloca un capacitor electrolítico (ej. 100µF) entre VMOT y GND para proteger el driver de picos de tensión.
- STEP: Pulso para mover el motor.
- DIR: Dirección del giro.
- EN (Enable): Generalmente se activa poniéndolo en nivel bajo (GND).
- 1A, 1B, 2A, 2B: Salidas hacia las bobinas del motor 17HS4401S.

### Conexión al Motor 17HS4401S

- Siguiendo los pares que identificamos antes, la conexión sería:
- 2B / 2A: Bobina A (Rojo / Azul)
- 1A / 1B: Bobina B (Negro / Verde)
- Si el motor gira al revés, simplemente invierte el orden de una de las bobinas en el software o da vuelta el conector.

### Configuración de Corriente (Vref)

- Antes de darle potencia, debes ajustar el potenciómetro diminuto que trae el driver para limitar la corriente y no sobrecalentar el motor.
- Para el motor 17HS4401S (1.5A - 1.7A), una configuración segura de Vref suele estar entre 0.9V y 1.2V. La fórmula aproximada es: Vref = (Irms 1.41)/2.5
- O más simple: Mide con un multímetro entre el tornillo del potenciómetro y GND, y ajústalo a 1.0V para empezar.

### Consideraciones Especiales

Modo StealthChop:

- El TMC2209 viene por defecto en modo ultra-silencioso. Si notas que el motor pierde fuerza en altas velocidades, podrías necesitar configurar el pin SpreadCycle.
- Sensorless Homing: Este driver permite detectar cuando el motor choca (StallGuard). Si no vas a usar esto, asegúrate de que el pin de diagnóstico (DIAG) no esté interfiriendo con tus finales de carrera físicos.
- Disipación: El TMC2209 genera calor por la parte inferior. Asegúrate de pegar el disipador de aluminio que trae sobre el integrado y, si es posible, usa un pequeño ventilador.

Para conectar el TMC2209 a un ESP32-C3 SuperMini, debes tener especial cuidado con el espacio, ya que la placa es muy pequeña, y con los voltajes. El C3 trabaja a 3.3V, que es perfectamente compatible con la lógica del driver.

Aquí tienes el esquema de conexión paso a paso:

- Diagrama de Conexión (Modo Step/Dir)

# 40. FUENTES  & PANELES SOLARES

### Fuentes Switching

Una fuente switching (o fuente conmutada) es un dispositivo electrónico que convierte la electricidad de la red eléctrica (ej. 220V CA en Argentina) al voltaje y corriente continua que necesitan tus dispositivos (ej. 12V CC para tu motor NEMA 17).

A diferencia de las fuentes antiguas (llamadas lineales), que eran pesadas, grandes y desperdiciaban mucha energía en forma de calor, las switching son compactas, livianas y muy eficientes.

Se llaman "switching" (conmutada) porque utiliza un transistor que se abre y se cierra miles de veces por segundo (como un interruptor de alta velocidad).

- Rectificación: Toma la corriente alterna (CA) de la pared y la convierte en continua de alto voltaje.
- Conmutación (Switching): El interruptor electrónico "trocea" esa corriente a una frecuencia altísima (usualmente más de 20,000 veces por segundo).
- Transformación: Esa corriente troceada pasa por un transformador mucho más pequeño que los tradicionales.
- Salida: Se rectifica y filtra nuevamente para entregar un voltaje estable (como los 12V que necesitás).

### Fuentes Aisladas

Una fuente aislada es aquella que tiene una separación física y eléctrica total entre la entrada (los 220V de la red) y la salida (los 12V que van a tu motor y al ESP32).

En términos simples: no hay conexión directa de cables entre la electricidad peligrosa de la pared y los componentes que vos tocás.

### ¿Cómo funciona el aislamiento?

Para pasar la energía de un lado al otro sin que los cables se toquen, utilizan un transformador. La energía "salta" de un bobinado a otro a través de un campo magnético.

¿Por qué es importante para tu proyecto?

Seguridad Personal: Si hay una falla en la fuente, el aislamiento evita que los 220V pasen al eje del motor, al riel de metal de la cortina o al puerto USB de tu computadora si la tenés conectada para programar. Es lo que evita que "te dé una patada" al tocar el equipo.

Protección de Electrónica: Protege al ESP32-C3 de ruidos eléctricos y picos de tensión de la red. Si cae un rayo cerca o hay un problema en la línea, es mucho más probable que solo se queme la fuente y no todo tu sistema (motor, driver y microcontrolador).

Evita "Bucles de Tierra": Si conectás el ESP32 a la PC mientras está alimentado por la fuente, una fuente aislada previene que circulen corrientes raras entre la PC y el riel, lo cual suele quemar los puertos USB.

### ¿Cómo saber si tu fuente es aislada?

Casi todas las fuentes switching modernas de buena calidad (como los cargadores de notebooks o los "bricks" de 12V que parecen de plástico cerrado) ya son aisladas.

Podes identificar una fuente segura buscando estos símbolos en la etiqueta:

El símbolo de "Doble Aislamiento": Son dos cuadrados, uno dentro del otro.

Transformador con barrera: Un dibujo de dos bobinas con una línea en el medio.

Recomendación para tu Riel

Como vas a tener un motor montado en un riel metálico y cerca de una cortina:

Usá siempre una fuente aislada. No uses fuentes baratas de mala calidad que no tengan certificaciones de seguridad (como el sello de Seguridad Eléctrica en Argentina).

Si la fuente es metálica (tipo panal), asegurate de conectar el borne de Tierra (PE) a la instalación de tu casa.

## 40.1 FUENTE SWITCHING AISLADA 220VAC a 3,3VDC - 1,5A / 5w

MODULO HLK-5M03

Convierte corriente CA a CC con el Módulo de Fuente de Alimentación HLK-5M03 de 3,3V 5W. Compacto y eficiente, cumple con normas UL/CE, proporcionando alta densidad de potencia, bajo ruido y protección contra sobrecarga y cortocircuito.

Ideal para aplicaciones médicas, industriales, instrumentación y comunicación.

Su diseño resistente incluye aislamiento de 3000VAC y carcasa anti-polvo, anti-humedad y retardante de llama. Opera en un rango de entrada de 85-264 V CA con consumo en vacío inferior a 0.1W, garantizando seguridad y eficiencia en un formato ultra compacto.

El Módulo de Fuente de Alimentación HLK-5M03 de 3,3V 5W es una solución de conversión CA a CC diseñada para ofrecer rendimiento superior y alta eficiencia. Cumple con certificaciones de seguridad UL/CE y EMC, garantizando un funcionamiento fiable en aplicaciones industriales, médicas y de comunicación. Su diseño compacto y de bajo consumo energético reduce las pérdidas a menos de 0.1W en vacío, contribuyendo a la eficiencia energética y la protección ambiental.

Fabricado con aislamiento de 3000VAC y protección térmica avanzada, el módulo soporta condiciones extremas de operación, incluyendo temperaturas de -25°C a +60°C y vibraciones de transporte secundario. Su carcasa con revestimiento de pegamento termo-conductor asegura protección contra polvo, humedad y choques, prolongando la vida útil a más de 100,000 horas de funcionamiento continuo.

Las características de entrada admiten un rango universal de 85-264 V CA o 70-350 V CC, con una corriente de arranque baja de menor o igual que 10A y un fusible recomendado de 1A/250V para seguridad adicional. Su salida regulada a 3,3 ofrece estabilidad con un margen de aproximadamente 0.2% y protección automática contra sobrecarga, manteniendo tus dispositivos a salvo.

Este módulo es ideal para fuentes de alimentación de dispositivos electrónicos, automatización industrial y sistemas de monitoreo. Su compatibilidad con pruebas de radiación y descarga electrostática (IEC/EN 61000-4-2) lo convierte en una opción robusta y segura para proyectos avanzados.

![HLK-5M02](../assets/Fuente_HLK5M03.png)

### 40.1.1 Especificaciones

- Modelo: HLK-5M03
- Tensión nominal de entrada: 100-240 V CA
- Máxima corriente de entrada: menor o igual que 0,2 A
- Tensión de entrada máxima: 270 V CA
- Arranque lento de entrada: menor o igualque 50 mS
- Fiabilidad a largo plazo: MTBF mayor o igual que 100,000 horas
- Sin carga tensión de salida: 3,3 +- 0,2 %
- Carga completa tensión de salida: 3,3 +- 0,2 %
- Corriente de salida: 5W
- Regulación de tensión: +- 0,2%
- Regulación de carga: +- 0,5%
- Ondulación de salida y ruido: mayor o igual que 100 mVp-p
- Sobrecorriente de salida protección: Carga máxima de 110-150%
- Protección contra cortocircuito de salida: Sí
- Tamaño: 3,8 x 2,3 x 1,5 cm (Largo x Ancho x Alto)
- Peso: 32 g

## 40.2 FUENTE SWITCHING AISLADA 220VAC a 5VDC - 0,7A / 3,5w

![FUENTE 5VDC](../assets/Fuente_05VDC.png)

### 40.2.1 Especificaciones

- Entrada: AC 50V-277V
- Salida: 5V DC
- Corriente de Salida Max: 700mA
- Potencia Maxima: 3,5W
- Proteccion contra sobretensión, temperatura y cortocircuito

## 40.3 FUENTE SWITCHING AISLADA 220VAC a 12VDC - 2A / 24w

Fuente de alimentación aislada, con protección de temperatura, sobrecorriente y cortocircuito, tamaño pequeño, fiable.

![FUENTE 12VDC](../assets/Fuente_12VDC.png)


### 40.3.1 Especificaciones

- Voltaje de entrada: 100V-265V AC
- Voltaje de salida: 12V DC (± 0.3V)
- Corriente de salida Constante: 2A
- Potencia de salida: 24W
- Ripple: <40mV
- Protecciones: Corto circuito / Sobretension / Sobre corriente / Temperatura
- Dimensiones: 69mm x 36mm x 35mm (Largo x Ancho x Altura)

## 40.4 MODULO DE CARGA TP4056

Este módulo de carga de baterías de litio TP4056 con protección, específicamente la versión que utiliza un puerto USB-C.Es un estándar en el mundo de la electrónica DIY y el desarrollo con microcontroladores (como los ESP32 que usas) porque permite cargar y proteger celdas Li-ion de 3.7V (como las populares 18650) de forma segura y sencilla.

El módulo se divide en dos funciones principales:

- Cargador (Chip CSM4056T / TP4056): Gestiona la carga de la batería mediante un ciclo de corriente constante/voltaje constante (CC/CV). Por defecto, viene configurado para cargar a 1A.
- Protección (Chips DW01A y 8205A): Esta versión incluye un circuito de protección que evita que la batería se dañe. Sus funciones son:
  - Sobrecarga: Corta la carga si el voltaje supera los 4.2V.
  - Sobredescarga: Corta el suministro si la batería baja de ~2.4V (evita que la batería muera permanentemente).
  - Sobrecorriente/Cortocircuito: Protege el circuito si hay un consumo excesivo.

![TMP4056](../assets/Carga_TP4056.jpg)

### 40.4.1 Esquema de conexiones (PIN OUT)

Mirando la placa con el USB-C hacia la izquierda:

| PIN |FUNCION |
|:--- |:---|
| B+ | Conectar al polo positivo de la batería de litio |
| B- | Conectar al polo negativo de la batería de litio|
| OUT+ | Salida positiva hacia tu circuito (ej. tu ESP32 o sensores)|
| OUT- | Salida negativa hacia tu circuito. |
| Pad +/- | Al lado del USB, permiten soldar una entrada de 5V externa si no quieres usar el conector. |

### 40.4.2 Recomendaciones de Uso

- Carga y Uso simultáneo: Puedes alimentar tu proyecto a través de los pines OUT mientras la batería se carga. Sin embargo, si el consumo de tu circuito es muy alto, el cargador podría no detectar correctamente cuándo terminar la carga.
- LEDs de estado:
  - Rojo: Cargando.
  - Azul (o Verde): Carga completa.
- Disipación de calor: El chip central puede calentarse bastante si cargas a 1A de forma constante. Es normal, pero asegúrate de que tenga algo de ventilación si va dentro de una caja estanca.

Es una pieza excelente para tus proyectos de monitoreo, ya que te permite hacerlos portátiles o agregarles una batería de respaldo (UPS) de forma muy barata.

## 40.5 STEP DOWN LM2596 - DC 1,25A / 35VDC - 3A

Un módulo Step-Down sirve como regulador de tensión, entregando voltajes de salida menores que los de su entrada.

Fuente basada en regulador step-down DC-DC LM2596 con salida regulable de 1.25V a 35V

Con preset multivuelta de alta precisión es capaz de alimentar cargas de hasta 3A.

Para corrientes de salida mayores a 2A, se debe utilizar un disipador de calor sobre el integrado.

![LM2596](../assets/Step_Down_LM2596.png)

### 40.5.1 Especificaciones

- Voltaje de operación: 4.0V ~ 40V DC
- Voltaje de Salida: 1.23V ~ 35V DC Ajustable (el voltaje de entrada debe tener al menos 1.5V más que la salida).
- Corriente de Salida: máx. 3A (usar disipador para corrientes mayores a 2A).
- Potencia de salida: 50-70W, utilizar disipador
- Eficiencia de conversión: 92%
- Frecuencia de Trabajo: 150KHz
- Ripple en la salida: 30mV (máx.) 20M bandwidth
- Protección de cortocircuito y sobre temperatura.
- Dimensiones: 43 x 21 x 18 mm.

## 40.6 BATERIAS 18650

Las mejores baterías 18650 del mercado (marcas como Panasonic, LG, Samsung o Sony/Murata) tienen una capacidad máxima real de entre 3.400 mAh y 3.600 mAh.

No existe actualmente en el mundo una batería 18650 real que supere los 3.600 mAh.

Una batería que dice tener 8.800 mAh (casi el triple del límite físico) es una etiqueta mentirosa pegada sobre una batería de muy mala calidad.

Cuando compras esas baterías (que suelen ser muy baratas y livianas, a veces con marcas como "Ultrafire" o nombres genéricos dorados/rojos).

- Capacidad Real: Suelen tener entre 300 mAh y 800 mAh. Es decir, rinden 10 veces menos de lo que prometen.
- Peso: Si las pesás, notarás que son muy livianas. Una 18650 real pesa entre 44 y 48 gramos. Las falsas suelen pesar menos de 30 gramos porque a veces tienen arena o simplemente una celda diminuta adentro.

Las mejores marcas en baterías son:

- NCR18650B (Panasonic/Sanyo - Verde)
- Samsung 30Q (Rosada)
- LG HG2 (Marrón)
- Sony VTC6 (Verde)

Cualquiera de estas de 3000 mAh reales va a durar diez veces más que esa de "8.800 mAh" y va a funcionar perfecto con tu panel solar y el capacitor de 470 µF.

# 50. PANEL SOLAR

Un panel solar (o módulo fotovoltaico) es un dispositivo que aprovecha la energía de la luz solar para generar electricidad. Funciona mediante el efecto fotovoltaico: cuando los fotones (partículas de luz) impactan sobre las celdas de silicio del panel, liberan electrones, creando una corriente eléctrica continua (DC).

### Tipos de Paneles Solares

Existen principalmente tres tipos según la forma en que se fabrica el silicio:

- Monocristalinos (Los más eficientes)

  Se fabrican a partir de un solo cristal de silicio puro. Se reconocen porque sus celdas son de color negro o azul muy oscuro y tienen las esquinas recortadas (forma octogonal).

  Ventajas: Son los más eficientes (aprovechan mejor el espacio) y rinden mejor en condiciones de luz escasa o días nublados.

  Uso ideal: Si tenés poco espacio o necesitás que la batería se cargue lo más rápido posible.

- Policristalinos

  Se fabrican fundiendo varios fragmentos de silicio. Tienen un aspecto azul brillante jaspeado (se ven como cristales rotos en el interior).

  Ventajas: Son más económicos de fabricar.

  Desventajas: Tienen una eficiencia ligeramente menor y rinden un poco menos cuando hace mucho calor (la temperatura alta afecta su voltaje).

### Componentes de un Sistema Solar (Para tu caso)

Para que ese panel de 1W o 5W funcione con tu placa, el sistema se compone de:

- Celdas Solares: Las que captan la luz.
- Cubierta de Vidrio/Resina: Protege las celdas de la humedad y el granizo.
- Marco de Aluminio: (Solo en paneles grandes) Da rigidez.
- Caja de Conexión: Donde soldás los cables que van a tu TP4056.

### Tipos de paneles 6V

Un panel de 6V y 1W entrega, en condiciones ideales (sol de mediodía, despejado), unos 166mA (I = P / V).

- Pérdidas (33%) Entre el calor, la eficiencia del módulo TP4056 y que el sol no siempre está perfecto, contá con que vas a recibir unos 100mA a 120mA reales de carga.
- Tiempo de carga: Si tenés una batería 18650 estándar de 3000mAh, tardarías unas 25 a 30 horas de sol pleno en cargarla de 0 a 100%. Es decir, unos 3 o 4 días de buen sol.

| CAPACIDAD | CORRIENTE TEORICA | CORRIENTE REAL (67%) | HORAS CARGA | PRECIO |
|:--- |:---|:--- |:--- |:--- |
| 1 W | 166 mA| 110 mA | 24| 6.000 |
| 2,5 W | 410 mA | 279 mA | 10 | 14.000     |
| 3 W | 500 mA | 335 mA | 8 | 19.000     |
| 6 W |1.000 mA |  667 mA | 4,5| 30.970
| 10 W | 1.666 mA | 1.120 mA | 3 | 27.000 |

## 50.1 PANEL SOLAR - 6VDC - 2,5W

![PANEL SOLAR 2.5W](../assets/Panel_6V-2,5W.png)

### 50.1.1 Especificaciones
-	Voltaje: 6 VDC
-	Potencia: 2,5 W
-	Corriente de trabajo: 410 mA
-	Medidas: 213 x 92 x 3 mm
-	Peso: 

## 50.2 PANEL SOLAR - 6VDC 6W

Panel Solar Monocristalino Fotovoltaico de 6V y 6W
Una solución eficiente y compacta para tus necesidades energéticas. 
Con un tamaño de 170x200mm y un peso ligero de aproximadamente 115 g, este panel es ideal para proyectos de energía renovable en espacios reducidos. 
Su voltaje de circuito abierto de 7,2 V y una corriente de trabajo de hasta 1000 mA garantizan un rendimiento óptimo. 
Aunque no es de una marca destacada, su calidad está respaldada por la marca Candy-Ho, que ofrece productos nuevos y confiables. 
Este panel solar monocristalino es perfecto para aplicaciones que requieren una potencia máxima de 6 W, brindando una opción accesible y eficiente para quienes buscan aprovechar la energía solar.

![PANEL SOLAR 6W](../assets//Panel_6V-6W.png)

### 50.2.1	Especificaciones
-	Voltaje: 7,2 VDC
-	Potencia: 6 W
-	Corriente de trabajo: 1.000 mA
-	Medidas: 170x 220 mm
-	Peso: 115 g


## 50.3 CONEXIÓN A PANEL SOLAR

Para conectar este módulo a un panel solar, el concepto clave es que el TP4056 necesita una entrada estable de aproximadamente 5V para funcionar correctamente. Los paneles solares tienen un voltaje variable que depende de la intensidad del sol, por lo que no siempre es ideal conectarlos directamente.

### Requisitos del Panel Solar

- Voltaje: Lo ideal es un panel de 5V o 6V (voltaje nominal).
- Potencia: Un panel de 5W o 6W es perfecto, ya que entregará cerca de 1A bajo sol pleno, que es el límite de carga de este módulo.

### Esquema de Conexión (PIN OUT)

Tienes dos opciones principales para el "enganche" físico:

- Opción A (Directa): Soldar los cables del panel solar directamente a los Pads "+" y "-" que están a los costados del conector USB-C en la placa.
  - Positivo del panel al pad + (marcado como IN+ en algunos modelos).
  - Negativo del panel al pad - (marcado como IN-).
- Opción B (Recomendada para estabilidad): Usar un Regulador de Voltaje DC-DC (Step-Down) entre el panel y el módulo.

Si usas un panel de 12V, necesitas bajarlo a 5V fijos. El TP4056 soporta hasta 8V de entrada máximo, pero se calienta muchísimo por encima de los 5.5V y puede quemarse.

### Consideraciones Críticas

- Diodo de Bloqueo: Aunque el TP4056 suele prevenir el retorno de corriente, muchos instaladores agregan un diodo Schottky (como el 1N5817) en el cable positivo del panel para evitar que la batería se descargue a través del panel solar durante la noche.
- Capacitor de Estabilidad: Si el sol parpadea (nubes), el módulo puede entrar y salir del estado de carga constantemente. Soldar un capacitor electrolítico (ej. 1000uF 16V) en los terminales de entrada (+ y -) ayuda a suavizar esas fluctuaciones.
- Calor: En días de mucho sol en zonas como Mendoza o el interior de Buenos Aires, el chip puede levantar mucha temperatura. Si vas a meterlo en una caja estanca para un proyecto de AgTech, te recomiendo pegarle un pequeño disipador de aluminio sobre el chip CSM4056T.

# 60. CABLES - Sección y Denominación

El término AWG son las siglas de American Wire Gauge (Calibre de Alambre Estadounidense). Es un sistema estandarizado que se usa desde 1857 para medir el diámetro de los cables eléctricos, principalmente los que no son de potencia pesada (como los de electrónica).

Lo más curioso (y lo que marea a mucha gente al principio) es que cuanto más grande es el número AWG, más fino es el cable. La razón es histórica y tiene que ver con el proceso de fabricación del cable (llamado trefilado):

- El cable se fabrica pasando una barra de metal gruesa por una serie de agujeros o "hileras" cada vez más pequeñas para estirarlo.
- El número AWG representa la cantidad de veces que el alambre tuvo que pasar por esos agujeros para llegar a ese grosor.
  - Un cable AWG 1 pasó una sola vez (es muy grueso).
  - Un cable AWG 20 pasó por 20 agujeros (es mucho más fino).

### ¿Cómo se compara con los milímetros (mm²)?

- El sistema AWG mide el diámetro del cable (mm). En Argentina y Europa, solemos usar la sección en milímetros cuadrados (mm<sup>2</sup>), que mide el área del corte transversal del cable.
- Aquí tenés una tabla rápida para que te ubiques con los cables que estuvimos mencionando:

| AWG| DIAMETRO (mm) | SECCION (mm<sup>2</sup>) | USO COMUN|
|:---|:---|:---|:---|
| AWG 14| 1.63| 2.08 | Instalaciones eléctricas de casas (enchufes)|
| AWG 16  | 1.29 | 1.31 | Cables electricos de 1.5mm<sup>2</sup> están por aca |
| AWG 22  | 0.64 | 0.32 | Audio, alarmas, proyectos con sensores.              |
| AWG 24  | 0.51 | 0.20  | Cables de red (UTP) y electrónica general           |
| AWG 28  | 0.32 | 0.08 | Cables Dupont (muy finitos)|

A mayor número AWG (cable más fino), menor es la sección y por lo tanto mayor es la resistencia.

- **Caída de voltaje:** Si usás un AWG 28 (muy fino) con tu panel solar, la resistencia del cable se va a "comer" parte del voltaje antes de que llegue al módulo TP4056. Por eso te decía que el cable de 1,5 mm<sup>2</sup> (cercano a AWG 16) es excelente: tiene un área enorme y casi no ofrece resistencia.
- En resumen: Usamos AWG porque es el lenguaje estándar de los componentes que compramos (como tu módulo o los sensores). Si un sensor dice que requiere AWG 24, ya sabés que es un cable finito pero no tanto como un pelo.
- El mayor punto débil de los Dupont no es solo el cable, sino el **conector de plástico**.
- Con el tiempo y la humedad (especialmente si el proyecto va afuera), los contactos se oxidan o se aflojan.
- Cualquier falso contacto en la línea de la batería o del panel puede hacer que el módulo deje de funcionar o que el ESP32 se reinicie aleatoriamente.

### Conectores "JST"
 "JST" no es el nombre de un conector, sino el nombre de la empresa (Japan Solderless Terminal). Ellos fabrican miles de series de conectores para casi cualquier industria imaginable, desde electrodomésticos hasta el sector automotriz.  
Sin embargo, para los entusiastas de la electrónica y el maker, solo un puñado son "estándar". Aquí tienes los más comunes clasificados por su paso (distancia entre pines):

|Serie JST |Paso (Pitch) |Uso típico
|:---|:---|:---
SH|1.00 mm|Ultra-compacto, cámaras de drones, wearables.
|ZH|1.50 mm|Electrónica de consumo muy pequeña.
|PH|2.00 mm|Dispositivos portátiles, robótica pequeña.
|XH|2.50 mm|Baterías LiPo, impresoras 3D, placas base.
|SYP|2.50 mm	|Alimentación (Poder)	Robusto, polarizado, traba interna
|RCY|2.50 mm| Alimentación de Radio Control
|SM|2.54 mm|Tiras LED, conexiones externas de sensores.
|VH|3.96 mm|Alta corriente (fuentes de poder).


### Diferencias clave: JST-SM vs JST-XH

|Característica|JST-SM|JST-XH
|:---|:---|:---
|Paso (pitch)|2.54 mm|2.50 mm
|Uso principal|Conexiones cable-a-cable (exterior)|Conexión cable-a-PCB (placa)||Bloqueo|Sí, tiene pestaña de seguridad (clic)|No, se mantiene por fricción
|Aplicación típica|Tiras LED, sensores externos|Baterías LiPo, motores de impresora 3D
|Robustez|Alta (protege contra tirones)|Baja (se desconecta fácilmente)

### ¿Por qué la confusión?

Aunque el paso es casi idéntico (2.54 mm vs 2.50 mm), no son compatibles entre sí. Si intentas forzar un JST-SM en un cabezal JST-XH, probablemente dañes los pines o el plástico de la carcasa.

### Resumen para tus proyectos:
- Usa JST-SM cuando necesites una conexión "volante" (cable con cable) que esté expuesta a movimiento o vibraciones; el mecanismo de bloqueo evitará que el circuito se abra accidentalmente.
- Usa JST-XH cuando necesites conectar un periférico (como un motor o un sensor) directamente a la placa controladora (PCB), aprovechando que son compactos y ahorran espacio en la placa.

Siempre debes preguntar el "paso" (pitch) o el nombre de la serie (PH, XH, SM, etc.).La regla de oro: El paso es lo más importante. Si el paso no coincide, el conector no entrará o, peor aún, harás un cortocircuito.

No son intercambiables: Aunque un JST-PH de 2.00 mm pueda entrar "a la fuerza" en algún lado, nunca intentes mezclar series.

En tus proyectos, tu "kit de supervivencia" debería cubrir básicamente los tres tamaños principales:
- 2.54mm (SM): Para conexiones externas que se mueven mucho.
- 2.50mm (XH): Para las conexiones de alimentación y balanceo estándar.
- 2.00mm (PH): Para cuando el espacio es realmente reducido y estás trabajando con componentes muy pequeños.

### ¿Por qué el SYP es un "caso especial"?
Aunque comparte el paso de 2.50 mm con el XH, el SYP no encaja en la misma categoría técnica por tres razones fundamentales:

- Su rol es eléctrico, no de datos: Mientras que el XH se usa mucho para señales de sensores o balanceo de celdas (donde la precisión de contacto es vital), el SYP está diseñado para aguantar el flujo constante de corriente de una batería a un dispositivo.

- Arquitectura de pines: El XH usa pines planos delgados. El SYP utiliza terminales internos más robustos diseñados para soportar ciclos de conexión más frecuentes sin perder la presión de contacto.

- Filosofía de diseño: * El XH está pensado para "nacer y quedarse" en la placa (o para procesos de mantenimiento poco frecuentes).

El SYP está diseñado para ser la "puerta de entrada" de energía del dispositivo; es decir, la pieza que el usuario va a conectar y desconectar constantemente.

### ¿Por qué se utiliza el RCY?
A diferencia del JST-SYP, que es más pequeño y estilizado, el RCY está diseñado para ser manipulado con dedos o incluso con guantes si estás en un campo de vuelo o pista de RC. Es un conector que soporta bien los tirones constantes y la vibración intensa, características normales en el mundo del control remoto.

¿Cuándo usarlo?

- No lo uses para sensores de precisión ni para conectar cosas a una placa Arduino (es demasiado grande y tosco para eso).

- Úsalo si estás reparando algún juguete RC, si necesitas un conector muy resistente para una alimentación de 12V en un proyecto de exterior, o si estás trabajando con baterías de LiPo de formato estándar RC.

### ¿Cuándo usar cuál?
Si estás conectando un sensor de temperatura a tu placa -> JST-XH (por su bajo perfil).

Si estás conectando una batería LiPo de 3.7V o 7.4V para alimentar tu proyecto -> JST-SYP (por su robustez y polaridad clara).

Si estás conectando tiras de LED direccionables que van de una caja a otra -> JST-SM (por su mecanismo de bloqueo mecánico que evita desconexiones por movimiento).

En resumen, el SYP es el conector que eliges cuando quieres que la energía llegue a tu proyecto de manera segura, clara (rojo/negro) y resistente al uso rudo.

## 60.1 CONECTOR DUPONT

Para que un conector Dupont (los cables tipo jumper que usas habitualmente con el ESP32 o placas de prueba) encaje correctamente y haga buen contacto, la altura del pin es clave.

La medida estándar

- La altura estándar de la parte metálica expuesta de un pin header (macho) debe ser de 6 mm.

- Longitud total del pin: Generalmente es de 11.5 mm a 12 mm.

Distribución:

- 6 mm hacia arriba (donde conecta el Dupont hembra).
- 2.5 mm de base de plástico (el aislante negro).
- 3 mm hacia abajo (la parte que se suelda a la placa o PCB).

### Detalles técnicos a tener en cuenta:

- El Paso (Pitch): Casi todos los conectores Dupont utilizan un paso de 2.54 mm (0.1 pulgadas). Si el pin es más corto de 6 mm, el "clic" interno del conector hembra no morderá con suficiente fuerza y podrías tener desconexiones intermitentes o caídas de tensión (ruido en los sensores).
- Pines Largos (Long Reach): Existen pines de hasta 15 mm o 20 mm de largo. Se usan mucho cuando necesitas atravesar una carcasa o cuando usas "stackable headers" (como los de las shields de Arduino), pero para un cable Dupont común, los 6 mm son la norma.
- Grosor del Pin: El pin debe ser cuadrado, de aproximadamente 0.64 mm de lado. Si intentas usar cables Dupont en pines redondos muy finos (como los de algunos componentes integrados), la conexión quedará floja.

Un consejo práctico: Si estás soldando los pines a una placa para un proyecto (como el de los sensores de pulso para el gas o el agua), asegúrate de que el pin quede perfectamente vertical. Si queda inclinado, el conector Dupont sufrirá estrés mecánico y terminará rompiendo el cable interno o falseando la lectura del sensor.

![CONECTOR DUPONT](../assets/Conector_DUPONT254.png)

## 60.2 FICHA JST-SM (2.54 mm)

Una ficha JST-SM es un tipo de conector eléctrico muy popular en el mundo de la electrónica, especialmente en proyectos de DIY, robótica, tiras LED y prototipado con microcontroladores (como los ESP32 que sueles utilizar).

Se caracteriza por ser un conector de paso de 2.54 mm (el mismo estándar que los protoboards y pines de cabecera), lo que los hace extremadamente compatibles y fáciles de usar.

![FICHA JST-SM](../assets/Ficha_JST_SM.png)

### 60.2.1 Características Principales
- Polarización: Tienen una forma física que impide que se conecten al revés. Poseen unas pestañas guía que aseguran una inserción correcta.

- Bloqueo de seguridad: A diferencia de los conectores tipo "Dupont" (que se sueltan fácilmente), los JST-SM incluyen un mecanismo de traba (latch). Una vez conectados, hacen un "clic" y requieren presionar una pestaña para soltarlos, evitando desconexiones accidentales por vibración o movimiento.

- Versatilidad: Vienen en configuraciones de 2 a 8 pines (aunque los de 2, 3 y 4 son los más comunes).

### 60.2.3 Uso común

- Tiras LED direccionables: Es el estándar industrial para las tiras tipo WS2812B/NeoPixel.

- Sensores: Ideales para sensores externos en tus proyectos de Home Assistant donde quieres asegurar que el cable no se suelte.

- Conexiones de alimentación: Soportan corrientes moderadas, lo que los hace perfectos para alimentar sensores y actuadores pequeños.

### 60.2.3 ¿Por qué son útiles para tus proyectos?
- Fiabilidad: Son mucho más robustos que los cables tipo jumper sueltos. Si estás integrando un sensor de nivel de agua o un sensor de flujo en tus proyectos de automatización, estos conectores garantizan una conexión eléctrica estable.

- Mantenibilidad: Si necesitas cambiar un sensor o realizar un mantenimiento en el panel de control, puedes desconectar el bloque completo sin tener que desarmar cable por cable.

- Montaje profesional: Permiten una gestión de cables mucho más limpia en cajas estancas o gabinetes eléctricos, algo fundamental cuando realizas instalaciones fijas en casa.

## 60.3 FICHA JST-XH (2.50 mm)
El JST-XH es un conector de paso de 2.50 mm. 
Es extremadamente común en el mundo del modelismo RC y la electrónica de baterías.
- Uso principal: Es el estándar mundial para el puerto de balanceo de las baterías LiPo (cargas de celdas individuales). También es muy frecuente en impresoras 3D para conectar motores paso a paso (nema 17) y finales de carrera.
- Diseño: Es un conector diseñado para ir "pegado" o montado directamente sobre un circuito impreso (PCB). 
Tiene una forma compacta y plana que facilita su conexión en placas densas.

- Fijación: No tiene un mecanismo de traba mecánico fuerte. Se mantiene unido por fricción (el ajuste preciso de sus pines).

- Pines: pueden venir de diferente cantidad de pines (como los conectores JST-SM)

![JSM-XH](../assets/Conector_JSM-XH.png)


## 60.5 CONECTOR JST-PH (2.00 mm)

El conector JST-PH es uno de los estándares más utilizados en el mundo de la electrónica, especialmente en proyectos de robótica, drones, baterías de litio pequeñas y prototipado con sensores (muy común en el ecosistema Arduino/ESP32).

Aquí tienes su descripción técnica y características principales:

- ¿Qué significa "PH"?
JST: Es el acrónimo de Japan Solderless Terminal, la empresa que inventó estos conectores.

- PH: Es la serie específica del conector.

- 2.0mm: Es el paso (pitch), que es la distancia exacta medida de centro a centro entre cada uno de los pines.

![JST-PH](../assets/Conector_JST-PH.png)

### 60.5.1 Especificaciones Técnicas Clave
- Paso (Pitch): 2.0 mm (no lo confundas con el JST-XH de 2.54 mm, que es un poco más grande).
- Voltaje nominal: 250V AC/DC.
- Corriente nominal: 2.0A AC/DC (usando cable AWG #24).
- Rango de temperatura: -25°C a +85°C.
- Tipo de contacto: Son conectores de tipo crimp (para insertar los cables usando una herramienta especial o soldarlos con cuidado).
- Mecánica: Cuentan con un mecanismo de bloqueo de fricción (friction lock), lo que significa que el conector hace un pequeño "clic" al entrar y se mantiene firme, evitando que se desconecte por vibración.

### 60.5.2 ¿Por qué es tan popular?
- Tamaño compacto: Es mucho más pequeño que los conectores tipo "header" estándar (2.54mm), lo que permite ahorrar mucho espacio en placas de circuito impreso (PCB).

- Fiabilidad: A pesar de ser pequeño, su conexión es muy segura y difícil de desconectar accidentalmente.

- Polarización: Tienen una forma que impide que se conecten al revés (a menos que fuerces la conexión), lo cual protege tus circuitos de cortocircuitos.

### 60.5.3 Uso típico en tus proyectos
- Baterías: Es el conector estándar para baterías LiPo de una celda (1S) pequeñas.
- Sensores: Muchos módulos sensores (como sensores de gas, temperatura o cámaras como la ESP32-CAM que mostraste en tu imagen) usan este formato para conectar los cables a la placa principal.
- Ventiladores: Es el estándar para los ventiladores pequeños de 5V o 12V en dispositivos electrónicos.


## 60.6 FICHA JST-SH (1.0 mm)
El conector JST-SH es el hermano "pequeño y avanzado" de la familia JST. Mientras que el PH es de 2.0 mm y el Dupont es de 2.54 mm, el SH es mucho más compacto y está diseñado para dispositivos de alta densidad y tecnología moderna.

![JST-SH](../assets/Conector_JST-SH.png)

Aquí tienes la descripción técnica:

### 60.6.1	Especificaciones Principales
-	Paso (Pitch): 1.0 mm. Es la mitad del tamaño del JST-PH.
-	Perfil: Es extremadamente bajo (muy "bajito" una vez conectado), ideal para dispositivos donde el espacio es crítico.
-	Tipo: Conector de tipo SMT (Surface Mount Technology). Esto significa que los zócalos están diseñados para soldarse directamente sobre la superficie de la placa (PCB), no atravesándola.
-	Mecanismo de seguridad: Posee un sistema de bloqueo positivo que asegura que el conector no se suelte accidentalmente, incluso ante vibraciones fuertes.

### 60.4.2	Diferencias visuales clave

Para no confundirte con otros conectores pequeños:

-	JST-PH (2.0mm): Es un poco más robusto y se siente "normal" al tacto. Es muy común en baterías de drones 1S.
-	JST-SH (1.0mm): Es notablemente diminuto. Si intentas meterle un cable JST-PH, no entrará ni de lejos. Se utiliza frecuentemente en periféricos de alta gama o en placas donde el espacio es tan reducido que no cabe nada más grande.

### 60.4.3 ¿Dónde lo encontrarás?
Drones y cámaras de alta gama: Muchos sistemas de transmisión de video (como los VTX de drones de carreras) usan SH para ahorrar peso y espacio.
Dispositivos portátiles: Relojes inteligentes, rastreadores GPS diminutos o sensores médicos portátiles.
Conectividad Qwiic / Stemma QT: Aunque muchas placas modernas usan conectores JST-SH de 4 pines para sus sistemas de conexión rápida (como los buses I2C), ten mucho cuidado: estos suelen ser una variante de JST-SH 1.0mm.

Esta serie es para casos de miniaturización. 

|Serie JST|Paso (Pitch)|Características
|:---|:---|:---
|SH|1.00 mm|El más pequeño, usado en drones y cámaras. Es la serie diseñada para ultraminiaturización
|GH|1.25 mm|Muy común en controladores de vuelo modernos (Pixhawk)
|ZH|1.50 mm|Intermedio, usado en impresoras 3D
|PH|2.00 mm|El estándar para baterías pequeñas y sensores


# 70. CONEXIONES

## 70.1 MODULO EXPANSION I2C - PCF 8574

MODELO: Pcf8574 Board Io I2c-bus

Esa tarjeta es un "caballito de batalla" cuando te quedás sin pines en el microcontrolador (como un ESP32 o un Arduino). Básicamente, es un expansor de entradas y salidas (I/O) que se comunica por el protocolo I2C.

Soporta conexión en cascada, permitiendo usar hasta 8 módulos para extender 64 puertos de E/S simultáneamente. La dirección del módulo se puede cambiar ajustando el interruptor de palanca.

Cada 2 segundos, el puerto P0-P7 alterna entre nivel alto y nivel bajo. Si se ponen 3 interruptores de palanca en ON, la dirección del módulo será 0x27.

Aquí te explico los usos más prácticos para tus proyectos:

![PCF 8574](../assets/Expansion_I2C_PCF8574.png)

### Manejo de Pantallas LCD (El uso más común)

- Si alguna vez viste un LCD 16x2 que solo tiene 4 cables en lugar de 16, es porque tiene una PCF8574 soldada atrás.
- Para qué sirve: Convierte la comunicación serie I2C en los 8 bits paralelos que necesita la pantalla para funcionar. Te ahorras muchísimos cables y pines.

### Ahorro de Pines (Pasa de 2 a 8 o más)

- Si estás usando un microcontrolador pequeño o tenés muchos sensores ocupando pines, con solo 2 cables (SDA y SCL) podés controlar:
- 8 Salidas: Para prender LEDs, activar relés o transistores.
- 8 Entradas: Para leer pulsadores, finales de carrera o interruptores.
- Dato clave: Podés conectar hasta 8 módulos PCF8574 en el mismo bus I2C cambiando las direcciones con los puentes (jumpers) A0, A1 y A2. ¡Eso te da hasta 64 pines extra usando solo los mismos 2 cables de comunicación!

### Teclados Matriciales

- En lugar de usar 8 pines para un teclado 4x4, conectás el teclado a la PCF8574 y el microcontrolador solo "pregunta" por I2C qué tecla se apretó. Es ideal para paneles de control donde el teclado está lejos del procesador principal.

### Detalles técnicos que te van a servir

- Corriente de salida (Source vs Sink): Tené cuidado porque la PCF8574 es muy "floja" entregando corriente (High), pero muy buena recibiéndola (Low).
- Mal: Conectar el ánodo de un LED al pin de la PCF y el cátodo a tierra (brillará muy poco).
- Bien: Conectar el ánodo a VCC y el cátodo al pin de la PCF (el LED se prende cuando el pin está en \`LOW\`).
- Pin de Interrupción (INT): Tiene un pin de interrupción que avisa al micro cuando alguna entrada cambió de estado. Así no tenés que estar preguntando todo el tiempo ("polling"), lo que hace tu código mucho más eficiente.

### Cuándo te conviene usarla

Si estás armando un tablero (como el que planeabas con las borneras) y necesitás muchos indicadores LED o botones de estado, la PCF8574 es la solución para no cablear todo directamente al microcontrolador y evitar una maraña de cables.

## 70.2 OFFSET, RESISTENCIA DE CARGA y PULL UP

En electrónica y automatización, entender estos tres conceptos es fundamental para asegurar que las lecturas de tus sensores sean precisas y que tus circuitos funcionen sin quemarse o perder señal.

### 70.2.1 Offset

El offset (o desplazamiento) es una diferencia constante entre el valor real de una variable y el valor que mide un sensor. Es un "error sistemático" que desplaza toda la curva de medición hacia arriba o hacia abajo.

En la práctica: Si un sensor de temperatura marca 22°C cuando la temperatura real es 20°C, tienes un offset de +2°C.

En señales eléctricas: Se refiere a un voltaje de CC (corriente continua) que se suma a una señal de CA (corriente alterna). Si una onda senoidal oscila entre -1V y 1V, pero tiene un offset de 2V, pasará a oscilar entre 1V y 3V.

### Corrección del Offset

Normalmente se soluciona mediante calibración por software. En herramientas como ESPHome o Home Assistant, simplemente aplicas un filtro lineal:

\`valor_corregido = valor_leido - offset\`

### 70.2.2 Resistencia de Carga (Load Resistor)

La resistencia de carga (R_L) es el componente o la resistencia total que "consume" la energía de un circuito de salida. No es un componente específico llamado así, sino el rol que cumple cualquier dispositivo conectado a una fuente.

En el contexto de sensores y microcontroladores, tiene dos usos principales:

- Conversión de Corriente a Voltaje (Lazo de 4-20 mA)

  Muchos sensores industriales entregan una señal en corriente. Como los pines analógicos del ESP32 o Arduino solo leen voltaje, necesitas una resistencia de carga para "traducir" esa señal.

  Si usas una resistencia de 250 ohm en un lazo de 20 mA, el voltaje será:

  V = 0.020 A 250 ohm = 5V

- Estabilidad de la Señal

  Sirve para asegurar que la salida de un dispositivo no quede "flotando". Por ejemplo, en sensores de colector abierto o protocolos como I2C, se usan resistencias de carga (pull-up) para mantener la línea en un estado lógico definido cuando no hay transmisión.

### Resumen de Diferencias

| CONCEPTO | DEFINICION SIMPLE | FUNCION PRINCIPAL |
|:--- |:--- |:--- |
| OFFSET | Error de desplazamiento constante | Calibrar el punto cro de una medición |
| RESISTENCIA DE CARGA | Impedancia que recibe la señal   | Convertir corriente a voltaje o estabilizar circuitos |

### Ejemplo Aplicado

Si estás armando un medidor de nivel de agua con un sensor de presión sumergible (4-20 mA):

- Usas una resistencia de carga para que el ESP32 pueda leer voltios en lugar de miliamperios.
- Si el tanque está vacío pero el sensor dice que hay "5 cm", configuras un offset de -5 en tu código para que la lectura real sea cero.

### 70.2.3 Resistencia Pull-Up / Pull-Down
Una resistencia pull-up (y su contraparte, la pull-down) es un componente fundamental en electrónica digital para evitar lo que llamamos un "estado flotante".

### El problema: El "Estado Flotante"
Los pines de entrada de un microcontrolador (como tu ESP32 o Arduino) son extremadamente sensibles. Si conectas un botón a un pin, pero el botón no está presionado, el pin queda "al aire" (desconectado).

En ese estado, el pin actúa como una antena que capta ruido electromagnético del ambiente. El microcontrolador no sabe si leer un 0 (LOW) o un 1 (HIGH) y empieza a cambiar de valor aleatoriamente. Esto se llama estado flotante y causa lecturas falsas constantes.

### La solución: Resistencia Pull-Up
La resistencia pull-up es simplemente un "puente" de resistencia que conecta el pin de entrada directamente al voltaje positivo (VCC, 3.3V o 5V).

Cuando el botón NO está presionado: La resistencia "tira" (pull-up) del pin hacia el voltaje positivo. El microcontrolador lee un 1 (HIGH) lógico.

Cuando el botón SÍ está presionado: El botón conecta el pin directamente a Tierra (GND). Como GND tiene más "fuerza" que la resistencia, el pin cae a 0V y el microcontrolador lee un 0 (LOW) lógico.

### ¿Por qué se usa una resistencia?
Si conectaras el pin directamente a 3.3V sin una resistencia, al presionar el botón ocurriría un cortocircuito (conectarías 3.3V directamente a Tierra). La resistencia sirve para limitar la corriente y proteger tanto tu placa como el pin, permitiendo que la señal llegue sin quemar nada.

### La gran ventaja: "Pull-up Interna"
Aquí está el truco que te ahorrará mucho trabajo en tus proyectos: Casi todos los microcontroladores (como el ESP32) tienen resistencias pull-up integradas en sus propios pines.

No necesitas comprar ni soldar resistencias externas.

En tu código (ESPHome o Arduino), solo tienes que activar esta opción en la configuración del pin.

```yaml
binary_sensor:
  - platform: gpio
    pin:
      number: GPIO2
      mode: INPUT_PULLUP  # <--- Aquí le dices al ESP32 que use su resistencia interna
    name: "Botón de cortina"
```
