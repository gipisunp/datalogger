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

  * Compartido: 5 pines.
  * Separado: 8 pines.

**Canal 485**
>Cantidad total: 2 pines. 

**Canal 232**
>Cantidad total: 4 pines. 

