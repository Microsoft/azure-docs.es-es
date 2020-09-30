---
title: Ejecución del ejemplo de contenedor del comando de ejecución de Docker
titleSuffix: Azure Cognitive Services
description: Comando docker run para el contenedor de análisis de sentimiento
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 04/29/2020
ms.author: aahi
ms.openlocfilehash: e4a08217b5476f4934325518bec00d640003aece
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906107"
---
Para ejecutar el contenedor *Análisis de sentimiento v3*, ejecute el siguiente comando `docker run`. Reemplace los marcadores de posición por sus propios valores:

| Marcador de posición | Value | Formato o ejemplo |
|-------------|-------|---|
| **{CLAVE_API}** | Clave del recurso de Text Analytics. Puede encontrarla en la página **Clave y punto de conexión** del recurso en Azure Portal. |`xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`|
| **{URI_PUNTODECONEXIÓN}** | Punto de conexión para acceder a Text Analytics API. Puede encontrarla en la página **Clave y punto de conexión** del recurso en Azure Portal. | `https://<your-custom-subdomain>.cognitiveservices.azure.com` |

```bash
docker run --rm -it -p 5000:5000 --memory 8g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Este comando:

* Ejecuta un contenedor de *Análisis de sentimiento* desde la imagen de contenedor.
* Asigna un núcleo de CPU y 8 gigabytes (GB) de memoria.
* Expone el puerto TCP 5000 y asigna un seudo-TTY para el contenedor.
* Una vez que se produce la salida, quita automáticamente el contenedor. La imagen del contenedor sigue estando disponible en el equipo host.