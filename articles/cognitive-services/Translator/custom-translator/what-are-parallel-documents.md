---
title: ¿Qué son los documentos paralelos? - Custom Translator
titleSuffix: Azure Cognitive Services
description: Los documentos paralelos son pares de documentos en los que uno es la traducción del otro. Un documento en el par contiene frases en el idioma de origen y el otro documento contiene estas mismas frases traducidas al idioma de destino.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: ce9f7e922b9a168205ea1d193830dd654691998a
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88510817"
---
# <a name="what-are-parallel-documents"></a>¿Qué son los documentos paralelos?

Los documentos paralelos son pares de documentos en los que uno es la traducción del otro. Un documento en el par contiene frases en el idioma de origen y el otro documento contiene estas mismas frases traducidas al idioma de destino.
No importa qué idioma está marcado como "origen" y qué idioma como "destino": un documento paralelo se puede usar para entrenar un sistema de traducción en cualquier dirección.

## <a name="requirements"></a>Requisitos

Necesita un mínimo de 10 000 frases paralelas alineadas para entrenar a un sistema. Esta limitación es una red de seguridad para asegurarse de que las frases paralelas contienen suficiente vocabulario único para entrenar correctamente un modelo de traducción. Como procedimiento recomendado, agregue continuamente más contenido paralelo y entrene el sistema de nuevo para mejorar la calidad de la traducción. Consulte [Alineación de frases](https://docs.microsoft.com/azure/cognitive-services/translator/custom-translator/sentence-alignment).

Microsoft requiere que los documentos cargados en Custom Translator no infrinjan los derechos de propiedad intelectual o de copyright de terceros. Para más información, consulte los [términos de uso](https://azure.microsoft.com/support/legal/cognitive-services-terms/).
Cargar un documento mediante el portal no modifica la propiedad intelectual del documento en sí.

## <a name="use-of-parallel-documents"></a>Uso de documentos paralelos

El sistema usa los documentos paralelos para:

1.  Conocer cómo se asignan normalmente las palabras, expresiones y frases entre los dos idiomas.

2.  Saber cómo procesar el contexto adecuado según las frases que lo rodean. Puede que una palabra no se traduzca siempre exactamente como otra palabra del otro idioma.

Como procedimiento recomendado, asegúrese de que hay una correspondencia completa entre las frases en las versiones de los documentos de origen y de destino.

Si el proyecto es de un dominio (o categoría) específico, los documentos deben contener una terminología coherente con esa categoría. La calidad del sistema de traducción resultante depende del número de frases del conjunto de documentos y de la calidad de las mismas. Cuantos más ejemplos contengan los documentos con usos diversos de una palabra específica de la categoría, mejor realizará el sistema la traducción.

Los documentos que se cargan son privados para cada área de trabajo y se pueden usar en tantos proyectos o aprendizajes como desee. Las frases extraídas de los documentos se almacenan de forma independiente en el repositorio como archivos de texto Unicode sin formato y están disponibles para su eliminación cuando sea necesario. No use Custom Translator como repositorio de documentos ya que no podrá descargar los documentos que cargó previamente en el mismo formato que los cargó.



## <a name="next-steps"></a>Pasos siguientes

- Aprenda a usar un [diccionario](what-is-dictionary.md) en Custom Translator.
