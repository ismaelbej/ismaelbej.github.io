---
layout: post
title: Programando en QuickTime
author: Ismael
---

Como creando un achivo .mov con la API de QuickTime SDK.

QuickTime es el framework para manejo de video, sonido e imágenes
desarrollado por Apple para Mac. Tiene una versión disponible para
Windows, que es donde probe estos ejemplos.

Es un framework bastante sofisticado y potente. Conociendo algunas
pocas funciones es posible empezar a utilizarlo. Pero tambien permite
agregar funcionalidad más avanzada como códecs propios.

## Inicializando QuickTime

Primero hay que incluir los headers de QuickTime

```c++
#include <QTML.h> // sólo en Windows
#include <Movies.h>
#include <Components.h>
#include <QuickTimeComponents.h>
```

La versión para Windows usa un layer de emulación de la API nativa
de la Mac, cuyas declaraciones estan en el archivo QTML.h.

Antes de poder llamar a cualquier función de la libreria hay que
llamar a un par de funciones para inicializr la libreria QuickTime.
Si no lo hacemos las demás funciones van a fallar.

```c++
InitializeQTML(0); // sólo en Windows
EnterMovies();
```

Al terminar de usar las funciones del framework debemos llamar a las 
funciones que liberan los recursos reservados.

```c++
ExitMovies();
TerminateQTML(); // sólo en Windows
```

Las funciones `InitializeQTML` y `TerminateQTML` sólo existen en el
layer de emulación en Windows.

## Creando el archivo

Existen diferentes formas de como referenciar a un archivo en el
framework, aquí voy a usar la más moderna.

Primero creo una `CFStringRef` que contenga el nombre del archivo

```c++
WCHAR filename[] = L"f:\\qt-test.mov";
WCHAR filename[] = L"f:\\qt-test.mov";
CFStringRef filenameRef;
filenameRef = CFStringCreateWithCharacters(NULL, 
                                           (const UniChar*)filename,
                                           wcslen(filename));
```

El framework permite utilizar distintas fuentes de datos, archivos en 
disco, en memoria, ó en una URL. Para poder referenciar al contenedor
de una `Movie` necesitamos un `Handle` y un `OSType`. El `OSType`
indica que componente del sistema operativo, llamado _DataHandler_,
puede leer el contenedor, y el `Handle` tiene una referenci a los
datos que necesita el DataHandler para leer/escribir el archivo.

```c++
Handle dataRef;
OSType dataRefType;
QTNewDataReferenceFromFullPathCFString(filenameRef, 
                                       kQTNativeDefaultPathStyle, 
                                       0, 
                                       &dataRef, 
                                       &dataRefType);
```

Ahora podemos crear nuestra `Movie`.

```c++
Movie movie = NULL;
DataHandler dataHandler = NULL;
CreateMovieStorage(dataRef, 
                   dataRefType, 
                   FOUR_CHAR_CODE('TVOD'), 
                   smCurrentScript, 
                   createMovieFileDeleteCurFile | createMovieFileDontCreateResFile, 
                   &dataHandler, 
                   &movie);
```

## Agregando contenido

Un archivo .mov puede contener una o más _pistas_, dentro de cada pista
el contenido va en una ó varias _medias_. Para nuestro ejemplo
agregamos una pista de video con una sola media.

```c++
Track track = NewMovieTrack(movie, 
                            FixRatio(720, 1), // ancho, 0 para audio
                            FixRatio(576, 1), // alto, 0 para audio
                            kNoVolume);       // kFullVolume para audio
Media media = NewTrackMedia(track, 
                            VideoMediaType,   // SoundMediaType para audio
                            600,              // 1 segundo en la escala usada por la media
                            nil,
                            0);  
```

Para agregar a la media procedemos de la siguiente manera:

* Primero le indicamos al sistema que vamos a editar la media

```c++
BeginMediaEdits(media);
```

* Agregamos cada muestra de la media de la siguiente forma

```c++
AddMediaSample(media,      // media donde agregar la muestra
               data,       // Handle a la muestra
               0,          // offset a la muestra dentro del Handle
               size,       // tamaño de la muestra
               duration,   // duración en las unidades de la media
               sampleDescription,  // descripción de la muestra
               1,          // cantidad de muestras
               0,          // flags
               nil);       // devuelve el tiempo de la muestra
```

Finalmente terminamos de editar la media, insertamos la media
en el track correspondiente, actualizamos el archivo y finalmente
cerramos nuestra movie.

```c++
EndMediaEdits(media);
InsertMovieIntoTrack(track, 
                     0,    // inicio del track respecto del archivo
                     0,    // inicio de la media en el track
                     GetMediaDuration(media),  // duracion de la media
                     fixed1);  // velocidad de reproducción
UpdateMovieInStorage(movie, dataHandler);
CloseMovieInStorage(dataHandler);
``` 
