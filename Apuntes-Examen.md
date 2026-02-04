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

---

### Profundidad de bits (Bit Depth)

Si la frecuencia de muestreo eran las “fotos”, la profundidad de bits es la **calidad de cada foto**.

- Es la cantidad de bits transmitidos por segundo.  
- A mayor profundidad → mayor calidad.

**Estándar:** 16 bit (calidad CD)

La diapositiva muestra comparaciones entre 8-bit, 16-bit y señal analógica.

---

### Canales

Número de **audios independientes** que viajan en el mismo stream.

La diapositiva muestra un ejemplo de sistema **5.1** (varios canales independientes).

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

2. **Si emitimos un streaming en MP3 a un bitrate constante (CBR) de 128 kbps, ¿cuánto ancho de banda total consumirá el servidor si tiene 25 oyentes simultáneos?**

3. **Calcula el bitrate de un flujo de audio que utiliza una frecuencia de 48 kHz, 24 bits de profundidad y un solo canal (mono).**

4. **Tienes un servidor con un límite de subida de 10 Mbps. ¿Cuántos oyentes a 192 kbps puede soportar teóricamente antes de saturar la red?**
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

Peso(bits) = Frecuencia_de_muestreo × Bits × Canales × Segundos
Peso(bytes) = Peso(bits) / 8

2. Bitrate de audio sin comprimir

Bitrate = Frecuencia × Bits × Canales

3. Consumo total de ancho de banda en streaming

BW_total = Bitrate_stream × Número_de_oyentes

### Vídeo
4. Peso de vídeo sin comprimir

Peso(bits) = (Ancho × Alto) × Profundidad × FPS × Tiempo
Peso(bytes) = Peso(bits) / 8

5. Peso de vídeo comprimido

Peso = Bitrate × Tiempo

### Red / Topología

6. Ancho de banda en unicast

BW_total = BW_stream × N_usuarios











