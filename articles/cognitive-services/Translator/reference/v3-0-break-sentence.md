---
title: Método BreakSentence de Translator Text API.
titlesuffix: Azure Cognitive Services
description: Use el método BreakSentence de Translator Text API.
services: cognitive-services
author: v-pawal
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 02/01/2019
ms.author: v-jansko
ms.openlocfilehash: b2c0fbbfbcd2e3cf71ef85b9d06f2f805698bfd0
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65797432"
---
# <a name="translator-text-api-30-breaksentence"></a>Translator Text API 3.0: BreakSentence

Identifica el posicionamiento de los límites de las oraciones en un fragmento de texto.

## <a name="request-url"></a>URL de la solicitud

Envíe una solicitud `POST` a:

```HTTP
https://api.cognitive.microsofttranslator.com/breaksentence?api-version=3.0
```

## <a name="request-parameters"></a>Parámetros de solicitud

Los parámetros de solicitud que se pasaron en la cadena de consulta son:

<table width="100%">
  <th width="20%">Parámetro de consulta</th>
  <th>DESCRIPCIÓN</th>
  <tr>
    <td>api-version</td>
    <td>*parámetro de consulta obligatorio*.<br/>Versión de la API que el cliente solicitó. El valor debe ser `3.0`.</td>
  </tr>
  <tr>
    <td>language</td>
    <td>*Parámetro de consulta opcional*.<br/>Etiqueta de idioma que identifica el idioma del texto de entrada. Si no se especifica un código, se aplicará la detección automática del idioma.</td>
  </tr>
  <tr>
    <td>script</td>
    <td>*Parámetro de consulta opcional*.<br/>Etiqueta de script que identifica el script que usa el texto de entrada. Si no se especifica un script, se asumirá el script predeterminado del idioma.</td>
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

El cuerpo de la solicitud es una matriz JSON. Cada elemento de la matriz es un objeto JSON con una propiedad de cadena denominada `Text`. Los límites de la oración se calculan en función del valor de la propiedad `Text`. Un cuerpo de la solicitud de muestra con una sola pieza de texto tiene este aspecto:

```json
[
    { "Text": "How are you? I am fine. What did you do today?" }
]
```

Se aplican las siguientes limitaciones:

* La matriz puede tener como máximo 100 elementos.
* El valor de texto de un elemento de la matriz no puede superar los 10 000 caracteres, incluyendo los espacios.
* El texto completo incluido en la solicitud no puede superar los 50 000 caracteres, incluidos los espacios.
* Si se especifica el parámetro de consulta `language`, todos los elementos de la matriz deberán estar en el mismo idioma. De lo contrario, la autodetección de idioma se aplica a cada elemento de la matriz de forma independiente.

## <a name="response-body"></a>Response body

Una respuesta correcta es una matriz JSON con un resultado para cada cadena en la matriz de entrada. Un objeto del resultado incluye las siguientes propiedades:

  * `sentLen`: es una matriz de enteros que representan las longitudes de las oraciones en el elemento de texto. La longitud de la matriz es el número de oraciones y los valores son la longitud de cada oración. 

  * `detectedLanguage`: objeto que describe el idioma detectado mediante las siguientes propiedades:

     * `language`: Código del idioma detectado.

     * `score`: valor flotante que indica la confianza en el resultado. La puntuación varía entre cero y uno, y una puntuación baja indica una confianza baja.
     
    Tenga en cuenta que la propiedad `detectedLanguage` solo está presente en el objeto de resultado cuando se solicita la detección automática del idioma.

Un ejemplo de respuesta JSON es:

```json
[
  {
    "sentenceLengths": [ 13, 11, 22 ]
    "detectedLanguage": {
      "language": "en",
      "score": 401
    },
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

En el siguiente ejemplo se muestra cómo obtener límites de oraciones para una sola oración. El servicio se encarga de detectar automáticamente el idioma de la frase.

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/breaksentence?api-version=3.0" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'How are you? I am fine. What did you do today?'}]"
```

---

