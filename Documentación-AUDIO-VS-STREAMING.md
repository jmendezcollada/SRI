<h1 align="center">AUDIO VS STREAMING</h1>

<p align="center"><img src=sri.png></p>



---

## ÍNDICE

 1. [Introducción](#introducción) 
 2. [Ejercicio 1: Radio Online (Audio)](#ejercicio-1-radio-online-audio) 
 3. [Ejercicio 2: Procesado y Streaming de Vídeo](#ejercicio-2-procesado-y-streaming-de-vídeo)
    
    3.1 [Objetivo del Ejercicio](#objetivo-del-ejercicio)
    
    3.2 [Instalación FFmpeg](#instalación-ffmpeg)
    
    3.3 [Descarga del Vídeo Original AULES](#descarga-del-vídeo-original-aules)
    
    3.4 [Ánalisis del Vídeo con ffprobe](#análisis-del-vídeo-con-ffprobe)
    
    3.5 [Remuxing: Cambio de Contenedor (.MP4 -> .MKV).](#remuxing-cambio-de-contenedor-mp4--mkv)
    
    3.6 [Cambio de Códecs y Comparación](#cambio-de-códecs-y-comparación)
    
    3.7 [Simulación de Streaming con Diferentes tipos de Fichero.](#simulación-de-streaming-con-diferentes-tipos-de-fichero)
 5. [Anexos](#anexos)
 6. [Comandos Interesantes](#comandosinteresantes)



---



### INTRODUCCIÓN

El consumo de contenidos multimedia a través de Internet se ha convertido en una parte esencial de la comunicación digital actual. Servicios como la **radio online**, **las plataformas de vídeo bajo demanda** o **las retransmisiones en directo** dependen de tecnologías de **streaming** capaces de transportar **audio y vídeo** de forma eficiente, estable y con la menor latencia posible.

En esta memoria se documentan 2 ejercicios del módulo de *Servicios en Red e Internet*:  
1. La creación de una *emisora de radio online* utilizando **Icecast2 y Mixxx.**  
2. El *análisis, procesado y simulación de streaming de vídeo* mediante **FFmpeg.**

El objetivo es comprender cómo funcionan los protocolos, códecs y mecanismos que permiten la transmisión de contenido multimedia, así como adquirir experiencia práctica configurando servidores, generando flujos de audio/vídeo y evaluando su calidad y consumo de recursos.  

La documentación se ha elaborado íntegramente en formato Markdown y alojada en GitHub, siguiendo una estructura clara y apoyándose en anexos donde se incluyen todas las capturas necesarias.


---
### EJERCICIO 1: Radio Online (Audio)





---
### EJERCICIO 2: Procesado y Streaming de Vídeo
#### Objetivo del Ejercicio

El objetivo de este ejercicio es trabajar con FFmpeg para comprender cómo se analiza, transforma y prepara un vídeo para diferentes escenarios. A lo largo del proceso se realizan tareas como:
1. Inspeccionar un archivo de vídeo con ffprobe.
2. Cambiar el contenedor sin recodificar (remuxing).
3. Recodificar el vídeo usando distintos códecs (H.264 y H.265).
4. Comparar calidad y tamaño de archivos a igual bitrate.
5. Generar perfiles de streaming con distintos bitrates y resoluciones.
6. Realizar cálculos de almacenamiento y capacidad de red.

Este ejercicio permite entender la relación entre códec, contenedor, bitrate, resolución, calidad y uso de recursos.

#### Instalación FFmpeg

 Este comando instala FFmpeg desde los repositorios oficiales del sistema. Incluye tanto ffmpeg como ffprobe.
 
```
sudo apt install ffmpeg
```

Comprobamos que FFmpeg está instalado con el siguiente comando:
```
ffmpeg -version
```
Este comando directamente no comprueba que se haya descargado FFmpeg, sino que comprueba la version en que tenemos de ese paquete e indirectamente comprueba que de verdad existe.

<img width="872" height="485" alt="imagen" src="https://github.com/user-attachments/assets/2fd52559-49ab-4c29-870e-9181fa1b8eab" />

La salida muestra la versión instalada, los códecs soportados y las librerías disponibles. Esto confirma que la herramienta está lista para usarse.


#### Descarga del Vídeo Original AULES

El vídeo proporcionado en Aules es un archivo MP4 en resolución 1080p. Una vez descargado, lo colocamos en una carpeta de trabajo.

<img width="636" height="89" alt="imagen" src="https://github.com/user-attachments/assets/b1985a1d-935a-4a37-a6c5-c96752973b38" />

Comprobación del Archivo:

<img width="555" height="56" alt="imagen" src="https://github.com/user-attachments/assets/6d95cf51-4201-4081-8f27-5cbdae1d409d" />

Con este comando verificamos el tamaño del archivo y confirmamos que se descargó correctamente.


#### Ánalisis del Vídeo con ffprobe

Con el siguiente comando vamos a comprobar cierta información que nos será útil del vídeo:
```
ffprobe -v error -show_entries stream=codec_name,width,height,r_frame_rate,bit_rate,sample_rate,channels,duration -of default=noprint_wrappers=1 big-buck-bunny-1080p-30sec.mp4
```
- codec_name=h264
- width=1920
- height=1080
- r_frame_rate=24/1
- duration=30.000000
- bit_rate=5860720
- codec_name=aac
- sample_rate=48000
- channels=6
- r_frame_rate=0/0
- duration=30.000000
- bit_rate=192580

<img width="878" height="276" alt="image" src="https://github.com/user-attachments/assets/0fb28550-f3b8-493a-862b-57b1a86c2d04" />

Ffprobe es fundamental para entender cómo está construido un archivo multimedia antes de modificarlo.

#### Remuxing: Cambio de Contenedor (.MP4 -> .MKV).

Con el siguiente comando vamos a hacer un Remuxing, pero antes de nada, voy a explicar que es Remuxing:

El Remuxing consiste en cambiar el contenedor de un archivo de vídeo sin modificar el contenido interno.
Es decir:
- No se recodifica el vídeo
- No se recodifica el audio
- Solo se “empaqueta” todo dentro de otro contenedor
Piensa en ello como cambiar una caja sin tocar lo que hay dentro.

**Contenedor vs Códec**

- El contenedor es el archivo externo: .mp4, .mkv, .avi, .mov…
   Su función es guardar dentro los streams de vídeo, audio, subtítulos, metadatos, etc.
  
- El códec es la forma en que está comprimido el vídeo o el audio:
   H.264, H.265, AAC, Opus…
  
Cuando haces remuxing:
- El contenedor cambia (por ejemplo, de .mp4 a .mkv)
- Los códecs NO cambian (siguen siendo los mismos)

Ahora vamos a pasar con el comando para hacer Remuxing:
```
ffmpeg -i original.mp4 -c:v copy -c:a copy salida.mkv
```
En mi caso, he usado como entrada el fichero big-buck-bunny-1080p-30sec.mp4, renombrado a original.mp4 para seguir el enunciado.

<img width="1830" height="783" alt="image" src="https://github.com/user-attachments/assets/0dc9c5c8-8d1d-459a-b03e-3e02dbaf4934" />

Explicación del comando

- **-i original.mp4:** indica el archivo de entrada.
- **-c:v copy:** copia el stream de vídeo tal cual, sin recodificar.
- **-c:a copy:** copia el stream de audio tal cual, sin recodificar.
- **salida.mkv:** es el nuevo archivo, con contenedor MKV.
  
Aquí solo cambia el contenedor, no el contenido de vídeo ni de audio.

**Respuestas a las Preguntas**

> **a) ¿Ha cambiado el tamaño de forma significativa?**
>
> Podemos usar el comando: **ls -lh original.mp4 salida.mkv** para comprobarlo.
> ```
> jaime@jaime-VirtualBox:~/Descargas$ ls -lh original.mp4 salida.mkv
> -rw-rw-r-- 1 jaime jaime 22M feb  1 18:54 original.mp4
> -rw-rw-r-- 1 jaime jaime 22M feb  1 18:54 salida.mkv
> ```
> No, el tamaño no ha cambiado de forma significativa. El vídeo y el audio son los mismos, solo cambia el contenedor.

> **b) ¿Ha habido carga de CPU? ¿Ha tardado mucho?**
> Como no se recodifica nada (solo se copian los streams), FFmpeg:
> - Apenas usa CPU.
> - Termina el proceso muy rápido, casi instantáneo para un vídeo corto.
> No ha habido una carga de CPU apreciable y el proceso no ha tardado mucho;
> el remuxing es muy rápido porque no recodifica.


#### Cambio de Códecs y Comparación

En esta actividad se recodifica el mismo vídeo usando dos códecs distintos (H.264 y H.265) pero con el mismo bitrate. El objetivo es comparar:

- Calidad visual
- Artefactos en escenas con movimiento
- Tamaño final del archivo

**Generación del Archivo en H.264 (2 Mbps)**
Usaremos este comando para poder hacerlo:
```
ffmpeg -i original.mp4 -c:v libx264 -b:v 2M -c:a copy h264_2mbps.mp4
```
**Explicación**

- **libx264:** codificador para H.264.
- **-b:v 2M:** fija el bitrate del vídeo en 2 Mbps.
- **-c:a copy:** copia el audio sin recodificar.
- El archivo resultante se llama **h264_2mbps.mp4**.

H.264 es un códec muy extendido, compatible con prácticamente cualquier dispositivo.

**Generación del Archivo en H.265 (2 Mbps)**
Usaremos este comando para poder hacerlo:
```
ffmpeg -i original.mp4 -c:v libx265 -b:v 2M -c:a copy h265_2mbps.mp4
```
**Explicación**

- **libx265:** codificador para H.265 (HEVC).
- **-b:v 2M:** mismo bitrate que en H.264.
- **-c:a copy:** el audio se mantiene igual.
- El archivo resultante se llama **h265_2mbps.mp4.**
  
H.265 es un códec más moderno y eficiente: ofrece mejor calidad con menos bitrate

**Comparación Visual**

Para hacer la comparación visual correctamente necesitamos hacer lo siguiente:

- Abrimos ambos vídeos al mismo tiempo.
- Buscamos una escena con mucho movimiento.
- Pausamos en el mismo fotograma en ambos videos.

**Respuestas a las Preguntas**

> **1. ¿Cuál de los dos presenta más “artefactos” (cuadraditos)?**
> El vídeo codificado en H.264 es el que presenta más artefactos, especialmente en escenas con mucho movimiento.
> **Por qué ocurre**
>   - H.264 es un códec más antiguo y menos eficiente.
>   - A igual bitrate (2 Mbps), necesita “sacrificar” más información para mantener el tamaño.
>   - Esto provoca bloques, pixelación y cuadraditos en zonas complejas.
> En cambio, **H.265** gestiona mejor el movimiento y la compresión, por lo que mantiene más detalle y
reduce los artefactos.

> **2. Si ambos tienen el mismo bitrate (2 Mbps), ¿pesan lo mismo los archivos finales.**
> No exactamente.
> Aunque ambos usan 2 Mbps, el tamaño final puede variar ligeramente.
> - El bitrate marca la media de datos por segundo, pero no es exacto al 100%.
> - Cada códec distribuye los bits de forma distinta.
> - H.265 suele ser más eficiente y a veces genera archivos ligeramente más pequeños, aunque ambos deberían estar muy cerca en tamaño.

#### Simulación de Streaming con Diferentes tipos de Fichero.

En esta actividad se generan dos perfiles de vídeo pensados para distintos tipos de conexión:

- Perfil Low (móvil) → baja resolución y bajo bitrate
- Perfil High (fibra) → alta resolución y bitrate más alto
  
Después se responden dos preguntas relacionadas con almacenamiento y capacidad de red.

**Perfil Low (móvil)**
- Resolución: 240p
- Bitrate: 400 kbps

Usaremos este comando:
```
ffmpeg -i original.mp4 -s 426x240 -b:v 400k -c:a aac low_240p.mp4
```
**Explicación**

- **-s 426x240** reduce la resolución a 240p.
- **-b:v 400k** fija el bitrate del vídeo en 400 kbps.
- **-c:a aac** mantiene un audio compatible y ligero.

Este perfil está pensado para conexiones lentas o dispositivos móviles

**Perfil High (fibra)**
- Resolución: 1080p
- Bitrate: 2 Mbps

Usaremos este comando:
```
ffmpeg -i original.mp4 -s 1920x1080 -b:v 2M -c:a aac high_1080p.mp4
```
**Explicación**

- **-s 1920x1080** mantiene la resolución Full HD.
- **-b:v 2M** fija el bitrate en 2 Mbps.

Este perfil está pensado para usuarios con fibra o buena conexión

**Respuestas a las Preguntas**

> 1. Almacenamiento: Si tu servidor tiene un disco de 500 GB, ¿cuántas horas de vídeo del perfil HD (2 Mbps) podrías almacenar?
> **Cálculo:**
> 2 Mbps / 8 = 0.25 MB/s --> 0.25 MB/s × 3600 s = 900 MB/hora
>
> 500 × 1024 = 512000 MB --> - 512000 MB ÷ 900 MB/h = 568 horas
> **Resultado Final:** El servidor puede almacenar unas **568 horas de vídeo HD a 2 Mbps.**

> 2. Capacidad de Red: Tienes una línea de 100 Mbps simétricos. ¿Cuántos usuarios pueden ver el perfil Móvil (400 kbps) sin superar el 80% de la línea.
> **Cálculo**
> 100 Mbps × 0.8 = 80 Mbps disponibles
> 400 kbps = 0.4 Mbps
>                      --> 80 Mbps ÷ 0.4 Mbps = 200 usuarios
> **Resultado Final:** Pueden conectarse **200 usuarios simultáneamente usando el perfil móvil antes de saturar el 80% de la línea.**








---
### ANEXOS




---
### COMANDOS INTERESANTES

```
git add Documentación-AUDIO-VS-STREAMING.md
```

```
git commit -m "Estrutura Doc"
```

```
git push origin main
```
