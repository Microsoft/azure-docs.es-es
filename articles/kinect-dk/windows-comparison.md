---
title: Comparación entre Azure Kinect DK y Kinect para Windows
description: Diferencias de hardware y software entre Azure Kinect DK y Kinect para Windows v2
author: tesych
ms.author: tesych
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: article
keywords: kinect, windows, v2, azure kinect, comparación, sdk, diferencias, hardware, software
ms.openlocfilehash: 0a8d399370f354524858bdd658ffd65c0494dd4d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87031582"
---
# <a name="azure-kinect-and-kinect-windows-v2-comparison"></a>Comparación entre Azure Kinect y Kinect para Windows v2

Los kits de desarrollo de software y hardware de Azure Kinect DK tienen diferencias con respecto a Kinect para Windows v2. Las aplicaciones existentes de Kinect para Windows v2 no funcionan directamente con Azure Kinect DK y exigen la portabilidad al nuevo SDK.  

## <a name="hardware"></a>Hardware

En la tabla siguiente se indican las diferencias generales entre el kit de desarrollo de Azure Kinect y Kinect para Windows v2.

| Característica | Tipo | Azure Kinect DK | Kinect para Windows v2 |
| ------- | ---- | --------------- | --------------------- |
| **Audio** | Detalles  | Matriz circular de 7 micrófonos | Matriz por fases lineal de 4 micrófonos |
| **Sensor de movimiento** | Detalles | Giroscopio de 3 ejes y acelerómetro de 3 ejes | Acelerómetro de 3 ejes |
| **Cámara RGB**    | Detalles | 3840 x 2160 px @30 fps | 1920 x 1080 px @30 fps |
| **Cámara de profundidad**  | Método   | Tiempo de vuelo | Tiempo de vuelo |
|                   | Resolución | 640 x 576 px @30 fps | 512 x 424 px a 30 fps |
|                   |            | 512 x 512 px @30 fps |                       |
|                   |            | 1024 x 1024 px @15 fps |                       |
| **Conectividad** | data | USB 3.1 Gen 1 con tipo USB-C  | USB 3.1 Gen 1|
|  | Power | PSU externo o USB-C | PSU externo |
|  | Synchronization | RGB y de profundidad interna, externa dispositivo a dispositivo| RGB y de profundidad interna únicamente |
| **Mecánica** | Dimensions | 103 x 39 x 126 mm | 249 x 66 x 67 mm |
|  | Peso | 440 g | 970 g |
| | Montaje | Un ¼-20 UNC. Cuatro puntos de atornillado internos | Un ¼-20 UNC |

Puede obtener más detalles en el documento [Especificaciones de hardware de Azure Kinect DK](hardware-specification.md).

## <a name="sensor-access"></a>Acceso a los sensores

En la tabla siguiente se proporciona una comparación detallada de capacidades de acceso a los sensores del dispositivo.

| **Funcionalidad**| **Azure Kinect** | **Kinect para Windows** | **Notas** |
|---------|---------|------------|---------|
| **Profundidad** | ✔️ | ✔️ |    |   |
| **IR** | ✔️ | ✔️ |  |
| **Color** | ✔️ | ✔️ | El formato de color admite diferencias, Azure Kinect DK admite estos controles de cámara: exposición, balance de blancos, brillo, contraste, saturación, nitidez y control de ganancia |
| **Audio** | ✔️ | ✔️ | El acceso a los micrófonos de Azure Kinect DK se realiza a través del SDK de Voz o la API nativa de Windows |
| **IMU** | ✔️ |  | Azure Kinect DK tiene un IMU completo de 6 ejes y Kinect para Windows solo proporciona un eje |
| **Datos de calibración** | ✔️ | ✔️ | Calibración de modelo de cámara compatible con OpenCV |
| **Sincronización interna de profundidad y RGB** | ✔️ | ✔️ |  |
| **Sincronización externa**| ✔️|  | Azure Kinect DK permite un retraso programable para la sincronización externa |
| **Acceso compartido con varios clientes** | | ✔️ | El SDK del sensor Azure Kinect se basa en WinUSB/libUSB para acceder al dispositivo y no tiene ningún servicio implementado para permitir el uso compartido del acceso al dispositivo con varios procesos |
| **Herramienta de grabación o reproducción de la transmisión** | ✔️ | ✔️ | Azure Kinect DK usa una implementación basada en contenedor Matroska de código abierto |

## <a name="features"></a>Características

El conjunto de características del SDK de Azure Kinect es diferente al de Kinect para Windows v2, como se detalla a continuación:

| **Característica de Kinect v2** | **Tipo de datos de Kinect v2** | **SDK o servicio de Azure Kinect** |
|--------|--------|------|
| Acceso a datos del sensor |DepthFrame| [SDK del sensor: recuperación de imágenes](retrieve-images.md) 
| |InfraredFrame | [SDK del sensor: recuperación de imágenes](retrieve-images.md) 
| | ColorFrame | [SDK del sensor: recuperación de imágenes](retrieve-images.md) | 
| | AudioBeamFrame |No se admite actualmente. 
| Seguimiento de personas | BodyFrame | SDK de seguimiento de personas |
| | BodyIndexFrame | SDK de seguimiento de personas  |
| Asignación de coordenadas|CoordinateMapper| [SDK del sensor: transformaciones de imágenes](use-image-transformation.md) |
|Seguimiento facial | FaceFrame | [Cognitive Services: Face](https://azure.microsoft.com/services/cognitive-services/face/)       |
|    Reconocimiento de voz    |    N/D                      |    [Cognitive Services: Voz](https://azure.microsoft.com/services/cognitive-services/directory/speech/)     |

## <a name="next-steps"></a>Pasos siguientes

[Páginas para desarrolladores de Kinect para Windows](https://developer.microsoft.com/windows/kinect)
