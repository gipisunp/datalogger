# **LISTA DE PINES**
>Datalogger GIPIS (HELTEC ESP32 LORA V4).

Dado que el ESP32-S3 cuenta con tecnología **IOMUX**, la mayoría de los periféricos internos (SPI, I2C, UART, PWM) pueden enrutarse 
a cualquier GPIO libre. Se solicita asignar los pines óptimos según el layout del PCB, respetando las restricciones de los pines ya 
utilizados internamente por la placa Heltec.

**Contexto Rápido**
>aaaa


Preliminarmente, se establece la necesidad de **15 pines GPIO** disponibles para las distintas funciones, las cuales se detallan a continuación: 

**Canal Analog - SPI**

Según la estrategia de diseño, la tarjeta SD y los canales analógicos, gobernados por el chip MCP3208, pueden compartir o no el mismo bus
SPI. Esto trae distintas estrategias y particularidades de diseño, con sus respectivas ventajas y desventajas. 

> 1. MOSI: Envio de datos. Master-to-Slave. "TX del Heltec". 
> 2. MISO: Recepción de datos. Slave-to-Master. "RX del Heltec".
> 3. SCLK: Señal de reloj de 1MHz. 
>   * El estándar de las tarjetas SD exige que al encenderse en modo SPI, siempre deban inicializarse a una velocidad menor , entre 100 kHz y 400 kHz, para garantizar estabilidad antes de subir la velocidad.
>   * El MCP3208 tiene como valores máximos 2MHz a 5V de alimentación, 1MHz a 2.7V de alimentación. Requiere 20 ciclos de reloj para completar un ciclo de muestreo. Es decir a 5V, la frecuencia máxima de muestreo será de 100 ksps (kilo samples por segundo).
> 4. CS_MCP3208: Enciende el MCP.
> 5. CS_SD: Enciende la SD. 

  * Compartido: **5 pines**.
     * **Ventajas**: Menor consumo de pines. Menor número de pistas de alta velocidad ("antenas" en el PCB). 
    * **Desventajas**: Gestión de Reloj, las distintas clases deberán reconfigurar el baudrate dinámicamente según si es MCP o SD. Riesgo de Solapamiento, algunas SD no liberan la linea MISO al desactivar su CS (por error de diseño, al conectar permanentemente el pin de habilitación de salida OE a GND, o por exigencia del estandar SD/MMC al trabajar en 3,3V sin módulos intermedios, el cual exige un pulso de reloj extra (byte 0xFF) para entrar en estado de alta impedancia y liberar la línea.
      * NOTA: La arquitectura antigua de EMAC realizaba esta mala práctica de re-instanciar el bus SPI completo en cada lectura analógica (read_adc()), lo que consume recursos y tiempo.
  * Separado: **8 pines**.
    * **Ventajas**: Aislamiento del MCP frente al ruido de línea de que puedan generar los picos de consumo de la SD. 
    * **Desventajas**: Mayor consumo de pines. 

**Canal 485**
    
El estandar RS485 maneja el bus de datos en un par de cables diferencial (A+ y B-). Todos los sensores se cuelgan en paralelo a este mismo bus y lo escuchan al mismo tiempo. Cada sensor se identifica y reconoce por su ID Header único, adoptado por el estándar de fabricante. 

Por ej: El ID de cabecera de trama para equipos que utilizan MODBUS RTU es un identificador de 1 byte, el cual puede asignarse a cada equipo por Dip-switches/selectores fisicos, pantalla-teclado integrados o por software del fabricante enviando un comando Modbus temporal al ID por defecto de fábrica (que suele ser el 1). Por otro lado, la trama de datos del OCR-507 de Satlantic se identifica por cadena ASCII "SAT", seguidas del tipo de sensor y luego del número de serie del equipo.  

Si se detecta un identificado de 1 byte -> MODBUS. 
Si se detecta SAT -> OCR y luego yendo a buscar el serial sabemos si es el de radiancia o irradiancia. 


Para poder integrar un sensor que maneja RS485 a un uC de 3,3V se requiere de un hardware conversor intermedio. El modulo conversor debe manejar el chip **MAX3485** (limitando la red a un máximo de 32 dispositivos, max 10Mbps), o bien el chip **MAX13487** (Posee una impedancia de entrada de 1/4 de unidad, lo que permite conectar hasta 128 dispositivos en el mismo bus, hasta 500kbps) o **MAX13488** (misma capacidad de dispositivos, velocidad de hasta 16Mbps).

El 3485 requiere nativamente de pinesde control RE/DE adicionales, los cuales le permiten al master tomar o ceder el control del bus, mientras que los 1348x lo resuelven nativamente sin necesidad de estos pines. Sin embargo, algunos modulos basados en 3485 lo solucionan con hardware adicional externo, es decir que algunos modulos que SI ESTAN BASADOS EN MAX3485 pueden no requerir las lineas de pines adicionales, requiriendo solamente de las lineas de datos.  

En ultima instancia, estos pines RE/DE suelen puentearse externamente a un solo GPIO del uC. Por ejemplo si el uC pone en alto su GPIO tanto RE como DE se ponene en alto y el datalogger toma control del bus, y viceversa para un bajo en el GPIO.  

Dicho todo esto, el bus de datos A+/B- con los equipos RS485 se conectar a la bornera de entrada del modulo conversor, del cual salen las lineas TX y RX listas para conectar al uC Heltec. 

> 1. TX: Transmición de datos. 
> 2. RX: Recepción de datos.
> 3. RE/DE: OPCIONAL. Línea de control de bus, si bien son dos pines físicos que salen del modulo conversor, gasta solo 1 pin pq están puenteados al uC.

  * Con Módulo Reciente: **2 pines**.
  * Con Módulo Anterior: **3 pines**.

**No se recomienda un modulo basado en el chip MAX485 ni el SN75176** ya que este no están pensados para manejar tensiones 3,3V sino de 5V. 


**Canal 232**
  * Compartido: **4 pines**.


