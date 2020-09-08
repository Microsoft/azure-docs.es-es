---
title: Creación de una instancia administrada (plantilla de ARM y PowerShell)
titleSuffix: Azure SQL Managed Instance
description: Use este script de ejemplo de Azure PowerShell para crear una instancia administrada.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: seo-dt-2019, devx-track-azurepowershell
ms.devlang: PowerShell
ms.topic: sample
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein
ms.date: 03/12/2019
ms.openlocfilehash: 6758aed67949032bb93086527623b60af33e716e
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89079564"
---
# <a name="use-powershell-with-an-azure-resource-manager-template-to-create-a-managed-instance"></a>Uso de PowerShell con una plantilla de Azure Resource Manager para crear una instancia administrada

[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqlmi.md)]

Puede crear una instancia administrada mediante la biblioteca de Azure PowerShell y plantillas de Azure Resource Manager.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de manera local, en este tutorial se requiere la versión 1.4.0 o posterior de Azure PowerShell. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, ejecute `Connect-AzAccount` para crear una conexión con Azure.

Los comandos de Azure PowerShell pueden iniciar la implementación mediante una plantilla predefinida de Azure Resource Manager. Se pueden especificar las siguientes propiedades en la plantilla:

- Nombre de la instancia administrada
- Nombre de usuario y contraseña de administrador SQL.
- Tamaño de la instancia de (número de núcleos y tamaño de almacenamiento máximo).
- Red virtual y subred en la que se colocará la instancia.
- Intercalación en el nivel de servidor de la instancia (versión preliminar).

El nombre de la instancia, el nombre de usuario del administrador SQL, la red virtual o la subred, y la intercalación no se podrán modificar posteriormente. Se podrán cambiar otras propiedades de la instancia.

## <a name="prerequisites"></a>Requisitos previos

En este ejemplo se supone que ha [creado un entorno de red válido](../virtual-network-subnet-create-arm-template.md) o que ha [modificado la red virtual existente](../vnet-existing-add-subnet.md) para la Instancia administrada. Puede preparar el entorno de red mediante una [plantilla de Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sql-managed-instance-azure-environment) independiente, si es necesario. 


El ejemplo usa los cmdlets [New-AzResourceGroupDeployment](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroupdeployment) y [Get-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/get-azvirtualnetwork), así que asegúrese de que ha instalado los módulos de PowerShell siguientes:

```powershell
Install-Module Az.Network
Install-Module Az.Resources
```

## <a name="azure-resource-manager-template"></a>Plantilla del Administrador de recursos de Azure


Guarde el siguiente script en un archivo .json y anote la ubicación del archivo: 

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.1",
    "parameters": {
        "instance": {
            "type": "string"
        },
        "user": {
            "type": "string"
        },
        "pwd": {
            "type": "securestring"
        },
        "subnetId": {
            "type": "string"
        }
    },
    "resources": [
        {
            "name": "[parameters('instance')]",
            "location": "West Central US",
            "tags": {
                "Description":"GP Instance with custom instance collation - Serbian_Cyrillic_100_CS_AS"
            },
            "sku": {
                "name": "GP_Gen5",
                "tier": "GeneralPurpose"
            },
            "properties": {
                "administratorLogin": "[parameters('user')]",
                "administratorLoginPassword": "[parameters('pwd')]",
                "subnetId": "[parameters('subnetId')]",
                "storageSizeInGB": 256,
                "vCores": 8,
                "licenseType": "LicenseIncluded",
                "hardwareFamily": "Gen5",
                "collation": "Serbian_Cyrillic_100_CS_AS"
            },
            "type": "Microsoft.Sql/managedInstances",
            "identity": {
                "type": "SystemAssigned"
            },
            "apiVersion": "2015-05-01-preview"
        }
    ]
}
```

Actualice el siguiente script de PowerShell con la ruta de acceso de archivo correcta para el archivo .json que guardó anteriormente y cambie los nombres de los objetos en el script:

```powershell
$subscriptionId = "ed827499-xxxx-xxxx-xxxx-xxxxxxxxxx"
Select-AzSubscription -SubscriptionId $subscriptionId

# Managed instance properties
$resourceGroup = "rg_mi"
$location = "West Central US"
$name = "managed-instance-name"
$user = "miSqlAdmin"
$secpasswd = ConvertTo-SecureString "<Put some strong password here>" -AsPlainText -Force

# Network configuration
$vNetName = "my_vnet"
$vNetResourceGroup = "rg_mi_vnet"
$subnetName = "ManagedInstances"
$vNet = Get-AzVirtualNetwork -Name $vNetName -ResourceGroupName $vNetResourceGroup
$subnet = Get-AzVirtualNetworkSubnetConfig -Name $SubnetName -VirtualNetwork $vNet
$subnetId = $subnet.Id

# Deploy instance using Azure Resource Manager template:
New-AzResourceGroupDeployment  -Name MyDeployment -ResourceGroupName $resourceGroup  `
                                    -TemplateFile 'C:\...\create-managed-instance.json' `
                                    -instance $name -user $user -pwd $secpasswd -subnetId $subnetId
```

Una vez completado el script, se puede acceder a la instancia administrada desde todos los servicios de Azure y la dirección IP configurada.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Puede encontrar más ejemplos de scripts de PowerShell para Instancia administrada de Azure SQL en [Scripts de PowerShell de Instancia administrada de Azure SQL](../../database/powershell-script-content-guide.md).
