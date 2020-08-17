---
title: 'Azure ExpressRoute: Trasladar los circuitos clásicos a Administrador de recursos'
description: En esta página se describe cómo mover un circuito clásico al modelo de implementación de Resource Manager mediante PowerShell.
services: expressroute
author: charwen
ms.service: expressroute
ms.topic: how-to
ms.date: 02/25/2019
ms.author: charwen
ms.openlocfilehash: 3d094f614cbaf75158c38b86345fd5f925310d6b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87039317"
---
# <a name="move-expressroute-circuits-from-classic-to-resource-manager-deployment-model-using-powershell"></a>Transición de los circuitos ExpressRoute desde el modelo de implementación clásica al modelo de implementación de Resource Manager mediante PowerShell

Para usar un circuito ExpressRoute con el modelo de implementación clásica y el modelo de implementación de Resource Manager, debe mover el circuito al modelo de implementación de Resource Manager. Las siguientes secciones le ayudan a mover su circuito mediante PowerShell.

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

* Compruebe que ha instalado localmente en el equipo los módulos clásico y Az de Azure PowerShell. Para obtener más información, consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/).
* Asegúrese de haber revisado los [requisitos previos](expressroute-prerequisites.md), los [requisitos de enrutamiento](expressroute-routing.md) y los [flujos de trabajo](expressroute-workflows.md) antes de comenzar la configuración.
* Revise la información que se proporciona en [Transición de un circuito ExpressRoute desde la implementación clásica a la implementación de Resource Manager](expressroute-move.md). Asegúrese de que comprende perfectamente los límites y restricciones.
* Compruebe que el circuito está totalmente operativo en el modelo de implementación clásica.
* Asegúrese de tener un grupo de recursos creado en el modelo de implementación de Resource Manager.

## <a name="move-an-expressroute-circuit"></a>Movimiento de un circuito ExpressRoute

### <a name="step-1-gather-circuit-details-from-the-classic-deployment-model"></a>Paso 1: Recopilación de detalles del circuito desde el modelo de implementación clásica

Inicie sesión en el entorno clásico de Azure y recopile la clave de servicio.

1. Inicie sesión en la cuenta de Azure.

   ```powershell
   Add-AzureAccount
   ```

2. Seleccione la suscripción de Azure apropiada.

   ```powershell
   Select-AzureSubscription "<Enter Subscription Name here>"
   ```

3. Importe los módulos de PowerShell para Azure y ExpressRoute.

   ```powershell
   Import-Module 'C:\Program Files\WindowsPowerShell\Modules\Azure\5.1.1\Azure\Azure.psd1'
   Import-Module 'C:\Program Files\WindowsPowerShell\Modules\Azure\5.1.1\ExpressRoute\ExpressRoute.psd1'
   ```

4. Use el cmdlet siguiente para obtener las claves de servicio de todos los circuitos ExpressRoute. Después de recuperar las claves, copie la **clave de servicio** del circuito que desea mover al modelo de implementación de Resource Manager.

   ```powershell
   Get-AzureDedicatedCircuit
   ```

### <a name="step-2-sign-in-and-create-a-resource-group"></a>Paso 2: Inicio de sesión y creación de un grupo de recursos

Inicie sesión en el entorno de Resource Manager y cree un grupo de recursos nuevo.

1. Inicie sesión en el entorno de Azure Resource Manager.

   ```powershell
   Connect-AzAccount
   ```

2. Seleccione la suscripción de Azure apropiada.

   ```powershell
   Get-AzSubscription -SubscriptionName "<Enter Subscription Name here>" | Select-AzSubscription
   ```

3. Modifique el fragmento de código siguiente para crear un nuevo grupo de recursos si aún no lo tiene.

   ```powershell
   New-AzResourceGroup -Name "DemoRG" -Location "West US"
   ```

### <a name="step-3-move-the-expressroute-circuit-to-the-resource-manager-deployment-model"></a>Paso 3: Transición del circuito ExpressRoute al modelo de implementación de Resource Manager

Ya está listo para mover el circuito ExpressRoute desde el modelo de implementación clásica al modelo de implementación de Resource Manager. Antes de continuar, revise la información que se proporciona en [Transición de un circuito ExpressRoute desde el modelo de implementación clásica al modelo de implementación de Resource Manager](expressroute-move.md).

Para mover el circuito, modifique y ejecute el siguiente fragmento de código:

```powershell
Move-AzExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "DemoRG" -Location "West US" -ServiceKey "<Service-key>"
```

En el modo clásico, un circuito ExpressRoute no tiene el concepto de estar asociado a una región. Sin embargo, en Resource Manager, todos los recursos deben asignarse a una región de Azure. La región especificada en el cmdlet Move-AzureRmExpressRouteCircuit técnicamente puede ser cualquier región. Por motivos organizativos, puede que quiera elegir una región que represente con precisión la ubicación de emparejamiento.

> [!NOTE]
> Una vez que se termine la transición, se usará el nombre nuevo que aparece en el cmdlet anterior para referirse al recurso. Básicamente, se cambiará el nombre del circuito.
> 

## <a name="modify-circuit-access"></a>Modificación del acceso al circuito

### <a name="to-enable-expressroute-circuit-access-for-both-deployment-models"></a>Habilitación del acceso al circuito ExpressRoute para ambos modelos de implementación

Después de mover el circuito ExpressRoute creado con el modelo clásico al modelo de implementación de Resource Manager, puede habilitar el acceso a ambos modelos de implementación. Ejecute el siguiente cmdlet para habilitar el acceso a ambos modelos de implementación:

1. Obtenga los detalles del circuito.

   ```powershell
   $ckt = Get-AzExpressRouteCircuit -Name "DemoCkt" -ResourceGroupName "DemoRG"
   ```

2. Establezca "Allow Classic Operations" (Permitir operaciones clásicas) en TRUE.

   ```powershell
   $ckt.AllowClassicOperations = $true
   ```

3. Actualice el circuito. Una vez que esta operación finalice correctamente, podrá ver el circuito en el modelo de implementación clásica.

   ```powershell
   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```

4. Ejecute el siguiente cmdlet para obtener los detalles del circuito ExpressRoute. También debe poder ver la clave de servicio.

   ```powershell
   get-azurededicatedcircuit
   ```

5. Ahora puede administrar los vínculos al circuito ExpressRoute mediante los comandos del modelo de implementación clásica para redes virtuales clásicas y los comandos del modelo de implementación de Resource Manager para redes virtuales de Resource Manager. Los artículos siguientes le ayudan a administrar los vínculos al circuito ExpressRoute:

    * [Vinculación de redes virtuales a circuitos ExpressRoute en el modelo de implementación de Resource Manager](expressroute-howto-linkvnet-arm.md)
    * [Vinculación de redes virtuales a circuitos ExpressRoute en el modelo de implementación clásica](expressroute-howto-linkvnet-classic.md)

### <a name="to-disable-expressroute-circuit-access-to-the-classic-deployment-model"></a>Deshabilitación del acceso al circuito ExpressRoute en el modelo de implementación clásica

Ejecute los siguientes cmdlets para deshabilitar el acceso al modelo de implementación clásica.

1. Obtenga los detalles del circuito ExpressRoute.

   ```powershell
   $ckt = Get-AzExpressRouteCircuit -Name "DemoCkt" -ResourceGroupName "DemoRG"
   ```

2. Establezca "Allow Classic Operations" (Permitir operaciones clásicas) en FALSE.

   ```powershell
   $ckt.AllowClassicOperations = $false
   ```

3. Actualice el circuito. Una vez que esta operación finalice correctamente, no podrá ver el circuito en el modelo de implementación clásica.

   ```powershell
   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```

## <a name="next-steps"></a>Pasos siguientes

* [Crear y modificar el enrutamiento para el circuito ExpressRoute](expressroute-howto-routing-arm.md)
* [Vincular la red virtual a su circuito ExpressRoute](expressroute-howto-linkvnet-arm.md)
