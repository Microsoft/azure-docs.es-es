---
title: Azure Backup de SAP HANA en el nivel de archivo | Microsoft Docs
description: Existen dos posibilidades de copia de seguridad principales para SAP HANA en Azure Virtual Machines. Este artículo trata sobre Azure Backup de SAP HANA en el nivel de archivo
services: virtual-machines-linux
documentationcenter: ''
author: hermanndms
manager: juergent
editor: ''
ms.service: virtual-machines-linux
ms.topic: article
ums.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 03/01/2020
ms.author: juergent
ms.openlocfilehash: 183273e6f93bbfda8ed4e5fe913192994a0b6ce2
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87833392"
---
# <a name="sap-hana-azure-backup-on-file-level"></a>Azure Backup de SAP HANA en el nivel de archivo

## <a name="introduction"></a>Introducción

Este es un artículo relacionado con [Guía de copia de seguridad de SAP HANA en Azure Virtual Machines](./sap-hana-backup-guide.md), en el que se proporciona información general inicial y más detalles sobre el servicio Azure Backup y las instantáneas de almacenamiento. 

Otros tipos de máquinas virtuales de Azure permiten otro número de discos duros virtuales conectados. Los detalles exactos se documentan en [Tamaños de las máquinas virtuales Linux en Azure](../../sizes.md). Para las pruebas que se mencionan en esta documentación se usó una máquina virtual de GS5 de Azure, que permite conectar 64 discos de datos. Para los sistemas SAP HANA de gran tamaño, es posible que ya se haya tomado un número significativo de discos para los archivos de registro y datos, posiblemente junto con la fragmentación de software para ofrecer un rendimiento óptimo de E/S del disco. Para más información sobre las configuraciones de disco sugeridas para las implementaciones de SAP HANA en máquinas virtuales de Azure, lea el artículo [Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA](./hana-vm-operations-storage.md). Entre las recomendaciones realizadas también se incluyen recomendaciones de espacio en disco para las copias de seguridad locales.

Es el método estándar de administración de la copia de seguridad/restauración en el nivel de archivo es con una copia de seguridad basada en archivos a través de SAP HANA Studio o a través de instrucciones SQL de SAP HANA. Para más información, lea el artículo acerca de [SAP HANA SQL y referencia de vistas del sistema](https://help.sap.com/hana/SAP_HANA_SQL_and_System_Views_Reference_en.pdf).

![Esta ilustración muestra el cuadro de diálogo del elemento de menú de copia de seguridad en SAP HANA Studio](media/sap-hana-backup-file-level/backup-menue-dialog.png)

Esta ilustración muestra el cuadro de diálogo del elemento de menú de copia de seguridad en SAP HANA Studio. Al elegir el tipo &quot;archivo&quot;, tiene que especificar una ruta de acceso den el sistema de archivos donde SAP HANA escribe los archivos de copia de seguridad. La restauración funciona del mismo modo.

Aunque esta opción suena simple y sencilla, existen algunas consideraciones. Las máquinas virtuales de Azure tienen una limitación al respecto del número de discos de datos que se pueden conectar. Es posible que no haya capacidad para almacenar archivos de copia de seguridad de SAP HANA en los sistemas de archivos de la máquina virtual, en función del tamaño de los requisitos de rendimiento del disco y la base de datos, que podría implicar la fragmentación de software en varios discos de datos. Se proporcionan más opciones para mover estos archivos de copia de seguridad y administrar las restricciones de tamaño de archivo y el rendimiento cuando se administran terabytes de datos más adelante en este artículo.

Otra opción, que ofrece más libertad con respecto a la capacidad total, es Azure Blob Storage. Aunque un único blob también se restringe a 1 TB, la capacidad total de un único contenedor de blobs es actualmente de 500 TB. Además, ofrece a los clientes la opción de seleccionar el denominado almacenamiento de blobs en &quot;frío&quot;, que tiene un costo-beneficio. Para más información sobre el almacenamiento esporádico de blobs, consulte [Azure Blob Storage: niveles de acceso frecuente, esporádico y de archivo](../../../storage/blobs/storage-blob-storage-tiers.md?tabs=azure-portal).

Para obtener una seguridad adicional, utilice una cuenta de almacenamiento con replicación geográfica para almacenar las copias de seguridad de SAP HANA. Para más información acerca de la redundancia y la replicación de almacenamiento, consulte [Redundancia de Azure Storage](../../../storage/common/storage-redundancy.md).

Puede colocar VHD dedicados para copias de seguridad de SAP HANA en una cuenta de almacenamiento de copia de seguridad dedicada con replicación geográfica. También puede copiar los VHD que conserven las copias de seguridad de SAP HANA en una cuenta de almacenamiento de replicación geográfica o en una cuenta de almacenamiento que se encuentre en una región diferente.

## <a name="azure-blobxfer-utility-details"></a>Detalles de la utilidad bloxfer de Azure

Para almacenar directorios y archivos en Azure Storage, puede usar CLI o PowerShell, o desarrollar una herramienta mediante uno de los [Azure SDK](https://azure.microsoft.com/downloads/). También hay una utilidad lista para usar, AzCopy, para copiar datos en Azure Storage. (consulte [Transferencia de datos con la utilidad en línea de comandos AzCopy](../../../storage/common/storage-use-azcopy-v10.md)).

Por tanto, se usó blobxfer para copiar los archivos de copia de seguridad de SAP HANA. Se trata de código abierto, que usan muchos clientes en entornos de producción, y que está disponible en [GitHub](https://github.com/Azure/blobxfer). Esta herramienta permite copiar datos directamente a Azure Blob Storage o al recurso compartido de archivos de Azure. También ofrece una gama de características útiles, como hash md5 o el paralelismo automático al copiar un directorio con varios archivos.

## <a name="sap-hana-backup-performance"></a>Rendimiento de copia de seguridad de SAP HANA
En este capítulo, se tratan las consideraciones acerca del rendimiento. Es posible que los números que aparezcan no representen el estado más reciente, ya que hay un desarrollo estable para lograr un mejor rendimiento del almacenamiento de Azure. En consecuencia, debe realizar pruebas individuales de la configuración y la región de Azure.

![Esta captura de pantalla es de la consola de copia de seguridad de SAP HANA en SAP HANA Studio](media/sap-hana-backup-file-level/backup-console-hana-studio.png)

En esta captura de pantalla se muestra la consola de copias de seguridad de SAP HANA de SAP HANA Studio. Se tardó 42 minutos en realizar la copia de seguridad de 230 GB en un único disco de almacenamiento HDD estándar de Azure conectado a la máquina virtual de HANA con el sistema de archivos XFS en el único disco.

![Esta captura de pantalla es de YaST en la VM de prueba de SAP HANA](media/sap-hana-backup-file-level/one-backup-disk.png)

Esta captura de pantalla es de YaST en la VM de prueba de SAP HANA. Puede ver el disco único de 1 TB para la copia de seguridad de SAP HANA. Se tardó 42 minutos en realizar la copia de seguridad de 230 GB. Además, se asocian cinco discos de 200 GB y software RAID md0 creado, con la fragmentación en la parte superior de estaos cinco discos de datos de Azure.

![La repetición de la misma copia de seguridad en software RAID con fragmentación en cinco discos de datos de Azure Standard Storage asociados](media/sap-hana-backup-file-level/five-backup-disks.png)

La repetición de la misma copia de seguridad en software RAID con fragmentación en cinco discos de datos de Azure Standard Storage asociados provocó que el tiempo de copia de seguridad descendiera de 42 a 10 minutos. Los discos se asociaron sin el almacenamiento en caché en la VM. En este ejercicio se muestra la importancia del rendimiento de la escritura en disco para lograr un buen tiempo de copia de seguridad. Se podría cambiar al almacenamiento SSD estándar de Azure o a Azure Premium Storage para acelerar aún más el proceso, con el fin de obtener un rendimiento óptimo. En general, no se recomienda el almacenamiento HDD estándar de Azure y se ha usado solo con fines de demostración. Se recomienda usar un mínimo de almacenamiento SSD estándar de Azure o Azure Premium Storage para los sistemas de producción.

## <a name="copy-sap-hana-backup-files-to-azure-blob-storage"></a>Copia de archivos de copia de seguridad de SAP HA en Azure Blob Storage
Las cifras de rendimiento, duración de la copia de seguridad y duración de la copia que aparecen podrían no representar el estado más reciente de la tecnología de Azure. Microsoft está mejorando constantemente el almacenamiento de Azure para ofrecer mejor rendimiento y latencias más bajas. Por lo tanto, los números de usan solo con fines de demostración. Para poder juzgar cual de los métodos es el mejor para usted, es preciso que pruebe lo que necesita en la región de Azure que prefiera.

Otra opción para almacenar rápidamente los archivos de copia de seguridad de SAP HANA es Azure Blob Storage. Los contenedores de blobs individuales tienen un límite aproximado de 500 TB, que es suficiente para que los sistemas de SAP HANA en los que se usan los tipos de máquina virtual M32ts, M32ls, M64ls y GS5 de Azure mantengan copias de seguridad de SAP HANA suficientes. Los clientes pueden elegir entre el almacenamiento de blobs de &quot;acceso frecuenta&quot; y &quot;esporádico&quot; (consulte[Azure Blob Storage: niveles de acceso frecuente, esporádico y de archivo](../../../storage/blobs/storage-blob-storage-tiers.md?tabs=azure-portal)).

Con la herramienta blobxfer, es fácil copiar los archivos de copia de seguridad de SAP HANA directamente en Azure Blob Storage.

![Aquí puede ver los archivos de una copia de seguridad completa de archivos de SAP HANA](media/sap-hana-backup-file-level/list-of-backups.png)

Puede ver los archivos de una copia de seguridad completa de archivos de SAP HANA. De los cuatro archivos, el mayor tiene un tamaño aproximado de 230 GB.

![Se tardó aproximadamente 3000 segundos en copiar 230 GB en un contenedor de blobs de la cuenta de Azure Standard Storage](media/sap-hana-backup-file-level/copy-duration-blobxfer.png)

Sin usar el hash md5 en la prueba inicial, se tardó aproximadamente 3000 segundos en copiar 230 GB en un contenedor de blobs de la cuenta de Azure Standard Storage.

La consola de copia de seguridad de HANA Studio permite restringir el tamaño máximo de archivo de los archivos de copia de seguridad de HANA. En el entorno de ejemplo, mejoró el rendimiento teniendo varios archivos de copia de seguridad más pequeños, en lugar de un archivo grande de 230 GB.

El establecimiento del límite del tamaño del archivo de copia de seguridad en la parte de HANA no mejora el tiempo de copia de seguridad porque los archivos se escriben secuencialmente. El límite de tamaño de archivo se estableció en 60 GB, por lo que la copia de seguridad creó cuatro archivos de datos de gran tamaño en lugar del archivo único de 230 GB. El uso de varios archivos de copia de seguridad puede ser necesario para realizar copias de seguridad de bases de datos de HANA si los destinos de copia de seguridad tienen limitaciones en los tamaños de archivo de los tamaños de blob.

![Para probar el paralelismo de la herramienta blobxfer, el tamaño máximo de archivo para las copias de seguridad HANA se estableció en 15 GB](media/sap-hana-backup-file-level/parallel-copy-multiple-backup-files.png)

Para probar el paralelismo de la herramienta blobxfer, el tamaño máximo de archivo para las copias de seguridad HANA se estableció en 15 GB, que generó 19 archivos de copia de seguridad. Esta configuración provocó que el tiempo en el que blobxfer copió 230 GB en Azure Blob Storage descendiera de 3000 a 875 segundos.

A medida que explora la copia de copias de seguridad realizadas en discos locales en otras ubicaciones, como Azure Blob Storage, tenga en cuenta que el ancho de banda que usa un eventual proceso de copia en paralelo se contabiliza en la red o en la cuota de almacenamiento de su tipo de máquina virtual individual. Por consiguiente, debe equilibrar la duración de la copia en la red y el ancho de banda de almacenamiento que requiere la carga de trabajo normal que se ejecuta en la máquina virtual. 


## <a name="copy-sap-hana-backup-files-to-nfs-share"></a>Copia de archivos de copia de seguridad de SAP HA en el recurso compartido NFS

Microsoft Azure ofrece recursos compartidos de NFS nativos a través de [Azure NetApp Files](https://azure.microsoft.com/services/netapp/). Puede crear diferentes volúmenes de docenas de terabytes de capacidad para almacenar y administrar copias de seguridad. También puede tomar instantáneas de dichos mediante la tecnología de NetApp. Azure NetApp Files (ANF) se ofrece en tres niveles de servicio diferentes que proporcionan distintos rendimientos de almacenamiento. Para más información, lea el artículo [Niveles de servicio para Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md). Puede crear y montar un volumen de NFS desde ANF como se describe en el artículo [Inicio rápido: Configuración de Azure NetApp Files y creación de un volumen de NFS](../../../azure-netapp-files/azure-netapp-files-quickstart-set-up-account-create-volumes.md?tabs=azure-portal).

Además de usar volúmenes NFS nativos de Azure a través de ANF, existen varias posibilidades de crear implementaciones propias que proporcionen recursos compartidos de NFS en Azure. Todas ellas tienen la desventaja de que es el propio usuario el que necesita implementar y administrar las soluciones. En estos artículos se describen algunas de estas posibilidades:

- [Alta disponibilidad para NFS en máquinas virtuales de Azure en SUSE Linux Enterprise Server](./high-availability-guide-suse-nfs.md)
- [GlusterFS en Azure Virtual Machines en Red Hat Enterprise Linux para SAP NetWeaver](./high-availability-guide-rhel-glusterfs.md)

Los recursos compartidos de NFS creados por los medios descritos anteriormente se pueden usar para ejecutar directamente copias de seguridad de HANA en esos recursos compartidos de NFS o para copiar copias de seguridad realizadas en discos locales en ellos.

> [!NOTE]
> SAP HANA admite NFS, versión 3 y NFS, versión 4.x. Cualquier otro formato, como SMB con el sistema de archivos CIFS, no se admite para escribir copias de seguridad de HANA. Consulte también la [nota de soporte técnico de SAP 1820529](https://launchpad.support.sap.com/#/notes/1820529)

## <a name="copy-sap-hana-backup-files-to-azure-files"></a>Copia de archivos de copia de seguridad de SAP HANA en Azure Files

Es posible montar un recurso compartido de archivos de Azure Files dentro de una máquina virtual Linux de Azure. En el artículo [Uso de Azure File Storage con Linux](../../../storage/files/storage-how-to-use-files-linux.md) encontrará información sobre cómo realizar la configuración. Para conocer las limitaciones de Azure Files o Azure Premium Files, lea el artículo [Objetivos de escalabilidad y rendimiento de Azure Files](../../../storage/files/storage-files-scale-targets.md).

> [!NOTE]
> SAP HANA no admite SMB con el sistema de archivos CIFS para escribir copias de seguridad de HANA. Consulte también la [nota de soporte técnico de SAP 1820529](https://launchpad.support.sap.com/#/notes/1820529). Como resultado, esta solución solo se puede usar como destino final de una copia de seguridad de base de datos de HANA que se ha llevado a cabo directamente en discos locales conectados
> 

En una prueba realizada en Azure Files, no en Azure Premium Files, tardó aproximadamente 929 segundos en copiar 19 archivos de copia de seguridad con un volumen global de 230 GB. Esperamos que si se usa Azure Premium Files el tiempo mejore. Sin embargo, es preciso tener en cuenta que es preciso equilibrar los intereses de una copia rápida con los requisitos que la carga de trabajo tiene en el ancho de banda de red. Dado que todos los tipos de máquina virtual de Azure aplica la cuota de ancho de banda de la red, es preciso permanecer dentro del rango de dicha cuota con la carga de trabajo, además de la copia de los archivos de copia de seguridad.

![Esta ilustración muestra que se tardó aproximadamente 929 segundos en copiar 19 archivos de copia de seguridad de SAP HANA](media/sap-hana-backup-file-level/parallel-copy-to-azure-files.png)


El almacenamiento de los archivos de copia de seguridad de SAP HANA en Azure Files podría ser una opción interesante, especialmente con la latencia y el rendimiento mejorados de Azure Premium Files.

## <a name="next-steps"></a>Pasos siguientes
* [Guía de copia de seguridad de SAP HANA en Azure Virtual Machines](sap-hana-backup-guide.md) ofrece una introducción e información sobre los pasos iniciales.
* Para obtener información sobre cómo establecer la alta disponibilidad y planear la recuperación ante desastres de SAP HANA en Azure (instancias grandes), vea [Alta disponibilidad y recuperación ante desastres de SAP HANA en Azure (instancias grandes)](hana-overview-high-availability-disaster-recovery.md).