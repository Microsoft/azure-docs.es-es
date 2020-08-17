---
title: Información general sobre la API de REST de operaciones de Media Services | Microsoft Docs
description: La API REST "Media Services Operations" se usa para crear trabajos, recursos, canales en vivo y otros recursos en una cuenta de Media Services. En este artículo se proporciona información general sobre la API REST de Azure Media Services v2.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: a5f1c5e7-ec52-4e26-9a44-d9ea699f68d9
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.reviewer: johndeu
ms.openlocfilehash: 83e945fccfbfbec207723d6c16f2a4dfc7290c52
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87000054"
---
# <a name="media-services-operations-rest-api-overview"></a>Información general sobre la API de REST de operaciones de Media Services 

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

La API de **REST de Media Services Operations** se usa para crear trabajos, activos, canales en vivo y otros recursos de una cuenta de Media Services. Para obtener más información, consulte la [referencia de la API de REST de Media Services Operations](/rest/api/media/operations/azure-media-services-rest-api-reference).

Media Services proporciona una API de REST que acepta ambos formatos JSON o XML atom+pub. La API de REST de Media Services requiere encabezados HTTP específicos que cada cliente debe enviar cuando se conecta a Media Services, así como un conjunto de encabezados opcionales. En las secciones siguientes se describen los encabezados y los verbos HTTP que puede usar al crear solicitudes y recibir respuestas de los Media Services.

La autenticación para la API REST de Media Services se realiza a través de la autenticación de Azure Active Directory que se detalla en el artículo [Usar autenticación de Azure AD para acceder a la API de Azure Media Services con REST](media-services-rest-connect-with-aad.md).

## <a name="considerations"></a>Consideraciones

Las siguientes consideraciones se aplican al usar REST.

* Al consultar entidades, hay un límite de 1000 entidades devueltas a la vez, porque la REST v2 pública limita los resultados de consulta a 1000. Debe usar **Skip** y **Take** (.NET)/ **top** (REST) como se describe en [este ejemplo de .NET](media-services-dotnet-manage-entities.md#enumerating-through-large-collections-of-entities) y de [API de REST](media-services-rest-manage-entities.md#enumerating-through-large-collections-of-entities). 
* Cuando se utiliza JSON y se especifica el uso de la palabra clave **__metadata** en la solicitud (por ejemplo, para hacer referencia a un objeto vinculado) DEBE establecer el encabezado **Accept** en [formato JSON detallado](https://www.odata.org/documentation/odata-version-3-0/json-verbose-format/) (consulte el ejemplo siguiente). Odata no entiende la propiedad **__metadata** en la solicitud, a menos que se establezca en el modo detallado.  

    ```console
    POST https://media.windows.net/API/Jobs HTTP/1.1
    Content-Type: application/json;odata=verbose
    Accept: application/json;odata=verbose
    DataServiceVersion: 3.0
    MaxDataServiceVersion: 3.0
    x-ms-version: 2.19
    Authorization: Bearer <ENCODED JWT TOKEN> 
    Host: media.windows.net

    {
        "Name" : "NewTestJob", 
        "InputMediaAssets" : 
            [{"__metadata" : {"uri" : "https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3Aba5356eb-30ff-4dc6-9e5a-41e4223540e7')"}}]
    . . . 
   ```

## <a name="standard-http-request-headers-supported-by-media-services"></a>Encabezados de solicitud HTTP estándar compatibles con los Media Services
Para cada llamada que realice en los Media Services, hay un conjunto de encabezados necesarios que debe incluir en la solicitud y también un conjunto de encabezados opcionales que puede incluir si lo desea. En la tabla siguiente se enumeran los encabezados necesarios:

| Encabezado | Tipo | Value |
| --- | --- | --- |
| Authorization |Portador |El portador es el único mecanismo de autorización aceptado. El valor también debe incluir el token de acceso proporcionado por Azure Active Directory. |
| x-ms-version |Decimal |2.17 (o una versión más reciente)|
| DataServiceVersion |Decimal |3.0 |
| MaxDataServiceVersion |Decimal |3.0 |

> [!NOTE]
> Dado que Media Services usa OData para exponer sus API de REST, los encabezados DataServiceVersion y MaxDataServiceVersion deben incluirse en todas las solicitudes; sin embargo, si no se incluyen, Media Services supone que el valor de DataServiceVersion en uso es 3.0.
> 
> 

A continuación se muestra un conjunto de encabezados opcionales:

| Encabezado | Tipo | Value |
| --- | --- | --- |
| Date |Fecha RFC 1123 |Marca de tiempo de la solicitud |
| Accept |Tipo de contenido |El tipo de contenido solicitado para la respuesta, por ejemplo, el siguiente:<p> -application/json;odata=verbose<p> - application/atom+xml<p> Las respuestas pueden tener distintos tipos de contenido, como una captura de blob, donde una respuesta correcta contenga la secuencia de blob como la carga. |
| Accept-Encoding |Gzip, deflate |Codificación GZIP y DEFLATE, según corresponda. Nota: En el caso de recursos grandes, Media Services puede ignorar el encabezado y devolver datos sin comprimir. |
| Accept-Language |"en", "es", etc. |Especifica el idioma preferido para la respuesta. |
| Accept-Charset |Tipo de juego de caracteres como "UTF-8" |El valor predeterminado es UTF-8. |
| X-HTTP-Method |Método HTTP |Permite a los clientes o firewalls que no admiten métodos HTTP como PUT o DELETE usar estos métodos mediante túnel a través de una llamada GET. |
| Content-Type |Tipo de contenido |Tipo de contenido del cuerpo de la solicitud en solicitudes PUT o POST. |
| client-request-id |String |Valor definido por el autor de la llamada que identifica la solicitud determinada. Si se especifica, este valor se incluirá en el mensaje de respuesta como una manera de asignar la solicitud. <p><p>**Importante**<p>Los valores deben estar limitados a 2096b (2k). |

## <a name="standard-http-response-headers-supported-by-media-services"></a>Encabezados de respuesta HTTP estándar compatibles con los Media Services
A continuación se muestra un conjunto de encabezados que pueden devolverse según el recurso que estuviera solicitando y la acción que deseara realizar.

| Encabezado | Tipo | Value |
| --- | --- | --- |
| request-id |String |Identificador único de la operación o servicio actual generado. |
| client-request-id |String |Identificador especificado por el autor de la llamada en la solicitud original, si está presente. |
| Date |Fecha RFC 1123 |Fecha y hora en que se procesó la solicitud. |
| Content-Type |Varía |Tipo de contenido del cuerpo de respuesta. |
| Content-Encoding |Varía |Gzip o deflate, según corresponda. |

## <a name="standard-http-verbs-supported-by-media-services"></a>Verbos HTTP estándar compatibles con los Media Services
A continuación se muestra una lista completa de verbos HTTP que se pueden usar al realizar solicitudes HTTP:

| Verbo | Descripción |
| --- | --- |
| GET |Devuelve el valor actual de un objeto. |
| POST |Crea un objeto basado en los datos proporcionados o envía un comando. |
| PUT |Reemplaza un objeto o crea un objeto con nombre (si procede). |
| Delete |Elimina un objeto. |
| MERGE |Actualiza un objeto existente con los cambios de propiedad con nombre. |
| HEAD |Devuelve los metadatos de un objeto para una respuesta GET. |

## <a name="discover-and-browse-the-media-services-entity-model"></a>Detección y exploración del modelo de entidad de Media Services
Para que las entidades de los Media Services sean más reconocibles, se puede usar la operación de metadatos. Permite recuperar todos los tipos de entidad válidos, las propiedades de entidad, las asociaciones, las funciones, las acciones, etc. Si agrega la operación $metadata al final de su punto de conexión de API de REST de Media Services, puede acceder a este servicio de detección.

 /api/$metadata.

Anexe "?api-version=2.x" al final del URI si desea ver los metadatos en un explorador o no incluya el encabezado x-ms-version en la solicitud.

## <a name="authenticate-with-media-services-rest-using-azure-active-directory"></a>Autenticación con REST de Media Services mediante Azure Active Directory

La autenticación en la API de REST se lleva a cabo con Azure Active Directory (ADD).
Para obtener más información acerca de cómo obtener los detalles de autenticación necesarios de su cuenta de Media Services desde Azure Portal, consulte [Acceso a Azure Media Services API con la autenticación de Azure AD](media-services-use-aad-auth-to-access-ams-api.md).

Para obtener más información sobre la escritura de código que se conecte a la API de REST mediante la autenticación de Azure AD, vea el artículo [Usar autenticación de Azure AD para acceder a la API de Azure Media Services con REST](media-services-rest-connect-with-aad.md).

## <a name="next-steps"></a>Pasos siguientes
Para obtener información sobre cómo utilizar la autenticación de Azure AD con la API de REST de Media Services, consulte [Usar autenticación de Azure AD para acceder a la API de Azure Media Services con REST](media-services-rest-connect-with-aad.md).

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
