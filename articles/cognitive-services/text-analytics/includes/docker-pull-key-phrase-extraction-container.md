---
title: Docker pull para el contenedor de extracción de frases clave
titleSuffix: Azure Cognitive Services
description: Comando docker pull para el contenedor de extracción de frases clave
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 02dde8a27b9687e58bf1a09c1a951f306937f0d6
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906033"
---
#### <a name="docker-pull-for-the-key-phrase-extraction-container"></a>Docker pull para el contenedor de extracción de frases clave

Use el comando [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) para descargar la imagen de contenedor del Registro de contenedor de Microsoft.

Para obtener una descripción completa de las etiquetas disponibles para los contenedores de Text Analytics, consulte el contenedor [Extracción de frases clave](https://go.microsoft.com/fwlink/?linkid=2018757) en Docker Hub.

```
docker pull mcr.microsoft.com/azure-cognitive-services/textanalytics/keyphrase:latest
```
