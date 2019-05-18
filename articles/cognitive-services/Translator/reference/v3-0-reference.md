---
title: Referencia de Translator Text API V3.0
titlesuffix: Azure Cognitive Services
description: Documentación de referencia para Translator Text API V3.0.
services: cognitive-services
author: v-pawal
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 03/29/2018
ms.author: v-jansko
ms.openlocfilehash: b59e4d574264f82a5875edad65e99bfb57150197
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65796876"
---
# <a name="translator-text-api-v30"></a>Translator Text API v3.0

## <a name="whats-new"></a>Novedades

La versión 3 de Translator Text API proporciona una API web moderna basada en JSON. Mejora la facilidad de uso y rendimiento mediante la consolidación de las características existentes en menos operaciones y proporciona nuevas características.

 * Transliteración para convertir texto en un idioma de un script a otro.
 * Traducción a varios idiomas en una sola solicitud.
 * Detección de idioma, traducción y transliteración en una sola solicitud.
 * Diccionario para buscar traducciones alternativas de un término, traducciones inversas y ejemplos que muestren los términos usados en contexto.
 * Resultados de detección de idioma más informativos.

## <a name="base-urls"></a>Direcciones URL base

Microsoft Translator está disponible en varias ubicaciones de centros de datos. En la actualidad, se encuentra en seis [regiones de Azure](https://azure.microsoft.com/global-infrastructure/regions):

* **América**: Oeste de EE. UU. 2 y Centro-oeste de EE. UU. 
* **Asia Pacífico:** Asia Suroriental y Sur de Corea del Sur
* **Europa:** Europa del Norte y Europa Occidental

En la mayoría de los casos, las solicitudes dirigidas a Microsoft Translator Text API se administran en el centro de datos que está más próximo a la ubicación donde se originó la solicitud. En caso de que se produzca un error en un centro de datos, la solicitud puede enrutarse fuera de la región.

Para hacer que la solicitud se administre en un centro de datos específico, cambie el punto de conexión Global en la solicitud de API por el punto de conexión regional que desee:

|DESCRIPCIÓN|Área|URL base|
|:--|:--|:--|
|Azure|Global|  api.cognitive.microsofttranslator.com|
|Azure|Norteamérica|   api-nam.cognitive.microsofttranslator.com|
|Azure|Europa|  api-eur.cognitive.microsofttranslator.com|
|Azure|Asia Pacífico|    api-apc.cognitive.microsofttranslator.com|


## <a name="authentication"></a>Authentication

Suscríbase a Translator Text API o [varios servicios de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/) en Microsoft Cognitive Services y usar su suscripción de clave (disponible en el portal de Azure) para autenticar. 

Hay tres encabezados que puede usar para autenticar su suscripción. En esta tabla, se explica cómo se utiliza cada uno de ellos:

|Encabezados|DESCRIPCIÓN|
|:----|:----|
|Ocp-Apim-Subscription-Key|*Úselo con la suscripción a Cognitive Services si pasa su clave secreta*.<br/>El valor es la clave secreta de Azure para su suscripción a Translator Text API.|
|Autorización|*Úselo con la suscripción a Cognitive Services si pasa un token de autenticación.*<br/>El valor es el token de portador: `Bearer <token>`.|
|Ocp-Apim-Subscription-Region|*Utilizar con suscripciones de varios servicios de Cognitive Services si se pasa una clave secreta de varios servicio.*<br/>El valor es la región de la suscripción de varios servicio. Este valor es opcional cuando no se usa una suscripción de varios servicio.|

###  <a name="secret-key"></a>Clave secreta
La primera opción consiste en realizar la autenticación con el encabezado `Ocp-Apim-Subscription-Key`. Basta con agregar el encabezado `Ocp-Apim-Subscription-Key: <YOUR_SECRET_KEY>` a la solicitud.

### <a name="authorization-token"></a>Token de autorización
Si lo desea, también puede cambiar la clave secreta por un token de acceso. Este token se incluirá en cada solicitud como un encabezado `Authorization`. Para obtener un token de autorización, realice una solicitud `POST` a la dirección URL siguiente:

| Entorno     | URL del servicio de autenticación                                |
|-----------------|-----------------------------------------------------------|
| Azure           | `https://api.cognitive.microsoft.com/sts/v1.0/issueToken` |

Estas son algunas solicitudes de ejemplo para obtener un token una vez proporcionada una clave secreta:

```
// Pass secret key using header
curl --header 'Ocp-Apim-Subscription-Key: <your-key>' --data "" 'https://api.cognitive.microsoft.com/sts/v1.0/issueToken'

// Pass secret key using query string parameter
curl --data "" 'https://api.cognitive.microsoft.com/sts/v1.0/issueToken?Subscription-Key=<your-key>'
```

Una solicitud correcta devuelve el token de acceso codificado como texto sin formato en el cuerpo de respuesta. El token válido se pasa al servicio Translator como un token de portador en la autorización.

```
Authorization: Bearer <Base64-access_token>
```

Un token de autenticación tiene una validez de 10 minutos. El token debe volver a usarse al realizar varias llamadas a las API de Translator. Sin embargo, si el programa realiza las solicitudes a la API de Translator durante un período de tiempo prolongado, el programa debe solicitar un nuevo token de acceso a intervalos regulares (por ejemplo, cada 8 minutos).

### <a name="multi-service-subscription"></a>Suscripción a varios servicios

La última opción de autenticación es usar la suscripción de varios servicio cognitivos de un servicio. Esta opción le permite utilizar una única clave secreta para autenticar las solicitudes de varios servicios. 

Cuando se usa una clave secreta varios servicio, debe incluir dos encabezados de autenticación con su solicitud. El primero pasa la clave secreta, mientras que el segundo especifica la región asociada con la suscripción. 
* `Ocp-Apim-Subscription-Key`
* `Ocp-Apim-Subscription-Region`

Región es obligatoria para la suscripción varios servicio de API de texto. La región seleccionada es la única región en la que puede usar para la traducción de texto cuando se usa la clave de suscripción de varios servicios, y debe ser la misma región que seleccionó al suscribirse a su suscripción de varios servicio a través del portal de Azure.

Las regiones disponibles son `australiaeast`, `brazilsouth`, `canadacentral`, `centralindia`, `centraluseuap`, `eastasia`, `eastus`, `eastus2`, `japaneast`, `northeurope`, `southcentralus`, `southeastasia`, `uksouth`, `westcentralus`, `westeurope`, `westus` y `westus2`.

Si decide pasar la clave secreta en la cadena de consulta con el parámetro `Subscription-Key`, tendrá que especificar la región con el parámetro de consulta `Subscription-Region`.

Si utiliza un token de portador, tendrá que obtener el token del punto de conexión de la región: `https://<your-region>.api.cognitive.microsoft.com/sts/v1.0/issueToken`.


## <a name="errors"></a>Errors

Una respuesta de error estándar es un objeto JSON con el par de nombre/valor denominado `error`. El valor también es un objeto JSON con propiedades:

  * `code`: código de error definido por el servidor.

  * `message`: cadena que proporciona una representación legible del error.

Por ejemplo, un cliente con una suscripción de prueba gratuita recibiría el error siguiente una vez agotada la cuota gratuita:

```
{
  "error": {
    "code":403001,
    "message":"The operation is not allowed because the subscription has exceeded its free quota."
    }
}
```
El código de error es un número de 6 dígitos que combina el código de estado HTTP de 3 dígitos y otro número de 3 dígitos que ayuda a categorizar aún más el error. Los códigos de error comunes son:

| Código | DESCRIPCIÓN |
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
| 400050| El texto de entrada es demasiado largo.|
| 400064| Falta un parámetro "translation" o no es válido.|
| 400070| El número de scripts de destino (parámetro ToScript) no coincide con el número de lenguajes de destino (parámetro To).|
| 400071| Valor no válido para TextType.|
| 400072| La matriz de texto de entrada tiene demasiados elementos.|
| 400073| El parámetro de script no es válido.|
| 400074| El cuerpo de la solicitud no es un JSON válido.|
| 400075| La combinación de categoría y par de lenguaje no es válida.|
| 400077| Se superó el tamaño máximo de solicitud.|
| 400079| El sistema personalizado solicitado para la traducción entre, desde y hacia el lenguaje no existe.|
| 401000| La solicitud no está autorizada porque faltan las credenciales o no son válidas.|
| 401015| "Las credenciales proporcionadas son para Speech API. Esta solicitud requiere credenciales para Text API. Use una suscripción a Translator Text API."|
| 403000| No se permite la operación.|
| 403001| No se permite la operación porque la suscripción superó su cuota disponible.|
| 405000| No se admite el método de solicitud para el recurso solicitado.|
| 408001| El sistema de traducción personalizado solicitado aún no está disponible. Vuelva a intentarlo en unos minutos.|
| 415000| Falta el encabezado Content-Type o no es válido.|
| 429000, 429001, 429002| El servidor rechazó la solicitud porque el cliente ha superado los límites de solicitudes.|
| 500000| Se ha producido un error inesperado. Si el error continúa, notifíquelo con la fecha y hora del error, con el identificador de la solicitud del encabezado de respuesta X-RequestId y con el identificador de cliente del encabezado de solicitud X-ClientTraceId.|
| 503000| El servicio no está disponible temporalmente. Inténtelo de nuevo. Si el error continúa, notifíquelo con la fecha y hora del error, con el identificador de la solicitud del encabezado de respuesta X-RequestId y con el identificador de cliente del encabezado de solicitud X-ClientTraceId.|

