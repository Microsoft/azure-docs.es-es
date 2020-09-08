---
title: Implementación de DBMS de Azure Virtual Machines de SAP ASE para la carga de trabajo de SAP | Microsoft Docs
description: Implementación de DBMS de Azure Virtual Machines de SAP ASE para la carga de trabajo de SAP
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 04/13/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: ce13c3bce7cdeb0f3e6dcf1f731be22d93a65587
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88654606"
---
# <a name="sap-ase-azure-virtual-machines-dbms-deployment-for-sap-workload"></a>Implementación de DBMS de Azure Virtual Machines de SAP ASE para la carga de trabajo de SAP

En este documento se describen las diferentes áreas que se deben tener en cuenta al implementar SAP ASE en IaaS de Azure. Como condición previa a este documento, debe haber leído el documento [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](dbms_guide_general.md) y otras guías de la [documentación de carga de trabajo de SAP en Azure](./get-started.md). En este documento se describe SAP ASE que se ejecuta en Linux y en sistemas operativos Windows. La versión mínima admitida en Azure es SAP ASE 16.0.02 (versión 16, paquete de soporte 2). Se recomienda implementar la versión más reciente de SAP y el nivel de revisión (PL) más reciente.  Como mínimo, se recomienda SAP ASE 16.0.03.07 (versión 16, paquete de soporte 3, nivel de revisión 7).  La versión más reciente de SAP puede encontrarse en [Targeted ASE 16.0 Release Schedule and CR list Information](https://wiki.scn.sap.com/wiki/display/SYBASE/Targeted+ASE+16.0+Release+Schedule+and+CR+list+Information).

Además, encontrará información adicional sobre la compatibilidad de versiones con las aplicaciones de SAP o la ubicación de los medios de instalación en la matriz de disponibilidad de productos de SAP en estas ubicaciones:

- [Nota de soporte técnico de SAP 2134316](https://launchpad.support.sap.com/#/notes/2134316)
- [Nota de soporte técnico de SAP 1941500](https://launchpad.support.sap.com/#/notes/1941500)
- [Nota de soporte técnico de SAP 1590719](https://launchpad.support.sap.com/#/notes/1590719)
- [Nota de soporte técnico de SAP 1973241](https://launchpad.support.sap.com/#/notes/1973241)

Comentario: En toda la documentación dentro y fuera del mundo de SAP, se hace referencia al nombre del producto como Sybase ASE o SAP ASE o, en algunos casos, ambos. Con el fin de mantener la coherencia, en esta documentación usamos el nombre **SAP ASE**.

## <a name="operating-system-support"></a>Compatibilidad con sistema operativo
La matriz de disponibilidad de productos de SAP contiene las combinaciones de sistema operativo y kernel de SAP compatibles para cada aplicación de SAP.  Las distribuciones de Linux SUSE 12.x, SUSE 15.x, Red Hat 7.x son totalmente compatibles.  No se admite Oracle Linux como sistema operativo para SAP ASE.  Se recomienda usar las versiones más recientes de Linux disponibles. Los clientes de Windows deben usar las versiones de Windows Server 2016 o Windows Server 2019.  Técnicamente, se admiten las versiones anteriores de Windows, como Windows 2012, pero siempre se recomienda la versión más reciente de Windows.


## <a name="specifics-to-sap-ase-on-windows"></a>Características específicas de SAP ASE en Windows
Desde Microsoft Azure, se pueden migrar las aplicaciones de SAP ASE a Azure Virtual Machines. SAP ASE en una máquina virtual de Azure permite reducir el costo total de propiedad de implementación, administración y mantenimiento de las aplicaciones empresariales al migrarlas fácilmente a Microsoft Azure. Con SAP ASE en una máquina virtual de Azure, los administradores y los desarrolladores pueden seguir usando las mismas herramientas de desarrollo y administración que se encuentran disponibles de forma local.

Microsoft Azure ofrece numerosos tipos de máquinas virtuales diferentes que permiten ejecutar desde los sistemas e infraestructuras SAP más pequeños hasta otros de gran tamaño con miles de usuarios. Los números de SAPS de tamaño de SAP de las distintas SKU para VM certificadas por SAP se proporcionan en la [nota de soporte técnico de SAP 1928533](https://launchpad.support.sap.com/#/notes/1928533).

Puede encontrar documentación para instalar SAP ASE en Windows en la [guía de instalación para Windows de SAP ASE](https://help.sap.com/viewer/36031975851a4f82b1022a9df877280b/16.0.3.7/en-US/a660d3f1bc2b101487cbdbf10069c3ac.html)

Bloquear páginas en la memoria es un valor que evitará que el búfer de la base de datos de SAP ASE se transfiera.  Este valor es útil para sistemas ocupados de gran tamaño con mucha memoria. Para más información, póngase en contacto con BC-DB-SYB. 


## <a name="linux-operating-system-specific-settings"></a>Configuración específica del sistema operativo Linux
En VM Linux, la ejecución de `saptune` con el perfil Linux Huge Pages de SAP-ASE debe estar habilitado de forma predeterminada y se pueden verificar con el comando  

`cat /proc/meminfo` 

Normalmente, el tamaño de página es de 2048 KB. Para obtener más información, consulte el artículo [Huge Pages on Linux](https://help.sap.com/viewer/ecbccd52e7024feaa12f4e780b43bc3b/16.0.3.7/en-US/a703d580bc2b10149695f7d838203fad.html). 


## <a name="recommendations-on-vm-and-disk-structure-for-sap-ase-deployments"></a>Recomendaciones sobre la estructura de discos y VM para implementaciones de SAP ASE

SAP ASE para SAP NetWeaver Applications es compatible con cualquier tipo de VM que se enumere en la [nota de soporte técnico de SAP 1928533](https://launchpad.support.sap.com/#/notes/1928533). Los tipos de VM habituales que se usan para los servidores de bases de datos de SAP ASE de tamaño medio incluyen Esv3.  Las grandes bases de datos de varios terabytes pueden aprovechar los tipos de VM de la serie M. El rendimiento de escritura en disco del registro de transacciones de SAP ASE se puede mejorar si se habilita el Acelerador de escritura de la serie M. El Acelerador de escritura debe probarse atentamente con SAP ASE debido a la manera en que SAP ASE realiza escrituras en el registro.  Revise la [nota de soporte técnico de SAP 2816580](../../how-to-enable-write-accelerator.md) y tenga en cuenta la posibilidad de ejecutar una prueba de rendimiento.  
El Acelerador de escritura está diseñado únicamente para el disco del registro de transacciones. La memoria caché de nivel de disco debe establecerse en NONE. No se sorprenda si el Acelerador de escritura de Azure no muestra mejoras similares como con otro DBMS. En función de la forma en que SAP ASE escribe en el registro de transacciones, podría deberse a que el Acelerador de escritura de Azure muestre poca o ninguna aceleración.
Se recomiendan discos independientes para dispositivos de datos y dispositivos de registro.  Las bases de datos del sistema sybsecurity y `saptools` no requieren discos dedicados y se pueden colocar en los discos que contienen los dispositivos de datos y de registro de la base de datos de SAP. 

![Configuración del almacenamiento en SAP ASE](./media/dbms-guide-sap-ase/sap-ase-disk-structure.png)

### <a name="file-systems-stripe-size--io-balancing"></a>Sistemas de archivos, tamaño de franjas y equilibrio de E/S 
SAP ASE escribe los datos secuencialmente en los dispositivos de almacenamiento en disco, a menos que se configure lo contrario. Esto significa que una base de datos de SAP ASE vacía con cuatro dispositivos escribirá datos solo en el primer dispositivo.  Solo se escribirá en los demás dispositivos de disco cuando el primer dispositivo esté lleno.  Es probable que la cantidad de E/S de lectura y escritura en cada dispositivo de SAP ASE sea diferente. Para equilibrar la E/S de disco en todos los discos de Azure disponibles, es necesario usar espacios de almacenamiento de Windows o Linux LVM2. En Linux, se recomienda usar el sistema de archivos XFS para formatear los discos. El tamaño de franja LVM debe probarse con una prueba de rendimiento. Un buen punto de partida es un tamaño de franja de 128 KB. En Windows, se debe probar el tamaño de la unidad de asignación (AUS) de NTFS. Se puede usar 64 KB como valor inicial. 

Se recomienda configurar la expansión automática de base de datos como se describe en el artículo [Configuring Automatic Database Space Expansion in SAP Adaptive Server Enterprise](https://blogs.sap.com/2014/07/09/configuring-automatic-database-space-expansion-in-sap-adaptive-server-enterprise/) y en la [nota de soporte técnico de SAP 1815695](https://launchpad.support.sap.com/#/notes/1815695). 

### <a name="sample-sap-ase-on-azure-virtual-machine-disk-and-file-system-configurations"></a>Ejemplo de SAP ASE en máquinas virtuales de Azure, configuraciones de disco y sistema de archivos 
Las plantillas siguientes muestran configuraciones de ejemplo para Linux y Windows. Antes de confirmar la configuración de máquina virtual y disco, asegúrese de que las cuotas de ancho de banda de red y almacenamiento de la VM individual sean suficientes para satisfacer los requisitos empresariales. Tenga en cuenta también que diferentes tipos de VM de Azure tienen un número máximo de discos que se pueden conectar a la VM. Por ejemplo, una VM E4s_v3 tiene un rendimiento límite de E/S de almacenamiento de 48 MB/s. Si el rendimiento de almacenamiento requerido por la actividad de copia de seguridad de base de datos requiere más de 48 MB/s, es inevitable usar un tipo de VM mayor con más rendimiento de ancho de banda de almacenamiento. Al configurar Azure Storage, también debe tener en cuenta que, especialmente con [Azure Premium Storage](../../windows/premium-storage-performance.md), cambian el rendimiento y las IOPS por GB de capacidad. Consulte más información sobre este tema en el artículo [¿Qué tipos de disco están disponibles en Azure?](../../disks-types.md) Las cuotas para tipos específicos de VM de Azure se documentan en el artículo [Tamaños de máquina virtual optimizada para memoria](../../sizes-memory.md) y en otros artículos vinculados a este. 

> [!NOTE]
>  Si va a mover un sistema DBMS de un entorno local a Azure, se recomienda realizar la supervisión en la VM y evaluar la CPU, la memoria, las IOPS y el rendimiento de almacenamiento. Compare los valores máximos observados con los límites de cuota de VM documentados en los artículos mencionados anteriormente.

Los ejemplos que se indican a continuación tienen fines ilustrativos y se pueden modificar en función de las necesidades individuales. Debido al diseño de SAP ASE, el número de dispositivos de datos no es tan importante como con otras bases de datos. El número de dispositivos de datos que se detallan en este documento es solo una guía. 

Un ejemplo de una configuración para un servidor pequeño de base de datos de SAP ASE con un tamaño de base de datos entre 50 y 250 GB, como el administrador de soluciones de SAP, podría ser similar a lo siguiente:

| Configuración | Windows | Linux | Comentarios |
| --- | --- | --- | --- |
| Tipo de máquina virtual | E4s_v3 (4 vCPU/32 GB de RAM) | E4s_v3 (4 vCPU/32 GB de RAM) | --- |
| Redes aceleradas | Habilitar | Habilitar | ---|
| Versión de SAP ASE | 16.0.03.07 o superior | 16.0.03.07 o superior | --- |
| N.º de dispositivos de datos | 4 | 4 | ---|
| N.º de dispositivos de registro | 1 | 1 | --- |
| N.º de dispositivos temporales | 1 | 1 | Más para la carga de trabajo de SAP BW |
| Sistema operativo | Windows Server 2019 | SUSE 12 SP4/15 SP1 o RHEL 7.6 | --- |
| Agregación de discos | Espacios de almacenamiento | LVM2 | --- |
| Sistema de archivos | NTFS | XFS |
| Tamaño de bloque de formato | Necesita pruebas de carga de trabajo | Necesita pruebas de carga de trabajo | --- |
| N.º y tipo de discos de datos | Premium Storage: 2 x P10 (RAID0) | Premium Storage: 2 x P10 (RAID0)| Caché = Solo lectura |
| N.º y tipo de discos de registro | Premium Storage: 1 x P20  | Premium Storage: 1 x P20 | Caché = NONE |
| Parámetro MaxMemory de ASE | 90 % de RAM física | 90 % de RAM física | Suponiendo una sola instancia |
| N.º de dispositivos de copia de seguridad | 4 | 4| --- |
| N.º y tipo de discos de copia de seguridad | 1 | 1 | --- |


Un ejemplo de una configuración para un servidor medio de base de datos de SAP ASE con un tamaño de base de datos entre 250 y 750 GB, como el sistema SAP Business Suite más pequeño, podría ser similar a lo siguiente:

| Configuración | Windows | Linux | Comentarios |
| --- | --- | --- | --- |
| Tipo de máquina virtual | E16s_v3 (16 vCPU/128 GB de RAM) | E16s_v3 (16 vCPU/128 GB de RAM) | --- |
| Redes aceleradas | Habilitar | Habilitar | ---|
| Versión de SAP ASE | 16.0.03.07 o superior | 16.0.03.07 o superior | --- |
| N.º de dispositivos de datos | 8 | 8 | ---|
| N.º de dispositivos de registro | 1 | 1 | --- |
| N.º de dispositivos temporales | 1 | 1 | Más para la carga de trabajo de SAP BW |
| Sistema operativo | Windows Server 2019 | SUSE 12 SP4/15 SP1 o RHEL 7.6 | --- |
| Agregación de discos | Espacios de almacenamiento | LVM2 | --- |
| Sistema de archivos | NTFS | XFS |
| Tamaño de bloque de formato | Necesita pruebas de carga de trabajo | Necesita pruebas de carga de trabajo | --- |
| N.º y tipo de discos de datos | Premium Storage: 4 x P20 (RAID0) | Premium Storage: 4 x P20 (RAID0)| Caché = Solo lectura |
| N.º y tipo de discos de registro | Premium Storage: 1 x P20  | Premium Storage: 1 x P20 | Caché = NONE |
| Parámetro MaxMemory de ASE | 90 % de RAM física | 90 % de RAM física | Suponiendo una sola instancia |
| N.º de dispositivos de copia de seguridad | 4 | 4| --- |
| N.º y tipo de discos de copia de seguridad | 1 | 1 | --- |

Un ejemplo de una configuración para un servidor pequeño de base de datos de SAP ASE con un tamaño de base de datos entre 750 y 2000 GB, como el sistema SAP Business Suite más grande, podría ser similar a lo siguiente:

| Configuración | Windows | Linux | Comentarios |
| --- | --- | --- | --- |
| Tipo de máquina virtual | E64s_v3 (64 vCPU/432 GB de RAM) | E64s_v3 (64 vCPU/432 GB de RAM) | --- |
| Redes aceleradas | Habilitar | Habilitar | ---|
| Versión de SAP ASE | 16.0.03.07 o superior | 16.0.03.07 o superior | --- |
| N.º de dispositivos de datos | 16 | 16 | ---|
| N.º de dispositivos de registro | 1 | 1 | --- |
| N.º de dispositivos temporales | 1 | 1 | Más para la carga de trabajo de SAP BW |
| Sistema operativo | Windows Server 2019 | SUSE 12 SP4/15 SP1 o RHEL 7.6 | --- |
| Agregación de discos | Espacios de almacenamiento | LVM2 | --- |
| Sistema de archivos | NTFS | XFS |
| Tamaño de bloque de formato | Necesita pruebas de carga de trabajo | Necesita pruebas de carga de trabajo | --- |
| N.º y tipo de discos de datos | Premium Storage: 4 x P30 (RAID0) | Premium Storage: 4 x P30 (RAID0)| Caché = Solo lectura |
| N.º y tipo de discos de registro | Premium Storage: 1 x P20  | Premium Storage: 1 x P20 | Caché = NONE |
| Parámetro MaxMemory de ASE | 90 % de RAM física | 90 % de RAM física | Suponiendo una sola instancia |
| N.º de dispositivos de copia de seguridad | 4 | 4| --- |
| N.º y tipo de discos de copia de seguridad | 1 | 1 | --- |


Un ejemplo de una configuración para un servidor pequeño de base de datos de SAP ASE con un tamaño de base de datos de más de 2 TB, como un sistema SAP Business Suite más grande usado ampliamente, podría ser similar a lo siguiente:

| Configuración | Windows | Linux | Comentarios |
| --- | --- | --- | --- |
| Tipo de máquina virtual | Serie M (de 1,0 a 4,0 TB de RAM)  | Serie M (de 1,0 a 4,0 TB de RAM) | --- |
| Redes aceleradas | Habilitar | Habilitar | ---|
| Versión de SAP ASE | 16.0.03.07 o superior | 16.0.03.07 o superior | --- |
| N.º de dispositivos de datos | 32 | 32 | ---|
| N.º de dispositivos de registro | 1 | 1 | --- |
| N.º de dispositivos temporales | 1 | 1 | Más para la carga de trabajo de SAP BW |
| Sistema operativo | Windows Server 2019 | SUSE 12 SP4/15 SP1 o RHEL 7.6 | --- |
| Agregación de discos | Espacios de almacenamiento | LVM2 | --- |
| Sistema de archivos | NTFS | XFS |
| Tamaño de bloque de formato | Necesita pruebas de carga de trabajo | Necesita pruebas de carga de trabajo | --- |
| N.º y tipo de discos de datos | Premium Storage: 4 x P30 (RAID0) | Premium Storage: 4 x P30 (RAID0)| Caché = Solo lectura, tenga en cuenta discos Ultra de Azure |
| N.º y tipo de discos de registro | Premium Storage: 1 x P20  | Premium Storage: 1 x P20 | Caché = NONE, tenga en cuenta discos Ultra de Azure |
| Parámetro MaxMemory de ASE | 90 % de RAM física | 90 % de RAM física | Suponiendo una sola instancia |
| N.º de dispositivos de copia de seguridad | 16 | 16 | --- |
| N.º y tipo de discos de copia de seguridad | 4 | 4 | Use LVM2/Espacios de almacenamiento |


### <a name="backup--restore-considerations-for-sap-ase-on-azure"></a>Consideraciones de copia de seguridad y restauración para SAP ASE en Azure
Aumentar el número de dispositivos de datos y de copia de seguridad aumenta el rendimiento de copia de seguridad y de restauración. Se recomienda seccionar los discos de Azure que hospedan el dispositivo de copia de seguridad de SAP ASE tal como se muestra en las tablas anteriores. Se debe tener cuidado para equilibrar el número de discos y dispositivos de copia de seguridad y asegurarse de que el rendimiento de copia de seguridad no supere el 40 al 50 % de la cuota total de rendimiento de la VM. Se recomienda usar la compresión de copia de seguridad de SAP como valor predeterminado. Se puede encontrar más información en los artículos siguientes:

- [Nota de soporte técnico de SAP 1588316](https://launchpad.support.sap.com/#/notes/1588316)
- [Nota de soporte técnico de SAP 1801984](https://launchpad.support.sap.com/#/notes/1801984)
- [Nota de soporte técnico de SAP 1585981](https://launchpad.support.sap.com/#/notes/1585981) 

No utilice la unidad D:\ ni espacio temporal como destino de los volcados de bases de datos o registros.

### <a name="impact-of-database-compression"></a>Impacto de la compresión de las bases de datos
En configuraciones donde el ancho de banda de E/S puede volverse un factor limitante, las medidas que reduzcan el número de IOPS podrían ayudar a ajustar la carga de trabajo que se puede ejecutar en un escenario de IaaS, como Azure. Por lo tanto, se recomienda asegurarse de emplear la compresión de SAP ASE antes de cargar una base de datos existente de SAP en Azure.

La recomendación de aplicar la compresión antes de cargar en Azure viene motivada por varias razones:

* La cantidad de datos que se cargarán en Azure es menor.
* La duración de la ejecución de la compresión es menor, suponiendo que se pueda utilizar un hardware más eficaz con más capacidad de CPU, un mayor ancho de banda de E/S o una latencia de E/S menor en el entorno local.
* Si se utilizan tamaños de base de datos más pequeños, se incurriría en menos costes de asignación de disco.

La compresión de datos y de LOB funciona en una máquina virtual hospedada en Azure Virtual Machines del mismo modo a como lo hace en un entorno local. Para obtener más información sobre cómo comprobar si ya se está usando compresión en una base de datos de SAP ASE existente, consulte la [nota de soporte técnico de SAP 1750510](https://launchpad.support.sap.com/#/notes/1750510). Para obtener más información sobre la compresión de base de datos de SAP ASE, consulte la [nota de soporte técnico de SAP 2121797](https://launchpad.support.sap.com/#/notes/2121797).

## <a name="high-availability-of-sap-ase-on-azure"></a>Alta disponibilidad de SAP ASE en Azure 
La guía del usuario de HADR detalla la instalación y configuración de una solución "Always-On" de dos nodos de SAP ASE.  Además, también se admite un tercer nodo de recuperación ante desastres. SAP ASE admite muchas configuraciones de alta disponibilidad, entre otras, discos compartidos y la agrupación en clústeres de sistema operativo nativo (IP flotante). La única configuración admitida en Azure es usar el administrador de errores sin la dirección IP flotante.  El método de dirección IP flotante no funcionará en Azure.  El kernel de SAP es una aplicación "compatible con alta disponibilidad" y conoce los servidores de SAP ASE principal y secundarios. No hay ninguna integración cercana entre SAP ASE y Azure, pero no se usa el equilibrador de carga interno de Azure. Por lo tanto, se debe seguir la documentación estándar de SAP ASE a partir de [guía del usuario de HADR de SAP ASE](https://help.sap.com/viewer/efe56ad3cad0467d837c8ff1ac6ba75c/16.0.3.7/en-US/a6645e28bc2b1014b54b8815a64b87ba.html). 

> [!NOTE]
> La única configuración admitida en Azure es usar el administrador de errores sin la dirección IP flotante.  El método de dirección IP flotante no funcionará en Azure. 

### <a name="third-node-for-disaster-recovery"></a>Tercer nodo para la recuperación ante desastres
Más allá del uso de SAP ASE Always-on para la alta disponibilidad local, puede que desee extender la configuración a un nodo replicado de forma asincrónica en otra región de Azure. La documentación para dicho escenario se puede encontrar [aquí](https://techcommunity.microsoft.com/t5/running-sap-applications-on-the/installation-procedure-for-sybase-16-3-patch-level-3-always-on/ba-p/368199).

## <a name="sap-ase-database-encryption--ssl"></a>Cifrado de base de datos y SSL de SAP ASE 
SAP Software Provisioning Manager (SWPM) proporciona una opción para cifrar la base de datos durante la instalación.  Si desea usar el cifrado, se recomienda usar el cifrado de base de datos completo de SAP.  Consulte los detalles que se documentan en:

- [Nota de soporte técnico de SAP 2556658](https://launchpad.support.sap.com/#/notes/2556658)
- [Nota de soporte técnico de SAP 2224138](https://launchpad.support.sap.com/#/notes/2224138)
- [Nota de soporte técnico de SAP 2401066](https://launchpad.support.sap.com/#/notes/2401066)
- [Nota de soporte técnico de SAP 2593925](https://launchpad.support.sap.com/#/notes/2593925) 

> [!NOTE]
> Si una base de datos de SAP ASE está cifrada, la compresión de volcado de copia de seguridad no funcionará. Consulte también la [nota de soporte técnico de SAP 2680905](https://launchpad.support.sap.com/#/notes/2680905). 

## <a name="sap-ase-on-azure-deployment-checklist"></a>Lista de comprobación de implementación de SAP ASE en Azure
 
- Implemente SAP ASE 16.0.03.07 o superior.
- Actualice a la versión más reciente y a las revisiones de FaultManager y HostAgent.
- Implemente en el sistema operativo certificado más reciente disponible, como Windows 2019, SUSE 15.1 o RedHat 7.6 o superior.
- Use VM con certificación de SAP: se recomiendan SKU de VM de Azure de memoria alta, como Es_v3 o, para sistemas de gran tamaño, SKU de VM de la serie M.
- Haga coincidir las IOPS del disco y la cuota total de rendimiento agregado de VM de la VM con el diseño del disco.  Implemente un número suficiente de discos.
- Agregue los discos mediante Espacios de almacenamiento de Windows o Linux LVM2 con el tamaño de franja y el sistema de archivos correctos.
- Cree un número suficiente de dispositivos para fines de datos, registro, datos temporales y copia de seguridad.
- Considere la posibilidad de usar discos Ultra para sistemas muy grandes. 
- Ejecute `saptune` SAP-ASE en el sistema operativo Linux. 
- Proteja la base de datos con cifrado de base de datos: almacene las claves manualmente en Azure Key Vault. 
- Complete la [lista de comprobación de SAP en Azure](./sap-deployment-checklist.md). 
- Configure la copia de seguridad de registros y la copia de seguridad completa. 
- Pruebe la alta disponibilidad y recuperación ante desastres, la copia de seguridad y restauración, y realice pruebas de carga y volumen. 
- Confirme que la extensión de base de datos automática funcione. 

## <a name="using-dbacockpit-to-monitor-database-instances"></a>Uso de DBACockpit para supervisar instancias de base de datos
En el caso de sistemas SAP que usan SAP ASE como plataforma de bases de datos, se puede acceder a DBACockpit mediante ventanas integradas en exploradores en transacciones de DBACockpit o de Webdynpro. Sin embargo, la funcionalidad completa de las actividades de supervisión y administración de bases de datos únicamente está disponible en la implementación de Webdynpro de DBACockpit.

Al igual que en el caso de los sistemas locales, es preciso seguir varios pasos para habilitar toda la funcionalidad de SAP NetWeaver que usa la implementación de Webdynpro de DBACockpit. Siga las indicaciones de la [nota de soporte técnico de SAP 1245200](https://launchpad.support.sap.com/#/notes/1245200) para habilitar el uso de los recursos de Webdynpro y generar los necesarios. Las instrucciones de las notas anteriores también le ayudarán a configurar el administrador de comunicación de Internet (`ICM`) y los puertos que se utilizarán para las conexiones http y https. La configuración de http predeterminada tiene este aspecto:

> icm/server_port_0 = PROT=HTTP,PORT=8000,PROCTIMEOUT=600,TIMEOUT=600
> 
> icm/server_port_1 = PROT=HTTPS,PORT=443$$,PROCTIMEOUT=600,TIMEOUT=600
> 
> 

y los vínculos generados en la transacción de DBACockpit tienen un aspecto similar a este:

> https:\//\<fullyqualifiedhostname>:44300/sap/bc/webdynpro/sap/dba_cockpit
> 
> http:\//\<fullyqualifiedhostname>:8000/sap/bc/webdynpro/sap/dba_cockpit
> 
> 

En función de cómo se conecta la máquina virtual de Azure que hospeda el sistema SAP a AD y DNS, debe asegurarse de que ICM usa un nombre de host completo que se puede resolver en el equipo desde el que se abre DBACockpit. Consulte la [nota de soporte técnico de SAP 773830](https://launchpad.support.sap.com/#/notes/773830) para obtener información sobre cómo ICM determina el nombre de host completo según los parámetros de perfil y, si es necesario, establezca el parámetro icm/host_name_full de forma explícita.

Si implementó la VM en un escenario solo de nube sin conectividad en varios entornos entre locales y Azure, debe definir una dirección IP pública y un valor de `domainlabel`. El formato del nombre DNS público de la máquina virtual tiene este aspecto:

> `<custom domainlabel`&gt;.`<azure region`&gt;.cloudapp.azure.com
> 
> 

Encontrará más detalles relacionados con el nombre DNS [aquí][virtual-máquinas-azurerm-frente a-azuresm].

El ajuste del parámetro de perfil de SAP icm/host_name_full con el nombre DNS del vínculo de la máquina virtual de Azure puede tener este aspecto:

> https:\//mydomainlabel.westeurope.cloudapp.net:44300/sap/bc/webdynpro/sap/dba_cockpit
> 
> http:\//mydomainlabel.westeurope.cloudapp.net:8000/sap/bc/webdynpro/sap/dba_cockpit

En ese caso, debe asegurarse de lo siguiente:

* Agregación de reglas de entrada al grupo de seguridad de red en Azure Portal para los puertos TCP/IP que se utilizan en la comunicación con el ICM
* Agregar reglas de entrada a la configuración del Firewall de Windows para los puertos TCP/IP que se utilizan en la comunicación con el ICM

Para importar de forma automatizada todas las correcciones disponibles, se recomienda aplicar periódicamente la colección de correcciones de la nota de SAP correspondiente a su versión de SAP:

* [Nota de soporte técnico de SAP 1558958](https://launchpad.support.sap.com/#/notes/1558958)
* [Nota de soporte técnico de SAP 1619967](https://launchpad.support.sap.com/#/notes/1619967)
* [Nota de soporte técnico de SAP 1882376](https://launchpad.support.sap.com/#/notes/1882376)

Encontrará más información sobre DBA Cockpit para SAP ASE en las siguientes notas de SAP:

* [Nota de soporte técnico de SAP 1605680](https://launchpad.support.sap.com/#/notes/1605680)
* [Nota de soporte técnico de SAP 1757924](https://launchpad.support.sap.com/#/notes/1757924)
* [Nota de soporte técnico de SAP 1757928](https://launchpad.support.sap.com/#/notes/1757928)
* [Nota de soporte técnico de SAP 1758182](https://launchpad.support.sap.com/#/notes/1758182)
* [Nota de soporte técnico de SAP 1758496](https://launchpad.support.sap.com/#/notes/1758496)    
* [Nota de soporte técnico de SAP 1814258](https://launchpad.support.sap.com/#/notes/1814258)
* [Nota de soporte técnico de SAP 1922555](https://launchpad.support.sap.com/#/notes/1922555)
* [Nota de soporte técnico de SAP 1956005](https://launchpad.support.sap.com/#/notes/1956005)


## <a name="useful-links-notes--whitepapers-for-sap-ase"></a>Notas, vínculos útiles, y notas del producto de SAP ASE
La página de inicio de la [documentación de SAP ASE 16.0.03.07](https://help.sap.com/viewer/product/SAP_ASE/16.0.3.7/en-US) proporciona vínculos a varios documentos de:

- Recorrido de aprendizaje de SAP ASE: administración y supervisión
- Recorrido de aprendizaje de SAP ASE: instalación y actualización

que resultan útiles. Otro documento útil es [SAP Applications on SAP Adaptive Server Enterprise Best Practices for Migration and Runtime](https://assets.cdn.sap.com/sapcom/docs/2016/06/26450353-767c-0010-82c7-eda71af511fa.pdf).

Otras notas de soporte técnico de SAP útiles son:

- [Nota de soporte técnico de SAP 2134316](https://launchpad.support.sap.com/#/notes/2134316) 
- [Nota de soporte técnico de SAP 1748888](https://launchpad.support.sap.com/#/notes/1748888) 
- [Nota de soporte técnico de SAP 2588660](https://launchpad.support.sap.com/#/notes/2588660) 
- [Nota de soporte técnico de SAP 1680803](https://launchpad.support.sap.com/#/notes/1680803) 
- [Nota de soporte técnico de SAP 1724091](https://launchpad.support.sap.com/#/notes/1724091) 
- [Nota de soporte técnico de SAP 1775764](https://launchpad.support.sap.com/#/notes/1775764) 
- [Nota de soporte técnico de SAP 2162183](https://launchpad.support.sap.com/#/notes/2162183) 
- [Nota de soporte técnico de SAP 1928533](https://launchpad.support.sap.com/#/notes/1928533)
- [Nota de soporte técnico de SAP 2015553](https://launchpad.support.sap.com/#/notes/2015553)
- [Nota de soporte técnico de SAP 1750510](https://launchpad.support.sap.com/#/notes/1750510) 
- [Nota de soporte técnico de SAP 1752266](https://launchpad.support.sap.com/#/notes/1752266) 
- [Nota de soporte técnico de SAP 2162183](https://launchpad.support.sap.com/#/notes/2162183) 
- [Nota de soporte técnico de SAP 1588316](https://launchpad.support.sap.com/#/notes/158831) 

Otra información se publica en: 

- [SAP Applications on SAP Adaptive Server Enterprise](https://community.sap.com/topics/applications-on-ase)
- [Infocenter de SAP ASE](http://infocenter.sybase.com/help/index.jsp) 
- [SAP ASE Always-on con configuración de tercer nodo de recuperación ante desastres](https://techcommunity.microsoft.com/t5/running-sap-applications-on-the/installation-procedure-for-sybase-16-3-patch-level-3-always-on/ba-p/368199)

Un boletín mensual se publica a través de la [nota de soporte técnico de SAP 2381575](https://launchpad.support.sap.com/#/notes/2381575). 


## <a name="next-steps"></a>Pasos siguientes
Consulte el artículo [Lista de comprobación de planeamiento e implementación de cargas de trabajo de SAP en Azure](./sap-deployment-checklist.md).
