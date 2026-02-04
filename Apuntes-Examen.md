# Apuntes Examen

## TEORIA

### Índice

- [Descarga directa vs Streaming](#descarga-directa-vs-streaming)
- [Topología de red](#topología-de-red)
- [Capa de transporte: tcp-vs-udp](#capa-de-transporte-tcp-vs-udp)
- [QoS: Jitter, Buffer](#qos-jitter-buffer)
- [Protocolos de streaming](#protocolos-de-streaming)
- [Códecs](#códecs)
- [Ejercicios Cálculo](#ejercicios-cálculo)
- [Fórmulas](#fórmulas)

---
## Descarga directa vs Streaming

#### Descarga directa
- El usuario demanda un fichero con un peso de **100 MB** y **10 minutos** de duración.
- Comienza la descarga.
- Se almacena en **buffer** y comienza la reproducción.
- El usuario termina la reproducción a los **2 minutos**.
- El servidor ha entregado los **100 MB**.

#### Streaming
- Datos enviados en **flujo constante**.
- No hay almacenamiento local permanente.
- Solo se consume el **ancho de banda que el cliente ha utilizado**  
  (2 minutos según el ejemplo anterior).

---
## Topología de red

#### Unicast
- Conexión **1 a 1** (estándar de internet).
- Mecánica: Si hay **100 oyentes**, el servidor abre **100 sockets TCP** y envía la información **100 veces**.
- Cálculo de ancho de banda:  
  `BW(tot) = BW(stream) × N(usuarios)`
- Desventaja: **Poco escalable**.

#### Multicast
- Mecánica: El servidor envía la información a una **dirección multicast**  
  (rango: `224.0.0.0 – 239.255.255.255`).
- Los **routers replican el paquete** solo si tienen suscriptores.
- Desventaja: Los **routers bloquean paquetes multicast**.  
  Solo viable en **redes internas**.

#### Broadcast
- El servidor envía datos a **todos los clientes** de la red.

<img width="607" height="325" alt="image" src="https://github.com/user-attachments/assets/2648b899-b534-476c-a33e-184f238f2858" />

---
## Capa de transporte: TCP vs UDP

#### TCP
- Representado como una persona **bebiendo agua directamente de la botella**.
- La imagen transmite:
  - Flujo **controlado**.
  - El agua llega **donde tiene que llegar**.
  - No se desperdicia.
  - La acción es **ordenada y fiable**.
- Interpretación directa del meme:
  - TCP entrega los datos **de forma precisa**, **en orden** y **sin pérdidas**.

---

#### UDP
- Representado como una persona intentando beber agua que **le cae desde arriba**.
- La imagen transmite:
  - Parte del agua **se pierde**.
  - El flujo es **rápido**, pero **descontrolado**.
  - No hay garantía de que el agua llegue a la boca.
- Interpretación directa del meme:
  - UDP envía datos **sin asegurar entrega**, **sin confirmar recepción** y **sin orden**.

---

#### Resumen visual (extraído del meme)
- **TCP = beber de la botella → control, fiabilidad, orden.**
- **UDP = agua cayendo desde arriba → rapidez, pérdidas, desorden.**

## QoS: Jitter, Buffer

#### Jitter (Fluctuación)
Es la variación en el tiempo de llegada de los paquetes.
- Ejemplo: El paquete 1 tarda 20ms, el paquete 2 tarda 150ms, el paquete 3 tarda 20ms.
- Si el Jitter es superior al tamaño del buffer, el audio se corta (**Buffer Underrun**).

#### Buffer (Amortiguador)
Es una memoria temporal en el cliente (y en el servidor).
- Función: Acumular suficientes segundos de audio para absorber el Jitter de la red.
- Efecto: A mayor buffer → Mayor estabilidad → Mayor latencia (retraso).

---

### QoS: Jitter, Buffer — Burst-on-Connect (Ráfaga de conexión)

Una característica específica de servidores como Icecast.
- Problema: Al conectarse, el oyente tardaría varios segundos en llenar su buffer a velocidad normal (1x).
- Solución (Burst): El servidor envía los datos iniciales (ej. 64KB) a la máxima velocidad posible que permita la red (ej. 10x), llenando el buffer del cliente casi instantáneamente para que el audio empiece a sonar de inmediato (**Time-to-first-byte reducido**).

## Protocolos de Streaming

#### 1. Capa de transporte: TCP vs UDP

#### TCP
- Si un paquete de audio/vídeo se pierde, **el cliente no lo reproduce**.
- El servidor lo **reenvía** (ACK/NACK).
- **Ventaja:** calidad y compatibilidad con firewalls, NAT y proxy (usa puertos estándar).
- **Desventaja:** alta latencia. La retransmisión introduce retraso.

#### UDP
- Se sacrifica calidad.
- **Latencia mínima**.

---

### 1. Capa de transporte: TCP vs UDP (continuación)

**¿Entonces todo audio/vídeo por UDP?**  
NO

---

### 2. Capa de aplicación (tres modelos)

1. **HTTP Legacy** (como usa Icecast2)
2. **HTTP Adaptativo**
3. **Real-time**

**HTTP… TCP ¿no?**  
(Sí, HTTP funciona sobre TCP.)

---

### HTTP Legacy (Icecast2)

- **Protocolo:** ICY  
- **Mecánica:** se abre conexión TCP y el servidor envía flujo continuo hasta que el cliente cierra.  
- **Puertos:** 80, 443, 8000 (Icecast2).  
- **Formato:** flujo continuo de bytes (MP3, Ogg, AAC).

---

### HTTP Adaptativo

- **Protocolos:** HLS (Apple), MPEG-DASH.
- **Mecánica:** el servidor **trocea el fichero** en chunks de 2–10 segundos.
- **Formatos:** `.ts`, `.m4s`.
- **Pro:** calidad adaptativa.  
  El servidor envía un **Manifest** que permite elegir chunks de distinta calidad.

---

### Real-Time

- **RTMP (Real-Time Messaging Protocol)**  
  - Funciona sobre TCP.  
  - Obsoleto para usuario final.  
  - Se usa para enviar vídeo al servidor (ej.: OBS → YouTube/Twitch).

- **RTSP (Real-Time Streaming Protocol)**  
  - Usado en cámaras de seguridad (CCTV) y domótica.  
  - Generalmente usa **UDP para datos** y **TCP para control**.

- **WebRTC**  
  - Videoconferencia.  
  - P2P, cifrado, UDP.  
  - Funciona en navegador sin plugins (Google Meet, Discord).

---

### Cuadro resumen

| Protocolo | Base | Latencia Típica | Uso Principal | Facilidad Firewall | Caché (CDN) |
|-----------|------|------------------|----------------|--------------------|-------------|
| Icecast (ICY) | TCP/HTTP | 10s – 30s | Radio online, audio simple | Muy fácil | Difícil |
| HLS / DASH | TCP/HTTP | 15s – 45s | Netflix, YouTube, TV online | Muy fácil | Excelente |
| RTMP | TCP | 2s – 5s | Ingesta (OBS → servidor) | Medio | No |
| WebRTC | UDP/TCP | < 0.5s | Zoom, Meet, Discord | Complejo | No |
| RTSP | UDP+TCP | < 1s | Cámaras IP, seguridad | Problemas NAT | No |

---

### Conclusión de la industria

- La industria usa **mucho más TCP que UDP**.
- Netflix, HBO, Disney+, etc. usan **HTTP adaptativo**.  
  Al ver una película, descargas **chunks** secuenciales vía TCP.
- Spotify y Apple Music también usan **TCP**.  
  Nunca pierdes fragmentos: si falla, se para, pero no suena “raro”.
- Twitch (lado del receptor): **TCP**, por eso hay delay.
- La radio online también es **TCP** (como Icecast2).
- Cuando se necesita **interacción en tiempo real**, el delay no es admisible → **UDP**.

## Códecs

Los códecs son algoritmos que permiten la **compresión y descompresión** de audio/vídeo.  
Su objetivo es **reducir la cantidad de información a transferir sin perder calidad**.

- Ejemplo:  
  - Canción de 3 minutos en calidad CD sin comprimir → ~32 MB  
  - La misma canción en MP3 (comprimida) → ~3 MB

#### Ejemplos de códecs de audio
- MP3  
- AAC  
- Vorbis  
- WAV

#### Ejemplos de códecs de vídeo
- H.264  
- H.265  
- AV1

---

### Frecuencia de muestreo

El audio es una onda analógica. Para digitalizarla, debe **muestrearse**, como si se tomaran “fotos” de la onda cada cierto tiempo.

- **Estándar:** 44.1 kHz  
- A mayor frecuencia de muestreo → más fidelidad a la onda original.

(La diapositiva muestra ejemplos visuales de muestreo con 10, 6 y 2 puntos.)

<img width="337" height="288" alt="image" src="https://github.com/user-attachments/assets/c3e20812-9624-4cbc-8de3-2d306715b04f" />


---

### Profundidad de bits (Bit Depth)

Si la frecuencia de muestreo eran las “fotos”, la profundidad de bits es la **calidad de cada foto**.

- Es la cantidad de bits transmitidos por segundo.  
- A mayor profundidad → mayor calidad.

**Estándar:** 16 bit (calidad CD)

La diapositiva muestra comparaciones entre 8-bit, 16-bit y señal analógica.

<img width="442" height="250" alt="image" src="https://github.com/user-attachments/assets/dbf15d0b-8dec-493a-9a1b-6fba31fba325" />

---

### Canales

Número de **audios independientes** que viajan en el mismo stream.

La diapositiva muestra un ejemplo de sistema **5.1** (varios canales independientes).

<img width="250" height="333" alt="image" src="https://github.com/user-attachments/assets/96c047ac-2b9e-4168-8a26-368d7c8d1b95" />

---

### Códecs con pérdida / sin pérdida

### Códecs con pérdida
- Reducen peso **eliminando información** que puede ser imperceptible para el oído humano.  
- Esa información **no se puede recuperar**.  
- Ejemplo: **MP3**

### Códecs sin pérdida
- Comprimen el fichero **sin eliminar información**, como un .zip.  
- Al descomprimir, el flujo de bits es **idéntico al original**.  
- El factor de compresión es menor que en los códecs con pérdida.  
- Ejemplos: **FLAC**, **WAV**

---
## Ejercicios Cálculo

### Cálculo de Peso Sonido

La fórmula general para calcular el peso de un archivo de audio es:

**Peso = Frecuencia × Bits × Canales × Segundos**

### Ejemplo (WAV sin compresión)
3 minutos de audio estéreo, 44.1 kHz, 16 bits:

P = 44100 × 16 × 2 × 3 × 60  
= 254016000 / 8  
= 31752000 bytes  
≈ **31.75 MB**

---

### Ejercicios

1. **Calcula el peso aproximado de un archivo de audio sin compresión (WAV) de 5 minutos, con una frecuencia de muestreo de 44,1 kHz, 16 bits y estéreo.**
```
  5 min = 300 seg
  44.100 Hz x 16 bits x 2 x 300 = 423.360.000 bits / 8 = 52.920.000 Bytes
  52.920.000 Bytes / 1.000.000 = **52,92 MB**
```
2. **Si emitimos un streaming en MP3 a un bitrate constante (CBR) de 128 kbps, ¿cuánto ancho de banda total consumirá el servidor si tiene 25 oyentes simultáneos?**
```
  128 kbps x 25 oyentes = 3200 Kbps / 1000 = **3,2 Mbps**
```
3. **Calcula el bitrate de un flujo de audio que utiliza una frecuencia de 48 kHz, 24 bits de profundidad y un solo canal (mono).**
```
  48.000 Hz x 24 bits x 1 = 1.152.000 b / 1000 = **1.152 kbps**
```
4. **Tienes un servidor con un límite de subida de 10 Mbps. ¿Cuántos oyentes a 192 kbps puede soportar teóricamente antes de saturar la red?**
```
  Limtie = 10 Mbps
  10 Mbps x 1000 = 10.000 Kbps / 192 Kbps = 52,08 - **52 oyentes**
```
---

### Cálculo de Peso Vídeo

### Fórmula general (vídeo sin comprimir)
**Peso = (Ancho × Alto) × Profundidad de color × FPS × Tiempo**

### Resoluciones comunes
- **1080p:** 1920 × 1080  
- **4K:** 4096 × 2160  
- **8K:** 7680 × 4320  

### Conceptos
- **Profundidad de color:** bits usados para definir el color de cada píxel (típico: 24 bits → 8+8+8).  
- **FPS:** fotogramas por segundo.

---

### Cálculo de peso (vídeo comprimido)

Los códecs eliminan la necesidad de enviar píxeles uno a uno.

### Nueva fórmula
**Peso = Bitrate × Tiempo**

El bitrate es la cantidad de información que puede enviarse por segundo.

---

### Tabla de bitrates recomendados

| Resolución | Calidad | Bitrate mínimo | Bitrate recomendado |
|-----------|---------|----------------|----------------------|
| 4K (2160p) | Ultra HD | 15 Mbps | 25–45 Mbps |
| 1080p (Full HD) | Alta | 4 Mbps | 6–9 Mbps |
| 720p (HD) | Media | 1.5 Mbps | 2.5–5 Mbps |
| 480p (SD) | Estándar | 500 kbps | 1 Mbps |
| 360p | Baja | 400 kbps | 700 kbps |

---

### Ejercicio 1: La pesadilla del almacenamiento

Un estudio de cine graba en RAW (sin comprimir) con una cámara 4K (3840×2160),  
a 60 fps y 30 bits de profundidad de color (HDR).

A. Calcula el bitrate en Gbps.  
B. ¿Cuánto ocuparán **10 segundos** de vídeo?  
C. Con un disco de **1 TB**, ¿cuántos minutos podrías guardar?

---

### Ejercicio 2

Quieres retransmitir una graduación por YouTube.  
Tienes **20 Mbps de subida** y quieres emitir en **1080p** con **H.264**.

A. Si configuras un bitrate de **6 Mbps**, ¿qué porcentaje de tu subida consumes?  
B. Si otros 3 alumnos emiten también a 6 Mbps, ¿qué ocurre?  
   (buffering, saturación, latencia…)  
C. ¿Qué solución técnica aplicarías para que los 4 puedan emitir sin saturar la línea?


## Fórmulas

### Audio
1. Peso de un archivo sin compresión (WAV)
```
Peso(bits) = Frecuencia_de_muestreo × Bits × Canales × Segundos
```
```
Peso(bytes) = Peso(bits) / 8
```

2. Bitrate de audio sin comprimir
```
Bitrate = Frecuencia × Bits × Canales
```
3. Consumo total de ancho de banda en streaming
```
BW_total = Bitrate_stream × Número_de_oyentes
```
### Vídeo
4. Peso de vídeo sin comprimir
```
Peso(bits) = (Ancho × Alto) × Profundidad × FPS × Tiempo
```
```
Peso(bytes) = Peso(bits) / 8
```

5. Peso de vídeo comprimido
```
Peso = Bitrate × Tiempo
```
### Red / Topología

6. Ancho de banda en unicast
```
BW_total = BW_stream × N_usuarios
```



**1. Ejercicio**

Si emites un streaming de audio a un bitrate constante (CBR) de 128 kbps y tienes 25 oyentes simultáneos en una red Unicast, ¿cuál es el ancho de banda total consumido?
🔍 Explicación Paso a Paso

    Identificar el tipo de red (Unicast): En el modelo Unicast, el servidor establece una conexión única para cada cliente. Esto significa que el ancho de banda se multiplica por el número de usuarios.

        Fórmula: A n c h o  de banda total = B i t r a t e × U s u a r i o s

    Realizar el cálculo en kbps: Multiplicamos el bitrate individual por la cantidad de oyentes: 128  kbps × 25  oyentes = 3.200  kbps

    Conversión de unidades (kbps a Mbps): Para obtener un resultado más legible (y que coincida con las opciones del examen), dividimos entre 1.000: 3.200 / 1.000 = 3 , 2  Mbps

✅ Respuesta Correcta

La respuesta correcta es la c. 3.2 Mbps.
💡 Análisis de las opciones:

    a. 25 Mbps: Incorrecto. No se multiplica el número de usuarios por 1 Mbps.
    b. 3200 Mbps: Incorrecto. El número es correcto, pero la unidad (Mega) es demasiado grande. Serían 3200 kbps.
    c. 3.2 Mbps: CORRECTO.
    d. 128 kbps: Incorrecto. Este sería el ancho de banda si solo hubiera 1 oyente o si la red fuera Multicast.

**2. Ejercicio**

Tienes un disco de 500 GB. ¿Cuántas horas de vídeo HD a 2 Mbps podrías alojar aproximadamente?
🔍 Resolución Paso a Paso

Para resolver esto, primero debemos saber cuánto "pesa" una hora de ese vídeo y luego ver cuántas veces cabe en el disco.
**1. Calcular cuánto ocupa 1 hora de vídeo (en Gigabytes)**

    Paso A (Tiempo a segundos): 1  hora = 3.600  segundos .
    Paso B (Peso en Megabits): 2  Mbps × 3.600  seg = 7.200  Megabits (Mb) .
    Paso C (Cruzar el puente de bits a Bytes): 7.200  Mb / 8 = 900  Megabytes (MB) .
    Paso D (Pasar a Gigabytes): 900  MB / 1.000 = 0 , 9  GB . (Por lo tanto, cada hora de vídeo ocupa 0,9 GB).

**2. Calcular cuántas horas caben en el disco**

Dividimos la capacidad total del disco entre lo que ocupa una hora: Total horas = 500  GB 0 , 9  GB/hora = 555 , 55  horas
✅ Respuesta Correcta

La respuesta más aproximada es la a. 555 horas.
💡 Por qué fallan las otras:

    b. 1000 horas: Es el error de no dividir entre 8 (500 / 0.5 Mbps si no se hiciera el cambio de bits a Bytes).
    c. 250 horas: No corresponde a la división correcta.
    d. 277 horas: Es un error común de cálculo si se divide mal el bitrate inicial.

**3. Ejercicio: Cálculo de Bitrate de Audio (LPCM)**

Enunciado: ¿Cuál es el bitrate de un flujo de audio que utiliza una frecuencia de muestreo de 48 kHz, una profundidad de 24 bits y un solo canal (mono)?

Paso a paso:

    Identificar la fórmula del Bitrate de audio: B i t r a t e = F r e c u e n c i a  de muestreo × P r o f u n d i d a d  de bit × Nº de canales .

    Sustituir con los datos del enunciado: 48.000  Hz × 24  bits × 1  canal = 1.152 .000  bits por segundo (bps) .

    Convertir a kilobits por segundo (kbps): Dividimos entre 1.000: 1.152 .000 / 1.000 = 1.152  kbps .

    Convertir a Megabits por segundo (Mbps): Dividimos entre 1.000 otra vez: 1.152 / 1.000 = 1 , 152  Mbps .

Respuesta correcta: b. 1.152 Mbps
**4. Ejercicio: Cálculo de almacenamiento para vídeo de alto bitrate**

Enunciado: Con un bitrate de 14,93 Gbps, ¿cuánto espacio de disco ocupará una toma de 10 segundos?

Paso a paso:

    Calcular el peso total en Gigabits (Gb): Multiplicamos el bitrate por el tiempo de la toma: 14 , 93  Gbps × 10  segundos = 149 , 3  Gigabits (Gb) .

    Cruzar el puente de bits a Bytes (Dividir entre 8): Para saber cuánto ocupa en el disco (Gigabytes), debemos dividir el total de bits entre 8: 149 , 3 / 8 = 18 , 6625  Gigabytes (GB) .

    Verificar la unidad resultante: El resultado directo de la operación anterior ya está en GB, que es una de las opciones de la lista.

Respuesta correcta: b. 18.66 GB
**5. Ejercicio: Cálculo de peso de audio WAV (Calidad CD)**

Enunciado: Calcula el peso aproximado de un archivo de audio WAV de 5 minutos, con 44.1 kHz, 16 bits y estéreo.

Paso a paso:

    Convertir el tiempo a segundos: 5  minutos × 60  segundos = 300  segundos .

    Calcular el Bitrate (bps): Multiplicamos frecuencia × profundidad × canales (estéreo = 2): 44.100  Hz × 16  bits × 2  canales = 1.411 .200  bits por segundo (bps) .

    Calcular el peso total en bits: 1.411 .200  bps × 300  segundos = 423.360 .000  bits .

    Pasar de bits a Bytes (Dividir entre 8): 423.360 .000 / 8 = 52.920 .000  Bytes (B) .

    Convertir a Megabytes (MB): Dividimos entre 1.000.000 (o dos veces entre 1.000): 52.920 .000 / 1.000 .000 = 52 , 92  MB .

Respuesta correcta: a. 50.47 MB (Nota: Es la opción más cercana; la diferencia decimal suele deberse a si el profesor usa 1024 o 1000 en el último paso).
**6. Ejercicio: Cálculo de Bitrate Vídeo 4K RAW**

Enunciado: Un estudio graba en RAW a 3840x2160, a 60 fps y 30 bits de color. ¿Cuál es el bitrate resultante en Gbps?

Paso a paso:

    Calcular el total de píxeles por cuadro: 3840 × 2160 = 8.294 .400  píxeles .

    Calcular los bits por segundo (bps): Multiplicamos: Píxeles × Profundidad de color × FPS. 8.294 .400 × 30  bits × 60  fps = 14.929 .920 .000  bps .

    Convertir a Gigabits por segundo (Gbps): Dividimos entre 1.000.000.000 (o tres veces entre 1.000): 14.929 .920 .000 / 1.000 .000 .000 = 14 , 93  Gbps .

Respuesta correcta: b. 14.93 Gbps
**7. Ejercicio: Capacidad de usuarios en red (Unicast)**

Enunciado: En una línea de 100 Mbps simétricos, ¿cuántos usuarios podrían ver un streaming de vídeo de 2 Mbps?

Paso a paso:

    Identificar el tipo de cálculo: Estamos calculando la capacidad de una red para servir flujos individuales (Unicast). Debemos dividir el ancho de banda total disponible entre lo que consume cada flujo.

    Verificar que las unidades coinciden:
        Ancho de banda total: 100  Mbps
        Bitrate del vídeo: 2  Mbps (Ambas están en Megabits por segundo, así que no hace falta convertir nada).

    Realizar la división: Usuarios = Ancho de banda total Bitrate por usuario 100 / 2 = 50  usuarios .

Respuesta correcta: a. 50 usuarios
**8. Ejercicio: Saturación de Ancho de Banda (Subida)**

Enunciado: Si 4 alumnos emiten a 6 Mbps cada uno en una línea de 20 Mbps de subida, ¿qué ocurrirá?

Paso a paso:

    Calcular la demanda total de la red: Cada alumno es un emisor independiente. Debemos sumar sus bitrates individuales: 4  alumnos × 6  Mbps/alumno = 24  Mbps .

    Comparar con la capacidad disponible:
        Demanda: 24  Mbps
        Capacidad de subida: 20  Mbps

    Analizar el resultado: Como la demanda ( 24 ) es mayor que la capacidad ( 20 ), la línea no puede transportar todos los datos en tiempo real. Esto genera un cuello de botella.

    Identificar la consecuencia técnica: Cuando los paquetes de datos no pueden salir a la velocidad necesaria, se produce latencia y buffering (parones), ya que el servidor no recibe la información completa a tiempo.

Respuesta correcta: a. La red se saturará (24 Mbps requeridos) provocando buffering.
**9. Ejercicio: Cálculo de porcentaje de uso de red**

Enunciado: Si tienes una conexión de 20 Mbps de subida y emites vídeo a 6 Mbps, ¿qué porcentaje de tu línea estás utilizando?

Paso a paso:

    Identificar los valores:
        Capacidad total (100%): 20  Mbps .
        Consumo actual: 6  Mbps .

    Aplicar la fórmula del porcentaje: Porcentaje = ( Uso actual Capacidad total ) × 100

    Realizar el cálculo: 6 20 = 0 , 3 0 , 3 × 100 = 30 .

Respuesta correcta: b. 30%
**10. Ejercicio: Límite de oyentes por ancho de banda**

Enunciado: Un servidor tiene un límite de subida de 10 Mbps. ¿Cuántos oyentes simultáneos puede soportar si cada uno consume 192 kbps?

Paso a paso:

    Igualar las unidades: Para poder dividir, ambos datos deben estar en la misma unidad. Pasamos los Mbps del servidor a kbps: 10  Mbps × 1.000 = 10.000  kbps .

    Aplicar la fórmula de capacidad (Unicast): Nº de oyentes = Ancho de banda total Bitrate por oyente

    Realizar el cálculo: 10.000  kbps / 192  kbps = 52 , 083. . .

    Interpretar el resultado: Como no puedes tener "un trozo" de oyente, redondeamos hacia abajo para no superar el límite de la línea. El servidor soporta un máximo de 52 oyentes.

Respuesta correcta: d. 52 oyentes
**11. Ejercicio: Cálculo de peso de película comprimida**

Enunciado: Calcula el peso de una película de 2 horas (120 min) comprimida a un bitrate de 1 Mbps (Calidad SD estándar).

Paso a paso:

    Convertir el tiempo a segundos: 120  minutos × 60  segundos = 7.200  segundos .

    Calcular el peso total en Megabits (Mb): Multiplicamos el bitrate por el tiempo: 1  Mbps × 7.200  segundos = 7.200  Megabits (Mb) .

    Cruzar el puente de bits a Bytes (Dividir entre 8): Para saber cuánto ocupa en disco (Megabytes), dividimos entre 8: 7.200  Mb / 8 = 900  Megabytes (MB) .

    Verificar unidades: El resultado es 900 MB. Si lo pasáramos a GB (dividiendo entre 1.000) sería 0,9 GB.

Respuesta correcta: b. 900 MB
**12. Ejercicio: Bitrate de Audio Estéreo de Alta Calidad**

Enunciado: Calcula el bitrate de un flujo de audio que utiliza una frecuencia de 48 kHz, 24 bits de profundidad y 2 canales (estéreo).

Paso a paso:

    Identificar los datos:
        Frecuencia: 48.000  Hz
        Profundidad: 24  bits
        Canales: 2 (al ser estéreo)

    Aplicar la fórmula de Bitrate: Bitrate = Frecuencia × Profundidad × Canales 48.000 × 24 × 2 = 2.304 .000  bits por segundo (bps)

    Convertir a Megabits por segundo (Mbps): Dividimos entre 1.000 para pasar a kbps y otra vez entre 1.000 para Mbps (o directamente entre 1.000.000): 2.304 .000 / 1.000 .000 = 2 , 304  Mbps

Respuesta correcta: c. 2.304 Mbps
**13. Ejercicio: Almacenamiento de vídeo 4K comprimido**

Enunciado: Si una película 4K tiene un bitrate de 45 Mbps, ¿cuánto espacio en disco ocuparán 2 horas de metraje?

Paso a paso:

    Convertir el tiempo a segundos: 2  horas = 120  minutos = 7.200  segundos .

    Calcular el total de Megabits (Mb): Multiplicamos el bitrate por la duración: 45  Mbps × 7.200  segundos = 324.000  Megabits (Mb) .

    Cruzar el puente de bits a Bytes (Dividir entre 8): Para saber el peso en disco (Megabytes), dividimos entre 8: 324.000 / 8 = 40.500  Megabytes (MB) .

    Convertir a Gigabytes (GB): Dividimos entre 1.000: 40.500 / 1.000 = 40 , 5  GB .

Respuesta correcta: c. 40,5 GB
**14. Ejercicio: Porcentaje de uso en fibra óptica**

Enunciado: Si emites un directo en 4K con un bitrate recomendado de 25 Mbps en una línea de fibra de 300 Mbps de subida, ¿qué porcentaje de la línea consumes?

Paso a paso:

    Identificar los valores de la red:
        Capacidad total (100%): 300  Mbps .
        Consumo del streaming: 25  Mbps .

    Aplicar la fórmula de porcentaje: Porcentaje = ( Consumo Capacidad ) × 100

    Realizar el cálculo:
        25 300 = 0 , 08333. . .
        0 , 08333 × 100 = 8 , 33 .

Respuesta correcta: a. 8,33%
**15. Ejercicio: Capacidad de almacenamiento de audio**

Enunciado: Si tienes un disco de 10 GB disponible, ¿cuántas horas de audio a 192 kbps puedes almacenar aproximadamente?

Paso a paso:

    Calcular cuánto ocupa 1 hora de este audio:
        Tiempo: 3.600  segundos .
        Peso en Megabits: 192  kbps × 3.600  s = 691.200  kilobits .
        Pasar a Megabits: 691.200 / 1.000 = 691 , 2  Mb .
        Pasar a Megabytes (Dividir entre 8): 691 , 2 / 8 = 86 , 4  MB .
        Pasar a Gigabytes: 86 , 4 / 1.000 = 0 , 0864  GB por hora.

    Calcular el total de horas en el disco: Dividimos la capacidad total entre lo que ocupa una hora: Horas = 10  GB 0 , 0864  GB/h = 115 , 74  horas

    Interpretar el resultado: El valor más cercano en las opciones es 115 horas.

Respuesta correcta: a. 115 horas
**16. Ejercicio: Cálculo de Déficit de Ancho de Banda**

Enunciado: Seis alumnos emiten simultáneamente vídeo a 4 Mbps cada uno. Si la línea de subida es de 20 Mbps, ¿cuál es el déficit de ancho de banda?

Paso a paso:

    Calcular el ancho de banda total necesario: Multiplicamos el número de emisores por su consumo individual: 6  alumnos × 4  Mbps/alumno = 24  Mbps requeridos .

    Identificar la capacidad disponible: La línea de subida es de 20 Mbps.

    Calcular la diferencia (déficit): Restamos lo que tenemos de lo que necesitamos: 24  Mbps (necesarios) − 20  Mbps (disponibles) = 4  Mbps .

Respuesta correcta: c. 4 Mbps (requiere 24 Mbps)
**17. Ejercicio: Peso de 1 minuto de audio "Calidad CD"**

Enunciado: ¿Cuál es el peso de 1 minuto de audio en "calidad CD" (44,1 kHz, 16 bits, estéreo) sin compresión?

Paso a paso:

    Convertir el tiempo a segundos: 1  minuto = 60  segundos .

    Calcular el Bitrate (bps): Multiplicamos frecuencia × profundidad × canales (estéreo = 2): 44.100  Hz × 16  bits × 2  canales = 1.411 .200  bits por segundo (bps) .

    Calcular el peso total en bits: 1.411 .200  bps × 60  segundos = 84.672 .000  bits .

    Convertir de bits a Bytes (Dividir entre 8): 84.672 .000 / 8 = 10.584 .000  Bytes (B) .

    Convertir a Megabytes (MB): Dividimos entre 1.000.000 (o dos veces entre 1.000): 10.584 .000 / 1.000 .000 = 10 , 58  MB .

Respuesta correcta: c. 10,58 MB
**18. Ejercicio: Tiempo de Descarga Directa**

Enunciado: Un usuario quiere escuchar un archivo de 100 MB. Si su conexión es de 10 Mbps, ¿cuánto tiempo tardará en completarse la "Descarga Directa"?

Paso a paso:

    Igualar las unidades (El paso clave): El archivo está en Bytes (MB) y la conexión en bits (Mbps). Debemos pasar el archivo a Megabits multiplicando por 8: 100  MB × 8 = 800  Megabits (Mb) .

    Aplicar la fórmula del tiempo: Tiempo = Tamaño del archivo (en bits) Velocidad de conexión (en bps)

    Realizar el cálculo: 800  Mb / 10  Mbps = 80  segundos .

Respuesta correcta: a. 80 segundos
**19. Ejercicio: Consumo total de un servidor de Streaming (Unicast)**

Enunciado: Si el servidor emite un stream de audio de alta calidad a 320 kbps y hay 50 oyentes conectados por Unicast, ¿qué ancho de banda total consume el servidor?

Paso a paso:

    Entender el concepto Unicast: En Unicast, el servidor debe enviar un flujo de datos independiente por cada usuario conectado. Por tanto, el consumo es acumulativo.

    Calcular el consumo total en kbps: Multiplicamos el bitrate por el número de oyentes: 320  kbps × 50  oyentes = 16.000  kbps .

    Convertir a Megabits por segundo (Mbps): Dividimos entre 1.000: 16.000 / 1.000 = 16  Mbps .

Respuesta correcta: a. 16 Mbps
**20. Ejercicio: Peso total de vídeo 720p**

Enunciado: Un vídeo de 15 minutos en resolución 720p tiene un bitrate de 4 Mbps. ¿Cuál es su peso total en GB?

Paso a paso:

    Convertir el tiempo a segundos: 15  minutos × 60  segundos = 900  segundos .

    Calcular el total en Megabits (Mb): Multiplicamos el bitrate por la duración: 4  Mbps × 900  segundos = 3.600  Megabits (Mb) .

    Cruzar el puente de bits a Bytes (Dividir entre 8): Para obtener el peso en Megabytes (MB): 3.600 / 8 = 450  Megabytes (MB) .

    Convertir a Gigabytes (GB): Dividimos entre 1.000: 450 / 1.000 = 0 , 45  GB .

Respuesta correcta: d. 0,45 GB
**21. Ejercicio: Capacidad de red local para 4K**

Enunciado: En una red de 1 Gbps (1000 Mbps), ¿cuántos usuarios pueden ver simultáneamente un streaming 4K configurado a 40 Mbps?

Paso a paso:

    Identificar la capacidad total: El enunciado ya nos facilita la conversión: 1  Gbps = 1.000  Mbps .

    Identificar el consumo por flujo (Unicast): Cada usuario que ve el streaming consume 40 Mbps.

    Realizar la división: Usuarios = Ancho de banda total Bitrate por usuario 1.000 / 40 = 25  usuarios .

Respuesta correcta: c. 25 usuarios
**22. Ejercicio: Eficiencia en Redes Multicast**

Enunciado: En una red Multicast, si enviamos un audio de 128 kbps a 100 oyentes, ¿cuál es el ancho de banda que sale del servidor?

Paso a paso:

    Entender el concepto Multicast: A diferencia del Unicast (donde el servidor envía una copia del archivo por cada oyente), en Multicast el servidor envía un único flujo de datos a la red. Son los nodos de la red (routers/switches) los que se encargan de replicar la señal solo donde sea necesario.

    Calcular el consumo del servidor: Dado que el servidor solo emite una vez la señal, el ancho de banda de salida es igual al bitrate de la fuente, independientemente de si hay 1, 100 o 1.000 oyentes.
        Bitrate de la fuente: 128  kbps .
        Consumo de salida: 128 kbps.

Respuesta correcta: b. 128 kbps
**23. Ejercicio: Capacidad máxima de oyentes (Unicast)**

Enunciado: En una red con 100 Mbps de subida, ¿cuántos oyentes simultáneos pueden recibir un flujo de audio de 256 kbps antes de saturar la línea?

Paso a paso:

    Igualar las unidades: Convertimos la capacidad de la red (Mbps) a la unidad del flujo de audio (kbps): 100  Mbps × 1.000 = 100.000  kbps .

    Aplicar la fórmula de capacidad: Dividimos el ancho de banda total entre el consumo por oyente: Nº de oyentes = 100.000  kbps 256  kbps

    Realizar el cálculo: 100.000 / 256 = 390 , 625 .

    Interpretar el resultado: Como no podemos tener una fracción de oyente, redondeamos al número entero inferior para no sobrepasar la capacidad física de la línea. El máximo es 390 oyentes.

Respuesta correcta: a. 390 oyentes
**24. Ejercicio: Cálculo de peso de vídeo RAW (1080p)**

Enunciado: ¿Cuánto pesa una toma de 5 segundos de vídeo 1080p (1920x1080) a 30 fps y 24 bits de color sin comprimir?

Paso a paso:

    Calcular los píxeles por cuadro: 1920 × 1080 = 2.073 .600  píxeles .

    Calcular el total de bits de la toma: Multiplicamos: Píxeles × Profundidad × FPS × Tiempo. 2.073 .600 × 24  bits × 30  fps × 5  segundos = 7.464 .960 .000  bits .

    Convertir de bits a Bytes (Dividir entre 8): 7.464 .960 .000 / 8 = 933.120 .000  Bytes (B) .

    Convertir a Megabytes (MB): Dividimos entre 1.000.000 (o dos veces entre 1.000): 933.120 .000 / 1.000 .000 = 933 , 12  MB .

Respuesta correcta: b. 933,12 MB
**25. Ejercicio: Reducción de peso por compresión (MP3)**

Enunciado: Una canción de 4 minutos pesa 42,33 MB en formato WAV. Si se comprime a MP3 a 128 kbps, ¿cuál será su nuevo peso aproximado?

Paso a paso:

    Ignorar el dato del WAV: El peso original (42,33 MB) es irrelevante para el cálculo final, ya que el peso de un archivo comprimido depende exclusivamente de su bitrate y su duración.

    Convertir el tiempo a segundos: 4  minutos × 60  segundos = 240  segundos .

    Calcular el peso en kilobits (kb): Multiplicamos el bitrate por el tiempo: 128  kbps × 240  segundos = 30.720  kilobits (kb) .

    Convertir a Megabits (Mb): 30.720 / 1.000 = 30 , 72  Megabits (Mb) .

    Convertir a Megabytes (MB) - (Dividir entre 8): 30 , 72 / 8 = 3 , 84  MB .

Respuesta correcta: d. 3,84 MB

**26. Ejercicio: Peso de Audio Mono de Alta Calidad (WAV)**

Enunciado: Calcula el peso de 10 minutos de audio sin compresión (WAV) a 48 kHz, 24 bits y un solo canal (mono).

Paso a paso:

    Convertir el tiempo a segundos: 10  minutos × 60  segundos = 600  segundos .

    Calcular el Bitrate (bps): Frecuencia × Profundidad × Canales
    48.000  Hz × 24  bits × 1  canal = 1.152 .000  bps (1,152 Mbps) .

    Calcular el peso total en bits: 1.152 .000  bps × 600  segundos = 691.200 .000  bits .

    Cruzar el puente de bits a Bytes (Dividir entre 8): 691.200 .000 / 8 = 86.400 .000  Bytes (B) .

    Convertir a Megabytes (MB): Dividimos entre 1.000.000:
    86.400 .000 / 1.000 .000 = 86 , 4  MB .

Respuesta correcta: a. 86,4 MB

**27. Ejercicio: Bitrate de Vídeo RAW (Baja Resolución)**

Enunciado: ¿Cuál es el bitrate de un vídeo de seguridad antiguo de 360p (640x360) a 15 fps y 24 bits de color sin comprimir?

Paso a paso:

    Calcular los píxeles por cuadro: 640 × 360 = 230.400  píxeles .

    Calcular el Bitrate en bits por segundo (bps): Multiplicamos: Píxeles × Profundidad × FPS. 230.400 × 24  bits × 15  fps = 82.944 .000  bps .

    Convertir a Megabits por segundo (Mbps): Dividimos entre 1.000.000 para obtener la unidad en Megas: 82.944 .000 / 1.000 .000 = 82 , 944  Mbps .

Respuesta correcta: a. 82,94 Mbps

**28. Ejercicio: Capacidad de almacenamiento masivo (TB)**

Enunciado: ¿Cuántas horas de vídeo de perfil "HD" (2 Mbps) se pueden guardar en un disco de 1 TB?

Paso a paso:

    Calcular cuánto ocupa 1 hora de este vídeo:
        Tiempo: 3.600  segundos .
        Consumo en Megabits: 2  Mbps × 3.600  s = 7.200  Megabits (Mb) .
        Convertir a Megabytes (Dividir entre 8): 7.200 / 8 = 900  MB .
        Convertir a Gigabytes: 900 / 1.000 = 0 , 9  GB por hora .

    Identificar la capacidad del disco en la misma unidad (GB):
        1  TB = 1.000  GB .

    Realizar la división final: Horas = Capacidad total (GB) Peso por hora (GB/h) 1.000 / 0 , 9 = 1.111 , 11  horas .

Respuesta correcta: d. 1.111 horas

**29. Ejercicio: Tamaño de un Podcast (MP3 CBR)**

Enunciado: ¿Cuánto espacio ocupará un podcast de 30 minutos comprimido a un bitrate constante (CBR) de 128 kbps?

Paso a paso:

    Convertir el tiempo a segundos: 30  minutos × 60  segundos = 1.800  segundos .

    Calcular el total en kilobits (kb): Multiplicamos el bitrate por la duración: 128  kbps × 1.800  segundos = 230.400  kilobits (kb) .

    Convertir de bits a Bytes (Dividir entre 8): Para obtener el peso en Kilobytes (KB): 230.400 / 8 = 28.800  Kilobytes (KB) .

    Convertir a Megabytes (MB): Dividimos entre 1.000: 28.800 / 1.000 = 28 , 8  MB .

Respuesta correcta: d. 28,8 MB

**30. Ejercicio: Bitrate de Vídeo 8K RAW (HDR)**

Enunciado: Calcula el bitrate en Gbps para un vídeo 8K (7680x4320) a 60 fps con una profundidad de color de 30 bits (HDR) sin compresión.

Paso a paso:

    Calcular los píxeles por cuadro: 7.680 × 4.320 = 33.177 .600  píxeles .

    Calcular el Bitrate en bits por segundo (bps): Multiplicamos: Píxeles × Profundidad × FPS. 33.177 .600 × 30  bits × 60  fps = 59.719 .680 .000  bps .

    Convertir a Gigabits por segundo (Gbps): Dividimos entre 1.000.000.000 (mil millones) para pasar de bits a Gigabits: 59.719 .680 .000 / 1.000 .000 .000 = 59 , 71  Gbps .

Respuesta correcta: b. 59,71 Gbps







