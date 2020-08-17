---
title: Directrices de rendimiento para SQL Server en Azure | Microsoft Docs
description: Ofrece directrices para optimizar el rendimiento de SQL Server en Microsoft Azure Virtual Machines.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
editor: ''
tags: azure-service-management
ms.assetid: a0c85092-2113-4982-b73a-4e80160bac36
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 10/18/2019
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 3ce829a9fd58fb2940ee3265a66717af3dc9c0b5
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87289067"
---
# <a name="performance-guidelines-for-sql-server-on-azure-virtual-machines"></a>Directrices de rendimiento para SQL Server en Azure Virtual Machines
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este tema se ofrece una guía para optimizar el rendimiento de SQL Server en Microsoft Azure Virtual Machines.

## <a name="overview"></a>Información general

 Mientras se ejecuta SQL Server en Azure Virtual Machines, se recomienda seguir usando las mismas opciones de ajuste de rendimiento de base de datos que son aplicables a SQL Server en los entornos de servidor local. Sin embargo, el rendimiento de una base de datos relacional en una nube pública depende de muchos factores como el tamaño de una máquina virtual y la configuración de los discos de datos.

En las [imágenes aprovisionadas de SQL Server en Azure Portal](sql-vm-create-portal-quickstart.md), siga los procedimientos recomendados para la configuración del almacenamiento general (para más información sobre cómo se configura el almacenamiento, consulte [Configuración del almacenamiento para máquinas virtuales de SQL Server](storage-configuration.md)). Después del aprovisionamiento, considere la posibilidad de aplicar otras de las optimizaciones que se describen en este artículo. Elija según la carga de trabajo y realice pruebas de verificación.

> [!TIP]
> Por lo general, existe un equilibrio entre la optimización de los costos y la optimización del rendimiento. Este artículo se centra en cómo obtener el *mejor* rendimiento de SQL Server en Azure Virtual Machines. Si su carga de trabajo no es menos exigente, podría no necesitar todas las optimizaciones enumeradas a continuación. Tenga en cuenta sus necesidades de rendimiento, costos y patrones de carga de trabajo a medida que evalúa estas recomendaciones.

## <a name="quick-checklist"></a>Lista de comprobación rápida

La siguiente es una lista de comprobación rápida para un rendimiento óptimo de SQL Server en Azure Virtual Machines:

| Área | Optimizaciones |
| --- | --- |
| [Tamaño de VM](#vm-size-guidance) | - Use tamaños de máquina virtual con cuatro o más vCPU, como [E4S_v3](../../../virtual-machines/ev3-esv3-series.md), o superior, o [DS12_v2](../../../virtual-machines/dv2-dsv2-series-memory.md), o superior.<br/><br/> Las - [series Es, Eas, Ds y Das](../../../virtual-machines/sizes-general.md) ofrecen una relación óptima entre memoria y vCPU, necesaria para el rendimiento de la carga de trabajo de OLTP. <br/><br/> La - [serie M](../../../virtual-machines/m-series.md) ofrece la mayor proporción de memoria a vCPU necesaria para el rendimiento crítico y es ideal para cargas de trabajo de almacenamiento de datos. <br/><br/> - Recopile los requisitos de [IOPS](../../../virtual-machines/windows/premium-storage-performance.md#iops) de la carga de trabajo de destino[, rendimiento](../../../virtual-machines/windows/premium-storage-performance.md#throughput) y [latencia](../../../virtual-machines/windows/premium-storage-performance.md#latency) en horas punta siguiendo [la lista de comprobación de los requisitos de rendimiento de la aplicación](../../../virtual-machines/windows/premium-storage-performance.md#application-performance-requirements-checklist) y, luego, seleccione el [tamaño de máquina virtual](../../../virtual-machines/sizes-general.md) que se puede escalar a los requisitos de rendimiento de la carga de trabajo.|
| [Storage](#storage-guidance) | - Para realizar pruebas detalladas del rendimiento de SQL Server en Azure Virtual Machines con los bancos de pruebas TPC-E y TPC_C, consulte el blog en el que se explica cómo [optimizar el rendimiento de OLTP](https://techcommunity.microsoft.com/t5/SQL-Server/Optimize-OLTP-Performance-with-SQL-Server-on-Azure-VM/ba-p/916794). <br/><br/> - Use [discos SSD Premium](https://techcommunity.microsoft.com/t5/SQL-Server/Optimize-OLTP-Performance-with-SQL-Server-on-Azure-VM/ba-p/916794) para obtener la mejor relación precio/rendimiento. Configure la [caché de solo lectura](../../../virtual-machines/windows/premium-storage-performance.md#disk-caching) para los archivos de datos y que el archivo de registro no se almacene en la caché. <br/><br/> - Use [Ultra Disks](../../../virtual-machines/windows/disks-types.md#ultra-disk) si la carga de trabajo requiere latencias de almacenamiento inferiores a 1 ms. Consulte cómo [realizar una migración a Disco Ultra](storage-migrate-to-ultradisk.md) para obtener más información. <br/><br/> - Recopile los requisitos de latencia de almacenamiento de los archivos de datos, de registro y de base de datos temporal de SQL Server, para lo que deberá [supervisar la aplicación](../../../virtual-machines/windows/premium-storage-performance.md#application-performance-requirements-checklist) antes de elegir el tipo de disco. Si se requieren latencias de almacenamiento inferiores a 1 ms, use Ultra Disks; de lo contrario, use discos SSD Premium. Si solo se necesitan latencias bajas para el archivo de registro, no para los archivos de datos, [aprovisione el Ultra Disk](../../../virtual-machines/windows/disks-enable-ultra-ssd.md) en los niveles de IOPS y rendimiento necesarios solo para el archivo de registro. <br/><br/> Los -  [recursos compartidos de archivos Premium](failover-cluster-instance-premium-file-share-manually-configure.md) se recomiendan como almacenamiento compartido para las instancias del clúster de conmutación por error de SQL Server. Los recursos compartidos de archivos Premium no admiten el almacenamiento en caché y ofrecen un rendimiento limitado, en comparación con los discos SSD Premium. Elija discos administrados con SSD Premium, en lugar de recursos compartidos de archivos Premium, para instancias de SQL independientes; pero aproveche los recursos compartidos de archivos Premium para el almacenamiento compartido de instancias del clúster de conmutación por error, ya que facilitan el mantenimiento y proporcionan una escalabilidad flexible. <br/><br/> - El almacenamiento estándar solo se recomienda con fines de desarrollo y realización de pruebas, o bien para archivos de copia de seguridad, y no debe usarse para cargas de trabajo de producción. <br/><br/> - Mantenga la [cuenta de almacenamiento](../../../storage/common/storage-create-storage-account.md) y la máquina virtual con SQL Server en la misma región.<br/><br/> - Deshabilite el [almacenamiento con redundancia geográfica](../../../storage/common/storage-redundancy.md) (replicación geográfica) de Azure en la cuenta de almacenamiento.  |
| [Discos](#disks-guidance) | - Use un mínimo de 2 [discos SSD Premium](../../../virtual-machines/windows/disks-types.md#premium-ssd) (uno para el archivo de registro y el otro para los archivos de datos). <br/><br/> - En cargas de trabajo que requieran latencias de E/S inferiores a 1 ms, habilite el acelerador de escritura, en el caso de la serie M, y considere la posibilidad de usar discos SSD Ultra, en el caso de las series Es y DS. <br/><br/> - Habilite el [almacenamiento en caché de solo lectura](../../../virtual-machines/windows/premium-storage-performance.md#disk-caching) en los discos que hospedan los archivos de datos.<br/><br/> - Agregue un 20 % más de capacidad de IOPS/rendimiento Premium de lo que requiere la carga de trabajo cuando [configure el almacenamiento para archivos de datos, de registro y de Temp DB de SQL Server](storage-configuration.md). <br/><br/> - Evite el uso del sistema operativo o de discos temporales para el registro o almacenamiento de bases de datos.<br/><br/> - No habilite el almacenamiento en caché en los discos que hospedan el archivo de registro.  **Importante**: Detenga el servicio de SQL Server al cambiar la configuración de caché para un disco de Azure Virtual Machines.<br/><br/> - Seccione varios discos de datos de Azure para obtener un mayor rendimiento de almacenamiento.<br/><br/> - Formatee con tamaños de asignación documentados. <br/><br/> - Coloque TempDB en el disco SSD local `D:\` para cargas de trabajo de SQL Server críticas (después de elegir el tamaño de máquina virtual correcto). Si crea la máquina virtual desde Azure Portal o con las plantillas de inicio rápido de Azure y [coloca Temp DB en el disco local](https://techcommunity.microsoft.com/t5/SQL-Server/Announcing-Performance-Optimized-Storage-Configuration-for-SQL/ba-p/891583), no es preciso que haga nada más; en los demás casos, siga los pasos descritos en el blog acerca del [Uso de unidades de estado sólido para almacenar bases de datos temporales](https://cloudblogs.microsoft.com/sqlserver/2014/09/25/using-ssds-in-azure-vms-to-store-sql-server-tempdb-and-buffer-pool-extensions/) para evitar que se produzcan errores después de reiniciarse. Si la capacidad de la unidad local no es suficiente para el tamaño de la base de datos temporal, coloque la base de datos temporal en un bloque de almacenamiento [seccionado](../../../virtual-machines/windows/premium-storage-performance.md) en discos SSD Premium con [almacenamiento en caché de solo lectura](../../../virtual-machines/windows/premium-storage-performance.md#disk-caching). |
| [E/S](#io-guidance) |- Habilite la compresión de páginas de bases de datos.<br/><br/> - Habilite la inicialización instantánea de archivos para archivos de datos.<br/><br/> - Limite el crecimiento automático de la base de datos.<br/><br/> - Deshabilite la reducción automática de la base de datos.<br/><br/> - Mueva todas las bases de datos a discos de datos, incluidas las bases de datos del sistema.<br/><br/> - Mueva los directorios de archivos de seguimiento y registros de errores de SQL Server a discos de datos.<br/><br/> - Configure ubicaciones predeterminadas para los archivos de base de datos y de copia de seguridad.<br/><br/> - [Habilite las páginas bloqueadas en la memoria](/sql/database-engine/configure-windows/enable-the-lock-pages-in-memory-option-windows?view=sql-server-2017).<br/><br/> - Aplique correcciones de rendimiento de SQL Server. |
| [Características específicas](#feature-specific-guidance) | - Realice copias de seguridad directamente en Azure Blob Storage.<br/><br/>- Use [copias de seguridad de instantáneas de archivos](/sql/relational-databases/backup-restore/file-snapshot-backups-for-database-files-in-azure) en caso de que las bases de datos sean de más de 12 TB. <br/><br/>- Use varios archivos Temp BD, un archivo por núcleo, hasta ocho archivos.<br/><br/>-Establezca la memoria máxima de servidor en un 90 % o deje un máximo de 50 GB para el sistema operativo. <br/><br/>- Habilite el acceso a la memoria no uniforme temporal. |

Para más información sobre *cómo* y *por qué* llevar a cabo estas optimizaciones, repase los detalles y las instrucciones proporcionadas en las siguientes secciones.

## <a name="vm-size-guidance"></a>Orientación sobre el tamaño de máquina virtual

Comience por recopilar los requisitos de rendimiento de CPU, memoria y almacenamiento de la carga de trabajo en los momentos de máxima actividad. Los contadores de rendimiento de \LogicalDisk\Disk Reads/Sec y \LogicalDisk\Disk Writes/Sec se pueden usar para recopilar los requisitos de IOPS de lectura y escritura, mientras que el contador de \LogicalDisk\Disk Bytes/Sec se puede usar para recopilar los [requisitos de rendimiento del almacenamiento](../../../virtual-machines/windows/premium-storage-performance.md#disk-caching) de los archivos de datos, de registro y de base de datos temporal. Después de definir IOPS y los requisitos de rendimiento en los momentos de máxima actividad, evalúe si los tamaños de las máquinas virtuales ofrecen esa capacidad. Por ejemplo, si la carga de trabajo requiere 20 000 IOPS de lectura y 10 000 IOPS de escritura en los momentos de máxima actividad, puede elegir E16s_v3 (con hasta 32 000 IOPS en caché y 25 600 no almacenadas en la caché), o bien M16_s (con un máximo de 20 000 IOPS en caché y 10 000 no almacenadas en la caché) con 2 discos P30. Asegúrese de que conoce los requisitos tanto de rendimiento como de IOPS de la carga de trabajo, ya que las máquinas virtuales tienen diferentes límites de escala para ambos.<br/><br/>Las series [DSv_3](../../../virtual-machines/dv3-dsv3-series.md) y [Es_v3](../../../virtual-machines/ev3-esv3-series.md) se hospedan en hardware de uso general con procesadores Intel Haswell o Broadwell. La [serie M](../../../virtual-machines/m-series.md) ofrece el mayor número de vCPU y la máxima memoria para las cargas de trabajo de SQL Server más grandes y se hospeda en hardware con memoria optimizada con la familia de procesadores Skylake. Estas series admiten el almacenamiento Premium, lo que se recomienda para lograr el mejor rendimiento posible con la caché de lectura de nivel de host. Las series Es_v3 y M también están disponibles con [menores tamaños de núcleo](../../../virtual-machines/windows/constrained-vcpu.md), lo que ahorra dinero en caso de cargas de trabajo con menores exigencias de proceso y alta capacidad de almacenamiento. 

## <a name="storage-guidance"></a>Orientación sobre el almacenamiento

Para realizar pruebas detalladas del rendimiento de SQL Server en Azure Virtual Machines con los bancos de pruebas TPC-E y TPC_C, consulte el blog en el que se explica cómo [optimizar el rendimiento de OLTP](https://techcommunity.microsoft.com/t5/SQL-Server/Optimize-OLTP-Performance-with-SQL-Server-on-Azure-VM/ba-p/916794). 

Para las cargas de trabajo de producción se recomienda la caché de blobs de Azure con discos SSD Premium. 

> [!WARNING]
> Los HDD estándar y las unidades de estado sólido (SSD) presentan ancho de banda y latencias variables, de modo que solo se recomiendan para cargas de trabajo de desarrollo o de prueba. En las cargas de trabajo de producción conviene usar SSD prémium.

Además, se recomienda crear su cuenta de almacenamiento de Azure en el mismo centro de datos que sus máquinas virtuales de SQL Server para reducir los retrasos en la transferencia. Al crear una cuenta de almacenamiento, deshabilite la replicación geográfica ya que no se garantiza el orden de escritura coherente entre varios discos. En su lugar, considere la posibilidad de configurar una tecnología de recuperación ante desastres de SQL Server entre dos centros de datos de Azure. Para más información, consulte [Alta disponibilidad y recuperación ante desastres para SQL Server en Azure Virtual Machines](business-continuity-high-availability-disaster-recovery-hadr-overview.md).

## <a name="disks-guidance"></a>Orientación sobre los discos

En las máquinas virtuales de Azure hay tres tipos de disco principales:

* **Disco del sistema operativo**: al crear una máquina virtual de Azure, la plataforma conectará al menos un disco (etiquetado como unidad **C**) a la máquina virtual como disco del sistema operativo. El disco es un VHD almacenado como blob en páginas en almacenamiento.
* **Disco temporal**: las máquinas virtuales de Azure contienen otro disco denominado disco temporal (etiquetado como unidad **D**:). Se trata de un disco en el nodo que se puede usar para el espacio de desecho.
* **Discos de datos**: También puede conectar discos adicionales a la máquina virtual como discos de datos y estos se almacenarán en el almacenamiento como blobs en páginas.

En las siguientes secciones se incluyen recomendaciones sobre cómo usar cada uno de estos discos.

### <a name="operating-system-disk"></a>Disco del sistema operativo

Un disco del sistema operativo es un VHD que se puede arrancar y montar como una versión en ejecución de un sistema operativo y está etiquetado como la unidad **C**.

La directiva del almacenamiento en caché predeterminada en el disco del sistema operativo es **Lectura/escritura**. Para aplicaciones sensibles al rendimiento, se recomienda usar discos de datos en lugar del disco del sistema operativo. Vea la sección sobre Discos de datos a continuación.

### <a name="temporary-disk"></a>Disco temporal

La unidad de almacenamiento temporal, etiquetada como unidad **D**, no se conserva en Azure Blob Storage. No almacene archivos de base de datos de usuarios ni archivos de registro de transacciones de usuarios en la unidad **D:** .

Coloque TempDB en la unidad de estado sólido local `D:\` para cargas de trabajo de SQL Server críticas (después de elegir el tamaño de máquina virtual correcto). Si crea la máquina virtual desde Azure Portal o con las plantillas de inicio rápido de Azure y [coloca Temp DB en el disco local](https://techcommunity.microsoft.com/t5/SQL-Server/Announcing-Performance-Optimized-Storage-Configuration-for-SQL/ba-p/891583), no es preciso que haga nada más; en los restantes casos, siga los pasos descritos en el blog acerca del [Uso de unidades de estado sólido para almacenar bases de datos temporales](https://cloudblogs.microsoft.com/sqlserver/2014/09/25/using-ssds-in-azure-vms-to-store-sql-server-tempdb-and-buffer-pool-extensions/) para evitar que se produzcan errores después de reiniciarse. Si la capacidad de la unidad local no es suficiente para el tamaño de la base de datos temporal, coloque la base de datos temporal en un bloque de almacenamiento [seccionado](../../../virtual-machines/windows/premium-storage-performance.md) en discos SSD Premium con [almacenamiento en caché de solo lectura](../../../virtual-machines/windows/premium-storage-performance.md#disk-caching).

En las máquinas virtuales que admiten discos SSD Premium, las bases de datos temporales también se puede almacenar en un disco que admita discos SSD Premium con el almacenamiento en caché de lectura habilitado.


### <a name="data-disks"></a>Discos de datos.

* **Uso de discos SSD Premium para archivos de datos y de registro**: si no usa el seccionamiento de discos, utilice dos discos SSD Premium, uno para el archivo de registro y otro para los datos. Cada disco SSD Premium proporciona un número de IOPS y ancho de banda (MB/s) según su tamaño, como se describe en el artículo en el que se indica cómo se debe [seleccionar el tipo de disco](../../../virtual-machines/windows/disks-types.md). Si usa una técnica de fragmentación de discos, como los espacios de almacenamiento, puede lograr un rendimiento óptimo si tiene dos grupos: uno para los archivos de registro y otro para los archivos de datos. Sin embargo, si planea usar instancias de clúster de conmutación por error (FCI) de SQL Server (FCI), debe configurar un grupo o utilizar [recursos compartidos de archivos Premium](failover-cluster-instance-premium-file-share-manually-configure.md).

   > [!TIP]
   > - Para obtener resultados de la pruebas con varias configuraciones de discos y cargas de trabajo, consulte la siguiente entrada de blog: [Instrucciones de configuración del almacenamiento para SQL Server en Azure Virtual Machines](https://blogs.msdn.microsoft.com/sqlserverstorageengine/2018/09/25/storage-configuration-guidelines-for-sql-server-on-azure-vm/).
   > - Para obtener un rendimiento crítico en servidores con SQL Server que necesitan aproximadamente 50.000 E/S por segundo, considere la posibilidad de reemplazar 10 - discos P30 por uno SSD Ultra. Para más información, consulte la siguiente entrada de blog: [Mission critical performance with Ultra SSD](https://azure.microsoft.com/blog/mission-critical-performance-with-ultra-ssd-for-sql-server-on-azure-vm/) (Rendimiento crítico con SSD Ultra).

   > [!NOTE]
   > Al aprovisionar una máquina virtual de SQL Server en el portal, tiene la opción de modificar la configuración de almacenamiento. Según la configuración, Azure configura uno o varios discos. Varios discos se combinan en un único grupo de almacenamiento con seccionamiento. Los archivos de datos y de registro residen juntos en esta configuración. Para más información, consulte [Configuración del almacenamiento para máquinas virtuales de SQL Server](storage-configuration.md).

* **Seccionamiento de discos**: Para disfrutar de un mayor rendimiento, puede agregar más discos de datos y usar el seccionamiento de discos. Para determinar el número de discos de datos, debe analizar el número de IOPS y ancho de banda necesario para los archivos de registro, así como para los datos y los archivos TempDB. Observe que diferentes tamaños de máquinas virtuales tienen distintos límites en el número de IOPS y ancho de banda admitidos. Vea las tablas de IOPS por [tamaño de máquina virtual](../../../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). Use estas directrices:

  * Para Windows 8/Windows Server 2012 o posterior, use [espacios de almacenamiento](https://technet.microsoft.com/library/hh831739.aspx) con las siguientes directrices:

      1. Configure la intercalación (tamaño de sección) en 64 KB (65 536 bytes) en el caso de las cargas de trabajo de OLTP y 256 KB (262 144 bytes) en el caso de las cargas de trabajo de almacenamiento de datos para evitar que el rendimiento se vea afectado debido a una mala alineación de las particiones. Esta característica debe establecerse con PowerShell.
      2. Establezca recuento de columnas = número de discos físicos. Use PowerShell (no la interfaz de usuario del Administrador del servidor) al configurar más de 8 discos. 

    Por ejemplo, aquí PowerShell crea un nuevo grupo de almacenamiento con el tamaño de intercalación de 64 KB y un número de columnas igual a la cantidad de disco físico en el bloque de almacenamiento:

    ```powershell
    $PhysicalDisks = Get-PhysicalDisk | Where-Object {$_.FriendlyName -like "*2" -or $_.FriendlyName -like "*3"}
    
    New-StoragePool -FriendlyName "DataFiles" -StorageSubsystemFriendlyName "Storage Spaces*" `
        -PhysicalDisks $PhysicalDisks | New- VirtualDisk -FriendlyName "DataFiles" `
        -Interleave 65536 -NumberOfColumns $PhysicalDisks .Count -ResiliencySettingName simple `
        –UseMaximumSize |Initialize-Disk -PartitionStyle GPT -PassThru |New-Partition -AssignDriveLetter `
        -UseMaximumSize |Format-Volume -FileSystem NTFS -NewFileSystemLabel "DataDisks" `
        -AllocationUnitSize 65536 -Confirm:$false 
    ```

  * Para Windows 2008 R2 o versiones anteriores, puede usar discos dinámicos (volúmenes seccionados del SO) y el tamaño de la franja siempre es 64 KB. Esta opción está en desuso a partir de Windows 8 y Windows Server 2012. Para obtener información, vea la declaración de soporte técnico en [Servicio de disco virtual está realizando la transición a la API de administración de almacenamiento de Windows](https://msdn.microsoft.com/library/windows/desktop/hh848071.aspx).

  * Si usa [Espacios de almacenamiento directo (S2D)](/windows-server/storage/storage-spaces/storage-spaces-direct-in-vm) con [instancias del clúster de conmutación por error de SQL Server](failover-cluster-instance-storage-spaces-direct-manually-configure.md), debe configurar un solo grupo. Aunque se pueden crear diferentes volúmenes en ese único grupo, todos ellos compartirán las mismas características, como por ejemplo, la misma directiva de almacenamiento en caché.

  * Determine el número de discos asociados al grupo de almacenamiento en función de sus expectativas de carga. Tenga en cuenta que diferentes tamaños de máquina virtual permiten diferentes números de discos de datos conectados. Para más información, consulte [Tamaños de las máquinas virtuales Linux en Azure](../../../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

  * Si no usa SSD Premium (escenarios de desarrollo y pruebas), se recomienda agregar el número máximo de discos de datos admitidos por el [tamaño de la máquina virtual](../../../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) y usar el seccionamiento de discos.

* **Directiva de caché**: tenga en cuenta las siguientes recomendaciones para la directiva según la configuración del almacenamiento en caché.

  * Si usa discos independientes para los archivos de datos y de registro, habilite el almacenamiento en caché de lectura en los discos de datos que hospeden los archivos de datos y de datos de TempDB. Esto puede suponer una mejora del rendimiento significativa. No habilite el almacenamiento en caché en el disco que contiene el archivo de registro, ya que esto produce una pequeña disminución del rendimiento.

  * Si usa la fragmentación de discos en un solo grupo de almacenamiento, la mayoría de las cargas de trabajo se beneficiarán de la memoria caché de lectura. Si tiene grupos de almacenamiento independientes para los archivos de registro y de datos, habilite el almacenamiento en caché de lectura solo en el grupo de almacenamiento de los archivos de datos. Para ciertas cargas de trabajo de escritura intensivas, podría lograr un mejor rendimiento sin almacenamiento en caché. Esto solo se puede determinar mediante pruebas.

  * Las recomendaciones anteriores se aplican a los SSD prémium. Si no usa SSD prémium, no habilite el almacenamiento en caché en los discos de datos.

  * Para obtener instrucciones sobre la configuración del almacenamiento en caché de disco, consulte los siguientes artículos. Para el modelo de implementación clásico (ASM), consulte: [Set-AzureOSDisk](https://msdn.microsoft.com/library/azure/jj152847) y [Set-AzureDataDisk](https://msdn.microsoft.com/library/azure/jj152851.aspx). Para el modelo de implementación mediante Azure Resource Manager, consulte: [Set-AzOSDisk](https://docs.microsoft.com/powershell/module/az.compute/set-azvmosdisk) y [Set-AzVMDataDisk](https://docs.microsoft.com/powershell/module/az.compute/set-azvmdatadisk).

     > [!WARNING]
     > Detenga el servicio SQL Server al cambiar la configuración de la caché de los discos de Azure Virtual Machines para evitar la posibilidad de que se produzcan daños en la base de datos.

* **Tamaño de la unidad de asignación de NTFS**: Al formatear el disco de datos, se recomienda usar un tamaño de unidad de asignación de 64 KB para los archivos de datos y de registro, además de TempDB. Si TempDB se incluye en el disco temporal (unidad D:\), el rendimiento obtenido aprovechando esta unidad supera la necesidad de un tamaño de la unidad de asignación de 64 K. 

* **Procedimientos recomendados de administración de discos**: al quitar un disco de datos o cambiar su tipo de caché, detenga el servicio SQL Server durante el cambio. Cuando se modifica la configuración de almacenamiento en caché en el disco del sistema operativo, Azure detiene la máquina virtual, cambia el tipo de caché y reinicia la máquina virtual. Cuando se modifica la configuración de la caché de un disco de datos, no se detiene la máquina virtual, pero el disco de datos se desconecta de la máquina virtual durante el cambio y, posteriormente, se vuelve a conectar.

  > [!WARNING]
  > De no detenerse el servicio SQL Server durante estas operaciones, podrían producirse daños en la base de datos.


## <a name="io-guidance"></a>Orientación sobre E/S

* Los mejores resultados con las unidades de estado sólido prémium se logran al establecer paralelismos entre la aplicación y las solicitudes. Las SSD prémium están diseñadas para escenarios donde la profundidad de la cola de E/S es mayor que 1, por lo que verá poco o ningún aumento del rendimiento para las solicitudes de serie de subproceso único (incluso si son intensivas de almacenamiento). Por ejemplo, esto podría afectar a los resultados de pruebas de subproceso único de herramientas de análisis de rendimiento, como SQLIO.

* Recuerde que también puede usar la [compresión de páginas de bases de datos](https://msdn.microsoft.com/library/cc280449.aspx) , ya que puede ayudarle a mejorar el rendimiento de las cargas de trabajo intensivas de E/S. Sin embargo, la compresión de datos puede aumentar el consumo de la CPU en el servidor de bases de datos.

* Considere la posibilidad de habilitar la inicialización instantánea de archivos para reducir el tiempo necesario para la asignación inicial de archivos. Para aprovechar la inicialización instantánea de archivos, se concede la cuenta de servicio de SQL Server (MSSQLSERVER) con SE_MANAGE_VOLUME_NAME y se agrega a la directiva de seguridad **Realizar tareas de mantenimiento del volumen**. Si usa una imagen de la plataforma de SQL Server para Azure, la cuenta de servicio predeterminada (NT Service\MSSQLSERVER) no se agrega a la directiva de seguridad **Realizar tareas de mantenimiento del volumen**. En otras palabras, la inicialización instantánea de archivos no se habilita en una imagen de plataforma de SQL Server Azure. Después de agregar la cuenta de servicio de SQL Server a la directiva de seguridad **Realizar tareas de mantenimiento del volumen** , reinicie el servicio de SQL Server. Puede haber consideraciones de seguridad para usar esta característica. Para obtener más información, vea [Inicialización de archivos de base de datos](https://msdn.microsoft.com/library/ms175935.aspx).

* Tenga en cuenta que el **crecimiento automático** apenas se considera una contingencia para el crecimiento inesperado. No administre su crecimiento de datos y registros a diario con el crecimiento automático. Si se usa el crecimiento automático, haga crecer previamente el archivo mediante el modificador Tamaño.

* Asegúrese de que la **reducción automática** está deshabilitada para evitar una sobrecarga innecesaria que puede afectar negativamente al rendimiento.

* Mueva todas las bases de datos a discos de datos, incluidas bases de datos del sistema. Para obtener más información, vea [Mover bases de datos del sistema](https://msdn.microsoft.com/library/ms345408.aspx).

* Mueva los directorios de archivos de seguimiento y registros de errores de SQL Server a discos de datos. Para llevar esto a cabo en el Administrador de configuración de SQL Server, haga clic con el botón secundario en la instancia de SQL Server y seleccione Propiedades. La configuración de los archivos de registro y de seguimiento de errores se puede cambiar en la pestaña **Parámetros de inicio** . El directorio de volcado se especifica en la pestaña **Opciones avanzadas** . En la siguiente captura de pantalla se muestra dónde buscar el parámetro de inicio del registro de errores.

    ![Captura de pantalla de registro de errores de SQL](./media/performance-guidelines-best-practices/sql_server_error_log_location.png)

* Configure ubicaciones predeterminadas para los archivos de base de datos y de copia de seguridad. Use las recomendaciones de este artículo y realice los cambios necesarios en la ventana Propiedades del servidor. Para obtener instrucciones, consulte [Ver o cambiar las ubicaciones predeterminadas de los archivos de datos y registro (SQL Server Management Studio)](https://msdn.microsoft.com/library/dd206993.aspx). En la siguiente captura de pantalla se muestra dónde realizar estos cambios.

    ![Archivos de copia de seguridad y de registro de datos de SQL](./media/performance-guidelines-best-practices/sql_server_default_data_log_backup_locations.png)
* Habilite páginas bloqueadas para reducir la E/S y las actividades de paginación. Para más información, consulte [Habilitar la opción Bloquear páginas en la memoria (Windows)](https://msdn.microsoft.com/library/ms190730.aspx).

* Si está ejecutando SQL Server 2012, instale la actualización acumulativa 10 del Service Pack 1. Esta actualización contiene la corrección para el rendimiento deficiente en la E/S al ejecutar select en la instrucción de tabla temporal en SQL Server 2012. Para obtener información, consulte este [artículo de Knowledge Base](https://support.microsoft.com/kb/2958012).

* Considere la posibilidad de comprimir los archivos de datos cuando se transfieren dentro y fuera de Azure.

## <a name="feature-specific-guidance"></a>Guía específica de la característica

Algunas implementaciones pueden lograr ventajas de rendimiento adicionales mediante técnicas de configuración más avanzadas. En la siguiente lista se destacan algunas características de SQL Server que pueden ayudarle a lograr un mejor rendimiento:

### <a name="back-up-to-azure-storage"></a>Copia de seguridad en Azure Storage
Al realizar copias de seguridad para SQL Server que se ejecutan en Azure Virtual Machines, puede usar [Copia de seguridad en URL de SQL Server](https://msdn.microsoft.com/library/dn435916.aspx). Esta característica está disponible a partir de SQL Server 2012 SP1 CU2 y se recomienda para las copias de seguridad en los discos de datos conectados. Al realizar copias de seguridad o restaurar en o desde Azure Storage, siga las recomendaciones que se ofrecen en [Solución de problemas y prácticas recomendadas de copia de seguridad de SQL Server en URL y restauración a partir de copias de seguridad almacenadas en Azure Storage](https://msdn.microsoft.com/library/jj919149.aspx). También puede automatizar estas copias de seguridad mediante la [Copia de seguridad automatizada para SQL Server en Azure Virtual Machines](../../../azure-sql/virtual-machines/windows/automated-backup-sql-2014.md).

Antes de SQL Server 2012, puede usar la [Herramienta de copia de seguridad de SQL Server a Azure](https://www.microsoft.com/download/details.aspx?id=40740). Esta herramienta puede ayudar a aumentar el rendimiento de la copia de seguridad con varios destinos de franjas de copia de seguridad.

### <a name="sql-server-data-files-in-azure"></a>Archivos de datos de SQL Server en Azure

esta nueva característica ([Archivos de datos de SQL Server en Azure](https://msdn.microsoft.com/library/dn385720.aspx)) está disponible a partir de SQL Server 2014. La ejecución de SQL Server con archivos de datos en Azure muestra características de rendimiento comparables con el uso de discos de datos de Azure.

### <a name="failover-cluster-instance-and-storage-spaces"></a>Instancia de clúster de conmutación por error y Espacios de almacenamiento

Si usa Espacios de almacenamiento, al agregar nodos al clúster en la página **Confirmación**, desactive la casilla **Agregar todo el almacenamiento apto al clúster**. 

![Desactivación del almacenamiento apto](./media/performance-guidelines-best-practices/uncheck-eligible-cluster-storage.png)

Si utiliza espacios de almacenamiento y no desactiva la casilla **Add all eligible storage to the cluster** (Agregar todo el almacenamiento apto al clúster), Windows separa los discos virtuales durante el proceso de agrupación en clústeres. Como resultado, no aparecen en el Administrador de discos ni el Explorador hasta que se quiten los espacios de almacenamiento del clúster y se vuelvan a asociar mediante PowerShell. Espacios de almacenamiento agrupa varios discos en grupos de almacenamiento. Para obtener más información, consulte el artículo sobre [espacios de almacenamiento](/windows-server/storage/storage-spaces/overview).

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el almacenamiento y el rendimiento, consulte [Instrucciones de configuración del almacenamiento para SQL Server en Azure Virtual Machines](https://blogs.msdn.microsoft.com/sqlserverstorageengine/2018/09/25/storage-configuration-guidelines-for-sql-server-on-azure-vm/).

Para ver las prácticas recomendadas de seguridad, consulte [Consideraciones de seguridad para SQL Server en Azure Virtual Machines](security-considerations-best-practices.md).

Revise otros artículos sobre la máquina virtual de SQL Server en [Introducción a SQL Server en Azure Virtual Machines](sql-server-on-azure-vm-iaas-what-is-overview.md). Si tiene alguna pregunta sobre las máquinas virtuales de SQL Server, consulte las [Preguntas más frecuentes](frequently-asked-questions-faq.md).
