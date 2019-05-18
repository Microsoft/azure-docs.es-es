---
title: Vídeo escenas de indizador, capturas y fotogramas clave - Azure
titlesuffix: Azure Media Services
description: En este tema se proporciona información general de Video Indexer escenas, capturas y fotogramas clave.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 05/15/2019
ms.author: juliako
ms.openlocfilehash: d312a93f83ef38fa1ae855a1e313280fc608948d
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65799399"
---
# <a name="scenes-shots-and-keyframes"></a>Escenas, capturas y fotogramas clave

Video Indexer es compatible con vídeos segmentar en unidades temporales basadas en propiedades estructurales y semánticas. Esta funcionalidad permite a los clientes buscar fácilmente, administrar y editar su contenido de vídeo en función de las granularidades variables. Por ejemplo, se basa en segundo plano, las capturas y fotogramas clave, que se describe en este tema. El **detección escena** característica está actualmente en versión preliminar.   

![Escenas, capturas y fotogramas clave](./media/scenes-shots-keyframes/scenes-shots-keyframes.png)

## <a name="scene-detection-preview"></a>Detección de escena (versión preliminar)

Video Indexer determina cuando se cambia una escena en según las indicaciones visuales de vídeo. Una escena representa un evento único y se compone de una serie de capturas consecutivas, que son semánticamente relacionados. Una vista en miniatura de escena es el primer fotograma clave de su captura subyacente. Video indexer segmenta un vídeo en segundo plano en función de la coherencia de color en capturas consecutivas y recupera el principio y la hora de finalización de cada escena. Detección de la escena se considera una tarea complicada porque implica cuantificación aspectos semánticas de vídeos.

> [!NOTE]
> Se aplica a los vídeos que contengan al menos 3 escenas.

## <a name="shot-detection"></a>Detección de cortes

Video Indexer se determina cuando cambia una instantánea en el vídeo en función de las indicaciones visuales, mediante el seguimiento de las transiciones graduales y repentina en la combinación de colores de fotogramas adyacentes. Metadatos de captura incluyen un inicio y hora de finalización, así como la lista de fotogramas clave incluidos en esa captura. Las capturas son fotogramas consecutivos que se toman de la cámara del misma al mismo tiempo.

## <a name="keyframe-detection"></a>Detección de fotogramas clave

Selecciona los fotogramas que mejor representen la captura. Los fotogramas clave son los marcos representativos seleccionados en todo el vídeo en función de propiedades estéticas (por ejemplo, el contraste y stableness). Video Indexer recupera una lista de identificadores de fotograma clave como parte de metadatos de la captura, basándose en los clientes que pueden extraer la miniatura del fotograma clave. 

Los fotogramas clave se asocian con capturas de pantalla en la salida JSON. 

## <a name="next-steps"></a>Pasos siguientes

[Examine la salida de Video Indexer producida por la API](video-indexer-output-json-v2.md#scenes)