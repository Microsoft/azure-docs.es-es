---
title: Montaje de un sistema de archivos virtual en un grupo
description: Obtenga información sobre cómo montar un sistema de archivos virtual en un grupo de Batch.
ms.topic: how-to
ms.custom: devx-track-csharp
ms.date: 08/13/2019
ms.openlocfilehash: df03275fdeea88df1a2f2b6e2cda55021497cdf7
ms.sourcegitcommit: d7352c07708180a9293e8a0e7020b9dd3dd153ce
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2020
ms.locfileid: "89145491"
---
# <a name="mount-a-virtual-file-system-on-a-batch-pool"></a>Montaje de un sistema de archivos virtual en un grupo de Batch

Ahora Azure Batch admite el montaje de almacenamiento en la nube o un sistema de archivos externo en nodos de ejecución de Windows o Linux en los grupos de Batch. Cuando un nodo de ejecución se une a un grupo, el sistema de archivos virtual se monta y se trata como una unidad local en ese nodo. Puede montar sistemas de archivos como Azure Files, Azure Blob Storage, Network File System (NFS), incluida una [caché de Avere vFXT](../avere-vfxt/avere-vfxt-overview.md), o Sistema de archivos de Internet común (CIFS).

En este artículo, aprenderá a montar un sistema de archivos virtual en un grupo de nodos de ejecución mediante la [biblioteca de administración de Batch para .NET](/dotnet/api/overview/azure/batch).

> [!NOTE]
> El montaje de un sistema de archivos virtual se admite en grupos de Batch creados el 19 de agosto de 2019 o posteriormente. Los grupos de Batch creados antes de esta fecha no admiten esta característica.
> 
> Las API para el montaje de sistemas de archivos en un nodo de ejecución forman parte de la biblioteca de [Batch para .NET](/dotnet/api/microsoft.azure.batch).

## <a name="benefits-of-mounting-on-a-pool"></a>Ventajas del montaje en un grupo

Montar el sistema de archivos en el grupo, en lugar de permitir que las tareas recuperen sus propios datos de un conjunto de datos de gran tamaño, facilita y hace más eficaz el acceso de las tareas a los datos necesarios.

Considere un escenario con varias tareas que requieren acceso a un conjunto de datos común, como la representación de una película. Cada tarea representa uno o varios fotogramas a la vez de los archivos de la escena. Al montar una unidad que contiene los archivos de la escena, es más fácil que los nodos de ejecución tengan acceso a los datos compartidos. Además, el sistema de archivos subyacente se puede elegir y escalar de forma independiente según el rendimiento y la escala (rendimiento e IOPS) requeridos por el número de nodos de ejecución que tienen acceso simultáneamente a los datos. Por ejemplo, se puede usar una caché en memoria distribuida de [Avere vFXT](../avere-vfxt/avere-vfxt-overview.md) para admitir representaciones cinematográficas de gran tamaño con miles de nodos de representación simultáneos, teniendo acceso a los datos de origen que residen localmente. En el caso de los datos que ya residen en instancias de Blob Storage basadas en la nube, también se puede usar [blobfuse](../storage/blobs/storage-how-to-mount-container-linux.md) para montar estos datos como un sistema de archivos local. Blobfuse solo está disponible en los nodos de Linux; sin embargo, [Azure Files](https://azure.microsoft.com/blog/a-new-era-for-azure-files-bigger-faster-better/) proporciona un flujo de trabajo similar y está disponible en Windows y Linux.

## <a name="mount-a-virtual-file-system-on-a-pool"></a>Montaje de un sistema de archivos virtual en un grupo  

El montaje de un sistema de archivos virtual en un grupo hace que el sistema de archivos esté disponible para todos los nodos de ejecución del grupo. El sistema de archivos se configura cuando un nodo de ejecución se une a un grupo o cuando se reinicia el nodo o se restablece su imagen inicial.

Para montar un sistema de archivos en un grupo, cree un objeto `MountConfiguration`. Elija el objeto que se adapte al sistema de archivos virtual: `AzureBlobFileSystemConfiguration`, `AzureFileShareConfiguration`, `NfsMountConfiguration` o `CifsMountConfiguration`.

Todos los objetos de configuración de montaje necesitan los siguientes parámetros base. Algunas configuraciones de montaje tienen en uso parámetros específicos del sistema de archivos, que se describen con más detalle en los ejemplos de código.

- **Nombre de cuenta u origen**: para montar un recurso compartido de archivos virtual, necesita el nombre de la cuenta de almacenamiento o su origen.
- **Ruta de acceso de montaje relativa u origen**: ubicación del sistema de archivos montado en el nodo de ejecución, en relación con el directorio `fsmounts` estándar accesible en el nodo mediante `AZ_BATCH_NODE_MOUNTS_DIR`. La ubicación exacta varía en función del sistema operativo que se use en el nodo. Por ejemplo, la ubicación física en un nodo de Ubuntu está asignada a `mnt\batch\tasks\fsmounts`, y en un nodo de CentOS se asigna a `mnt\resources\batch\tasks\fsmounts`.
- **Opciones de montaje u opciones de blobfuse**: estas opciones describen parámetros específicos para el montaje de un sistema de archivos.

Una vez creado el objeto `MountConfiguration`, asigne el objeto a la propiedad `MountConfigurationList` al crear el grupo. El sistema de archivos se monta cuando un nodo se une a un grupo o cuando se reinicia el nodo o se restablece su imagen inicial.

Cuando se monta el sistema de archivos, se crea una variable de entorno `AZ_BATCH_NODE_MOUNTS_DIR` que apunta a la ubicación de los sistemas de archivos montados, así como a los archivos de registro, que son útiles para la solución de problemas y la depuración. Los archivos de registro se explican con más detalle en la sección [Diagnóstico de errores de montaje](#diagnose-mount-errors).  

> [!IMPORTANT]
> El número máximo de sistemas de archivos montados en un grupo es 10. Consulte [Límites y cuotas del servicio Batch](batch-quota-limit.md#other-limits) para obtener más información y otros límites.

## <a name="examples"></a>Ejemplos

En los ejemplos de código siguientes se muestra cómo montar una variedad de recursos compartidos de archivos en un grupo de nodos de ejecución.

### <a name="azure-files-share"></a>Recurso compartido de Azure Files

Azure Files es la oferta estándar del sistema de archivos en la nube de Azure. Para obtener más información sobre cómo obtener cualquiera de los parámetros en el ejemplo de código de configuración de montaje, consulte [Uso de un recurso compartido de Azure Files](../storage/files/storage-how-to-use-files-windows.md).

```csharp
new PoolAddParameter
{
    Id = poolId,
    MountConfiguration = new[]
    {
        new MountConfiguration
        {
            AzureFileShareConfiguration = new AzureFileShareConfiguration
            {
                AccountName = "{storage-account-name}",
                AzureFileUrl = "https://{storage-account-name}.file.core.windows.net/{file-share-name}",
                AccountKey = "{storage-account-key}",
                RelativeMountPath = "S",
                MountOptions = "-o vers=3.0,dir_mode=0777,file_mode=0777,sec=ntlmssp"
            },
        }
    }
}
```

### <a name="azure-blob-file-system"></a>Sistema de archivos de Azure Blob

Otra opción consiste en usar Azure Blob Storage a través de [blobfuse](../storage/blobs/storage-how-to-mount-container-linux.md). El montaje de un sistema de archivos de blob requiere `AccountKey` o `SasKey` para la cuenta de almacenamiento. Para obtener información sobre cómo obtener estas claves, consulte los artículos sobre la [administración de claves de cuenta de almacenamiento](../storage/common/storage-account-keys-manage.md) o el [uso de firmas de acceso compartido (SAS)](../storage/common/storage-sas-overview.md). Para obtener más información sobre el uso de blobfuse, consulte las [Troubleshoot FAQ](https://github.com/Azure/azure-storage-fuse/wiki/3.-Troubleshoot-FAQ) (Preguntas más frecuentes sobre la solución de problemas de blobfuse). Para obtener acceso predeterminado al directorio montado de blobfuse, ejecute la tarea como **administrador**. Blobfuse monta el directorio en el espacio de usuario y, cuando se crea el grupo, se monta como raíz. En Linux, todas las tareas de **administrador** son raíz. Todas las opciones del módulo FUSE se describen en la [página de referencia de FUSE](https://manpages.ubuntu.com/manpages/xenial/man8/mount.fuse.8.html).

Además de la guía de solución de problemas, los problemas de GitHub en el repositorio de blobfuse son una manera útil de comprobar los problemas y las soluciones actuales de blobfuse. Para obtener más información, consulte los [problemas de blobfuse](https://github.com/Azure/azure-storage-fuse/issues).

```csharp
new PoolAddParameter
{
    Id = poolId,
    MountConfiguration = new[]
    {
        new MountConfiguration
        {
            AzureBlobFileSystemConfiguration = new AzureBlobFileSystemConfiguration
            {
                AccountName = "StorageAccountName",
                ContainerName = "containerName",
                AccountKey = "StorageAccountKey",
                SasKey = "",
                RelativeMountPath = "RelativeMountPath",
                BlobfuseOptions = "-o attr_timeout=240 -o entry_timeout=240 -o negative_timeout=120 "
            },
        }
    }
}
```

### <a name="network-file-system"></a>Network File System

Los sistemas de Network File System (NFS) también se pueden montar en nodos de grupo, lo que permite que los nodos de Azure Batch puedan tener acceso fácilmente a los sistemas de archivos tradicionales. Puede tratarse de un solo servidor NFS implementado en la nube o un servidor NFS local al que se tiene acceso a través de una red virtual. También puede aprovechar las ventajas de la solución de caché en memoria distribuida de [Avere vFXT](../avere-vfxt/avere-vfxt-overview.md), que proporciona una conectividad sin problemas al almacenamiento local, mediante la lectura de datos a petición en su memoria caché, y ofrece un alto rendimiento y escala en nodos de ejecución basados en la nube.

```csharp
new PoolAddParameter
{
    Id = poolId,
    MountConfiguration = new[]
    {
        new MountConfiguration
        {
            NfsMountConfiguration = new NFSMountConfiguration
            {
                Source = "source",
                RelativeMountPath = "RelativeMountPath",
                MountOptions = "options ver=1.0"
            },
        }
    }
}
```

### <a name="common-internet-file-system"></a>Sistema de archivos de Internet común

Los sistemas de archivos de Internet común (CIFS) también se pueden montar en nodos de grupo, lo que permite que los nodos de Azure Batch puedan tener acceso fácilmente a los sistemas de archivos tradicionales. CIFS es un protocolo de uso compartido de archivos que proporciona un mecanismo abierto y multiplataforma para solicitar servicios y archivos de servidor de red. CIFS se basa en la versión mejorada del protocolo de bloque de mensajes del servidor (SMB) de Microsoft para el uso compartido de archivos de Internet e intranet y se usa para montar sistemas de archivos externos en nodos de Windows. Para obtener más información acerca de SMB, consulte [Servidor de archivos y SMB](/windows-server/storage/file-server/file-server-smb-overview).

```csharp
new PoolAddParameter
{
    Id = poolId,
    MountConfiguration = new[]
    {
        new MountConfiguration
        {
            CifsMountConfiguration = new CIFSMountConfiguration
            {
                Username = "StorageAccountName",
                RelativeMountPath = "cifsmountpoint",
                Source = "source",
                Password = "StorageAccountKey",
                MountOptions = "-o vers=3.0,dir_mode=0777,file_mode=0777,serverino"
            },
        }
    }
}
```

## <a name="diagnose-mount-errors"></a>Diagnóstico de errores de montaje

Si se produce un error en una configuración de montaje, se producirá un error en el nodo de ejecución del grupo y el estado del nodo pasará a ser inutilizable. Para diagnosticar un error de configuración de montaje, inspeccione la propiedad [`ComputeNodeError`](/rest/api/batchservice/computenode/get#computenodeerror) para obtener más información sobre el error.

Para obtener los archivos de registro para la depuración, use [OutputFiles](batch-task-output-files.md) para cargar los archivos `*.log`. Los archivos `*.log` contienen información sobre el montaje del sistema de archivos en la ubicación `AZ_BATCH_NODE_MOUNTS_DIR`. Los archivos de registro de montaje tienen el formato `<type>-<mountDirOrDrive>.log` para cada montaje. Por ejemplo, un montaje `cifs` en un directorio de montaje denominado `test` tendrá un archivo de registro de montaje denominado `cifs-test.log`.

## <a name="supported-skus"></a>SKU compatibles

| Publicador | Oferta | SKU | Recurso compartido de Azure Files | Blobfuse | Montaje de NFS | Montaje de CIFS |
|---|---|---|---|---|---|---|
| proceso por lotes | rendering-centos73 | rendering | :heavy_check_mark: <br>Nota: Compatible con CentOS 7.7</br>| :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Canonical | UbuntuServer | 16.04-LTS, 18.04-LTS | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Credativ | Debian | 8| :heavy_check_mark: | :x: | :heavy_check_mark: | :heavy_check_mark: |
| Credativ | Debian | 9 | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| microsoft-ads | linux-data-science-vm | linuxdsvm | :heavy_check_mark: <br>Nota: Compatible con CentOS 7.4. </br> | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| microsoft-azure-batch | centos-container | 7.6 | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| microsoft-azure-batch | centos-container-rdma | 7.4 | :heavy_check_mark: <br>Nota: Admite almacenamiento A_8 o 9</br> | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| microsoft-azure-batch | ubuntu-server-container | 16.04-LTS | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| microsoft-dsvm | linux-data-science-vm-ubuntu | linuxdsvmubuntu | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| OpenLogic | CentOS | 7.6 | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| OpenLogic | CentOS-HPC | 7.4, 7.3, 7.1 | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Oracle | Oracle-Linux | 7.6 | :x: | :x: | :x: | :x: |
| Windows | Windows Server | 2012, 2016, 2019 | :heavy_check_mark: | :x: | :x: | :x: |

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre el montaje de un recurso compartido de Azure Files con [Windows](../storage/files/storage-how-to-use-files-windows.md) o [Linux](../storage/files/storage-how-to-use-files-linux.md).
- Obtenga información sobre el uso y el montaje de sistemas de archivos virtuales de [blobfuse](https://github.com/Azure/azure-storage-fuse).
- Consulte [Network File System overview](/windows-server/storage/nfs/nfs-overview) (Introducción a Network File System) para obtener información sobre NFS y sus aplicaciones.
- Consulte [Microsoft SMB protocol and CIFS protocol overview](/windows/desktop/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) (Introducción a los protocolos CIFS y SMB de Microsoft) para obtener más información sobre CIFS.
