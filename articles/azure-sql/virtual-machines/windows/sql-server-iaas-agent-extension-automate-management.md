---
title: Automatización de las tareas de administración con la extensión Agente de IaaS
description: En este artículo se describe cómo administrar la extensión Agente de IaaS de SQL Server, que automatiza tareas de administración específicas de SQL Server. Entre ellas se incluyen la copia de seguridad automatizada, la aplicación de revisiones automatizada y la integración de Azure Key Vault.
services: virtual-machines-windows
documentationcenter: ''
author: MashaMSFT
editor: ''
tags: azure-resource-manager
ms.assetid: effe4e2f-35b5-490a-b5ef-b06746083da4
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 08/30/2019
ms.author: mathoma
ms.reviewer: jroth
ms.custom: seo-lt-2019
ms.openlocfilehash: 638a7fa96f4112f9805fd46618dc0acd1c2f382f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87063228"
---
# <a name="automate-management-tasks-on-azure-virtual-machines-by-using-the-sql-server-iaas-agent-extension"></a>Automatización de las tareas de administración en Azure Virtual Machines con la extensión del Agente de IaaS de SQL Server
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]


La extensión del agente de IaaS SQL Server (SqlIaasExtension) se ejecuta en Azure Virtual Machines para automatizar las tareas de administración. En este artículo se proporciona información general de los servicios que admite la extensión. En este artículo también se proporcionan instrucciones para la instalación, el estado y la eliminación de la extensión.

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-rm-include.md)]

Para ver la versión clásica de este artículo, consulte [Extensión del Agente de IaaS de SQL Server para VM con SQL Server (clásico)](../../../virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-sql-server-agent-extension.md).


## <a name="supported-services"></a>Servicios admitidos
La extensión del Agente de IaaS SQL Server es compatible con las siguientes tareas de administración:

| Característica de administración | Descripción |
| --- | --- |
| **Copia de seguridad automatizada de SQL Server** |Automatiza la programación de copias de seguridad de todas las bases de datos para la instancia predeterminada o una instancia con nombre [instalada de forma correcta](frequently-asked-questions-faq.md#administration) de SQL Server en la máquina virtual. Para más información, consulte [Automated Backup para SQL Server en Azure Virtual Machines (Resource Manager)](automated-backup-sql-2014.md). |
| **Aplicación de revisiones automatizada de SQL Server** |Configura una ventana de mantenimiento durante la cual tienen lugar actualizaciones importantes de Windows en la máquina virtual, de tal forma que puede evitar las actualizaciones durante las horas punta de la carga de trabajo. Para más información, consulte [Automated Patching para SQL Server en Azure Virtual Machines (Resource Manager)](automated-patching.md). |
| **Integración de Azure Key Vault** |Permite instalar y configurar automáticamente Azure Key Vault en la máquina virtual SQL Server. Para más información, consulte [Configurar la integración de Azure Key Vault para SQL Server en Azure Virtual Machines (Resource Manager)](azure-key-vault-integration-configure.md). |

Una vez que la extensión del Agente de IaaS de SQL Server está instalada y en ejecución, hace que las características de administración estén disponibles:

* En el panel de SQL Server de la máquina virtual en Azure Portal y a través de Azure PowerShell para las imágenes de SQL Server en Azure Marketplace.
* A través de Azure PowerShell para instalaciones manuales de la extensión. 

## <a name="prerequisites"></a>Requisitos previos
A continuación se indican los requisitos para usar la extensión del Agente de IaaS SQL Server en la VM:

**Sistema operativo**:

* Windows Server 2008 R2
* Windows Server 2012
* Windows Server 2012 R2
* Windows Server 2016
* Windows Server 2019 

**Versión de SQL Server**:

* SQL Server 2008 
* SQL Server 2008 R2
* SQL Server 2012
* SQL Server 2014
* SQL Server 2016
* SQL Server 2017
* SQL Server 2019

**Azure PowerShell**:

* [Descargar y configurar los comandos de Azure PowerShell más recientes](/powershell/azure/)

[!INCLUDE [updated-for-az.md](../../../../includes/updated-for-az.md)]


##  <a name="installation"></a>Instalación
La extensión IaaS de SQL Server se instala al registrar la máquina virtual con SQL Server con el [proveedor de recursos de VM con SQL Server](sql-vm-resource-provider-register.md). Si es necesario, puede instalar el agente de IaaS de SQL Server manualmente con el comando de PowerShell siguiente: 

  ```powershell-interactive
    Set-AzVMSqlServerExtension -VMName "sql2017" `
    -ResourceGroupName "LabsqlIAASagent" -Name "SQLIaasExtension" `
    -Version "2.0" -Location "Central US";  
  ```

> [!NOTE]
> Al instalar la extensión, se reinicia el servicio de SQL Server. 


### <a name="install-on-a-vm-with-a-single-named-sql-server-instance"></a>Instalación en una máquina virtual con una única instancia de SQL Server con nombre
La extensión IaaS de SQL Server funcionará con una instancia con nombre en un servidor de SQL Server si la instancia predeterminada se ha desinstalado y se vuelve a instalar la extensión IaaS.

Para usar una instancia con nombre de SQL Server, siga estos pasos:
   1. Implemente una VM SQL Server desde Azure Marketplace. 
   1. Desinstale la extensión IaaS desde [Azure Portal](https://portal.azure.com).
   1. Desinstale SQL Server por completo de la máquina virtual SQL Server.
   1. Instale SQL Server con una instancia con nombre en la máquina virtual SQL Server. 
   1. Desinstale la extensión IaaS desde Azure Portal.  


## <a name="get-the-status-of-the-sql-server-iaas-extension"></a>Obtención del estado de la extensión IaaS de SQL Server
Una manera de comprobar que la extensión está instalada consiste en ver el estado del agente en Azure Portal. Seleccione **Todas las configuraciones** en la ventana de la máquina virtual y seleccione **Extensiones**. Debería aparecer la extensión **SqlIaasExtension**.

![Estado de la extensión del Agente de IaaS de SQL Server en Azure Portal](./media/sql-server-iaas-agent-extension-automate-management/azure-rm-sql-server-iaas-agent-portal.png)

También puede usar el cmdlet de Azure PowerShell **Get-AzVMSqlServerExtension**:

   ```powershell-interactive
   Get-AzVMSqlServerExtension -VMName "vmname" -ResourceGroupName "resourcegroupname"
   ```

El comando anterior confirma que el agente está instalado y proporciona información de estado general. Puede obtener información de estado específica sobre Automated Backup y Automated Patching con los siguientes comandos:

   ```powershell-interactive
    $sqlext = Get-AzVMSqlServerExtension -VMName "vmname" -ResourceGroupName "resourcegroupname"
    $sqlext.AutoPatchingSettings
    $sqlext.AutoBackupSettings
   ```

## <a name="removal"></a>Eliminación
En Azure Portal, puede desinstalar la extensión si selecciona los puntos suspensivos de la ventana **Extensiones** de las propiedades de la máquina virtual. A continuación, seleccione **Eliminar**.

![Desinstalación de la extensión del Agente de IaaS de SQL Server en Azure Portal](./media/sql-server-iaas-agent-extension-automate-management/azure-rm-sql-server-iaas-agent-uninstall.png)

También puede usar el cmdlet de PowerShell **Remove-AzVMSqlServerExtension**:

   ```powershell-interactive
    Remove-AzVMSqlServerExtension -ResourceGroupName "resourcegroupname" -VMName "vmname" -Name "SqlIaasExtension"
   ```

## <a name="next-steps"></a>Pasos siguientes
Empiece utilizando uno de los servicios admitidos por la extensión. Para obtener más información, consulte los artículos a los que se hace referencia en la sección [Servicios admitidos](#supported-services) de este artículo.

Para obtener más información sobre cómo ejecutar SQL Server en Azure Virtual Machines, consulte [¿Qué es SQL Server en Azure Virtual Machines?](sql-server-on-azure-vm-iaas-what-is-overview.md).
