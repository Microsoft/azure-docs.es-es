---
title: Introducción a blobs en páginas de Azure | Microsoft Docs
description: Una introducción a los blobs en páginas de Azure y sus ventajas, con casos de uso y scripts de ejemplo.
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 06/15/2020
ms.author: tamram
ms.reviewer: wielriac
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: 542c9374b70cd765ed27dd4dd158ad81035269f0
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018848"
---
# <a name="overview-of-azure-page-blobs"></a>Introducción a blobs en páginas de Azure

Azure Storage ofrece tres tipos de almacenamiento en blobs: Blobs en bloques, en páginas y en anexos. Blobs en bloques está compuesto por bloques y son ideales para almacenar archivos de texto o binarios, así como para cargar archivos grandes de forma eficaz. Blobs en anexos también está compuesto por bloques, pero están optimizados para la anexión de operaciones, lo que resulta ideal para escenarios de registro. Los blobs en páginas están compuestos por páginas de 512 bytes de hasta 8 TB en total y se han diseñado para las operaciones aleatorias y frecuentes de lectura/escritura. Blobs en páginas constituye la base de los discos de IaaS de Azure. Este artículo se centra en explicar las características y ventajas de los blobs en páginas.

Los blobs en páginas son una colección de páginas de 512 bytes que proporcionan la capacidad de leer y escribir intervalos arbitrarios de bytes. Por lo tanto, son ideales para almacenar estructuras de datos esparcidos basadas en índices, como discos de datos y de sistema operativo para máquinas virtuales y bases de datos. Por ejemplo, Azure SQL DB usa blobs en páginas como almacenamiento persistente subyacente para sus bases de datos. Además, los blobs en páginas suelen usarse para archivos con actualizaciones basadas en intervalos.  

Las características clave de los blobs en páginas de Azure son la interfaz REST, la durabilidad del almacenamiento subyacente y las excepcionales funcionalidades de migración a Azure. En la sección siguiente, analizaremos detalladamente estas características. Además, los blobs en páginas de Azure se admiten actualmente en dos tipos de almacenamiento: Premium Storage y Standard Storage. La opción Premium Storage está diseñada específicamente para cargas de trabajo que requieren alto rendimiento coherente y baja latencia, lo que hace que los blobs en páginas premium sean perfectos para escenarios de almacenamiento de alto rendimiento. Las cuentas de Standard Storage son más rentables para cargas de trabajo que se ejecutan sin tener en cuenta la latencia.

## <a name="sample-use-cases"></a>Casos de uso de ejemplo

Hablemos sobre un par de casos de uso de los blobs en páginas, comenzando con los discos de IaaS de Azure. Los blobs en páginas de Azure son la columna vertebral de la plataforma virtual de discos de IaaS de Azure. Tanto los discos de datos como el sistema operativo de Azure se implementan como discos virtuales donde los datos se conservan de forma duradera en la plataforma de Azure Storage y, después, se entregan a las máquinas virtuales para obtener el máximo rendimiento. Los discos de Azure se conservan en [formato VHD](https://technet.microsoft.com/library/dd979539.aspx) Hyper-V y se almacenan como [blobs en páginas](/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs#about-page-blobs) en Azure Storage. Además de usar discos virtuales para las máquinas virtuales de IaaS de Azure, los blobs en páginas también permiten escenarios de PaaS y DBaaS, como el servicio Azure SQL DB, que actualmente usa los blobs en páginas para almacenar datos SQL, lo que permite operaciones de lectura y escritura aleatorias y rápidas en la base de datos. Otro ejemplo sería si dispone de un servicio PaaS para acceder a elementos multimedia compartidos para aplicaciones colaborativas de edición de vídeo, los blobs en páginas habilitan un acceso rápido a ubicaciones aleatorias en los elementos multimedia. También permite una edición y combinación rápida y eficaz de dichos elementos multimedia por parte de varios usuarios. 

Servicios de Microsoft de primera entidad, como Azure Site Recovery o Azure Backup, así como muchos terceros desarrolladores de aplicaciones han implementado las innovaciones líderes en el sector mediante el uso de la interfaz REST de los blobs en páginas. A continuación se incluyen algunos de los escenarios únicos implementados en Azure: 

* Administración de instantáneas incrementales orientadas a la aplicación: las aplicaciones pueden aprovechar las API REST y las instantáneas de los blobs en páginas para guardar los puntos de control de las aplicaciones sin incurrir en costosas duplicaciones de datos. Azure Storage admite las instantáneas locales para blobs en páginas, que no requieren copiar todo el blob. Estas API de instantáneas públicas también permiten acceder y copiar las diferencias entre instantáneas.
* Migración en vivo de aplicación y datos de local a la nube: Copie los datos locales y use las API de REST para escribir directamente en un blob en páginas de Azure mientras la máquina virtual local sigue ejecutándose. Una vez alcanzado el objetivo, puede conmutar por error rápidamente a la máquina virtual de Azure con esos datos. De esta forma, puede migrar las máquinas virtuales y los discos virtuales de local a la nube con un tiempo de inactividad mínimo, ya que la migración de datos se realiza en segundo plano mientras se sigue usando la máquina virtual y el tiempo de inactividad necesario para la conmutación por error es reducido (en minutos).
* El acceso compartido [basado en SAS](../common/storage-sas-overview.md) permite escenarios como varios lectores y un único escritor, y admite el control de simultaneidad.

## <a name="pricing"></a>Precios

Ambos tipos de almacenamiento que se ofrecen con blobs en páginas tienen su propio modelo de precios. Los blobs en páginas Premium siguen el modelo de precios de los discos administrados, mientras que los blobs en páginas estándar se facturan, según el tamaño usado, por transacción. Para más información, consulte la página [Precios de blobs en páginas de Azure](https://azure.microsoft.com/pricing/details/storage/page-blobs/).

## <a name="page-blob-features"></a>Características de blobs en páginas

### <a name="rest-api"></a>API DE REST

Consulte el documento siguiente para empezar a [desarrollar con los blobs en páginas](storage-dotnet-how-to-use-blobs.md). Por ejemplo, vea cómo acceder a los blobs en páginas mediante la biblioteca cliente de Storage para .NET. 

En el siguiente diagrama se describen las relaciones globales entre la cuenta, los contenedores y los blobs en páginas.

![Captura de pantalla en la que se muestran las relaciones entre cuentas, contenedores y blobs en páginas](./media/storage-blob-pageblob-overview/storage-blob-pageblob-overview-figure1.png)

#### <a name="creating-an-empty-page-blob-of-a-specified-size"></a>Creación de un blob en páginas vacío de tamaño específico

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

En primer lugar, obtenga una referencia a un contenedor. Para crear un blob en páginas, llame al método [GetPageBlobClient](/dotnet/api/azure.storage.blobs.specialized.specializedblobextensions.getpageblobclient) y, a continuación, llame al método [PageBlobClient.Create](/dotnet/api/azure.storage.blobs.specialized.pageblobclient.create). Pase el tamaño máximo del blob que se va a crear. Ese tamaño debe ser un múltiplo de 512 bytes.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_CreatePageBlob":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Para crear un blob en páginas, primero debe crear un objeto **CloudBlobClient**, con el URI base para acceder al almacenamiento de blobs de la cuenta de almacenamiento (*pbaccount* en la figura 1) junto con el objeto **StorageCredentialsAccountAndKey**, tal y como se muestra en el ejemplo siguiente. El ejemplo muestra cómo crear una referencia a un objeto **CloudBlobContainer** y, a continuación, cómo crear el contenedor (*testvhds*) si aún no existe. A continuación, con el objeto **CloudBlobContainer**, cree una referencia a un objeto **CloudPageBlob** especificando el nombre de blob en página (os4.vhd) al que se quiere acceder. Para crear el blob en páginas, llame a [CloudPageBlob.Create](/dotnet/api/microsoft.azure.storage.blob.cloudpageblob.create) pasando el tamaño máximo del blob que se va a crear. *blobSize* debe ser un múltiplo de 512 bytes.

```csharp
using Microsoft.Azure;
using Microsoft.Azure.Storage;
using Microsoft.Azure.Storage.Blob;

long OneGigabyteAsBytes = 1024 * 1024 * 1024;
// Retrieve storage account from connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the blob client.
CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

// Retrieve a reference to a container.
CloudBlobContainer container = blobClient.GetContainerReference("testvhds");

// Create the container if it doesn't already exist.
container.CreateIfNotExists();

CloudPageBlob pageBlob = container.GetPageBlobReference("os4.vhd");
pageBlob.Create(16 * OneGigabyteAsBytes);
```

---

#### <a name="resizing-a-page-blob"></a>Cambio del tamaño de un blob en páginas

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Para cambiar el tamaño de un blob en páginas después de crearlo, use el método [Resize](/dotnet/api/azure.storage.blobs.specialized.pageblobclient.resize?view=azure-dotnet). El tamaño solicitado debe ser un múltiplo de 512 bytes.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_ResizePageBlob":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Para cambiar el tamaño de un blob en páginas después de crearlo, use el método [Resize](/dotnet/api/microsoft.azure.storage.blob.cloudpageblob.resize). El tamaño solicitado debe ser un múltiplo de 512 bytes.

```csharp
pageBlob.Resize(32 * OneGigabyteAsBytes);
```

---

#### <a name="writing-pages-to-a-page-blob"></a>Escritura de páginas en un blob en páginas

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Para escribir páginas, use el método [PageBlobClient.UploadPages](/dotnet/api/azure.storage.blobs.specialized.pageblobclient.uploadpages).  

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_WriteToPageBlob":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Para escribir páginas, use el método [CloudPageBlob.WritePages](/dotnet/api/microsoft.azure.storage.blob.cloudpageblob.beginwritepages).  

```csharp
pageBlob.WritePages(dataStream, startingOffset); 
```

---

Esto le permitirá escribir un conjunto secuencial de páginas de hasta 4 MB. El desplazamiento que se está escribiendo debe comenzar en un límite de 512 bytes (startingOffset % 512 == 0), y terminar en un límite de 1 byte. 

Tan pronto como una solicitud de escritura para un conjunto secuencial de páginas se realiza correctamente en Blob service y se replica para su durabilidad y resistencia, la operación de escritura se confirma y se envía una notificación al cliente.  

En el siguiente diagrama se muestran dos operaciones de escritura independientes:

![Diagrama que muestra las dos opciones de escritura independientes.](./media/storage-blob-pageblob-overview/storage-blob-pageblob-overview-figure2.png)

1.  Una operación de escritura, que comienza en el desplazamiento 0 de 1024 bytes de longitud. 
2.  Una operación de escritura, que comienza en el desplazamiento 4096 de 1024 bytes de longitud. 

#### <a name="reading-pages-from-a-page-blob"></a>Lectura de páginas de un blob en páginas

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Para leer las páginas, use el método [PageBlobClient.download](/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.download) para leer un intervalo de bytes del blob en páginas. 

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_ReadFromPageBlob":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Para leer las páginas, use el método [CloudPageBlob.DownloadRangeToByteArray](/dotnet/api/microsoft.azure.storage.blob.icloudblob.downloadrangetobytearray) para leer un intervalo de bytes desde el blob en páginas. 

```csharp
byte[] buffer = new byte[rangeSize];
pageBlob.DownloadRangeToByteArray(buffer, bufferOffset, pageBlobOffset, rangeSize); 
```

---

Esto permite descargar el blob completo o un intervalo de bytes a partir de cualquier posición de desplazamiento en el blob. Al leer, el desplazamiento no tiene que partir de un múltiplo de 512. Al leer bytes de una página NUL, el servicio devuelve cero bytes.

En la siguiente ilustración se muestra una operación de lectura con un desplazamiento de 256 y un rango de tamaño de 4352. Los datos devueltos están resaltados en naranja. Se devuelven ceros para las páginas NUL.

![Un diagrama que muestra una operación de lectura con un desplazamiento de 256 y un rango de tamaño de 4352](./media/storage-blob-pageblob-overview/storage-blob-pageblob-overview-figure3.png)

Si tiene un blob apenas lleno, es posible que quiera cargar solo las regiones de página válidas para evitar pagar la salida de cero bytes y reducir la latencia de descarga.  

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Para determinar qué páginas están respaldadas por datos, use [PageBlobClient.GetPageRanges](/dotnet/api/azure.storage.blobs.specialized.pageblobclient.getpageranges). A continuación, puede enumerar los intervalos devueltos y descargar los datos de cada intervalo. 

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD.cs" id="Snippet_ReadValidPageRegionsFromPageBlob":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Para determinar qué páginas están respaldadas por datos, use [CloudPageBlob.GetPageRanges](/dotnet/api/microsoft.azure.storage.blob.cloudpageblob.getpageranges). A continuación, puede enumerar los intervalos devueltos y descargar los datos de cada intervalo. 

```csharp
IEnumerable<PageRange> pageRanges = pageBlob.GetPageRanges();

foreach (PageRange range in pageRanges)
{
    // Calculate the range size
    int rangeSize = (int)(range.EndOffset + 1 - range.StartOffset);

    byte[] buffer = new byte[rangeSize];

    // Read from the correct starting offset in the page blob and
    // place the data in the bufferOffset of the buffer byte array
    pageBlob.DownloadRangeToByteArray(buffer, bufferOffset, range.StartOffset, rangeSize); 

    // Then use the buffer for the page range just read
}
```

---

#### <a name="leasing-a-page-blob"></a>Concesión de un blob en páginas

La operación de concesión de blob establece y administra un bloqueo en un blob para las operaciones de escritura y lectura. Esta operación es útil en escenarios donde se tiene acceso desde varios clientes a un blob en páginas para asegurarse de que un único cliente puede escribir en el blob a la vez. Discos de Azure, por ejemplo, aprovecha este mecanismo de concesión para asegurar que solo una máquina virtual está administrando el disco. La duración del bloqueo puede ser de 15 a 60 segundos, o puede ser infinita. Consulte la documentación [aquí](/rest/api/storageservices/lease-blob) para obtener más detalles.

Además de API de REST enriquecidas, los blobs en páginas también proporcionan acceso compartido, durabilidad y seguridad mejorada. Trataremos esos beneficios con más detalle en los párrafos siguientes. 

### <a name="concurrent-access"></a>simultáneo

La API REST del blob en páginas y su mecanismo de concesión permite que las aplicaciones accedan al blob en páginas desde varios clientes. Por ejemplo, supongamos que necesita crear un servicio en la nube distribuido que comparte los objetos de almacenamiento con varios usuarios. Podría tratarse de una aplicación web que sirve una extensa colección de imágenes a varios usuarios. Una opción para esta implementación es usar una máquina virtual con discos conectados. Las desventajas de esto son (i) la restricción de que un disco solo puede adjuntarse a una única máquina virtual, y esto limita la escalabilidad, la flexibilidad y aumenta los riesgos. Si hay un problema con la máquina virtual o el servicio que se ejecuta en la máquina virtual, no se podrá acceder a la imagen, debido a la concesión, hasta que esta expire o se interrumpa. Otra desventaja es (ii) el costo adicional de tener una máquina virtual de IaaS. 

Una alternativa es usar los blobs en páginas directamente a través de la API REST de Azure Storage. Esta opción elimina la necesidad de costosas máquinas virtuales de IaaS, ofrece total flexibilidad de acceso directo desde varios clientes, simplifica el modelo de implementación clásica mediante la eliminación de la necesidad de asociar o desasociar discos, y elimina el riesgo de problemas en la máquina virtual. Además, proporciona el mismo nivel de rendimiento para las operaciones de lectura y escritura aleatorias que un disco.

### <a name="durability-and-high-availability"></a>Durabilidad y alta disponibilidad

Las opciones Standard Storage y Premium Storage ofrecen almacenamiento duradero donde los datos del blob en páginas siempre se replican para asegurar su durabilidad y alta disponibilidad. Para más información acerca de la redundancia de Azure Storage, consulte esta [documentación](../common/storage-redundancy.md). Azure ha ofrecido sistemáticamente durabilidad de nivel empresarial para discos IaaS y blobs en páginas, con un [Porcentaje de errores anualizado](https://en.wikipedia.org/wiki/Annualized_failure_rate) líder del sector del 0 %.

### <a name="seamless-migration-to-azure"></a>Migración sin problemas a Azure

Para los clientes y desarrolladores interesados en la implementación de su propia solución de copia de seguridad personalizada, Azure también ofrece instantáneas incrementales que solo contienen las diferencias. Esta característica evita el costo de la copia inicial completa, lo que reduce considerablemente el costo de copia de seguridad. Junto con la capacidad eficaz de lectura y copia de datos diferenciales, esta es otra eficaz funcionalidad que permite aún más innovaciones por parte de los desarrolladores, lo que conduce a una experiencia inmejorable de copia de seguridad y recuperación ante desastres en Azure. Puede configurar su propia copia de seguridad y recuperación ante desastres para máquinas virtuales en Azure mediante el uso de [Blob Snapshot](/rest/api/storageservices/snapshot-blob) API junto con [Get Page Ranges](/rest/api/storageservices/get-page-ranges) API y [Incremental Copy Blob](/rest/api/storageservices/incremental-copy-blob) API, que puede usar para copiar fácilmente datos incrementales para recuperación ante desastres. 

Además, muchas empresas tienen cargas de trabajo críticas que se ejecutan en centros de datos locales. Para migrar la carga de trabajo a la nube, una de las principales preocupaciones sería el tiempo de inactividad necesario para copiar los datos y el riesgo de imprevistos tras el cambio. En muchos casos, el tiempo de inactividad puede ser un factor negativo para la migración a la nube. Azure resuelve este problema con la API REST de los blobs en páginas, al habilitar la migración a la nube con una interrupción mínima para las cargas de trabajo críticas. 

Para obtener ejemplos acerca de cómo realizar una instantánea y cómo restaurar un blob en páginas desde una instantánea, consulte el artículo acerca de la [configuración de un proceso de copia de seguridad con instantáneas incrementales](../../virtual-machines/windows/incremental-snapshots.md).
