---
ms.openlocfilehash: 9c1b521a0f10da77295fd2457793566d787cb2cd
ms.sourcegitcommit: d0541eccc35549db6381fa762cd17bc8e72b3423
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89570110"
---
> [!div class="mx-imgBorder"]
> :::image type="content" source="../../../media/quickstarts/overview-qs4.svg" alt-text="Flujo de señales":::

En el diagrama anterior se muestra cómo fluyen las señales en este inicio rápido. [Un módulo perimetral](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555) simula una cámara IP que hospeda un servidor de protocolo RTSP. Un nodo de [origen RTSP](../../../media-graph-concept.md#rtsp-source) extrae la fuente de vídeo de este servidor y envía fotogramas de vídeo al nodo del [procesador de detección del movimiento](../../../media-graph-concept.md#motion-detection-processor). El origen RTSP envía los mismos fotogramas de vídeo a un nodo de [procesador de la puerta de señales](../../../media-graph-concept.md#signal-gate-processor), que permanece cerrado hasta que un evento lo desencadena.

Cuando el procesador de detección del movimiento detecta movimiento en el vídeo, envía un evento al nodo de procesador de la puerta de señales y lo desencadena. La puerta se abre durante el tiempo configurado y envía fotogramas de vídeo al nodo del [receptor de archivos](../../../media-graph-concept.md#file-sink). Este nodo del receptor registra el vídeo como un archivo MP4 en el sistema de archivos local del dispositivo perimetral. El archivo se guarda en la ubicación configurada.

En este inicio rápido realizará lo siguiente:

1. Creará e implementará el grafo de elementos multimedia.
1. Interpretará los resultados.
1. Limpieza de recursos.
