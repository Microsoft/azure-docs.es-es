---
title: Introducción a la entrega de contenido a petición mediante REST | Microsoft Docs
description: Este tutorial le guiará por los pasos necesarios para implementar una aplicación de entrega de contenido a petición con Azure Media Services mediante la API de REST.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 88194b59-e479-43ac-b179-af4f295e3780
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: 8c15883f4e2a515e8c6015a39951a8729cf77274
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87544003"
---
# <a name="get-started-with-delivering-content-on-demand-using-rest"></a>Introducción a la entrega de contenido a petición mediante REST  

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

Esta guía de inicio rápido le guiará por los pasos necesarios para implementar una aplicación de entrega de contenido de vídeo bajo demanda (VoD) mediante API de REST de Azure Media Services (AMS).

Presenta el flujo de trabajo básico de Media Services y la mayoría de los objetos y tareas de programación más comunes necesarios para el desarrollo de Media Services. Al término del tutorial, puede transmitir o descargar progresivamente un archivo multimedia de ejemplo que cargó, codificó y descargó.

En la ilustración siguiente se muestran algunos de los objetos que se utilizan con más frecuencia al desarrollar aplicaciones de VoD con el modelo OData de Media Services.

Haga clic en la imagen para verla a tamaño completo.  

[![Diagrama que muestra algunos de los objetos que se usan con más frecuencia en el modelo de datos de objetos de Azure Media Services para desarrollar aplicaciones de vídeo bajo demanda.](./media/media-services-rest-get-started/media-services-overview-object-model-small.png)](./media/media-services-rest-get-started/media-services-overview-object-model.png#lightbox)

## <a name="prerequisites"></a>Prerrequisitos
Los siguientes requisitos previos son necesarios para empezar a desarrollar con las API de REST de Media Services.

* Una cuenta de Azure. Para obtener más información, consulte [Evaluación gratuita de Azure](https://azure.microsoft.com/pricing/free-trial/).
* Una cuenta de Media Services. Para crear una cuenta de Media Services, consulte el tema [Creación de una cuenta de Media Services](media-services-portal-create-account.md).
* Información sobre cómo desarrollar con las API de REST de Media Services. Para más información, consulte [Información general sobre la API de REST de Media Services](media-services-rest-how-to-use.md).
* Una aplicación de su elección que puede enviar solicitudes y respuestas HTTP. En este tutorial se usa [Fiddler](https://www.telerik.com/download/fiddler).

En este tutorial rápido se muestran las siguientes tareas.

1. Inicio de punto de conexión de streaming (mediante Azure Portal).
2. Conexión a la cuenta de Media Services con API de REST
3. Creación de un nuevo recurso y carga de un archivo de vídeo con la API de REST
4. Codificación del archivo de origen en un conjunto de archivos MP4 de velocidad de bits adaptable con API de REST
5. Publicación del recurso y obtención de direcciones URL de streaming y de descarga progresiva con API de REST
6. Reproduzca el contenido.

>[!NOTE]
>Hay un límite de 1 000 000 directivas para diferentes directivas de AMS (por ejemplo, para la directiva de localizador o ContentKeyAuthorizationPolicy). Use el mismo identificador de directiva si siempre usa los mismos permisos de acceso y días, por ejemplo, directivas para localizadores que vayan a aplicarse durante mucho tiempo (directivas distintas a carga). Para obtener más información, consulte [este](media-services-dotnet-manage-entities.md#limit-access-policies) artículo.

Para más información acerca de las entidades de AMS REST utilizadas en este artículo, consulte [Referencia de la API de REST de Azure Media Services](/rest/api/media/operations/azure-media-services-rest-api-reference). Consulte también [Conceptos de Azure Media Services](media-services-concepts.md).

>[!NOTE]
>Al obtener acceso a las entidades de Media Services, debe establecer los campos de encabezado específicos y los valores en las solicitudes HTTP. Para obtener más información, consulte [Configuración del desarrollo de la API de REST de Media Services](media-services-rest-how-to-use.md).

## <a name="start-streaming-endpoints-using-the-azure-portal"></a>Inicio de los puntos de conexión de streaming con Azure Portal

Cuando se trabaja con Azure Media Services, uno de los escenarios más comunes es entregar contenido de vídeo a los clientes mediante streaming con velocidad de bits adaptable. Media Services proporciona empaquetado dinámico que permite entregar contenido codificado MP4 con velocidad de bits adaptable en formatos de streaming admitidos por Media Services (MPEG DASH, HLS, Smooth Streaming) justo a tiempo, sin tener que almacenar versiones previamente empaquetadas de cada uno de estos formatos.

>[!NOTE]
>Cuando se crea la cuenta de AMS, se agrega un punto de conexión de streaming **predeterminado** a la cuenta en estado **Stopped** (Detenido). Para iniciar la transmisión del contenido y aprovechar el empaquetado dinámico y el cifrado dinámico, el punto de conexión de streaming desde el que va a transmitir el contenido debe estar en estado **Running** (En ejecución).

Para iniciar el punto de conexión de streaming, haga lo siguiente:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. En la ventana Settings (Configuración), haga clic en Streaming endpoints (Puntos de conexión de streaming).
3. Haga clic en el punto de conexión de streaming predeterminado.

    Aparecerá la ventana de detalles del punto de conexión de streaming predeterminado.

4. Haga clic en el icono de inicio.
5. Haga clic en el botón Save (Guardar) para guardar los cambios.

## <a name="connect-to-the-media-services-account-with-rest-api"></a><a id="connect"></a>Conexión a la cuenta de Media Services con API de REST

Para obtener más información sobre cómo conectarse a la API de Azure Media Services, consulte [Acceso a la API de Azure Media Services con la autenticación de Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

## <a name="create-a-new-asset-and-upload-a-video-file-with-rest-api"></a><a id="upload"></a>Creación de un nuevo recurso y carga de un archivo de vídeo con la API de REST

En Media Services, cargue los archivos digitales en un recurso. La entidad **Asset** puede contener archivos de vídeo, audio, imágenes, colecciones de miniaturas, pistas de texto y subtítulos (y los metadatos sobre estos archivos).  Una vez cargados los archivos en el recurso, el contenido se almacena de forma segura en la nube para un posterior procesamiento y streaming.

Uno de los valores que se deben proporcionar al crear un recurso son las opciones de creación de activos. La propiedad **Options** es un valor de enumeración que describe las opciones de cifrado con las que se puede crear un recurso. Un valor válido es uno de los valores de la lista siguiente, no una combinación de valores de esta lista:

* **None** = **0**: no se utiliza cifrado. Cuando se utiliza esta opción, el contenido no está protegido en tránsito o en reposo en el almacenamiento.
    Si tiene previsto entregar un MP4 mediante una descarga progresiva, utilice esta opción.
* **StorageEncrypted** = **1**: cifra el contenido no cifrado localmente mediante el cifrado AES de 256 bits y, a continuación, lo carga en Azure Storage, donde se almacena cifrado en reposo. Los recursos protegidos con el cifrado de almacenamiento se descifran automáticamente y se colocan en un sistema de archivos cifrados antes de la codificación y, opcionalmente, se vuelven a cifrar antes de volver a cargarlos como un nuevo recurso de salida. El caso de uso principal para el cifrado de almacenamiento es cuando desea proteger los archivos multimedia de entrada de alta calidad con un sólido cifrado en reposo en disco.
* **CommonEncryptionProtected** = **2**: use esta opción si va a cargar contenido que ya se cifró y protegió con cifrado común o DRM de PlayReady (por ejemplo, Smooth Streaming protegido con DRM de PlayReady).
* **EnvelopeEncryptionProtected** = **4**: use esta opción si va a cargar HLS cifrado con AES. Los archivos deben haberse codificado y cifrado con Transform Manager.

### <a name="create-an-asset"></a>Creación de un recurso
Un recurso es un contenedor para varios tipos o conjuntos de objetos en Media Services, como vídeo, audio, imágenes, colecciones de miniaturas, pistas de texto y archivos de subtítulos. En la API de REST, crear un recurso requiere el envío de una solicitud POST a Media Services y colocar la información sobre cualquier propiedad del recurso en el cuerpo de solicitud.

En el ejemplo siguiente se muestra cómo crear un recurso.

**Solicitud HTTP**

```console
POST https://wamsbayclus001rest-hs.cloudapp.net/api/Assets HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
x-ms-client-request-id: c59de965-bc89-4295-9a57-75d897e5221e
Host: wamsbayclus001rest-hs.cloudapp.net
Content-Length: 45

{"Name":"BigBuckBunny.mp4", "Options":"0"}
```

**Respuesta HTTP**

Si se realiza correctamente, se devuelve lo siguiente:

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 452
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Assets('nb%3Acid%3AUUID%3A9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1')
Server: Microsoft-IIS/8.5
x-ms-client-request-id: c59de965-bc89-4295-9a57-75d897e5221e
request-id: e98be122-ae09-473a-8072-0ccd234a0657
x-ms-request-id: e98be122-ae09-473a-8072-0ccd234a0657
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Sun, 18 Jan 2015 22:06:40 GMT

{  
    odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Assets/@Element",
   "Id":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
   "State":0,
   "Created":"2015-01-18T22:06:40.6010903Z",
   "LastModified":"2015-01-18T22:06:40.6010903Z",
   "AlternateId":null,
   "Name":"BigBuckBunny2.mp4",
   "Options":0,
   "Uri":"https://storagetestaccount001.blob.core.windows.net/asset-9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
   "StorageAccountName":"storagetestaccount001"
}
```

### <a name="create-an-assetfile"></a>Creación de AssetFile
La entidad [AssetFile](/rest/api/media/operations/assetfile) representa un archivo de audio o vídeo que se almacena en un contenedor de blobs. Un archivo de recursos siempre está asociado a un recurso y un recurso puede contener uno o varios archivos de recursos. La tarea de Media Services produce un error si un objeto de archivo de recursos no está asociado a un archivo digital de un contenedor de blobs.

Después de cargar el archivo multimedia digital en un contenedor de blobs, usará la solicitud HTTP **MERGE** para actualizar la entidad AssetFile con información sobre el archivo multimedia (tal como se muestra más adelante en este tema).

**Solicitud HTTP**

```console
POST https://wamsbayclus001rest-hs.cloudapp.net/api/Files HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
Content-Length: 164

{  
   "IsEncrypted":"false",
   "IsPrimary":"false",
   "MimeType":"video/mp4",
   "Name":"BigBuckBunny.mp4",
   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1"
}
```

**Respuesta HTTP**

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 535
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Files('nb%3Acid%3AUUID%3Af13a0137-0a62-9d4c-b3b9-ca944b5142c5')
Server: Microsoft-IIS/8.5
request-id: 98a30e2d-f379-4495-988e-0b79edc9b80e
x-ms-request-id: 98a30e2d-f379-4495-988e-0b79edc9b80e
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Mon, 19 Jan 2015 00:34:07 GMT

{  
   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Files/@Element",
   "Id":"nb:cid:UUID:f13a0137-0a62-9d4c-b3b9-ca944b5142c5",
   "Name":"BigBuckBunny.mp4",
   "ContentFileSize":"0",
   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
   "EncryptionVersion":null,
   "EncryptionScheme":null,
   "IsEncrypted":false,
   "EncryptionKeyId":null,
   "InitializationVector":null,
   "IsPrimary":false,
   "LastModified":"2015-01-19T00:34:08.1934137Z",
   "Created":"2015-01-19T00:34:08.1934137Z",
   "MimeType":"video/mp4",
   "ContentChecksum":null
}
```

### <a name="creating-the-accesspolicy-with-write-permission"></a>Creación de AccessPolicy con permiso de escritura
Antes de cargar los archivos en el almacenamiento de blobs, establezca los derechos de la directiva de acceso para escribir en un recurso. Para ello, envíe una solicitud HTTP al conjunto de entidades AccessPolicies. Defina un valor DurationInMinutes tras la creación o recibirá un mensaje de error de servidor interno 500 como respuesta. Para obtener más información sobre AccessPolicies, consulte [AccessPolicy](/rest/api/media/operations/accesspolicy).

En el ejemplo siguiente se muestra cómo crear una entidad AccessPolicy:

**Solicitud HTTP**

```console
POST https://wamsbayclus001rest-hs.cloudapp.net/api/AccessPolicies HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
Content-Length: 74

{"Name":"NewUploadPolicy", "DurationInMinutes":"440", "Permissions":"2"}
```

**Respuesta HTTP**

Si se realiza correctamente, se devuelve la respuesta siguiente:

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 312
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: https://wamsbayclus001rest-hs.cloudapp.net/api/AccessPolicies('nb%3Apid%3AUUID%3Abe0ac48d-af7d-4877-9d60-1805d68bffae')
Server: Microsoft-IIS/8.5
request-id: 74c74545-7e0a-4cd6-a440-c1c48074a970
x-ms-request-id: 74c74545-7e0a-4cd6-a440-c1c48074a970
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Sun, 18 Jan 2015 22:18:06 GMT

{  
   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#AccessPolicies/@Element",
   "Id":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
   "Created":"2015-01-18T22:18:06.6370575Z",
   "LastModified":"2015-01-18T22:18:06.6370575Z",
   "Name":"NewUploadPolicy",
   "DurationInMinutes":440.0,
   "Permissions":2
}
```

### <a name="get-the-upload-url"></a>Obtención de la dirección URL de carga

Para recibir la dirección URL de carga real, cree un localizador de SAS. Los localizadores definen la hora de inicio y el tipo de extremo de conexión para los clientes que desean tener acceso a archivos de un recurso. Puede crear varias entidades Locator para un par AccessPolicy y Asset determinado a fin de controlar las distintas solicitudes y necesidades del cliente. Cada uno de estos localizadores usa el valor StartTime más el valor DurationInMinutes de AccessPolicy para determinar la cantidad de tiempo que se puede usar una dirección URL. Para obtener más información, consulte [Localizador](/rest/api/media/operations/locator).

Una dirección URL de SAS tiene el formato siguiente:

`{https://myaccount.blob.core.windows.net}/{asset name}/{video file name}?{SAS signature}`

Se aplican algunas consideraciones:

* No puede tener más de cinco localizadores únicos asociados a un recurso determinado a la vez. 
* Si necesita cargar los archivos inmediatamente, debe establecer el valor StartTime cinco minutos antes de la hora actual. Esto se debe a que puede haber un desplazamiento de reloj entre el equipo cliente y Media Services. Además, el valor de StartTime debe estar en el siguiente formato de fecha/hora: AAAA-MM-DDTHH (por ejemplo, "2014-05-23T17:53:50Z").    
* Puede haber un retraso de 30 a 40 segundos desde el momento en que se crea un localizador hasta que este está disponible para su uso. Este problema afecta a la [dirección URL de SAS](../../storage/common/storage-sas-overview.md) y a los localizadores de origen.

En el ejemplo siguiente se muestra cómo crear un localizador de dirección URL de SAS, tal como se define en la propiedad Type del cuerpo de solicitud ("1" para un localizador de SAS y "2" para un localizador de origen bajo demanda). La propiedad **Path** devuelta contiene la dirección URL que debe usar para cargar el archivo.

**Solicitud HTTP**

```console
POST https://wamsbayclus001rest-hs.cloudapp.net/api/Locators HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
Content-Length: 178

{  
   "AccessPolicyId":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
   "AssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
   "StartTime":"2015-02-18T16:45:53",
   "Type":1
}
```

**Respuesta HTTP**

Si se realiza correctamente, se devuelve la respuesta siguiente:

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 949
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Locators('nb%3Alid%3AUUID%3Aaf57bdd8-6751-4e84-b403-f3c140444b54')
Server: Microsoft-IIS/8.5
request-id: 2adeb1f8-89c5-4cc8-aa4f-08cdfef33ae0
x-ms-request-id: 2adeb1f8-89c5-4cc8-aa4f-08cdfef33ae0
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Mon, 19 Jan 2015 03:01:29 GMT

{  
   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Locators/@Element",
   "Id":"nb:lid:UUID:af57bdd8-6751-4e84-b403-f3c140444b54",
   "ExpirationDateTime":"2015-02-19T00:05:53",
   "Type":1,
   "Path":"https://storagetestaccount001.blob.core.windows.net/asset-f438649c-313c-46e2-8d68-7d2550288247?sv=2012-02-12&sr=c&si=af57bdd8-6751-4e84-b403-f3c140444b54&sig=fE4btwEfZtVQFeC0Wh3Kwks2OFPQfzl5qTMW5YytiuY%3D&st=2015-02-18T16%3A45%3A53Z&se=2015-02-19T00%3A05%3A53Z",
   "BaseUri":"https://storagetestaccount001.blob.core.windows.net/asset-f438649c-313c-46e2-8d68-7d2550288247",
   "ContentAccessComponent":"?sv=2012-02-12&sr=c&si=af57bdd8-6751-4e84-b403-f3c140444b54&sig=fE4btwEfZtVQFeC0Wh3Kwks2OFPQfzl5qTMW5YytiuY%3D&st=2015-02-18T16%3A45%3A53Z&se=2015-02-19T00%3A05%3A53Z",
   "AccessPolicyId":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
   "AssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
   "StartTime":"2015-02-18T16:45:53",
   "Name":null
}
```

### <a name="upload-a-file-into-a-blob-storage-container"></a>Carga de un archivo en un contenedor de almacenamiento de blobs
Una vez establecidas las propiedades AccessPolicy y Locator, el archivo real se carga en un contenedor de Azure Blob Storage mediante las API de REST de Azure Storage. Debe cargar los archivos como blobs en bloques. Azure Media Services no admite los blobs en páginas.  

> [!NOTE]
> Debe agregar el nombre de archivo para el archivo que desea cargar en el valor **Path** del localizador recibido en la sección anterior. Por ejemplo, `https://storagetestaccount001.blob.core.windows.net/asset-e7b02da4-5a69-40e7-a8db-e8f4f697aac0/BigBuckBunny.mp4?`.
>
>

Para obtener más información sobre cómo trabajar con blobs de Almacenamiento de Azure, consulte [API de REST del servicio Blob](/rest/api/storageservices/blob-service-rest-api).

### <a name="update-the-assetfile"></a>Actualización de AssetFile
Ahora que ha cargado el archivo, actualice la información de tamaño de FileAsset (y otro tipo de información). Por ejemplo:

```console
MERGE https://wamsbayclus001rest-hs.cloudapp.net/api/Files('nb%3Acid%3AUUID%3Af13a0137-0a62-9d4c-b3b9-ca944b5142c5') HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net

{  
   "ContentFileSize":"1186540",
   "Id":"nb:cid:UUID:f13a0137-0a62-9d4c-b3b9-ca944b5142c5",
   "MimeType":"video/mp4",
   "Name":"BigBuckBunny.mp4",
   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1"
}
```

**Respuesta HTTP**

Si se realiza correctamente, se devuelve lo siguiente:

```console
HTTP/1.1 204 No Content
...
```

## <a name="delete-the-locator-and-accesspolicy"></a>Eliminación de Locator y AccessPolicy
**Solicitud HTTP**

```console
DELETE https://wamsbayclus001rest-hs.cloudapp.net/api/Locators('nb%3Alid%3AUUID%3Aaf57bdd8-6751-4e84-b403-f3c140444b54') HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
```

**Respuesta HTTP**

Si se realiza correctamente, se devuelve lo siguiente:

```console
HTTP/1.1 204 No Content
...
```

**Solicitud HTTP**

```console
DELETE https://wamsbayclus001rest-hs.cloudapp.net/api/AccessPolicies('nb%3Apid%3AUUID%3Abe0ac48d-af7d-4877-9d60-1805d68bffae') HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
```

**Respuesta HTTP**

Si se realiza correctamente, se devuelve lo siguiente:

```console
HTTP/1.1 204 No Content
...
```

## <a name="encode-the-source-file-into-a-set-of-adaptive-bitrate-mp4-files"></a><a id="encode"></a>Codificación del archivo de origen en un conjunto de archivos MP4 de velocidad de bits adaptativa

Después de introducir los recursos en Media Services, los elementos multimedia se pueden codificar, transmultiplexar, agregar una marca de agua, entre otras opciones, antes de entregarse a los clientes. Estas actividades se programan y se ejecutan en varias instancias de rol en segundo plano para garantizar la disponibilidad y alto rendimiento. Estas actividades se denominan trabajos y cada trabajo está compuesto de tareas atómicas que realizan el trabajo real en el archivo del recurso (para más información, consulte las descripciones de [trabajo](/rest/api/media/operations/job) y [tarea](/rest/api/media/operations/task)).

Como se ha indicado antes, cuando se trabaja con Azure Media Services, uno de los escenarios más comunes es ofrecer streaming de velocidad de bits adaptable a los clientes. Media Services puede empaquetar de manera dinámica un conjunto de archivos MP4 de velocidad de bits adaptable en uno de los siguientes formatos: HTTP Live Streaming (HLS), Smooth Streaming, MPEG DASH.

En la sección siguiente se muestra cómo crear un trabajo que contiene una tarea de codificación. La tarea especifica la transcodificación del archivo intermedio en un conjunto de archivos MP4 de velocidad de bits adaptable con **Codificador multimedia estándar**. La sección también muestra cómo supervisar el progreso del procesamiento de trabajo. Una vez finalizado el trabajo, podría crear los localizadores necesarios para acceder a los recursos.

### <a name="get-a-media-processor"></a>Obtención de un procesador multimedia
En los Media Services, un procesador multimedia es un componente que controla una tarea de procesamiento específica, como codificación, conversión de formato, cifrado o descifrado de contenido multimedia. Para la tarea de codificación que se muestra en este tutorial, usaremos el Codificador multimedia estándar.

El código siguiente solicita el identificador del codificador.

**Solicitud HTTP**

```console
    GET https://wamsbayclus001rest-hs.cloudapp.net/api/MediaProcessors()?$filter=Name%20eq%20'Media%20Encoder%20Standard' HTTP/1.1
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    Authorization: Bearer <ENCODED JWT TOKEN>
    x-ms-version: 2.19
    Host: wamsbayclus001rest-hs.cloudapp.net
```

**Respuesta HTTP**

```console
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 273
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Server: Microsoft-IIS/8.5
request-id: 6beb04b4-55a7-480d-8aa8-e5c5d59ffa1f
x-ms-request-id: 6beb04b4-55a7-480d-8aa8-e5c5d59ffa1f
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Mon, 19 Jan 2015 07:54:09 GMT

{  
   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#MediaProcessors",
   "value":[  
      {  
         "Id":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
         "Description":"Media Encoder Standard",
         "Name":"Media Encoder Standard",
         "Sku":"",
         "Vendor":"Microsoft",
         "Version":"1.1"
      }
   ]
}
```

### <a name="create-a-job"></a>Creación de un trabajo
Cada trabajo puede tener una o más tareas según el tipo de procesamiento que desee llevar a cabo. A través de la API de REST, puede crear trabajos y sus tareas relacionadas en una de las dos maneras siguientes: Las tareas se pueden definir en línea mediante la propiedad de navegación de las tareas en entidades de trabajo o a través del procesamiento por lotes de OData. El SDK de Media Services usa el procesamiento por lotes. Sin embargo, para que sea fácil entender los ejemplos de código de este artículo, las tareas se definen en línea. Para obtener información sobre el procesamiento por lotes, consulte [Procesamiento por lotes del protocolo Open Data (OData)](https://www.odata.org/documentation/odata-version-3-0/batch-processing/).

En el ejemplo siguiente se muestra cómo crear y publicar un trabajo con un conjunto de tareas para codificar un vídeo con una resolución y calidad específicas. La siguiente sección de la documentación contiene la lista de todos los [valores preestablecidos de tareas](/azure/media-services/previous/media-services-mes-presets-overview) admitidos por el procesador de Codificador multimedia estándar.  

**Solicitud HTTP**

```console
POST https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
Content-Length: 482

{  
   "Name":"NewTestJob",
   "InputMediaAssets":[  
      {  
         "__metadata":{  
            "uri":"https://wamsbayclus001rest-hs.net/api/Assets('nb%3Acid%3AUUID%3A9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1')"
         }
      }
   ],
   "Tasks":[  
      {  
         "Configuration":"Adaptive Streaming",
         "MediaProcessorId":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
         "TaskBody":"<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset>
            <outputAsset>JobOutputAsset(0)</outputAsset></taskBody>"
      }
   ]
}
```

**Respuesta HTTP**

Si se realiza correctamente, se devuelve la respuesta siguiente:

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 1215
Content-Type: application/json;odata=verbose;charset=utf-8
Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')
Server: Microsoft-IIS/8.5
request-id: 532ac1ec-a475-4dce-b2d5-7c8ce94ac87c
x-ms-request-id: 532ac1ec-a475-4dce-b2d5-7c8ce94ac87c
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Mon, 19 Jan 2015 08:04:35 GMT

{  
   "d":{  
      "__metadata":{  
         "id":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')",
         "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')",
         "type":"Microsoft.Cloud.Media.Vod.Rest.Data.Models.Job"
      },
      "Tasks":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/Tasks"
         }
      },
      "OutputMediaAssets":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/OutputMediaAssets"
         }
      },
      "InputMediaAssets":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1')/InputMediaAssets"
         }
      },
      "Id":"nb:jid:UUID:71d2dd33-efdf-ec43-8ea1-136a110bd42c",
      "Name":"NewTestJob",
      "Created":"2015-01-19T08:04:34.3287057Z",
      "LastModified":"2015-01-19T08:04:34.3287057Z",
      "EndTime":null,
      "Priority":0,
      "RunningDuration":0,
      "StartTime":null,
      "State":0,
      "TemplateId":null,
      "JobNotificationSubscriptions":{  
         "__metadata":{  
            "type":"Collection(Microsoft.Cloud.Media.Vod.Rest.Data.Models.JobNotificationSubscription)"
         },
         "results":[  

         ]
      }
   }
}
```

Hay algunas cuestiones importantes a tener en cuenta en cualquier solicitud de trabajo:

* Las propiedades TaskBody deben usar archivos XML literales para definir el número de recursos de entrada o salida que se usarán en la tarea. El artículo de la tarea contiene la definición de esquema XML del archivo XML.
* En la definición de TaskBody, cada valor interno para `<inputAsset>` y `<outputAsset>` se debe establecer como JobInputAsset(value) o JobOutputAsset(value).
* Una tarea puede tener varios recursos de salida. Un elemento JobOutputAsset(x) solo se puede usar una vez como salida de una tarea en un trabajo.
* Puede especificar JobInputAsset o JobOutputAsset como un recurso de entrada de una tarea.
* Las tareas no pueden formar un ciclo.
* El parámetro de valor que se pasa a JobInputAsset o JobOutputAsset representa el valor de índice para un recurso. Los recursos reales se definen en las propiedades de navegación InputMediaAssets y OutputMediaAssets en la definición de la entidad Job.

> [!NOTE]
> Dado que Media Services se basa en OData v3, se hace referencia a los recursos individuales de las colecciones de propiedades de navegación InputMediaAssets y OutputMediaAssets a través de un par nombre-valor "__metadata: uri".
>
>

* InputMediaAssets se asigna a uno o más recursos que ha creado en Media Services. El sistema crea OutputMediaAssets. Estos no hacen referencia a ningún recurso existente.
* Se puede asignar un nombre a OutputMediaAssets con el atributo assetName. Si este atributo no está presente, el nombre de OutputMediaAsset será el valor del texto interno del elemento `<outputAsset>` con un sufijo del valor Job Name o del valor Job Id (en el caso de que no se haya definido la propiedad Name). Por ejemplo, si establece un valor para assetName como "Sample", se establecería la propiedad de OutputMediaAsset Name en "Sample". Sin embargo, si no se ha definido un valor para assetName, pero se ha especificado el nombre del trabajo como "NewJob", OutputMediaAsset Name será "JobOutputAsset (value) _NewJob".

    En el ejemplo siguiente se muestra cómo establecer el atributo assetName:

    ```console
    "<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset assetName=\"CustomOutputAssetName\">JobOutputAsset(0)</outputAsset></taskBody>"
    ```
* Para habilitar el encadenamiento de tareas:

  * Un trabajo debe tener al menos dos tareas.
  * Debe haber al menos una tarea cuya entrada sea salida de otra tarea del trabajo.

Para obtener más información, consulte [Creación de un trabajo de Encoding con la API de REST de Media Services](media-services-rest-encode-asset.md).

### <a name="monitor-processing-progress"></a>Supervisión del progreso del procesamiento
Puede recuperar el estado del trabajo mediante la propiedad State, tal como se muestra en el ejemplo siguiente:

**Solicitud HTTP**

```console
GET https://wamsbayclus001rest-hs.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/State HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: wamsbayclus001rest-hs.net
Content-Length: 0
```

**Respuesta HTTP**

Si se realiza correctamente, se devuelve la respuesta siguiente:

```console
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 17
Content-Type: application/json;odata=verbose;charset=utf-8
Server: Microsoft-IIS/7.5
x-ms-request-id: 01324d81-18e2-4493-a3e5-c6186209f0c8
X-Content-Type-Options: nosniff
DataServiceVersion: 1.0;
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
Date: Sun, 13 May 2012 05:16:53 GMT

{"d":{"State":2}}
```

### <a name="cancel-a-job"></a>Cancelación de un trabajo
Media Services le permite cancelar trabajos en ejecución a través de la función CancelJob. Esta llamada devolverá un código de error 400 si intenta cancelar un trabajo cuando su estado sea cancelado, cancelando, error o finalizado.

En el ejemplo siguiente se muestra cómo llamar a CancelJob.

**Solicitud HTTP**

```console
GET https://wamsbayclus001rest-hs.net/API/CancelJob?jobid='nb%3ajid%3aUUID%3a71d2dd33-efdf-ec43-8ea1-136a110bd42c' HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: wamsbayclus001rest-hs.net
```

Si se realiza correctamente, se devuelve un código de respuesta 204 sin cuerpo del mensaje.

> [!NOTE]
> Debe codificar con URL el identificador del trabajo (normalmente, nb:jid:UUID: somevalue) al pasarlo como parámetro a CancelJob.
>
>

### <a name="get-the-output-asset"></a>Obtención del resultado de salida
En el código siguiente se muestra cómo solicitar el identificador del recurso de salida.

**Solicitud HTTP**

```console
GET https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/OutputMediaAssets() HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
User-Agent: Microsoft ADO.NET Data Services
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
```

**Respuesta HTTP**

```console
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 445
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Server: Microsoft-IIS/8.5
request-id: 73cd605d-066a-46f1-8358-f4bd25a9220a
x-ms-request-id: 73cd605d-066a-46f1-8358-f4bd25a9220a
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Mon, 19 Jan 2015 08:28:13 GMT

{  
   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Assets",
   "value":[  
      {  
         "Id":"nb:cid:UUID:71d2dd33-efdf-ec43-8ea1-136a110bd42c",
         "State":0,
         "Created":"2015-01-19T07:52:15.603",
         "LastModified":"2015-01-19T07:52:15.603",
         "AlternateId":null,
         "Name":"Multibitrate MP4s",
         "Options":0,
         "Uri":"https://storagetestaccount001.blob.core.windows.net/asset-71d2dd33-efdf-ec43-8ea1-136a110bd42c",
         "StorageAccountName":"storagetestaccount001"
      }
   ]
}
```

## <a name="publish-the-asset-and-get-streaming-and-progressive-download-urls-with-rest-api"></a><a id="publish_get_urls"></a>Publicación del recurso y obtención de direcciones URL de streaming y de descarga progresiva con API de REST

Para transmitir o descargar un recurso, necesita "publicarlo" mediante la creación de un localizador. Los localizadores proporcionan acceso a los archivos contenidos en el recurso. Media Services admite dos tipos de localizadores: OnDemandOrigin, usados para transmitir contenido (por ejemplo, MPEG DASH, HLS o Smooth Streaming) y localizadores de firma de acceso (SAS), que se usan para descargar archivos multimedia. 

Una vez que se crean los localizadores, puede generar las direcciones URL que se utilizan para transmitir o descargar los archivos.

>[!NOTE]
>Cuando se crea la cuenta de AMS, se agrega un punto de conexión de streaming **predeterminado** a la cuenta en estado **Stopped** (Detenido). Para iniciar la transmisión del contenido y aprovechar el empaquetado dinámico y el cifrado dinámico, el punto de conexión de streaming desde el que va a transmitir el contenido debe estar en estado **Running** (En ejecución).

Una dirección URL de streaming de Smooth Streaming tiene el formato siguiente:

```console
{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest
```

Una dirección URL de streaming de HLS tiene el formato siguiente:

```console
{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest(format=m3u8-aapl)
```

Una dirección URL de streaming de MPEG DASH tiene el formato siguiente:

```console
{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest(format=mpd-time-csf)
```

Una dirección URL de SAS utilizada para descargar archivos tiene el formato siguiente:

```console
{blob container name}/{asset name}/{file name}/{SAS signature}
```

En estas secciones se muestra cómo realizar las siguientes tareas necesarias para "publicar" los recursos.  

* Creación de AccessPolicy con permiso de lectura
* Creación de una URL de SAS para descargar contenido
* Creación de una URL de origen para transmitir contenido

### <a name="creating-the-accesspolicy-with-read-permission"></a>Creación de AccessPolicy con permiso de lectura
Antes de descargar o transmitir contenido multimedia, primero defina una AccessPolicy con permisos de lectura y cree la entidad Locator adecuada que especifique el tipo de mecanismo de entrega que desea habilitar para los clientes. Para obtener más información sobre las propiedades disponibles, consulte [Propiedades de la entidad AccessPolicy](/rest/api/media/operations/accesspolicy#accesspolicy_properties).

En el ejemplo siguiente se muestra cómo especificar AccessPolicy para los permisos de lectura de un recurso determinado.

```console
POST https://wamsbayclus001rest-hs.net/API/AccessPolicies HTTP/1.1
Content-Type: application/json
Accept: application/json
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: wamsbayclus001rest-hs.net
Content-Length: 74
Expect: 100-continue

{"Name": "DownloadPolicy", "DurationInMinutes" : "300", "Permissions" : 1}
```

Si se realiza correctamente, se devuelve un código de correcto 201 que describe la entidad AccessPolicy que ha creado. A continuación, usará AccessPolicy Id junto con Asset Id del recurso que contiene el archivo que desea entregar (por ejemplo, un recurso de salida) para crear la entidad Locator.

> [!NOTE]
> Este flujo de trabajo básico es el mismo que la carga de un archivo al ingerir un recurso (como se explicó anteriormente en este tema). Además, como en la carga de archivos, si usted (o sus clientes) necesitan tener acceso a los archivos inmediatamente, establezca el valor StartTime en cinco minutos antes de la hora actual. Esta acción es necesaria porque puede haber un desplazamiento de reloj entre el cliente y Media Services. Además, el valor de StartTime debe estar en el siguiente formato de fecha/hora: AAAA-MM-DDTHH (por ejemplo, "2014-05-23T17:53:50Z").
>
>

### <a name="creating-a-sas-url-for-downloading-content"></a>Creación de una URL de SAS para descargar contenido
El código siguiente muestra cómo obtener una dirección URL que se puede usar para descargar un archivo multimedia creado y cargado previamente. AccessPolicy tiene permisos de lectura establecidos y a ruta de acceso de Locator hace referencia a una dirección URL de descarga de SAS.

```console
POST https://wamsbayclus001rest-hs.net/API/Locators HTTP/1.1
Content-Type: application/json
Accept: application/json
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: wamsbayclus001rest-hs.net
Content-Length: 182
Expect: 100-continue

{"AccessPolicyId": "nb:pid:UUID:38c71dd0-44c5-4c5f-8418-08bb6fbf7bf8", "AssetId" : "nb:cid:UUID:71d2dd33-efdf-ec43-8ea1-136a110bd42c", "StartTime" : "2014-05-17T16:45:53", "Type":1}
```

Si se realiza correctamente, se devuelve la respuesta siguiente:

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 1150
Content-Type: application/json;odata=verbose;charset=utf-8
Location: https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A8e5a821d-2194-4d00-8884-adf979856874')
Server: Microsoft-IIS/7.5
x-ms-request-id: 8cfd8556-3064-416a-97f2-67d9e35f0911
X-Content-Type-Options: nosniff
DataServiceVersion: 1.0;
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
Date: Mon, 14 May 2012 21:41:32 GMT

{  
   "d":{  
      "__metadata":{  
         "id":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A8e5a821d-2194-4d00-8884-adf979856874')",
         "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A8e5a821d-2194-4d00-8884-adf979856874')",
         "type":"Microsoft.Cloud.Media.Vod.Rest.Data.Models.Locator"
      },
      "AccessPolicy":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A8e5a821d-2194-4d00-8884-adf979856874')/AccessPolicy"
         }
      },
      "Asset":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/Asset"
         }
      },
      "Id":"nb:lid:UUID:8e5a821d-2194-4d00-8884-adf979856874",
      "ExpirationDateTime":"\/Date(1337049393000)\/",
      "Type":1,
      "Path":"https://storagetestaccount001.blob.core.windows.net/asset-71d2dd33-efdf-ec43-8ea1-136a110bd42c?st=2012-05-14T21%3A36%3A33Z&se=2012-05-15T02%3A36%3A33Z&sr=c&si=8e5a821d-2194-4d00-8884-adf979856874&sig=y75dViDpC5V8WutrXM%2B%2FGpR3uOtqmlISiNlHU1YUBOg%3D",
      "AccessPolicyId":"nb:pid:UUID:38c71dd0-44c5-4c5f-8418-08bb6fbf7bf8",
      "AssetId":"nb:cid:UUID:71d2dd33-efdf-ec43-8ea1-136a110bd42c",
      "StartTime":"\/Date(1337031393000)\/"
   }
}
```

La propiedad **Path** devuelta contiene la dirección URL de SAS.

> [!NOTE]
> Si descarga contenido cifrado del almacenamiento, debe descifrarlo manualmente antes de presentarlo o usar el procesador multimedia de descifrado de almacenamiento en una tarea de procesamiento para extraer archivos procesados sin cifrar en un recurso de salida y, a continuación, descargar desde ese recurso. Para obtener más información sobre el procesamiento, consulte Creación de un trabajo de Encoding con la API de REST de Media Services. Además, los localizadores de dirección URL de SAS no se pueden actualizar una vez creados. Por ejemplo, no se puede reutilizar el mismo localizador con un valor StartTime actualizado. Esto se debe a la manera en que se crean las direcciones URL de SAS. Si desea obtener acceso a un recurso para descargarlo después de que expire un localizador, debe crear uno nuevo con un valor StartTime.
>
>

### <a name="download-files"></a>Descarga de archivos
Una vez establecidos AccessPolicy y Locator, puede descargar archivos mediante las API de REST de Azure Storage.  

> [!NOTE]
> Debe agregar el nombre de archivo para el archivo que desea descargar en el valor **Path** del localizador recibido en la sección anterior. ¿Por ejemplo, `https://storagetestaccount001.blob.core.windows.net/asset-e7b02da4-5a69-40e7-a8db-e8f4f697aac0/BigBuckBunny.mp4`? . . .

Para obtener más información sobre cómo trabajar con blobs de Almacenamiento de Azure, consulte [API de REST del servicio Blob](/rest/api/storageservices/blob-service-rest-api).

Como resultado del trabajo de codificación que realizó anteriormente (codificación en un conjunto de archivos MP4 adaptables), hay varios archivos MP4 que se pueden descargar progresivamente. Por ejemplo:    

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_400kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_3400kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_2250kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_1500kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_1000kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_AAC_und_ch2_56kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

### <a name="creating-a-streaming-url-for-streaming-content"></a>Creación de una URL de streaming para transmitir contenido
El siguiente código muestra cómo crear un localizador de direcciones URL de streaming:

```console
POST https://wamsbayclus001rest-hs/API/Locators HTTP/1.1
Content-Type: application/json
Accept: application/json
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: wamsbayclus001rest-hs
Content-Length: 182
Expect: 100-continue

{"AccessPolicyId": "nb:pid:UUID:38c71dd0-44c5-4c5f-8418-08bb6fbf7bf8", "AssetId" : "nb:cid:UUID:eb5540a2-116e-4d36-b084-7e9958f7f3c3", "StartTime" : "2014-05-17T16:45:53",, "Type":2}
```

Si se realiza correctamente, se devuelve la respuesta siguiente:

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 981
Content-Type: application/json;odata=verbose;charset=utf-8
Location: https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')
Server: Microsoft-IIS/7.5
x-ms-request-id: 2eac4158-fc78-4fa2-81ee-c9f582dc385f
X-Content-Type-Options: nosniff
DataServiceVersion: 1.0;
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
Date: Mon, 14 May 2012 21:41:39 GMT

{  
   "d":{  
      "__metadata":{  
         "id":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')",
         "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')",
         "type":"Microsoft.Cloud.Media.Vod.Rest.Data.Models.Locator"
      },
      "AccessPolicy":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')/AccessPolicy"
         }
      },
      "Asset":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')/Asset"
         }
      },
      "Id":"nb:lid:UUID:52034bf6-dfae-4d83-aad3-3bd87dcb1a5d",
      "ExpirationDateTime":"\/Date(1337049395000)\/",
      "Type":2,
      "Path":"http://wamsbayclus001rest-hs.net/52034bf6-dfae-4d83-aad3-3bd87dcb1a5d/",
      "AccessPolicyId":"nb:pid:UUID:38c71dd0-44c5-4c5f-8418-08bb6fbf7bf8",
      "AssetId":"nb:cid:UUID:eb5540a2-116e-4d36-b084-7e9958f7f3c3",
      "StartTime":"\/Date(1337031395000)\/"
   }
}
```

Para transmitir una URL de origen de Smooth Streaming en un reproductor multimedia de streaming, debe anexar la propiedad Path con el nombre del archivo de manifiesto de Smooth Streaming seguido de "/manifest".

`http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest`

Para transmitir HLS, anexe(format=m3u8-aapl) después de "/manifest".

`http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest(format=m3u8-aapl)`

Para transmitir MPEG DASH, anexe(format=mpd-time-csf) después de "/manifest".

`http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest(format=mpd-time-csf)`


## <a name="play-your-content"></a><a id="play"></a>Reproducción del contenido
Para transmitir vídeo, use [Reproductor de Azure Media Services](https://aka.ms/azuremediaplayer).

Para probar la descarga progresiva, pegue una dirección URL en un explorador (por ejemplo, Internet Explorer, Chrome o Safari).

## <a name="next-steps-media-services-learning-paths"></a>Pasos siguientes: Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
