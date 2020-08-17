---
title: Codificación de un recurso de Azure mediante Media Encoder Standard | Microsoft Docs
description: Aprenda a usar Media Encoder Standard para codificar el contenido multimedia en Azure Media Services. Los ejemplos de código usan la API de REST.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 2a7273c6-8a22-4f82-9bfe-4509ff32d4a4
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: e9cf5a65c664df321e8cee1c867e637a4718c9ee
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87837336"
---
# <a name="how-to-encode-an-asset-by-using-media-encoder-standard"></a>Codificación de un recurso mediante Media Encoder Standard
> [!div class="op_single_selector"]
> * [.NET](media-services-dotnet-encode-with-media-encoder-standard.md)
> * [REST](media-services-rest-encode-asset.md)
> * [Portal](media-services-portal-encode.md)
>
>

## <a name="overview"></a>Información general

Para entregar vídeo digital a través de Internet, debe comprimir los archivos multimedia. Los archivos de vídeo digital son grandes y pueden ser demasiado pesados para entregarlos a través de Internet o para que los dispositivos de sus clientes los muestren correctamente. La codificación es el proceso de compresión de vídeo y audio para que los clientes puedan ver el contenido multimedia.

Los trabajos de codificación son una de las operaciones de procesamiento más habituales en Azure Media Services. Los trabajos de codificación se crean para convertir archivos multimedia de una codificación a otra. Al codificar, puede usar el codificador integrado en Media Services (Media Encoder Standard). También puede usar un codificador proporcionado por un partner de Media Services. Los codificadores de terceros están disponibles a través de Azure Marketplace. Puede especificar los detalles de las tareas de codificación mediante cadenas preestablecidas definidas para el codificador o mediante archivos de configuración preestablecidos. Para consultar los tipos de valores preestablecidos disponibles, vea [Valores preestablecidos de tareas para el Codificador multimedia estándar](./media-services-mes-presets-overview.md).

Cada trabajo puede tener una o más tareas según el tipo de procesamiento que desee llevar a cabo. A través de la API de REST, puede crear trabajos y sus tareas relacionadas en una de las dos maneras siguientes:

* Las tareas se pueden definir en línea mediante la propiedad de navegación Tasks en entidades Job.
* Uso del procesamiento por lotes de OData.

Se recomienda codificar siempre los archivos fuente en un conjunto MP4 de velocidad de bits adaptable y, a continuación, convertirlo al formato deseado con el [empaquetado dinámico](media-services-dynamic-packaging-overview.md).

Si el recurso de salida tiene el almacenamiento cifrado, asegúrese de configurar la directiva de entrega de recursos. Para obtener más información, consulte [Configuración de directivas de entrega de recursos](media-services-rest-configure-asset-delivery-policy.md).

## <a name="considerations"></a>Consideraciones

Al obtener acceso a las entidades de Media Services, debe establecer los campos de encabezado específicos y los valores en las solicitudes HTTP. Para obtener más información, consulte [Configuración del desarrollo de la API de REST de Media Services](media-services-rest-how-to-use.md).

Antes de empezar a hacer referencia a procesadores de multimedia, compruebe que dispone del identificador de procesador de multimedia correcto. Para obtener más información, consulte [Obtención de procesadores multimedia](media-services-rest-get-media-processor.md).

## <a name="connect-to-media-services"></a>Conexión con Media Services

Para obtener más información sobre cómo conectarse a la API de Azure Media Services, consulte [Acceso a la API de Azure Media Services con la autenticación de Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

## <a name="create-a-job-with-a-single-encoding-task"></a>Creación de un trabajo con una sola tarea de codificación

> [!NOTE]
> Al trabajar con la API de REST de Media Services, se aplican las consideraciones siguientes:
>
> Al obtener acceso a las entidades de Media Services, debe establecer los campos de encabezado específicos y los valores en las solicitudes HTTP. Para obtener más información, consulte [Configuración del desarrollo de la API de REST de Media Services](media-services-rest-how-to-use.md).
>
> Cuando se usa JSON y se especifica el uso de la palabra clave **__metadata** en la solicitud (por ejemplo, para hacer referencia a un objeto vinculado) DEBE establecer el encabezado **Accept** en [formato JSON detallado](https://www.odata.org/documentation/odata-version-3-0/json-verbose-format/): Accept: application/json;odata=verbose.
>
>

En el ejemplo siguiente se muestra cómo crear y publicar un trabajo con un conjunto de tareas para codificar un vídeo con una resolución y calidad específicas. Al codificar con Media Encoder Standard, puede usar los valores preestablecidos de configuración de tareas especificados [aquí](./media-services-mes-presets-overview.md).

Solicitud:

```console
POST https://media.windows.net/API/Jobs HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
    Authorization: Bearer <ENCODED JWT TOKEN> 
    x-ms-client-request-id: 00000000-0000-0000-0000-000000000000
    Host: media.windows.net

{"Name" : "NewTestJob", "InputMediaAssets" : [{"__metadata" : {"uri" : "https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3Aaab7f15b-3136-4ddf-9962-e9ecb28fb9d2')"}}],  "Tasks" : [{"Configuration" : "Adaptive Streaming", "MediaProcessorId" : "nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",  "TaskBody" : "<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset>JobOutputAsset(0)</outputAsset></taskBody>"}]}
```

Respuesta:

```console
HTTP/1.1 201 Created

. . .
```

### <a name="set-the-output-assets-name"></a>Establecimiento del nombre del recurso de salida
En el ejemplo siguiente se muestra cómo establecer el atributo assetName:

```console
{ "TaskBody" : "<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset assetName=\"CustomOutputAssetName\">JobOutputAsset(0)</outputAsset></taskBody>"}`
```

## <a name="considerations"></a>Consideraciones
* Las propiedades TaskBody deben usar archivos XML literales para definir el número de recursos de entrada o salida que se usarán en la tarea. El artículo de la tarea contiene la definición de esquema XML del archivo XML.
* En la definición de TaskBody, cada valor interno para `<inputAsset>` y `<outputAsset>` se debe establecer como JobInputAsset(value) o JobOutputAsset(value).
* Una tarea puede tener varios recursos de salida. Un elemento JobOutputAsset(x) solo se puede usar una vez como salida de una tarea en un trabajo.
* Puede especificar JobInputAsset o JobOutputAsset como un recurso de entrada de una tarea.
* Las tareas no pueden formar un ciclo.
* El parámetro de valor que se pasa a JobInputAsset o JobOutputAsset representa el valor de índice para un recurso. Los recursos reales se definen en las propiedades de navegación InputMediaAssets y OutputMediaAssets en la definición de la entidad Job.
* Dado que Media Services se basa en OData v3, se hace referencia a los recursos individuales de las colecciones de propiedades de navegación InputMediaAssets y OutputMediaAssets a través de un par nombre-valor "__metadata : uri".
* InputMediaAssets se asigna a uno o más recursos que ha creado en Media Services. El sistema crea OutputMediaAssets. Estos no hacen referencia a ningún recurso existente.
* Se puede asignar un nombre a OutputMediaAssets con el atributo assetName. Si este atributo no está presente, el nombre de OutputMediaAsset será el valor del texto interno del elemento `<outputAsset>` con un sufijo del valor Job Name o del valor Job Id (en el caso de que no se haya definido la propiedad Name). Por ejemplo, si establece un valor para assetName como "Sample", se establece la propiedad de OutputMediaAsset Name en "Sample". Sin embargo, si no se ha definido un valor para assetName, pero se ha especificado el nombre del trabajo como "NewJob", OutputMediaAsset Name será "JobOutputAsset(value)_NewJob".

## <a name="create-a-job-with-chained-tasks"></a>Creación de un trabajo con tareas encadenadas
En muchos escenarios de aplicaciones, los desarrolladores desean crear una serie de tareas de procesamiento. En Media Services, puede crear una serie de tareas encadenadas. Cada tarea realiza distintos pasos de procesamiento diferentes y puede usar diferentes procesadores multimedia. Las tareas encadenadas pueden entregar un recurso de una tarea a otra, realizando una secuencia lineal de tareas en el recurso. Sin embargo, no es necesario que las tareas realizadas en un trabajo estén en una secuencia. Al crear una tarea encadenada, los objetos **ITask** encadenados se crean en un solo objeto **IJob**.

> [!NOTE]
> Actualmente hay un límite de 30 tareas por trabajo. Si necesita encadenar más de 30 tareas, cree más de un trabajo para incluir las tareas.
>
>

```console
POST https://media.windows.net/api/Jobs HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-client-request-id: 00000000-0000-0000-0000-000000000000

{  
   "Name":"NewTestJob",
   "InputMediaAssets":[  
      {  
         "__metadata":{  
            "uri":"https://testrest.cloudapp.net/api/Assets('nb%3Acid%3AUUID%3A910ffdc1-2e25-4b17-8a42-61ffd4b8914c')"
         }
      }
   ],
   "Tasks":[  
      {  
         "Configuration":"H264 Adaptive Bitrate MP4 Set 720p",
         "MediaProcessorId":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
         "TaskBody":"<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset>JobOutputAsset(0)</outputAsset></taskBody>"
      },
      {  
         "Configuration":"H264 Smooth Streaming 720p",
         "MediaProcessorId":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
         "TaskBody":"<?xml version=\"1.0\" encoding=\"utf-16\"?><taskBody><inputAsset>JobOutputAsset(0)</inputAsset><outputAsset>JobOutputAsset(1)</outputAsset></taskBody>"
      }
   ]
}
```

### <a name="considerations"></a>Consideraciones
Para habilitar el encadenamiento de tareas:

* Un trabajo debe tener al menos dos tareas.
* Debe haber al menos una tarea cuya entrada sea la salida de otra tarea del trabajo.

## <a name="use-odata-batch-processing"></a>Uso del procesamiento por lotes de OData
En el ejemplo siguiente se muestra cómo usar el procesamiento por lotes de OData para crear un trabajo y tareas. Para obtener información sobre el procesamiento por lotes, consulte [Procesamiento por lotes del protocolo Open Data (OData)](https://www.odata.org/documentation/odata-version-3-0/batch-processing/).

```console
POST https://media.windows.net/api/$batch HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Content-Type: multipart/mixed; boundary=batch_a01a5ec4-ba0f-4536-84b5-66c5a5a6d34e
Accept: multipart/mixed
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
x-ms-client-request-id: 00000000-0000-0000-0000-000000000000
Host: media.windows.net


--batch_a01a5ec4-ba0f-4536-84b5-66c5a5a6d34e
Content-Type: multipart/mixed; boundary=changeset_122fb0a4-cd80-4958-820f-346309967e4d

--changeset_122fb0a4-cd80-4958-820f-346309967e4d
Content-Type: application/http
Content-Transfer-Encoding: binary

POST https://media.windows.net/api/Jobs HTTP/1.1
Content-ID: 1
Content-Type: application/json
Accept: application/json
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
x-ms-client-request-id: 00000000-0000-0000-0000-000000000000

{"Name" : "NewTestJob", "InputMediaAssets@odata.bind":["https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3A2a22445d-1500-80c6-4b34-f1e5190d33c6')"]}

--changeset_122fb0a4-cd80-4958-820f-346309967e4d
Content-Type: application/http
Content-Transfer-Encoding: binary

POST https://media.windows.net/api/$1/Tasks HTTP/1.1
Content-ID: 2
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
x-ms-client-request-id: 00000000-0000-0000-0000-000000000000

{  
   "Configuration":"H264 Adaptive Bitrate MP4 Set 720p",
   "MediaProcessorId":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
   "TaskBody":"<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset assetName=\"Custom output name\">JobOutputAsset(0)</outputAsset></taskBody>"
}

--changeset_122fb0a4-cd80-4958-820f-346309967e4d--
--batch_a01a5ec4-ba0f-4536-84b5-66c5a5a6d34e--
```


## <a name="create-a-job-by-using-a-jobtemplate"></a>Creación de un trabajo mediante una plantilla JobTemplate
Al procesar varios recursos usando un conjunto común de tareas, utilice una plantilla JobTemplate para especificar los valores preestablecidos de las tareas predeterminadas o establezca el orden de las tareas.

En el ejemplo siguiente se muestra cómo crear una plantilla JobTemplate con una plantilla TaskTemplate definida en línea. La plantilla TaskTemplate usa Media Encoder Standard como elemento MediaProcessor para codificar el archivo de recursos. Sin embargo, también se pueden usar otros elementos MediaProcessor.

```console
POST https://media.windows.net/API/JobTemplates HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
Host: media.windows.net


{"Name" : "NewJobTemplate25", "JobTemplateBody" : "<?xml version=\"1.0\" encoding=\"utf-8\"?><jobTemplate><taskBody taskTemplateId=\"nb:ttid:UUID:071370A3-E63E-4E81-A099-AD66BCAC3789\"><inputAsset>JobInputAsset(0)</inputAsset><outputAsset>JobOutputAsset(0)</outputAsset></taskBody></jobTemplate>", "TaskTemplates" : [{"Id" : "nb:ttid:UUID:071370A3-E63E-4E81-A099-AD66BCAC3789", "Configuration" : "H264 Smooth Streaming 720p", "MediaProcessorId" : "nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56", "Name" : "SampleTaskTemplate2", "NumberofInputAssets" : 1, "NumberofOutputAssets" : 1}] }
```

> [!NOTE]
> A diferencia de otras entidades de Media Services, debe definir un nuevo identificador GUID para cada plantilla de tarea y colocarlo en el identificador de la plantilla de tarea y en la propiedad Id en el cuerpo de la solicitud. El esquema de identificación de contenido debe seguir el esquema descrito en Identificación de entidades de Azure Media Services. Además, las plantillas de trabajo no se pueden actualizar. Por el contrario, debe crear una nueva con los cambios actualizados.
>
>

Si se realiza correctamente, se devuelve la respuesta siguiente:

```console
HTTP/1.1 201 Created

. . .
```

En el ejemplo siguiente se muestra cómo crear un trabajo que hace referencia a un identificador de plantilla JobTemplate:

```console
POST https://media.windows.net/API/Jobs HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
Host: media.windows.net


{"Name" : "NewTestJob", "InputMediaAssets" : [{"__metadata" : {"uri" : "https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3A3f1fe4a2-68f5-4190-9557-cd45beccef92')"}}], "TemplateId" : "nb:jtid:UUID:15e6e5e6-ac85-084e-9dc2-db3645fbf0aa"}
```

Si se realiza correctamente, se devuelve la respuesta siguiente:

```console
HTTP/1.1 201 Created

. . .
```


## <a name="advanced-encoding-features-to-explore"></a>Características avanzadas de codificación para explorar
* [Cómo generar vistas en miniatura](media-services-dotnet-generate-thumbnail-with-mes.md)
* [Generación de miniaturas durante la codificación](media-services-dotnet-generate-thumbnail-with-mes.md#example-of-generating-a-thumbnail-while-encoding)
* [Recorte de vídeos durante la codificación](media-services-crop-video.md)
* [Personalización de valores preestablecidos de codificación](media-services-custom-mes-presets-with-dotnet.md)
* [Superposición o marca de agua de un vídeo con una imagen](media-services-advanced-encoding-with-mes.md#overlay)

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>Pasos siguientes
Ahora que sabe cómo crear un trabajo para codificar un recurso, consulte [Comprobación del progreso del trabajo con Media Services](media-services-rest-check-job-progress.md).

## <a name="see-also"></a>Consulte también
[Obtención de una instancia de procesador multimedia](media-services-rest-get-media-processor.md)