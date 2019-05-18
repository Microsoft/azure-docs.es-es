---
title: Método de idiomas de Translator Text API
titlesuffix: Azure Cognitive Services
description: Use el método de idiomas de Translator Text API.
services: cognitive-services
author: v-pawal
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 02/01/2019
ms.author: v-jansko
ms.openlocfilehash: 6e0342d876db424454526637322d67d55c0432a8
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65797295"
---
# <a name="translator-text-api-30-languages"></a>Translator Text API 3.0: Idiomas

Obtiene el conjunto de idiomas admitidos actualmente por otras operaciones de Translator Text API. 

## <a name="request-url"></a>URL de la solicitud

Envíe una solicitud `GET` a:
```HTTP
https://api.cognitive.microsofttranslator.com/languages?api-version=3.0
```

## <a name="request-parameters"></a>Parámetros de solicitud

Los parámetros de solicitud que se pasaron en la cadena de consulta son:

<table width="100%">
  <th width="20%">Parámetro de consulta</th>
  <th>DESCRIPCIÓN</th>
  <tr>
    <td>api-version</td>
    <td><em>Parámetro obligatorio</em>.<br/>Versión de la API que el cliente solicitó. El valor debe ser `3.0`.</td>
  </tr>
  <tr>
    <td>ámbito</td>
    <td>*Parámetro opcional*.<br/>Lista de nombres separados por comas que definen el grupo de idiomas que se devolverá. Los nombres de grupo permitidos son: `translation`, `transliteration` y `dictionary`. Si no se proporciona ningún ámbito, se devuelven todos los grupos, lo que es equivalente a pasar `scope=translation,transliteration,dictionary`. Para decidir qué conjunto de idiomas admitidos es adecuado para su escenario, vea la descripción del [objeto de respuesta](#response-body).</td>
  </tr>
</table> 

Los encabezados de solicitud son:

<table width="100%">
  <th width="20%">Encabezados</th>
  <th>DESCRIPCIÓN</th>
  <tr>
    <td>Accept-Language</td>
    <td>*Encabezado de solicitud opcional*.<br/>Idioma que se usará para las cadenas de la interfaz de usuario. Algunos de los campos de la respuesta son nombres de idiomas o regiones. Utilice este parámetro para definir el idioma en que se devolverán los nombres. El idioma se especifica al proporcionar una etiqueta de idioma BCP 47 con el formato correcto. Por ejemplo, utilice el valor `fr` para solicitar nombres en francés o utilice el valor `zh-Hant` para solicitar nombres en chino tradicional.<br/>Los nombres se proporcionan en inglés cuando no se especifica un idioma de destino o cuando la localización no está disponible.
    </td>
  </tr>
  <tr>
    <td>X-ClientTraceId</td>
    <td>*Encabezado de solicitud opcional*.<br/>GUI generado por el cliente para identificar de forma única la solicitud.</td>
  </tr>
</table> 

No es obligatorio que la autenticación obtenga recursos de idioma.

## <a name="response-body"></a>Response body

Un cliente usa el parámetro de consulta `scope` para definir los grupos de idiomas en los que está interesado.

* `scope=translation` proporciona los idiomas admitidos para traducir texto de un idioma a otro;

* `scope=transliteration` proporciona funcionalidades para convertir texto en un idioma de un script a otro;

* `scope=dictionary` proporciona pares de idiomas para los que las operaciones de `Dictionary` devuelven datos.

Un cliente puede recuperar varios grupos simultáneamente mediante la especificación de una lista de nombres separados por comas. Por ejemplo, `scope=translation,transliteration,dictionary` devolvería los idiomas admitidos para todos los grupos.

Una respuesta correcta es un objeto JSON con una propiedad para cada grupo solicitado:

```json
{
    "translation": {
        //... set of languages supported to translate text (scope=translation)
    },
    "transliteration": {
        //... set of languages supported to convert between scripts (scope=transliteration)
    },
    "dictionary": {
        //... set of languages supported for alternative translations and examples (scope=dictionary)
    }
}
```

El valor de cada propiedad es el siguiente.

* Propiedad `translation`

  El valor de la propiedad `translation` es un diccionario de pares (clave, valor). Cada clave es una etiqueta de idioma BCP 47. Una clave identifica un idioma de origen o destino para la traducción de un texto. El valor asociado a la clave es un objeto JSON con propiedades que describen el idioma:

  * `name`: nombre para mostrar del idioma en la configuración regional solicitada a través del encabezado `Accept-Language`.

  * `nativeName`: nombre para mostrar del idioma en la configuración regional nativa del idioma.

  * `dir`: direccionalidad, que es `rtl` para los idiomas de derecha a izquierda o `ltr` para los idiomas de izquierda a derecha.

  Ejemplo:
          
  ```json
  {
    "translation": {
      ...
      "fr": {
        "name": "French",
        "nativeName": "Français",
        "dir": "ltr"
      },
      ...
    }
  }
  ```

* Propiedad `transliteration`

  El valor de la propiedad `transliteration` es un diccionario de pares (clave, valor). Cada clave es una etiqueta de idioma BCP 47. Una clave identifica un idioma para el que se puede convertir texto de un script a otro. El valor asociado a la clave es un objeto JSON con propiedades que describen el idioma y sus scripts admitidos:

  * `name`: nombre para mostrar del idioma en la configuración regional solicitada a través del encabezado `Accept-Language`.

  * `nativeName`: nombre para mostrar del idioma en la configuración regional nativa del idioma.

  * `scripts`: lista de scripts de origen de la conversión. Cada elemento de la lista `scripts` tiene las siguientes propiedades:

    * `code`: código de identificación del script.

    * `name`: nombre para mostrar del script en la configuración regional solicitada a través del encabezado `Accept-Language`.

    * `nativeName`: nombre para mostrar del idioma en la configuración regional nativa del idioma.

    * `dir`: direccionalidad, que es `rtl` para los idiomas de derecha a izquierda o `ltr` para los idiomas de izquierda a derecha.

    * `toScripts`: lista de scripts disponibles como destino para la conversión de texto. Cada elemento de la lista `toScripts` tiene las propiedades `code`, `name`, `nativeName` y `dir` descritas anteriormente.

  Ejemplo:

  ```json
  {
    "transliteration": {
      ...
      "ja": {
        "name": "Japanese",
        "nativeName": "日本語",
        "scripts": [
          {
            "code": "Jpan",
            "name": "Japanese",
            "nativeName": "日本語",
            "dir": "ltr",
            "toScripts": [
              {
                "code": "Latn",
                "name": "Latin",
                "nativeName": "ラテン語",
                "dir": "ltr"
              }
            ]
          },
          {
            "code": "Latn",
            "name": "Latin",
            "nativeName": "ラテン語",
            "dir": "ltr",
            "toScripts": [
              {
                "code": "Jpan",
                "name": "Japanese",
                "nativeName": "日本語",
                "dir": "ltr"
              }
            ]
          }
        ]
      },
      ...
    }
  }
  ```

* Propiedad `dictionary`

  El valor de la propiedad `dictionary` es un diccionario de pares (clave, valor). Cada clave es una etiqueta de idioma BCP 47. La clave identifica un idioma para el que existen traducciones alternativas y traducciones inversas disponibles. El valor es un objeto JSON que describe los idiomas de origen y destino con las traducciones disponibles:

  * `name`: nombre para mostrar del idioma de origen en la configuración regional solicitada a través del encabezado `Accept-Language`.

  * `nativeName`: nombre para mostrar del idioma en la configuración regional nativa del idioma.

  * `dir`: direccionalidad, que es `rtl` para los idiomas de derecha a izquierda o `ltr` para los idiomas de izquierda a derecha.

  * `translations`: lista de idiomas con traducciones alternativas y ejemplos para la consulta expresada en el idioma de origen. Cada elemento de la lista `translations` tiene las siguientes propiedades:

    * `name`: nombre para mostrar del idioma de destino en la configuración regional solicitada a través del encabezado `Accept-Language`.

    * `nativeName`: nombre para mostrar del idioma de destino en la configuración regional nativa del idioma de destino.

    * `dir`: direccionalidad, que es `rtl` para los idiomas de derecha a izquierda o `ltr` para los idiomas de izquierda a derecha.
    
    * `code`: código de idioma que identifica el idioma de destino.

  Ejemplo:

  ```json
  "es": {
    "name": "Spanish",
    "nativeName": "Español",
    "dir": "ltr",
    "translations": [
      {
        "name": "English",
        "nativeName": "English",
        "dir": "ltr",
        "code": "en"
      }
    ]
  },
  ```

La estructura del objeto de respuesta no cambiará sin un cambio en la versión de la API. Para la misma versión de la API, la lista de idiomas disponibles puede cambiar con el tiempo porque Microsoft Translator amplía continuamente la lista de idiomas admitidos en sus servicios.

La lista de idiomas admitidos no cambiará con frecuencia. Para ahorrar ancho de banda de red y mejorar la capacidad de respuesta, una aplicación cliente debe considerar el almacenamiento en caché de los recursos de idioma y la correspondiente etiqueta de entidad (`ETag`). A continuación, la aplicación cliente puede consultar periódicamente (por ejemplo, una vez cada 24 horas) al servicio para capturar el conjunto más reciente de idiomas admitidos. Pasar el valor `ETag` actual en un campo de encabezado `If-None-Match` permitirá al servicio optimizar la respuesta. Si no se ha modificado el recurso, el servicio devolverá el código de estado 304 y un cuerpo de respuesta vacío.

## <a name="response-headers"></a>Encabezados de respuesta

<table width="100%">
  <th width="20%">Encabezados</th>
  <th>DESCRIPCIÓN</th>
  <tr>
    <td>ETag</td>
    <td>Valor actual de la etiqueta de entidad para los grupos de idiomas admitidos solicitados. Para mejorar la eficacia de solicitudes posteriores, el cliente puede enviar el valor `ETag` en un campo de encabezado `If-None-Match`.
    </td>
  </tr>
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
    <td>304</td>
    <td>El recurso no se ha modificado desde la versión especificada por los encabezados de solicitud `If-None-Match`.</td>
  </tr>
  <tr>
    <td>400</td>
    <td>Uno de los parámetros de consulta falta o no es válido. Corrija los parámetros de la solicitud antes de volver a intentarlo.</td>
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
curl "https://api.cognitive.microsofttranslator.com/languages?api-version=3.0&scope=translation"
```

---
