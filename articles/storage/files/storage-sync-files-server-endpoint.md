---
title: Adición o eliminación de un punto de conexión de servidor de Azure File Sync | Microsoft Docs
description: Aprenda a agregar o quitar un punto de conexión de servidor con Azure File Sync. Un punto de conexión de servidor es una ubicación determinada en un servidor registrado, como una carpeta en un volumen de servidor.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 07/19/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 9273ca66c0304afc5df58ace5dd584c20c90abfd
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87905064"
---
# <a name="addremove-an-azure-file-sync-server-endpoint"></a>Adición o eliminación de un punto de conexión de servidor de Azure File Sync
Azure File Sync le permite centralizar los recursos compartidos de archivos de su organización en Azure Files sin renunciar a la flexibilidad, el rendimiento y la compatibilidad de un servidor de archivos local. Para ello, transforma los servidores de Windows Server en una caché rápida del recurso compartido de archivos de Azure. Puede usar cualquier protocolo disponible en Windows Server para tener acceso a los datos localmente (incluidos SMB, NFS y FTPS) y puede tener tantas cachés según sea necesario en todo el mundo.

Un *punto de conexión de servidor* representa una ubicación específica en un *servidor registrado*, como una carpeta en un volumen de servidor o la raíz del volumen. Pueden existir varios puntos de conexión de servidor en el mismo volumen si sus espacios de nombres no se superponen (por ejemplo, F:\sync1 y F:\sync2). Además, puede configurar directivas de niveles de nube de manera individual para cada punto de conexión de servidor. Si agrega una ubicación de servidor con un conjunto de archivos existente como punto de conexión de servidor a un grupo de sincronización, esos archivos se combinarán con otros archivos que ya se encuentren en otros puntos de conexión del grupo de sincronización.

Para más información sobre cómo implementar Azure File Sync de un extremo a otro, vea [How to deploy Azure File Sync](storage-sync-files-deployment-guide.md) (Implementación de Azure Files Sync).

## <a name="prerequisites"></a>Prerequisites
Para crear un punto de conexión de servidor, primero debe asegurarse de que se cumplen los criterios siguientes: 
- El servidor tiene instalado el agente de Azure File Sync y se ha registrado. Puede encontrar instrucciones para instalar el agente de Azure File Sync en el artículo [Registro y cancelación del registro de un servidor con Azure File Sync](storage-sync-files-server-registration.md). 
- Asegúrese de que se ha implementado un servicio de sincronización de almacenamiento. Para más información sobre cómo implementar un servicio de sincronización de almacenamiento, vea [How to deploy Azure File Sync](storage-sync-files-deployment-guide.md) (Implementación de Azure Files Sync). 
- Asegúrese de que se ha implementado un grupo de sincronización. Aprenda a [crear un grupo de sincronización](storage-sync-files-deployment-guide.md#create-a sync-group-and-a-cloud-endpoint).
- Asegúrese de que el servidor está conectado a Internet y que se puede acceder a Azure. Se usa el puerto 443 para todas las comunicaciones entre el servidor y el servicio.

## <a name="add-a-server-endpoint"></a>Adición de un punto de conexión de servidor
Para agregar un punto de conexión de servidor, vaya al grupo de sincronización que prefiera y seleccione "Agregar punto de conexión del servidor".

![Adición de un nuevo punto de conexión de servidor al panel Grupo de sincronización](media/storage-sync-files-server-endpoint/add-server-endpoint-1.png)

La siguiente información es necesaria en **Agregar punto de conexión del servidor**:

- **Servidor registrado**: nombre del servidor o el clúster en el que crear el punto de conexión de servidor.
- **Ruta de acceso**: ruta de acceso del servidor de Windows Server que se va a sincronizar como parte del grupo de sincronización.
- **Niveles de la nube**: un conmutador para habilitar o deshabilitar los niveles de la nube. Cuando está habilitada, la característica de niveles de nube *apilará* los archivos en los recursos compartidos de archivos de Azure. Esto convierte los recursos compartidos de archivos locales en una caché, en lugar de en una copia completa del conjunto de datos, para ayudarlo a administrar la eficiencia del espacio en su servidor.
- **Espacio disponible del volumen**: cantidad de espacio libre que se reserva en el volumen en el que reside el punto de conexión de servidor. Por ejemplo, si el espacio disponible del volumen se establece en el 50 % en un volumen con un único punto de conexión de servidor, casi la mitad de la cantidad de datos se coloca en niveles en Azure Files. Con independencia de si la característica de niveles en la nube está habilitada, el recurso compartido de archivos de Azure siempre tiene una copia completa de los datos en el grupo de sincronización.

Seleccione **Crear** para agregar el punto de conexión de servidor. Los archivos de un espacio de nombres de un grupo de sincronización se mantienen ahora sincronizados. 

## <a name="remove-a-server-endpoint"></a>Eliminación de un punto de conexión de servidor
Si desea dejar de utilizar Azure File Sync para un punto de conexión de servidor determinado, puede quitar el punto de conexión del servidor. 

> [!Warning]  
> No intente solucionar los problemas de sincronización, niveles de la nube ni otros aspectos de Azure File Sync mediante la eliminación y nueva creación de los puntos de conexión de un servidor, salvo que lo indique explícitamente un ingeniero de Microsoft. La eliminación de un punto de conexión de un servidor es una operación destructiva, y los archivos en capas del punto de conexión de servidor no se "volverán a conectar" a sus ubicaciones del recurso compartido de archivos de Azure después de que el servidor se vuelva a crear, lo que provocará errores de sincronización. Tenga también en cuenta que los archivos en capas que haya fuera de un espacio de nombres de un punto de conexión de servidor pueden perderse de forma permanente. Puede haber archivos en capas en los puntos de conexión de un servidor aunque nunca se hayan habilitado los niveles de la nube.

Para asegurarse de que todos los archivos en capas se recuperen antes de eliminar el punto de conexión del servidor, deshabilite los niveles de la nube en el punto final del servidor y luego ejecute el siguiente cmdlet de PowerShell para recuperar todos los archivos en capas dentro del espacio de nombres del punto de conexión del servidor:

```PowerShell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Invoke-StorageSyncFileRecall -Path <path-to-to-your-server-endpoint> -Order CloudTieringPolicy
```
Si se especifica el parámetro `-Order CloudTieringPolicy`, se recuperarán primero los archivos modificados más recientemente.
Los siguientes son otros parámetros opcionales pero útiles que se deben tener en cuenta:
* `-ThreadCount` determina el número de archivos que se pueden recuperar en paralelo.
* `-PerFileRetryCount` determina la frecuencia con que se intentará recuperar un archivo que está bloqueado actualmente.
* `-PerFileRetryDelaySeconds` determina el tiempo en segundos entre los reintentos de recuperación y siempre se debe usar en combinación con el parámetro anterior.

> [!Note]  
> Si el volumen local que hospeda el servidor no tiene suficiente espacio disponible para recuperar todos los datos con niveles, el cmdlet `Invoke-StorageSyncFileRecall` dará error.  

Para quitar el punto de conexión de servidor:

1. Vaya al servicio de sincronización de almacenamiento donde está registrado el servidor.
2. Desplácese al grupo de sincronización deseado.
3. Quite el punto de conexión de servidor que desee del grupo de sincronización del servicio de sincronización de almacenamiento. Para ello, haga clic con el botón derecho en el punto de conexión de servidor que le interese en el panel Grupo de sincronización.

    ![Eliminación de un punto de conexión de servidor de un grupo de sincronización](media/storage-sync-files-server-endpoint/remove-server-endpoint-1.png)

## <a name="next-steps"></a>Pasos siguientes
- [Registro y cancelación del registro de un servidor con Azure File Sync](storage-sync-files-server-registration.md)
- [Planeamiento de una implementación de Azure File Sync](storage-sync-files-planning.md)
- [Supervisión de Azure File Sync](storage-sync-files-monitoring.md)
