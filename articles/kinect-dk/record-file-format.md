---
title: Uso del formato de archivo de grabación del SDK del sensor de Azure Kinect
description: Aprenda a usar el formato del archivo grabado del SDK del sensor de Azure Kinect.
author: xthexder
ms.author: jawirth
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: reference
keywords: kinect, azure, sensor, sdk, depth, rgb, record, playback, matroska, mkv
ms.openlocfilehash: 3a78ed158c78df1c9a6eadcc0bfb2e40b22750bd
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87501787"
---
# <a name="use-azure-kinect-sensor-sdk-to-record-file-format"></a>Uso del formato de archivo de grabación del SDK del sensor de Azure Kinect

Para grabar los datos del sensor, se usa el formato de contenedor Matroska (.mkv), que permite el almacenamiento de varias pistas mediante una amplia variedad de códecs. El archivo de grabación contiene pistas para el color, la profundidad, las imágenes IR y las IMU.

Los detalles de nivel bajo del formato de contenedor .mkv se pueden encontrar en el [sitio web de Matroska](https://www.matroska.org/index.html).

| Nombre de la pista | Formato del códec                          |
|------------|---------------------------------------|
| COLOR      | Dependiente del modo (MJPEG, NV12 o YUY2) |
| DEPTH      | b16g (escala de grises de 16 bits, big endian)   |
| IR         | b16g (escala de grises de 16 bits, big endian)   |
| IMU        | Estructura personalizada; consulte a continuación [Estructura de muestras de IMU](record-file-format.md#imu-sample-structure). |

## <a name="using-third-party-tools"></a>Uso de herramientas de terceros

Se pueden usar herramientas de terceros, como `ffmpeg` o el comando `mkvinfo` del kit de herramientas [MKVToolNix](https://mkvtoolnix.download/) para ver y extraer información de los archivos de grabación.

Por ejemplo, el siguiente comando extrae la pista de profundidad como una secuencia de PNG de 16 bits en la misma carpeta:

```
ffmpeg -i output.mkv -map 0:1 -vsync 0 depth%04d.png
```

El parámetro `-map 0:1` extrae el índice de seguimiento 1, que, para la mayoría de las grabaciones, será de profundidad. Si la grabación no contiene una pista de color, se usará `-map 0:0`.

El parámetro `-vsync 0` fuerza a ffmpeg a extraer fotogramas tal y como están en lugar de intentar igualar una velocidad de fotogramas de 30 fps, 15 fps o 5 fps.

## <a name="imu-sample-structure"></a>Estructura de muestras de IMU

Si los datos de IMU se extraen del archivo sin usar la API de reproducción, los datos estarán en formato binario.
La estructura de los datos de IMU es la siguiente: Todos los campos son little endian.

| Campo                        | Tipo     |
|------------------------------|----------|
| Marca de tiempo del acelerómetro (μs) | uint64   |
| Datos del acelerómetro (x, y, z) | float[3] |
| Marca de tiempo del giroscopio (μs)     | uint64   |
| Datos del giroscopio (x, y, z)     | float[3] |

## <a name="identifying-tracks"></a>Identificación de las pistas

Puede que sea necesario identificar qué pista contiene color, profundidad, IR, etc. La identificación de las pistas es necesaria cuando se trabaja con herramientas de terceros para leer un archivo Matroska.
Los números de seguimiento varían en función del modo de la cámara y del conjunto de pistas habilitadas. Se usan etiquetas para determinar el significado de cada pista.

En la lista siguiente, cada etiqueta está asociada a un elemento Matroska específico, y se puede usar para buscar la pista o los datos adjuntos correspondientes.

Estas etiquetas se pueden ver con herramientas como `ffmpeg` y `mkvinfo`.
La lista completa de etiquetas se muestra en la página [Grabación y reproducción](record-playback-api.md).

| Nombre de etiqueta             | Destino de etiqueta             | Valor de etiqueta             |
|----------------------|------------------------|-----------------------|
| K4A_COLOR_TRACK      | Pista de color            | UID de seguimiento de Matroska    |
| K4A_DEPTH_TRACK      | Pista de profundidad            | UID de seguimiento de Matroska    |
| K4A_IR_TRACK         | Pista de IR               | UID de seguimiento de Matroska    |
| K4A_IMU_TRACK        | Pista de IMU              | UID de seguimiento de Matroska    |
| K4A_CALIBRATION_FILE | Datos adjuntos de calibración | Nombre de archivo adjunto   |

## <a name="next-steps"></a>Pasos siguientes

[Grabación y reproducción](record-playback-api.md)
