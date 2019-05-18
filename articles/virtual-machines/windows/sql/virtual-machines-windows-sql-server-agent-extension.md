---
title: Automatizar tareas de administración en máquinas virtuales de SQL (Resource Manager) | Microsoft Docs
description: En este artículo se describe cómo administrar la extensión Agente de SQL Server, que automatiza tareas de administración específicas de SQL Server. Entre ellas se incluyen la copia de seguridad automatizada, la aplicación de revisiones automatizada y la integración de Azure Key Vault.
services: virtual-machines-windows
documentationcenter: ''
author: MashaMSFT
manager: craigg
editor: ''
tags: azure-resource-manager
ms.assetid: effe4e2f-35b5-490a-b5ef-b06746083da4
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 07/12/2018
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 2943a1501e1d81ff4884c21b5aa3861e16523bbb
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65827738"
---
# <a name="automate-management-tasks-on-azure-virtual-machines-with-the-sql-server-agent-extension-resource-manager"></a>Automatizar las tareas de administración en Azure Virtual Machines con la extensión del Agente SQL Server (Resource Manager)
> [!div class="op_single_selector"]
> * [Resource Manager](virtual-machines-windows-sql-server-agent-extension.md)
> * [Clásico](../sqlclassic/virtual-machines-windows-classic-sql-server-agent-extension.md)

La extensión del agente de IaaS SQL Server (SqlIaasExtension) se ejecuta en Azure Virtual Machines para automatizar las tareas de administración. En este artículo se proporciona información general sobre los servicios admitidos por la extensión, así como instrucciones para la instalación, el estado y la eliminación.

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-rm-include.md)]

Para ver la versión clásica de este artículo, consulte [Extensión del Agente SQL Server para VM con SQL Server (clásico)](../sqlclassic/virtual-machines-windows-classic-sql-server-agent-extension.md).

## <a name="supported-services"></a>Servicios admitidos
La extensión del Agente de IaaS SQL Server es compatible con las siguientes tareas de administración:

| Característica de administración | DESCRIPCIÓN |
| --- | --- |
| **Automated Backup de SQL** |Automatiza la programación de copias de seguridad para todas las bases de datos para cualquier instancia predeterminada o una [correctamente instalados](virtual-machines-windows-sql-server-iaas-faq.md#administration) con el nombre de instancia de SQL Server en la máquina virtual. Para más información, consulte [Copia de seguridad automatizada para SQL Server en Azure Virtual Machines (Resource Manager)](virtual-machines-windows-sql-automated-backup.md). |
| **Aplicación de revisiones automatizada de SQL** |Configura una ventana de mantenimiento durante la cual tienen lugar actualizaciones importantes de Windows en la máquina virtual, de tal forma que puede evitar las actualizaciones durante las horas punta de la carga de trabajo. Para más información, consulte [Aplicación de revisión automatizada para SQL Server en Azure Virtual Machines (Resource Manager)](virtual-machines-windows-sql-automated-patching.md). |
| **Integración de Azure Key Vault** |Permite instalar y configurar automáticamente Azure Key Vault en la máquina virtual SQL Server. Para más información, consulte [Configuración de la integración de Azure Key Vault para SQL Server en máquinas virtuales de Azure (Resource Manager)](virtual-machines-windows-ps-sql-keyvault.md). |

Una vez instalada y en ejecución, la extensión Agente de IaaS de SQL Server permite que estas características de administración estén disponibles en el panel de SQL Server de la máquina virtual de Azure Portal y a través de Azure PowerShell, tanto para imágenes de Marketplace de SQL Server como para instalaciones manuales de la extensión. 

## <a name="prerequisites"></a>Requisitos previos
Requisitos para usar la extensión del Agente de IaaS SQL Server en la máquina virtual:

**Sistema operativo**:

* Windows Server 2008 R2
* Windows Server 2012
* Windows Server 2012 R2
* Windows Server 2016

**Versiones de SQL Server**:

* SQL Server 2008 
* SQL Server 2008 R2
* SQL Server 2012
* SQL Server 2014
* SQL Server 2016

**Azure PowerShell**:

* [Descargar y configurar los comandos de Azure PowerShell más recientes](/powershell/azure/overview)

[!INCLUDE [updated-for-az.md](../../../../includes/updated-for-az.md)]

> [!IMPORTANT]
> En este momento, la [extensión del agente de IaaS de SQL Server](virtual-machines-windows-sql-server-agent-extension.md) no es compatible con FCI de SQL Server en Azure. Se recomienda que desinstale la extensión de las máquinas virtuales que participan en una FCI. Las características admitidas por la extensión no estarán disponibles para las máquinas virtuales de SQL después de desinstalar el agente.

## <a name="installation"></a>Instalación
La extensión del Agente de IaaS SQL Server se instala automáticamente cuando aprovisiona una de las imágenes de la galería de máquina virtual de SQL Server. La extensión SQL IaaS ofrece la capacidad de administración para una única instancia en la máquina virtual de SQL Server. Si hay una instancia predeterminada, a continuación, la extensión funciona con la instancia predeterminada y no admitirá la administración de otras instancias. Si no hay ninguna instancia predeterminada, pero solo una instancia con nombre, administrará la instancia con nombre. Si no hay ninguna instancia predeterminada y hay varias instancias con nombre, a continuación, la extensión se producirá un error al instalar. 



Si tiene que volver a instalar la extensión manualmente en una de estas máquinas virtuales con SQL Server, use el siguiente comando de PowerShell:

```powershell
Set-AzVMSqlServerExtension -ResourceGroupName "resourcegroupname" -VMName "vmname" -Name "SqlIaasExtension" -Version "2.0" -Location "East US 2"
```

> [!WARNING]
> Si la extensión todavía no está instalada, su instalación reiniciará el servicio de SQL Server. Sin embargo, la actualización de la extensión IaaS de SQL no reinicia el servicio SQL Server. 

> [!NOTE]
> Aunque es posible instalar la extensión del agente de IaaS de SQL Server para las imágenes personalizadas de SQL Server, la funcionalidad está actualmente limitada a [cambiar el tipo de licencia](virtual-machines-windows-sql-ahb.md). Otras características proporcionadas por la extensión SQL IaaS solo funcionará en [imágenes de la Galería de VM de SQL Server](virtual-machines-windows-sql-server-iaas-overview.md#get-started-with-sql-vms) (pago por uso o bring-your-propia licencia).

### <a name="use-a-single-named-instance"></a>Usar una sola instancia con nombre
La extensión SQL IaaS funcionará con una instancia con nombre en una imagen de SQL Server si se ha desinstalado correctamente a la instancia predeterminada y se vuelve a instalar la extensión de IaaS.

Para usar una instancia con nombre de SQL Server, realice lo siguiente:
   1. Implementar una máquina virtual de SQL Server desde marketplace. 
   1. Desinstalar la extensión de IaaS desde el [portal Azure](https://portal.azure.com).
   1. Desinstalar SQL Server por completo dentro de la máquina virtual de SQL Server.
   1. Instalar a SQL Server con una instancia con nombre dentro de la máquina virtual de SQL Server. 
   1. Instale la extensión de IaaS desde dentro del portal de Azure.  

## <a name="status"></a>Status
Una manera de comprobar que la extensión está instalada consiste en ver el estado del agente en Azure Portal. Seleccione **Todas las configuraciones** en la ventana de la máquina virtual y haga clic en **Extensiones**. Debería aparecer la extensión **SqlIaasExtension**.

![Extensión del Agente de IaaS de SQL Server en Azure Portal](./media/virtual-machines-windows-sql-server-agent-extension/azure-rm-sql-server-iaas-agent-portal.png)

También puede usar el cmdlet de Azure PowerShell **Get-AzVMSqlServerExtension**.

    Get-AzVMSqlServerExtension -VMName "vmname" -ResourceGroupName "resourcegroupname"

El comando anterior confirma que el agente está instalado y proporciona información de estado general. También puede obtener información de estado específica sobre Automated Backup y Automated Backup con los siguientes comandos.

    $sqlext = Get-AzVMSqlServerExtension -VMName "vmname" -ResourceGroupName "resourcegroupname"
    $sqlext.AutoPatchingSettings
    $sqlext.AutoBackupSettings

## <a name="removal"></a>Eliminación
En el portal de Azure, puede desinstalar la extensión haciendo clic en el botón de puntos suspensivos en el **extensiones** ventana de propiedades de la máquina virtual. Después, haga clic en **Eliminar**.

![Desinstalación de la extensión Agente de IaaS de SQL Server en Azure Portal](./media/virtual-machines-windows-sql-server-agent-extension/azure-rm-sql-server-iaas-agent-uninstall.png)

También puede usar el cmdlet de PowerShell **Remove-AzVMSqlServerExtension**.

    Remove-AzVMSqlServerExtension -ResourceGroupName "resourcegroupname" -VMName "vmname" -Name "SqlIaasExtension"

## <a name="next-steps"></a>Pasos siguientes
Empiece utilizando uno de los servicios admitidos por la extensión. Para más información, consulte los artículos a los que se hace referencia en la sección [Servicios admitidos](#supported-services) de este artículo.

Para obtener más información sobre cómo ejecutar SQL Server en Azure Virtual Machines, consulte [Información general sobre SQL Server en Azure Virtual Machines](virtual-machines-windows-sql-server-iaas-overview.md).

