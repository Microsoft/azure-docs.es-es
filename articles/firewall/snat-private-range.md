---
title: Intervalos de direcciones IP privadas de SNAT de Azure Firewall
description: Se pueden configurar los intervalos de direcciones IP para SNAT.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: how-to
ms.date: 08/31/2020
ms.author: victorh
ms.openlocfilehash: 272f5b747efbc3776b1b2ba7c3546ade717c2452
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89231374"
---
# <a name="azure-firewall-snat-private-ip-address-ranges"></a>Intervalos de direcciones IP privadas de SNAT de Azure Firewall

Azure Firewall proporciona SNAT automática para todo el tráfico saliente a las IP públicas. De forma predeterminada, Azure Firewall no aplica SNAT con reglas de red cuando la dirección IP de destino es un intervalo de direcciones IP privadas, de acuerdo con la normativa [RFC 1918 de IANA](https://tools.ietf.org/html/rfc1918). Las reglas de aplicación se aplican siempre mediante un [proxy transparente](https://wikipedia.org/wiki/Proxy_server#Transparent_proxy), independientemente de la dirección IP de destino.

Esta lógica funciona bien cuando se enruta el tráfico directamente a Internet. Sin embargo, si ha se habilitado [tunelización forzada](forced-tunneling.md), al tráfico enlazado a Internet se le aplica SNAT a una de las direcciones IP privadas del firewall en AzureFirewallSubnet, ocultando el origen del firewall local.

Si su organización usa un intervalo de direcciones IP públicas para las redes privadas, Azure Firewall aplicará SNAT al tráfico para una de las direcciones IP privadas de firewall en AzureFirewallSubnet. Sin embargo, puede configurar Azure Firewall de modo que **no** aplique SNAT al intervalo de direcciones IP públicas. Por ejemplo, para especificar una dirección IP individual, puede especificarla de la siguiente manera: `192.168.1.10`. Para especificar un intervalo de direcciones IP, puede especificarlo de la siguiente manera: `192.168.1.0/24`.

Para configurar Azure Firewall en "nunca aplicar SNAT" independientemente de la dirección IP de destino, use **0.0.0.0/0** como intervalo de direcciones IP privadas. Con esta configuración, Azure Firewall nunca puede enrutar el tráfico directamente a Internet. Para configurar el firewall en "siempre aplicar SNAT" independientemente de la dirección IP de destino, use **255.255.255.255/32** como intervalo de direcciones IP privadas.

> [!IMPORTANT]
> Si desea especificar sus propios intervalos de direcciones IP privadas y mantener los intervalos de direcciones RFC 1918 de IANA predeterminados, asegúrese de que la lista personalizada todavía incluye el intervalo de RFC 1918 de IANA. 

## <a name="configure-snat-private-ip-address-ranges---azure-powershell"></a>Configuración de intervalos de direcciones IP privadas de SNAT: Azure PowerShell

Puede usar Azure PowerShell a fin de especificar intervalos de direcciones IP privadas para el firewall.

### <a name="new-firewall"></a>Nuevo firewall

En el caso de un nuevo firewall, el comando de Azure PowerShell es:

`New-AzFirewall -Name $GatewayName -ResourceGroupName $RG -Location $Location -VirtualNetworkName $vnet.Name -PublicIpName $LBPip.Name -PrivateRange @("IANAPrivateRanges","IPRange1", "IPRange2")`

> [!NOTE]
> IANAPrivateRanges se expande a los valores predeterminados actuales en Azure Firewall mientras que los demás intervalos se agregan a él. Para mantener el valor predeterminado de IANAPrivateRanges en su especificación de intervalo privado, debe permanecer en la especificación de `PrivateRange`, tal como se muestra en los ejemplos siguientes.

Para más información, consulte [New-AzFirewall](https://docs.microsoft.com/powershell/module/az.network/new-azfirewall?view=azps-3.3.0).

### <a name="existing-firewall"></a>Firewall existente

Para configurar un firewall existente, use los siguientes comandos de Azure PowerShell:

```azurepowershell
$azfw = Get-AzFirewall -ResourceGroupName "Firewall Resource Group name"
$azfw.PrivateRange = @("IANAPrivateRanges","IPRange1", "IPRange2")
Set-AzFirewall -AzureFirewall $azfw
```

### <a name="templates"></a>Plantillas

Puede agregar lo siguiente a la sección `additionalProperties`:

```
"additionalProperties": {
                    "Network.SNAT.PrivateRanges": "IANAPrivateRanges , IPRange1, IPRange2"
                },
```

## <a name="configure-snat-private-ip-address-ranges---azure-portal"></a>Configuración de intervalos de direcciones IP privadas de SNAT: Azure Portal

Puede usar Azure Portal a fin de especificar intervalos de direcciones IP privadas para el firewall.

1. Seleccione el grupo de recursos y, después, el firewall.
2. En la página **Información general**, **Intervalos IP privados**, seleccione el valor predeterminado **IANA RFC 1918**.

   Se abre la página **Editar prefijos IP privados**:

   :::image type="content" source="media/snat-private-range/private-ip.png" alt-text="Editar prefijos IP privados":::

1. De forma predeterminada, **IANAPrivateRanges** está configurado.
2. Edite los intervalos de direcciones IP privadas para su entorno y, después, seleccione **Guardar**.

## <a name="next-steps"></a>Pasos siguientes

- Obtener más información sobre [Tunelización forzada de Azure Firewall](forced-tunneling.md).