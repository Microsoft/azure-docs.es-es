---
title: 'Azure ExpressRoute: Configuración del emparejamiento: clásico'
description: Este artículo le guiará por los pasos necesarios para crear y aprovisionar las configuraciones entre pares privados, públicos y de Microsoft de un circuito ExpressRoute. Este artículo también muestra cómo comprobar el estado, actualizar, o eliminar configuraciones entre pares en el circuito.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 12/06/2019
ms.author: duau
ms.openlocfilehash: a4a3bad1e868fa0e75611630ffb5db5ba13126b6
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89395560"
---
# <a name="create-and-modify-peering-for-an-expressroute-circuit-classic"></a>Creación y modificación del emparejamiento de un circuito ExpressRoute (clásica)
> [!div class="op_single_selector"]
> * [Azure Portal](expressroute-howto-routing-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-routing-arm.md)
> * [CLI de Azure](howto-routing-cli.md)
> * [Vídeo: pares privados](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-azure-private-peering-for-your-expressroute-circuit)
> * [Vídeo: pares públicos](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-azure-public-peering-for-your-expressroute-circuit)
> * [Vídeo: emparejamiento de Microsoft](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-microsoft-peering-for-your-expressroute-circuit)
> * [PowerShell (clásico)](expressroute-howto-routing-classic.md)
> 

Este artículo le guiará por los pasos necesarios para crear y administrar la configuración de enrutamiento o emparejamiento para un circuito ExpressRoute con PowerShell y el modelo de implementación clásica. Los siguientes pasos también le mostrarán cómo comprobar el estado, actualizar, o eliminar y desaprovisionar las configuraciones entre pares para un circuito ExpressRoute. Puede configurar una, dos o las tres configuraciones entre pares (Azure privado, Azure público y Microsoft) para un circuito ExpressRoute. Puede establecer las configuraciones entre pares en cualquier orden. Pero tiene que asegurarse de que completa cada configuración entre pares de una en una. 

Estas instrucciones se aplican solo a los circuitos que se crean con proveedores de servicios que ofrecen servicios de conectividad de nivel 2. Si usa un proveedor de servicios que ofrece servicios administrados de nivel 3 (normalmente VPN IP, como MPLS), el mismo proveedor de conectividad configurará y administrará el enrutamiento.

[!INCLUDE [expressroute-classic-end-include](../../includes/expressroute-classic-end-include.md)]

**Información sobre los modelos de implementación de Azure**

[!INCLUDE [vpn-gateway-classic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

## <a name="configuration-prerequisites"></a>Requisitos previos de configuración

* Antes de comenzar la configuración, asegúrese de que ha revisado la página de [requisitos previos](expressroute-prerequisites.md), la página de [requisitos de enrutamiento](expressroute-routing.md) y la página de [flujos de trabajo](expressroute-workflows.md).
* Tiene que tener un circuito ExpressRoute activo. Siga las instrucciones para [crear un circuito ExpressRoute](expressroute-howto-circuit-classic.md) y habilite el circuito mediante el proveedor de conectividad antes de continuar. El circuito ExpressRoute debe estar en un estado habilitado y aprovisionado para poder ejecutar los cmdlets que se describen a continuación.

### <a name="download-the-latest-powershell-cmdlets"></a>Descarga de los cmdlets de PowerShell más recientes

[!INCLUDE [classic powershell install instructions](../../includes/expressroute-poweshell-classic-install-include.md)]

## <a name="azure-private-peering"></a>Configuración entre pares privados de Azure

Esta sección proporciona instrucciones sobre cómo crear, obtener, actualizar y eliminar la configuración entre pares privados de Azure para un circuito ExpressRoute. 

### <a name="to-create-azure-private-peering"></a>Creación de un emparejamiento privado de Azure

1. **Creación de un circuito ExpressRoute.**

   Siga las instrucciones para crear un [circuito ExpressRoute](expressroute-howto-circuit-classic.md) y habilite su aprovisionamiento a través del proveedor de conectividad. Si su proveedor de conectividad ofrece servicios administrados de nivel 3, puede solicitarle que habilite la configuración entre pares privados de Azure. En ese caso, no necesita seguir las instrucciones que aparecen en las secciones siguientes. Por otra parte, si su proveedor de conectividad no administra este enrutamiento, una vez que cree el circuito siga las instrucciones siguientes.
2. **Compruebe el circuito ExpressRoute para asegurarse de que está aprovisionado.**
   
   Compruebe si el circuito ExpressRoute tiene los estados Aprovisionado y Habilitado.

   ```powershell
   Get-AzureDedicatedCircuit -ServiceKey "*********************************"
   ```

   Devuelve:

   ```powershell
   Bandwidth                        : 200
   CircuitName                      : MyTestCircuit
   Location                         : Silicon Valley
   ServiceKey                       : *********************************
   ServiceProviderName              : equinix
   ServiceProviderProvisioningState : Provisioned
   Sku                              : Standard
   Status                           : Enabled
   ```
   
   Asegúrese de que el circuito se muestra como Aprovisionado y Habilitado. Si no es así, colabore con su proveedor de conectividad para obtener el circuito para establecer el estado y la condición necesarios para el circuito.

   ```powershell
   ServiceProviderProvisioningState : Provisioned
   Status                           : Enabled
   ```
3. **Establecimiento de la configuración entre pares privados de Azure para el circuito.**

   Asegúrese de que tiene los elementos siguientes antes de continuar con los siguientes pasos:
   
   * Una subred /30 para el vínculo principal. No debe ser parte de ningún espacio de direcciones reservado para redes virtuales.
   * Una subred /30 para el vínculo secundario. No debe ser parte de ningún espacio de direcciones reservado para redes virtuales.
   * Un identificador VLAN válido para establecer esta configuración entre pares. Asegúrese de que ningún otro emparejamiento en el circuito usa el mismo identificador de VLAN.
   * Número de sistema autónomo (AS) para la configuración entre pares. Puede usar 2 bytes o 4 bytes como números AS. Puede usar un número AS privado para esta configuración entre pares. Compruebe que no usa 65515.
   * Un hash MD5, en caso de que haya decidido usarlo. **Opcional**.
     
   Puede usar el ejemplo siguiente para configurar el emparejamiento privado de Azure para su circuito:

   ```powershell
   New-AzureBGPPeering -AccessType Private -ServiceKey "*********************************" -PrimaryPeerSubnet "10.0.0.0/30" -SecondaryPeerSubnet "10.0.0.4/30" -PeerAsn 1234 -VlanId 100
   ```    

   Si desea usar un hash MD5, use el ejemplo siguiente para configurar el emparejamiento privado para su circuito:

   ```powershell
   New-AzureBGPPeering -AccessType Private -ServiceKey "*********************************" -PrimaryPeerSubnet "10.0.0.0/30" -SecondaryPeerSubnet "10.0.0.4/30" -PeerAsn 1234 -VlanId 100 -SharedKey "A1B2C3D4"
   ```
     
   > [!IMPORTANT]
   > Compruebe que especifica su número AS como ASN de emparejamiento, no como cliente ASN.
   > 

### <a name="to-view-azure-private-peering-details"></a>Visualización de los detalles del emparejamiento privado

Puede ver los detalles de la configuración mediante el siguiente cmdlet:

```powershell
Get-AzureBGPPeering -AccessType Private -ServiceKey "*********************************"
```

Devuelve:

```
AdvertisedPublicPrefixes       : 
AdvertisedPublicPrefixesState  : Configured
AzureAsn                       : 12076
CustomerAutonomousSystemNumber : 
PeerAsn                        : 1234
PrimaryAzurePort               : 
PrimaryPeerSubnet              : 10.0.0.0/30
RoutingRegistryName            : 
SecondaryAzurePort             : 
SecondaryPeerSubnet            : 10.0.0.4/30
State                          : Enabled
VlanId                         : 100
```

### <a name="to-update-azure-private-peering-configuration"></a>Actualización del establecimiento de configuración del emparejamiento privado de Azure

Puede actualizar cualquier parte de la configuración mediante el siguiente cmdlet. En el ejemplo siguiente, se está actualizando el identificador de VLAN del circuito de 100 a 500.

```powershell
Set-AzureBGPPeering -AccessType Private -ServiceKey "*********************************" -PrimaryPeerSubnet "10.0.0.0/30" -SecondaryPeerSubnet "10.0.0.4/30" -PeerAsn 1234 -VlanId 500 -SharedKey "A1B2C3D4"
```

### <a name="to-delete-azure-private-peering"></a>Eliminación del emparejamiento privado de Azure

Puede quitar el establecimiento de configuración entre pares ejecutando el siguiente cmdlet. Tiene que asegurarse de que todas las redes virtuales se desvinculan del circuito ExpressRoute antes de ejecutar este cmdlet.

```powershell
Remove-AzureBGPPeering -AccessType Private -ServiceKey "*********************************"
```

## <a name="azure-public-peering"></a>Configuración entre pares públicos de Azure

En esta sección se proporcionan instrucciones sobre cómo crear, obtener, actualizar y eliminar la configuración del emparejamiento público de Azure para un circuito ExpressRoute.

> [!NOTE]
> El emparejamiento público de Azure está en desuso para circuitos nuevos.
>

### <a name="to-create-azure-public-peering"></a>Creación de un emparejamiento público de Azure

1. **Creación de un circuito ExpressRoute**

   Siga las instrucciones para crear un [circuito ExpressRoute](expressroute-howto-circuit-classic.md) y habilite su aprovisionamiento a través del proveedor de conectividad. Si el proveedor de conectividad ofrece servicios administrados de nivel 3, puede solicitarle que habilite la configuración entre pares públicos de Azure. En ese caso, no necesita seguir las instrucciones que aparecen en las secciones siguientes. Por otra parte, si su proveedor de conectividad no administra este enrutamiento, una vez que cree el circuito siga las instrucciones siguientes.
2. **Comprobación del circuito ExpressRoute para comprobar que está aprovisionado**

   En primer lugar tiene que comprobar si el circuito ExpressRoute tiene los estados Aprovisionado y Habilitado.

   ```powershell
   Get-AzureDedicatedCircuit -ServiceKey "*********************************"
   ```

   Devuelve:

   ```powershell
   Bandwidth                        : 200
   CircuitName                      : MyTestCircuit
   Location                         : Silicon Valley
   ServiceKey                       : *********************************
   ServiceProviderName              : equinix
   ServiceProviderProvisioningState : Provisioned
   Sku                              : Standard
   Status                           : Enabled
   ```
   
   Compruebe que el circuito se muestra como Aprovisionado y Habilitado. Si no es así, colabore con su proveedor de conectividad para obtener el circuito para establecer el estado y la condición necesarios para el circuito.

   ```powershell
   ServiceProviderProvisioningState : Provisioned
   Status                           : Enabled
   ```
4. **Establecimiento de la configuración entre pares públicos de Azure para el circuito.**
   
   Asegúrese de que tiene la siguiente información antes de empezar:
   
   * Una subred /30 para el vínculo principal. Tiene que ser un prefijo IPv4 público válido.
   * Una subred /30 para el vínculo secundario. Tiene que ser un prefijo IPv4 público válido.
   * Un identificador VLAN válido para establecer esta configuración entre pares. Asegúrese de que ningún otro emparejamiento en el circuito usa el mismo identificador de VLAN.
   * Número de sistema autónomo (AS) para la configuración entre pares. Puede usar 2 bytes o 4 bytes como números AS.
   * Un hash MD5, en caso de que haya decidido usarlo. **Opcional**.

   > [!IMPORTANT]
   > Asegúrese de especificar su número AS como ASN de emparejamiento y no como cliente ASN.
   >  
     
   Puede usar el siguiente ejemplo para configurar el emparejamiento público de Azure para su circuito:

   ```powershell
   New-AzureBGPPeering -AccessType Public -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -PeerAsn 1234 -VlanId 200
   ```
     
   Si desea usar un hash MD5, use el ejemplo siguiente para configurar el circuito:
     
   ```powershell
   New-AzureBGPPeering -AccessType Public -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -PeerAsn 1234 -VlanId 200 -SharedKey "A1B2C3D4"
   ```
     
### <a name="to-view-azure-public-peering-details"></a>Visualización de detalles de un emparejamiento público de Azure

Para ver los detalles de la configuración, use el siguiente cmdlet:

```powershell
Get-AzureBGPPeering -AccessType Public -ServiceKey "*********************************"
```

Devuelve:

```powershell
AdvertisedPublicPrefixes       : 
AdvertisedPublicPrefixesState  : Configured
AzureAsn                       : 12076
CustomerAutonomousSystemNumber : 
PeerAsn                        : 1234
PrimaryAzurePort               : 
PrimaryPeerSubnet              : 131.107.0.0/30
RoutingRegistryName            : 
SecondaryAzurePort             : 
SecondaryPeerSubnet            : 131.107.0.4/30
State                          : Enabled
VlanId                         : 200
```

### <a name="to-update-azure-public-peering-configuration"></a>Actualización del establecimiento de configuración del emparejamiento público de Azure

Puede actualizar cualquier parte de la configuración mediante el siguiente cmdlet. En este ejemplo, se va a actualizar el identificador de VLAN del circuito de 200 a 600.

```powershell
Set-AzureBGPPeering -AccessType Public -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -PeerAsn 1234 -VlanId 600 -SharedKey "A1B2C3D4"
```

Compruebe que el circuito se muestra como Aprovisionado y Habilitado. 
### <a name="to-delete-azure-public-peering"></a>Eliminación del emparejamiento público de Azure

Puede quitar la configuración de emparejamiento ejecutando el siguiente cmdlet:

```powershell
Remove-AzureBGPPeering -AccessType Public -ServiceKey "*********************************"
```

## <a name="microsoft-peering"></a>Emparejamiento de Microsoft

En esta sección se proporcionan instrucciones sobre cómo crear, obtener, actualizar y eliminar la configuración del emparejamiento de Microsoft para un circuito ExpressRoute. 

### <a name="to-create-microsoft-peering"></a>Creación del emparejamiento de Microsoft

1. **Creación de un circuito ExpressRoute**
  
   Siga las instrucciones para crear un [circuito ExpressRoute](expressroute-howto-circuit-classic.md) y habilite su aprovisionamiento a través del proveedor de conectividad. Si su proveedor de conectividad ofrece servicios administrados de nivel 3, puede solicitarle que habilite la configuración entre pares privados de Azure. En ese caso, no necesita seguir las instrucciones que aparecen en las secciones siguientes. Por otra parte, si su proveedor de conectividad no administra este enrutamiento, una vez que cree el circuito siga las instrucciones siguientes.
2. **Comprobación del circuito ExpressRoute para comprobar que está aprovisionado**

   Compruebe que el circuito se muestra como Aprovisionado y Habilitado. 
   
   ```powershell
   Get-AzureDedicatedCircuit -ServiceKey "*********************************"
   ```

   Devuelve:
   
   ```powershell
   Bandwidth                        : 200
   CircuitName                      : MyTestCircuit
   Location                         : Silicon Valley
   ServiceKey                       : *********************************
   ServiceProviderName              : equinix
   ServiceProviderProvisioningState : Provisioned
   Sku                              : Standard
   Status                           : Enabled
   ```
   
   Compruebe que el circuito se muestra como Aprovisionado y Habilitado. Si no es así, colabore con su proveedor de conectividad para obtener el circuito para establecer el estado y la condición necesarios para el circuito.

   ```powershell
   ServiceProviderProvisioningState : Provisioned
   Status                           : Enabled
   ```
3. **Establecimiento de la configuración entre pares de Microsoft para el circuito**
   
    Asegúrese de que tiene la siguiente información antes de empezar:
   
   * Una subred /30 para el vínculo principal. Debe ser un prefijo de IPv4 público válido que sea de su propiedad y esté registrado en un Registro regional de Internet (RIR) o un Registro de enrutamiento de Internet (IRR).
   * Una subred /30 para el vínculo secundario. Debe ser un prefijo de IPv4 público válido que sea de su propiedad y esté registrado en un Registro regional de Internet (RIR) o un Registro de enrutamiento de Internet (IRR).
   * Un identificador VLAN válido para establecer esta configuración entre pares. Asegúrese de que ningún otro emparejamiento en el circuito usa el mismo identificador de VLAN.
   * Número de sistema autónomo (AS) para la configuración entre pares. Puede usar 2 bytes o 4 bytes como números AS.
   * Prefijos anunciados: tiene que proporcionar una lista de todos los prefijos que planea anunciar en la sesión BGP. Se aceptan solo prefijos de direcciones IP públicas. Puede enviar una lista separada por comas si tiene pensado enviar un conjunto de prefijos. Estos prefijos tienen que estar registrados a su nombre en un Registro regional de Internet (RIR) o un Registro de enrutamiento de Internet (IRR).
   * Cliente ASN: si anuncia prefijos que no están registrados en el número de AS de emparejamiento, puede especificar el número de AS en el que están registrados. **Opcional**.
   * Nombre del enrutamiento del Registro: puede especificar el RIR o TIR en el que están registrados el número AS y los prefijos.
   * Un hash MD5, en caso de que haya decidido usarlo. **Opcional.**
     
   Ejecute el siguiente cmdlet para configurar el emparejamiento de Microsoft para el circuito:
 
   ```powershell
   New-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -VlanId 300 -PeerAsn 1234 -CustomerAsn 2245 -AdvertisedPublicPrefixes "123.0.0.0/30" -RoutingRegistryName "ARIN" -SharedKey "A1B2C3D4"
   ```

### <a name="to-view-microsoft-peering-details"></a>Visualización de detalles del emparejamiento de Microsoft

Puede ver los detalles de la configuración mediante el siguiente cmdlet:

```powershell
Get-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************"
```
Devuelve:

```powershell
AdvertisedPublicPrefixes       : 123.0.0.0/30
AdvertisedPublicPrefixesState  : Configured
AzureAsn                       : 12076
CustomerAutonomousSystemNumber : 2245
PeerAsn                        : 1234
PrimaryAzurePort               : 
PrimaryPeerSubnet              : 10.0.0.0/30
RoutingRegistryName            : ARIN
SecondaryAzurePort             : 
SecondaryPeerSubnet            : 10.0.0.4/30
State                          : Enabled
VlanId                         : 300
```

### <a name="to-update-microsoft-peering-configuration"></a>Actualización de la configuración de emparejamiento de Microsoft

Puede actualizar cualquier parte de la configuración mediante el siguiente cmdlet:

```powershell
Set-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -VlanId 300 -PeerAsn 1234 -CustomerAsn 2245 -AdvertisedPublicPrefixes "123.0.0.0/30" -RoutingRegistryName "ARIN" -SharedKey "A1B2C3D4"
```

### <a name="to-delete-microsoft-peering"></a>Eliminación del emparejamiento de Microsoft

Puede quitar la configuración de emparejamiento ejecutando el siguiente cmdlet:

```powershell
Remove-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************"
```

## <a name="next-steps"></a>Pasos siguientes

A continuación, [Vinculación de una red virtual a un circuito ExpressRoute](expressroute-howto-linkvnet-classic.md)

* Para obtener más información sobre los flujos de trabajo, consulte [Flujos de trabajo de ExpressRoute](expressroute-workflows.md).
* Para más información sobre el emparejamiento de circuitos, vea [Circuitos y dominios de enrutamiento de ExpressRoute](expressroute-circuit-peerings.md).
