---
title: 'Azure ExpressRoute: Configuración de MACsec'
description: Este artículo le ayuda a configurar MACsec para proteger las conexiones entre los enrutadores perimetrales y los enrutadores perimetrales de Microsoft.
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: how-to
ms.date: 10/22/2019
ms.author: cherylmc
ms.openlocfilehash: ce2e4ca2cad0fc080d6a750b3f1682210a5b9b20
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87387666"
---
# <a name="configure-macsec-on-expressroute-direct-ports"></a>Configuración de MACsec para los puertos de ExpressRoute Direct

Este artículo le ayuda a configurar MACsec para proteger las conexiones entre los enrutadores perimetrales y los enrutadores perimetrales de Microsoft mediante PowerShell.

## <a name="before-you-begin"></a>Antes de empezar

Antes de comenzar la configuración, confirme lo siguiente:

* Conoce los [flujos de trabajo del aprovisionamiento de ExpressRoute Direct](expressroute-erdirect-about.md).
* Ha creado un [recurso de puerto de ExpressRoute Direct](expressroute-howto-erdirect.md).
* Si quiere ejecutar PowerShell localmente, compruebe que la versión más reciente de Azure PowerShell está instalada en el equipo.

### <a name="working-with-azure-powershell"></a>Trabajo con Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

[!INCLUDE [expressroute-cloudshell](../../includes/expressroute-cloudshell-powershell-about.md)]

### <a name="sign-in-and-select-the-right-subscription"></a>Inicio de sesión y selección de la suscripción correcta

Para iniciar la configuración, inicie sesión en su cuenta de Azure y seleccione la suscripción que quiere usar.

   [!INCLUDE [sign in](../../includes/expressroute-cloud-shell-connect.md)]

## <a name="1-create-azure-key-vault-macsec-secrets-and-user-identity"></a>1. Crear Azure Key Vault, secretos de MACsec e identidad del usuario

1. Cree una instancia de Key Vault para almacenar los secretos de MACsec en un nuevo grupo de recursos.

    ```azurepowershell-interactive
    New-AzResourceGroup -Name "your_resource_group" -Location "resource_location"
    $keyVault = New-AzKeyVault -Name "your_key_vault_name" -ResourceGroupName "your_resource_group" -Location "resource_location" -EnableSoftDelete 
    ```

    Si ya tiene un grupo de recursos o un almacén de claves, puede reutilizarlos. Sin embargo, es fundamental que habilite la característica de [**eliminación temporal**](../key-vault/general/soft-delete-overview.md) en el almacén de claves existente. Si la eliminación temporal no está habilitada, ejecute los siguientes comandos para habilitarla:

    ```azurepowershell-interactive
    ($resource = Get-AzResource -ResourceId (Get-AzKeyVault -VaultName "your_existing_keyvault").ResourceId).Properties | Add-Member -MemberType "NoteProperty" -Name "enableSoftDelete" -Value "true"
    Set-AzResource -resourceid $resource.ResourceId -Properties $resource.Properties
    ```
2. Cree una identidad de usuario.

    ```azurepowershell-interactive
    $identity = New-AzUserAssignedIdentity  -Name "identity_name" -Location "resource_location" -ResourceGroupName "your_resource_group"
    ```

    Si New-AzUserAssignedIdentity no se reconoce como un cmdlet de PowerShell válido, instale el siguiente módulo (en modo de administrador) y vuelva a ejecutar el comando anterior.

    ```azurepowershell-interactive
    Install-Module -Name Az.ManagedServiceIdentity
    ```
3. Cree una clave de asociación de conectividad (CAK) y un nombre de clave de asociación de conectividad (CKN), y almacénelos en el almacén de claves.

    ```azurepowershell-interactive
    $CAK = ConvertTo-SecureString "your_key" -AsPlainText -Force
    $CKN = ConvertTo-SecureString "your_key_name" -AsPlainText -Force
    $MACsecCAKSecret = Set-AzKeyVaultSecret -VaultName "your_key_vault_name" -Name "CAK_name" -SecretValue $CAK
    $MACsecCKNSecret = Set-AzKeyVaultSecret -VaultName "your_key_vault_name" -Name "CKN_name" -SecretValue $CKN
    ```
4. Asigne el permiso GET a la identidad del usuario.

    ```azurepowershell-interactive
    Set-AzKeyVaultAccessPolicy -VaultName "your_key_vault_name" -PermissionsToSecrets get -ObjectId $identity.PrincipalId
    ```

   Ahora esta identidad puede obtener los secretos (por ejemplo, CAK y CKN) del almacén de claves.
5. Establezca esta identidad de usuario para que ExpressRoute la use.

    ```azurepowershell-interactive
    $erIdentity = New-AzExpressRoutePortIdentity -UserAssignedIdentityId $identity.Id
    ```
 
## <a name="2-configure-macsec-on-expressroute-direct-ports"></a>2. Configuración de MACsec para los puertos de ExpressRoute Direct

### <a name="to-enable-macsec"></a>Para habilitar MACsec

Cada instancia de ExpressRoute Direct tiene dos puertos físicos. Puede optar por habilitar MACsec en ambos puertos al mismo tiempo o en uno cada vez. Hacer un puerto a la vez (cambiando el tráfico a un puerto activo mientras se atiende el otro) puede ayudar a minimizar la interrupción si ExpressRoute Direct ya está en servicio.

1. Establezca los secretos y el cifrado de MACsec, y asocie la identidad del usuario al puerto para que el código de administración de ExpressRoute pueda acceder a los secretos de MACsec si fuera necesario.

    ```azurepowershell-interactive
    $erDirect = Get-AzExpressRoutePort -ResourceGroupName "your_resource_group" -Name "your_direct_port_name"
    $erDirect.Links[0]. MacSecConfig.CknSecretIdentifier = $MacSecCKNSecret.Id
    $erDirect.Links[0]. MacSecConfig.CakSecretIdentifier = $MacSecCAKSecret.Id
    $erDirect.Links[0]. MacSecConfig.Cipher = "GcmAes256"
    $erDirect.Links[1]. MacSecConfig.CknSecretIdentifier = $MacSecCKNSecret.Id
    $erDirect.Links[1]. MacSecConfig.CakSecretIdentifier = $MacSecCAKSecret.Id
    $erDirect.Links[1]. MacSecConfig.Cipher = "GcmAes256"
    $erDirect.identity = $erIdentity
    Set-AzExpressRoutePort -ExpressRoutePort $erDirect
    ```
2. (Opcional) Si los puertos están en estado administrativo de fuera de servicio, puede ejecutar los siguientes comandos para abrir los puertos.

    ```azurepowershell-interactive
    $erDirect = Get-AzExpressRoutePort -ResourceGroupName "your_resource_group" -Name "your_direct_port_name"
    $erDirect.Links[0].AdminState = "Enabled"
    $erDirect.Links[1].AdminState = "Enabled"
    Set-AzExpressRoutePort -ExpressRoutePort $erDirect
    ```

    En este momento, MACsec está habilitado en los puertos de ExpressRoute Direct, en Microsoft. Si no lo ha configurado en los dispositivos perimetrales, puede continuar con la configuración con los mismos secretos y cifrado de MACsec.

### <a name="to-disable-macsec"></a>Para deshabilitar MACsec

Si ya no desea que MACsec se use en la instancia de ExpressRoute Direct, puede ejecutar los siguientes comandos para deshabilitarlo.

```azurepowershell-interactive
$erDirect = Get-AzExpressRoutePort -ResourceGroupName "your_resource_group" -Name "your_direct_port_name"
$erDirect.Links[0]. MacSecConfig.CknSecretIdentifier = $null
$erDirect.Links[0]. MacSecConfig.CakSecretIdentifier = $null
$erDirect.Links[1]. MacSecConfig.CknSecretIdentifier = $null
$erDirect.Links[1]. MacSecConfig.CakSecretIdentifier = $null
$erDirect.identity = $null
Set-AzExpressRoutePort -ExpressRoutePort $erDirect
```

En este momento, MACsec se habilita en los puertos de ExpressRoute Direct, en Microsoft.

### <a name="test-connectivity"></a>Comprobación de la conectividad
Después de configurar MACsec (incluida la actualización de la clave MACsec) en los puertos ExpressRoute Direct, [compruebe](expressroute-troubleshooting-expressroute-overview.md) si las sesiones de BGP de los circuitos están en funcionamiento. Si aún no tiene ningún circuito en los puertos, cree uno primero y configure el emparejamiento privado de Azure o el emparejamiento de Microsoft del circuito. Si MACsec no está configurado correctamente, lo que incluye la falta de coincidencia de la clave MACsec, entre los dispositivos de red y los dispositivos de red de Microsoft, no verá la resolución ARP en el establecimiento de la capa 2 y el BGP en el nivel 3. Si todo está configurado correctamente, debería ver las rutas BGP anunciadas correctamente en ambas direcciones y en el flujo de datos de la aplicación en consecuencia en ExpressRoute.

## <a name="next-steps"></a>Pasos siguientes
1. [Creación de un circuito ExpressRoute en ExpressRoute Direct](expressroute-howto-erdirect.md)
2. [Vinculación de un circuito ExpressRoute a una red virtual de Azure](expressroute-howto-linkvnet-arm.md)
3. [Comprobación de la conectividad de ExpressRoute](expressroute-troubleshooting-expressroute-overview.md)
