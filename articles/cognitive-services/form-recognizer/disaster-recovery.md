---
title: Guía de recuperación ante desastres para Azure Form Recognizer
titleSuffix: Azure Cognitive Services
description: Aprenda a usar la API de copia de modelos para realizar copias de seguridad de los recursos de Form Recognizer.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: how-to
ms.date: 05/27/2020
ms.author: pafarley
ms.openlocfilehash: 42faf4ba0a596fc5b2b34f403a5117e5ceea82ed
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87903347"
---
# <a name="back-up-and-recover-your-form-recognizer-models"></a>Copia de seguridad y recuperación de los modelos de Form Recognizer

Cuando se crea un recurso de Form Recognizer en Azure Portal, se especifica una región. Desde ese momento, el recurso y todas sus operaciones permanecen asociadas a esa región concreta del servidor de Azure. No suele ser habitual, aunque tampoco imposible, encontrar un problema de red que afecte a toda una región. Si la solución debe estar siempre disponible, debe diseñarla para que conmute por error en otra región o divida la carga de trabajo entre dos o más regiones. Ambos enfoques requieren al menos dos recursos de Form Recognizer en distintas regiones y la capacidad de sincronizar [modelos personalizados](./quickstarts/curl-train-extract.md) entre regiones.

La API de copia admite este escenario, ya que permite copiar modelos personalizados de una cuenta de Form Recognizer o a otros usuarios, que pueden existir en cualquier región geográfica compatible. En esta guía se muestra cómo usar la API REST de copia con cURL. También puede usar un servicio de solicitud HTTP como Postman para emitir las solicitudes.

## <a name="business-scenarios"></a>Escenarios empresariales

Si su aplicación o empresa dependen del uso de un modelo personalizado de Form Recognizer, se recomienda copiar el modelo a otra cuenta de Form Recognizer en otra región. Si se produce una interrupción regional, puede acceder al modelo en la región donde se copió.

##  <a name="prerequisites"></a>Requisitos previos

1. Dos recursos de Azure Form Recognizer en diferentes regiones de Azure. Si no dispone de ellos, vaya a Azure Portal y <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer" title="Crear un recurso de Form Recognizer" target="_blank">cree un recurso de Form Recognizer <span class="docon docon-navigate-external x-hidden-focus"></span></a>.
1. La clave de suscripción, la dirección URL del punto de conexión y el identificador de suscripción del recurso de Form Recognizer. Puede encontrar estos valores en la pestaña **Información general** del recurso en Azure Portal.


## <a name="copy-api-overview"></a>Introducción a la API de copia

El proceso de copia de un modelo personalizado consta de los siguientes pasos:

1. En primer lugar, emita una solicitud de autorización de copia en el recurso de destino; es decir, el recurso que recibirá el modelo copiado. Obtendrá la dirección URL del modelo de destino que se acaba de crear, que recibirá los datos copiados.
1. A continuación, envíe la solicitud de copia al recurso de origen; es decir, el recurso que contiene el modelo que se va a copiar. Obtendrá una dirección URL que puede consultar para realizar el seguimiento del progreso de la operación.
1. Usará las credenciales del recurso de origen para consultar la dirección URL de progreso hasta que la operación finalice correctamente. También puede consultar el nuevo identificador del modelo en el recurso de destino para obtener el estado del nuevo modelo.

## <a name="generate-copy-authorization-request"></a>Generación de la solicitud de autorización de copia

La siguiente solicitud HTTP obtiene la autorización de copia del recurso de destino. Deberá especificar el punto de conexión y la clave del recurso de destino como encabezados.

```
POST https://{TARGET_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/copyAuthorization HTTP/1.1
Ocp-Apim-Subscription-Key: {TARGET_FORM_RECOGNIZER_RESOURCE_API_KEY}
```

Obtendrá una respuesta `201\Created` con un valor `modelId` en el cuerpo. Esta cadena es el identificador del modelo recién creado (en blanco). La clase `accessToken` es necesaria para que la API copie los datos en este recurso, y el valor de `expirationDateTimeTicks` es la expiración del token. Guarde los tres valores en una ubicación segura.

```
HTTP/1.1 201 Created
Location: https://{TARGET_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/33f4d42c-cd2f-4e74-b990-a1aeafab5a5d
{"modelId":"<your model ID>","accessToken":"<your access token>","expirationDateTimeTicks":637233481531659440}
```

## <a name="start-copy-operation"></a>Inicio de la operación de copia

La siguiente solicitud HTTP inicia la operación de copia en el recurso de origen. Deberá especificar el punto de conexión y la clave del recurso de origen como encabezados. Observe que la dirección URL de la solicitud contiene el identificador de modelo del modelo de origen que desea copiar.

```
POST https://{SOURCE_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/<your model ID>/copy HTTP/1.1
Ocp-Apim-Subscription-Key: {SOURCE_FORM_RECOGNIZER_RESOURCE_API_KEY}
```

El cuerpo de la solicitud debe tener el formato siguiente. Deberá escribir el identificador de recurso y el nombre de la región del recurso de destino. También necesitará el identificador de modelo, el token de acceso y el valor de expiración que copió del paso anterior.

```json
{
   "targetResourceId": "{TARGET_AZURE_FORM_RECOGNIZER_RESOURCE_ID}",  
   "targetResourceRegion": "{TARGET_AZURE_FORM_RECOGNIZER_RESOURCE_REGION_NAME}",
   "copyAuthorization": {"modelId":"<your model ID>","accessToken":"<your access token>","expirationDateTimeTicks":637233481531659440}
}
```

> [!NOTE]
> La API de copia admite de forma transparente la característica [AEK/CMK](https://msazure.visualstudio.com/Cognitive%20Services/_wiki/wikis/Cognitive%20Services.wiki/52146/Customer-Managed-Keys). No se requiere ningún tratamiento especial, pero tenga en cuenta que si va a copiar de un recurso sin cifrar a un recurso cifrado, debe incluir el encabezado de solicitud `x-ms-forms-copy-degrade: true`. Si no se incluye este encabezado, se producirá un error en la operación de copia y se devolverá el error `DataProtectionTransformServiceError`.

Recibirá una respuesta `202\Accepted` con un encabezado Operation-Location (Operación-Ubicación). Este valor es la dirección URL que se usará para realizar el seguimiento del progreso de la operación. Cópielo en una ubicación temporal para el siguiente paso.

```
HTTP/1.1 202 Accepted
Operation-Location: https://{SOURCE_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/eccc3f13-8289-4020-ba16-9f1d1374e96f/copyresults/02989ba8-1296-499f-aaf4-55cfff41b8f1
```

### <a name="common-errors"></a>Errores comunes

|Error|Resolución|
|:--|:--|
| 400/Solicitud incorrecta con `"code:" "1002"` | Indica un error de validación o una solicitud de copia con formato incorrecto. Entre los problemas habituales se incluyen: a) Una carga `copyAuthorization` no válida o modificada. b) Un valor expirado para el token `expirationDateTimeTicks` (la carga `copyAuhtorization` es válida durante 24 horas). c) `targetResourceRegion` no válido o no admitido. d) Cadena `targetResourceId` no válida o con el formato incorrecto.
|

## <a name="track-copy-progress"></a>Seguimiento del progreso de la copia

Realice un seguimiento del progreso consultando la API de **obtención del resultado del modelo de copia** en el punto de conexión del recurso de origen.

```
GET https://{SOURCE_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/eccc3f13-8289-4020-ba16-9f1d1374e96f/copyresults/02989ba8-1296-499f-aaf4-55cfff41b8f1 HTTP/1.1
Ocp-Apim-Subscription-Key: {SOURCE_FORM_RECOGNIZER_RESOURCE_API_KEY}
```

La respuesta variará en función del estado de la operación. Busque el campo `"status"` en el cuerpo JSON. Si está automatizando esta llamada API en un script, se recomienda consultar la operación una vez por segundo.

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
{"status":"succeeded","createdDateTime":"2020-04-23T18:18:01.0275043Z","lastUpdatedDateTime":"2020-04-23T18:18:01.0275048Z","copyResult":{}}
```

### <a name="common-errors"></a>Errores comunes

|Error|Resolución|
|:--|:--|
|"errors":[{"code":"AuthorizationError",<br>"message":"Error de autorización debido a <br>notificaciones de autorización faltantes o no válidas".}]   | Se produce cuando se modifica la carga o el contenido de `copyAuthorization` se modifica en el resultado de la API de `copyAuthorization`. Asegúrese de que la carga tenga exactamente el mismo contenido que devolvió la llamada `copyAuthorization` anterior.|
|"errors":[{"code":"AuthorizationError",<br>"message":"No se pudieron recuperar los metadatos <br>de autorización. Si el problema persiste, use otro <br>modelo de destino para copiar los datos".}] | Indica que la carga de `copyAuthorization` se está usando de nuevo con una solicitud de copia. Una solicitud de copia que se realiza correctamente no permitirá que otras solicitudes usen la misma carga de `copyAuthorization`. Si genera un error independiente (como los que se indican a continuación) y, posteriormente, intenta de nuevo la copia con la misma carga de autorización, se genera este error. La resolución consiste en generar una nueva carga de `copyAuthorization` y, a continuación, volver a emitir la solicitud de copia.|
|"errors":[{"code":"DataProtectionTransformServiceError",<br>"message": "No se permite la solicitud de transferencia de datos, <br>ya que cambia a un esquema de protección de datos menos seguro. Consulte la documentación o póngase en contacto con el administrador de servicios <br>para obtener más detalles".}]    | Se produce cuando se realiza una copia entre un recurso con `AEK` habilitado y un recurso que no tiene `AEK` habilitado. Para permitir que se copie el modelo de cifrado al destino como uno sin cifrar, especifique el encabezado `x-ms-forms-copy-degrade: true` con la solicitud de copia.|
|"errors":[{"code":"ResourceResolverError",<br>"message": "No se pudo obtener información del recurso cognitivo con el identificador "...". Asegúrese de que el recurso sea válido y exista en la región especificada "westus2"".}] | Indica que el recurso de Azure señalado por `targetResourceId` no es un recurso cognitivo válido o no existe. Compruebe y vuelva a emitir la solicitud de copia para resolver el problema.|


### <a name="optional-track-the-target-model-id"></a>Seguimiento del identificador del modelo de destino (opcional) 

También puede usar la API de **obtención del modelo personalizado** para realizar un seguimiento del estado de la operación consultando el modelo de destino. Llame a esta API con el identificador del modelo de destino que copió en el primer paso.

```
GET https://{TARGET_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/33f4d42c-cd2f-4e74-b990-a1aeafab5a5d HTTP/1.1
Ocp-Apim-Subscription-Key: {TARGET_FORM_RECOGNIZER_RESOURCE_API_KEY}
```

En el cuerpo de la respuesta, verá información sobre el modelo. Compruebe el estado del modelo en el campo `"status"`.

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
{"modelInfo":{"modelId":"33f4d42c-cd2f-4e74-b990-a1aeafab5a5d","status":"ready","createdDateTime":"2020-02-26T16:59:28Z","lastUpdatedDateTime":"2020-02-26T16:59:34Z"},"trainResult":{"trainingDocuments":[{"documentName":"0.pdf","pages":1,"errors":[],"status":"succeeded"},{"documentName":"1.pdf","pages":1,"errors":[],"status":"succeeded"},{"documentName":"2.pdf","pages":1,"errors":[],"status":"succeeded"},{"documentName":"3.pdf","pages":1,"errors":[],"status":"succeeded"},{"documentName":"4.pdf","pages":1,"errors":[],"status":"succeeded"}],"errors":[]}}
```

## <a name="curl-sample-code"></a>Código de ejemplo de cURL

En los fragmentos de código siguientes se usa cURL para realizar las llamadas API que se describen en los pasos anteriores. Todavía tendrá que rellenar los identificadores de modelo y la información de suscripción correspondientes a sus propios recursos.

### <a name="generate-copy-authorization-request"></a>Generación de la solicitud de autorización de copia

```bash
curl -i -X POST "https://{TARGET_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/copyAuthorization" -H "Ocp-Apim-Subscription-Key: {TARGET_FORM_RECOGNIZER_RESOURCE_API_KEY}" 
```

### <a name="start-copy-operation"></a>Inicio de la operación de copia

```bash
curl -i -X POST "https://{TARGET_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/copyAuthorization" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {TARGET_FORM_RECOGNIZER_RESOURCE_API_KEY}" --data-ascii "{ \"targetResourceId\": \"{TARGET_AZURE_FORM_RECOGNIZER_RESOURCE_ID}\",   \"targetResourceRegion\": \"{TARGET_AZURE_FORM_RECOGNIZER_RESOURCE_REGION_NAME}\", \"copyAuthorization\": "{\"modelId\":\"33f4d42c-cd2f-4e74-b990-a1aeafab5a5d\",\"accessToken\":\"1855fe23-5ffc-427b-aab2-e5196641502f\",\"expirationDateTimeTicks\":637233481531659440}"}"
```

### <a name="track-copy-progress"></a>Seguimiento del progreso de la copia

```bash
curl -i GET "https://<SOURCE_FORM_RECOGNIZER_RESOURCE_ENDPOINT>/formrecognizer/v2.0/custom/models/{SOURCE_MODELID}/copyResults/{RESULT_ID}" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {SOURCE_FORM_RECOGNIZER_RESOURCE_API_KEY}"
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía, ha aprendido a usar la API de copia para realizar una copia de seguridad de los modelos personalizados en un recurso de Form Recognizer secundario. A continuación, consulte los documentos de referencia de la API para ver qué más puede hacer con Form Recognizer.
* [Documentación de referencia de API REST](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeWithCustomForm)