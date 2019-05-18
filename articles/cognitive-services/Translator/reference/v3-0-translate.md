---
title: Método de traducción de Translator Text API
titleSuffix: Azure Cognitive Services
description: Use el método de traducción de Translator Text API.
services: cognitive-services
author: v-pawal
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 02/01/2019
ms.author: v-jansko
ms.openlocfilehash: e7ab2eba54aafda6bb00696939bf9bc32bb627ca
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65797008"
---
# <a name="translator-text-api-30-translate"></a>Translator Text API 3.0: Trasladar

Traduce el texto.

## <a name="request-url"></a>URL de la solicitud

Envíe una solicitud `POST` a:

```HTTP
https://api.cognitive.microsofttranslator.com/translate?api-version=3.0
```

## <a name="request-parameters"></a>Parámetros de solicitud

Los parámetros de solicitud que se pasaron en la cadena de consulta son:

<table width="100%">
  <th width="20%">Parámetro de consulta</th>
  <th>DESCRIPCIÓN</th>
  <tr>
    <td>api-version</td>
    <td><em>Parámetro obligatorio</em>.<br/>Versión de la API que el cliente solicitó. El valor debe ser <code>3.0</code>.</td>
  </tr>
  <tr>
    <td>from</td>
    <td><em>Parámetro opcional</em>.<br/>Especifica el idioma del texto de entrada. Busque los idiomas que están disponibles desde los que realizar la traducción mediante la busca de <a href="./v3-0-languages.md">idiomas admitidos</a> con el ámbito <code>translation</code>. Si no se ha especificado el parámetro <code>from</code>, se aplica la detección de idioma automática para determinar el idioma de origen.</td>
  </tr>
  <tr>
    <td>to</td>
    <td><em>Parámetro obligatorio</em>.<br/>Especifica el idioma del texto de salida. El idioma de destino debe ser uno de los <a href="./v3-0-languages.md">idiomas admitidos</a> que están incluidos en el ámbito <code>translation</code>. Por ejemplo, utilice <code>to=de</code> para traducir al alemán.<br/>Es posible traducir a varios idiomas simultáneamente mediante la repetición del parámetro en la cadena de consulta. Por ejemplo, utilice <code>to=de&to=it</code> para traducir al alemán e italiano.</td>
  </tr>
  <tr>
    <td>textType</td>
    <td><em>Parámetro opcional</em>.<br/>Define si el texto que se está traduciendo es texto sin formato o texto HTML. El código HTML debe ser un elemento completo y bien formado. Los valores posibles son <code>plain</code> (valor predeterminado) o <code>html</code>.</td>
  </tr>
  <tr>
    <td>category</td>
    <td><em>Parámetro opcional</em>.<br/>Una cadena que especifica la categoría (dominio) de la traducción. Este parámetro se utiliza para obtener las traducciones de un sistema personalizado creado con <a href="../customization.md">Custom Translator</a>. Añada el identificador de categoría de su proyecto de Traductor personalizado a este parámetro para usar el sistema personalizado implementado. El valor predeterminado es <code>general</code>.</td>
  </tr>
  <tr>
    <td>profanityAction</td>
    <td><em>Parámetro opcional</em>.<br/>Especifica cómo se deben tratar las palabras soeces en las traducciones. Los valores posibles son <code>NoAction</code> (valor predeterminado), <code>Marked</code> o <code>Deleted</code>. Para entender las maneras de tratar las palabras soeces, consulte <a href="#handle-profanity">Control de palabras soeces</a>.</td>
  </tr>
  <tr>
    <td>profanityMarker</td>
    <td><em>Parámetro opcional</em>.<br/>Especifica cómo deben marcarse las palabras soeces en las traducciones. Los valores posibles son <code>Asterisk</code> (valor predeterminado) o <code>Tag</code>. Para entender las maneras de tratar las palabras soeces, consulte <a href="#handle-profanity">Control de palabras soeces</a>.</td>
  </tr>
  <tr>
    <td>includeAlignment</td>
    <td><em>Parámetro opcional</em>.<br/>Especifica si se debe incluir la proyección de la alineación del texto de origen en el texto traducido. Los valores posibles son <code>true</code> o <code>false</code> (valor predeterminado). </td>
  </tr>
  <tr>
    <td>includeSentenceLength</td>
    <td><em>Parámetro opcional</em>.<br/>Especifica si se deben incluir los límites de oraciones del texto de entrada y el texto traducido. Los valores posibles son <code>true</code> o <code>false</code> (valor predeterminado).</td>
  </tr>
  <tr>
    <td>suggestedFrom</td>
    <td><em>Parámetro opcional</em>.<br/>Especifica un idioma de reserva si no se puede identificar el idioma del texto de entrada. La detección de idioma automática se aplica cuando se omite el parámetro <code>from</code>. Si se produce un error en la detección, se asume el idioma <code>suggestedFrom</code>.</td>
  </tr>
  <tr>
    <td>fromScript</td>
    <td><em>Parámetro opcional</em>.<br/>Especifica el script del texto de entrada.</td>
  </tr>
  <tr>
    <td>toScript</td>
    <td><em>Parámetro opcional</em>.<br/>Especifica el script del texto traducido.</td>
  </tr>
  <tr>
    <td>allowFallback</td>
    <td><em>Parámetro opcional</em>.<br/>Especifica que el servicio puede recurrir a un sistema general cuando no existe un sistema personalizado. Los valores posibles son <code>true</code> (valor predeterminado) o <code>false</code>.<br/><br/><code>allowFallback=false</code> especifica que la traducción solo debe usar sistemas entrenados para la <code>category</code> especificada por la solicitud. Si una traducción del idioma X al idioma Y requiere de encadenamiento a través de un idioma puente E, entonces todos los sistemas de la cadena (X -> E y E -> Y) deberán estar personalizados y tener la misma categoría. Si no se encuentra ningún sistema con la categoría específica, la solicitud devolverá un código de estado de 400. <code>allowFallback=true</code> especifica que el servicio puede recurrir a un sistema general cuando no existe un sistema personalizado.
</td>
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
    <td><em>Encabezado de solicitud obligatorio</em>.<br/>Especifica el tipo de contenido de la carga. Los valores posibles son: <code>application/json</code>.</td>
  </tr>
  <tr>
    <td>Content-Length</td>
    <td><em>Encabezado de solicitud obligatorio</em>.<br/>Longitud del cuerpo de la solicitud.</td>
  </tr>
  <tr>
    <td>X-ClientTraceId</td>
    <td><em>Opcional</em>.<br/>GUID generado por el cliente para identificar de forma única la solicitud. Puede omitir este encabezado si incluye el id. de seguimiento en la cadena de la consulta mediante un parámetro de consulta denominado <code>ClientTraceId</code>.</td>
  </tr>
</table> 

## <a name="request-body"></a>Cuerpo de la solicitud

El cuerpo de la solicitud es una matriz JSON. Cada elemento de la matriz es un objeto JSON con una propiedad de cadena denominada `Text`, que representa la cadena que se va a traducir.

```json
[
    {"Text":"I would really like to drive your car around the block a few times."}
]
```

Se aplican las siguientes limitaciones:

* La matriz puede tener como máximo 100 elementos.
* El texto completo incluido en la solicitud no puede superar los 5000 caracteres, incluidos los espacios.

## <a name="response-body"></a>Response body

Una respuesta correcta es una matriz JSON con un resultado para cada cadena en la matriz de entrada. Un objeto del resultado incluye las siguientes propiedades:

  * `detectedLanguage`: objeto que describe el idioma detectado mediante las siguientes propiedades:

      * `language`: cadena que representa el código del idioma detectado.

      * `score`: valor flotante que indica la confianza en el resultado. La puntuación varía entre cero y uno, y una puntuación baja indica una confianza baja.

    La propiedad `detectedLanguage` solo está presente en el objeto de resultado cuando se solicita la detección de idioma automática.

  * `translations`: matriz de resultados de la traducción. El tamaño de la matriz coincide con el número de idiomas de destino especificados a través del parámetro de consulta `to`. Cada elemento de la matriz incluye:

    * `to`: cadena que representa el código del idioma de destino.

    * `text`: cadena que proporciona el texto traducido.

    * `transliteration`: objeto que proporciona el texto traducido en el script especificado por el parámetro `toScript`.

      * `script`: cadena que especifica el script de destino.   

      * `text`: cadena que proporciona el texto traducido en el script de destino.

    El objeto `transliteration` no se incluye si no se produce la transliteración.

    * `alignment`: objeto con una propiedad de cadena única denominada `proj`, que asigna el texto de entrada al texto traducido. La información de alineación solo se proporciona cuando el parámetro de solicitud `includeAlignment` es `true`. La alineación se devuelve como un valor de cadena del siguiente formato: `[[SourceTextStartIndex]:[SourceTextEndIndex]–[TgtTextStartIndex]:[TgtTextEndIndex]]`.  Los dos puntos separan el índice inicial y final, el guión separa los idiomas y el espacio separa las palabras. Una palabra se puede alinear con ninguna, una o varias palabras de otro idioma y las palabras alineadas pueden ser no contiguas. Si no existe información de alineación disponible, el elemento alignment estará vacío. Consulte [Obtener información de alineación](#obtain-alignment-information) para ver un ejemplo y restricciones.

    * `sentLen`: objeto que devuelve los límites de las oraciones en los textos de entrada y salida.

      * `srcSentLen`: matriz de enteros que representan las longitudes de las oraciones en el texto de entrada. La longitud de la matriz es el número de oraciones y los valores son la longitud de cada oración.

      * `transSentLen`:  matriz de enteros que representan las longitudes de las oraciones en el texto traducido. La longitud de la matriz es el número de oraciones y los valores son la longitud de cada oración.

    Los límites de oraciones solo se incluyen cuando el parámetro de solicitud `includeSentenceLength` es `true`.

  * `sourceText`: objeto con una propiedad de cadena única denominada `text`, que proporciona el texto de entrada en el script predeterminado del idioma de origen. La propiedad `sourceText` solo está presente cuando la entrada se expresa en un script que no es el habitual del idioma. Por ejemplo, si la entrada era árabe escrito en alfabeto latino, `sourceText.text` será el mismo texto árabe convertido al alfabeto árabe.

En la sección de [ejemplos](#examples) se proporcionan ejemplos de respuestas JSON.

## <a name="response-headers"></a>Encabezados de respuesta

<table width="100%">
  <th width="20%">Encabezados</th>
  <th>DESCRIPCIÓN</th>
    <tr>
    <td>X-RequestId</td>
    <td>Valor generado por el servicio para identificar la solicitud. Se usa para solucionar problemas.</td>
  </tr>
  <tr>
    <td>Sistema X-MT</td>
    <td>Especifica el tipo de sistema que se usó para la traducción en cada idioma de 'destino' solicitado para la traducción. El valor es una lista de cadenas separadas por comas. Cada cadena indica un tipo:<br/><ul><li>Custom: la solicitud incluye un sistema personalizado y se usó al menos un sistema personalizado durante la traducción.</li><li>Team: todas las demás solicitudes.</li></td>
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
    <td>408</td>
    <td>No se pudo satisfacer la solicitud porque falta un recurso. Compruebe los detalles del mensaje de error. Cuando se usa una <code>category</code> personalizada, a menudo, esto indica que el sistema de traducción personalizada todavía no está disponible para atender las solicitudes. Se debe reintentar la solicitud tras un período de espera (por ejemplo, 1 minuto).</td>
  </tr>
  <tr>
    <td>429</td>
    <td>El servidor rechazó la solicitud porque el cliente ha superado los límites de solicitudes.</td>
  </tr>
  <tr>
    <td>500</td>
    <td>Se ha producido un error inesperado. Si el error continúa, notifíquelo con: fecha y hora del error, identificador de la solicitud del encabezado de respuesta <code>X-RequestId</code> e identificador de cliente del encabezado de solicitud <code>X-ClientTraceId</code>.</td>
  </tr>
  <tr>
    <td>503</td>
    <td>Servidor no disponible temporalmente. Vuelva a intentarlo. Si el error continúa, notifíquelo con: fecha y hora del error, identificador de la solicitud del encabezado de respuesta <code>X-RequestId</code> e identificador de cliente del encabezado de solicitud <code>X-ClientTraceId</code>.</td>
  </tr>
</table> 

Si se produce un error, la solicitud también devolverá una respuesta de error JSON. El código de error es un número de 6 dígitos que combina el código de estado HTTP de 3 dígitos y otro número de 3 dígitos que ayuda a categorizar aún más el error. Códigos de error comunes que pueden encontrarse en la [página de referencia de Translator Text API v3](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference#errors). 

## <a name="examples"></a>Ejemplos

### <a name="translate-a-single-input"></a>Traducción de una única entrada

En este ejemplo se muestra cómo traducir una única oración del inglés al chino simplificado.

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=zh-Hans" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'Hello, what is your name?'}]"
```

---

El cuerpo de la respuesta es:

```
[
    {
        "translations":[
            {"text":"你好, 你叫什么名字？","to":"zh-Hans"}
        ]
    }
]
```

La matriz `translations` incluye un elemento, que proporciona la traducción de la parte única de texto en la entrada.

### <a name="translate-a-single-input-with-language-auto-detection"></a>Traducción de una única entrada con la detección de idioma automática

En este ejemplo se muestra cómo traducir una única oración del inglés al chino simplificado. La solicitud no especifica el idioma de entrada. En su lugar, se usa la detección automática del idioma de origen.

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&to=zh-Hans" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'Hello, what is your name?'}]"
```

---

El cuerpo de la respuesta es:

```
[
    {
        "detectedLanguage": {"language": "en", "score": 1.0},
        "translations":[
            {"text": "你好, 你叫什么名字？", "to": "zh-Hans"}
        ]
    }
]
```
La respuesta es similar a la respuesta del ejemplo anterior. Dado que se solicitó la detección de idioma automática, la respuesta también incluye información sobre el idioma detectado del texto de entrada. 

### <a name="translate-with-transliteration"></a>Traducción con transliteración

Vamos a ampliar el ejemplo anterior mediante la adición de la transliteración. La siguiente solicitud pide una traducción de chino escrita en alfabeto latino.

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&to=zh-Hans&toScript=Latn" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'Hello, what is your name?'}]"
```

---

El cuerpo de la respuesta es:

```
[
    {
        "detectedLanguage":{"language":"en","score":1.0},
        "translations":[
            {
                "text":"你好, 你叫什么名字？",
                "transliteration":{"script":"Latn", "text":"nǐ hǎo , nǐ jiào shén me míng zì ？"},
                "to":"zh-Hans"
            }
        ]
    }
]
```

El resultado de la traducción incluye ahora una propiedad `transliteration`, que proporciona el texto traducido con caracteres latinos.

### <a name="translate-multiple-pieces-of-text"></a>Traducción de varios fragmentos de texto

Traducir varias cadenas a la vez es simplemente cuestión de especificar una matriz de cadenas en el cuerpo de la solicitud.

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=zh-Hans" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'Hello, what is your name?'}, {'Text':'I am fine, thank you.'}]"
```

---

El cuerpo de la respuesta es:

```
[
    {
        "translations":[
            {"text":"你好, 你叫什么名字？","to":"zh-Hans"}
        ]
    },            
    {
        "translations":[
            {"text":"我很好，谢谢你。","to":"zh-Hans"}
        ]
    }
]
```

### <a name="translate-to-multiple-languages"></a>Traducción a varios idiomas

En este ejemplo se muestra cómo traducir la misma entrada a diferentes idiomas en una sola solicitud.

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=zh-Hans&to=de" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'Hello, what is your name?'}]"
```

---

El cuerpo de la respuesta es:

```
[
    {
        "translations":[
            {"text":"你好, 你叫什么名字？","to":"zh-Hans"},
            {"text":"Hallo, was ist dein Name?","to":"de"}
        ]
    }
]
```

### <a name="handle-profanity"></a>Control de palabras soeces

Normalmente, el servicio de Traductor conserva en la traducción las palabras soeces que están presentes en el origen. El grado de palabras soeces y el contexto que las convierte en ello difieren entre culturas y, como resultado, el grado de palabras soeces del idioma de destino puede aumentar o reducirse.

Si quiere evitar obtener palabras soeces en la traducción, independientemente de su presencia en el texto de origen, puede usar la opción de filtrado de palabras soeces. La opción permite elegir si quiere que se eliminen las palabras soeces, si quiere marcarlas con etiquetas adecuadas (lo que le ofrece la opción de agregar su propio postprocesamiento) o si no quiere que se realice ninguna acción. Los valores aceptados de `ProfanityAction` son `Deleted`, `Marked` y `NoAction` (valor predeterminado).

<table width="100%">
  <th width="20%">ProfanityAction</th>
  <th>.</th>
  <tr>
    <td><code>NoAction</code></td>
    <td>Este es el comportamiento predeterminado. Las palabras soeces pasarán del origen al destino.<br/><br/>
    <strong>Ejemplo de origen (japonés)</strong>: 彼はジャッカスです。<br/>
    <strong>Ejemplo de traducción (español)</strong>: Es un idiota.
    </td>
  </tr>
  <tr>
    <td><code>Deleted</code></td>
    <td>Las palabras soeces se quitarán de la salida sin reemplazo.<br/><br/>
    <strong>Ejemplo de origen (japonés)</strong>: 彼はジャッカスです。<br/>
    <strong>Ejemplo de traducción (español)</strong>: Es un.
    </td>
  </tr>
  <tr>
    <td><code>Marked</code></td>
    <td>Las palabras soeces se reemplazan por un marcador en la salida. El marcador depende del parámetro <code>ProfanityMarker</code>.<br/><br/>
En el caso de <code>ProfanityMarker=Asterisk</code>, las palabras soeces se reemplazan por <code>***</code>:<br/>
    <strong>Ejemplo de origen (japonés)</strong>: 彼はジャッカスです。<br/>
    <strong>Ejemplo de traducción (español)</strong>: Es un \*\*\*.<br/><br/>
En el caso de <code>ProfanityMarker=Tag</code>, las palabras soeces aparecerán rodeadas por las etiquetas XML &lt;profanity&gt; y &lt;/profanity&gt;:<br/>
    <strong>Ejemplo de origen (japonés)</strong>: 彼はジャッカスです。<br/>
    <strong>Ejemplo de traducción (español)</strong>: Es un &lt;profanity&gt;estúpido&lt;/profanity&gt;.
  </tr>
</table> 

Por ejemplo:

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=de&profanityAction=Marked" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'This is a freaking good idea.'}]"
```

---

Esto devuelve:

```
[
    {
        "translations":[
            {"text":"Das ist eine *** gute Idee.","to":"de"}
        ]
    }
]
```

Comparar con:

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=de&profanityAction=Marked&profanityMarker=Tag" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'This is a freaking good idea.'}]"
```

---

La última solicitud devuelve:

```
[
    {
        "translations":[
            {"text":"Das ist eine <profanity>verdammt</profanity> gute Idee.","to":"de"}
        ]
    }
]
```

### <a name="translate-content-with-markup-and-decide-whats-translated"></a>Traducción del contenido con marcado y decisión de qué traducir

Es común traducir el contenido que incluye marcado como contenido de una página HTML o de un documento XML. Incluya el parámetro de consulta `textType=html` al traducir contenido con etiquetas. Además, a veces resulta útil para excluir contenido específico de la traducción. Puede utilizar el atributo `class=notranslate` para especificar contenido que debería permanecer en el idioma original. En el ejemplo siguiente, el contenido del primer elemento `div` no se traducirá, mientras que el del segundo elemento `div`, sí.

```
<div class="notranslate">This will not be translated.</div>
<div>This will be translated. </div>
```

A continuación, se muestra una solicitud de ejemplo para ilustrar.

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=zh-Hans&textType=html" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'<div class=\"notranslate\">This will not be translated.</div><div>This will be translated.</div>'}]"
```

---

La respuesta es:

```
[
    {
        "translations":[
            {"text":"<div class=\"notranslate\">This will not be translated.</div><div>这将被翻译。</div>","to":"zh-Hans"}
        ]
    }
]
```

### <a name="obtain-alignment-information"></a>Obtención de información de alineación

Para recibir información de alineación, especifique `includeAlignment=true` en la cadena de consulta.

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=fr&includeAlignment=true" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'The answer lies in machine translation.'}]"
```

---

La respuesta es:

```
[
    {
        "translations":[
            {
                "text":"La réponse se trouve dans la traduction automatique.",
                "to":"fr",
                "alignment":{"proj":"0:2-0:1 4:9-3:9 11:14-11:19 16:17-21:24 19:25-40:50 27:37-29:38 38:38-51:51"}
            }
        ]
    }
]
```

La información de alineación empieza por `0:2-0:1`, lo que significa que los tres primeros caracteres del texto de origen (`The`) se asignan a los dos primeros caracteres del texto traducido (`La`).

Tenga en cuenta las restricciones que se indican a continuación:

* La alineación solo se devuelve para un subconjunto de pares de idiomas:
  - de inglés a cualquier otro idioma;
  - de cualquier otro idioma a inglés, excepto de chino simplificado, chino tradicional y letón a inglés;
  - de japonés a coreano, o viceversa.
* No recibirá alineación si la oración es una traducción preestablecida. Un ejemplo de traducción preestablecida es "Esto es una prueba", "Te quiero" y otras frases que se usan con mucha frecuencia.

### <a name="obtain-sentence-boundaries"></a>Obtención de los límites de oración

Para recibir información acerca de la longitud de la oración del texto de origen y del texto traducido, especifique `includeSentenceLength=true` en la cadena de consulta.

# <a name="curltabcurl"></a>[curl](#tab/curl)

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=fr&includeSentenceLength=true" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'The answer lies in machine translation. The best machine translation technology cannot always provide translations tailored to a site or users like a human. Simply copy and paste a code snippet anywhere.'}]"
```

---

La respuesta es:

```
[
    {
        "translations":[
            {
                "text":"La réponse se trouve dans la traduction automatique. La meilleure technologie de traduction automatique ne peut pas toujours fournir des traductions adaptées à un site ou des utilisateurs comme un être humain. Il suffit de copier et coller un extrait de code n’importe où.",
                "to":"fr",
                "sentLen":{"srcSentLen":[40,117,46],"transSentLen":[53,157,62]}
            }
        ]
    }
]
```

### <a name="translate-with-dynamic-dictionary"></a>Traducción con diccionario dinámico

Si ya conoce la traducción que quiere aplicar a una palabra o frase, puede proporcionarla como marcado dentro de la solicitud. El diccionario dinámico solo es seguro para los nombres compuestos, como nombres propios y nombres de productos.

El marcado que se va a proporcionar utiliza la sintaxis siguiente.

``` 
<mstrans:dictionary translation=”translation of phrase”>phrase</mstrans:dictionary>
```

Por ejemplo, considere la frase en inglés "The word wordomatic is a dictionary entry" (La palabra "wordomatic" es una entrada del diccionario). Para conservar la palabra _wordomatic_ en la traducción, envíe la solicitud:

```
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=de" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'The word <mstrans:dictionary translation=\"wordomatic\">word or phrase</mstrans:dictionary> is a dictionary entry.'}]"
```

El resultado es el siguiente:

```
[
    {
        "translations":[
            {"text":"Das Wort "wordomatic" ist ein Wörterbucheintrag.","to":"de"}
        ]
    }
]
```

Esta característica funciona del mismo modo con `textType=text` o con `textType=html`. La característica debe usarse con moderación. La mejor forma de personalización de traducción, y más adecuada, es mediante el uso de Custom Translator. Custom Translator hace un uso completo del contexto y las probabilidades estadísticas. Si tiene o puede permitirse crear datos de entrenamiento que muestren el trabajo o una frase en contexto, obtendrá resultados mucho mejores. [Más información acerca de Custom Translator](../customization.md).
 





