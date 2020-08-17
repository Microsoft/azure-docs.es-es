---
title: Mover el almacenamiento del sistema central a Azure Storage
description: Los recursos de almacenamiento de Azure escalables de forma masiva pueden ayudar a las organizaciones basadas en el sistema central a migrar y modernizar las aplicaciones z14 de IBM.
author: njray
ms.author: larryme
ms.date: 04/02/2019
ms.topic: article
ms.service: storage
ms.openlocfilehash: 7f83d798bbffeb232b45701beee828114ad3db7d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87042597"
---
# <a name="move-mainframe-storage-to-azure"></a>Mover el almacenamiento del sistema central a Azure

Para ejecutar cargas de trabajo de sistema central en Microsoft Azure, debe saber cómo las funcionalidades del sistema central se comparan con Azure. Los recursos de almacenamiento escalables de forma masiva pueden ayudar a las organizaciones a empezar a modernizarse sin dejar de usar las aplicaciones en las que dependen.

Azure proporciona características similares a las del sistema central y una capacidad de almacenamiento comparable a la de los sistemas basados en z14 de IBM (el modelo más reciente durante la redacción de este documento). En este artículo se explica cómo obtener resultados comparables en Azure.

## <a name="mainframe-storage-at-a-glance"></a>Información general sobre el almacenamiento del sistema central

El sistema central de IBM caracteriza el almacenamiento de dos formas: la primera es un dispositivo de almacenamiento de acceso directo (DASD) y la segunda es almacenamiento secuencial. Para administrar el almacenamiento, el sistema central proporciona el subsistema de administración de almacenamiento de la facilidad de datos (DFSMS), que administra el acceso a datos a varios dispositivos de almacenamiento.

[DASD](https://en.wikipedia.org/wiki/Direct-access_storage_device) hace referencia a un dispositivo independiente para el almacenamiento secundario (no en memoria) que permite el uso de una dirección única que se usará para el acceso directo de los datos. Originalmente, el término DASD se aplicaba a unidades de discos, tambores magnéticos o celdas de datos. Sin embargo, ahora el término también se puede aplicar a dispositivos de almacenamiento de estado sólido (SSD), redes de área de almacenamiento (SAN), almacenamiento adjunto a la red (NAS) y unidades de disco ópticas. Para los fines de este documento, DASD hace referencia a unidades de disco, SAN y SSD.

A diferencia del almacenamiento DASD, el almacenamiento secuencial en un sistema central hace referencia a dispositivos como unidades de cinta en las que se accede a los datos desde un punto de partida y después se leen o escriben en una línea.

Los dispositivos de almacenamiento acostumbran a conectarse mediante una conexión de fibra (FICON) o se accede directamente a ellos desde el bus de E/S del sistema central usando [HiperSockets](https://www.ibm.com/support/knowledgecenter/zosbasics/com.ibm.zos.znetwork/znetwork_85.htm), una tecnología de IBM para comunicaciones de alta velocidad entre las particiones de un servidor con un hipervisor.

La mayoría de los sistemas centrales diferencian entre dos tipos de almacenamiento:

- *Almacenamiento en línea* (también conocido como almacenamiento de acceso frecuente): es necesario para las operaciones diarias. Normalmente, el almacenamiento DASD se usa con este fin. Sin embargo, para ello también se puede usar el almacenamiento secuencial, como las copias de seguridad en cinta diarias (lógicas o físicas).

- *Almacenamiento de archivo* (también conocido como almacenamiento en frío): no se garantiza que se monte en un momento determinado, sino que se monta y se accede a él cuando es necesario. A menudo, el almacenamiento de archivo se implementa mediante copias de seguridad en cinta secuenciales (físicas o lógicas) para el almacenamiento.

## <a name="mainframe-versus-io-latency-and-iops"></a>Sistema central frente a latencia de E/S e IOPS

Con frecuencia se usan los sistemas centrales para las aplicaciones que requieren un alto rendimiento de E/S y una baja latencia de E/S. Pueden hacerlo usando las conexiones FICON a dispositivos de E/S e HiperSockets. Cuando se usan HiperSockets para conectar dispositivos y aplicaciones directamente al canal de E/S de un sistema central, la latencia se puede conseguir en microsegundos.

## <a name="azure-storage-at-a-glance"></a>Información general sobre el almacenamiento de Azure

Las opciones de infraestructura como servicio ([IaaS](https://azure.microsoft.com/overview/what-is-iaas/)) de Azure para el almacenamiento ofrecen una capacidad de sistema central comparable.

Microsoft ofrece petabytes de almacenamiento para las aplicaciones hospedadas en Azure, con lo que tendrá varias opciones de almacenamiento, que van desde almacenamiento SSD para alto rendimiento hasta almacenamiento de blobs de bajo coste para archivos y almacenamiento masivo. Además, Azure proporciona una opción de redundancia de datos para el almacenamiento, que es más difícil de configurar en un entorno de sistemas centrales.

El almacenamiento de Azure está disponible como [Azure Disks](../../../windows/managed-disks-overview.md), [Azure Files](../../../../storage/files/storage-files-introduction.md) y [Azure Blobs](../../../../storage/blobs/storage-blobs-overview.md), tal como se resume en la tabla siguiente. Obtenga más información sobre [cuándo usar cada tipo de almacenamiento](../../../../storage/common/storage-introduction.md).

<!-- markdownlint-disable MD033 -->

<table>
<thead>
    <tr><th>Tipo</th><th>Descripción</th><th>Úselo cuando quiera:</th></tr>
</thead>
<tbody>
<tr><td>Archivos de Azure
</td>
<td>
Proporciona una interfaz de SMB, bibliotecas de cliente y una interfaz de <a href="https://docs.microsoft.com/rest/api/storageservices/file-service-rest-api">REST</a> que permite acceder desde cualquier lugar a los archivos almacenados.
</td>
<td><ul>
<li>Migrar mediante lift-and-shift una aplicación a la nube cuando la aplicación usa las API del sistema de archivos nativos para compartir datos entre esta y otras aplicaciones que se ejecutan en Azure.</li>
<li>Almacenar herramientas de desarrollo y depuración a las que es necesario acceder desde muchas máquinas virtuales.</li>
</ul>
</td>
</tr>
<tr><td>Azure Blobs
</td>
<td>Proporciona bibliotecas de cliente y una interfaz de <a href="https://docs.microsoft.com/rest/api/storageservices/blob-service-rest-api">REST</a> que permite el almacenamiento de los datos no estructurados y el acceso a ellos a gran escala en blobs en bloques. También admite <a href="/azure/storage/blobs/data-lake-storage-introduction">Azure Data Lake Storage Gen2</a> para soluciones de análisis de macrodatos empresariales.
</td>
<td><ul>
<li>Tener escenarios de acceso aleatorio y streaming en una aplicación.</li>
<li>Tener acceso a datos de la aplicación desde cualquier lugar.</li>
<li>Crear una instancia empresarial de lago de datos en Azure y realizar análisis de macrodatos.</li>
</ul></td>
</tr>
<tr><td>Azure Disks
</td>
<td>Proporciona bibliotecas de cliente y una interfaz de <a href="https://docs.microsoft.com/rest/api/compute/disks">REST</a> que permite el almacenamiento persistente de los datos y el acceso a ellos desde un disco duro virtual conectado.
</td>
<td><ul>
<li>Migrar mediante lift-and-shift aplicaciones que usan las API del sistema de archivos nativo para leer y escribir datos en discos persistentes.</li>
<li>Almacenar datos a los que no se necesita acceder desde fuera de la máquina virtual a la que está conectado el disco.</li>
</ul></td>
</tr>
</tbody>
</table>
<!-- markdownlint-enable MD033 -->

## <a name="azure-hot-online-and-cold-archive-storage"></a>Almacenamiento en caliente (en línea) y en frío (en archivo) de Azure

El tipo de almacenamiento de un sistema determinado depende de los requisitos del sistema, que incluyen tamaño de almacenamiento, rendimiento e IOPS. En cambio, para el almacenamiento de tipo DASD en un sistema central, las aplicaciones de Azure normalmente usan almacenamiento en unidades de Azure Disks. Para el almacenamiento de archivo del sistema central, se usa el almacenamiento de blobs en Azure.

Las SSD ofrecen el máximo rendimiento de almacenamiento en Azure. Las siguientes opciones están disponibles (en el momento de escribir este documento):

| Tipo         | Size           | E/S                  |
|--------------|----------------|-----------------------|
| SSD ultra    | De 4 GB a 64 TB  | De 1200 a 160.000 IOPS |
| SSD Premium  | De 32 GB a 32 TB | De 12 a 15.000 IOPS     |
| SSD estándar | De 32 GB a 32 TB | De 12 a 2.000 IOPS      |

El almacenamiento de blobs proporciona el mayor volumen de almacenamiento en Azure. Además de tamaño de almacenamiento, Azure ofrece tanto almacenamiento administrado como no administrado. Con el almacenamiento administrado, Azure se encarga de administrar las cuentas de almacenamiento subyacente. Con el almacenamiento no administrado, el usuario se encarga de configurar las cuentas de almacenamiento de Azure del tamaño adecuado para cumplir los requisitos de almacenamiento.

## <a name="next-steps"></a>Pasos siguientes

- [Migración del sistema central](/azure/architecture/cloud-adoption/infrastructure/mainframe-migration/overview)
- [Rehospedaje del sistema central en Azure Virtual Machines](../overview.md)
- [Mover el proceso del sistema central a Azure](mainframe-compute-Azure.md)
- [Decidir cuándo usar Azure Blobs, Azure Files o Azure Disks](../../../../storage/common/storage-introduction.md)
- [Discos administrados SSD estándar para cargas de trabajo de la máquina virtual de Azure](../../../windows/disks-types.md#standard-ssd)

### <a name="ibm-resources"></a>Recursos de IBM

- [Parallel Sysplex on IBM Z](https://www.ibm.com/it-infrastructure/z/technologies/parallel-sysplex-resources) (Parallel Sysplex en IBM Z)
- [IBM CICS and the Coupling Facility: Beyond the Basics](https://www.redbooks.ibm.com/redbooks/pdfs/sg248420.pdf) (IBM CICS y el recurso de acoplamiento: más allá de los aspectos básicos)
- [Creación de los usuarios necesarios para una instalación de Db2 pureScale Feature](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_11.1.0/com.ibm.db2.luw.qb.server.doc/doc/t0055374.html?pos=2)
- [Mandato db2icrt - Crear instancia](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_11.1.0/com.ibm.db2.luw.admin.cmd.doc/doc/r0002057.html)
- [Db2 pureScale Clustered Database Solution](https://www.ibmbigdatahub.com/blog/db2-purescale-clustered-database-solution-part-1) (Solución de base de datos en clúster de Db2 pureScale)
- [IBM Data Studio](https://www.ibm.com/developerworks/downloads/im/data/index.html/)

### <a name="azure-government"></a>Azure Government

- [Microsoft Azure Government cloud for mainframe applications](https://azure.microsoft.com/resources/microsoft-azure-government-cloud-for-mainframe-applications/) (Nube de Microsoft Azure Government para aplicaciones de sistema central)
- [Microsoft y FedRAMP](https://www.microsoft.com/TrustCenter/Compliance/FedRAMP)

### <a name="more-migration-resources"></a>Más recursos sobre migración

- [Azure Virtual Data Center Lift and Shift Guide](https://azure.microsoft.com/resources/azure-virtual-datacenter-lift-and-shift-guide/) (Centro de datos virtual de Azure: guía para una migración lift-and-shift)
- [iSCSI en GlusterFS](https://docs.gluster.org/en/latest/Administrator%20Guide/GlusterFS%20iSCSI/)
