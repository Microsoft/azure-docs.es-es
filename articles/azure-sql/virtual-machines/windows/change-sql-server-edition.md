---
title: Cambio local de la edición de SQL Server
description: Aprenda a cambiar la edición de la máquina virtual de SQL Server en Azure por una versión anterior para reducir el costo o actualícela para habilitar más características.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 01/14/2020
ms.author: mathoma
ms.reviewer: jroth
ms.custom: seo-lt-2019
ms.openlocfilehash: 60020667126fe76d33264795ddb2e17e1241a7b2
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88236291"
---
# <a name="in-place-change-of-sql-server-edition-on-azure-vm"></a>Cambio en contexto de la edición de SQL Server de la máquina virtual de Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se describe cómo cambiar la edición de SQL Server en una máquina virtual Windows en Azure. 

La edición de SQL Server viene determinada por la clave de producto y se especifica durante el proceso de instalación mediante el soporte de instalación. La edición determina qué [características](/sql/sql-server/editions-and-components-of-sql-server-2017) están disponibles en SQL Server. Puede cambiar la edición de SQL Server con el soporte de instalación: cambiar a una versión anterior para reducir los costos o actualizar para habilitar más características.

Una vez que se ha cambiado internamente la edición de SQL Server a la máquina virtual con SQL Server, debe actualizar la propiedad de edición de SQL Server en Azure Portal para que se refleje en la facturación. 

## <a name="prerequisites"></a>Requisitos previos

Para hacer un cambio local de la edición de SQL Server, necesita lo siguiente: 

- Una [suscripción de Azure](https://azure.microsoft.com/free/).
- Una [VM con SQL Server en Windows](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-server-provision) registrada en el [proveedor de recursos de máquina virtual de SQL](sql-vm-resource-provider-register.md).
- Soporte de instalación con la **edición deseada** de SQL Server. Los clientes que tengan [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default) pueden obtener el soporte de instalación en el [centro de licencias por volumen](https://www.microsoft.com/Licensing/servicecenter/default.aspx). Los clientes que no tengan Software Assurance pueden usar el soporte de instalación de una imagen de máquina virtual con SQL Server de Azure Marketplace que tenga la edición deseada (ubicada normalmente en `C:\SQLServerFull`). 


## <a name="upgrade-an-edition"></a>Actualización de una edición

> [!WARNING]
> Al actualizar la edición de SQL Server se reiniciará el servicio para SQL Server y los demás servicios asociados, como Analysis Services y R Services. 

Para actualizar la edición de SQL Server, obtenga el soporte de instalación de SQL Server para la edición que desee y haga lo siguiente:

1. Ejecute Setup.exe desde el soporte de instalación de SQL Server. 
1. Vaya a **Mantenimiento** y elija la opción **Actualización de edición**. 

   ![Selección de la opción Actualización de edición de SQL Server](./media/change-sql-server-edition/edition-upgrade.png)

1. Seleccione **Siguiente** hasta llegar a la página **Listo para actualizar la edición** y seleccione **Actualizar**. La ventana de configuración pude dejar de responder durante unos minutos mientras el cambio surte efecto. Una página **Completado** confirmará que la actualización de edición ha finalizado. 

Una vez actualizada la edición de SQL Server, modifique la propiedad de edición de la máquina virtual con SQL Server en Azure Portal. De este modo, se actualizarán los metadatos y la facturación asociados a esta máquina virtual.

## <a name="downgrade-an-edition"></a>Cambio a una edición anterior

Para cambiar a una edición de SQL Server anterior, debe desinstalar el producto totalmente y volver a instalarlo con el soporte de instalación de la edición deseada. 

> [!WARNING]
> La desinstalación de SQL Server puede suponer tiempo de inactividad adicional. 

Para cambiar la edición de SQL Server a una anterior, siga estos pasos:

1. Realice una copia de seguridad de todas las bases de datos, incluidas del sistema. 
1. Traslade las bases de datos del sistema (master, model y msdb) a una nueva ubicación. 
1. Desinstale completamente todos los servicios asociados y SQL Server. 
1. Reinicie la máquina virtual. 
1. Instale SQL Server usando el soporte de instalación con la edición deseada.
1. Instale los últimos Service Pack y las actualizaciones acumulativas.  
1. Reemplace la nuevas bases de datos del sistema que se han creado durante la instalación por las que había trasladado antes a otra ubicación. 

Una vez que haya cambiado a una edición anterior de SQL Server, modifique la propiedad de edición de la máquina virtual con SQL Server en Azure Portal. De este modo, se actualizarán los metadatos y la facturación asociados a esta máquina virtual.

## <a name="change-edition-in-portal"></a>Cambio de la edición en el portal 

Una vez que haya cambiado la edición de SQL Server con el soporte de instalación y haya registrado la máquina virtual con SQL Server en el [proveedor de recursos de máquina virtual con SQL](sql-vm-resource-provider-register.md), puede usar Azure Portal para modificar la propiedad de edición de la máquina virtual con SQL Server para que se refleje en la facturación. Para hacerlo, siga estos pasos: 

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 
1. Vaya al recurso de máquina virtual de SQL Server. 
1. En **Configuración**, seleccione **Configurar**. A continuación, elija la edición de SQL Server que quiera en la lista desplegable **Edición**. 

   ![Metadatos del cambio de edición](./media/change-sql-server-edition/edition-change-in-portal.png)

1. Revise la advertencia que indica que primero debe cambiar la edición de SQL Server y que la propiedad de edición debe coincidir con la edición de SQL Server. 
1. Seleccione **Aplicar** para aplicar los cambios en los metadatos de edición. 


## <a name="remarks"></a>Observaciones

- La propiedad de edición de la VM con SQL Server debe coincidir con la edición de la instancia de SQL Server instalada para todas las máquinas virtuales con SQL Server, incluidos los tipos de licencia Pago por uso y Traiga su propia licencia.
- Si quita el recurso de máquina virtual con SQL Server, volverá a la configuración de edición codificada de forma rígida de la imagen.
- La capacidad de cambiar la edición es una característica del proveedor de recursos de máquina virtual de SQL. Al implementar una imagen de Azure Marketplace desde Azure Portal, se registra automáticamente una VM con SQL Server en el proveedor de recursos. Sin embargo, los clientes que instalen automáticamente SQL Server deberán [registrar su VM con SQL Server](sql-vm-resource-provider-register.md) de forma manual.
- Para agregar una VM con SQL Server a un conjunto de disponibilidad, es necesario volver a crear la máquina virtual. Cualquier máquina virtual que se agregue a un conjunto de disponibilidad volverá a la edición predeterminada y será necesario volver a cambiarla.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los siguientes artículos. 

* [Introducción a SQL Server en máquinas virtuales Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Preguntas más frecuentes de SQL Server en máquinas virtuales Windows](frequently-asked-questions-faq.md)
* [Orientación de precios de SQL Server para máquinas virtuales de Azure](pricing-guidance.md)
* [Notas de la versión de SQL Server en Azure Virtual Machines](doc-changes-updates-release-notes.md)


