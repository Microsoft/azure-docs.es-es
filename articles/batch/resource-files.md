---
title: Crear y usar archivos de recursos
description: Aprenda a crear archivos de recursos de Azure Batch desde diversos orígenes de entrada. En este artículo se abordan algunos métodos comunes para crear archivos de recursos y colocarlos en una máquina virtual.
ms.date: 03/18/2020
ms.topic: how-to
ms.openlocfilehash: 22c0220d08660402bef3fd4aaf0add6adc12a295
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89230949"
---
# <a name="creating-and-using-resource-files"></a>Crear y usar archivos de recursos

A menudo, una tarea de Azure Batch requiere algún tipo de procesamiento de datos. Los archivos de recursos constituyen el modo de proporcionar estos datos a la máquina virtual de Batch a través de una tarea. Todos los tipos de tareas admiten archivos de recursos: tareas de inicio, tareas de preparación del trabajo, tareas de liberación del trabajo, etc. En este artículo se abordan algunos métodos comunes para crear archivos de recursos y colocarlos en una máquina virtual.  

Los archivos de recursos colocan datos en una máquina virtual en Batch. El tipo de datos y cómo se usen son flexibles. De todas formas, existen algunos casos de uso comunes:

1. Aprovisionamiento de archivos comunes en cada máquina virtual usando archivos de recursos en una tarea de inicio
1. Aprovisionamiento de datos de entrada que se van a procesar mediante tareas

Los archivos comunes pueden ser, por ejemplo, archivos de una tarea de inicio usados para instalar las aplicaciones que ejecutan las tareas. Los datos de entrada pueden ser cualquier dato de vídeo o imagen sin procesar, o información que Batch va a procesar.

## <a name="types-of-resource-files"></a>Tipos de archivos de recursos

Existen diversas opciones disponibles para generar archivos de recursos. El proceso de creación de archivos de recursos varía según dónde estén almacenados los datos originales.

Opciones para crear un archivo de recursos:

- [Dirección URL del contenedor de almacenamiento](#storage-container-url): genera un archivo de recursos a partir de cualquier contenedor de almacenamiento de Azure.
- [Nombre del contenedor de almacenamiento](#storage-container-name): genera un archivo de recursos a partir del nombre de un contenedor de una cuenta de Azure Storage vinculada a Batch.
- [Punto de conexión web](#web-endpoint): genera un archivo de recursos a partir de una dirección URL HTTP válida.

### <a name="storage-container-url"></a>Dirección URL del contenedor de almacenamiento

El uso de una dirección URL de contenedor de almacenamiento conlleva que se puede acceder a los archivos de cualquier contenedor de almacenamiento de Azure, si se tienen los permisos correctos. 

En este ejemplo de C#, los archivos ya se han cargado en un contenedor de almacenamiento de Azure como Blob Storage. Para acceder a los datos necesarios para crear un archivo de recursos, primero hay que acceder al contenedor de almacenamiento.

Cree un URI de firma de acceso compartido (SAS) con los permisos correctos para acceder al contenedor de almacenamiento. Establezca la fecha de vencimiento y los permisos de la SAS. En este caso, no se especifica ninguna fecha de inicio, por lo que la SAS entrará en vigor inmediatamente y expirará dos horas después de que se genere.

```csharp
SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
{
    SharedAccessExpiryTime = DateTime.UtcNow.AddHours(2),
    Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.List
};
```

> [!NOTE]
> Para acceder al contenedor, debe tener los permisos `Read` y `List`, mientras que para acceder al blob solo se necesita el permiso `Read`.

Una vez configurados los permisos, cree el token de la SAS y dé formato a la dirección URL de la SAS para acceder al contenedor de almacenamiento. Usando la dirección URL de la SAS (con el formato adecuado) del contenedor de almacenamiento, genere un archivo de recursos con [`FromStorageContainerUrl`](/dotnet/api/microsoft.azure.batch.resourcefile.fromstoragecontainerurl).

```csharp
CloudBlobContainer container = blobClient.GetContainerReference(containerName);

string sasToken = container.GetSharedAccessSignature(sasConstraints);
string containerSasUrl = String.Format("{0}{1}", container.Uri, sasToken);

ResourceFile inputFile = ResourceFile.FromStorageContainerUrl(containerSasUrl);
```

Otra forma de generar una dirección URL de SAS consiste en habilitar el acceso de lectura anónimo y público a un contenedor y sus blobs en Azure Blob Storage. Al hacerlo, puede conceder acceso de solo lectura a estos recursos sin compartir la clave de cuenta y sin necesidad de una SAS. El acceso de lectura público se suele usar en escenarios donde se quiere que ciertos blobs estén siempre disponibles para el acceso de lectura anónimo. Si este escenario encaja con su solución, vea el artículo [Administración del acceso de lectura anónimo a contenedores y blobs](../storage/blobs/storage-manage-access-to-resources.md) para obtener más información sobre cómo administrar el acceso a los datos de blob.

### <a name="storage-container-name"></a>Nombre del contenedor de almacenamiento

En lugar de configurar y crear una dirección URL de SAS, puede usar el nombre del contenedor de almacenamiento de Azure para tener acceso a los datos de blob. El contenedor de almacenamiento que use debe estar en la cuenta de Azure Storage que esté vinculada a su cuenta de Batch. Dicha cuenta de almacenamiento se conoce como la cuenta de almacenamiento automático. El contenedor de almacenamiento automático permite omitir la configuración y creación de una dirección URL de SAS para acceder a un contenedor de almacenamiento.

En este ejemplo, se da por hecho que los datos que se van a usar para crear el archivo de recursos ya están en una cuenta de Azure Storage vinculada a su cuenta de Batch. Si no tiene una cuenta de almacenamiento automático, vea los pasos de [Creación de una cuenta de Batch](batch-account-create-portal.md) para obtener más información sobre cómo crear y vincular una cuenta.

Si se usa una cuenta de almacenamiento vinculada, no es necesario crear y configurar una dirección URL de SAS que lleve a un contenedor de almacenamiento; solo tiene que indicar el nombre del contenedor de almacenamiento en la cuenta de almacenamiento vinculada.

```csharp
ResourceFile inputFile = ResourceFile.FromAutoStorageContainer(containerName);
```

### <a name="web-endpoint"></a>Punto de conexión web

Los datos que no se cargan en Azure Storage se pueden seguir usando para crear archivos de recursos. Puede especificar cualquier dirección URL HTTP válida que contenga los datos de entrada. La dirección URL se suministra a la API de Batch y, seguidamente, se usan los datos para crear un archivo de recursos.

En el siguiente ejemplo de C#, los datos de entrada están hospedados en un punto de conexión de GitHub ficticio. La API recupera el archivo desde el punto de conexión web válido y genera un archivo de recursos que la tarea consumirá. En este escenario no se requieren credenciales.

```csharp
ResourceFile inputFile = ResourceFile.FromUrl("https://github.com/foo/file.txt", filePath);
```

## <a name="tips-and-suggestions"></a>Recomendaciones y sugerencias

Cada tarea de Azure Batch usa los archivos de forma diferente, y ese es el motivo por el que Batch tiene opciones disponibles para administrar archivos en las tareas. Los siguientes escenarios no pretenden ser exhaustivos, sino que solo abordan algunas situaciones comunes y ofrecen recomendaciones.

### <a name="many-resource-files"></a>Muchos archivos de recursos

Su trabajo de Batch puede contener varias tareas, y que todas ellas usen los mismos archivos comunes. Si los archivos de tareas comunes se comparten entre muchas tareas, puede que sea mejor opción usar un paquete de aplicación que contenga los archivos, en vez de usar archivos de recursos. Los paquetes de aplicación proporcionan una velocidad de descarga optimizada. Además, los datos de los paquetes de aplicación se almacenan en la memoria caché entre las tareas, de modo que, si los archivos de tarea no cambian con frecuencia, los paquetes de aplicación pueden ser una buena elección para su solución. Con los paquetes de aplicación, no es necesario administrar varios archivos de recursos manualmente ni generar direcciones URL de SAS para acceder a los archivos en Azure Storage. Batch funciona en segundo plano con Azure Storage para almacenar paquetes de aplicación e implementarlos en los nodos de proceso.

Si una tarea tiene muchos archivos únicos, los archivos de recursos son la mejor opción, ya que es habitual que las tareas que usan archivos únicos tengan que actualizarse o reemplazarse, lo que no resulta fácil con el contenido de los paquetes de aplicaciones. Los archivos de recursos reportan una mayor flexibilidad a la hora de actualizar, agregar o modificar archivos individuales.

### <a name="number-of-resource-files-per-task"></a>Número de archivos de recursos por tarea

Si hay varios cientos de archivos de recursos especificados en una tarea, Batch puede rechazar la tarea por ser demasiado grande. Lo mejor es mantener tareas pequeñas, reduciendo para ello el número de archivos de recursos en la propia tarea.

Si no hay ninguna manera de reducir el número de archivos que la tarea necesita, puede optimizar la tarea creando un único archivo de recursos que haga referencia a un contenedor de almacenamiento de archivos de recursos. Para ello, coloque los archivos de recursos en un contenedor de Azure Storage y use los diferentes [métodos](/dotnet/api/microsoft.azure.batch.resourcefile#methods) "contenedor" de los archivos de recursos. Use las opciones de prefijo de blob para especificar colecciones de archivos para descargarlas para sus tareas.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre los [paquetes de aplicación](batch-application-packages.md) como alternativa a los archivos de recursos.
- Para obtener más información sobre cómo usar contenedores para los archivos de recursos, vea [Cargas de trabajo de contenedor](batch-docker-container-workloads.md).
- Para obtener información sobre cómo recopilar y guardar los datos de salida de las tareas, vea [Trabajo persistente y resultado de la tarea](batch-task-output.md).
- Obtenga información acerca de las [API y herramientas de Batch](batch-apis-tools.md) disponibles para la creación de soluciones de Batch.
