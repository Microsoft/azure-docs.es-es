---
title: Administración de entidades de Media Services con REST| Microsoft Docs
description: En este artículo se muestra cómo administrar entidades de Media Services con la API REST.
author: juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: 95262a32-0f2a-4286-b9e2-1a1ca6399b5b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: 95124f101b1f14a70dabedd7d44077c5c1b6e99c
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89264273"
---
# <a name="managing-media-services-entities-with-rest"></a>Administración de entidades de Media Services con REST

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [REST](media-services-rest-manage-entities.md)
> * [.NET](media-services-dotnet-manage-entities.md)
> 
> 

Microsoft Azure Media Services es un servicio REST basado en OData v3. Puede agregar, consultar, actualizar y eliminar entidades prácticamente del mismo modo que con cualquier otro servicio de OData. Se indicarán las excepciones cuando proceda. Para obtener más información sobre OData, consulte la [documentación de Open Data Protocol](https://www.odata.org/documentation/).

En este tema se muestra cómo administrar entidades de Azure Media Services con REST.

>[!NOTE]
> A partir del 1 de abril de 2017, se eliminarán automáticamente los registros de trabajo de más de 90 días de su cuenta, junto con los registros de tarea asociados, aunque el número total de registros no llegue a la cuota máxima. Por ejemplo, el 1 de abril de 2017, todos los registros de trabajo de la cuenta que sean anteriores al 31 de diciembre de 2016 se eliminarán automáticamente. Si desea archivar la información del trabajo o la tarea, puede usar el código que se describe en este tema.

## <a name="considerations"></a>Consideraciones  

Al obtener acceso a las entidades de Media Services, debe establecer los campos de encabezado específicos y los valores en las solicitudes HTTP. Para obtener más información, consulte [Configuración del desarrollo de la API de REST de Media Services](media-services-rest-how-to-use.md).

## <a name="connect-to-media-services"></a>Conexión con Media Services

Para obtener más información sobre cómo conectarse a la API de Azure Media Services, consulte [Acceso a la API de Azure Media Services con la autenticación de Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

## <a name="adding-entities"></a>Incorporación de entidades
Todas las entidades de Media Services se agregan a un conjunto de entidades, como recursos, a través de una solicitud HTTP POST.

En el ejemplo siguiente se muestra cómo crear una entidad AccessPolicy.

```console
POST https://media.windows.net/API/AccessPolicies HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
Host: media.windows.net
Content-Length: 74
Expect: 100-continue

{"Name": "DownloadPolicy", "DurationInMinutes" : "300", "Permissions" : 1}
```

## <a name="querying-entities"></a>Consulta de entidades
La consulta y enumeración de entidades es sencilla y solo implica una solicitud HTTP GET y operaciones OData opcionales.
En el ejemplo siguiente se recupera una lista de todas las entidades MediaProcessor.

```console
GET https://media.windows.net/API/MediaProcessors HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
Host: media.windows.net
```

También puede recuperar una entidad específica o todos los conjuntos de entidades asociados con una entidad específica, como en los ejemplos siguientes:

```console
GET https://media.windows.net/API/JobTemplates('nb:jtid:UUID:e81192f5-576f-b247-b781-70a790c20e7c') HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
Host: media.windows.net

GET https://media.windows.net/API/JobTemplates('nb:jtid:UUID:e81192f5-576f-b247-b781-70a790c20e7c')/TaskTemplates HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
Host: media.windows.net
```

El ejemplo siguiente devuelve la propiedad State de todos los trabajos.

```console
GET https://media.windows.net/API/Jobs?$select=State HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
Host: media.windows.net
```

El ejemplo siguiente devuelve todos los JobTemplates con el nombre "SampleTemplate".

```console
GET https://media.windows.net/API/JobTemplates?$filter=startswith(Name,%20'SampleTemplate') HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
Host: media.windows.net
```

> [!NOTE]
> Media Services no admite la operación $expand, ni los métodos no admitidos de LINQ que se describen en las consideraciones sobre LINQ (Servicios de datos de WCF).
> 
> 

## <a name="enumerating-through-large-collections-of-entities"></a>Enumeración de grandes colecciones de entidades
Al consultar entidades, hay un límite de 1000 entidades devueltas a la vez, porque la REST v2 pública limita los resultados de consulta a 1000. Utilice **skip** y **top** para enumerar la gran colección de entidades. 

En el ejemplo siguiente se muestra cómo usar **skip** y **top** para omitir los 2000 primeros trabajos y obtener los 1000 siguientes.  

```console
GET https://media.windows.net/api/Jobs()?$skip=2000&$top=1000 HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: media.windows.net
```

## <a name="updating-entities"></a>Actualización de entidades
Según el tipo de entidad y el estado en el que se encuentre, puede actualizar las propiedades de entidad mediante solicitudes HTTP PATCH, PUT o MERGE. Para obtener más información acerca de estas operaciones, vea [PATCH, PUT, MERGE](/openspecs/windows_protocols/ms-odata/59d5abd3-7b12-490a-a0e2-9d9324b91893).

En el ejemplo de código siguiente se muestra cómo actualizar la propiedad Name en una entidad Asset.

```console
MERGE https://media.windows.net/API/Assets('nb:cid:UUID:80782407-3f87-4e60-a43e-5e4454232f60') HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: media.windows.net
Content-Length: 21
Expect: 100-continue

{"Name" : "NewName" }
```

## <a name="deleting-entities"></a>Eliminación de entidades
Las entidades pueden eliminarse en Media Services mediante una solicitud HTTP DELETE. Según la entidad, el orden con que se eliminan las entidades puede ser importante. Por ejemplo, entidades como Assets requieren que revoque (o elimine) todos los localizadores que hagan referencia a ese recurso específico antes de eliminar el recurso.

En el ejemplo siguiente se muestra cómo eliminar un localizador que se usó para cargar un archivo en el almacenamiento de blobs.

```console
DELETE https://media.windows.net/API/Locators('nb:lid:UUID:76dcc8e8-4230-463d-97b0-ce25c41b5c8d') HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: media.windows.net
Content-Length: 0
```

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
