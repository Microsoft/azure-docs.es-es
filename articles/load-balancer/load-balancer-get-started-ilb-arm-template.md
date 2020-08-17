---
title: 'Creación de un equilibrador de carga interno: plantilla de Azure'
titleSuffix: Azure Load Balancer
description: Obtenga información sobre cómo crear un equilibrador de carga interno mediante una plantilla en el Administrador de recursos
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.custom: seodec18, devx-track-azurecli
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: allensu
ms.openlocfilehash: fc64f4b3d338704568cb70a542a82fe58d08d4d9
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502314"
---
# <a name="create-an-internal-load-balancer-using-a-template"></a>Creación del equilibrador de carga interno con una plantilla

> [!div class="op_single_selector"]
> * [Azure Portal](../load-balancer/load-balancer-get-started-ilb-arm-portal.md)
> * [PowerShell](../load-balancer/load-balancer-get-started-ilb-arm-ps.md)
> * [CLI de Azure](../load-balancer/load-balancer-get-started-ilb-arm-cli.md)
> * [Plantilla](../load-balancer/load-balancer-get-started-ilb-arm-template.md)

[!INCLUDE [load-balancer-get-started-ilb-intro-include.md](../../includes/load-balancer-get-started-ilb-intro-include.md)]

[!INCLUDE [load-balancer-get-started-ilb-scenario-include.md](../../includes/load-balancer-get-started-ilb-scenario-include.md)]

## <a name="deploy-the-template-by-using-click-to-deploy"></a>Implementar la plantilla por medio de un solo clic para implementar

La plantilla de ejemplo disponible en el repositorio público usa un archivo de parámetros que contiene los valores predeterminados utilizados para generar el escenario descrito anteriormente. Para implementar esta plantilla mediante el método de hacer clic para implementar, siga [esta plantilla](https://github.com/Azure/azure-quickstart-templates/tree/master/201-2-vms-internal-load-balancer), haga clic en **Implementar en Azure**, reemplace los valores de parámetro predeterminados si es necesario y siga las instrucciones del portal.

## <a name="deploy-the-template-by-using-powershell"></a>Implementación de la plantilla mediante PowerShell

Para implementar la plantilla que descargó con PowerShell, siga estos pasos.

1. Si es la primera vez que usa Azure PowerShell, consulte [Cómo instalar y configurar Azure PowerShell](/powershell/azure/) y siga las instrucciones hasta el final para iniciar sesión en Azure y seleccionar su suscripción.
2. Descargue el archivo de parámetros en el disco local.
3. Edite el archivo y guárdelo.
4. Ejecute el cmdlet **New-AzResourceGroupDeployment** para crear un grupo de recursos mediante la plantilla.

    ```azurepowershell-interactive
    New-AzResourceGroupDeployment -Name TestRG -Location westus `
        -TemplateFile 'https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-internal-load-balancer/azuredeploy.json' `
        -TemplateParameterFile 'C:\temp\azuredeploy.parameters.json'
    ```

## <a name="deploy-the-template-by-using-the-azure-cli"></a>Implementación la plantilla ARM mediante la CLI de Azure

Para implementar la plantilla ARM mediante la CLI de Azure, siga estos pasos.

1. Si nunca ha usado la CLI de Azure, consulte [Instalación y configuración de la CLI de Azure](../cli-install-nodejs.md) y siga las instrucciones hasta el punto donde deba seleccionar su cuenta y suscripción de Azure.
2. Vaya a [https://shell.azure.com](https://shell.azure.com) para abrir Cloud Shell en el explorador. Ejecute el comando **azure config mode** para cambiar al modo de Administrador de recursos, como se muestra a continuación.

    ```console
    azure config mode arm
    ```

    Este es el resultado esperado del comando anterior:

    ```output
    info:    New mode is arm
    ```

3. Abra el archivo de parámetros, seleccione su contenido y guárdelo en un archivo en el equipo. Para este ejemplo, guardamos el archivo de parámetros en *parameters.json*.
4. Ejecute el comando **azure group deployment create** para implementar el nuevo equilibrador de carga interno mediante la plantilla y los archivos de parámetros que ha descargado y modificado anteriormente. En la lista que se muestra en la salida se explican los parámetros utilizados.

    ```console
    azure group create --name TestRG --location westus --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-internal-load-balancer/azuredeploy.json --parameters-file parameters.json
    ```

## <a name="next-steps"></a>Pasos siguientes

[Configurar un modo de distribución del equilibrador de carga mediante la afinidad IP de origen](load-balancer-distribution-mode.md)

[Configuración de opciones de tiempo de espera de inactividad de TCP para el equilibrador de carga](load-balancer-tcp-idle-timeout.md)

Para la sintaxis JSON y las propiedades de un equilibrador de carga en una plantilla, consulte [Microsoft.Network/loadBalancers](/azure/templates/microsoft.network/loadbalancers).
