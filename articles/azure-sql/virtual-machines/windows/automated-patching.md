---
title: Aplicación de revisión automatizada para VM con SQL Server (Resource Manager) | Microsoft Docs
description: En este artículo se explica la característica Aplicación de revisión automatizada para máquinas virtuales de SQL Server que se ejecutan en Azure mediante Resource Manager.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
editor: ''
tags: azure-resource-manager
ms.assetid: 58232e92-318f-456b-8f0a-2201a541e08d
ms.service: virtual-machines-sql
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 03/07/2018
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 822d74c6f080a1bb1e5e5af38bdf2c221af16d52
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87086768"
---
# <a name="automated-patching-for-sql-server-on-azure-virtual-machines-resource-manager"></a>Aplicación de revisión automatizada para SQL Server en Azure Virtual Machines (Resource Manager)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

La aplicación de revisión automatizada establece una ventana de mantenimiento para máquinas virtuales de Azure con SQL Server. Actualizaciones automatizadas solo puede instalarse durante esta ventana de mantenimiento. Para SQL Server, esta restricción garantiza que se actualice el sistema y que cualquier reinicio asociado se produzca en el mejor momento posible para la base de datos. 

> [!IMPORTANT]
> Solo se instalan las actualizaciones de Windows y SQL Server marcadas como **Importantes** o **Críticas**. Otras actualizaciones de SQL Server, como los paquetes de servicio y las actualizaciones acumulativas que no están marcadas como **Importantes** o **Críticas** se deben instalar de forma manual. 

La aplicación de revisión automatizada depende de la [extensión del agente de infraestructura como servicio (IaaS) de SQL Server](sql-server-iaas-agent-extension-automate-management.md).

## <a name="prerequisites"></a>Requisitos previos
Para utilizar Aplicación de revisión automatizada, tenga en cuenta los siguientes requisitos previos:

**Sistema operativo**:

* Windows Server 2008 R2
* Windows Server 2012
* Windows Server 2012 R2
* Windows Server 2016

**Versión de SQL Server**:

* SQL Server 2008 R2
* SQL Server 2012
* SQL Server 2014
* SQL Server 2016
* SQL Server 2017

**Azure PowerShell**:

* [Instale los comandos de Azure PowerShell más recientes](/powershell/azure/) si planea configurar Aplicación de revisión automatizada con PowerShell.

[!INCLUDE [updated-for-az.md](../../../../includes/updated-for-az.md)]

> [!NOTE]
> Aplicación de revisión automatizada se basa en la Extensión Agente de IaaS de SQL Server. Las imágenes actuales de la galería de máquinas virtuales de SQL agregan esta extensión de manera predeterminada. Para más información, consulte la [extensión Agente de IaaS de SQL Server](sql-server-iaas-agent-extension-automate-management.md).
> 
> 

## <a name="settings"></a>Configuración
En la siguiente tabla se describen las opciones que pueden configurarse para Aplicación de revisión automatizada. Los pasos de configuración reales varían si usa Azure Portal o comandos de Windows PowerShell de Azure.

| Configuración | Valores posibles | Descripción |
| --- | --- | --- |
| **Aplicación de revisiones automatizada** |Habilitar/deshabilitar (deshabilitado) |Habilita o deshabilita Aplicación de revisión automatizada para una máquina virtual de Azure. |
| **Programación de mantenimiento** |Cada día, el lunes, el martes, el miércoles, el jueves, el viernes, el sábado, el domingo |La programación para descargar e instalar actualizaciones de Windows, SQL Server y Microsoft para la máquina virtual. |
| **Hora de inicio de mantenimiento** |0-24 |La hora de inicio local para actualizar la máquina virtual. |
| **Duración de la ventana de mantenimiento** |30-180 |El número de minutos permitido para completar la descarga y la instalación de actualizaciones. |
| **Categoría de la revisión** |Importante | La categoría de las actualizaciones de Windows para descargar e instalar.|

## <a name="configure-in-the-azure-portal"></a>Configuración en Azure Portal
Puede usar el Portal de Azure para configurar Aplicación de revisión automatizada durante el aprovisionamiento o para las máquinas virtuales existentes.

### <a name="new-vms"></a>Nuevas máquinas virtuales
Use Azure Portal para configurar la característica Aplicación de revisión automatizada cuando cree una máquina virtual de SQL Server en el modelo de implementación de Resource Manager.

En la pestaña **Configuración de SQL Server**, seleccione **Cambiar configuración** en **Aplicación de revisión automatizada**. La siguiente captura de pantalla del Portal de Azure muestra la hoja **Aplicación de revisión automatizada de SQL** .

![Aplicación de revisión automatizada de SQL en Azure Portal](./media/automated-patching/azure-sql-arm-patching.png)

Para más información, consulte [Aprovisionamiento de una máquina virtual de SQL Server en Azure](create-sql-vm-portal.md).

### <a name="existing-vms"></a>Máquinas virtuales existentes

[!INCLUDE [windows-virtual-machines-sql-use-new-management-blade](../../../../includes/windows-virtual-machines-sql-new-resource.md)]

En el caso de las máquinas virtuales de SQL Server existentes, abra su [recurso de máquinas virtuales SQL](manage-sql-vm-portal.md#access-the-sql-virtual-machines-resource) y seleccione **Aplicación de revisión** en **Configuración**. 

![Aplicación de revisión automatizada de SQL para máquinas virtuales existentes](./media/automated-patching/azure-sql-rm-patching-existing-vms.png)


Cuando termine, haga clic en el botón **Aceptar** situado en la parte inferior de la hoja **Configuración de SQL Server** para guardar los cambios.

Si habilita Aplicación de revisión automatizada por primera vez, Azure configura el agente de IaaS de SQL Server en segundo plano. Durante este tiempo, es posible que el Portal de Azure no muestre que se ha configurado Aplicación de revisión automatizada. Espere unos minutos hasta que el agente se instale y configure. Después, el Portal de Azure muestra la nueva configuración.

## <a name="configure-with-powershell"></a>Configuración con PowerShell
Después de aprovisionar la máquina virtual de SQL, use PowerShell para configurar Aplicación de revisión automatizada.

En el ejemplo siguiente, se usa PowerShell para configurar Aplicación de revisión automatizada en una máquina virtual de SQL Server existente. El comando **New-AzVMSqlServerAutoPatchingConfig** configura una nueva ventana de mantenimiento para actualizaciones automáticas.

```azurepowershell
$vmname = "vmname"
$resourcegroupname = "resourcegroupname"
$aps = New-AzVMSqlServerAutoPatchingConfig -Enable -DayOfWeek "Thursday" -MaintenanceWindowStartingHour 11 -MaintenanceWindowDuration 120  -PatchCategory "Important"
s
Set-AzVMSqlServerExtension -AutoPatchingSettings $aps -VMName $vmname -ResourceGroupName $resourcegroupname
```

> [!IMPORTANT]
> Si la extensión no está ya instalada, al hacerlo se reinicia SQL Server.

Según este ejemplo, la siguiente tabla describe el efecto práctico en la máquina virtual de Azure de destino:

| Parámetro | Efecto |
| --- | --- |
| **DayOfWeek** |Las revisiones instaladas cada jueves. |
| **MaintenanceWindowStartingHour** |Inicia las actualizaciones a las 11:00 a.m. |
| **MaintenanceWindowsDuration** |Las revisiones deben instalarse en un plazo de 120 minutos. Según la hora de inicio, deben haberse completado a las 1:00 p.m. |
| **PatchCategory** |La única configuración posible para este parámetro es **Importante**. De este modo, se instalan las actualizaciones de Windows marcadas como importantes; no se instala ninguna actualización de SQL Server que no se incluya en esta categoría. |

La instalación y configuración del agente de Iaas de SQL Server puede tardar algunos minutos.

Para deshabilitar Aplicación de revisión automatizada, ejecute el mismo script sin el parámetro **-Enable** en **New-AzVMSqlServerAutoPatchingConfig**. La ausencia del parámetro **-Enable** indica al comando que deshabilite la característica.

## <a name="next-steps"></a>Pasos siguientes
Para más información acerca de otras tareas de automatización disponibles, consulte la [extensión Agente de IaaS de SQL Server](sql-server-iaas-agent-extension-automate-management.md).

Para más información sobre cómo ejecutar SQL Server en máquinas virtuales de Azure, consulte [Introducción a SQL Server en Azure máquinas virtuales de Azure](sql-server-on-azure-vm-iaas-what-is-overview.md).

