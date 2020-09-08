---
title: Invalidación de la subred de Azure Traffic Manager mediante Azure PowerShell | Microsoft Docs
description: Este artículo lo ayudará a entender cómo la invalidación de la subred de Traffic Manager se usa para reemplazar el método de enrutamiento de un perfil de Traffic Manager para dirigir el tráfico a un punto de conexión en función de la dirección IP del usuario final a través de un intervalo IP predefinido a las asignaciones de punto de conexión mediante Azure PowerShell.
services: traffic-manager
documentationcenter: ''
author: duongau
ms.topic: how-to
ms.service: traffic-manager
ms.date: 09/18/2019
ms.author: duau
ms.openlocfilehash: 01bd3b1e945ee7c9ac16af7048536c0a9e2d731a
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401595"
---
# <a name="traffic-manager-subnet-override-using-azure-powershell"></a>Invalidación de la subred de Traffic Manager mediante Azure PowerShell

La invalidación de la subred de Traffic Manager le permite modificar el método de enrutamiento de un perfil.  La incorporación de una invalidación dirigirá el tráfico en función de la dirección IP del usuario final con un intervalo IP predefinido a una asignación de punto de conexión. 

## <a name="how-subnet-override-works"></a>Funcionamiento de la invalidación de subred

Cuando se agregan invalidaciones de subred a un perfil de Traffic Manager, Traffic Manager comprobará primero si existe una invalidación de subred para la dirección IP del usuario final. Si se encuentra una, la consulta de DNS del usuario se dirigirá al punto de conexión correspondiente.  Si no se encuentra una asignación, Traffic Manager revertirá al método de enrutamiento original del perfil. 

Los intervalos de direcciones IP se pueden especificar como intervalos CIDR (por ejemplo, 1.2.3.0/24) o como intervalos de direcciones (por ejemplo, 1.2.3.4-5.6.7.8). Los intervalos IP asociados a cada punto de conexión deben ser únicos para ese punto de conexión. Cualquier superposición de intervalos IP entre distintos puntos de conexión hará Traffic Manager rechace el perfil.

Hay dos tipos de perfiles de enrutamiento que admiten invalidaciones de subred:

* **Geográfico**: si Traffic Manager encuentra una invalidación de subred para la dirección IP de la consulta de DNS, enrutará la consulta al punto de conexión, independientemente del estado del punto de conexión.
* **Rendimiento**: si Traffic Manager encuentra una invalidación de subred para la dirección IP de la consulta de DNS, solo enrutará el tráfico al punto de conexión si tiene un estado correcto.  Traffic Manager volverá a la heurística de enrutamiento del rendimiento si el punto de conexión de invalidación de subred no tiene un estado correcto.

## <a name="create-a-traffic-manager-subnet-override"></a>Creación de una invalidación de subred de Traffic Manager

Para crear una invalidación de subred de Traffic Manager, puede usar Azure PowerShell para agregar las subredes para la invalidación al punto de conexión de Traffic Manager.

## <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Puede ejecutar los comandos siguientes en [Azure Cloud Shell](https://shell.azure.com/powershell), o mediante la ejecución de PowerShell en el equipo. Azure Cloud Shell es un shell interactivo gratuito. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. Si ejecuta PowerShell desde el equipo, necesita el módulo Azure PowerShell, versión 1.0.0 o posterior. Puede ejecutar `Get-Module -ListAvailable Az` para encontrar la versión instalada. Si necesita instalarla o actualizarla, consulte el artículo sobre [cómo instalar el módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si ejecuta PowerShell localmente, también debe ejecutar `Login-AzAccount` para iniciar sesión en Azure.


1. **Recupere el punto de conexión de Traffic Manager:**

    Para habilitar la invalidación de subred, recupere el punto de conexión que quiere agregar a la invalidación y almacénelo en una variable con [Get-AzTrafficManagerEndpoint](https://docs.microsoft.com/powershell/module/az.trafficmanager/get-aztrafficmanagerendpoint?view=azps-2.5.0).

    Reemplace los valores de Name, ProfileName y ResourceGroupName por los valores del punto de conexión que va a cambiar.

    ```powershell

    $TrafficManagerEndpoint = Get-AzTrafficManagerEndpoint -Name "contoso" -ProfileName "ContosoProfile" -ResourceGroupName "ResourceGroup" -Type AzureEndpoints

    ```
2. **Agregue el intervalo de direcciones IP en el punto de conexión:**
    
    Para agregar el intervalo de direcciones IP al punto de conexión, usará [Add-AzTrafficManagerIpAddressRange](https://docs.microsoft.com/powershell/module/az.trafficmanager/add-aztrafficmanageripaddressrange?view=azps-2.5.0&viewFallbackFrom=azps-2.4.0) para agregar el intervalo.

    ```powershell

    ### Add a range of IPs ###
    Add-AzTrafficManagerIPAddressRange -TrafficManagerEndpoint $TrafficManagerEndpoint -First "1.2.3.4" -Last "5.6.7.8"

    ### Add a subnet ###
    Add-AzTrafficManagerIPAddressRange -TrafficManagerEndpoint $TrafficManagerEndpoint -First "9.10.11.0" -Scope 24

    ### Add a range of IPs with a subnet ###
    Add-AzTrafficManagerIPAddressRange -TrafficManagerEndpoint $TrafficManagerEndpoint -First "12.13.14.0" -Last "12.13.14.31" -Scope 27
 
    ```
    Una vez que se agreguen los intervalos, use [Set-AzTrafficManagerEndpoint](https://docs.microsoft.com/powershell/module/az.trafficmanager/set-aztrafficmanagerendpoint?view=azps-2.5.0) para actualizar el punto de conexión.

    ```powershell

    Set-AzTrafficManagerEndpoint -TrafficManagerEndpoint $TrafficManagerEndpoint

    ```
Para completar la eliminación del intervalo de direcciones IP, use [Remove-AzTrafficManagerIpAddressRange](https://docs.microsoft.com/powershell/module/az.trafficmanager/remove-aztrafficmanageripaddressrange?view=azps-2.5.0).

1.  **Recupere el punto de conexión de Traffic Manager:**

    Para habilitar la invalidación de subred, recupere el punto de conexión que quiere agregar a la invalidación y almacénelo en una variable con [Get-AzTrafficManagerEndpoint](https://docs.microsoft.com/powershell/module/az.trafficmanager/get-aztrafficmanagerendpoint?view=azps-2.5.0).

    Reemplace los valores de Name, ProfileName y ResourceGroupName por los valores del punto de conexión que va a cambiar.

    ```powershell

    $TrafficManagerEndpoint = Get-AzTrafficManagerEndpoint -Name "contoso" -ProfileName "ContosoProfile" -ResourceGroupName "ResourceGroup" -Type AzureEndpoints

    ```
2. **Quite el intervalo de direcciones IP del punto de conexión:**

    ```powershell
    
    ### Remove a range of IPs ###
    Remove-AzTrafficManagerIpAddressRange -TrafficManagerEndpoint $TrafficManagerEndpoint -First "1.2.3.4" -Last "5.6.7.8"

    ### Remove a subnet ###
    Remove-AzTrafficManagerIpAddressRange -TrafficManagerEndpoint $TrafficManagerEndpoint -First "9.10.11.0" -Scope 24

    ### Remove a range of IPs with a subnet ###
    Remove-AzTrafficManagerIpAddressRange -TrafficManagerEndpoint $TrafficManagerEndpoint -First "12.13.14.0" -Last "12.13.14.31" -Scope 27

    ```
     Una vez que se quiten los intervalos, use [Set-AzTrafficManagerEndpoint](https://docs.microsoft.com/powershell/module/az.trafficmanager/set-aztrafficmanagerendpoint?view=azps-2.5.0) para actualizar el punto de conexión.

    ```powershell

    Set-AzTrafficManagerEndpoint -TrafficManagerEndpoint $TrafficManagerEndpoint

    ```

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre los [métodos de enrutamiento del tráfico](traffic-manager-routing-methods.md) de Traffic Manager.

Obtenga más información sobre el [método de enrutamiento de tráfico de subred](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-routing-methods#subnet-traffic-routing-method).
