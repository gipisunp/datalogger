# **LISTA DE PINES**
>Datalogger GIPIS (HELTEC ESP32 LORA V4).

Dado que el ESP32-S3 cuenta con tecnología **IOMUX**, la mayoría de los periféricos internos (SPI, I2C, UART, PWM) pueden enrutarse 
a cualquier GPIO libre. Se solicita asignar los pines óptimos según el layout del PCB, respetando las restricciones de los pines ya 
utilizados internamente por la placa Heltec.

**Contexto Rápido**
>aaaa


Preliminarmente, se establece la necesidad de **15 pines GPIO** disponibles para las distintas funciones, las cuales se detallan a continuación: 

* Canal 485
>Cantidad total: 2 pines. 

* Canal 232
>Cantidad total: 4 pines. 

* Canal Analog - SPI
>Cantidad total: 5 pines. 
