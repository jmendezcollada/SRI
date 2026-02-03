## Apuntes Examen

### TEORIA

### Índice

- [Descarga directa vs Streaming](#descarga-directa-vs-streaming)
- [Topología de red](#topología-de-red)
- [Capa de transporte: tcp-vs-udp](#capa-de-transporte-tcp-vs-udp)
- [QoS: Jitter, Buffer](#qos-jitter-buffer)
- [Protocolos de streaming](#protocolos-de-streaming)
- [Icecast](#icecast)


### Descarga directa vs Streaming

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


### Topología de red

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

### Capa de transporte: TCP vs UDP

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

### QoS: Jitter, Buffer

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

### Protocolos de Streaming

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





















