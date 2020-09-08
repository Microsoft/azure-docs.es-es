---
title: Preguntas más frecuentes - Traductor personalizado
titleSuffix: Azure Cognitive Services
description: En este artículo se incluyen respuestas a preguntas frecuentes sobre Custom Translator de Azure Cognitive Services.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: swmachan
ms.topic: reference
ms.openlocfilehash: 06caafe75682a2375a8023787f9905ca1e8117c9
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88507236"
---
# <a name="custom-translator-frequently-asked-questions"></a>Traductor personalizado: preguntas más frecuentes

En este artículo se incluyen respuestas a preguntas frecuentes sobre [Custom Translator](https://portal.customtranslator.azure.ai).

## <a name="what-are-the-current-restrictions-in-custom-translator"></a>¿Cuáles son las restricciones actuales de Custom Translator?

Existen restricciones y límites con respecto al tamaño del archivo, el entrenamiento de los modelos y la implementación de los modelos. Recuerde estas restricciones cuando configure el entrenamiento para crear un modelo en Custom Translator.

- Los archivos enviados deben tener un tamaño inferior a 100 MB.
- No se admiten datos monolingües.

## <a name="when-should-i-request-deployment-for-a-translation-system-that-has-been-trained"></a>¿Cuándo debo solicitar la implementación de un sistema de traducción que ya se ha entrenado?

Crear el sistema de traducción óptimo para su proyecto puede requerir varios entrenamientos. Si la puntuación BLEU o los resultados de la prueba no son satisfactorios, es posible que quiera volver a intentarlo con más datos de aprendizaje o con datos filtrados más detenidamente. Debe ser estricto y cuidadoso mientras diseña sus conjuntos de ajuste y de pruebas, de forma que ambos estén completamente relacionados con la terminología y el estilo del material que quiere traducir. Puede ser más flexible en la creación de los datos de aprendizaje y experimentar con distintas opciones. Solicite una implementación de sistema cuando esté satisfecho con las traducciones en los resultados de prueba del sistema, cuando no tenga más datos que agregar al entrenamiento para mejorar el sistema, y cuando quiera acceder al modelo entrenado a través de las API.

## <a name="how-many-trained-systems-can-be-deployed-in-a-project"></a>¿Cuántos sistemas entrenados se pueden implementar en un proyecto?

Solo puede implementar un sistema entrenado por proyecto. Puede requerir de varios entrenamientos para crear un sistema de traducción adecuado para el proyecto, y recomendamos que solicite la implementación de un entrenamiento que le ofrezca el mejor resultado. Puede determinar la calidad del entrenamiento según la puntuación BLEU (mayor es mejor) y consultando con los revisores antes de decidir si la calidad de las traducciones es adecuada para la implementación.

## <a name="when-can-i-expect-my-trainings-to-be-deployed"></a>¿Cuándo se implementarán mis entrenamientos?

La implementación normalmente tarda menos de una hora.

## <a name="how-do-you-access-a-deployed-system"></a>¿Cómo se tiene acceso a un sistema implementado?

Se puede acceder a los sistemas implementados a través de Microsoft Translator Text API V3 especificando el valor del id. de categoría. Puede encontrar más información sobre Translator Text API en la página web de [referencia de API](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference).

## <a name="how-do-i-skip-alignment-and-sentence-breaking-if-my-data-is-already-sentence-aligned"></a>¿Cómo omito la alineación y la separación de oraciones si mis datos ya están alineados por oración?

Custom Translator omite la alineación de oraciones y la separación de oraciones en los archivos TMX y en los archivos de texto con la extensión `.align`. Los archivos `.align` proporcionan a los usuarios una alternativa para omitir el proceso de alineación y separación de oraciones de Traductor personalizado para los archivos que están perfectamente alineados y no necesitan de ningún procesamiento adicional. Se recomienda usar la extensión `.align` solo para los archivos que estén perfectamente alineados.

Si el número de oraciones extraídas no coincide con los dos archivos del mismo nombre base, Custom Translator ejecutará el proceso de alineación de oraciones en los archivos `.align` de todos modos.

## <a name="i-tried-uploading-my-tmx-but-it-says-document-processing-failed"></a>He intentado cargar mi TMX, pero recibo el mensaje "Error de procesamiento de documento"


Asegúrese de que el TMX se ajusta a las especificaciones TMX 1.4b en <https://www.gala-global.org/tmx-14b>.
