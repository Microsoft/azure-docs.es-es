---
title: Configuración de firewalls y redes virtuales de Azure Key Vault
description: Instrucciones detalladas paso a paso para configurar los firewalls y las redes virtuales de Azule Key Vault
services: key-vault
author: ShaneBala-keyvault
manager: ravijan
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 04/22/2020
ms.author: sudbalas
ms.custom: devx-track-azurecli
ms.openlocfilehash: d416c6fdef85b83d91ca0e341659f87618d268e0
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87504524"
---
# <a name="configure-azure-key-vault-firewalls-and-virtual-networks"></a>Configuración de firewalls y redes virtuales de Azure Key Vault

En este artículo se describen las instrucciones paso a paso para configurar los firewalls y las redes virtuales de Azure Key Vault para restringir el acceso al almacén de claves. Los [puntos de conexión de servicio de red virtual para Key Vault](overview-vnet-service-endpoints.md) le permiten restringir el acceso a la red virtual especificada y establecer intervalos de direcciones IPv4 (versión 4 del protocolo de Internet).

> [!IMPORTANT]
> Una vez que las reglas del firewall están en vigor, los usuarios solo pueden realizar operaciones del [plano de datos](secure-your-key-vault.md#data-plane-access-control) de Key Vault cuando las solicitudes se originan desde redes virtuales o rangos de direcciones IPv4 permitidos. Esto también se aplica al acceso de Key Vault desde Azure Portal. Aunque los usuarios pueden ir a un almacén de claves desde Azure Portal, es posible que no pueda enumerar las claves, los secretos o los certificados si su equipo cliente no está en la lista de dispositivos permitidos. Esto también afecta al selector de Key Vault de otros servicios de Azure. Los usuarios podrían ver la lista de almacenes de claves, pero no enumerar las claves si las reglas del firewall limitan su equipo cliente.

> [!NOTE]
> Tenga en cuenta las siguientes limitaciones de configuración:
> * Se permite un máximo de 127 reglas de red virtual y 127 reglas de IPv4. 
> * Los rangos de dirección pequeños con tamaños de prefijos "/31" o "/32" no son compatibles. En su lugar, configure estos rangos utilizando reglas de direcciones IP individuales.
> * Las reglas de red IP solo se permiten para direcciones IP públicas. No se permiten intervalos de direcciones IP reservados para redes privadas (tal y como se define en RFC 1918) en las reglas IP. Las redes privadas incluyen direcciones que comienzan por **10.** , **172.16-31** y **192.168.** . 
> * Solo se admiten direcciones IPV4 en este momento.

## <a name="use-the-azure-portal"></a>Uso de Azure Portal

Así es cómo puede configurar firewalls y redes virtuales de Key Vault mediante Azure Portal:

1. Vaya al almacén de claves que quiere proteger.
2. Seleccione **Redes** y, luego, elija la pestaña **Firewalls and virtual networks** (Firewalls y redes virtuales).
3. Haga clic en **Redes seleccionadas** en **Permitir el acceso desde**.
4. Para agregar redes virtuales existentes a los firewall y las reglas de red virtual, haga clic en **+ Agregar redes virtuales existentes**.
5. En la nueva hoja que se abrirá, seleccione la suscripción, las redes virtuales y las subredes a las cuales desea otorgar acceso a este almacén de claves. Si las redes virtuales y subredes que seleccionó no tienen puntos de conexión de servicio habilitados, confirme que desea habilitar los puntos de conexión de servicio y seleccione **Habilitar**. Esta operación podría tardar hasta 15 minutos en surtir efecto.
6. En **Redes IP**, agregue rangos de direcciones IPv4; para ello, escriba los rangos de direcciones IPv4 en la [notación CIDR (enrutamiento de interdominios sin clases)](https://tools.ietf.org/html/rfc4632) o en las direcciones IP individuales.
7. Si desea permitir que los servicios de confianza de Microsoft omitan el firewall de Key Vault, seleccione "Sí". Para ver una lista completa de los servicios de confianza Key Vault actuales, consulte el siguiente vínculo. [Servicios de confianza de Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview-vnet-service-endpoints#trusted-services)
7. Seleccione **Guardar**.

Asimismo, puede también agregar nuevas redes virtuales y subredes y habilitar los puntos de conexión de servicio de las redes virtuales y subredes recién creadas. Para ello, haga clic en **+ Agregar nueva red virtual**. A continuación, siga las indicaciones.

## <a name="use-the-azure-cli"></a>Uso de la CLI de Azure 

Estos son los pasos para configurar firewalls y redes virtuales de Key Vault mediante la CLI de Azure

1. [Instale la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli) e [inicie sesión](https://docs.microsoft.com/cli/azure/authenticate-azure-cli).

2. Se muestra la lista de reglas de red virtual disponibles. Si no ha establecido ninguna regla para este almacén de claves, la lista estará vacía.
   ```azurecli
   az keyvault network-rule list --resource-group myresourcegroup --name mykeyvault
   ```

3. Habilite un punto de conexión de servicio para Key Vault en una red virtual y subred existentes.
   ```azurecli
   az network vnet subnet update --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --service-endpoints "Microsoft.KeyVault"
   ```

4. Agregue una regla de red para una red virtual y subred.
   ```azurecli
   subnetid=$(az network vnet subnet show --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --query id --output tsv)
   az keyvault network-rule add --resource-group "demo9311" --name "demo9311premium" --subnet $subnetid
   ```

5. Agregue un rango de direcciones IP desde el que se va a permitir el tráfico.
   ```azurecli
   az keyvault network-rule add --resource-group "myresourcegroup" --name "mykeyvault" --ip-address "191.10.18.0/24"
   ```

6. Si cualquier servicio de confianza necesita obtener acceso a este almacén de claves, establezca la opción `bypass` en `AzureServices`.
   ```azurecli
   az keyvault update --resource-group "myresourcegroup" --name "mykeyvault" --bypass AzureServices
   ```

7. Active las reglas de red estableciendo la acción predeterminada en `Deny`.
   ```azurecli
   az keyvault update --resource-group "myresourcegroup" --name "mekeyvault" --default-action Deny
   ```

## <a name="use-azure-powershell"></a>Uso de Azure PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Estas son las instrucciones para configurar los firewalls y las redes virtuales de Key Vault mediante PowerShell:

1. Instale la última versión de [Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps) e [inicie sesión](https://docs.microsoft.com/powershell/azure/authenticate-azureps).

2. Se muestra la lista de reglas de red virtual disponibles. Si no ha establecido ninguna regla para este almacén de claves, la lista estará vacía.
   ```powershell
   (Get-AzKeyVault -VaultName "mykeyvault").NetworkAcls
   ```

3. Habilite el punto de conexión de servicio para Key Vault en una red virtual y subred existentes.
   ```powershell
   Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Set-AzVirtualNetworkSubnetConfig -Name "mysubnet" -AddressPrefix "10.1.1.0/24" -ServiceEndpoint "Microsoft.KeyVault" | Set-AzVirtualNetwork
   ```

4. Agregue una regla de red para una red virtual y subred.
   ```powershell
   $subnet = Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Get-AzVirtualNetworkSubnetConfig -Name "mysubnet"
   Add-AzKeyVaultNetworkRule -VaultName "mykeyvault" -VirtualNetworkResourceId $subnet.Id
   ```

5. Agregue un rango de direcciones IP desde el que se va a permitir el tráfico.
   ```powershell
   Add-AzKeyVaultNetworkRule -VaultName "mykeyvault" -IpAddressRange "16.17.18.0/24"
   ```

6. Si cualquier servicio de confianza necesita obtener acceso a este almacén de claves, establezca la opción `bypass` en `AzureServices`.
   ```powershell
   Update-AzKeyVaultNetworkRuleSet -VaultName "mykeyvault" -Bypass AzureServices
   ```

7. Active las reglas de red estableciendo la acción predeterminada en `Deny`.
   ```powershell
   Update-AzKeyVaultNetworkRuleSet -VaultName "mykeyvault" -DefaultAction Deny
   ```

## <a name="references"></a>Referencias
* Referencia de plantilla de ARM: [Referencia de la plantilla de ARM de Azure Key Vault](https://docs.microsoft.com/azure/templates/Microsoft.KeyVault/vaults)
* Comandos de la CLI de Azure: [az keyvault network-rule](https://docs.microsoft.com/cli/azure/keyvault/network-rule?view=azure-cli-latest)
* Cmdlets de Azure PowerShell: [Get-AzKeyVault](https://docs.microsoft.com/powershell/module/az.keyvault/get-azkeyvault), [Add-AzKeyVaultNetworkRule](https://docs.microsoft.com/powershell/module/az.KeyVault/Add-azKeyVaultNetworkRule), [Remove-AzKeyVaultNetworkRule](https://docs.microsoft.com/powershell/module/az.KeyVault/Remove-azKeyVaultNetworkRule), [Update-AzKeyVaultNetworkRuleSet](https://docs.microsoft.com/powershell/module/az.KeyVault/Update-azKeyVaultNetworkRuleSet)

## <a name="next-steps"></a>Pasos siguientes

* [Puntos de conexión de servicio de red virtual para Key Vault](overview-vnet-service-endpoints.md)
* [Protección del almacén de claves](secure-your-key-vault.md)
