---
title: Carga de archivos en una cuenta de Azure Media Services mediante REST | Microsoft Docs
description: Aprenda a obtener contenido multimedia en Media Services mediante la creación y carga de recursos.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: c3d776362b0447b148c0b2bdedba1287fa56058b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87000200"
---
# <a name="upload-files-into-a-media-services-account-using-rest"></a>Carga de archivos en una cuenta de Media Services mediante API de REST  
> [!div class="op_single_selector"]
> * [.NET](media-services-dotnet-upload-files.md)
> * [REST](media-services-rest-upload-files.md)
> * [Portal](media-services-portal-upload-files.md)
> 

En Media Services, cargue los archivos digitales en un recurso. La entidad [Asset](/rest/api/media/operations/asset) puede contener archivos de vídeo, audio, imágenes, colecciones de miniaturas, pistas de texto y subtítulos (y los metadatos sobre estos archivos).  Una vez cargados los archivos en el recurso, el contenido se almacena de forma segura en la nube para un posterior procesamiento y streaming. 

En este tutorial, obtendrá información sobre cómo cargar un archivo y otras operaciones asociadas:

> [!div class="checklist"]
> * Configuración de Postman para todas las operaciones de carga
> * Conexión con Media Services 
> * Creación de una directiva de acceso con permiso de escritura
> * Creación de un recurso
> * Creación de un localizador de SAS y de la dirección URL de carga
> * Carga de un archivo a Blob Storage mediante la dirección URL de carga
> * Creación de metadatos en el recurso para el archivo multimedia cargado

## <a name="prerequisites"></a>Prerrequisitos

- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) antes de empezar.
- [Cree una cuenta de Azure Media Services mediante Azure Portal](media-services-portal-create-account.md).
- Revise el artículo [Acceso a Azure Media Services API con la autenticación de Azure AD](media-services-use-aad-auth-to-access-ams-api.md).
- Para obtener más información, vea [Uso de la autenticación de Azure AD para acceder a la API de Media Services con REST](./media-services-rest-connect-with-aad.md).
- Configure **Postman** tal y como se describe en [Configure Postman for Media Services REST API calls](media-rest-apis-with-postman.md) (Configuración de Postman para llamadas API de REST de Media Services).

## <a name="considerations"></a>Consideraciones

Al usar la API de REST de Media Services, se aplican las consideraciones siguientes:
 
* Al obtener acceso a las entidades mediante la API de REST de Media Services, debe establecer campos de encabezado y valores específicos en las solicitudes HTTP. Para obtener más información, consulte [Configuración del desarrollo de la API de REST de Media Services](media-services-rest-how-to-use.md). <br/>La colección de Postman que se usa en este tutorial establece todos los encabezados necesarios.
* Media Services usa el valor de la propiedad IAssetFile.Name al generar direcciones URL para el contenido de streaming (por ejemplo, http://{AMSAccount}.origin.mediaservices.windows.net/{GUID}/{IAssetFile.Name}/streamingParameters.) Por esta razón, no se permite la codificación porcentual. El valor de la propiedad **Name**no puede tener ninguno de los siguientes [caracteres reservados para la codificación porcentual](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters):!*'();:@&=+$,/?%#[]" Además, solo puede haber un '.' para la extensión del nombre de archivo.
* La longitud del nombre no debe ser superior a 260 caracteres.
* Existe un límite máximo de tamaño de archivo admitido para el procesamiento en Media Services. Consulte [este](media-services-quotas-and-limitations.md) artículo para obtener información más detallada acerca de la limitación de tamaño de archivo.

## <a name="set-up-postman"></a>Configuración de Postman

Para obtener los pasos sobre cómo configurar Postman para este tutorial, consulte [Configuración de Postman](media-rest-apis-with-postman.md).

## <a name="connect-to-media-services"></a>Conexión con Media Services

1. Agregue valores de conexión al entorno. 

    Algunas variables que forman parte del [entorno](postman-environment.md) de **MediaServices** deben establecerse manualmente para poder empezar a ejecutar las operaciones definidas en la [colección](postman-collection.md).

    Para obtener los valores de las cinco primeras variables, consulte [Acceso a Azure Media Services API con la autenticación de Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

    ![Cargar un archivo](./media/media-services-rest-upload-files/postman-import-env.png)
2. Especifique el valor de la variable de entorno **MediaFileName**.

    Especifique el nombre de archivo del elemento multimedia que quiere cargar. En este ejemplo, se cargará el elemento BigBuckBunny.mp4. 
3. Examine el archivo **AzureMediaServices.postman_environment.json**. Verá que casi todas las operaciones de la colección ejecutan un script "test". Los scripts recogen algunos valores devueltos por la respuesta y establecen variables de entorno adecuadas.

    Por ejemplo, la primera operación obtiene un token de acceso y lo establece en la variable de entorno **AccessToken** que se usa en todas las demás operaciones.

    ```    
    "listen": "test",
    "script": {
        "type": "text/javascript",
        "exec": [
            "var json = JSON.parse(responseBody);",
            "postman.setEnvironmentVariable(\"AccessToken\", json.access_token);"
        ]
    }
    ```
4. En la parte izquierda de la ventana **Postman**, haga clic en **1. Get AAD Auth token (Obtener token de autenticación de AAD)**  -> **Get Azure AD Token for Service Principal (Obtener token de Azure AD para la entidad de servicio)** .

    La parte de la dirección URL se rellena con la variable de entorno **AzureADSTSEndpoint** (anteriormente en el tutorial estableció los valores de las variables de entorno para que admiten la colección).

    ![Cargar un archivo](./media/media-services-rest-upload-files/postment-get-token.png)

5. Presione **Enviar**.

    Puede ver la respuesta que contiene "access_token". El script "test" toma este valor y establece la variable de entorno **AccessToken** (tal y como se ha descrito anteriormente). Si examina las variables de entorno, verá que esta variable contiene ahora el valor del token de acceso (token de portador) que se usa en las demás operaciones. 

    Si expira el token, vuelva a realizar el paso "Get Azure AD Token for Service Principal" (Obtener token de Azure AD para la entidad de servicio). 

## <a name="create-an-access-policy-with-write-permission"></a>Creación de una directiva de acceso con permiso de escritura

### <a name="overview"></a>Información general 

>[!NOTE]
>Hay un límite de 1 000 000 directivas para diferentes directivas de AMS (por ejemplo, para la directiva de localizador o ContentKeyAuthorizationPolicy). Debe usar el mismo identificador de directiva si siempre usa los mismos permisos de acceso y días, por ejemplo, directivas para localizadores que vayan a aplicarse durante mucho tiempo (directivas distintas a carga). Para obtener más información, consulte [este](media-services-dotnet-manage-entities.md#limit-access-policies) artículo.

Antes de cargar los archivos en el almacenamiento de blobs, establezca los derechos de la directiva de acceso para escribir en un recurso. Para ello, envíe una solicitud HTTP al conjunto de entidades AccessPolicies. Defina un valor DurationInMinutes tras la creación o recibirá un mensaje de error de servidor interno 500 como respuesta. Para obtener más información sobre AccessPolicies, consulte [AccessPolicy](/rest/api/media/operations/accesspolicy).

### <a name="create-an-access-policy"></a>Creación de una directiva de acceso

1. Seleccione **AccessPolicy** -> **Create AccessPolicy for Upload (Crear AccessPolicy para la carga)** .
2. Presione **Enviar**.

    ![Cargar un archivo](./media/media-services-rest-upload-files/postman-access-policy.png)

    El script "test" obtiene el identificador de AccessPolicy y establece la variable de entorno adecuada.

## <a name="create-an-asset"></a>Creación de un recurso

### <a name="overview"></a>Información general

Un [recurso](/rest/api/media/operations/asset) es un contenedor para varios tipos o conjuntos de objetos en Media Services, como vídeo, audio, imágenes, colecciones de miniaturas, pistas de texto y archivos de subtítulos. En la API de REST, crear un recurso requiere el envío de una solicitud POST a Media Services y colocar la información sobre cualquier propiedad del recurso en el cuerpo de solicitud.

Una de las propiedades que se pueden agregar al crear un recurso es **Opciones**. Puede especificar una de las siguientes opciones de cifrado: **None** (predeterminada, no se usa ningún cifrado), **StorageEncrypted** (para el contenido que se ha cifrado previamente con el cifrado de almacenamiento en el cliente), **CommonEncryptionProtected** o **EnvelopeEncryptionProtected**. Cuando tenga un recurso cifrado, tiene que configurar una directiva de entrega. Para obtener más información, consulte [Configuración de directivas de entrega de recursos](media-services-rest-configure-asset-delivery-policy.md).

Si el recurso está cifrado, debe crear un elemento **ContentKey** y vincularlo al recurso, como se describe en el siguiente artículo: [How to create a ContentKey](media-services-rest-create-contentkey.md) (Creación de un elemento ContentKey). Después de cargar los archivos en el recurso, debe actualizar las propiedades de cifrado en la entidad **AssetFile** con los valores que obtuvo durante el cifrado del **recurso**. Para ello, use la solicitud HTTP **MERGE** . 

En este ejemplo, crearemos un recurso no cifrado. 

### <a name="create-an-asset"></a>Creación de un recurso

1. Seleccione **Recursos** -> **Crear recurso**.
2. Presione **Enviar**.

    ![Cargar un archivo](./media/media-services-rest-upload-files/postman-create-asset.png)

    El script "test" obtiene el identificador de recurso y establece la variable de entorno adecuada.

## <a name="create-a-sas-locator-and-create-the-upload-url"></a>Creación de un localizador SAS y de la dirección URL de carga

### <a name="overview"></a>Información general

Una vez establecidas AccessPolicy y Locator, el archivo real se carga en un contenedor de Azure Blob Storage mediante las API de REST de Azure Storage. Debe cargar los archivos como blobs en bloques. Azure Media Services no admite los blobs en páginas.  

Para obtener más información sobre cómo trabajar con blobs de Almacenamiento de Azure, consulte [API de REST del servicio Blob](/rest/api/storageservices/blob-service-rest-api).

Para recibir la dirección URL de carga real, cree un localizador de SAS (se muestra a continuación). Los localizadores definen la hora de inicio y el tipo de extremo de conexión para los clientes que desean tener acceso a archivos de un recurso. Puede crear varias entidades Locator para un par AccessPolicy y Asset determinado a fin de controlar las distintas solicitudes y necesidades del cliente. Cada uno de estos localizadores usa el valor StartTime más el valor DurationInMinutes de AccessPolicy para determinar la cantidad de tiempo que se puede usar una dirección URL. Para obtener más información, consulte [Localizador](/rest/api/media/operations/locator).

Una dirección URL de SAS tiene el formato siguiente:

`{https://myaccount.blob.core.windows.net}/{asset name}/{video file name}?{SAS signature}`

### <a name="considerations"></a>Consideraciones

Se aplican algunas consideraciones:

* No puede tener más de cinco localizadores únicos asociados a un recurso determinado a la vez. Para obtener más información, consulte Localizador.
* Si necesita cargar los archivos inmediatamente, debe establecer el valor StartTime cinco minutos antes de la hora actual. Esto se debe a que puede haber un desplazamiento de reloj entre el equipo cliente y Media Services. Además, el valor de StartTime debe estar en el siguiente formato de fecha/hora: AAAA-MM-DDTHH (por ejemplo, "2014-05-23T17:53:50Z").    
* Puede haber un retraso de 30 a 40 segundos desde el momento en que se crea un localizador hasta que este está disponible para su uso.

### <a name="create-a-sas-locator"></a>Creación de un localizador de SAS

1. Seleccione **Localizador** -> **Create SAS Locator (Crear localizador de SAS)** .
2. Presione **Enviar**.

    El script "test" crea la "dirección URL de carga" en función del nombre del archivo multimedia que ha especificado y la información del localizador de SAS, y establece la variable de entorno adecuada.

    ![Cargar un archivo](./media/media-services-rest-upload-files/postman-create-sas-locator.png)

## <a name="upload-a-file-to-blob-storage-using-the-upload-url"></a>Carga de un archivo a Blob Storage mediante la dirección URL de carga

### <a name="overview"></a>Información general

Ahora que tiene la dirección URL de carga, tiene que escribir código directamente mediante las API de Blob de Azure para cargar el archivo en el contenedor de SAS. Para más información, consulte los siguientes artículos.

- [Uso de la API de REST de Azure Storage](../../storage/common/storage-rest-api-auth.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
- [PUT Blob](/rest/api/storageservices/put-blob)
- [Carga de blobs a Blob Storage](/previous-versions/azure/storage/storage-use-azcopy#upload-blobs-to-blob-storage)

### <a name="upload-a-file-with-postman"></a>Carga de un archivo con Postman

Por ejemplo, usamos Postman para cargar un archivo .mp4 pequeño. Es posible que haya un límite de tamaño de archivo para la carga binaria mediante Postman.

La solicitud de carga no forma parte de la colección **AzureMedia**. 

Creación y configuración de una nueva solicitud:
1. Presione **+** para crear una nueva pestaña de solicitud.
2. Seleccione la operación **PUT** y pegue **{{UploadURL}}** en la dirección URL.
2. Deje la pestaña **Autorización** tal cual (no la establezca en **Bearer Token [Token de portador]** ).
3. En la pestaña **Encabezados** especifique: **Clave**: "x-ms-blob-type" y **Valor**: "BlockBlob".
2. En la pestaña **Cuerpo**, haga clic en **binario**.
4. Elija el archivo con el nombre que especificó en la variable de entorno **MediaFileName**.
5. Presione **Enviar**.

    ![Cargar un archivo](./media/media-services-rest-upload-files/postman-upload-file.png)

##  <a name="create-a-metadata-in-the-asset"></a>Creación de metadatos en el recurso

Una vez cargado el archivo, tiene que crear metadatos en el recurso para el archivo multimedia cargado en Blob Storage asociado al recurso.

1. Seleccione **AssetFiles** -> **CreateFileInfos**.
2. Presione **Enviar**.

    ![Cargar un archivo](./media/media-services-rest-upload-files/postman-create-file-info.png)

El archivo debería estar cargado con los metadatos establecidos.

## <a name="validate"></a>Validación

Para validar si el archivo se cargó correctamente, es recomendable consultar [AssetFile](/rest/api/media/operations/assetfile) y comparar **ContentFileSize** (u otros detalles) con lo que espera ver en el nuevo recurso. 

Por ejemplo, la siguiente operación **GET** obtiene los datos de archivo para el archivo de recursos (en este caso, el archivo BigBuckBunny.mp4). La consulta usa las [variables de entorno](postman-environment.md) establecidas anteriormente.

`{{RESTAPIEndpoint}}/Assets('{{LastAssetId}}')/Files`

La respuesta contendrá el tamaño, el nombre y otra información.

```console
"Id": "nb:cid:UUID:69e72ede-2886-4f2a-8d36-80a59da09913",
"Name": "BigBuckBunny.mp4",
"ContentFileSize": "3186542",
"ParentAssetId": "nb:cid:UUID:0b8f3b04-72fb-4f38-8e7b-d7dd78888938",
```
  
## <a name="next-steps"></a>Pasos siguientes

Ahora puede codificar los recursos cargados. Para más información, consulte [Encode an asset using Media Encoder Standard with the Azure portal](media-services-portal-encode.md)(Codificación de recursos mediante el estándar de codificador multimedia con Azure Portal).

También puede usar Azure Functions para desencadenar un trabajo de codificación basado en un archivo que llega al contenedor configurado. Para más información, consulte [este ejemplo](https://azure.microsoft.com/resources/samples/media-services-dotnet-functions-integration/ ).
