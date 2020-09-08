---
title: Configuración del tiempo de espera de inactividad TCP del equilibrador de carga en Azure
titleSuffix: Azure Load Balancer
description: En este artículo, aprenderá a configurar Azure Load Balancer tiempo de espera de inactividad de TCP.
services: load-balancer
documentationcenter: na
author: asudbring
ms.custom: seodec18
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/09/2020
ms.author: allensu
ms.openlocfilehash: 317f6a73812b0e4284564ca9b5593e09e22edf12
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89048744"
---
# <a name="configure-tcp-idle-timeout-settings-for-azure-load-balancer"></a>Modificación de la configuración de tiempo de espera de inactividad de TCP para Azure Load Balancer

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de forma local, para realizar los pasos de este artículo necesita la versión 5.4.1 del módulo de Azure PowerShell o cualquier versión posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="tcp-idle-timeout"></a>Tiempo de espera de inactividad de TCP
Azure Load Balancer tiene una configuración de tiempo de espera de inactividad de 4 a 30 minutos. De forma predeterminada, se establece en 4 minutos. Si un período de inactividad es mayor que el valor de tiempo de espera, no hay ninguna garantía de que todavía exista la sesión TCP o HTTP entre el cliente y el servicio en la nube.

Cuando se cierra la conexión, la aplicación cliente puede recibir el mensaje de error siguiente: "The underlying connection was closed: A connection that was expected to be kept alive was closed by the server" (Se ha terminado la conexión: El servidor cerró una conexión que se esperaba estuviera activa).

Una práctica común es usar TCP Keep-alive. Esta práctica mantiene la conexión activa durante un periodo más largo. Para obtener más información, consulte estos [ejemplos de .NET](https://msdn.microsoft.com/library/system.net.servicepoint.settcpkeepalive.aspx). Con Keep-alive habilitado, los paquetes se envían durante los periodos de inactividad en la conexión. Los paquetes de Keep-alive garantizan que no se alcance el valor de tiempo de espera de inactividad y la conexión se mantenga durante un largo período.

La configuración funciona solo para conexiones entrantes. Para evitar la pérdida de la conexión, configure el TCP keep-alive con un intervalo menor que el valor de tiempo de espera de inactividad o aumentar el valor de tiempo de espera de inactividad. Para admitir estos escenarios, se agregó compatibilidad con un tiempo de espera de inactividad configurable.

TCP keep-alive funciona en escenarios donde la batería no supone una restricción. No se recomienda para aplicaciones móviles. El uso de TCP Keep-alive desde una aplicación móvil puede agotarla batería del dispositivo más rápidamente.

![Tiempo de espera TCP](./media/load-balancer-tcp-idle-timeout/image1.png)

Las secciones siguientes describen cómo cambiar la configuración de tiempo de espera de inactividad para los recursos de IP pública y equilibrador de carga.

>[!NOTE]
> El tiempo de espera de inactividad de TCP no afecta a las reglas de equilibrio de carga en el protocolo UDP.


## <a name="configure-the-tcp-timeout-for-your-instance-level-public-ip-to-15-minutes"></a>Configuración del tiempo de espera de TCP para la IP pública a nivel de instancia en 15 minutos

```azurepowershell-interactive
$publicIP = Get-AzPublicIpAddress -Name MyPublicIP -ResourceGroupName MyResourceGroup
$publicIP.IdleTimeoutInMinutes = "15"
Set-AzPublicIpAddress -PublicIpAddress $publicIP
```

`IdleTimeoutInMinutes` es opcional. Si no se establece, el tiempo de espera predeterminado es de 4 minutos. El intervalo de tiempo de espera aceptable está entre 4 y 30 minutos.

## <a name="set-the-tcp-timeout-on-a-load-balanced-rule-to-15-minutes"></a>Establecimiento del tiempo de espera de TCP en una regla de carga equilibrada en 15 minutos

Para establecer el tiempo de espera de inactividad de un equilibrador de carga, se establece el valor de "IdleTimeoutInMinutes" en la regla de carga equilibrada. Por ejemplo:

```azurepowershell-interactive
$lb = Get-AzLoadBalancer -Name "MyLoadBalancer" -ResourceGroup "MyResourceGroup"
$lb | Set-AzLoadBalancerRuleConfig -Name myLBrule -IdleTimeoutInMinutes 15
```
## <a name="next-steps"></a>Pasos siguientes

[Información general sobre el equilibrador de carga interno](load-balancer-internal-overview.md)

[Introducción a la creación de un equilibrador de carga orientado a Internet](quickstart-load-balancer-standard-public-powershell.md)

[Configuración de un modo de distribución del equilibrador de carga](load-balancer-distribution-mode.md)
