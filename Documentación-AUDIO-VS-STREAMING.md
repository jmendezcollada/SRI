<h1 align="center">AUDIO VS STREAMING</h1>

<p align="center"><img src=sri.png></p>



---

## ÍNDICE

 1. [Introducción](#introducción) 
 2. [Ejercicio 1: Radio Online (Audio)](#ejercicio-1-radio-online-audio) 
 3. [Ejercicio 2: Procesado y Streaming de Vídeo](#ejercicio-2-procesado-y-streaming-de-vídeo) 
 4. [Anexos](#anexos)
 5. [Comandos Interesantes](#comandosinteresantes)



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
codec_name=h264
width=1920
height=1080
r_frame_rate=24/1
duration=30.000000
bit_rate=5860720
codec_name=aac
sample_rate=48000
channels=6
r_frame_rate=0/0
duration=30.000000
bit_rate=192580

Ffprobe es fundamental para entender cómo está construido un archivo multimedia antes de modificarlo.

### Remuxing: Cambio de Contenedor (.MP4 -> .MKV).

Con el siguiente comando vamos a hacer un Remuxing, pero antes de nada, voy a explicar que es Remuxing:

El Remuxing consiste en cambiar el contenedor de un archivo de vídeo sin modificar el contenido interno.
Es decir:
- No se recodifica el vídeo
- No se recodifica el audio
- Solo se “empaqueta” todo dentro de otro contenedor
Piensa en ello como cambiar una caja sin tocar lo que hay dentro.

**Contenedor vs Códec (la clave para entenderlo)**

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

Explicación del comando

- -i original.mp4: indica el archivo de entrada.
- -c:v copy: copia el stream de vídeo tal cual, sin recodificar.
- -c:a copy: copia el stream de audio tal cual, sin recodificar.
- salida.mkv: es el nuevo archivo, con contenedor MKV.
  
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
