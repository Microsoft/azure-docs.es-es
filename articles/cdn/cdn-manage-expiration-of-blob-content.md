---
title: Administración de la expiración de Azure Storage Blob
titleSuffix: Azure Content Delivery Network
description: Obtenga información sobre las opciones para controlar el período de vida de los blobs de almacenamiento en caché de Azure CDN.
services: cdn
documentationcenter: ''
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: ad4801e9-d09a-49bf-b35c-efdc4e6034e8
ms.service: azure-cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: how-to
ms.date: 02/1/2018
ms.author: mazha
ms.openlocfilehash: 49748b3d77d097e655ee6ec5777022c038841a6d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87073134"
---
# <a name="manage-expiration-of-azure-blob-storage-in-azure-cdn"></a>Administración de la expiración de Azure Storage Blob en Azure CDN
> [!div class="op_single_selector"]
> * [Contenido web de Azure](cdn-manage-expiration-of-cloud-service-content.md)
> * [Almacenamiento de blobs de Azure](cdn-manage-expiration-of-blob-content.md)
> 
> 

El [servicio Blob Storage](../storage/common/storage-introduction.md#blob-storage) de Azure Storage es uno de los distintos orígenes basados en Azure que están integrados en Azure Content Delivery Network (CDN). Cualquier contenido de blob accesible públicamente se puede almacenar en caché en Azure CDN hasta que transcurra su tiempo de vida (TTL). El período de vida viene determinado por el encabezado `Cache-Control` en la respuesta HTTP del servidor de origen. En este artículo se describen las distintas maneras en que se puede establecer el encabezado `Cache-Control` de un blob en Azure Storage.

También puede controlar la configuración de caché desde Azure Portal mediante el establecimiento de reglas de almacenamiento en caché de la red CDN. Si crea una regla de almacenamiento en caché y establece el comportamiento de dicho almacenamiento en **Invalidar** u **Omitir caché**, se omite la configuración de almacenamiento en caché proporcionada por el origen que se trata en este artículo. Para información sobre conceptos generales de almacenamiento en caché, consulte [Funcionamiento del almacenamiento en caché](cdn-how-caching-works.md).

> [!TIP]
> Puede optar por no configurar ningún TTL en un blob. En este caso, Azure CDN aplica automáticamente un TTL predeterminado de siete días, a menos que haya configurado reglas de almacenamiento en caché en Azure Portal. Este TTL predeterminado solo se aplica a las optimizaciones de entrega web general. Para las optimizaciones de archivos de gran tamaño, el TTL predeterminado es un día, y para las optimizaciones de streaming multimedia, el TTL predeterminado es un año.
> 
> Para obtener más información sobre el funcionamiento de Azure CDN para acelerar el acceso a los blobs y a otros archivos, consulte [Información general de Azure Content Delivery Network](cdn-overview.md).
> 
> Para obtener más información acerca de Azure Blob Storage, consulte [Introducción a Blob Storage](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-introduction).
 

## <a name="setting-cache-control-headers-by-using-cdn-caching-rules"></a>Establecimiento de los encabezados Cache-Control mediante reglas de almacenamiento en caché de CDN
El método preferido para establecer el encabezado `Cache-Control` de un blob consiste en usar las reglas de almacenamiento en caché en Azure Portal. Para más información sobre las reglas de almacenamiento en caché de CDN, consulte [Control del comportamiento del almacenamiento en caché de Azure CDN con reglas de almacenamiento en caché](cdn-caching-rules.md).

> [!NOTE] 
> Las reglas de almacenamiento en caché solo están disponibles para los perfiles **Azure CDN estándar de Verizon** y **Azure CDN estándar de Akamai**. Para perfiles **Azure CDN premium de Verizon**, debe usar el [motor de reglas de Azure CDN](cdn-rules-engine.md) en el portal **Administrar** para una funcionalidad similar.

**Para navegar a la página de reglas de almacenamiento en caché de CDN**:

1. En Azure Portal, seleccione un perfil de CDN y luego seleccione un punto de conexión para el blob.

2. En el panel izquierdo, debajo de Configuración, haga clic en **Reglas de caché**.

   ![Botón Reglas de caché de CDN](./media/cdn-manage-expiration-of-blob-content/cdn-caching-rules-btn.png)

   Aparece la página **Reglas de caché**.

   ![Página de caché de CDN](./media/cdn-manage-expiration-of-blob-content/cdn-caching-page.png)


**Para establecer encabezados Cache-Control del servicio de almacenamiento de blobs con reglas de almacenamiento en caché globales:**

1. En **Reglas de almacenamiento en caché globales**, establezca **Comportamiento del almacenamiento en caché de cadenas de consulta** en **Ignorar cadenas de consulta**, y establezca **Comportamiento de almacenamiento en caché** en **Invalidar**.
      
2. Para **Duración de expiración de caché**, escriba 3600 en el cuadro **Segundos** o 1 en el cuadro **Horas**. 

   ![Ejemplo de reglas de almacenamiento en caché globales de CDN](./media/cdn-manage-expiration-of-blob-content/cdn-global-caching-rules-example.png)

   Esta regla de almacenamiento en caché global establece una duración de caché de una hora y afecta a todas las solicitudes para el punto de conexión. Invalida todos los encabezados HTTP `Cache-Control` o `Expires` que envía el servidor de origen especificado por el punto de conexión.   

3. Seleccione **Guardar**.
 
**Para establecer encabezados Cache-Control de un archivo de blobs con reglas de almacenamiento en caché personalizadas:**

1. En **Reglas de almacenamiento en caché personalizadas**, cree dos condiciones de coincidencia:

     A. Para la primera condición de coincidencia, establezca **Condición de coincidencia** en **Ruta de acceso** y escriba `/blobcontainer1/*` en **Match value** (Valor de coincidencia). Establezca **Comportamiento de almacenamiento en caché** en **Invalidar** y escriba 4 en el cuadro **Horas**.

    B. Para la segunda condición de coincidencia, establezca **Condición de coincidencia** en **Ruta de acceso** y escriba `/blobcontainer1/blob1.txt` en **Match value** (Valor de coincidencia). Establezca **Comportamiento de almacenamiento en caché** en **Invalidar** y escriba 2 en el cuadro **Horas**.

    ![Ejemplo de reglas de almacenamiento en caché personalizadas de CDN](./media/cdn-manage-expiration-of-blob-content/cdn-custom-caching-rules-example.png)

    La primera regla de almacenamiento en caché personalizada establece una duración de caché de cuatro horas para los archivos de blobs en la carpeta `/blobcontainer1` en el servidor de origen especificado por el punto de conexión. La segunda regla invalida la primera regla para el archivo de blobs `blob1.txt` únicamente y establece una duración de caché de dos horas para él.

2. Seleccione **Guardar**.


## <a name="setting-cache-control-headers-by-using-azure-powershell"></a>Establecimiento de los encabezados Cache-Control mediante Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[Azure PowerShell](/powershell/azure/) es una de las formas más rápidas y eficaces de administrar los servicios de Azure. Utilice el cmdlet `Get-AzStorageBlob` para obtener una referencia al blob y, a continuación, establezca la propiedad `.ICloudBlob.Properties.CacheControl`. 

Por ejemplo:

```powershell
# Create a storage context
$context = New-AzStorageContext -StorageAccountName "<storage account name>" -StorageAccountKey "<storage account key>"

# Get a reference to the blob
$blob = Get-AzStorageBlob -Context $context -Container "<container name>" -Blob "<blob name>"

# Set the CacheControl property to expire in 1 hour (3600 seconds)
$blob.ICloudBlob.Properties.CacheControl = "max-age=3600"

# Send the update to the cloud
$blob.ICloudBlob.SetProperties()
```

> [!TIP]
> También puede usar PowerShell para [administrar los perfiles y puntos de conexión de la red CDN](cdn-manage-powershell.md).
> 
>

## <a name="setting-cache-control-headers-by-using-net"></a>Establecimiento de los encabezados Cache-Control mediante .NET
Para especificar el encabezado `Cache-Control` de un blob mediante el código .NET, use la [Biblioteca cliente de Azure Storage para .NET](../storage/blobs/storage-dotnet-how-to-use-blobs.md) para configurar la propiedad [CloudBlob.Properties.CacheControl](/dotnet/api/microsoft.azure.storage.blob.blobproperties.cachecontrol).

Por ejemplo:

```csharp
class Program
{
    const string connectionString = "<storage connection string>";
    static void Main()
    {
        // Retrieve storage account information from connection string
        CloudStorageAccount storageAccount = CloudStorageAccount.Parse(connectionString);

        // Create a blob client for interacting with the blob service.
        CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

        // Create a reference to the container
        CloudBlobContainer <container name> = blobClient.GetContainerReference("<container name>");

        // Create a reference to the blob
        CloudBlob <blob name> = container.GetBlobReference("<blob name>");

        // Set the CacheControl property to expire in 1 hour (3600 seconds)
        blob.Properties.CacheControl = "max-age=3600";

        // Update the blob's properties in the cloud
        blob.SetProperties();
    }
}
```

> [!TIP]
> Hay más ejemplos de código de .NET en [Azure Blob Storage Samples for .NET](https://azure.microsoft.com/documentation/samples/storage-blob-dotnet-getting-started/) (Ejemplos de Azure Blob Storage para .NET).
> 

## <a name="setting-cache-control-headers-by-using-other-methods"></a>Establecimiento de los encabezados Cache-Control mediante otros métodos

### <a name="azure-storage-explorer"></a>Explorador de Azure Storage
Con el [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/), puede ver y modificar los recursos de Blob Storage, incluidas las propiedades como *CacheControl*. 

Para actualizar la propiedad *CacheControl* de un blob con el Explorador de Azure Storage:
   1. Seleccione un blob y después seleccione **Propiedades** en el menú contextual. 
   2. Desplácese hacia abajo hasta la propiedad *CacheControl*.
   3. Escriba un valor y, después, seleccione **Guardar**.


![Propiedades del Explorador de Azure Storage](./media/cdn-manage-expiration-of-blob-content/cdn-storage-explorer-properties.png)

### <a name="azure-command-line-interface"></a>Interfaz de la línea de comandos de Azure
Con la [interfaz de la línea de comandos de Azure](https://docs.microsoft.com/cli/azure) (CLI), puede administrar recursos de blob de Azure desde la línea de comandos. Para establecer el encabezado de control de caché cuando cargue un blob con la CLI de Azure, establezca la propiedad *cacheControl* mediante el uso del modificador `-p`. En el siguiente ejemplo, se muestra cómo se establece el período de vida en una hora (3600 segundos):
  
```azurecli
azure storage blob upload -c <connectionstring> -p cacheControl="max-age=3600" .\<blob name> <container name> <blob name>
```

### <a name="azure-storage-services-rest-api"></a>API de REST de servicios de Azure Storage
Puede usar la [API de REST de servicios de Azure Storage](/rest/api/storageservices/) para establecer explícitamente la propiedad *x-ms-blob-cache-control* mediante el uso de las siguientes operaciones en una solicitud:
  
   - [Put Blob](/rest/api/storageservices/Put-Blob)
   - [Put Block List](/rest/api/storageservices/Put-Block-List)
   - [Set Blob Properties](/rest/api/storageservices/Set-Blob-Properties)

## <a name="testing-the-cache-control-header"></a>Prueba del encabezado Cache-Control
La configuración de TTL de los blobs se puede comprobar con facilidad. Con las [herramientas de desarrollo](https://developer.microsoft.com/microsoft-edge/platform/documentation/f12-devtools-guide/) del explorador, compruebe que el blob incluye el encabezado de respuesta `Cache-Control`. Asimismo, también puede usar una herramienta como [Wget](https://www.gnu.org/software/wget/), [Postman](https://www.getpostman.com/) o [Fiddler](https://www.telerik.com/fiddler) para examinar los encabezados de respuesta.

## <a name="next-steps"></a>Pasos siguientes
* [Aprenda a administrar la expiración del contenido del Servicio en la nube en Azure CDN](cdn-manage-expiration-of-cloud-service-content.md)
* [Obtenga información sobre conceptos del almacenamiento en caché](cdn-how-caching-works.md)

