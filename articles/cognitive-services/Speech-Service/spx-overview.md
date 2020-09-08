---
title: La CLI de voz de Azure
titleSuffix: Azure Cognitive Services
description: La CLI de voz es una herramienta de línea de comandos para usar el servicio de voz sin necesidad de escribir ningún código. La CLI de voz requiere una configuración mínima y es fácil empezar a experimentar de inmediato con características clave del servicio de voz para ver si pueden resultar útiles para sus casos de uso.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/14/2020
ms.author: trbye
ms.custom: devx-track-azurecli
ms.openlocfilehash: 6b852186834fba858e8a049a8230b38f3d69164d
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88067433"
---
# <a name="what-is-the-speech-cli"></a>¿Qué es la CLI de Voz?

La CLI de voz es una herramienta de línea de comandos para usar el servicio de voz sin necesidad de escribir ningún código. La CLI de voz requiere una configuración mínima y es fácil empezar a experimentar de inmediato con características clave del servicio de voz para ver si pueden resultar útiles para sus casos de uso. En cuestión de minutos, podrá ejecutar flujos de trabajo de prueba sencillos como, por ejemplo, el reconocimiento de voz por lotes de un directorio de archivos o la conversión de texto a voz en una colección de cadenas de un archivo. Además de los flujos de trabajo sencillos, la CLI de voz está preparada para producción y se puede escalar verticalmente para ejecutar procesos más grandes mediante `.bat` automatizados o scripts de shell.

La mayoría de las características principales del SDK de voz están disponibles en la CLI de voz y algunas características y personalizaciones avanzadas se han simplificado en esta última. Tenga en cuenta las siguientes instrucciones para decidir cuándo usar la CLI de voz o el SDK de voz.

Use la CLI de voz cuando:
* Quiera experimentar con las características del servicio de voz con una configuración mínima y sin código
* Tenga requisitos relativamente sencillos para una aplicación de producción que usa el servicio de voz

Use el SDK de voz cuando:
* Desee integrar la funcionalidad del servicio de voz en un lenguaje o plataforma específicos (por ejemplo C#, Python, C++).
* Tiene requisitos complejos que pueden requerir solicitudes de servicio avanzadas o desarrollar un comportamiento personalizado que incluye el streaming de respuesta

## <a name="core-features"></a>Características principales

* Reconocimiento de voz: convierta voz en texto a partir de archivos de audio o directamente de un micrófono, o transcriba una conversación grabada.

* Síntesis de voz: convierta texto a voz mediante la entrada de archivos de texto o directamente desde la línea de comandos. Personalización de las características de salida de voz mediante [configuraciones de SSML](speech-synthesis-markup.md) y [voces estándar o neuronales](speech-synthesis-markup.md#standard-neural-and-custom-voices).

* Traducción de voz: traduce el audio de un idioma de origen en texto o audio en un idioma de destino.

* Ejecución en recursos de proceso de Azure: envíe comandos de la CLI de Voz para ejecutarlos en un recurso de proceso remoto de Azure mediante `spx webjob`.

## <a name="get-started"></a>Introducción

Para empezar a usar la CLI de voz, consulte el [artículo sobre conceptos básicos](spx-basics.md). En este artículo se muestra cómo ejecutar algunos comandos básicos y también se muestran comandos algo más avanzados para ejecutar operaciones por lotes de conversión de voz a texto y texto a voz. Después de leer el artículo sobre conceptos básicos, debe tener suficiente conocimiento de la sintaxis para empezar a escribir algunos comandos personalizados o automatizar las operaciones sencillas de voz.

## <a name="next-steps"></a>Pasos siguientes

- [Conceptos básicos sobre la CLI de voz](spx-basics.md)
- Si su caso de uso es más complejo, [obtenga el SDK de voz](speech-sdk.md)
