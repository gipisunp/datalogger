# Análisis de trama OCR-507 (S/N 0099) con Saleae Logic

## 1. Configuración del analizador Async Serial (Saleae)

La configuración usada en Saleae Logic decodifica la trama sin errores (el checksum cuadra perfectamente), lo que confirma que estos parámetros son correctos para este equipo:

| Parámetro | Valor | Justificación |
|---|---|---|
| Bit Rate (Bits/s) | **115200** | Es uno de los baud rates estándar aceptados por el OCR-507 (parámetro `telbaud` del manual SAT-DN-0027). El default de fábrica es 57600, pero este instrumento está configurado a 115200 — confirmado porque el header ASCII de la trama salió legible. |
| Bits per Frame | 8 Bits per Transfer (Standard) | UART estándar de 8 bits de datos. |
| Stop Bits | 1 Stop Bit (Standard) | UART estándar. |
| Parity Bit | No Parity Bit (Standard) | UART estándar, sin bit de paridad. |
| Significant Bit | Least Significant Bit Sent First (Standard) | Orden normal de envío en UART. |
| Signal Inversion | **Inverted** | Necesario porque se está midiendo la línea RS-232 directamente (niveles ±V), sin un conversor de niveles (level shifter). El comparador de entrada del Saleae interpreta el nivel "mark" (reposo) de RS-232 como bajo si no se invierte la señal; al activar "Inverted" se recupera la lógica UART correcta. |
| Mode | Normal | Sin modificaciones adicionales del modo de captura. |

El instrumento tiene dos interfaces físicas que transmiten la misma información:
- **RS-232** (bidireccional, pines Tx/Rx) — es la que se está capturando aquí.
- **RS-422** (solo transmisión, par diferencial TA/TB).

## 2. Formato de trama (Satlantic Data Format Standard)

El OCR-507 sigue el **Satlantic Data Format Standard**, documentado en el manual `SAT-DN-0027` (sección D, página D-5/D-6). Cada muestra genera una trama con esta estructura:

| Campo | Tamaño (bytes) | Tipo | Descripción |
|---|---|---|---|
| Instrument | 6 | ASCII (AS) | Identificador del instrumento, empieza con "SAT" |
| Serial Number | 4 | ASCII (AS/AI) | Número de serie del instrumento |
| Timer | 10 | ASCII (AF) | Segundos transcurridos desde el fin de la inicialización, con 2 decimales |
| Sample Delay | 2 | Binario con signo (BS) | Offset en ms para el momento exacto del muestreo |
| Channel (λ1..λn) | 4 cada uno | Binario sin signo (BU) | Cuentas A/D crudas de cada canal óptico (7 canales en el OCR-507) |
| Vin Sense | 2 | BU | Voltaje de entrada regulado (crudo) |
| Va Sense | 2 | BU | Voltaje del riel analógico (crudo) |
| Int. Temp. | 2 | BU | Temperatura interna (crudo) |
| Frame Counter | 1 | BU | Contador de tramas (0–255, cíclico) |
| Check Sum | 1 | BU | Checksum de integridad |
| Terminator | 2 | — | CR/LF (0x0D 0x0A) |

## 3. Trama capturada (bytes en hex)

```
53 41 54 44 52 37 30 30 39 39 30 30 30 30 30 32 34 2E 30 38
FF 7B 80 1E 79 C0 80 2A 5D 80 80 29 4E 80 80 48 B2 00 80 2E
CA C0 80 43 74 80 80 3A 96 C0 01 1C 00 B0 00 98 92 CE
```

(Faltan los 2 bytes finales del terminador `0D 0A`; probablemente se perdieron al copiar/pegar la captura.)

### Decodificación campo por campo

| Campo | Bytes (hex) | Valor decodificado |
|---|---|---|
| Instrument | `53 41 54 44 52 37` | `"SATDR7"` |
| Serial Number | `30 30 39 39` | `"0099"` |
| Timer | `30 30 30 30 30 32 34 2E 30 38` | `"0000024.08"` → 24.08 s |
| Sample Delay | `FF 7B` | -133 ms (entero con signo, big-endian) |
| Frame Counter | `92` | 146 |
| Checksum | `CE` | 0xCE — **válido** (ver verificación abajo) |

### Verificación del checksum

El algoritmo estándar Satlantic: el checksum es tal que la suma de **todos** los bytes de la trama (incluyéndolo) sea 0 módulo 256.

```
suma de todos los bytes excepto el checksum (mod 256) = 0x32 (50)
checksum esperado = (256 - 50) mod 256 = 206 = 0xCE
checksum real en la trama = 0xCE   ✅ coincide
```

Esto confirma que la alineación de bytes y la configuración del Saleae (115200, 8N1, LSB first, Inverted) son exactamente correctas — no hay desfase de bits ni bytes corridos.

## 4. Canales ópticos: de cuentas crudas a irradiancia

Cada canal de 4 bytes es en realidad una cuenta A/D de 24 bits con un offset fijo de aproximadamente `0x80000000` (2,147,483,648), lo cual explica que el byte más significativo de los 7 canales sea siempre `0x80`.

Se localizó el archivo de calibración del instrumento (S/N 0099) en `CD/Calibration files/DR7099A.cal`, que define para cada canal los coeficientes `A0`, `A1` e `Im` (inmersión) usados en la fórmula estándar de Satlantic:

```
Lu(λ) = Im · A1 · (counts − A0)
```

| λ (nm) | Counts crudos (32-bit) | A0 | A1 | Im | Lu (µW/cm²/nm/sr) |
|---|---|---|---|---|---|
| 412 | 2,149,480,896 | 2147737684.0 | 2.61633442813e-09 | 1.758 | 0.008018 |
| 442 | 2,150,260,096 | 2147732433.1 | 2.42994833287e-09 | 1.752 | 0.010761 |
| 490 | 2,150,190,720 | 2147394199.9 | 2.76385713737e-09 | 1.746 | 0.013495 |
| 510 | 2,152,247,808 | 2147848672.0 | 1.61473599056e-09 | 1.743 | 0.012381 |
| 554 | 2,150,550,208 | 2147545599.4 | 1.6890122413e-09  | 1.739 | 0.008825 |
| 669 | 2,151,904,384 | 2147405434.0 | 9.94959320015e-10 | 1.731 | 0.007748 |
| 683 | 2,151,323,328 | 2147786140.3 | 1.03760492606e-09 | 1.730 | 0.006349 |

## 5. Valores auxiliares (sin calibración disponible en el manual)

| Campo | Valor crudo |
|---|---|
| Vin Sense | 284 |
| Va Sense | 176 |
| Int. Temp. | 152 |

El manual no incluye las constantes de conversión genéricas para estos tres campos (voltaje de entrada, voltaje del riel analógico y temperatura interna); esas constantes suelen estar en el "Instrument File Standard" genérico de Satlantic, que no forma parte de este manual de operación.

## 6. Conclusión

- Los parámetros del Saleae (115200 bps, 8N1, LSB first, Inverted) son correctos y quedan validados por el checksum.
- La trama corresponde a un OCR-507 modelo 507-ICSA, serie 0099, con 7 canales ópticos.
- Las cuentas crudas de los 7 canales se convirtieron a irradiancia espectral usando la calibración específica de ese instrumento.
