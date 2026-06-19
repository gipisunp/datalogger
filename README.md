# DATALOGGER GIPIS 

> **BASADO EN EL MODULO HELTEC LORA V4 ESP32**


**CANALES HABILITADOS**
* ANALÓGICOS.
* RS232.
* RS485.
* I2C.


**CANALES ANALOGICOS** 
Se manejan mediante el CI MCP3208. El mismo cuenta internamente con un multiplexor analógico y un ADC, los cuales se manejan mediante comandos SPI. 

               DIAGRAMA DE BLOQUES INTERNO DEL MCP3208
               =======================================

      +-------------------------------------------------------------+

      |  MCP3208 (ADC de 12-bits con MUX de 8 canales)              |
      |                                                             |
 CH0 -|--> [ M ]                                                    |
 CH1 -|--> [ U ]                                                    |
 CH2 -|--> [ X ]                                                    |
 CH3 -|--> [   ] --(Selección)--> [ SUESTREO / ]                    |
 CH4 -|--> [ D ]                  [ RETENCIÓN  ]                    |
 CH5 -|--> [ E ] (Single/Diff)          |                           |
 CH6 -|--> [ 8 ]                        v                           |
 CH7 -|--> [CH ]                  [ COMPARTIMENTO ]                 |

      |                             [   DAC/SAR   ]                 |
      |                                 |                           |
      |  +------------------------------+                           |
      |  |                                                          |
      |  v                                                          |
      | [ LÓGICA DE CONTROL ] <====================> [ INTERFAZ ]---|--> DIN
      | [  Y REGISTROS ESC   ]                        [   SPI    ]---|--> DOUT
      |                                               [ INTERNA  ]---|--> CLK
      |                                                    ^        |---|--> CS/SHDN
      |                                                    |        |
      | VREF ----------------------------------------------+        |
      | VDD  -------------------------------------------------------|
      | DGND -------------------------------------------------------|
      | AGND -------------------------------------------------------|
      +-------------------------------------------------------------+




