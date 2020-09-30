---
title: ¿Qué es Form Recognizer?
titleSuffix: Azure Cognitive Services
description: El Azure Cognitive Services Form Recognizer le permite identificar y extraer pares clave-valor y datos de tabla de documentos de formulario.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: overview
ms.date: 08/05/2020
ms.author: pafarley
ms.openlocfilehash: 070796cd260e56bb51115a7ef33ced8455bfb6a9
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89394404"
---
# <a name="what-is-form-recognizer"></a>¿Qué es Form Recognizer?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

Azure Form Recognizer es un servicio cognitivo que utiliza tecnología de aprendizaje automático para identificar y extraer texto, pares clave-valor, así como datos de tablas de documentos de formularios. Ingiere texto de formularios y genera datos estructurados que incluyen las relaciones del archivo original. Obtendrá rápidamente resultados precisos a la medida de su contenido específico sin necesidad de una intervención manual pesada o una amplia experiencia en ciencia de datos. Form Recognizer consta de modelos personalizados, el modelo de recibo pregenerado y la API de diseño. Puede llamar a los modelos de Form Recognizer mediante una API REST para reducir la complejidad e integrarla en su flujo de trabajo o aplicación.

Form Recognizer se compone de los siguientes servicios:
* **Modelos personalizados**: extraen los pares clave-valor y los datos de tablas de los formularios. Puede entrenar estos modelos con sus propios datos para que se adapten a sus formularios.
* **Modelos creados previamente**: extrae datos de tipos de formulario únicos mediante modelos pregenerados. Actualmente, hay modelos pregenerados para las confirmaciones de ventas y las tarjetas de presentación en inglés.
* **API de diseño**: extrae estructuras de texto y tabla, junto con sus coordenadas de los cuadros de límite, desde los documentos.

<!-- add diagram -->

## <a name="custom-models"></a>Modelos personalizados

Puede entrenar los modelos personalizados de Form Recognizer con sus propios datos y solo necesita cinco formularios de entrada de ejemplo para empezar. Un modelo entrenado puede generar datos estructurados que incluyen las relaciones del documento de formulario original. Después de entrenar el modelo, puede probarlo, volver a entrenarlo y finalmente usarlo para extraer datos de forma confiable de más formularios en función de las propias necesidades.

Tiene las siguientes opciones al entrenar modelos personalizados: entrenamiento con datos etiquetados y sin ellos.

### <a name="train-without-labels"></a>Entrenamiento sin etiquetas

De forma predeterminada, Form Recognizer usa el aprendizaje sin supervisión para comprender el diseño y las relaciones entre los campos y las entradas de los formularios. Cuando se envían los formularios de entrada, el algoritmo agrupa los formularios por tipos, descubre qué claves y tablas están presentes y asocia los valores a las claves y entradas a las tablas. Esto no requiere el etiquetado de datos manual ni una codificación intensiva, ni mantenimiento y se recomienda probar este método primero.

### <a name="train-with-labels"></a>Entrenamiento con etiquetas

Al entrenar con datos etiquetados, el modelo realiza un aprendizaje supervisado para extraer valores de interés mediante los formularios etiquetados que se proporcionan. Esto da como resultado modelos con un mejor rendimiento y puede generar modelos que funcionen con formularios complejos o formularios que contengan valores sin claves.

Form Recognizer usa la [API de diseño](#layout-api) para aprender los tamaños y posiciones esperados de los elementos de texto impresos y manuscritos. A continuación, usa etiquetas especificadas por el usuario para aprender las asociaciones clave-valor de los documentos. Se recomienda usar cinco formularios etiquetados manualmente del mismo tipo para empezar a entrenar un nuevo modelo y agregar más datos etiquetados según sea necesario para mejorar la precisión del modelo.

## <a name="prebuilt-models"></a>Modelos creados previamente

Form Recognizer también incluye modelos pregenerados para tipos de formulario únicos.

### <a name="prebuilt-receipt-model"></a>Modelo de recibo pregenerado
El modelo de recibo pregenerado se usa para leer los recibos de ventas en Inglés de Australia, Canadá, Gran Bretaña, India y Estados Unidos (el tipo que usan los restaurantes, las estaciones de gas, minoristas, etc). Este modelo extrae información clave como la hora y la fecha de la transacción, información del comerciante, importe de los impuestos, totales y mucho más. Además, el modelo de recibo pregenerado se entrena para reconocer y devolver todo el texto en un recibo. 

![recibo de ejemplo](./media/contoso-receipt-small.png)

### <a name="prebuilt-business-cards-model"></a>Modelo de tarjetas de presentación pregenerado
El modelo de tarjetas de presentación le permite extraer información como el nombre de la persona, el puesto de trabajo, la dirección, el correo electrónico, la empresa y los números de teléfono de las tarjetas de presentación en inglés. 

![tarjeta de presentación de ejemplo](./media/business-card-english.jpg)

## <a name="layout-api"></a>API de diseño

Form Recognizer también puede extraer texto y estructuras de tablas (los números de fila y columna asociados al texto) mediante el reconocimiento óptico de caracteres (OCR) de alta definición.

## <a name="get-started"></a>Introducción

Siga un inicio rápido para comenzar a extraer datos de los formularios. Se recomienda usar el servicio gratuito cuando se está aprendiendo la tecnología. Recuerde que el número de páginas gratuitas se limita a 500 al mes.

* [Inicio rápido de la biblioteca cliente](./quickstarts/client-library.md) (todos los lenguajes, varios escenarios)
* Inicios rápidos de la interfaz de usuario web
  * [Entrenamiento con etiquetas: herramienta de etiquetado de ejemplo](quickstarts/label-tool.md)
* Inicios rápidos de REST
  * Entrenamiento de modelos personalizados y extracción de datos de formularios
    * [Entrenamiento sin etiquetas: cURL](quickstarts/curl-train-extract.md)
    * [Entrenamiento sin etiquetas: Python](quickstarts/python-train-extract.md)
    * [ Entrenamiento con etiquetas: Python](quickstarts/python-labeled-data.md)
  * Extracción de datos de recibos de ventas
    * [Extracción de datos de recibos: cURL](quickstarts/curl-receipts.md)
    * [Extracción de datos de recibos: Python](quickstarts/python-receipts.md)
  * Extracción de datos de tarjetas de presentación
    * [Extracción de datos de tarjetas de presentación: Python](quickstarts/python-business-cards.md)
  * Extracción de texto y estructura de tablas de formularios
    * [Extracción de datos de diseño: Python](quickstarts/python-layout.md)


### <a name="review-the-rest-apis"></a>Revisión de las API REST

Va a utilizar las siguientes API para entrenar modelos y extraer datos estructurados de formularios.

|Nombre |Descripción |
|---|---|
| **Entrenamiento de modelos personalizados**| Entrene un nuevo modelo para analizar los formularios con cinco formularios del mismo tipo. Establezca el parámetro _useLabelFile_ en `true` para entrenar con datos etiquetados manualmente. |
| **Análisis de formularios** |Analice un documento único pasado como una secuencia para extraer texto, pares clave-valor y tablas del formulario con su modelo personalizado.  |
| **Análisis de recibos** |Analice un documento de entrada individual para extraer la información clave y cualquier otro texto del recibo.|
| **Análisis de una tarjeta de presentación** |Analice una tarjeta de presentación para extraer el texto y la información más importante.|
| **Análisis de diseño** |Analice el diseño de un formulario para extraer texto y estructuras de tablas.|

Explore la [documentación de referencia de API REST](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeWithCustomForm) para más información. Si está familiarizado con una versión anterior de la API, consulte el artículo sobre [novedades](./whats-new.md) para más información sobre los cambios recientes.

## <a name="input-requirements"></a>Requisitos de entrada
### <a name="custom-model"></a>Modelo personalizado

[!INCLUDE [input requirements](./includes/input-requirements.md)]

### <a name="prebuilt"></a>Creada previamente

Los requisitos de entrada para el modelo de recibo son ligeramente diferentes.

* El formato debe ser JPG, PNG, PDF (texto o digitalizado) o TIFF.
* El tamaño del archivo debe ser inferior a 20 MB.
* Las imágenes deben tener unas dimensiones entre 50 x 50 píxeles y 10 000 x 10 000 píxeles.
* Los archivos PDF deben tener unas dimensiones de 17 x 17 pulgadas como máximo, lo que se corresponde con los tamaños de papel Legal o A3 y más pequeños.
* En el caso de PDF y TIFF, solo se procesan las primeras 200 páginas (con una suscripción de nivel gratuitos, solo se procesan las dos primeras páginas).

## <a name="data-privacy-and-security"></a>Seguridad y privacidad de datos

Al igual que sucede con todas las instancias de Cognitive Services, los desarrolladores que usan el servicio Form Recognizer deben estar al tanto de las directivas de Microsoft sobre los datos de los clientes. Para más información, consulte la [página de Cognitive Services](https://www.microsoft.com/trustcenter/cloudservices/cognitiveservices) en Microsoft Trust Center.

## <a name="next-steps"></a>Pasos siguientes

Realice un [inicio rápido](quickstarts/curl-train-extract.md) para empezar a usar las [API de Form Recognizer](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeWithCustomForm).