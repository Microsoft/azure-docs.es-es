---
title: Método BreakSentence de Traductor
titleSuffix: Azure Cognitive Services
description: El método BreakSentence de Traductor identifica el posicionamiento de los límites de las oraciones en un fragmento de texto.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 08/06/2020
ms.author: swmachan
ms.openlocfilehash: 340b0ed02821fb98f271539ac39e0ccad8581082
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87904010"
---
# <a name="translator-30-breaksentence"></a>Traductor 3.0: BreakSentence

Identifica el posicionamiento de los límites de las oraciones en un fragmento de texto.

## <a name="request-url"></a>URL de la solicitud

Envíe una solicitud `POST` a:

```HTTP
https://api.cognitive.microsofttranslator.com/breaksentence?api-version=3.0
```

## <a name="request-parameters"></a>Parámetros de solicitud

Los parámetros de solicitud que se pasaron en la cadena de consulta son:

| Parámetro de consulta | Descripción |
| -------| ----------- |
| api-version <img width=200/>   | **parámetro de consulta obligatorio**.<br/>Versión de la API que el cliente solicitó. El valor debe ser `3.0`. |
| language | **Parámetro de consulta opcional**.<br/>Etiqueta de idioma que identifica el idioma del texto de entrada. Si no se especifica un código, se aplicará la detección automática del idioma. |
| script    | **Parámetro de consulta opcional**.<br/>Etiqueta de script que identifica el script que usa el texto de entrada. Si no se especifica un script, se asumirá el script predeterminado del idioma.  | 

Los encabezados de solicitud incluyen lo siguiente:

| encabezados | Descripción |
| ------- | ----------- |
| Encabezados de autenticación <img width=200/>  | **Encabezado de solicitud obligatorio**.<br/>Consulte las <a href="https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference#authentication">opciones disponibles para la autenticación</a>. |
| Content-Type | **Encabezado de solicitud obligatorio**.<br/>Especifica el tipo de contenido de la carga. Los valores posibles son: `application/json`. |
| Content-Length    | **Encabezado de solicitud obligatorio**.<br/>Longitud del cuerpo de la solicitud.  | 
| X-ClientTraceId   | **Opcional**.<br/>GUID generado por el cliente para identificar de forma única la solicitud. Puede omitir este encabezado si incluye el id. de seguimiento en la cadena de la consulta mediante un parámetro de consulta denominado `ClientTraceId`.  | 

## <a name="request-body"></a>Cuerpo de la solicitud

El cuerpo de la solicitud es una matriz JSON. Cada elemento de la matriz es un objeto JSON con una propiedad de cadena denominada `Text`. Los límites de la oración se calculan en función del valor de la propiedad `Text`. Un cuerpo de la solicitud de muestra con una sola pieza de texto tiene este aspecto:

```json
[
    { "Text": "How are you? I am fine. What did you do today?" }
]
```

Se aplican las siguientes limitaciones:

* La matriz puede tener como máximo 100 elementos.
* El valor de texto de un elemento de la matriz no puede superar los 50 000 caracteres, incluidos los espacios.
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
    "sentLen": [ 13, 11, 22 ]
    "detectedLanguage": {
      "language": "en",
      "score": 401
    },
  }
]
```

## <a name="response-headers"></a>Encabezados de respuesta

<table width="100%">
  <th width="20%">encabezados</th>
  <th>Descripción</th>
  <tr>
    <td>X-RequestId</td>
    <td>Valor generado por el servicio para identificar la solicitud. Se usa para solucionar problemas.</td>
  </tr>
</table> 

## <a name="response-status-codes"></a>Códigos de estado de respuesta

A continuación se indican los códigos de estado HTTP posibles que devuelve una solicitud. 

<table width="100%">
  <th width="20%">Código de estado</th>
  <th>Descripción</th>
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
    <td>El servidor rechazó la solicitud porque el cliente superó los límites de solicitudes.</td>
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

Si se produce un error, la solicitud también devolverá una respuesta de error JSON. El código de error es un número de 6 dígitos que combina el código de estado HTTP de 3 dígitos y otro número de 3 dígitos que ayuda a categorizar aún más el error. En la [página de referencia de Traductor v3](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference#errors) pueden encontrarse los códigos de error comunes. 

## <a name="examples"></a>Ejemplos

En el siguiente ejemplo se muestra cómo obtener límites de oraciones para una sola oración. El servicio se encarga de detectar automáticamente el idioma de la frase.

```curl
curl -X POST "https://api.cognitive.microsofttranslator.com/breaksentence?api-version=3.0" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'How are you? I am fine. What did you do today?'}]"
```

