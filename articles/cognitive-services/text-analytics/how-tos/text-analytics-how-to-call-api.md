---
title: Llamada a la API de Text Analytics
titleSuffix: Azure Cognitive Services
description: En este artículo se explica cómo se puede llamar a la API REST y a Postman de Text Analytics de Azure Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 07/30/2019
ms.author: aahi
ms.openlocfilehash: b2c994d23e63f9e2118cd3e6571c5dcc0449a367
ms.sourcegitcommit: 80b9c8ef63cc75b226db5513ad81368b8ab28a28
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90601102"
---
# <a name="how-to-call-the-text-analytics-rest-api"></a>Cómo llamar a la API REST de Text Analytics

Las llamadas a **Text Analytics API** son llamadas HTTP POST o GET, que se pueden formular en cualquier lenguaje. En este artículo, se usan REST y [Postman](https://www.postman.com/downloads/) para demostrar los conceptos clave.

Cada solicitud debe incluir la clave de acceso y un punto de conexión HTTP. El punto de conexión especifica la región seleccionada durante el registro, la dirección URL del servicio y un recurso que se usa en la solicitud: `sentiment`, `keyphrases`, `languages` y `entities`. 

Recuerde que Text Analytics no tiene estado, por lo que no hay ningún recurso de datos para administrar. El texto se carga, se analiza tras la recepción, y los resultados se devuelven inmediatamente a la aplicación que realiza la llamada.

[!INCLUDE [text-analytics-api-references](../includes/text-analytics-api-references.md)]

## <a name="prerequisites"></a>Prerequisites

[!INCLUDE [cognitive-services-text-analytics-signup-requirements](../../../../includes/cognitive-services-text-analytics-signup-requirements.md)]

<a name="json-schema"></a>

## <a name="json-schema-definition"></a>Definición de esquema JSON

La entrada debe ser JSON en texto no estructurado sin formato. XML no se admite. El esquema es sencillo y consta de los elementos descritos en la lista siguiente. 

Actualmente puede enviar los mismos documentos para todas las operaciones de Text Analytics: opiniones, frases clave, detección de idioma e identificación de la entidad. (Es probable que el esquema varíe para cada análisis en el futuro).

| Elemento | Valores válidos | ¿Necesario? | Uso |
|---------|--------------|-----------|-------|
|`id` |El tipo de datos es una cadena, pero en la práctica, los identificadores de documento tienden a ser números enteros. | Obligatorio | El sistema usa los identificadores que proporcione para estructurar la salida. Se generan códigos de idioma, frases clave y puntuaciones de opinión para cada identificador de la solicitud.|
|`text` | Texto sin formato no estructurado, hasta 5120 caracteres. | Obligatorio | Para la detección de idioma, el texto se puede expresar en cualquier idioma. Para el análisis de opiniones, la extracción de frases clave y la identificación de entidades, el texto debe estar en un [idioma admitido](../text-analytics-supported-languages.md). |
|`language` | Código [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) de dos caracteres para un [idioma admitido](../text-analytics-supported-languages.md) | Varía | Análisis de opiniones, extracción de frases clave, detección de idioma y vinculación de entidades; opcional para la detección de idioma. No hay ningún error si se excluye, pero sin él se debilita el análisis. El código de idioma se debe corresponder al `text` que proporcione. |

Para obtener más información sobre los límites, vea [Información general de Text Analytics > Límites de datos](../overview.md#data-limits). 


```json
{
  "documents": [
    {
      "language": "en",
      "id": "1",
      "text": "Sample text to be sent to the text analytics api."
    },
    {
      "language": "en",
      "id": "2",
      "text": "It's incredibly sunny outside! I'm so happy."
    },
    {
      "language": "en",
      "id": "3",
      "text": "Pike place market is my favorite Seattle attraction."
    }
  ]
}
```


## <a name="set-up-a-request-in-postman"></a>Configuración de una solicitud en Postman

El servicio acepta solicitudes de hasta 1 MB de tamaño. Si usa Postman (u otra herramienta de prueba de API web), configure el punto de conexión para incluir el recurso que quiera usar, y proporcione la clave de acceso en un encabezado de solicitud. Cada operación requiere que anexe el recurso apropiado para el punto de conexión. 

1. En Postman:

   + Elija **Post** como el tipo de solicitud.
   + Pegue el punto de conexión que copió de la página del portal.
   + Anexe un recurso.

   Los puntos de conexión de recursos son los siguientes (la región puede variar):

   + `https://westus.api.cognitive.microsoft.com/text/analytics/v3.0/sentiment`
   + `https://westus.api.cognitive.microsoft.com/text/analytics/v3.0/keyPhrases`
   + `https://westus.api.cognitive.microsoft.com/text/analytics/v3.0/languages`
   + `https://westus.api.cognitive.microsoft.com/text/analytics/v3.0/entities/recognition/general`

2. Establezca los tres encabezados de solicitud:

   + `Ocp-Apim-Subscription-Key`: la clave de acceso obtenida de Azure Portal.
   + `Content-Type`: application/json.
   + `Accept`: application/json.

   La solicitud debe ser similar a la captura de pantalla siguiente, en la que se asume un recurso **/keyPhrases**.

   ![Captura de pantalla de solicitud con el punto de conexión y los encabezados](../media/postman-request-keyphrase-1.png)

4. Haga clic en **Body** (Cuerpo) y elija **raw** (sin formato) para el formato.

   ![Captura de pantalla de la solicitud con la configuración del cuerpo](../media/postman-request-body-raw.png)

5. Pegue algunos documentos JSON en un formato válido para el análisis previsto. Para obtener más información sobre un análisis determinado, vea los temas siguientes:

  + [Detección de idioma](text-analytics-how-to-language-detection.md)  
  + [Extracción de frases clave](text-analytics-how-to-keyword-extraction.md)  
  + [Análisis de opiniones](text-analytics-how-to-sentiment-analysis.md)  
  + [Reconocimiento de entidades](text-analytics-how-to-entity-linking.md)  


6. Haga clic en **Send** (Enviar) para enviar la solicitud. Consulte la sección sobre [límites de datos](../overview.md#data-limits) de la introducción para obtener información sobre el número de solicitudes que puede enviar por minuto y segundo.

   En Postman, la respuesta se muestra en la ventana siguiente, como un solo documento JSON, con un elemento para cada identificador de documento proporcionado en la solicitud.

## <a name="see-also"></a>Consulte también 

 [Introducción a Text Analytics](../overview.md)  
 [Preguntas más frecuentes](../text-analytics-resource-faq.md)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Detección de idioma](text-analytics-how-to-language-detection.md)
