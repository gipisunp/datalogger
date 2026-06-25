# **DATALOGGER GIPIS** 

> **BASADO EN EL MODULO HELTEC LORA V4 ESP32**


**CANALES HABILITADOS**
* ANALÓGICOS.
* RS232.
* RS485.
* I2C.


# CANALES ANALOGICOS 
Se manejan mediante el CI MCP3208. El mismo cuenta internamente con un multiplexor analógico y un ADC, los cuales se manejan mediante comandos SPI. 

# CANALES RS232
Se manejan mediante el MIKROE MUX CLICK, modulo el cual permite convertir 4 señales RS232 en TTL/3.3V mediante la multiplexación por hardware.  

# CANALES RS485
Se manejan mediante un módulo conversor RS485 a TTL/3.3V. No requiere multiplexación, sino una lógica de reconocimiento y consulta por ID específicos asignados a cada sensor.   

# CANAL I2C

Inclinómetro GY-511 LSM303DLHC. 