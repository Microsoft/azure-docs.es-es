---
title: Planeamiento de una implementación de Azure File Sync |Microsoft Docs
description: Conozca los aspectos que debe tener en cuenta al planear una implementación de Azure Files.
services: storage
author: roygara
ms.service: storage
ms.topic: article
ms.date: 2/7/2019
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 7cbb934b87440d23e65fce53d7da40c5ffbd3150
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65597081"
---
# <a name="planning-for-an-azure-file-sync-deployment"></a>Planeamiento de una implementación de Azure Files Sync
Use Azure File Sync para centralizar los recursos compartidos de archivos de su organización en Azure Files sin renunciar a la flexibilidad, el rendimiento y la compatibilidad de un servidor de archivos local. Azure File Sync transforma Windows Server en una caché rápida de los recursos compartidos de archivos de Azure. Puede usar cualquier protocolo disponible en Windows Server para acceder a sus datos localmente, como SMB, NFS y FTPS. Puede tener todas las cachés que necesite en todo el mundo.

Este artículo describe las consideraciones importantes para una implementación de Azure File Sync. Se recomienda que lea [Planeamiento de una implementación de Azure Files](storage-files-planning.md). 

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="azure-file-sync-terminology"></a>Terminología de Azure File Sync
Antes de entrar en detalles para planear una implementación de Azure File Sync, es importante comprender la terminología.

### <a name="storage-sync-service"></a>Servicio de sincronización de almacenamiento
El servicio de sincronización de almacenamiento es el recurso de Azure de nivel superior para Azure File Sync. El recurso del servicio de sincronización de almacenamiento es un homólogo del recurso de la cuenta de almacenamiento, y se puede implementar igualmente en grupos de recursos de Azure. Se requiere un recurso de nivel superior distinto del recurso de la cuenta de almacenamiento dado que el servicio de sincronización de almacenamiento puede crear relaciones de sincronización con varias cuentas de almacenamiento mediante varios grupos de sincronización. Una suscripción puede tener implementados varios recursos del servicio de sincronización de almacenamiento.

### <a name="sync-group"></a>Grupo de sincronización
Un grupo de sincronización define la topología de sincronización de un conjunto de archivos. Los puntos de conexión dentro de un grupo de sincronización se mantienen sincronizados entre sí. Si, por ejemplo, tiene dos conjuntos distintos de archivos que desea administrar con Azure File Sync, podría crear dos grupos de sincronización y agregar a cada uno puntos de conexión diferentes. Un servicio de sincronización de almacenamiento puede hospedar tantos grupos de sincronización como sean necesarios.  

### <a name="registered-server"></a>Servidor registrado
El objeto de servidor registrado representa una relación de confianza entre el servidor (o clúster) y el servicio de sincronización de almacenamiento. Puede registrar tantos servidores como desee en una instancia del servicio de sincronización de almacenamiento. Sin embargo, un servidor (o clúster) se puede registrar con un solo servicio de sincronización de almacenamiento a la vez.

### <a name="azure-file-sync-agent"></a>Agente de Azure File Sync
El agente de Azure File Sync es un paquete descargable que permite la sincronización de Windows Server con un recurso compartido de archivos de Azure. El agente de Azure File Sync consta de tres componentes principales: 
- **FileSyncSvc.exe**: el servicio de Windows en segundo plano es responsable de supervisar los cambios en los puntos de conexión de servidor y de iniciar las sesiones de sincronización en Azure.
- **StorageSync.sys**: el filtro del sistema de archivos de Azure File Sync, responsable de apilar los archivos en Azure Files (cuando está habilitada la característica de nube por niveles).
- **Cmdlets de administración de PowerShell**: cmdlets de PowerShell para la interacción con el proveedor de recursos de Azure Microsoft.StorageSync. Puede encontrar estos elementos en las siguientes ubicaciones (valor predeterminado):
    - C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.PowerShell.Cmdlets.dll
    - C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll

### <a name="server-endpoint"></a>Punto de conexión de servidor
Un punto de conexión de servidor representa una ubicación específica en un servidor registrado, como una carpeta en un volumen de servidor. Pueden existir varios puntos de conexión de servidor en el mismo volumen si sus espacios de nombres no se superponen (por ejemplo, `F:\sync1` y `F:\sync2`). Además, puede configurar directivas de niveles de nube de manera individual para cada punto de conexión de servidor. 

Puede crear un punto de conexión de servidor a través de un punto de montaje. Tenga en cuenta que se omiten los puntos de montaje en el punto de conexión de servidor.  

Puede crear un punto de conexión de servidor en el volumen del sistema, pero si lo hace existen dos limitaciones:
* No se pueden habilitar los niveles de nube.
* La restauración rápida del espacio de nombres (en la que el sistema desactiva todo el espacio de nombres y, a continuación, empieza a recuperar contenido) no se realiza.


> [!Note]  
> Solo se admiten volúmenes no extraíbles.  Las unidades asignadas desde un recurso compartido remoto no son compatibles con rutas de acceso de punto de conexión de servidor.  Además, un punto de conexión de servidor puede encontrarse en el volumen del sistema de Windows, aunque los niveles de nube no se admitan en dicho volumen.

Si agrega una ubicación de servidor con un conjunto de archivos existente como punto de conexión de servidor a un grupo de sincronización, esos archivos se combinarán con otros archivos que ya se encuentren en otros puntos de conexión del grupo de sincronización.

### <a name="cloud-endpoint"></a>Punto de conexión de nube
Un punto de conexión de nube es un recurso compartido de archivos de Azure que forma parte de un grupo de sincronización. El recurso compartido de archivos de Azure entero se sincroniza; un recurso compartido de archivos de Azure solo puede ser miembro de un punto de conexión de nube. Por lo tanto, un recurso compartido de archivos de Azure puede ser un miembro de un solo grupo de sincronización. Si agrega un recurso compartido de archivos de Azure con un conjunto de archivos existente como punto de conexión de nube a un grupo de sincronización, esos archivos se combinarán con otros archivos que ya se encuentren en otros puntos de conexión del grupo de sincronización.

> [!Important]  
> Azure File Sync admite realizar cambios directamente en el recurso compartido de archivos de Azure. Sin embargo, los cambios realizados en el recurso compartido de archivos de Azure primero deben ser detectados por un trabajo de detección de cambios de Azure File Sync. Se inicia un trabajo de detección de cambios para un punto de conexión en la nube solo una vez cada 24 horas. Además, los cambios realizados en un recurso compartido de archivos de Azure a través del protocolo de REST no actualizarán la hora de la última modificación de SMB y no se verán como cambios derivados de la sincronización. Para obtener más información, consulte [Preguntas más frecuentes de Azure Files](storage-files-faq.md#afs-change-detection).

### <a name="cloud-tiering"></a>Niveles de nube 
La nube por niveles es una característica opcional de Azure File Sync por la que los archivos a los que se tiene acceso con frecuencia se almacenan en caché localmente en el servidor mientras que todos los demás archivos se organizan en niveles en Azure Files, según la configuración de directiva. Para saber más, vea [Cloud Tiering Overview](storage-sync-cloud-tiering.md) (Introducción a nube por niveles).

## <a name="azure-file-sync-system-requirements-and-interoperability"></a>Requisitos del sistema de Azure File Sync e interoperabilidad 
En esta sección se tratan los requisitos del sistema del agente de Azure File Sync y la interoperabilidad con las características y roles de Windows Server y las soluciones de terceros.

### <a name="evaluation-tool"></a>Herramienta de evaluación
Antes de implementar Azure File Sync, debe evaluar si es compatible con el sistema mediante la herramienta de evaluación de Azure File Sync. Esta herramienta es un cmdlet de Azure PowerShell que busca posibles problemas con el sistema de archivos y el conjunto de datos, tales como caracteres no admitidos o una versión de sistema operativo no compatible. Tenga en cuenta que las comprobaciones incluyen la mayoría de las características que se mencionan a continuación, pero no todas; se recomienda que lea el resto de esta sección detenidamente para asegurarse de que la implementación se realiza sin problemas. 

#### <a name="download-instructions"></a>Instrucciones de descarga
1. Asegúrese de que tiene las versiones más recientes de PackageManagement y PowerShellGet instaladas (esto le permite instalar módulos de versión preliminar).
    
    ```powershell
        Install-Module -Name PackageManagement -Repository PSGallery -Force
        Install-Module -Name PowerShellGet -Repository PSGallery -Force
    ```
 
2. Reinicie PowerShell.
3. Instalación de los módulos
    
    ```powershell
        Install-Module -Name Az.StorageSync -AllowPrerelease -AllowClobber -Force
    ```

#### <a name="usage"></a>Uso  
Puede invocar la herramienta de evaluación de varias maneras diferentes: puede realizar las comprobaciones del sistema, las comprobaciones del conjunto de datos o ambas. Para realizar las comprobaciones del sistema y el conjunto de datos: 

```powershell
    Invoke-AzStorageSyncCompatibilityCheck -Path <path>
```

Para probar solo el conjunto de datos:
```powershell
    Invoke-AzStorageSyncCompatibilityCheck -Path <path> -SkipSystemChecks
```
 
Para probar solo los requisitos del sistema:
```powershell
    Invoke-AzStorageSyncCompatibilityCheck -ComputerName <computer name>
```
 
Para mostrar los resultados en CSV:
```powershell
    $errors = Invoke-AzStorageSyncCompatibilityCheck […]
    $errors | Select-Object -Property Type, Path, Level, Description | Export-Csv -Path <csv path>
```

### <a name="system-requirements"></a>Requisitos de sistema
- Un servidor que ejecute Windows Server 2012 R2, Windows Server 2016 o Windows Server 2019:

    | Version | SKU compatibles | Opciones de implementación compatibles |
    |---------|----------------|------------------------------|
    | Windows Server 2019 | Datacenter y Standard | Completo (servidor con una interfaz de usuario) |
    | Windows Server 2016 | Datacenter y Standard | Completo (servidor con una interfaz de usuario) |
    | Windows Server 2012 R2 | Datacenter y Standard | Completo (servidor con una interfaz de usuario) |

    Las versiones futuras de Windows Server se agregarán tan pronto como se publiquen. Las versiones anteriores de Windows podrían agregarse en función de los comentarios del usuario.

    > [!Important]  
    > Se recomienda mantener sincronizadas todas las instancias que use con Azure File Sync con las actualizaciones más recientes de Windows Update. 

- Un servidor con un mínimo de 2 GiB de memoria.

    > [!Important]  
    > Si el servidor se ejecuta en una máquina virtual con la memoria dinámica habilitada, la máquina virtual debe configurarse con un mínimo de 2048 MiB de memoria.
    
- Un volumen asociado localmente con formato del sistema de archivos NTFS.

### <a name="file-system-features"></a>Características del sistema de archivos

| Característica | Compatibilidad con el estado | Notas |
|---------|----------------|-------|
| Listas de control de acceso (ACL) | Totalmente compatible | Azure File Sync conserva las listas ACL de Windows y Windows Server las exige en los puntos de conexión de servidor. Las listas ACL de Windows no se admiten (aún) en Azure Files cuando se accede directamente a los archivos en la nube. |
| Vínculos físicos | Omitido | |
| Vínculos simbólicos | Omitido | |
| Puntos de montaje | Compatibilidad parcial | Los puntos de montaje podrían ser la raíz de un punto de conexión de servidor, pero se omiten si están incluidos en el espacio de nombres del punto de conexión de servidor. |
| Uniones | Omitido | Por ejemplo, las carpetas DfrsrPrivate y DFSRoots del Sistema de archivos distribuido. |
| Puntos de repetición de análisis | Omitido | |
| Compresión NTFS | Totalmente compatible | |
| Archivos dispersos | Totalmente compatible | Los archivos dispersos se sincronizan (no se bloquean), pero lo hacen con la nube como un archivo completo. Si se cambia el contenido del archivo en la nube (o en otro servidor), el archivo ya no estará disperso cuando el cambio se haya descargado. |
| Flujos de datos alternativos (ADS) | Conservados, pero no sincronizados | Por ejemplo, las etiquetas de clasificación creadas por la infraestructura de clasificación de archivos no están sincronizadas. Las etiquetas de clasificación existentes en los archivos en cada uno de los puntos de conexión del servidor se dejan como están. |

> [!Note]  
> Solo se admiten volúmenes NTFS. No se admiten ReFS, FAT, FAT32 ni otros sistemas de archivos.

### <a name="files-skipped"></a>Archivos omitidos

| Archivo/carpeta | NOTA |
|-|-|
| Desktop.ini | Archivo específico del sistema |
| ethumbs.DB$ | Archivo temporal para miniaturas |
| ~$\*.\* | Archivo temporal de Office |
| \*.tmp | Archivo temporal |
| \*.laccdb | Archivo de bloqueo de base de datos de Access|
| 635D02A9D91C401B97884B82B3BCDAEA.* | Archivo de sincronización interna|
| \\System Volume Information | Carpeta específica del volumen |
| $RECYCLE.BIN| Carpeta |
| \\SyncShareState | Carpeta para sincronización |

### <a name="failover-clustering"></a>Clústeres de conmutación por error
La característica de clústeres de conmutación por error de Windows es compatible con Azure File Sync en la opción de implementación "Servidor de archivos para uso general". La característica de clústeres de conmutación por error no se admite en "Servidor de archivos de escalabilidad horizontal para datos de aplicación" (SOFS) o en volúmenes compartidos de clúster (CSV).

> [!Note]  
> El agente de Azure File Sync debe estar instalado en cada nodo de un clúster de conmutación por error para que la sincronización funcione correctamente.

### <a name="data-deduplication"></a>Desduplicación de datos
**Versión del agente 5.0.2.0**   
La desduplicación de datos ahora es compatible en volúmenes con la nube por niveles habilitada en Windows Server 2016 y Windows Server 2019. Habilitar la desduplicación en un volumen con la nube por niveles habilitada, le permite almacenar en caché más archivos en el entorno local sin necesidad de aprovisionar más almacenamiento.

**Windows Server 2012 R2 o versiones anteriores del agente**  
En el caso de volúmenes que no tengan habilitada la característica de niveles de nube, Azure File Sync admite la habilitación de la desduplicación de datos de Windows Server en el volumen.

### <a name="distributed-file-system-dfs"></a>Sistema de archivos distribuido (DFS)
Azure File Sync admite la interoperabilidad con espacios de nombres DFS (DFS-N) y la replicación DFS (DFS-R).

**Espacios de nombres DFS (DFS-N)**: Azure File Sync es totalmente compatible con servidores de DFS-N. Puede instalar el agente de Azure File Sync en uno o varios miembros DFS-N para sincronizar datos entre los puntos de conexión del servidor y el punto de conexión en la nube. Para más información, consulte [Información general de Espacios de nombres DFS](https://docs.microsoft.com/windows-server/storage/dfs-namespaces/dfs-overview).
 
**Replicación DFS (DFS-R)**: puesto que DFS-R y Azure File Sync son soluciones de replicación, en la mayoría de los casos, se recomienda reemplazar DFS-R por Azure File Sync. Hay, sin embargo, varios escenarios donde puede que desee usar DFS-R y Azure File Sync conjuntamente:

- Va a migrar desde una implementación de DFS-R a una implementación de Azure File Sync. Para más información, consulte [Migrate a DFS Replication (DFS-R) deployment to Azure File Sync](storage-sync-files-deployment-guide.md#migrate-a-dfs-replication-dfs-r-deployment-to-azure-file-sync) (Migración de una implementación de la replicación DFS (DFS-R) a Azure File Sync).
- No todos los servidores locales que necesitan una copia de los datos de archivo pueden estar conectados directamente a Internet.
- Los servidores de sucursales consolidan los datos en un único servidor central, para el que le gustaría utilizar Servidores de sucursales consolidan los datos en un único servidor concentrador, le gustaría utilizar Azure File Sync.

Para que Azure File Sync y DFS-R trabajen en paralelo:

1. Los niveles de nube de Azure File Sync deben deshabilitarse en volúmenes con carpetas replicadas DFS-R.
2. Los puntos de conexión de servidor no se deben configurar en carpetas de replicación de solo lectura DFS-R.

Para más información, consulte [Introducción a Espacios de nombres DFS y Replicación DFS](https://technet.microsoft.com/library/jj127250).

### <a name="sysprep"></a>Sysprep
No se admite el uso de sysprep en un servidor que tenga instalado el agente de Azure File Sync y puede provocar resultados inesperados. La instalación del agente y el registro del servidor se deben realizar después de implementar la imagen del servidor y completar la instalación mínima de sysprep.

### <a name="windows-search"></a>Windows Search
Si en un punto de conexión de un servidor están habilitados los niveles en la nube, Windows Search omite y no indexa los archivos que están en capas. Los archivos que no están en capas se indexan correctamente.

### <a name="antivirus-solutions"></a>Soluciones antivirus
Dado que un antivirus funciona examinando los archivos en busca de código malintencionado conocido, puede provocar la recuperación de archivos con niveles. En las versiones 4.0 y posteriores del agente de Azure File Sync, los archivos en niveles tienen establecido el atributo seguro de Windows FILE_ATTRIBUTE_RECALL_ON_DATA_ACCESS. Se recomienda consultar con el proveedor de software cómo configurar su solución para omitir la lectura de archivos que tengan establecido este atributo (muchas realizan la omisión automáticamente).

Las soluciones de antivirus internas de Microsoft, Windows Defender y System Center Endpoint Protection (SCEP), omiten de forma automática la lectura de archivos que tienen establecido dicho atributo. Hemos probado ambas soluciones e identificamos un problema menor: al agregar un servidor a un grupo de sincronización existente, se recuperan (descargan) los archivos de menos de 800 bytes en el nuevo servidor. Estos archivos permanecerán en el nuevo servidor y no se organizarán en niveles ya que no cumplen con el requisito de tamaño de niveles (>64 kb).

### <a name="backup-solutions"></a>Soluciones de copia de seguridad
Al igual que sucede con las soluciones antivirus, las soluciones de backup pueden provocar la recuperación de archivos con niveles. Se recomienda usar una solución de backup en la nube para realizar la copia de seguridad del recurso compartido de archivos de Azure en lugar de usar un producto de backup local.

Si está utilizando una solución de copia de seguridad local, las copias de seguridad deben realizarse en un servidor del grupo de sincronización que tenga los niveles en la nube deshabilitados. Al realizar una restauración, use las opciones de restauración de nivel de volumen o de archivo. Los archivos restaurados con la opción de restauración a nivel de archivo se sincronizarán con todos los puntos de conexión del grupo de sincronización y los archivos existentes se reemplazarán con la versión restaurada desde la copia de seguridad.  Las restauraciones a nivel de volumen no reemplazarán las versiones de archivo más recientes en el recurso compartido de archivos de Azure u otros puntos de conexión del servidor.

> [!Note]  
> La reconstrucción completa (BMR) puede causar resultados inesperados y actualmente no se admite.

> [!Note]  
> Las instantáneas de VSS (incluida la pestaña Versiones anteriores) no se admiten actualmente en los volúmenes que tienen habilitada la nube por niveles. Si la nube por niveles está habilitada, use las instantáneas de recurso compartido de Azure para restaurar un archivo desde una copia de seguridad.

### <a name="encryption-solutions"></a>Soluciones de cifrado
La compatibilidad con soluciones de cifrado depende de cómo se implementen. Está comprobado que Azure File Sync funciona con:

- Cifrado de BitLocker
- Azure Information Protection, Azure Rights Management Services (Azure RMS) y Active Directory RMS

está comprobado que Azure File Sync no funciona con:

- Sistema de cifrado de archivos NTFS (EFS)

En general, Azure File Sync debe admitir la interoperabilidad con soluciones de cifrado que se encuentran a continuación en el sistema de archivos, como BitLocker, y con las soluciones que se implementan en el formato de archivo, como Azure Information Protection. No se ha realizado ninguna interoperabilidad especial para las soluciones que se encuentran por encima del sistema de archivos (por ejemplo, EFS de NTFS).

### <a name="other-hierarchical-storage-management-hsm-solutions"></a>Otras soluciones de administración de almacenamiento jerárquico (HSM)
No deben utilizarse otras soluciones HSM deben utilizarse con Azure File Sync.

## <a name="region-availability"></a>Disponibilidad por región
Azure File Sync solo está disponible en las siguientes regiones:

| Área | Ubicación de centro de datos |
|--------|---------------------|
| Este de Australia | Nueva Gales del Sur |
| Sudeste de Australia | Victoria |
| Sur de Brasil | Sao Paolo State |
| Centro de Canadá | Toronto |
| Este de Canadá | Ciudad de Quebec |
| India Central | Pune |
| Centro de EE. UU. | Iowa |
| Asia oriental | Hong Kong RAE |
| Este de EE. UU | Virginia |
| Este de EE. UU. 2 | Virginia |
| Corea Central| Seúl |
| Corea del Sur| Busán |
| Este de Japón | Tokio, Saitama |
| Oeste de Japón | Osaka |
| Centro-Norte de EE. UU | Illinois |
| Europa del Norte | Irlanda |
| Centro-Sur de EE. UU | Texas |
| Sur de la India | Chennai |
| Sudeste asiático | Singapur |
| Sur de Reino Unido 2 | Londres |
| Oeste de Reino Unido | Cardiff |
| US Gov de Arizona (versión preliminar) | Arizona |
| US Gov de Texas (versión preliminar) | Texas |
| US Gov Virginia (versión preliminar) | Virginia |
| Europa occidental | Países bajos |
| Centro occidental de EE.UU. | Wyoming |
| Oeste de EE. UU. | California |
| Oeste de EE. UU. 2 | Washington |

Azure File Sync solo se admite la sincronización con recursos compartidos de archivos de Azure de la misma región que el servicio de sincronización de almacenamiento.

> [!Note]  
> Actualmente, Azure File Sync solo está disponible en versión preliminar privada para las regiones de gobierno. Consulte nuestra [notas de la versión](https://docs.microsoft.com/azure/storage/files/storage-files-release-notes#agent-version-5020) para obtener instrucciones sobre la inscripción en el programa de vista previa.

### <a name="azure-disaster-recovery"></a>Azure Disaster Recovery
Para protegerse contra la pérdida de una región de Azure, Azure File Sync se integra con la opción de [almacenamiento con redundancia geográfica](../common/storage-redundancy-grs.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json) (GRS). El almacenamiento GRS utiliza la replicación asincrónica de bloques entre el almacenamiento en la región primaria, con la que normalmente se interactúa, y el almacenamiento en la región secundaria emparejada. En caso de desastre que haga que una región de Azure se desconecte temporalmente o permanentemente, Microsoft conmutará por error el almacenamiento a la región emparejada. 

> [!Warning]  
> Si usa el recurso compartido de archivos de Azure como punto de conexión en la nube en una cuenta de almacenamiento GRS, no debe iniciar la conmutación por error de la cuenta de almacenamiento. Si lo hace, la sincronización dejará de funcionar y también podría provocar una pérdida inesperada de datos en el caso de archivos recién organizados en capas. En caso de pérdida de una región de Azure, Microsoft activará la conmutación por error de la cuenta de almacenamiento de forma que sea compatible con Azure File Sync.

Para admitir la integración de la conmutación por error entre el almacenamiento con redundancia geográfica y Azure File Sync, todas las regiones de Azure File Sync se emparejan con una región secundaria que coincide con la región secundaria que el almacenamiento utiliza. Se emparejan de la siguiente manera:

| Región principal      | Región emparejada      |
|---------------------|--------------------|
| Este de Australia      | Sudeste de Australia|
| Sudeste de Australia | Este de Australia     |
| Sur de Brasil        | Centro-Sur de EE. UU   |
| Centro de Canadá      | Este de Canadá        |
| Este de Canadá         | Centro de Canadá     |
| India Central       | Sur de la India        |
| Centro de EE. UU.          | Este de EE. UU. 2          |
| Asia oriental           | Sudeste asiático     |
| Este de EE. UU             | Oeste de EE. UU.            |
| Este de EE. UU. 2           | Centro de EE. UU.         |
| Este de Japón          | Oeste de Japón         |
| Oeste de Japón          | Este de Japón         |
| Corea Central       | Corea del Sur        |
| Corea del Sur         | Corea Central      |
| Europa del Norte        | Europa occidental        |
| Centro-Norte de EE. UU    | Centro-Sur de EE. UU   |
| Centro-Sur de EE. UU    | Centro-Norte de EE. UU   |
| Sur de la India         | India Central      |
| Sudeste asiático      | Asia oriental          |
| Sur de Reino Unido 2            | Oeste de Reino Unido            |
| Oeste de Reino Unido             | Sur de Reino Unido 2           |
| US Gov de Arizona      | US Gov de Texas       |
| US Gov Iowa         | US Gov Virginia    |
| US Gov Virgini      | US Gov de Texas       |
| Europa occidental         | Europa del Norte       |
| Centro occidental de EE.UU.     | Oeste de EE. UU. 2          |
| Oeste de EE. UU.             | Este de EE. UU            |
| Oeste de EE. UU. 2           | Centro occidental de EE.UU.    |

## <a name="azure-file-sync-agent-update-policy"></a>Directiva de actualización del agente de Azure File Sync
[!INCLUDE [storage-sync-files-agent-update-policy](../../../includes/storage-sync-files-agent-update-policy.md)]

## <a name="next-steps"></a>Pasos siguientes
* [Tenga en cuenta los valores de proxy y firewall](storage-sync-files-firewall-and-proxy.md)
* [Planeamiento de una implementación de Azure Files](storage-files-planning.md)
* [Implementación de Azure Files](storage-files-deployment-guide.md)
* [Implementación de Azure File Sync](storage-sync-files-deployment-guide.md)
* [Supervisión de Azure File Sync](storage-sync-files-monitoring.md)
