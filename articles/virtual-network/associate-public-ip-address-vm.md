---
title: Asociar una dirección IP pública a una máquina virtual
titlesuffix: Azure Virtual Network
description: Obtenga información sobre cómo asociar una dirección IP pública a una máquina virtual (VM) mediante Azure Portal o la CLI de Azure.
services: virtual-network
documentationcenter: ''
author: asudbring
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/21/2019
ms.author: allensu
ms.openlocfilehash: 9b5fc9e4118d98905138b7f205f61d85a96b60b0
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88035474"
---
# <a name="associate-a-public-ip-address-to-a-virtual-machine"></a>Asociar una dirección IP pública a una máquina virtual

En este artículo, obtendrá información sobre cómo asociar una dirección IP pública a una máquina virtual (VM) existente. Si quiere conectarse a una VM desde Internet, la VM debe tener una dirección IP pública asociada. Si quiere crear una nueva VM con una dirección IP pública, puede hacerlo mediante [Azure Portal](virtual-network-deploy-static-pip-arm-portal.md), la [interfaz de la línea de comandos (CLI) de Azure](virtual-network-deploy-static-pip-arm-cli.md) o [PowerShell](virtual-network-deploy-static-pip-arm-ps.md). Las direcciones IP públicas tienen un precio simbólico. Para obtener información detallada, consulte [Precios](https://azure.microsoft.com/pricing/details/ip-addresses/). Existe un límite para el número de direcciones IP públicas que puede usar por suscripción. Para obtener información detallada, consulte [Límites](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#publicip-address).

Puede usar [Azure Portal](#azure-portal), la [interfaz de la línea de comandos](#azure-cli) (CLI) de Azure o [PowerShell](#powershell) para asociar una dirección IP pública a una VM.

## <a name="azure-portal"></a>Portal de Azure

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Busque o vaya a la máquina virtual a la que quiere agregar la dirección IP pública y, a continuación, selecciónela.
3. En **Configuración**, seleccione **Redes** y, a continuación, seleccione la interfaz de red a la que quiere agregar la dirección IP pública, como se muestra en la siguiente imagen:

   ![Selección de interfaz de red](./media/associate-public-ip-address-vm/select-nic.png)

   > [!NOTE]
   > Las direcciones IP públicas están asociadas a las interfaces de red conectadas a una VM. En la imagen anterior, la VM solo tiene una interfaz de red. Si la VM tiene varias interfaces de red, aparecerán todas y deberá seleccionar la interfaz de red a la que quiere asociar la dirección IP pública.

4. Seleccione **Configuraciones de IP** y, a continuación, seleccione una configuración de IP, tal como se muestra en la siguiente imagen:

   ![Selección de la configuración de IP](./media/associate-public-ip-address-vm/select-ip-configuration.png)

   > [!NOTE]
   > Las direcciones IP públicas están asociadas a configuraciones de IP para una interfaz de red. En la imagen anterior, la interfaz de red tiene una configuración de IP. Si la interfaz de red tiene varias configuraciones de IP, aparecerán todas en la lista y deberá seleccionar la configuración de IP a la que quiere asociar la dirección IP pública.

5. Seleccione **Habilitado** y, a continuación, seleccione **Dirección IP (*Configurar los valores obligatorios*)** . Elija una dirección IP pública existente, que cerrará automáticamente el cuadro **Elegir dirección IP pública**. Si no hay ninguna dirección IP pública en la lista, debe crear una. Para obtener información, consulte [Crear una dirección IP pública](virtual-network-public-ip-address.md#create-a-public-ip-address). Seleccione **Guardar**, tal como se muestra en la imagen que sigue y, a continuación, cierre el cuadro de la configuración de IP.

   ![Habilitación de la dirección IP pública](./media/associate-public-ip-address-vm/enable-public-ip-address.png)

   > [!NOTE]
   > Las direcciones IP públicas que aparecen son aquellas que existen en la misma región que la VM. Si tiene varias direcciones IP públicas creadas en la región, aparecerán todas aquí. Si alguna aparece atenuada, es porque la dirección ya está asociada a otro recurso.

6. Vea la dirección IP pública asignada a la configuración de IP, como se muestra en la imagen siguiente. Puede tardar unos segundos en aparecer una dirección IP.

   ![Visualización de una dirección IP pública asignada](./media/associate-public-ip-address-vm/view-assigned-public-ip-address.png)

   > [!NOTE]
   > Las direcciones se asignan desde un grupo de direcciones que se usa en cada región de Azure. Para ver una lista de los grupos de direcciones que se usan en cada región, consulte [Microsoft Azure Datacenter IP Ranges](https://www.microsoft.com/download/details.aspx?id=41653) (Intervalos IP del centro de datos de Microsoft Azure). La dirección asignada puede ser cualquiera de los grupos usados para la región. Si necesita que la dirección se asigne desde un grupo específico de la región, use un [prefijo de dirección IP pública](public-ip-address-prefix.md).

7. [Permita el tráfico de red en la VM](#allow-network-traffic-to-the-vm) con reglas de seguridad en un grupo de seguridad de red.

## <a name="azure-cli"></a>Azure CLI

Instale la [CLI de Azure](/cli/azure/install-azure-cli?toc=%2fazure%2fvirtual-network%2ftoc.json) o use Azure Cloud Shell. Azure Cloud Shell es un shell de Bash gratuito que se puede ejecutar directamente en Azure Portal. Tiene la CLI de Azure preinstalada y configurada para utilizarla con la cuenta. Seleccione el botón **Probar** en los comandos siguientes de la CLI. Al seleccionar **Probar**, se invoca una instancia de Cloud Shell con la que puede iniciar sesión en la cuenta de Azure.

1. Si usa la CLI localmente en Bash, inicie sesión en Azure con `az login`.
2. Una dirección IP pública está asociada a una configuración de IP de una interfaz de red conectada a una VM. Use el comando [az network nic-ip-config update](/cli/azure/network/nic/ip-config?view=azure-cli-latest#az-network-nic-ip-config-update) para asociar una dirección IP pública a una configuración de IP. El siguiente ejemplo asocia una dirección IP pública denominada *myVMPublicIP*a la configuración de IP *ipconfigmyVM* de una interfaz de red existente *myVMVMNic* que existe en un grupo de recursos denominado *myResourceGroup*.
  
   ```azurecli-interactive
   az network nic ip-config update \
     --name ipconfigmyVM \
     --nic-name myVMVMNic \
     --resource-group myResourceGroup \
     --public-ip-address myVMPublicIP
   ```

   - Si no tiene una dirección IP pública existente, use el comando [az network public-ip create](/cli/azure/network/public-ip?view=azure-cli-latest#az-network-public-ip-create) para crear una. Por ejemplo, el comando siguiente crea una dirección IP pública denominada *myVMPublicIP* en el grupo de recursos denominado *myResourceGroup*.
  
     ```azurecli-interactive
     az network public-ip create --name myVMPublicIP --resource-group myResourceGroup
     ```

     > [!NOTE]
     > El comando anterior crea una dirección IP pública con valores predeterminados para varias configuraciones que es posible que quiera personalizar. Para más información sobre la configuración de las direcciones IP públicas, consulte [Crear una dirección IP pública](virtual-network-public-ip-address.md#create-a-public-ip-address). Las direcciones se asignan desde un grupo de direcciones IP públicas que se usa para cada región de Azure. Para ver una lista de los grupos de direcciones que se usan en cada región, consulte [Microsoft Azure Datacenter IP Ranges](https://www.microsoft.com/download/details.aspx?id=41653) (Intervalos IP del centro de datos de Microsoft Azure).

   - Si no conoce el nombre de una interfaz de red conectada a la VM, use el comando [az vm nic list](/cli/azure/vm/nic?view=azure-cli-latest#az-vm-nic-list) para verla. Por ejemplo, el comando siguiente enumera los nombres de las interfaces de red conectadas a una VM denominada *myVM* en un grupo de recursos denominado *myResourceGroup*:

     ```azurecli-interactive
     az vm nic list --vm-name myVM --resource-group myResourceGroup
     ```

     La salida incluye una o varias líneas que son similares al ejemplo siguiente:
  
     ```
     "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic",
     ```

     En el ejemplo anterior, *myVMVMNic* es el nombre de la interfaz de red.

   - Si no conoce el nombre de una configuración de IP para una interfaz de red, use el comando [az network nic ip-config list](/cli/azure/network/nic/ip-config?view=azure-cli-latest#az-network-nic-ip-config-list) para recuperarla. Por ejemplo, el comando siguiente enumera los nombres de las configuraciones de IP para una interfaz de red denominada *myVMVMNic* en un grupo de recursos denominado *myResourceGroup*:

     ```azurecli-interactive
     az network nic ip-config list --nic-name myVMVMNic --resource-group myResourceGroup --out table
     ```

3. Vea la dirección IP pública asignada a la configuración de IP con el comando [az vm list-ip-addresses](/cli/azure/vm?view=azure-cli-latest#az-vm-list-ip-addresses). En el ejemplo siguiente se muestran las direcciones IP asignadas a una VM existente denominada *myVM* en un grupo de recursos denominado *myResourceGroup*.

   ```azurecli-interactive
   az vm list-ip-addresses --name myVM --resource-group myResourceGroup --out table
   ```

   > [!NOTE]
   > Las direcciones se asignan desde un grupo de direcciones que se usa en cada región de Azure. Para ver una lista de los grupos de direcciones que se usan en cada región, consulte [Microsoft Azure Datacenter IP Ranges](https://www.microsoft.com/download/details.aspx?id=41653) (Intervalos IP del centro de datos de Microsoft Azure). La dirección asignada puede ser cualquiera de los grupos usados para la región. Si necesita que la dirección se asigne desde un grupo específico de la región, use un [prefijo de dirección IP pública](public-ip-address-prefix.md).

4. [Permita el tráfico de red en la VM](#allow-network-traffic-to-the-vm) con reglas de seguridad en un grupo de seguridad de red.

## <a name="powershell"></a>PowerShell

Instale [PowerShell](/powershell/azure/install-az-ps) o use Azure Cloud Shell. Azure Cloud Shell es un shell gratuito que se ejecuta directamente en Azure Portal. Tiene preinstalado y configurado PowerShell para usarlo con su cuenta. Seleccione el botón **Probar** en los comandos siguientes de PowerShell. Al seleccionar **Probar**, se invoca una instancia de Cloud Shell con la que puede iniciar sesión en la cuenta de Azure.

1. Si usa PowerShell localmente, inicie sesión en Azure con `Connect-AzAccount`.
2. Una dirección IP pública está asociada a una configuración de IP de una interfaz de red conectada a una VM. Use los comandos [Get-AzVirtualNetwork](/powershell/module/Az.Network/Get-AzVirtualNetwork) y [Get-AzVirtualNetworkSubnetConfig](/powershell/module/Az.Network/Get-AzVirtualNetworkSubnetConfig) para obtener la red virtual y la subred en las que se encuentra la interfaz de red. A continuación, use el comando [Get-AzNetworkInterface](/powershell/module/Az.Network/Get-AzNetworkInterface) para obtener una interfaz de red y el comando [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) para obtener una dirección IP pública existente. A continuación, use el comando [Set-AzNetworkInterfaceIpConfig](/powershell/module/Az.Network/Set-AzNetworkInterfaceIpConfig) para asociar la dirección IP pública a la configuración de IP y el comando [Set-AzNetworkInterface](/powershell/module/Az.Network/Set-AzNetworkInterface) para escribir la nueva configuración de IP para el interfaz de red.

   El siguiente ejemplo asocia una dirección IP pública denominada *myVMPublicIP* a la configuración de IP *ipconfigmyVM* de una interfaz de red existente denominada *myVMVMNic* que existe en una subred denominada *myVMSubnet* en una red virtual denominada *myVMVNet*. Todos los recursos están en un grupo de recursos denominado *myResourceGroup*.
  
   ```azurepowershell-interactive
   $vnet = Get-AzVirtualNetwork -Name myVMVNet -ResourceGroupName myResourceGroup
   $subnet = Get-AzVirtualNetworkSubnetConfig -Name myVMSubnet -VirtualNetwork $vnet
   $nic = Get-AzNetworkInterface -Name myVMVMNic -ResourceGroupName myResourceGroup
   $pip = Get-AzPublicIpAddress -Name myVMPublicIP -ResourceGroupName myResourceGroup
   $nic | Set-AzNetworkInterfaceIpConfig -Name ipconfigmyVM -PublicIPAddress $pip -Subnet $subnet
   $nic | Set-AzNetworkInterface
   ```

   - Si no tiene una dirección IP pública existente, use el comando [New-AzPublicIpAddress](/powershell/module/Az.Network/New-AzPublicIpAddress) para crear una. Por ejemplo, el comando siguiente crea una dirección IP pública *dinámica* denominada *myVMPublicIP* en el grupo de recursos denominado *myResourceGroup* en la región *eastus*.
  
     ```azurepowershell-interactive
     New-AzPublicIpAddress -Name myVMPublicIP -ResourceGroupName myResourceGroup -AllocationMethod Dynamic -Location eastus
     ```

     > [!NOTE]
     > El comando anterior crea una dirección IP pública con valores predeterminados para varias configuraciones que es posible que quiera personalizar. Para más información sobre la configuración de las direcciones IP públicas, consulte [Crear una dirección IP pública](virtual-network-public-ip-address.md#create-a-public-ip-address). Las direcciones se asignan desde un grupo de direcciones IP públicas que se usa para cada región de Azure. Para ver una lista de los grupos de direcciones que se usan en cada región, consulte [Microsoft Azure Datacenter IP Ranges](https://www.microsoft.com/download/details.aspx?id=41653) (Intervalos IP del centro de datos de Microsoft Azure).

   - Si no conoce el nombre de una interfaz de red conectada a la VM, use el comando [Get-AzVM](/powershell/module/Az.Compute/Get-AzVM) para verla. Por ejemplo, el comando siguiente enumera los nombres de las interfaces de red conectadas a una VM denominada *myVM* en un grupo de recursos denominado *myResourceGroup*:

     ```azurepowershell-interactive
     $vm = Get-AzVM -name myVM -ResourceGroupName myResourceGroup
     $vm.NetworkProfile
     ```

     La salida incluye una o varias líneas que son similares al ejemplo siguiente. En la salida de ejemplo, *myVMVMNic* es el nombre de la interfaz de red.
  
     ```
     "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic",
     ```

   - Si no conoce el nombre de la red virtual o subred en la que se encuentra la interfaz de red, use el comando `Get-AzNetworkInterface` para ver la información. Por ejemplo, el comando siguiente obtiene la información de la red virtual y la subred para una interfaz de red denominada *myVMVMNic* en un grupo de recursos denominado *myResourceGroup*:

     ```azurepowershell-interactive
     $nic = Get-AzNetworkInterface -Name myVMVMNic -ResourceGroupName myResourceGroup
     $ipConfigs = $nic.IpConfigurations
     $ipConfigs.Subnet | Select Id
     ```

     La salida incluye una o varias líneas que son similares al ejemplo siguiente. En la salida del ejemplo, *myVMVNET* es el nombre de la red virtual y *myVMSubnet* es el nombre de la subred.
  
     ```
     "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVMVNET/subnets/myVMSubnet",
     ```

   - Si no conoce el nombre de una configuración de IP para una interfaz de red, use el comando [Get-AzNetworkInterface](/powershell/module/Az.Network/Get-AzNetworkInterface) para recuperarla. Por ejemplo, el comando siguiente enumera los nombres de las configuraciones de IP para una interfaz de red denominada *myVMVMNic* en un grupo de recursos denominado *myResourceGroup*:

     ```azurepowershell-interactive
     $nic = Get-AzNetworkInterface -Name myVMVMNic -ResourceGroupName myResourceGroup
     $nic.IPConfigurations
     ```

     La salida incluye una o varias líneas que son similares al ejemplo siguiente. En la salida del ejemplo, *ipconfigmyVM* es el nombre de una configuración de IP.
  
     ```
     Id     : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic/ipConfigurations/ipconfigmyVM
     ```

3. Vea la dirección IP pública asignada a la configuración de IP con el comando [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress). El ejemplo siguiente muestra la dirección asignada a una dirección IP pública denominada *myVMPublicIP* en el grupo de recursos denominado *myResourceGroup*.

   ```azurepowershell-interactive
   Get-AzPublicIpAddress -Name myVMPublicIP -ResourceGroupName myResourceGroup | Select IpAddress
   ```

   Si no conoce el nombre de la dirección IP pública asignada a una configuración de IP, ejecute los siguientes comandos para obtenerlo:

   ```azurepowershell-interactive
   $nic = Get-AzNetworkInterface -Name myVMVMNic -ResourceGroupName myResourceGroup
   $nic.IPConfigurations
   $address = $nic.IPConfigurations.PublicIpAddress
   $address | Select Id
   ```

   La salida incluye una o varias líneas que son similares al ejemplo siguiente. En la salida del ejemplo, *myVMPublicIP* es el nombre de la dirección IP pública asignada a la configuración de IP.

   ```
   "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myVMPublicIP"
   ```

   > [!NOTE]
   > Las direcciones se asignan desde un grupo de direcciones que se usa en cada región de Azure. Para ver una lista de los grupos de direcciones que se usan en cada región, consulte [Microsoft Azure Datacenter IP Ranges](https://www.microsoft.com/download/details.aspx?id=41653) (Intervalos IP del centro de datos de Microsoft Azure). La dirección asignada puede ser cualquiera de los grupos usados para la región. Si necesita que la dirección se asigne desde un grupo específico de la región, use un [prefijo de dirección IP pública](public-ip-address-prefix.md).

4. [Permita el tráfico de red en la VM](#allow-network-traffic-to-the-vm) con reglas de seguridad en un grupo de seguridad de red.

## <a name="allow-network-traffic-to-the-vm"></a>Permitir tráfico de red en la VM

Para poder conectarse a la dirección IP pública desde Internet, asegúrese de que tiene los puertos necesarios abiertos en cualquier grupo de seguridad de red que haya asociado a la interfaz de red o la subred en la que se encuentra la interfaz de red. Aunque los grupo de seguridad de red filtran el tráfico en la dirección IP privada de la interfaz de red, una vez que el tráfico entrante llega a la dirección IP pública, Azure traduce la dirección pública a la dirección IP privada, por lo que, si un grupo de seguridad de red evita el flujo de tráfico, la comunicación con la dirección IP pública produce un error. Puede ver las reglas de seguridad vigentes para una interfaz de red y su subred mediante [Portal](diagnose-network-traffic-filter-problem.md#diagnose-using-azure-portal), la [CLI](diagnose-network-traffic-filter-problem.md#diagnose-using-azure-cli) o [PowerShell](diagnose-network-traffic-filter-problem.md#diagnose-using-powershell).

## <a name="next-steps"></a>Pasos siguientes

Permita el tráfico entrante de Internet a la VM con un grupo de seguridad de red. Para obtener información sobre cómo crear un grupo de seguridad de red, consulte [Trabajar con grupos de seguridad de red](manage-network-security-group.md#work-with-network-security-groups). Para obtener más información sobre los grupos de seguridad de red, consulte [Grupos de seguridad](security-overview.md).
