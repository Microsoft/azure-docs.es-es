---
title: Método Detect de Translator Text API
titlesuffix: Azure Cognitive Services
description: Use el método Detect de Translator Text API.
services: cognitive-services
author: v-pawal
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 02/01/2019
ms.author: v-jansko
ms.openlocfilehash: ea8fe989dd0ef7026957153fb5c9836742d008dd
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65797491"
---
# <a name="translator-text-api-30-detect"></a>Translator Text API 3.0: Detectar

Identifica el idioma de un fragmento de texto.

## <a name="request-url"></a>URL de la solicitud

Envíe una solicitud `POST` a:

```HTTP
https://api.cognitive.microsofttranslator.com/detect?api-version=3.0
```

## <a name="request-parameters"></a>Parámetros de solicitud

Los parámetros de solicitud que se pasaron en la cadena de consulta son:

<table width="100%">
  <th width="20%">Parámetro de consulta</th>
  <th>DESCRIPCIÓN</th>
  <tr>
    <td>api-version</td>
    <td>*Parámetro obligatorio*.<br/>Versión de la API que el cliente solicitó. El valor debe ser `3.0`.</td>
  </tr>
</table> 

Los encabezados de solicitud incluyen lo siguiente:

<table width="100%">
  <th width="20%">Encabezados</th>
  <th>DESCRIPCIÓN</th>
  <tr>
    <td>Encabezados de autenticación</td>
    <td><em>Encabezado de solicitud obligatorio</em>.<br/>Consulte las <a href="https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference#authentication">opciones disponibles para la autenticación</a>.</td>
  </tr>
  <tr>
    <td>Content-Type</td>
    <td>*Encabezado de solicitud obligatorio*.<br/>Especifica el tipo de contenido de la carga. Los valores posibles son: `application/json`.</td>
  </tr>
  <tr>
    <td>Content-Length</td>
    <td>*Encabezado de solicitud obligatorio*.<br/>Longitud del cuerpo de la solicitud.</td>
  </tr>
  <tr>
    <td>X-ClientTraceId</td>
    <td>*Opcional*.<br/>GUID generado por el cliente para identificar de forma única la solicitud. Puede omitir este encabezado si incluye el id. de seguimiento en la cadena de la consulta mediante un parámetro de consulta denominado `ClientTraceId`.</td>
  </tr>
</table> 

## <a name="request-body"></a>Cuerpo de la solicitud

El cuerpo de la solicitud es una matriz JSON. Cada elemento de la matriz es un objeto JSON con una propiedad de cadena denominada `Text`. La detección de idioma se aplica al valor de la propiedad `Text`. Un cuerpo de solicitud de muestra tiene el siguiente aspecto:

```json
[
    { "Text": "Ich würde wirklich gern Ihr Auto um den Block fahren ein paar Mal." }
]
```

Se aplican las siguientes limitaciones:

* La matriz puede tener como máximo 100 elementos.
* El valor de texto de un elemento de la matriz no puede superar los 10 000 caracteres, incluyendo los espacios.
* El texto completo incluido en la solicitud no puede superar los 50 000 caracteres, incluidos los espacios.

## <a name="response-body"></a>Response body

Una respuesta correcta es una matriz JSON con un resultado para cada cadena en la matriz de entrada. Un objeto del resultado incluye las siguientes propiedades:

  * `language`: Código del idioma detectado.

  * `score`: valor flotante que indica la confianza en el resultado. La puntuación varía entre cero y uno, y una puntuación baja indica una confianza baja.

  * `isTranslationSupported`: Valor booleano que tiene el valor "true" si el idioma detectado es uno de los idiomas admitidos para la traducción de texto.

  * `isTransliterationSupported`: Valor booleano que tiene el valor "true" si el idioma detectado es uno de los idiomas admitidos para la transliteración.
  
  * `alternatives`: Matriz de otros idiomas posibles. Cada elemento de la matriz es otro objeto con las mismas propiedades enumeradas anteriormente: `language`, `score`, `isTranslationSupported` y `isTransliterationSupported`.

Un ejemplo de respuesta JSON es:

```json
[
  {
    "language": "de",
    "score": 0.92,
    "isTranslationSupported": true,
    "isTransliterationSupported": false,
    "alternatives": [
      {
        "language": "pt",
        "score": 0.23,
        "isTranslationSupported": true,
        "isTransliterationSupported": false
      },
      {
        "language": "sk",
        "score": 0.23,
        "isTranslationSupported": true,
        "isTransliterationSupported": false
      }
    ]
  }
]
```

## <a name="response-headers"></a>Encabezados de respuesta

<table width="100%">
  <th width="20%">Encabezados</th>
  <th>DESCRIPCIÓN</th>
  <tr>
    <td>X-RequestId</td>
    <td>Valor generado por el servicio para identificar la solicitud. Se usa para solucionar problemas.</td>
  </tr>
</table> 

## <a name="response-status-codes"></a>Códigos de estado de respuesta

A continuación se indican los códigos de estado HTTP posibles que devuelve una solicitud. 

<table width="100%">
  <th width="20%">Código de estado</th>
  <th>DESCRIPCIÓN</th>
  <tr>
    <td>200</td>
    <td>Correcto.</td>
  </tr>
  <tr>
    <td>400</td>
    <td>Uno de los parámetros de consulta falta o no es válido. Corrija los parámetros de la solicitud antes de volver a intentarlo.</td>
  </tr>
  <tr>
    <td>401</td>
    <td>No pudo autenticarse la solicitud. Compruebe que las credenciales que se especificaron sean correctas.</td>
  </tr>
  <tr>
    <td>403</td>
    <td>La solicitud no está autenticada. Compruebe los detalles del mensaje de error. Esto a menudo indica que todas las traducciones gratuitas que proporciona la suscripción de prueba se han agotado.</td>
  </tr>
  <tr>
    <td>429</td>
    <td>El servidor rechazó la solicitud porque el cliente ha superado los límites de solicitudes.</td>
  </tr>
  <tr>
    <td>500</td>
    <td>Se ha producido un error inesperado. Si el error continúa, notifíquelo con: fecha y hora del error, identificador de la solicitud del encabezado de respuesta `X-RequestId` e identificador de cliente del encabezado de solicitud `X-ClientTraceId`.</td>
  </tr>
  <tr>
    <td>503</td>
    <td>Servidor no disponible temporalmente. Vuelva a intentarlo. Si el error continúa, notifíquelo con: fecha y hora del error, identificador de la solicitud del encabezado de respuesta `X-RequestId` e identificador de cliente del encabezado de solicitud `X-ClientTraceId`.</td>
  </tr>
</table> 

Si se produce un error, la solicitud también devolverá una respuesta de error JSON. El código de error es un número de 6 dígitos que combina el código de estado HTTP de 3 dígitos y otro número de 3 dígitos que ayuda a categorizar aún más el error. Códigos de error comunes que pueden encontrarse en la [página de referencia de Translator Text API v3](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference#errors). 

## <a name="examples"></a>Ejemplos

En el ejemplo siguiente se muestra cómo recuperar los idiomas admitidos para la traducción de texto.

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/detect?api-version=3.0" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'What language is this text written in?'}]"
```

---
