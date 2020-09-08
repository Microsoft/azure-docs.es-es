---
title: Envío de datos de entrada de lápiz a la API Ink Recognizer
titleSuffix: Azure Cognitive Services
description: Aprenda más sobre cómo llamar a la API Ink Analyzer para diferentes aplicaciones
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: ink-recognizer
ms.topic: conceptual
ms.date: 08/24/2020
ms.author: aahi
ms.openlocfilehash: 8665e27b6b7c43bce7d749d9bfe0a2f815a0f46d
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89051107"
---
# <a name="send-ink-data-to-the-ink-recognizer-api"></a>Envío de datos de entrada de lápiz a la API Ink Recognizer 

[!INCLUDE [ink-recognizer-deprecation](../includes/deprecation-note.md)]

La entrada de lápiz digital hace referencia a las tecnologías que permiten las representaciones digitales de la entrada de texto, como la escritura a mano y los dibujos. Esto se logra típicamente mediante un digitalizador que captura los movimientos de los dispositivos de entrada, como un lápiz. A medida que los dispositivos siguen permitiendo experiencias de entradas de lápiz digital enriquecidas, la inteligencia artificial y el aprendizaje automático permiten el reconocimiento de formas escritas y textos en cualquier contexto. La API Ink Recognizer le permite enviar trazos de entrada de lápiz y obtener información detallada sobre ellos. 

## <a name="the-ink-recognizer-api-vs-ocr-services"></a>API Ink Recognizer frente a los servicios OCR

La API Ink Recognizer no utiliza el reconocimiento óptico de caracteres (OCR). Los servicios OCR procesan los datos de píxeles de las imágenes para proporcionar el reconocimiento de escritura a mano y de texto. Esto se denomina a veces reconocimiento sin conexión. En su lugar, la API Ink Recognizer requiere datos de los trazos de la entrada de lápiz digital que se capturan a medida que se utiliza el dispositivo de entrada. Este procesamiento de los datos de la entrada de lápiz digital puede producir resultados de reconocimiento más precisos en comparación con los servicios OCR. 

## <a name="sending-ink-data"></a>Envío de datos de entrada de lápiz

La API Ink Recognizer requiere las coordenadas X e Y que representan los trazos de tinta creados por un dispositivo de entrada, desde el momento en que toca la superficie de detección hasta que se levanta. Los puntos de cada trazo deben ser una cadena de valores separados por comas y deben estar formateados en JSON como en el ejemplo siguiente. Además, cada trazo de entrada de lápiz debe tener un identificador único en cada solicitud. Si el identificador se repite dentro de la misma solicitud, la API devolverá un error. Para obtener los resultados de reconocimiento más precisos, tenga al menos ocho dígitos después del punto decimal. Se supone que el origen (0,0) del lienzo es la esquina superior izquierda del lienzo de entrada de lápiz.

> [!NOTE]
> El ejemplo siguiente no es un JSON válido. Puede encontrar una solicitud completa de JSON de Ink Recognizer en [GitHub](https://go.microsoft.com/fwlink/?linkid=2089909).
 
```json
{
  "language": "en-US",
  "strokes": [
   {
    "id": 43,
    "points": 
        "5.1365, 12.3845,
        4.9534, 12.1301,
        4.8618, 12.1199,
        4.7906, 12.2217,
        4.7906, 12.5372,
        4.8211, 12.9849,
        4.9534, 13.6667,
        5.0958, 14.4503,
        5.3299, 15.2441,
        5.6555, 16.0480,
        ..."
   },
    ...
  ]
}
```

## <a name="ink-recognizer-response"></a>Respuesta de Ink Recognizer

La API Ink Recognizer devuelve una respuesta de análisis sobre los objetos que reconoce a partir del contenido de la entrada de lápiz. La respuesta contiene unidades de reconocimiento que describen las relaciones entre los diferentes trazos de entrada de lápiz. Por ejemplo, los trazos que crean formas distintas e independientes se incluirán en unidades diferentes. Cada unidad contiene información detallada sobre los trazos de entrada de lápiz, incluido el objeto reconocido, sus coordenadas y otros atributos de dibujo.

## <a name="shapes-recognized-by-the-ink-recognizer-api"></a>Formas reconocidas por la API Ink Recognizer

La API Ink Recognizer puede identificar las formas más utilizadas en la toma de notas. La siguiente imagen muestra algunos ejemplos básicos. Para obtener una lista completa de las formas y otros contenidos de entrada de lápiz reconocidos por la API, consulte el [artículo de referencia de la API](https://go.microsoft.com/fwlink/?linkid=2089907). 

![Lista de formas reconocidas por la API Ink Recognizer](../media/shapes.png)

## <a name="recommended-calling-patterns"></a>Patrones de llamada recomendados

Puede llamar a la API REST Ink Recognizer con diferentes patrones según la aplicación. 

### <a name="user-initiated-api-calls"></a>Llamadas API iniciadas por el usuario

Si va a crear una aplicación que recibe las entradas de los usuarios (por ejemplo, una aplicación de anotación o de toma de notas), es posible que desee darles el control de cuándo y qué tinta se envía a la API Ink Recognizer. Esta funcionalidad es especialmente útil cuando el texto y las formas están presentes en el lienzo, y los usuarios desean realizar diferentes acciones para cada uno. Considere la posibilidad de agregar características de selección (como un lazo u otra herramienta de selección geométrica) que permitan a los usuarios elegir lo que se envía a la API.  

### <a name="app-initiated-api-calls"></a>Llamadas API iniciadas por la aplicación

También puede hacer que la aplicación llame a la API Ink Recognizer después de un tiempo de expiración. Al enviar de forma rutinaria los trazos actuales de la entrada de lápiz a la API, puede almacenar los resultados de reconocimiento a medida que se crean, a la vez que mejora el tiempo de respuesta de la API. Por ejemplo, puede enviar una línea de texto escrito a mano a la API después de detectar que el usuario la ha completado. 

Tener los resultados del reconocimiento por adelantado le da información sobre las características de los trazos de la entrada de lápiz según se relacionan entre sí. Por ejemplo, qué trazos se agrupan para formar la misma palabra, línea, lista, párrafo o forma. Esta información puede mejorar las características de selección de la entrada de lápiz de la aplicación al poder seleccionar, por ejemplo, grupos de trazos a la vez.

## <a name="integrate-the-ink-recognizer-api-with-windows-ink"></a>Integración de la API Ink Recognizer con Windows Ink

[Windows Ink](https://docs.microsoft.com/windows/uwp/design/input/pen-and-stylus-interactions) proporciona herramientas y tecnologías para permitir experiencias de entrada de lápiz digital en una amplia gama de dispositivos. Puede combinar la plataforma Windows Ink con la API Ink Recognizer para crear aplicaciones que muestren e interpreten los trazos de entrada de lápiz digital.

## <a name="next-steps"></a>Pasos siguientes

* [¿Qué es la API Ink Recognizer?](../overview.md)
* [Referencia de API REST Ink Recognizer](https://go.microsoft.com/fwlink/?linkid=2089907)

* Empiece a enviar datos de los trazos de entrada de lápiz digital mediante:
    * [C#](../quickstarts/csharp.md)
    * [Java](../quickstarts/java.md)
    * [JavaScript](../quickstarts/javascript.md)
