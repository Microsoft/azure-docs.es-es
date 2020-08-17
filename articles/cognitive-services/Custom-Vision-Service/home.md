---
title: ¿Qué es Custom Vision?
titleSuffix: Azure Cognitive Services
description: Aprenda a usar Custom Vision Service para crear clasificadores personalizados de imágenes en la nube de Azure.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: overview
ms.date: 08/05/2020
ms.author: pafarley
ms.openlocfilehash: 86e2d54cfe7559675e5e878cbc8a0df4b4eb3d0b
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87834293"
---
# <a name="what-is-custom-vision"></a>¿Qué es Custom Vision?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

Azure Custom Vision es un servicio cognitivo que permite crear, implementar y mejorar los clasificadores de imágenes propios. Un clasificador de imágenes es un servicio de inteligencia artificial que aplica etiquetas (que representan _clases_) a las imágenes, en función de sus características visuales. A diferencia del servicio [Computer Vision](https://docs.microsoft.com/azure/cognitive-services/computer-vision/home), Custom Vision permite especificar las etiquetas que se van a aplicar.

## <a name="what-it-does"></a>Qué hace

Custom Vision Service usa un algoritmo de aprendizaje automático para aplicar etiquetas a las imágenes. El desarrollador debe enviar los grupos de imágenes que presenten y carezcan de las características en cuestión. Las imágenes las etiqueta el propio usuario en el momento del envío. Luego, el algoritmo se entrena con estos datos y calcula su propia precisión probándose a sí mismo en esas mismas imágenes. Una vez que el algoritmo se ha entrenado, puede probarlo, volver a entrenarlo y, por último, usarlo para clasificar las imágenes nuevas en función de las necesidades de la aplicación. También puede exportar el mismo modelo para su uso sin conexión.

### <a name="classification-and-object-detection"></a>Clasificación y detección de objetos

La funcionalidad de Custom Vision puede dividirse en dos características. La **clasificación de las imágenes** aplica una o varias etiquetas a una imagen. La **detección de objetos** es similar, pero también devuelve las coordenadas de la imagen donde pueden encontrarse las etiquetas aplicadas.

### <a name="optimization"></a>Optimization

El servicio Custom Vision está optimizado para reconocer rápidamente las diferencias principales entre las imágenes, para que pueda empezar a crear el prototipo de su modelo con una pequeña cantidad de datos. Cincuenta imágenes por etiqueta suele ser un buen comienzo. Sin embargo, el servicio no es óptimo para detectar diferencias sutiles en las imágenes (por ejemplo, detectar grietas menores o abolladuras en escenarios de control de calidad).

Además, puede elegir entre distintas variedades del algoritmo de Custom Vision que están optimizadas para imágenes con cierto material temático&mdash;, por ejemplo, puntos de referencia o artículos en venta. Para más información, consulte la guía [Cómo crear un clasificador](getting-started-build-a-classifier.md).

## <a name="what-it-includes"></a>Qué incluye

Custom Vision Service está disponible como un conjunto de SDK nativos, así como mediante una interfaz basada en web en la [página principal de Custom Vision](https://customvision.ai/). Puede crear, probar y entrenar un modelo mediante cualquiera de las interfaces, o usar ambas.

![Página principal de Custom Vision en una ventana del explorador Chrome](media/browser-home.png)

## <a name="data-privacy-and-security"></a>Seguridad y privacidad de datos

Al igual que sucede con todas las instancias de Cognitive Services, los desarrolladores que usan Custom Vision Service deben estar al tanto de las directivas de Microsoft sobre los datos de clientes. Para más información, consulte la [página de Cognitive Services](https://www.microsoft.com/trustcenter/cloudservices/cognitiveservices) en Microsoft Trust Center.

## <a name="next-steps"></a>Pasos siguientes

Siga la guía para [crear un clasificador](getting-started-build-a-classifier.md) para empezar a usar Custom Vision en la web, o bien complete un [tutorial de clasificación de imágenes](quickstarts/image-classification.md) para implementar el escenario básico en el código.
