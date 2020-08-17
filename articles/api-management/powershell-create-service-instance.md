---
title: Creación de una instancia de Azure API Management con PowerShell | Microsoft Docs
description: Siga los pasos de este tutorial para crear una nueva instancia de API Management.
services: api-management
documentationcenter: ''
author: vladvino
manager: cflower
editor: ''
ms.service: api-management
ms.workload: integration
ms.topic: quickstart
ms.custom: mvc
ms.date: 11/15/2017
ms.author: apimpm
ms.openlocfilehash: f42a68d066898869574cb6ccf5099ccdd451e5fe
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86506935"
---
# <a name="create-a-new-azure-api-management-service-instance-by-using-powershell"></a>Creación de una nueva instancia del servicio Azure API Management mediante PowerShell

Azure API Management (APIM) ayuda a las organizaciones a publicar API para desarrolladores externos, asociados e internos a fin de desbloquear el potencial de sus datos y servicios. API Management proporciona las competencias esenciales para garantizar un programa de API de éxito mediante compromisos con desarrolladores, información detallada empresarial, análisis, seguridad y protección. APIM le permite crear y administrar modernas puertas de enlace de API para los servicios back-end existentes hospedados en cualquier lugar. Para obtener más información, consulte el tema de [introducción](api-management-key-concepts.md).

En este inicio rápido se describen los pasos que deben seguirse para crear una nueva instancia de API Management mediante scripts de PowerShell. El inicio rápido muestra cómo utilizar el servicio **Azure Cloud Shell** que puede ejecutar desde Azure Portal.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en https://portal.azure.com.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de forma local, en este tutorial necesitará la versión 1.0 del módulo de Azure PowerShell o cualquier versión posterior. Ejecute `Get-Module -ListAvailable Az` para encontrar la versión. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.


## <a name="create-resource-group"></a>Creación de un grupo de recursos

Cree un grupo de recursos de Azure con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. 

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location WestUS
```

## <a name="create-an-api-management-service"></a>Creación de un servicio de API Management

Se trata de una operación larga que puede tardar hasta 15 minutos.

```azurepowershell-interactive
New-AzApiManagement -ResourceGroupName "myResourceGroup" -Location "West US" -Name "apim-name" -Organization "myOrganization" -AdminEmail "myEmail" -Sku "Developer"
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, puede usar el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos y todos los recursos relacionados.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Importación y publicación de la primera API](import-and-publish.md)
