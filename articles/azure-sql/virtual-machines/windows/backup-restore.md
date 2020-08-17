---
title: Copia de seguridad y restauración de SQL Server en VM de Azure | Microsoft Docs
description: Describe las consideraciones de copia de seguridad y restauración de bases de datos de SQL Server que se ejecutan en Azure Virtual Machines.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
editor: ''
tags: azure-resource-management
ms.assetid: 95a89072-0edf-49b5-88ed-584891c0e066
ms.service: virtual-machines-sql
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/04/2018
ms.author: mikeray
ms.openlocfilehash: 6a03a91eeb9296e60aa147f97634a15e8d344209
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87293043"
---
# <a name="backup-and-restore-for-sql-server-on-azure-vms"></a>Copia de seguridad y restauración de SQL Server en VM de Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se proporciona orientación acerca de las opciones de copia de seguridad y restauración para SQL Server que se ejecuta en una máquina virtual (VM) de Windows en Azure. Azure Storage mantiene tres copias de cada disco de máquina virtual de Azure para garantizar la protección contra la pérdida de datos o los daños de los datos físicos. Por lo tanto, a diferencia de lo que ocurriría en un entorno local de SQL Server, no tiene que centrarse en los errores de hardware. Sin embargo, debe seguir haciendo una copia de seguridad de las bases de datos de SQL Server para protegerse frente a errores de la aplicación o un usuario, como las inserciones o eliminaciones accidentales de datos. En esta situación, es importante que pueda restaurar a un momento concreto en el tiempo.

En la primera parte de este artículo se proporciona información general de las opciones de copia de seguridad y restauración disponibles. A continuación se presentan secciones que proporcionan más información acerca de cada estrategia.

## <a name="backup-and-restore-options"></a>Opciones de copia de seguridad y restauración

En la tabla siguiente se proporciona información sobre varias opciones de copia de seguridad y restauración para SQL Server en VM de Azure:

| Estrategia | Versiones de SQL | Descripción |
|---|---|---|
| [Automated Backup](#automated) | 2014<br/> 2016<br/> 2017 | Copia de seguridad automatizada le permite programar copias de seguridad periódicas para todas las bases de datos en una VM de SQL Server. Las copias de seguridad se guardan en el almacenamiento de Azure durante 30 días. A partir de SQL Server 2016, la versión 2 de Copia de seguridad automatizada ofrece opciones adicionales, como la configuración de programación manual y la frecuencia de las copias de seguridad completas y de registros. |
| [Azure Backup para VM de SQL](#azbackup) | 2008<br/> 2012<br/> 2014<br/> 2016<br/> 2017 | Azure Backup proporciona una funcionalidad de copia de seguridad de clase empresarial para SQL Server en VM de Azure. Con este servicio, es posible administrar centralmente las copias de seguridad para varios servidores y miles de bases de datos. Las bases de datos se pueden restaurar a un momento concreto en el tiempo en el portal. Ofrece una directiva de retención personalizable que puede mantener las copias de seguridad durante años. |
| [Copia de seguridad manual](#manual) | All | Dependiendo de la versión de SQL Server, hay diversas técnicas para realizar manualmente la copia de seguridad y restauración de SQL Server en una VM de Azure. En este escenario, el usuario es responsable tanto de la manera de copiar las bases de datos como de la ubicación del almacenamiento y la administración de estas copias de seguridad. |

En las siguientes secciones se describe cada opción con más detalle. La sección final de este artículo proporciona un resumen en forma de una matriz de características.

## <a name="automated-backup"></a><a id="automated"></a> Copia de seguridad automatizada

Copia de seguridad automatizada proporciona un servicio de copia de seguridad automático para las ediciones de SQL Server Standard y Enterprise que se ejecutan en una VM de Windows en Azure. Este servicio se proporciona por la [extensión del agente de IaaS de SQL Server](sql-server-iaas-agent-extension-automate-management.md), que se instala automáticamente en las imágenes de máquina virtual de Windows de SQL Server en Azure Portal.

Se hace una copia de seguridad de todas las bases de datos en una cuenta de Azure Storage que configure. Las copias de seguridad pueden cifrarse y conservarse durante 30 días.

Las máquinas con SQL Server 2016 y versiones posteriores ofrecen más opciones de personalización con la versión 2 de Copia de seguridad automatizada. Estas mejoras incluyen:

- Copias de seguridad de bases de datos del sistema
- Programación de copia de seguridad manual y período de tiempo
- Frecuencia de copia de seguridad de archivos de registro y completa

Para restaurar una base de datos, debe buscar los archivos de copia de seguridad necesarios en la cuenta de almacenamiento y realizar una restauración en la VM de SQL mediante SQL Server Management Studio (SSMS) o comandos de Transact-SQL.

Para obtener más información sobre cómo configurar Copia de seguridad automatizada para máquinas virtuales de SQL, consulte uno de los siguientes artículos:

- **SQL Server 2016/2017**: [Automated Backup v2 para Azure Virtual Machines](automated-backup.md)
- **SQL Server 2014**: [Automated Backup para SQL Server 2014 en Azure Virtual Machines](automated-backup-sql-2014.md)

## <a name="azure-backup-for-sql-vms"></a><a id="azbackup"></a>Azure Backup para máquinas virtuales de SQL

[Azure Backup](/azure/backup/) proporciona una funcionalidad de copia de seguridad de clase empresarial para SQL Server en VM de Azure. Todas las copias de seguridad se almacenan y administran en un almacén de Recovery Services. Esta solución ofrece varias ventajas, especialmente para las empresas:

- **Copia de seguridad sin necesidad de infraestructura**: no es necesario administrar servidores de copia de seguridad o ubicaciones de almacenamiento.
- **Escala**: proteja muchas máquinas virtuales de SQL y miles de bases de datos.
- **Pago por uso**: esta funcionalidad es un servicio independiente proporcionado por Azure Backup, pero al igual que en todos los servicios de Azure, solo paga por lo que usa.
- **Supervisión y administración central**: administre centralmente todas las copias de seguridad, incluidas otras cargas de trabajo que admite Azure Backup, desde un único panel en Azure.
- **Retención y copia de seguridad basadas en directivas**: cree directivas de copia de seguridad estándar para copias de seguridad periódicas. Establezca directivas de retención para mantener copias de seguridad durante años.
- **Compatibilidad con SQL AlwaysOn**: detecte y proteja la configuración de SQL Server Always On y respete la preferencia de copia de seguridad del grupo de disponibilidad.
- **Objetivo de punto de recuperación (RPO) de 15 minutos**: configure copias de seguridad del registro de transacciones de SQL con una frecuencia de hasta 15 minutos.
- **Restauración a un momento dado**: use el portal para recuperar las bases de datos a un momento concreto en el tiempo sin tener que restaurar manualmente varias copias de seguridad completas, diferenciales y registros.
- **Alertas de errores por correo electrónico consolidadas**: configure notificaciones por correo electrónico consolidadas para cualquier error.
- **Control de acceso basado en rol**: determine quién puede administrar operaciones de copia de seguridad y restauración mediante el portal.

Para obtener una introducción rápida de cómo funciona junto con una demostración, vea el vídeo siguiente:

> [!VIDEO https://www.youtube.com/embed/wmbANpHos_E]

La solución de Azure Backup para máquinas virtuales de SQL se encuentra disponible con carácter general. Para obtener más información, consulte [Back up SQL Server database to Azure](../../../backup/backup-azure-sql-database.md) (Copia de seguridad de base de datos de SQL Server en Azure).

## <a name="manual-backup"></a><a id="manual"></a> Copia de seguridad manual

Si desea administrar manualmente operaciones de copia de seguridad y restauración en sus VM de SQL, tiene a su disposición varias opciones, en función de la versión de SQL Server que utilice. Para obtener información general de copia de seguridad y restauración, consulte uno de los siguientes artículos según la versión de SQL Server:

- [Copia de seguridad y restauración para SQL Server 2016 y versiones posteriores](https://docs.microsoft.com/sql/relational-databases/backup-restore/back-up-and-restore-of-sql-server-databases)
- [Copia de seguridad y restauración para SQL Server 2014](https://msdn.microsoft.com/library/ms187048%28v=sql.120%29.aspx)
- [Copia de seguridad y restauración para SQL Server 2012](https://msdn.microsoft.com/library/ms187048%28v=sql.110%29.aspx)
- [Copia de seguridad y restauración para SQL Server 2008 R2](https://msdn.microsoft.com/library/ms187048%28v=sql.105%29.aspx)
- [Copia de seguridad y restauración para SQL Server 2008](https://msdn.microsoft.com/library/ms187048%28v=sql.100%29.aspx)

Las siguientes secciones describen varias opciones manuales de copia de seguridad y restauración opciones con más detalle.

### <a name="backup-to-attached-disks"></a>Copia de seguridad en discos conectados

Para SQL Server que se ejecuta en VM de Azure, puede usar técnicas de copia de seguridad y restauración nativas mediante discos conectados en la VM para el destino de los archivos de copia de seguridad. No obstante, hay un límite en el número de discos que se pueden acoplar a una máquina virtual de Azure según el [tamaño de la máquina virtual](../../../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). Existe también una sobrecarga de la administración de discos que tener en cuenta.

Para obtener un ejemplo de cómo crear manualmente una copia de seguridad completa de la base de datos mediante SQL Server Management Studio (SSMS) o Transact-SQL, consulte [Crear una copia de seguridad completa de base de datos (SQL Server)](https://docs.microsoft.com/sql/relational-databases/backup-restore/create-a-full-database-backup-sql-server).

### <a name="backup-to-url"></a>Copia de seguridad en dirección URL

A partir de SQL Server 2012 SP1 CU2, puede realizar copias y restaurar directamente en Microsoft Azure Blob Storage, lo cual se conoce también como copia de seguridad en URL. SQL Server 2016 también incluye las siguientes mejoras para esta característica:

| Mejoras de la versión de 2016 | Detalles |
| --- | --- |
| **Seccionamiento** |Cuando realiza una copia de seguridad de Almacenamiento de blobs de Microsoft Azure, SQL Server 2016 admite la copia de seguridad de varios blobs para habilitar la copia de seguridad de bases de datos de gran tamaño, hasta un máximo de 12,8 TB. |
| **Copia de seguridad de instantáneas** |Mediante el uso de instantáneas de Azure, Copia de seguridad de instantáneas de archivos de SQL Server ofrece copias de seguridad prácticamente instantáneas y restauraciones rápidas de los archivos de base de datos almacenados con el servicio de Azure Blob Storage. Esta capacidad le permite simplificar las directivas de copia de seguridad y restauración. Copia de seguridad de instantáneas de archivos también es compatible con la restauración a un momento dado. Para obtener más información, vea [Copias de seguridad de instantáneas para archivos de base de datos en Azure](https://docs.microsoft.com/sql/relational-databases/backup-restore/file-snapshot-backups-for-database-files-in-azure). |

Para obtener más información, consulte uno de los siguientes artículos según la versión de SQL Server:

- **SQL Server 2016/2017**: [Copia de seguridad en dirección URL de SQL Server](https://docs.microsoft.com/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service)
- **SQL Server 2014**: [Copia de seguridad en dirección URL de SQL Server 2014](https://msdn.microsoft.com/library/jj919148%28v=sql.120%29.aspx)
- **SQL Server 2012**: [Copia de seguridad en dirección URL de SQL Server 2012](https://msdn.microsoft.com/library/jj919148%28v=sql.110%29.aspx)

### <a name="managed-backup"></a>Copia de seguridad administrada

A partir de SQL Server 2014, Copia de seguridad administrada automatiza la creación de copias de seguridad en Azure Storage. En segundo plano, Copia de seguridad administrada hace uso de la característica de copia de seguridad en dirección URL que se describe en la sección anterior de este artículo. Copia de seguridad administrada también es la característica subyacente que admite el servicio Copia de seguridad automatizada de VM con SQL Server.

A partir de SQL Server 2016, Copia de seguridad administrada tiene opciones adicionales para la programación, la copia de seguridad de base de datos del sistema y la frecuencia de copia de seguridad completa y de registros.

Para obtener más información, consulte uno de los siguientes artículos según la versión de SQL Server:

- [Managed Backup to Microsoft Azure for SQL Server 2016 and later](https://docs.microsoft.com/sql/relational-databases/backup-restore/sql-server-managed-backup-to-microsoft-azure) (Copia de seguridad administrada en Microsoft Azure para SQL Server 2016 y versiones posteriores)
- [Managed Backup to Microsoft Azure for SQL Server 2014](https://msdn.microsoft.com/library/dn449496%28v=sql.120%29.aspx) (Copia de seguridad administrada en Microsoft Azure para SQL Server 2014)

## <a name="decision-matrix"></a>Matriz de decisión

En la tabla siguiente se resumen las capacidades de cada opción de copia de seguridad y restauración para las máquinas virtuales de SQL Server en Azure.

| Opción | Copia de seguridad automatizada | Azure Backup para SQL | Copia de seguridad manual |
|---|---|---|---|
| Requiere servicio adicional de Azure |   | ![Sí](./media/backup-restore/yes.png) |   |
| Configurar la directiva de copia de seguridad en Azure Portal | ![Sí](./media/backup-restore/yes.png) | ![Sí](./media/backup-restore/yes.png) |   |
| Restaurar bases de datos en Azure Portal |   | ![Sí](./media/backup-restore/yes.png) |   |
| Administrar varios servidores en un panel |   | ![Sí](./media/backup-restore/yes.png) |   |
| Restauración a un momento dado | ![Sí](./media/backup-restore/yes.png) | ![Sí](./media/backup-restore/yes.png) | ![Sí](./media/backup-restore/yes.png) |
| Objetivo de punto de recuperación (RPO) de 15 minutos | ![Sí](./media/backup-restore/yes.png) | ![Sí](./media/backup-restore/yes.png) | ![Sí](./media/backup-restore/yes.png) |
| Directiva de retención de copia de seguridad a corto plazo (días) | ![Sí](./media/backup-restore/yes.png) | ![Sí](./media/backup-restore/yes.png) |   |
| Directiva de retención de copia de seguridad a largo plazo (meses, años) |   | ![Sí](./media/backup-restore/yes.png) |   |
| Compatibilidad integrada para SQL Server Always On |   | ![Sí](./media/backup-restore/yes.png) |   |
| Copia de seguridad en cuentas de Azure Storage | ![Sí](./media/backup-restore/yes.png)(automático) | ![Sí](./media/backup-restore/yes.png)(automático) | ![Sí](./media/backup-restore/yes.png)(administrado por el cliente) |
| Administración de archivos de copia de seguridad y almacenamiento | | ![Sí](./media/backup-restore/yes.png) |  |
| Copia de seguridad en discos conectados en la VM |   |   | ![Sí](./media/backup-restore/yes.png) |
| Informes de copia de seguridad personalizables centrales |   | ![Sí](./media/backup-restore/yes.png) |   |
| Alertas por correo electrónico consolidadas de errores |   | ![Sí](./media/backup-restore/yes.png) |   |
| Personalización de supervisión basada en registros de Azure Monitor |   | ![Sí](./media/backup-restore/yes.png) |   |
| Supervisión de trabajos de copia de seguridad con SSMS o scripts de Transact-SQL | ![Sí](./media/backup-restore/yes.png) | ![Sí](./media/backup-restore/yes.png) | ![Sí](./media/backup-restore/yes.png) |
| Restauración de bases de datos con SSMS o scripts de Transact-SQL | ![Sí](./media/backup-restore/yes.png) |   | ![Sí](./media/backup-restore/yes.png) |

## <a name="next-steps"></a>Pasos siguientes

Si está planeando la implementación de SQL Server en una VM de Azure, puede encontrar directrices de aprovisionamiento en la siguiente guía: [Aprovisionamiento de una máquina virtual Windows con SQL Server en Azure Portal](create-sql-vm-portal.md).

Aunque la copia de seguridad y la restauración pueden usarse para migrar los datos, existen rutas de acceso de migración de datos posiblemente más sencillas para SQL Server en VM. Para obtener una descripción completa de las opciones de migración y recomendaciones, vea [Migración de una base de datos a SQL Server en una VM de Azure](migrate-to-vm-from-sql-server.md).
