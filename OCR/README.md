# FUNCIONAMIENTO SISTEMA OCR 

El OCR se encontrará como Nodo Sensor Inalámbrico a metros de la Boya Oceanográfica, en su flotador particular. Un cable de alimentación llevará los 12V de la boya hasta el OCR. 

El sistema contará en su nodo de flotación: el propio OCR + BIOSHUTTER + Módulo HELTEC + Banco de Relay + Inclinómetro + Fuente de Alimentación tipo Switching a 5V.  


## Energía vs Datos 

* HELTEC lee OCR e inclinómetro localmente, empaqueta datos y los envía por RF a la boya ppal. 

* La hoja de datos establece que al transmitir el modulo LoRa, este consume una potencia máxima de 28 dBm, generando picos de 750 mA. Es necesario agregar un capacitor de 470uF en paralelo a la alimentación del HELTEC para amortiguar este pico. 

* Cómo el módulo switching en el flotador transforma la tensión de la batería en 5V, la caída de tensión de las baterías que exista entre el cable que salga de la boya y el flotador es despreciable. 

* Como el Relay y el BIOSHUTTER se alimentan de los 12V, deben asegurarse al otro lado del cable de la boya con un cable AWG 18 o más grueso para ese tramo. Si la tensión del BIOSHUTTER cae a menos de 10VDC no abre. 


## CONSUMOS
* FUENTE SWITCHING
* OCR
* HELTEC
* BIOSHUTTER
* RELAY
* INCLINÓMETRO
* POTENCIA EN OPERACIÓN CONSTANTE
* TRANSITORIOS
* CONSUMO EN UN CICLO DE MUESTREO

### FUENTE SWITCHING
Eficiencia de conversión 92% máxima. Todos los consumos en 5V y 3.3V se ajustan a este valor ya que le van a pedir energía extra a la batería. 

El modulo conectado de manera continua a una bateria con su circuito encendido, con su salida inactiva, consume una corriente de 0.5 mA. 

### OCR
El OCR se alimenta a los 12V directos de la batería consumiendo 45mA de operación.

Tasa de muestreo entre 7Hz y 24Hz, tiempo para capturar una medición completa (lectura de sus 7 canales ópticos y canales auxiliares) y enviar la trama es de apenas 41 a 142 milisegundos por ciclo de medición.

El OCR no está listo para medir en el milisegundo exacto en que recibe energía. Según su manual oficial, una vez que se le aplica energía, el instrumento comienza una "ventana de operación de cuatro segundos llamada secuencia de inicialización

### HELTEC
Según la hoja oficial de datos del HELTEC V4, cuando el equipo está despierto recibiendo datos consume 75mA. Si además está transmitiendo por Bluetooth este consumo sube a 115mA. 

Para transmitir por LoRa a máxima potencia (los 28 dbm irradiados por la antena) el chip consume 750 mA a 5V. 

        P_HELTEC = 5 V * 75 mA / 0.92 = 407.61 mW

El módulo abrirá el relay mediante Lógica Invertida (con un 0, 0V), absorbiendo unos 2mA de la fuente switching durante esta operación. 

Durante el cierre del relay, HELTEC pone su pin GPIO en estado alto. Esto hace que el led del optoacoplaador se polarice en inversa, haciendo que la corriente total sea 0 mA. 

### BIOSHUTTER 
Una vez abierto el BIOSHUTTER entra en estado de reposo consumiendo 0.16W. 

        I_Shutter_ON = 0.16 W / 12 V = 13.33 mA 

### RELAY
Para mantener el BIOSHUTTER abierto, el relay debe permanecer energizado o "chupado". El consumo de su bobina es 0.45W .

        I_relay = 0.45 W / 12 V = 37.5 mA  

Agregar una **resistencia de pull-up** a la línea de 3.3V y el pin de control del relay. Esto asegura que si el uC se cuelga y se ponge en HI-Z, se fuerce a HIGH el relay. 

Si bien el relay consume 3 veces más que el BIOSHUTTER solo para mantenerlo abierto, este nos brinda Aislamiento Galvánico (optoacoplador) y simplicidad, todo en un mismo y único bloque. 

Si la cantidad de mediciones diaras es elevada, consumiendo de sobremanera el banco de baterías, la opción alternativa a un relay es utilizar un transitor MOSFET como llave o relay de estado sólido.  

### INCLINÓMETRO
Consumo de corriente normal de 0.11mA. Se conecta al pin de alimentación 3.3V del HELTEC. 

## POTENCIA EN OPERACIÓN CONSTANTE
Consumo total de operación constante (sin transitorios):

        * Heltec despierto y recibiendo datos.
        * OCR operativo. 
        * Relay activado. 
        * Bioshutter Abierto.
        * Inclinometro activo. 

        Potencia_op = 12V * ( 13.33 mA + 37.5 mA + 45 mA ) + 5V * ( 2 mA + 75 mA ) / 0.92 + 3.3V * 0.11 mA / 0.92 = 1.57 W 

## TRANSITORIOS 

**Iniciar HELTEC**
200ms de despierte . 50mA a 50V 

        Consumo_Inicio = 5 V * 50 mA * 0.2 s / 3600 / 0.92 =  15 uWh 
        
        **DESPRECIABLE**

**APERTURA BIOSHUTTER**: 
Según hoja de datos, la apertura del BIOSHUTTER tarda 4 segundos y consume 1 mAh (que son exactamente 3.6 Coulombs de carga). 

        I_Shutter_pico_apertura = Q / t = 3.6 C / 4 s = 0.9 A 

        Potencia_apertura = 0.9 A * 12 V = 10.8 W

        Consumo_Apertura = 10.8 W * 4 seg * 1hora / 3600seg = 12 mWh 

Finalizado el sensado, se le corta la corriente al relay mientras que el BIOSHUTTER vuelve a su posición de reposo por un sistema de resortes, consumiendo 0 mA en este proceso. 

**TX DATOS HELTEC**: 
Por Bluetooth eL consumo sube a 115mA. 
Para transmitir por LoRa a máxima potencia (los 28 dbm irradiados por la antena) el chip consume 750 mA a 5V. 

        P_HELTEC_BTH = 5V * 115 mA / 0.92 = 625 mW

        P_HELTEC_LoRA = 5V * 750 mA / 0.92 = 4.07 W

El tiempo de transmisición dependerá de la cantidad de datos enviados en la trama **PAYLOAD**, del **SPREADING FACTOR** y del **ANCHO DE BANDA**.  

* Un paquete pequeño en SF7/125 kHz tarda apenas unos 15 a 50 milisegundos.
* Un paquete en SF12 con el mismo ancho de banda puede tardar 1.5 segundos.


Dado que se va a transmitir los 7 canales del OCR, la clave es transmitir los datos en formato **BINARIO PUNTO FLOTANTE PRECISIÓN SIMPLE**, donde cada dato empaqueta en 4 bytes. 

        PAYLOAD_OCR = 7 canales * 4 bytes = 28 bytes. 

Para el inclinómetro, tenemos la aceleración en los 3 ejes y el magnetómetro en los 3 ejes. Para máxima precisión decodificada, cada uno de los 6 ejes se guarda en un entero de 2 bytes en **int16_t** . 

        PAYLOAD_INCL = 6 ejes * 2 bytes = 12 bytes. 

Teniendo en cuenta el protocolo para la trama completa LoRa, estimamos una trama de datos en:

        **PAYLOAD = 53 bytes** 

Si tomamos el caso típico de SF9 BW 125 kHz, Coding Rate estándar de 4/5, el tiempo real teorico son 246 ms. Para este PAYLOAD lo aproximamos a 300 milisegundos. 

## REPOSO
La fuente switching consume mientras todo está en modo sleep. 
        P_Reposo = 12V * 0.5 mA = 6 mW


 ## CONSUMO DE UN CICLO DE MUESTREO 
El OCR no puede medir mientras el BioShutter se está abriendo. Por lo tanto, el HELTEC y el OCR van a tener que estar encendidos, consumiendo sus 1.57 W, durante esos 4 segundos de espera en los que el OCR se prepara.

        Consumo_Total_Muestreo = Consumo_Apertura + Consumo_Medición + Consumo_Transmisión

                Consumo_Apertura = 12 mWh
                Consumo_Medición = 1.57 W * 4.142 s / 3600 = 1.81 mWh
                Consumo_Transmisión_LORA =  4.07 W * 0.300 s / 3600 = 0.34 mWh

        Consumo_Total_Muestreo = 12 mWh + 1.81 mWh + 0.34 mWh = 14.15 mWh 

Esta potencia se sumará cada vez que se realice un ciclo de muestreo. 

Si el ciclo cambiase, este consumo debe recalcularse. Por ejemplo apertura de BIOSHUTTER -> 15 segundos de medición -> transmisicon. 

        Consumo_Total_Muestreo = 12 mWh + 1.57 W * (15 +4) s / 3600 + 0.34 mWh = 20.62 mWh 


## CONSUMO EN UN DÍA DE MUESTREO 
Como el OCR solo mide de día, toda la noche hay consumo de reposo.


        t_uso = n_mediciones_diarias * 4,442 s

        Consumo_Total_Día_Muestreo = (24 hs - t_uso_horas) * P_Reposo + n_mediciones_diarias * Consumo_Total_Muestreo

        Consumo_Total_Día_Muestreo = (24 hs - t_uso) * 6 mW + n_mediciones_diarias * 12.40 mWh