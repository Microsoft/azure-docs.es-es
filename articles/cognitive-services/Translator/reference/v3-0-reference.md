---
title: Referencia de Translator V3.0
titleSuffix: Azure Cognitive Services
description: Documentación de referencia para Translator V3.0. La versión 3 de Translator proporciona una API web moderna basada en JSON.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 8/11/2020
ms.author: swmachan
ms.openlocfilehash: 6b211dd8ca735ea9ee4a5209aa6030398cca472e
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88121024"
---
# <a name="translator-v30"></a>Translator v3.0

## <a name="whats-new"></a>Novedades

La versión 3 de Translator proporciona una API web moderna basada en JSON. Mejora la facilidad de uso y rendimiento mediante la consolidación de las características existentes en menos operaciones y proporciona nuevas características.

 * Transliteración para convertir texto en un idioma de un script a otro.
 * Traducción a varios idiomas en una sola solicitud.
 * Detección de idioma, traducción y transliteración en una sola solicitud.
 * Diccionario para buscar traducciones alternativas de un término, traducciones inversas y ejemplos que muestren los términos usados en contexto.
 * Resultados de detección de idioma más informativos.

## <a name="base-urls"></a>Direcciones URL base

Microsoft Translator está disponible en varias ubicaciones de centros de datos. En la actualidad, se encuentran en diez [regiones de Azure](https://azure.microsoft.com/global-infrastructure/regions):

* **América**: Este de EE. UU., Centro-sur de EE. UU. , Centro-oeste de EE. UU. y Oeste de EE. UU. 2 
* **Asia Pacífico:** Sur de Corea del Sur, Este de Japón, Sudeste de Asia y Este de Australia
* **Europa:** Norte de Europa y Oeste de Europa

En la mayoría de los casos, las solicitudes dirigidas a Microsoft Translator se administran en el centro de datos que está más próximo a la ubicación donde se originó la solicitud. En caso de que se produzca un error en un centro de datos, la solicitud puede enrutarse fuera de la geografía de Azure.

Para hacer que la solicitud se controle en una geografía de Azure específica, cambie el punto de conexión Global en la solicitud de API por el punto de conexión regional que desee:

|Descripción|Geografía de Azure|URL base|
|:--|:--|:--|
|Azure|Global (no regional)|   api.cognitive.microsofttranslator.com|
|Azure|Estados Unidos|   api-nam.cognitive.microsofttranslator.com|
|Azure|Europa|  api-eur.cognitive.microsofttranslator.com|
|Azure|Asia Pacífico|    api-apc.cognitive.microsofttranslator.com|

## <a name="authentication"></a>Authentication

Suscríbase a Translator o a los [varios servicios de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/) en Azure Cognitive Services y use la clave de suscripción (disponible en Azure Portal) para autenticarse. 

Hay tres encabezados que puede usar para autenticar su suscripción. En esta tabla, se explica cómo se utiliza cada uno de ellos:

|encabezados|Descripción|
|:----|:----|
|Ocp-Apim-Subscription-Key|*Úselo con la suscripción a Cognitive Services si pasa su clave secreta*.<br/>El valor es la clave secreta de Azure para su suscripción a Translator.|
|Authorization|*Úselo con la suscripción a Cognitive Services si pasa un token de autenticación.*<br/>El valor es el token de portador: `Bearer <token>`.|
|Ocp-Apim-Subscription-Region|*Úselo con el recurso de traductor regional y de varios servicios de Cognitive Services.*<br/>El valor es la región del recurso de traductor de varios servicios o regional. Este valor es opcional cuando se usa un recurso de traductor global.|

###  <a name="secret-key"></a>Clave secreta
La primera opción consiste en realizar la autenticación con el encabezado `Ocp-Apim-Subscription-Key`. Agregue el encabezado `Ocp-Apim-Subscription-Key: <YOUR_SECRET_KEY>` a la solicitud.

#### <a name="authenticating-with-a-global-resource"></a>Autenticación con un recurso global

Cuando se usa un [recurso de traductor global](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextTranslation), es necesario incluir un encabezado para llamar a Translator.

|encabezados|Descripción|
|:-----|:----|
|Ocp-Apim-Subscription-Key| El valor es la clave secreta de Azure para su suscripción a Translator.|

A continuación se muestra una solicitud de ejemplo para llamar a Translator mediante el recurso de traductor global.

```curl
// Pass secret key using headers
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&to=es" \
     -H "Ocp-Apim-Subscription-Key:<your-key>" \
     -H "Content-Type: application/json" \
     -d "[{'Text':'Hello, what is your name?'}]"
```

#### <a name="authenticating-with-a-regional-resource"></a>Autenticación con un recurso regional

Cuando se usa un [recursos de traductor regional](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextTranslation).
Hay dos encabezados que necesita para llamar a Translator.

|encabezados|Descripción|
|:-----|:----|
|Ocp-Apim-Subscription-Key| El valor es la clave secreta de Azure para su suscripción a Translator.|
|Ocp-Apim-Subscription-Region| El valor es la región del recurso de traductor. |

A continuación se muestra una solicitud de ejemplo para llamar a Translator mediante el recurso de traductor regional.

```curl
// Pass secret key and region using headers
curl -X POST "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&to=es" \
     -H "Ocp-Apim-Subscription-Key:<your-key>" \
     -H "Ocp-Apim-Subscription-Region:<your-region>" \
     -H "Content-Type: application/json" \
     -d "[{'Text':'Hello, what is your name?'}]"
```

#### <a name="authenticating-with-a-multi-service-resource"></a>Autenticación con un recurso de varios servicios

Cuando use un recurso de varios servicios de Cognitive Services. Esta opción le permite utilizar una única clave secreta para autenticar las solicitudes de varios servicios. 

Si usa una clave secreta para varios servicios, debe incluir dos encabezados de autenticación con la solicitud. Hay dos encabezados que necesita para llamar a Translator.

|encabezados|Descripción|
|:-----|:----|
|Ocp-Apim-Subscription-Key| El valor es la clave secreta de Azure para el recurso de varios servicios.|
|Ocp-Apim-Subscription-Region| El valor es la región del recurso de varios servicios. |

La región es obligatoria en la suscripción de varios servicios de Text API. La región seleccionada es la única región que puede usar para la traducción de texto al usar la clave de suscripción de varios servicios, y debe ser la misma región que seleccionó al registrarse en su suscripción de varios servicios a través de Azure Portal.

Las regiones disponibles son `australiaeast`, `brazilsouth`, `canadacentral`, `centralindia`, `centralus`, `centraluseuap`, `eastasia`, `eastus`, `eastus2`, `francecentral`, `japaneast`, `japanwest`, `koreacentral`, `northcentralus`, `northeurope`, `southcentralus`, `southeastasia`, `uksouth`, `westcentralus`, `westeurope`, `westus`, `westus2` y `southafricanorth`.

Si decide pasar la clave secreta en la cadena de consulta con el parámetro `Subscription-Key`, tendrá que especificar la región con el parámetro de consulta `Subscription-Region`.

### <a name="authenticating-with-an-access-token"></a>Autenticación con token de acceso
Si lo desea, también puede cambiar la clave secreta por un token de acceso. Este token se incluirá en cada solicitud como un encabezado `Authorization`. Para obtener un token de autorización, realice una solicitud `POST` a la dirección URL siguiente:

| Tipo de recurso     | URL del servicio de autenticación                                |
|-----------------|-----------------------------------------------------------|
| Global          | `https://api.cognitive.microsoft.com/sts/v1.0/issueToken` |
| Regional o de varios servicios | `https://<your-region>.api.cognitive.microsoft.com/sts/v1.0/issueToken` |

Estas son algunas solicitudes de ejemplo para obtener un token una vez proporcionada una clave secreta:

```curl
// Pass secret key using header
curl --header 'Ocp-Apim-Subscription-Key: <your-key>' --data "" 'https://api.cognitive.microsoft.com/sts/v1.0/issueToken'

// Pass secret key using query string parameter
curl --data "" 'https://api.cognitive.microsoft.com/sts/v1.0/issueToken?Subscription-Key=<your-key>'
```

Una solicitud correcta devuelve el token de acceso codificado como texto sin formato en el cuerpo de respuesta. El token válido se pasa al servicio Translator como un token de portador en la autorización.

```http
Authorization: Bearer <Base64-access_token>
```

Un token de autenticación tiene una validez de 10 minutos. El token debe volver a usarse al realizar varias llamadas a Translator. Sin embargo, si el programa realiza las solicitudes a Translator durante un período de tiempo prolongado, el programa debe solicitar un nuevo token de acceso a intervalos regulares (por ejemplo, cada 8 minutos).

## <a name="virtual-network-support"></a>Compatibilidad con redes virtuales

El servicio Traductor ahora está disponible con las funcionalidades de Virtual Network (VNET) en todas las regiones de la nube pública de Azure. Para habilitar la red virtual, consulte [Configuración de las redes virtuales de Azure Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-virtual-networks?tabs=portal). 

Una vez que se activa esta funcionalidad, se debe usar el punto de conexión personalizado para llamar a Translator. No se puede usar el punto de conexión de traductor global ("api.cognitive.microsofttranslator.com") y no se puede autenticar con un token de acceso.

Puede encontrar el punto de conexión personalizado después de crear un [recurso de Translator](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextTranslation) y permitir el acceso desde redes seleccionadas y puntos de conexión privados.

|encabezados|Descripción|
|:-----|:----|
|Ocp-Apim-Subscription-Key| El valor es la clave secreta de Azure para su suscripción a Translator.|
|Ocp-Apim-Subscription-Region| El valor es la región del recurso de traductor. Este valor es opcional si el recurso es `global`.|

A continuación se muestra una solicitud de ejemplo para llamar a Translator mediante el punto de conexión personalizado.

```curl
// Pass secret key and region using headers
curl -X POST "https://<your-custom-domain>.cognitiveservices.azure.com/translator/text/v3.0/translate?api-version=3.0&to=es" \
     -H "Ocp-Apim-Subscription-Key:<your-key>" \
     -H "Ocp-Apim-Subscription-Region:<your-region>" \
     -H "Content-Type: application/json" \
     -d "[{'Text':'Hello, what is your name?'}]"
```

## <a name="errors"></a>Errors

Una respuesta de error estándar es un objeto JSON con el par de nombre/valor denominado `error`. El valor también es un objeto JSON con propiedades:

  * `code`: código de error definido por el servidor.
  * `message`: cadena que proporciona una representación legible del error.

Por ejemplo, un cliente con una suscripción de prueba gratuita recibiría el error siguiente una vez agotada la cuota gratuita:

```json
{
  "error": {
    "code":403001,
    "message":"The operation is not allowed because the subscription has exceeded its free quota."
    }
}
```
El código de error es un número de 6 dígitos que combina el código de estado HTTP de 3 dígitos y otro número de 3 dígitos que ayuda a categorizar aún más el error. Los códigos de error comunes son:

| Código | Descripción |
|:----|:-----|
| 400000| Una de las entradas de la solicitud no es válida.|
| 400001| El parámetro "scope" no es válido.|
| 400002| El parámetro "category" no es válido.|
| 400003| Falta un especificador de lenguaje o no es válido.|
| 400004| Falta un especificador de script de destino ("To script") o no es válido.|
| 400005| Falta un texto de entrada o no es válido.|
| 400006| La combinación de lenguaje y script no es válida.|
| 400018| Falta un especificador de script de origen ("From script") o no es válido.|
| 400019| No se admite uno de los lenguajes especificados.|
| 400020| Uno de los elementos de la matriz de texto de entrada no es válido.|
| 400021| Falta un parámetro de la versión de API o no es válido.|
| 400023| Uno de los pares de lenguaje especificados no es válido.|
| 400035| El lenguaje fuente (campo "From") no es válido.|
| 400036| Falta el lenguaje de destino (campo "To") o no es válido.|
| 400042| Una de las opciones especificadas (campo "Options") no es válida.|
| 400043| Falta el id. de seguimiento del cliente (campo ClientTraceId o encabezado X-ClientTranceId) o no es válido.|
| 400050| El texto de entrada es demasiado largo. Consulte los [límites de solicitud](../request-limits.md).|
| 400064| Falta un parámetro "translation" o no es válido.|
| 400070| El número de scripts de destino (parámetro ToScript) no coincide con el número de lenguajes de destino (parámetro To).|
| 400071| Valor no válido para TextType.|
| 400072| La matriz de texto de entrada tiene demasiados elementos.|
| 400073| El parámetro de script no es válido.|
| 400074| El cuerpo de la solicitud no es un JSON válido.|
| 400075| La combinación de categoría y par de lenguaje no es válida.|
| 400077| Se superó el tamaño máximo de solicitud. Consulte los [límites de solicitud](../request-limits.md).|
| 400079| El sistema personalizado solicitado para la traducción entre, desde y hacia el lenguaje no existe.|
| 400080| La transliteración no se admite para el idioma o el script.|
| 401000| La solicitud no está autorizada porque faltan las credenciales o no son válidas.|
| 401015| "Las credenciales proporcionadas son para Speech API. Esta solicitud requiere credenciales para Text API. Use una suscripción a Translator".|
| 403000| No se permite la operación.|
| 403001| No se permite la operación porque la suscripción superó su cuota disponible.|
| 405000| No se admite el método de solicitud para el recurso solicitado.|
| 408001| Se está preparando el sistema de traducción solicitado. Vuelva a intentarlo en unos minutos.|
| 408002| La solicitud superó el tiempo de espera mientras se esperaba la secuencia entrante. El cliente no presentó una solicitud en el tiempo que el servidor estaba preparado para esperar. El cliente puede repetir la solicitud sin modificaciones en cualquier momento posterior.|
| 415000| Falta el encabezado Content-Type o no es válido.|
| 429000, 429001, 429002| El servidor rechazó la solicitud porque el cliente superó los límites de solicitudes.|
| 500000| Se ha producido un error inesperado. Si el error continúa, notifíquelo con la fecha y hora del error, con el identificador de la solicitud del encabezado de respuesta X-RequestId y con el identificador de cliente del encabezado de solicitud X-ClientTraceId.|
| 503000| El servicio no está disponible temporalmente. Inténtelo de nuevo. Si el error continúa, notifíquelo con la fecha y hora del error, con el identificador de la solicitud del encabezado de respuesta X-RequestId y con el identificador de cliente del encabezado de solicitud X-ClientTraceId.|

## <a name="metrics"></a>Métricas 
Las métricas le permiten ver la información de uso y disponibilidad del traductor en Azure Portal, en la sección de métricas, tal como se muestra en la captura de pantalla siguiente. Para obtener más información, vea [Métricas de datos y plataforma](https://docs.microsoft.com/azure/azure-monitor/platform/data-platform-metrics).

![Métricas del traductor](../media/translatormetrics.png)

En esta tabla se enumeran las métricas disponibles con la descripción de cómo se usan para supervisar las llamadas a la API de traducción.

| Métricas | Descripción |
|:----|:-----|
| TotalCalls| Número total de llamadas a API.|
| TotalTokenCalls| Número total de llamadas API a través del servicio de token mediante el token de autenticación.|
| SuccessfulCalls| Número de llamadas correctas.|
| TotalErrors| Número de llamadas con respuesta de error.|
| BlockedCalls| Número de llamadas que han superado la tasa o el límite de cuota.|
| ServerErrors| Número de llamadas con error interno del servidor (5XX).|
| ClientErrors| Número de llamadas con error en el cliente (4XX).|
| Latencia| Duración para completar la solicitud en milisegundos.|
| CharactersTranslated| Número total de caracteres de la solicitud entrante de texto.|
