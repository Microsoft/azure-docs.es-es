---
title: Aplicación de revisión automatizada para VM con SQL Server (Resource Manager) | Microsoft Docs
description: Explica la característica Automated Patching para máquinas virtuales de SQL Server que se ejecutan en Azure mediante Resource Manager.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
manager: craigg
editor: ''
tags: azure-resource-manager
ms.assetid: 58232e92-318f-456b-8f0a-2201a541e08d
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 03/07/2018
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 4893076da47528cb6765efc32f46e76819a915b1
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65793811"
---
# <a name="automated-patching-for-sql-server-in-azure-virtual-machines-resource-manager"></a>Automated Patching para SQL Server en Azure Virtual Machines (Resource Manager)
> [!div class="op_single_selector"]
> * [Resource Manager](virtual-machines-windows-sql-automated-patching.md)
> * [Clásico](../sqlclassic/virtual-machines-windows-classic-sql-automated-patching.md)

Aplicación de revisión automatizada establece una ventana de mantenimiento para una máquina virtual de Azure con SQL Server. Actualizaciones automatizadas solo puede instalarse durante esta ventana de mantenimiento. Para SQL Server, esta restricción garantiza que se actualice el sistema y que cualquier reinicio asociado se produzca en el mejor momento posible para la base de datos. 

> [!IMPORTANT]
> Solo se instalan las actualizaciones de Windows marcadas como **importantes**. Otras actualizaciones de SQL Server, como actualizaciones acumulativas, deben instalarse manualmente. 

Aplicación de revisión automatizada se basa en la [Extensión Agente de IaaS de SQL Server](virtual-machines-windows-sql-server-agent-extension.md).

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

* [Instale los comandos de Azure PowerShell más recientes](/powershell/azure/overview) si planea configurar Aplicación de revisión automatizada con PowerShell.

[!INCLUDE [updated-for-az.md](../../../../includes/updated-for-az.md)]

> [!NOTE]
> Aplicación de revisión automatizada se basa en la Extensión Agente de IaaS de SQL Server. Las imágenes actuales de la galería de máquinas virtuales de SQL agregan esta extensión de manera predeterminada. Para más información, consulte la [extensión Agente de IaaS de SQL Server](virtual-machines-windows-sql-server-agent-extension.md).
> 
> 

## <a name="settings"></a>Configuración
En la siguiente tabla se describen las opciones que pueden configurarse para Aplicación de revisión automatizada. Los pasos de configuración reales varían si usa Azure Portal o comandos de Windows PowerShell de Azure.

| Configuración | Valores posibles | DESCRIPCIÓN |
| --- | --- | --- |
| **Aplicación de revisiones automatizada** |Habilitar/deshabilitar (deshabilitado) |Habilita o deshabilita Aplicación de revisión automatizada para una máquina virtual de Azure. |
| **Programación de mantenimiento** |Cada día, el lunes, el martes, el miércoles, el jueves, el viernes, el sábado, el domingo |La programación para descargar e instalar actualizaciones de Windows, SQL Server y Microsoft para la máquina virtual. |
| **Hora de inicio de mantenimiento** |0-24 |La hora de inicio local para actualizar la máquina virtual. |
| **Duración de la ventana de mantenimiento** |30-180 |El número de minutos permitido para completar la descarga y la instalación de actualizaciones. |
| **Categoría de la revisión** |Importante | La categoría de las actualizaciones de Windows para descargar e instalar.|

## <a name="configuration-in-the-portal"></a>Configuración en el Portal
Puede usar el Portal de Azure para configurar Aplicación de revisión automatizada durante el aprovisionamiento o para las máquinas virtuales existentes.

### <a name="new-vms"></a>Nuevas máquinas virtuales
Use el Portal de Azure para configurar la opción Aplicación de revisión automatizada cuando cree una nueva máquina virtual de SQL Server en el modelo de implementación de Resource Manager.

En la hoja **Configuración de SQL Server**, seleccione **Aplicación de revisión automatizada**. La siguiente captura de pantalla del Portal de Azure muestra la hoja **Aplicación de revisión automatizada de SQL** .

![Aplicación de revisión automatizada de SQL en el Portal de Azure](./media/virtual-machines-windows-sql-automated-patching/azure-sql-arm-patching.png)

Para conocer el contexto, consulte el tema completo en [Aprovisionamiento de una máquina virtual de SQL Server en Azure Portal](virtual-machines-windows-portal-sql-server-provision.md).

### <a name="existing-vms"></a>Máquinas virtuales existentes
Para las máquinas virtuales de SQL Server existentes, seleccione su máquina virtual de SQL Server. Después, seleccione la sección **Configuración de SQL Server** de la hoja **Configuración**.

![Aplicación de revisión automatizada de SQL para máquinas virtuales existentes](./media/virtual-machines-windows-sql-automated-patching/azure-sql-rm-patching-existing-vms.png)

En la hoja **Configuración de SQL Server**, haga clic en el botón **Editar** en la sección Aplicación de revisión automatizada.

![Configuración de Aplicación de revisión automatizada de SQL para máquinas virtuales existentes](./media/virtual-machines-windows-sql-automated-patching/azure-sql-rm-patching-configuration.png)

Cuando termine, haga clic en el botón **Aceptar** situado en la parte inferior de la hoja **Configuración de SQL Server** para guardar los cambios.

Si habilita Aplicación de revisión automatizada por primera vez, Azure configura el agente de IaaS de SQL Server en segundo plano. Durante este tiempo, es posible que el Portal de Azure no muestre que se ha configurado Aplicación de revisión automatizada. Espere unos minutos hasta que el agente se instale y configure. Después, el Portal de Azure muestra la nueva configuración.

## <a name="configuration-with-powershell"></a>Configuración con PowerShell
Después de aprovisionar la máquina virtual de SQL, use PowerShell para configurar Aplicación de revisión automatizada.

En el ejemplo siguiente, se usa PowerShell para configurar Aplicación de revisión automatizada en una máquina virtual de SQL Server existente. El **New AzVMSqlServerAutoPatchingConfig** comando configura una nueva ventana de mantenimiento para actualizaciones automáticas.

    $vmname = "vmname"
    $resourcegroupname = "resourcegroupname"
    $aps = New-AzVMSqlServerAutoPatchingConfig -Enable -DayOfWeek "Thursday" -MaintenanceWindowStartingHour 11 -MaintenanceWindowDuration 120  -PatchCategory "Important"

    Set-AzVMSqlServerExtension -AutoPatchingSettings $aps -VMName $vmname -ResourceGroupName $resourcegroupname

> [!IMPORTANT]
> Si la extensión todavía no está instalada, su instalación reiniciará el servicio de SQL Server.

Según este ejemplo, la siguiente tabla describe el efecto práctico en la máquina virtual de Azure de destino:

| Parámetro | Efecto |
| --- | --- |
| **DayOfWeek** |Las revisiones instaladas cada jueves. |
| **MaintenanceWindowStartingHour** |Inicia las actualizaciones a las 11:00 a.m. |
| **MaintenanceWindowsDuration** |Las revisiones deben instalarse en un plazo de 120 minutos. Según la hora de inicio, deben haberse completado a las 1:00 p.m. |
| **PatchCategory** |La única configuración posible para este parámetro es **Importante**. De este modo, se instalan las actualizaciones de Windows marcadas como importantes; no se instala ninguna actualización de SQL Server que no se incluya en esta categoría. |

La instalación y configuración del agente de Iaas de SQL Server puede tardar algunos minutos.

Para deshabilitar la aplicación de revisiones automatizada, ejecute el mismo script sin el **-habilitar** parámetro para el **New AzVMSqlServerAutoPatchingConfig**. La ausencia del parámetro **-Enable** indica al comando que deshabilite la característica.

## <a name="next-steps"></a>Pasos siguientes
Para más información acerca de otras tareas de automatización disponibles, consulte la [extensión Agente de IaaS de SQL Server](virtual-machines-windows-sql-server-agent-extension.md).

Para más información sobre cómo ejecutar SQL Server en Azure Virtual Machines, consulte [Introducción a SQL Server en Azure Virtual Machines](virtual-machines-windows-sql-server-iaas-overview.md).

