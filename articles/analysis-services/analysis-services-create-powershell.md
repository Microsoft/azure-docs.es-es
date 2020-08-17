---
title: 'Inicio rápido: Creación de una instancia de Azure Analysis Services mediante PowerShell | Microsoft Docs'
description: Obtenga información sobre cómo crear un servidor de Azure Analysis Services mediante PowerShell
author: minewiskan
ms.service: azure-analysis-services
ms.topic: quickstart
ms.date: 03/30/2020
ms.author: owend
ms.reviewer: minewiskan
ms.custom: references_regions
ms.openlocfilehash: 375d72c8de0aa16d2c10da701fb9670a9c87b79f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87015466"
---
# <a name="quickstart-create-a-server---powershell"></a>Inicio rápido: Creación de un servidor (PowerShell)

Este inicio rápido describe el uso de PowerShell desde la línea de comandos para crear un servidor de Azure Analysis Services de su suscripción.

## <a name="prerequisites"></a>Prerequisites

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

- **Suscripción de Azure**: visite [Evaluación gratuita de Azure](https://azure.microsoft.com/offers/ms-azr-0044p/) para crear una cuenta.
- **Azure Active Directory**: la suscripción debe estar asociada a un inquilino de Azure Active Directory y debe tener una cuenta en ese directorio. Para más información, consulte [Permisos de usuario y autenticación](analysis-services-manage-users.md).
- **Azure PowerShell**. Ejecute `Get-Module -ListAvailable Az` para ver cuál es la versión instalada. Si necesita instalarla o actualizarla, consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/install-Az-ps).

## <a name="import-azanalysisservices-module"></a>Importación del módulo Az.AnalysisServices

Para crear un servidor en su suscripción, use el módulo [Az.AnalysisServices](/powershell/module/az.analysisservices). Cargue el módulo Az.AnalysisServices en la sesión de PowerShell.

```powershell
Import-Module Az.AnalysisServices
```

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en su suscripción de Azure mediante el comando [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Siga las instrucciones que aparecen en pantalla.

```powershell
Connect-AzAccount
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) es un contenedor lógico en el que se implementan y se administran los recursos de Azure como grupo. Cuando cree el servidor, deberá especificar un grupo de recursos de su suscripción. Si aún no dispone de un grupo de recursos, puede crearlo mediante el comando [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). En el siguiente ejemplo, se crea un grupo de recursos denominado `myResourceGroup` en la región del oeste de EE. UU.

```powershell
New-AzResourceGroup -Name "myResourceGroup" -Location "WestUS"
```

## <a name="create-a-server"></a>Creación de un servidor

Cree un servidor mediante el comando [New-AzAnalysisServicesServer](/powershell/module/az.analysisservices/new-azanalysisservicesserver). En el ejemplo siguiente se crea un servidor denominado myServer en myResourceGroup, en la región oeste de Estados Unidos, en el nivel de D1 (gratis), y se especifica philipc@adventureworks.com como administrador del servidor.

```powershell
New-AzAnalysisServicesServer -ResourceGroupName "myResourceGroup" -Name "myserver" -Location WestUS -Sku D1 -Administrator "philipc@adventure-works.com"
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Puede quitar el servidor de la suscripción mediante el comando [Remove-AzAnalysisServicesServer](/powershell/module/az.analysisservices/new-azanalysisservicesserver). Si va a seguir con otros inicios rápidos y tutoriales de esta colección, no quite el servidor. En el ejemplo siguiente se quita el servidor creado en el paso anterior.


```powershell
Remove-AzAnalysisServicesServer -Name "myserver" -ResourceGroupName "myResourceGroup"
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido aprendió a crear un servidor en la suscripción de Azure mediante PowerShell. Ahora que tiene un servidor, puede protegerlo si configura un firewall de servidor (opcional). También puede agregar un modelo de datos de ejemplo básico al servidor directamente desde el portal. Tener un modelo de ejemplo es útil al aprender cómo configurar los roles de la base de datos del modelo y probar las conexiones de cliente. Para más información, puede seguir el tutorial para agregar un modelo de ejemplo.

> [!div class="nextstepaction"]
> [Guía de inicio rápido: Configuración del firewall del servidor: Portal](analysis-services-qs-firewall.md)      
> [!div class="nextstepaction"]
> [Tutorial: Incorporación de un modelo de ejemplo al servidor](analysis-services-create-sample-model.md)
