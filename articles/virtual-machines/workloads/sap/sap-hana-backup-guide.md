---
title: Guía de copia de seguridad de SAP HANA en Azure Virtual Machines | Microsoft Azure
description: Guía de copia de seguridad para SAP HANA proporciona dos posibilidades de copia de seguridad principales para SAP HANA en Azure Virtual Machines
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: juergent
editor: ''
ms.service: virtual-machines-linux
ms.topic: article
ums.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 03/01/2020
ms.author: juergent
ms.openlocfilehash: b5a83b3976dd3d3af1bfd5695815f7571d73dd9d
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88652192"
---
# <a name="backup-guide-for-sap-hana-on-azure-virtual-machines"></a>Guía de copia de seguridad de SAP HANA en Azure Virtual Machines

## <a name="getting-started"></a>Introducción

La guía de copia de seguridad de SAP HANA en Azure Virtual Machines solo describirá temas específicos de Azure. Para ver elementos generales relacionados con la copia de seguridad de SAP HANA, consulte la documentación de SAP HANA. Esperamos que conozca las estrategias de copia de seguridad de bases de datos, así como las razones y las motivaciones para tener una estrategia de copia de seguridad sólida y válida, y que conozca los requisitos que su empresa tiene para el procedimiento de copia de seguridad, el período de retención de las copias de seguridad y el procedimiento de restauración.

SAP HANA se admite oficialmente en diversos tipos de máquina virtual de Azure, como la serie M de Azure. Para obtener una lista completa de las máquinas virtuales de Azure con certificación SAP HANA y las unidades de HANA (instancias grandes), consulte le página de [búsqueda de plataformas IaaS con certificación](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). Microsoft Azure ofrece varias unidades en las que SAP HANA se ejecuta sin virtualización en servidores físicos. Este servicio se denomina [HANA (instancias grandes)](hana-overview-architecture.md). En esta guía no se tratarán los procesos y las herramientas de copia de seguridad de HANA (instancias grandes), sino que va a estar limitada a las máquinas virtuales de Azure. Para más información acerca de los procesos de copia de seguridad y restauración con HANA (instancias grandes), lea el artículo [Copia de seguridad y restauración de HLI](./hana-backup-restore.md).

Este artículo se centra en las tres posibilidades de copia de seguridad de SAP HANA en las máquinas virtuales de Azure:

- Copia de seguridad de HANA mediante los [servicios de Azure Backup](../../../backup/backup-overview.md) 
- Copia de seguridad de HANA en el sistema de archivos en Azure Linux Virtual Machine (vea [Azure Backup de SAP HANA en el nivel de archivo](sap-hana-backup-file-level.md))
- Copia de seguridad de HANA basada en instantáneas de almacenamiento con la característica de instantáneas de Azure Storage Blob manualmente o el servicio Azure Backup


SAP HANA ofrece una API de copia de seguridad, que permite que las herramientas de copia de seguridad de terceros se integren directamente con SAP HANA. Productos como el servicio Azure Backup o [Commvault](https://azure.microsoft.com/resources/protecting-sap-hana-in-azure/) usan esta interfaz propietaria para desencadenar una base de datos de SAP HANA o copias de seguridad de registros de puesta al día. 


En el artículo [Qué software de SAP es compatible para su implementación en Azure](./sap-supported-product-on-azure.md) se puede encontrar la información necesaria para encontrar el software de SAP que se admite en Azure.

## <a name="azure-backup-service"></a>Servicio Azure Backup

El primer escenario que se muestra es un escenario en el que el servicio Azure Backup usa la interfaz `backint` de SAP HANA para realizar una copia de seguridad en streaming desde una base de datos de SAP HANA. O bien, se usa una funcionalidad más genérica del servicio Azure Backup para crear una instantánea de disco coherente de la aplicación y transferirla al servicio Azure Backup.

Azure Backup integra y tiene la certificación de solución de copia de seguridad para SAP HANA que usa la interfaz propietaria de SAP HANA denominada [backint](https://www.sap.com/dmc/exp/2013_09_adpd/enEN/#/d/solutions?id=8f3fd455-a2d7-4086-aa28-51d8870acaa5). Para más detalles de la solución, sus funcionalidades y las regiones de Azure en que está disponible, lea el artículo [Matriz de compatibilidad para la copia de seguridad de bases de datos de SAP HANA en máquinas virtuales de Azure](../../../backup/sap-hana-backup-support-matrix.md#scenario-support). Para conocer los detalles y principios del servicio Azure Backup para HANA, lea el artículo [Acerca de la copia de seguridad de bases de datos SAP HANA en máquinas virtuales de Azure](../../../backup/sap-hana-db-about.md). 

La segunda posibilidad para aprovechar el servicio Azure Backup es crear una copia de seguridad coherente con la aplicación mediante instantáneas de disco de Azure Premium Storage. Otras instancias de Azure Storage con certificación de HANA, como [Azure Ultra Disk](../../disks-enable-ultra-ssd.md) y [Azure NetApp Files](https://azure.microsoft.com/services/netapp/) no admiten este tipo de instantánea a través de un servicio de Azure Backup. Al leer estos artículos:

- [Planeación de la infraestructura de copia de seguridad de VM en Azure](../../../backup/backup-azure-vms-introduction.md)
- [Copias de seguridad coherentes con la aplicación de las máquinas virtuales Linux de Azure](../../../backup/backup-azure-linux-app-consistent.md) 

emerge esta secuencia de actividad:

- Azure Backup necesita ejecutar un script previo a la instantánea que coloque la aplicación, en este caso SAP HANA, en un estado coherente
- Cuando se confirme este estado coherente, Azure Backup ejecutará las instantáneas del disco
- Después de finalizar las instantáneas, Azure Backup deshará la actividad que realizó en el script previo a la instantánea
- Después de la ejecución correcta, Azure Backup hará streaming de los datos en el almacén de Backup

En el caso de SAP HANA, la mayoría de los clientes usan Acelerador de escritura de Azure para los volúmenes que contienen el registro de puestas al día de SAP HANA. El servicio Azure Backup excluirá automáticamente estos volúmenes de las instantáneas. Esta exclusión no perjudica la capacidad de restauración de HANA. No obstante, bloquearía la capacidad de realizar la restauración con casi todos los restantes DBMS compatibles con SAP.

El inconveniente de esta posibilidad es el hecho de que es preciso desarrollar un script anterior y posterior a la instantánea propio. El script anterior a la instantánea debe crear una instantánea de HANA y controlar los casos de excepción eventuales. Mientras que el script posterior a la instantánea debe volver a eliminar la instantánea de HANA. Para más información sobre la lógica necesaria, comience por la [nota de soporte de SAP #2039883](https://launchpad.support.sap.com/#/notes/2039883). Las consideraciones de la sección "Consistencia de datos de SAP HANA al realizar instantáneas del almacenamiento" de este artículo se aplican totalmente a este tipo de copia de seguridad.

> [!NOTE]
> Las copias de seguridad basadas en instantáneas de disco para SAP HANA en implementaciones en las que se usan varios contenedores de base de datos requieren una versión mínima de HANA 2.0 SP04
> 

En este mismo documento verá más información acerca de las instantáneas del almacenamiento.

![Esta ilustración muestra dos posibilidades para guardar el estado actual de la VM](media/sap-hana-backup-guide/azure-backup-service-for-hana.png)

## <a name="other-hana-backup-methods"></a>Otros métodos de copia de seguridad de HANA
Hay otros tres métodos de copia de seguridad o rutas de acceso que se pueden considerar:

- Realizar copias de seguridad con un recurso compartido de NFS basado en Azure NetApp Files (ANF). Una vez más, ANF tiene la capacidad de crear instantáneas de los volúmenes en los que se almacenan las copias de seguridad. Dado el rendimiento que se requiere para escribir las copias de seguridad, esta solución podría llegar a ser un método caro. No obstante, es fácil de establecer, ya que HANA puede escribir las copias de seguridad directamente en el recurso compartido de NFS nativo de Azure
- La ejecución de la copia de seguridad de HANA con discos conectados a una máquina virtual de SSD estándar o Azure Premium Storage. En el paso siguiente, puede copiar los archivos de copia de seguridad con Azure Blob Storage. Esta estrategia puede ser económicamente atractiva
- La ejecución de la copia de seguridad de HANA con discos conectados a una máquina virtual de SSD estándar o Azure Premium Storage. Como paso siguiente, se toma una instantánea del disco periódicamente. Después de la primera instantánea, se pueden usar instantáneas incrementales para reducir los costos

![Esta ilustración muestra opciones para realizar una copia de seguridad de archivos de SAP HANA dentro de la VM](media/sap-hana-backup-guide/other-hana-backup-paths.png)

Esta ilustración muestra opciones para realizar una copia de seguridad de archivos de SAP HANA dentro de la VM y luego almacenar los archivos de copia de seguridad de HANA en otro lugar mediante herramientas distintas. Sin embargo, todas las soluciones que no impliquen un servicio de copia de seguridad de terceros o un servicio de Azure Backup tienen varios problemas en común. Algunos de ellos se pueden enumerar, como la administración de retenciones, el proceso de restauración automática y proporcionar una recuperación automática a un momento dado, como proporcionan el servicio Azure Backup, u otros servicios y conjuntos de copia de seguridad y servicios de terceros especializados. Muchos de estos servicios de terceros pueden ejecutarse en Azure. 


## <a name="sap-resources-for-hana-backup"></a>Recursos SAP para copia de seguridad de HANA

### <a name="sap-hana-backup-documentation"></a>Documentación de copia de seguridad de SAP HANA

- [Introducción a la administración de SAP HANA](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.00/en-US)
- [Planeación de su estrategia de copia de seguridad y recuperación](https://help.sap.com/saphelp_hanaplatform/helpdata/en/ef/085cd5949c40b788bba8fd3c65743e/content.htm)
- [Programación de la copia de seguridad de HANA mediante ABAP DBACOCKPIT](https://www.hanatutorials.com/p/schedule-hana-backup-using-abap.html)
- [Programación de copias de seguridad de datos (SAP HANA Cockpit)](https://help.sap.com/saphelp_hanaplatform/helpdata/en/6d/385fa14ef64a6bab2c97a3d3e40292/frameset.htm)
- Preguntas más frecuentes sobre la copia de seguridad de SAP HANA en la [nota 1642148 de SAP](https://launchpad.support.sap.com/#/notes/1642148)
- Preguntas más frecuentes sobre las instantáneas de base de datos y almacenamiento en la [nota 2039883 de SAP](https://launchpad.support.sap.com/#/notes/2039883)
- Sistemas de archivos de red no apropiados para la copia de seguridad y la recuperación en la [nota 1820529 de SAP](https://launchpad.support.sap.com/#/notes/1820529)

### <a name="how-to-verify-correctness-of-sap-hana-backup"></a>Cómo comprobar la precisión de una copia de SAP HANA
Independientemente del método de copia de seguridad, la ejecución de una restauración de prueba en otro sistema es una necesidad absoluta. Este enfoque proporciona una manera de asegurarse de que una copia de seguridad es correcta y de que los procesos internos de copia de seguridad y restauración funcionan según lo esperado. Aunque la restauración de copias de seguridad puede ser un obstáculo a nivel local debido a su requisito de infraestructura, es mucho más fácil superarlo en la nube proporcionando temporalmente los recursos necesarios para este fin. Es correcto que algunas de las herramientas que se proporcionan con HANA pueden comprobar la capacidad de restauración de los archivos de copia de seguridad. Sin embargo, el objetivo de los frecuentes ejercicios de restauración es probar el proceso de la restauración de una base de datos y entrenar dicho proceso con el personal de operaciones.

Tenga en cuenta que realizar una restauración simple y comprobar si HANA está listo para empezar no es suficiente. Debe ejecutar una comprobación de la coherencia de las tablas para asegurarse de que la base de datos restaurada está bien. SAP HANA ofrece varios tipos de comprobaciones de coherencia que se describen en la [nota 1977584 de SAP](https://launchpad.support.sap.com/#/notes/1977584).

También podrá encontrar información sobre la comprobación de coherencia de la tabla en el sitio web de SAP en [Comprobaciones de coherencia de tabla y catálogo](https://help.sap.com/saphelp_hanaplatform/helpdata/en/25/84ec2e324d44529edc8221956359ea/content.htm#loio9357bf52c7324bee9567dca417ad9f8b).

### <a name="pros-and-cons-of-hana-backup-versus-storage-snapshot"></a>Ventajas y desventajas de la copia de seguridad de HANA frente a instantáneas de almacenamiento

SAP no da preferencia a las copias de seguridad de HANA frente a instantáneas de almacenamiento. Enumera sus ventajas y desventajas, por lo que puede determinar cuál usar según la situación y la tecnología de almacenamiento disponible (vea [Planeación de su estrategia de copia de seguridad y recuperación](https://help.sap.com/saphelp_hanaplatform/helpdata/en/ef/085cd5949c40b788bba8fd3c65743e/content.htm)).

En Azure, tenga en cuenta el hecho de que la característica de instantáneas de blobs de Azure no proporciona coherencia del sistema de archivos en varios discos (consulte el artículo acerca del [uso de instantáneas de blob con PowerShell](/archive/blogs/cie/using-blob-snapshots-with-powershell)). 

Además, es necesario que comprenda las implicaciones que tiene para la facturación trabajar con frecuencia con instantáneas de blobs como se describe en este artículo: [Comprender cómo las instantáneas incrementan los costos](/rest/api/storageservices/understanding-how-snapshots-accrue-charges) no es tan obvio como cuando se usan discos virtuales de Azure.

### <a name="sap-hana-data-consistency-when-taking-storage-snapshots"></a>Coherencia de datos de SAP HANA al tomar instantáneas de almacenamiento

Como ya se ha documentado, al realizar instantáneas del almacenamiento es obligatorio describir las funcionalidades de la copia de seguridad de instantáneas de Azure Backup. La manera más fácil de evitar problemas sería apagar SAP HANA, o quizás incluso la máquina virtual completa. Esto es algo que no es factible en una instancia de producción.

> [!NOTE]
> Las copias de seguridad basadas en instantáneas de disco para SAP HANA en implementaciones en las que se usan varios contenedores de base de datos requieren una versión mínima de HANA 2.0 SP04
> 

Azure Storage no proporciona coherencia del sistema de archivos si hay varios discos o volúmenes conectados a una máquina virtual durante el proceso de instantánea, lo que significa que la coherencia de la aplicación durante la instantánea debe ofrecerla la misma aplicación, en este caso la propia SAP HANA. En la [nota 2039883 de SAP](https://launchpad.support.sap.com/#/notes/2039883) puede encontrar información importante sobre las copias de seguridad de SAP HANA mediante instantáneas de almacenamiento. Por ejemplo, con los sistemas de archivos XFS, es preciso ejecutar **xfs\_freeze** antes de iniciar una instantánea de almacenamiento para proporcionar coherencia de la aplicación (consulte [xfs\_freeze(8) - Linux man page](https://linux.die.net/man/8/xfs_freeze) para más información sobre **xfs\_freeze**).

Suponiendo que exista un sistema de archivos XFS que abarque cuatro discos virtuales de Azure, los pasos siguientes proporcionan una instantánea coherente que representa el área de datos de HANA:

1. Cree la preparación de una instantánea de datos de HANA
1. Inmovilice los sistemas de archivos de todos los discos y volúmenes (por ejemplo, utilice **xfs\_freeze**)
1. Creación de todas las instantáneas de blob necesarias en Azure
1. Cancelación de la inmovilización del sistema de archivos
1. Confirme la instantánea de datos de HANA (se eliminará la instantánea)

Si se usa la funcionalidad de Azure Backup para realizar copias de seguridad de instantáneas coherentes con la aplicación, debe codificar o incluir en scripts los pasos #1 para el script anterior a la instantánea. El servicio Azure Backup ejecutará los pasos #2 y #3. Los pasos #4 y #5 también se deben proporcionar mediante el código en el script posterior a la instantánea. Si no usa el servicio Azure Backup, también necesitará código o incluir en un script los pasos #2 y #3 por su cuenta.
En estos artículos puede encontrar más información sobre la creación de instantáneas de datos de HANA:

- [Instantáneas de datos de HANA](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.04/en-US/ac114d4b34d542b99bc390b34f8ef375.html
- En el artículo en el que se explica cómo [crear una instantánea de datos (nativa de SQL)](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.04/en-US/9fd1c8bb3b60455caa93b7491ae6d830.html), se puede encontrar más información para ejecutar el paso #1 
- En el artículo en el que se explica cómo [crear una instantánea de datos (nativa de SQL)](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.04/en-US/9fd1c8bb3b60455caa93b7491ae6d830.html), se puede encontrar información necesaria para confirmar o eliminar instantáneas de datos de HANA, tal como se necesita en el paso #5 

Es importante confirmar la instantánea de HANA. Debido a la &quot;copia en escritura&quot;, SAP HANA no debe requerir espacio en disco adicional mientras esté en este modo de preparación de la instantánea. Además, no es posible iniciar nuevas copias de seguridad hasta que se confirme la instantánea de SAP HANA.


### <a name="sap-hana-backup-scheduling-strategy"></a>Estrategia de programación de copia de seguridad de SAP HANA

El artículo de SAP HANA en el que se explica cómo [planear una estrategia de copia de seguridad y recuperación](https://help.sap.com/saphelp_hanaplatform/helpdata/en/ef/085cd5949c40b788bba8fd3c65743e/content.htm) indica un plan básico para realizar copias de seguridad. Utilice la documentación de SAP acerca de HANA, así como su experiencia con otras DBMS al definir la estrategia y proceso de copia de seguridad y restauración de SAP HANA. Tanto la secuencia de los diferentes tipos de copias de seguridad como el período de retención dependen en gran medida de los Acuerdos de Nivel de Servicio que necesite aportar.


### <a name="sap-hana-backup-encryption"></a>Cifrado de copia de seguridad de SAP HANA

SAP HANA ofrece cifrado de datos y de registro. Si no se cifran el registro y los datos de SAP HANA y de registro, las copias de seguridad no se cifrarán de manera predeterminada. Sin embargo, SAP HANA ofrece un cifrado de copias de seguridad independiente, que se documenta en el documento relativo al [cifrado de copias de seguridad de SAP HANA](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.03/en-US/5f837a57ce5e468d9db21c8683bc84da.html). Si utiliza versiones anteriores de SAP HANA, es posible que tenga que comprobar si el cifrado de copias de seguridad formaba parte de la funcionalidad que se proporcionaba en ellas.  


## <a name="next-steps"></a>Pasos siguientes
* [Azure Backup de SAP HANA en el nivel de archivo](sap-hana-backup-file-level.md) describe la opción de copia de seguridad basada en archivos.
* Para obtener información sobre cómo establecer la alta disponibilidad y planear la recuperación ante desastres de SAP HANA en Azure (instancias grandes), vea [Alta disponibilidad y recuperación ante desastres de SAP HANA en Azure (instancias grandes)](hana-overview-high-availability-disaster-recovery.md).
