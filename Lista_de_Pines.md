# **LISTA DE PINES**
Datalogger GIPIS utilizará un modulo Heltec Lora V4 basado en el microcontrolador (uC) ESP32-S3R2, el cual a su vez le incorpora, entre otras cosas, un chip Semtech SX1262 para comunicaciones LoRa.

## **DESCRIPCIÓN DE PINES HELTEC**

El Heltec cuenta con un total 40 puntos de conexión, 36 pines físicos expuestos en sus dos tiras laterales, y 4 puntos soldables (GPIOs 15, 16 ,17 y 18) en su lateral inferior a la pantalla OLED.

El total de GPIO asignables es de 32 (contando los puntos soldables), de los cuales 28 se encuentran en los laterales. Los 8 restantes son 1 pin de Reset (pin físico 7) y 7 pines de alimentación (x2) GND, (x2) 3.3V_IN, (x1) 5V_IN, (x2) 3.3V_OUT . 

Un total de 20 de esos 32 GPIO ya están comprometidos a alguna tarea, el cual podemos clasificarlos en 6 "*Intocables*", 12 "*Sacrificables*" y 2 en "*Zona Gris*".

Los pines *intocables* si se utilizan cómo GPIO se corre el riesgo de dañar la placa o perder el control operativo. Estos son:

>   * GPIO 7: Front-End del Amplificador de RF.
>   * GPIO 19 y 20: Línea de datos del USB.
>   * GPIO 43 y 44: UART 0.
>   * GPIO 46: Habilitador del AMP.RF. 

Los GPIO clasificados de manera "sacrificable" se utilizan como tal prescindiendo de la tarea que cumplen según como están físicamente routeados: 

>   * GPIO 0: Botón PRG.
>   * GPIO 17 y 18: SDA y SCL pantala OLED.
>   * GPIO 21: OLED RST.
>   * GPIO 34, 38, 39, 40, 41 y 42: Interfaz GNSS, routeados directamente al conector SH1.25-8P de la parte de atrás.
>   * GPIO 35: LED BLANCO Testigo (Este está bueno pq si se coloca por ejemplo la linea MOSI de la SD, cada vez que se guarde un dato en la SD se va a encender el led. U otro ejemplo, si se conecta al relay que acciona el BIOSHUTTER, cada vez que se abra se va a encender este led. En ultima instancia, puede usarse para cualquier cosa y que quede el led encendiendose sin sentido, aunque estaría bueno el darselo). 
>   * GPIO 36: VEXT.


Finalmente, los que están en una zona "gris", es decir usables pero con cuidado, mejor evitarlos.
>   * GPIO 1: Lectura de Batería.
>   * GPIO 37: Control Batería

> El Heltec trae un divisor resistivo soldado internamente para medir el voltaje de la batería LiPo, el GPIO 37 enciende ese circuito y el GPIO 1 lee el voltaje analógico. El problema de usarlos para otra cosa, como un bus de datos, es que las resistencias físicas de ese divisor siguen soldadas ahí, lo que introduce impedancia no deseada.
  
    
Dado que el uC nativo cuenta con tecnología **IOMUX**, la mayoría de los periféricos internos (SPI, I2C, UART, PWM) pueden enrutarse 
a cualquier GPIO libre.  

Algunos GPIO pueden utilizarse alternativamente como canales ADC, pulsador TOUCH, o reloj de salida. **Consultar pines intocables y grises antes de utilizarlos**. 

**EN RESUMEN** 
>    * 32 GPIOs
>         * 18 GPIO Completamente Libres (Algunas pueden alternativamente ser ADC, CLK, TOUCH..).
>         * 14 GPIO Condicionadas
>             * 6 GPIO Intocables
>             * 6 GPIO Sacrificables
>             * 2 GPIO en Zona Gris. 

## **PUERTOS A OCUPAR POR EL DATALOGGER**

Preliminarmente, se establece la necesidad de entre **17 y 21 pines GPIO** disponibles para las distintas funciones según la arquitectura y la disponibilidad de hardware, las cuales se detallan a continuación: 

### **Canal Analog - SPI**

Los Pines necesarios son:

> 1. MOSI: Envio de datos. Master-to-Slave. "TX del Heltec". 
> 2. MISO: Recepción de datos. Slave-to-Master. "RX del Heltec".
> 3. SCLK: Señal de reloj de 1MHz. 
>   * El estándar de las tarjetas SD exige que al encenderse en modo SPI, siempre deban inicializarse a una velocidad menor , entre 100 kHz y 400 kHz, para garantizar estabilidad antes de subir la velocidad.
>   * El MCP3208 tiene como valores máximos 2MHz a 5V de alimentación, 1MHz a 2.7V de alimentación. Requiere 20 ciclos de reloj para completar un ciclo de muestreo. Es decir a 5V, la frecuencia máxima de muestreo será de 100 ksps (kilo samples por segundo).
> 4. CS_MCP3208: Enciende el MCP.
> 5. CS_SD: Enciende la SD. 

Según la estrategia de diseño, la tarjeta SD y los canales analógicos (gobernados por el chip MCP3208) pueden compartir o no el mismo bus
SPI. Esto trae distintas estrategias y particularidades de diseño, con sus respectivas ventajas y desventajas. 

* Compartido: **5 pines**.
>    * **Ventajas**: Menor consumo de pines. Menor número de pistas de alta velocidad ("antenas" en el PCB). 
>    * **Desventajas**: Gestión de Reloj, las distintas clases deberán reconfigurar el baudrate dinámicamente según si es MCP o SD. Riesgo de Solapamiento, algunas SD no liberan la linea MISO al desactivar su CS (por error de diseño, al conectar permanentemente el pin de habilitación de salida OE a GND, o por exigencia del estandar SD/MMC al trabajar en 3,3V sin módulos intermedios, el cual exige un pulso de reloj extra (byte 0xFF) para entrar en estado de alta impedancia y liberar la línea).

* Separado: **8 pines**.
>    * **Ventajas**: Aislamiento del MCP frente al ruido de línea de que puedan generar los picos de consumo de la SD. 
>    * **Desventajas**: Mayor consumo de pines. 

**NOTA**: La arquitectura antigua de EMAC realizaba esta mala práctica de re-instanciar el bus SPI completo en cada lectura analógica (read_adc()), lo que consume recursos y tiempo.

### **Canal 485**
    
El estandar RS485 maneja el bus de datos en un par de cables diferencial (A+ y B-). Todos los sensores se cuelgan en paralelo a este mismo bus y lo escuchan al mismo tiempo. Cada sensor se identifica y reconoce por su ID Header único, adoptado por el estándar de fabricante. 

Por ej: El ID de cabecera de trama para equipos que utilizan MODBUS RTU es un identificador de 1 byte, el cual puede asignarse a cada equipo por Dip-switches/selectores fisicos, pantalla-teclado integrados o por software del fabricante enviando un comando Modbus temporal al ID por defecto de fábrica (que suele ser el 1). Por otro lado, la trama de datos del OCR-507 de Satlantic se identifica por cadena ASCII "SAT", seguidas del tipo de sensor y luego del número de serie del equipo.  

Si se detecta un identificado de 1 byte -> MODBUS. 
Si se detecta SAT -> OCR y luego yendo a buscar el serial sabemos si es el de radiancia o irradiancia. 


Para poder integrar un sensor que maneja RS485 a un uC de 3,3V se requiere de un hardware conversor intermedio. El modulo conversor debe manejar el chip **MAX3485** (limitando la red a un máximo de 32 dispositivos, max 10Mbps), o bien el chip **MAX13487** (Posee una impedancia de entrada de 1/4 de unidad, lo que permite conectar hasta 128 dispositivos en el mismo bus, hasta 500kbps) o **MAX13488** (misma capacidad de dispositivos, velocidad de hasta 16Mbps).

El 3485 requiere nativamente de pines de control RE/DE adicionales, los cuales le permiten al master tomar o ceder el control del bus, mientras que los 1348x lo resuelven nativamente sin necesidad de estos pines. Sin embargo, algunos modulos basados en 3485 lo solucionan con hardware adicional externo, es decir que algunos modulos que SI ESTAN BASADOS EN MAX3485 pueden no requerir las lineas de pines adicionales, requiriendo solamente de las lineas de datos.  

En ultima instancia, estos pines RE/DE suelen puentearse externamente a un solo GPIO del uC, ya que uno es activo bajo y el otro activo alto. 

Dicho todo esto, los equipos RS485 se van a físicamente a la línea de datos A+/B- ingresantes al módulo conversor MAX, de este ultimo salen TX y RX convertidas y listas para conectar a la UART del uC Heltec. 

> 1. TX: Transmición de datos. Se conecta luego a RX de la UART del uC.
> 2. RX: Recepción de datos. Se conecta luego a TX de la UART del uC.
> 3. RE/DE: OPCIONAL. Línea de control de bus, si bien son dos pines físicos que salen del modulo conversor, gasta solo 1 pin ya que la práctica estándar y más habitual para trabajar en modo Half-Duplex es puentear los pines RE (Receiver Enable, activo bajo) y DE (Driver Enable, activo alto).

  * Con Módulo Reciente (sin RE/DE): **2 pines**.
  * Con Módulo Anterior (con RE/DE): **3 pines**.

**No se recomienda un modulo basado en el chip MAX485 ni el SN75176** ya que no están pensados para manejar tensiones 3,3V sino de 5V. 


### **Canal 232**
La UART asociada a los canales 232 se conecta al módulo MIKROE, el mismo está preparado para convertir señales RS232 en lógica TTL 3,3V y a su vez permite multiplexar 4 sensores, mediando la selección de A0 y A1. 

No puede implementarse más de 4 sensores 232 con esta arquitectura, ya que solo tenemos una única UART disponible para este fin, salvo de conseguir un MUX que sea de 8 canales en lugar del de 4 canales actual. Se recomienda colocar en esta UART-232 aquellos sensores que únicamente utilicen este protocolo, delegando el resto a canales analógicos o 485.

> * A0: Selección LSB.
> * A1: Selección MSB.
> * TX módulo: Se conecta al RX de la UART instanciada para canales 232.
> * RX módulo: Se conecta al TX de la UART instanciada para canales 232.

* UART-232: **4 pines**.

El datalogger deberá seleccionar el sensor que quiere consultar activando las lineas A1A0 correspondientes, esta lógica debería ser semejante a una maquina de estados que completa un ciclo de muestreo 232. Es decir, inicializando A1A0=00 y esperando que el sensor en el Canal 0 aporte su trama de datos, una vez recibida y decodificada esta trama se cambia A1A0=10 y se consulta el siguiente Canal, y así hasta completar el ciclo. La lógica de energización de sensores sincronizada con las A0A1 es crucial.  

Tabla de Verdad para la selección de canales (A1 es la MSB): 

**A1A0**

>00: Canal 0
>
>01: Canal 1
>
>10: Canal 2
>
>11: Canal 3

### **CANAL I2C**
En principio utilizaría los pines clasicos del I2C:

> SDA: Línea de Datos.
> 
> SCL: Reloj.

* I2C: **2 pines**.

Tener en cuenta que este protocolo utiliza resistencias de pull-up debidamente dimensionadas. La arquitectura y uso final de este Canal siguen pendientes de definición. 

Los buses I2C no están diseñados para equipos industriales/oceanográficos sino para perifericos de corto alcance. 

### **GPIOs PARA CONTROL DE ENERGÍA Y ACTUADORES**

Los relay se activan con lógica activa baja, es decir se pone a "0" el GPIO asignado para energízar BIOSHUTTER/BOMBA.

> ACTIVATE SENSORS: Despertar equipos para ahorrar energía de sensores, lógica gobernada por el uC semejante a lo ya implementado por EMAC utilizando un MOSFET IRF9540.
> 
> RELAY PARA ACCIONAR BIOSHUTTER.
> 
> RELAY PARA ACCIONAR BOMBA.
> 
> CONTROL DE BALIZA TIDELAND.

* ENERGÍA Y ACTUADORES: **4 pines**.
