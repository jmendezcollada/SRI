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

    Inspeccionar un archivo de vídeo con ffprobe.

    Cambiar el contenedor sin recodificar (remuxing).

    Recodificar el vídeo usando distintos códecs (H.264 y H.265).

    Comparar calidad y tamaño de archivos a igual bitrate.

    Generar perfiles de streaming con distintos bitrates y resoluciones.

    Realizar cálculos de almacenamiento y capacidad de red.

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
