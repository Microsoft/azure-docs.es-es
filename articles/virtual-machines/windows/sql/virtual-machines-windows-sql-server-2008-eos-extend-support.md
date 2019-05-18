---
title: Ampliar la compatibilidad de SQL Server 2008 y SQL Server 2008 R2 con Azure
description: Obtenga información sobre cómo ampliar la compatibilidad de SQL Server 2008 y SQL Server 2008 R2 por migrar su instancia de SQL Server a Azure, ni la compra de soporte extendido para mantener las instancias locales.
services: virtual-machines-windows
documentationcenter: ''
author: MashaMSFT
manager: craigg
tags: azure-service-management
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 04/08/2019
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 398b7a78c9bc13631e15c8725efb8cc0fba3f127
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65806482"
---
# <a name="extend-support-for-sql-server-2008-and-sql-server-2008-r2-with-azure"></a>Ampliar la compatibilidad de SQL Server 2008 y SQL Server 2008 R2 con Azure

SQL Server 2008 y SQL Server 2008 R2 se aproxima el [final de su ciclo de vida de soporte (EOS)](https://www.microsoft.com/sql-server/sql-server-2008). Dado que muchos de nuestros clientes sigue usando las dos versiones, ofrecemos varias opciones para seguir recibiendo soporte técnico. Puede migrar las instancias de SQL Server local a Azure virtual machines (VM), migrar a Azure SQL Database, o mantenerse de forma local y las actualizaciones de seguridad extendida de compra.

A diferencia de con una instancia administrada, migrar a una máquina virtual de Azure no requiere recertifying sus aplicaciones. Y a diferencia de con mantenerse en el entorno local, recibirá las revisiones de seguridad extendida libre mediante la migración a una máquina virtual de Azure. 

El resto de este artículo proporciona consideraciones sobre la migración de la instancia de SQL Server a una máquina virtual de Azure. 

## <a name="provisioning"></a>Aprovisionamiento 

Hay un pago por uso `SQL Server 2008 R2 on Windows Server 2008 R2` imagen disponible en Azure marketplace. 

Los clientes que están en SQL Server 2008 deberán instalar automáticamente, o actualizar a SQL Server 2008 R2. Del mismo modo, los clientes en Windows Server 2008 deberán implementar su máquina virtual desde un disco duro virtual personalizado, o actualizar a Windows Server 2008 R2. 

Las imágenes implementadas a través de Marketplace vienen con la extensión SQL IaaS preinstalada. La extensión IaaS de SQL es un requisito para licencias flexibles y aplicación de revisiones automatizada. Los clientes que implementen máquinas virtuales instaladas automáticamente deberá instalar manualmente la extensión SQL IaaS. La extensión IaaS de SQL no se admite en Windows 2008. 

  > [!NOTE]
  > Mientras que el servidor SQL Server `Create` y `Manage` hojas funcionará con la imagen de SQL Server 2008 R2 en Azure portal, las siguientes características están _no admite_: Copias de seguridad automáticas, integración de Azure Key Vault, R Services y configuración de almacenamiento.

## <a name="licensing"></a>Licencia
Las implementaciones de pago por uso SQL Server 2008 R2 se pueden convertir en [ventaja híbrida de Azure (AHB)](https://azure.microsoft.com/pricing/hybrid-benefit/).

Para convertir una licencia de Software Assurance (SA) basada en pago por uso, los clientes deben registrarse en la VM de SQL [proveedor de recursos](virtual-machines-windows-sql-ahb.md#register-sql-server-vm-with-sql-resource-provider). Una vez registrado con el proveedor de recursos de VM de SQL, el tipo de licencia SQL será intercambiable entre ventaja híbrida de AZURE y de pago por uso. 

Las instancias de SQL Server 2008 o SQL Server 2008 R2 instaladas automáticamente en la máquina virtual de Azure pueden registrar con el proveedor de recursos SQL y convertir su tipo de licencia a pago por uso.

## <a name="migration"></a>Migración
Puede migrar instancias de SQL Server de EOS a una máquina virtual de Azure con métodos de copia de seguridad/restauración manual; Este es el método más común de migración desde local a una máquina virtual de Azure.

### <a name="azure-site-recovery"></a>Azure Site Recovery

Para las migraciones de forma masiva, se recomienda [Azure Site Recovery](/azure/site-recovery/site-recovery-overview) service. Con Azure Site Recovery, los clientes pueden replicar la máquina virtual completa, incluidos SQL Server de un entorno local a una máquina virtual de Azure.

SQL Server requiere que las instantáneas coherentes con la aplicación de Azure Site Recovery para garantizar la recuperación; y Azure Site Recovery es compatible con las instantáneas coherentes con la aplicación con el intervalo mínimo de 1 hora. El RPO mínimo posible para SQL Server con las migraciones de Azure Site Recovery es 1 hora y el RTO es 2 horas más el tiempo de recuperación de SQL Server.

### <a name="database-migration-service"></a>Database Migration Service

El [Database Migration Service](/dms/dms-overview) es una opción para los clientes si la migración desde local a la máquina virtual de Azure mediante la actualización de SQL Server a SQL Server 2012 y versiones posteriores.

## <a name="disaster-recovery"></a>Recuperación ante desastres

Soluciones de recuperación ante desastres para SQL Server en VM de Azure de EOS son los siguientes:

- **Las copias de seguridad de SQL Server**: Las copias de seguridad de SQL Server pueden usarse para recuperar errores de zona o de SQL Server en el caso de configuración regional. Puesto que no se admite la característica de copia de seguridad administrada para EOS SQL Server, los clientes necesitarán realizar copias de seguridad manualmente.
- **Trasvase de registros**: Puede crear una réplica de trasvase de registros en otra zona o región de Azure con restauraciones continua para reducir el RTO. Los clientes deberá configurar manualmente el trasvase de registros.
- **Azure Site Recovery**: Puede replicar la máquina virtual entre zonas y regiones a través de la replicación de Azure Site Recovery. SQL Server necesita instantáneas coherentes de aplicación garantizar la recuperación en caso de desastre. Azure Site Recovery ofrece RPO de 1 hora mínimo y dos horas + tiempo RTO de recuperación de SQL Server para EOS SQL Server recuperación ante desastres.

## <a name="security-patching"></a>Aplicación de revisiones de seguridad
Las actualizaciones de seguridad extendida para las máquinas virtuales de SQL Server se entregarán a través de los canales de Microsoft Update cuando la máquina virtual de SQL Server se ha registrado con el lenguaje SQL [proveedor de recursos](virtual-machines-windows-sql-ahb.md#register-sql-server-vm-with-sql-resource-provider). O bien se pueden descargar las revisiones manual o automáticamente. 

**Automated patching** está habilitada de forma predeterminada. La aplicación de revisiones automatizada permite a Azure aplicar automáticamente las revisiones de SQL Server y del sistema operativo. Puede especificar un día de la semana, hora y duración de una ventana de mantenimiento si está instalada la IaaS extensión de SQL. Azure realiza la aplicación de revisión en esta ventana de mantenimiento. La programación de la ventana de mantenimiento utiliza la configuración regional de la máquina virtual para la hora.  Para más información, consulte [Automated Patching para SQL Server en Azure Virtual Machines](virtual-machines-windows-sql-automated-patching.md).


## <a name="next-steps"></a>Pasos siguientes

Migrar la máquina virtual de SQL Server en Azure

* [Migración de una base de datos SQL Server a SQL Server en una VM de Azure](virtual-machines-windows-migrate-sql.md)

Introducción a SQL Server en máquinas virtuales de Azure:

* [Creación de una máquina virtual con SQL Server en Azure Portal](quickstart-sql-vm-create-portal.md)

Obtenga respuestas a las preguntas más habituales acerca de las máquinas virtuales con SQL:

* [Preguntas más frecuentes sobre SQL Server en Azure Virtual Machines](virtual-machines-windows-sql-server-iaas-faq.md)
