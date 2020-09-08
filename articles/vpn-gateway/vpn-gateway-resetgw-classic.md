---
title: Restablecimiento de una instancia de Azure VPN Gateway para volver a establecer túneles IPsec
description: Restablezca la instancia de Azure VPN Gateway para restablecer los túneles IPsec de las puertas de enlace de VPN en los modelos de implementación clásica y de Resource Manager.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 01/09/2020
ms.author: cherylmc
ms.openlocfilehash: 9f78b33a95001639abfeaecb381b597f5273c0fc
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88033060"
---
# <a name="reset-a-vpn-gateway"></a>Restablecimiento de una instancia de VPN Gateway

Restablecer una puerta de enlace de VPN de Azure es útil si se pierde la conectividad VPN entre locales en uno o varios túneles VPN de sitio a sitio. En esta situación, todos tus dispositivos VPN locales funcionan correctamente, pero no pueden establecer túneles IPsec con las Puertas de enlace de VPN de Azure. Este artículo lo ayuda a restablecer la puerta de enlace de VPN.

### <a name="what-happens-during-a-reset"></a>¿Qué ocurre durante el restablecimiento?

Una puerta de enlace de VPN se compone de dos instancias de VM que se ejecutan en una configuración de modo de espera activo. Al restablecer la puerta de enlace, reinicia la puerta de enlace y después vuelve a aplicar las configuraciones entre locales. La puerta de enlace conserva la dirección IP pública que ya tiene. Esto significa que no tendrás que actualizar la configuración del enrutador VPN con una nueva dirección IP pública de Puerta de enlace de VPN de Azure.

Una vez que se emite el comando para restablecer la puerta de enlace, se reiniciará inmediatamente la instancia activa actual de la puerta de enlace de VPN de Azure. Habrá un breve intervalo durante la conmutación por error de la instancia activa (que se está reiniciando) a la instancia en modo de espera. El intervalo debe ser inferior a un minuto.

Si la conexión no se restaura después del primer reinicio, vuelve a ejecutar el mismo comando para reiniciar la segunda instancia de VM (la nueva puerta de enlace activa). Si se solicitan los dos reinicios consecutivamente, habrá un período un poco más largo durante el que se estén reiniciando ambas instancias de máquina virtual (activa y en espera). Esto causará una interrupción mayor en la conectividad de VPN, de 30 a 45 minutos, para que las máquinas virtuales completen los reinicios.

Después de dos reinicios, si sigue teniendo problemas de conectividad entre locales, abra una incidencia de soporte técnico en Azure Portal.

## <a name="before-you-begin"></a><a name="before"></a>Antes de empezar

Antes de restablecer la puerta de enlace, compruebe los elementos clave que se enumeran a continuación para todos los túneles VPN de sitio a sitio (S2S) de IPsec. Cualquier incoherencia que haya en los elementos provocará la desconexión de los túneles VPN de S2S. Comprobar y corregir la configuración de la puerta de enlace local y de Azure VPN Gateway le evitará reinicios e interrupciones innecesarios para las demás conexiones en funcionamiento de las puertas de enlace.

Compruebe los elementos siguientes antes de restablecer la puerta de enlace:

* Las direcciones IP de Internet (VIP) tanto de la puerta de enlace VPN de Azure como de la puerta de enlace VPN local están configuradas correctamente en Azure así como en las directivas VPN locales.
* La clave previamente compartida tiene que ser la misma en las Puertas de enlace VPN de Azure y en las locales.
* Si aplica la configuración específica de IPsec/IKE, como el cifrado, los algoritmos hash y PFS (confidencialidad directa total), asegúrese de que tanto las puertas de enlace de VPN de Azure como las locales tengan la misma configuración.

## <a name="azure-portal"></a><a name="portal"></a>Azure Portal

Puede restablecer una puerta de enlace de VPN de Resource Manager mediante Azure Portal. Si desea restablecer una puerta de enlace clásica, consulte los pasos de [PowerShell](#resetclassic).

### <a name="resource-manager-deployment-model"></a>Modelo de implementación del Administrador de recursos

1. Abra [Azure Portal](https://portal.azure.com) y vaya a la puerta de enlace de red virtual de Resource Manager que desea restablecer.
2. En la hoja de la puerta de enlace de red virtual, haga clic en "Restablecer".

   ![Hoja de restablecimiento de VPN Gateway](./media/vpn-gateway-howto-reset-gateway/reset-vpn-gateway-portal.png)
3. En la hoja de restablecimiento, haga clic en el botón **Restablecer**.

## <a name="powershell"></a><a name="ps"></a>PowerShell

### <a name="resource-manager-deployment-model"></a>Modelo de implementación del Administrador de recursos

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

El cmdlet para restablecer una puerta de enlace es **Reset-AzVirtualNetworkGateway**. Antes de realizar un restablecimiento, asegúrese de disponer de la versión más reciente de los [cmdlets Az de PowerShell](https://docs.microsoft.com/powershell/module/az.network). En el ejemplo siguiente, se restablece una puerta de enlace de red virtual denominada VNet1GW en el grupo de recursos TestRG1:

```powershell
$gw = Get-AzVirtualNetworkGateway -Name VNet1GW -ResourceGroupName TestRG1
Reset-AzVirtualNetworkGateway -VirtualNetworkGateway $gw
```

Resultado:

Cuando reciba un resultado devuelto, se puede suponer que la puerta de enlace se restableció correctamente. Sin embargo, no hay nada en el resultado devuelto que indique explícitamente que el restablecimiento se realizó correctamente. Si desea examinar detenidamente el historial para ver exactamente cuándo se produjo el restablecimiento de la puerta de enlace, puede ver esa información en [Azure Portal](https://portal.azure.com). En el portal, vaya a **"NombreDePuertaDeEnlace" -> Resource Health**.

### <a name="classic-deployment-model"></a><a name="resetclassic"></a>Modelo de implementación clásica

El cmdlet para restablecer una puerta de enlace es **Reset-AzureVNetGateway**. Los cmdlets de Azure PowerShell para la administración de servicios deben instalarse localmente en el escritorio. No se puede usar Azure Cloud Shell. Antes de realizar el restablecimiento, asegúrese de disponer de la versión más reciente de los [cmdlets de PowerShell de Service Management (SM)](https://docs.microsoft.com/powershell/azure/servicemanagement/install-azure-ps?view=azuresmps-4.0.0#azure-service-management-cmdlets). Cuando use este comando, asegúrese de que está usando el nombre completo de la red virtual. Las redes virtuales clásicas que se crearon con el portal tienen un nombre largo que es necesario para PowerShell. Puede ver el nombre largo mediante "Get-AzureVNetConfig -ExportToFile C:\Myfoldername\NetworkConfig.xml".

En el ejemplo siguiente se restablece la puerta de enlace de una red virtual denominada "Group TestRG1 TestVNet1" (que se muestra simplemente como "TestVNet1" en el portal):

```powershell
Reset-AzureVNetGateway –VnetName 'Group TestRG1 TestVNet1'
```

Resultado:

```powershell
Error          :
HttpStatusCode : OK
Id             : f1600632-c819-4b2f-ac0e-f4126bec1ff8
Status         : Successful
RequestId      : 9ca273de2c4d01e986480ce1ffa4d6d9
StatusCode     : OK
```

## <a name="azure-cli"></a><a name="cli"></a>Azure CLI

Para restablecer la puerta de enlace, use el comando [az network vnet-gateway reset](https://docs.microsoft.com/cli/azure/network/vnet-gateway). En el ejemplo siguiente, se restablece una puerta de enlace de red virtual denominada VNet5GW en el grupo de recursos TestRG5:

```azurecli
az network vnet-gateway reset -n VNet5GW -g TestRG5
```

Resultado:

Cuando reciba un resultado devuelto, se puede suponer que la puerta de enlace se restableció correctamente. Sin embargo, no hay nada en el resultado devuelto que indique explícitamente que el restablecimiento se realizó correctamente. Si desea examinar detenidamente el historial para ver exactamente cuándo se produjo el restablecimiento de la puerta de enlace, puede ver esa información en [Azure Portal](https://portal.azure.com). En el portal, vaya a **"NombreDePuertaDeEnlace" -> Resource Health**.
