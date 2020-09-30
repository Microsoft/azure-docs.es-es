---
title: Administración de máquinas virtuales con SQL Server en Azure mediante Azure Portal | Microsoft Docs
description: Aprenda a acceder al recurso de máquina virtual SQL en Azure Portal para una máquina virtual con SQL Server hospedada en Azure.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 05/13/2019
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 04f502183b8240515ee136ce27d99bd4b5d6277c
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89010178"
---
# <a name="manage-sql-server-vms-in-azure-by-using-the-azure-portal"></a>Administración de máquinas virtuales con SQL Server en Azure mediante Azure Portal
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En [Azure Portal](https://portal.azure.com), el recurso [**Máquinas virtuales SQL**](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.SqlVirtualMachine%2FSqlVirtualMachines) es un servicio de administración independiente para administrar SQL Server en las máquinas virtuales de Azure. Puede usarlo para ver todas las máquinas virtuales con SQL Server simultáneamente y modificar la configuración dedicada a SQL Server: 

![Recurso máquinas virtuales SQL](./media/manage-sql-vm-portal/sql-vm-manage.png)


## <a name="remarks"></a>Observaciones

- Se recomienda usar el recurso [**Máquinas virtuales SQL**](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.SqlVirtualMachine%2FSqlVirtualMachines) para ver y administrar las máquinas virtuales con SQL Server en Azure. Sin embargo, actualmente, el recurso **Máquinas virtuales SQL** no permite administrar máquinas virtuales con SQL Server con [finalización del soporte técnico](sql-server-2008-extend-end-of-support.md). Para administrar la configuración de las máquinas virtuales con SQL Server con finalización del soporte técnico, utilice en su lugar la [pestaña de configuración de SQL Server](#access-the-sql-server-configuration-tab) en desuso. 
- El recurso **Máquinas virtuales SQL** solo está disponible para las máquinas virtuales con SQL Server que se hayan [registrado con el proveedor de recursos de máquina virtual con SQL Server](sql-vm-resource-provider-register.md). 


## <a name="access-the-sql-virtual-machines-resource"></a>Acceso al recurso Máquinas virtuales SQL
Para acceder al recurso **Máquinas virtuales SQL**, haga lo siguiente:

1. Abra [Azure Portal](https://portal.azure.com). 
1. Seleccione **Todos los servicios**. 
1. Escriba **Máquinas virtuales SQL** en el cuadro de búsqueda.
1. (Optional): Seleccione la estrella que hay junto a **Máquinas virtuales SQL** para agregar esta opción al menú **Favoritos**. 
1. Seleccione **Máquinas virtuales SQL**. 

   ![Búsqueda de máquinas virtuales con SQL Server en todos los servicios](./media/manage-sql-vm-portal/sql-vm-search.png)

1. En el portal se enumeran todas las máquinas virtuales con SQL Server disponibles en la suscripción. Seleccione la que desee administrar para iniciar el recurso **Máquinas virtuales SQL**. Utilice el cuadro de búsqueda si la máquina virtual con SQL Server no aparece. 

   ![Todas las máquinas virtuales con SQL Server disponibles](./media/manage-sql-vm-portal/all-sql-vms.png)

   Al seleccionar la máquina virtual con SQL Server, se abrirá el recurso **Máquinas virtuales SQL**: 


   ![Visualización de recurso de máquinas virtuales SQL](./media/manage-sql-vm-portal/sql-vm-resource.png)

> [!TIP]
> El recurso **máquinas virtuales de SQL** es para la configuración de SQL Server dedicada. Seleccione el nombre de la máquina virtual en el campo **Máquina virtual** para ir a los valores específicos de la máquina virtual, no a los exclusivos de SQL Server. 

## <a name="access-the-sql-server-configuration-tab"></a>Acceso a la pestaña de configuración de SQL Server
La pestaña **Configuración de SQL Server** ha quedado en desuso. Actualmente, este es el único método para administrar las máquinas virtuales con SQL Server con [finalización del soporte técnico](sql-server-2008-extend-end-of-support.md) y las máquinas virtuales con SQL Server que no se hayan [registrado con el proveedor del recursos de máquina virtual con SQL Server](sql-vm-resource-provider-register.md).

Para acceder a la pestaña **Configuración de SQL Server** en desuso, debe ir al recurso **Máquinas virtuales**. Siga estos pasos:

1. Abra [Azure Portal](https://portal.azure.com). 
1. Seleccione **Todos los servicios**. 
1. Escriba **máquinas virtuales** en el cuadro de búsqueda.
1. (Optional): Seleccione la estrella que hay junto a **Máquinas virtuales** para agregar esta opción al menú **Favoritos**. 
1. Seleccione **Máquinas virtuales**. 

   ![Buscar máquinas virtuales](./media/manage-sql-vm-portal/vm-search.png)

1. El portal enumera todas las máquinas virtuales en la suscripción. Seleccione la que desee administrar para abrir el recurso **Máquinas virtuales**. Utilice el cuadro de búsqueda si la máquina virtual con SQL Server no aparece. 
1. Seleccione **Configuración de SQL Server** en el panel **Configuración** para administrar la máquina virtual con SQL Server. 

   ![Configuración de SQL Server](./media/manage-sql-vm-portal/sql-vm-configuration.png)

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los siguientes artículos. 

* [Introducción a SQL Server en máquinas virtuales Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Preguntas más frecuentes de SQL Server en máquinas virtuales Windows](frequently-asked-questions-faq.md)
* [Orientación de precios de SQL Server para máquinas virtuales de Azure](pricing-guidance.md)
* [Notas de la versión de SQL Server en Azure Virtual Machines](doc-changes-updates-release-notes.md)


