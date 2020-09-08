---
title: Controlar el tráfico de entrada en v1
description: Obtenga información sobre cómo controlar el tráfico de entrada a una instancia de App Service Environment. Este documento solo se proporciona para los clientes que usan App Service Environment v1 heredado.
author: ccompy
ms.assetid: 4cc82439-8791-48a4-9485-de6d8e1d1a08
ms.topic: article
ms.date: 01/11/2017
ms.author: stefsch
ms.custom: seodec18
ms.openlocfilehash: fe9326ea9ebd5afe981b7ba6c34b1a5d51e084b0
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962067"
---
# <a name="how-to-control-inbound-traffic-to-an-app-service-environment"></a>Cómo controlar el tráfico de entrada a un entorno de App Service
## <a name="overview"></a>Información general
Se puede crear una instancia de App Service Environment **o bien** en una red virtual de Azure Resource Manager **o** en una [red virtual][virtualnetwork] del modelo de implementación clásica.  Una nueva red virtual y una nueva subred pueden definirse en el momento en que se crea un entorno de App Service. En su lugar, se puede crear una instancia de App Service Environment en una red virtual y subred preexistentes.  A partir de junio de 2016, las instancias de ASE también se pueden implementar en redes virtuales que usen intervalos de direcciones públicas o espacios de direcciones RFC1918 (direcciones privadas).  Para más información, vea [Creación de App Service Environment][HowToCreateAnAppServiceEnvironment].

Cree siempre una instancia de App Service Environment en una subred. Una subred proporciona un límite de red que se puede usar para bloquear el tráfico entrante detrás de los dispositivos y servicios ascendentes. Esta configuración solo permite que las direcciones IP ascendentes específicas acepten tráfico HTTP y HTTPS.

El tráfico de red entrante y saliente de una subred se controla mediante un [grupo de seguridad de red][NetworkSecurityGroups]. Para controlar el tráfico entrante, cree reglas de seguridad en un grupo de seguridad de red. Después, asigne al grupo de seguridad de red la subred que contiene la instancia de App Service Environment.

Una asignado un grupo de seguridad de red a una subred, el tráfico entrante a aplicaciones en App Service Environment se permite o bloquea según las reglas de permiso y denegación definidas en el grupo de seguridad de red.

[!INCLUDE [app-service-web-to-api-and-mobile](../../../includes/app-service-web-to-api-and-mobile.md)]

## <a name="inbound-network-ports-used-in-an-app-service-environment"></a>Puertos de red usados en un entorno de App Service
Antes de bloquear el tráfico de red entrante con un grupo de seguridad de red, debe conocer el conjunto de puertos de red necesarios y opcionales utilizados por una instancia de App Service Environment.  El bloqueo accidental del tráfico a algunos puertos puede producir la pérdida de funcionalidad en un entorno de App Service.

A continuación, se muestra una lista de puertos utilizados por una instancia de App Service Environment. Todos los puertos son **TCP**, a menos que indique claramente:

* 454:  **puerto obligatorio** utilizado por la infraestructura de Azure para administrar y mantener instancias de App Service Environment a través de TLS.  No bloquee el tráfico a este puerto.  Este puerto siempre está enlazado a la dirección VIP pública de un ASE.
* 455:  **puerto obligatorio** utilizado por la infraestructura de Azure para administrar y mantener instancias de App Service Environment a través de TLS.  No bloquee el tráfico a este puerto.  Este puerto siempre está enlazado a la dirección VIP pública de un ASE.
* 80:  el puerto predeterminado para el tráfico HTTP entrante a aplicaciones que se ejecutan en planes de App Service en una instancia de App Service Environment.  En un ASE con un ILB, este puerto está enlazado a la dirección del ILB del ASE.
* 443: el puerto predeterminado para el tráfico TLS entrante a aplicaciones que se ejecutan en planes de App Service en una instancia de App Service Environment.  En un ASE con un ILB, este puerto está enlazado a la dirección del ILB del ASE.
* 21:  canal de control para FTP.  Este puerto se puede bloquear de forma segura si no se utiliza FTP.  En un ASE con un ILB, este puerto puede enlazarse a la dirección del ILB de un ASE.
* 990:  canal de control para FTPS.  Este puerto se puede bloquear de forma segura si no se utiliza FTPS.  En un ASE con un ILB, este puerto puede enlazarse a la dirección del ILB de un ASE.
* 10001-10020: canales de datos para FTP.  Al igual que con el canal de control, estos puertos se pueden bloquear de forma segura si no se utiliza FTP.  En un ASE con un ILB, este puerto puede enlazarse a la dirección del ILB de un ASE.
* 4016: usado para la depuración remota con Visual Studio 2012.  Este puerto se puede bloquear de forma segura si no se utiliza la característica.  En un ASE con un ILB, este puerto está enlazado a la dirección del ILB del ASE.
* 4018: usado para la depuración remota con Visual Studio 2013.  Este puerto se puede bloquear de forma segura si no se utiliza la característica.  En un ASE con un ILB, este puerto está enlazado a la dirección del ILB del ASE.
* 4020: usado para la depuración remota con Visual Studio 2015.  Este puerto se puede bloquear de forma segura si no se utiliza la característica.  En un ASE con un ILB, este puerto está enlazado a la dirección del ILB del ASE.

## <a name="outbound-connectivity-and-dns-requirements"></a>Requisitos de DNS y conectividad saliente
Para que un entorno de App Service funcione correctamente, necesita acceso de salida a varios puntos de conexión. Una lista completa de los puntos de conexión externos utilizados por un ASE en la sección "Conectividad de red necesaria" del artículo [Detalles de configuración de red para entornos del Servicio de aplicaciones con ExpressRoute](app-service-app-service-environment-network-configuration-expressroute.md#required-network-connectivity) .

Los Entornos de App Service requieren una infraestructura DNS válida configurada para la red virtual.  Si se cambia la configuración de DNS después de crear una instancia de App Service Environment, los desarrolladores pueden forzar que una instancia aplique la nueva configuración de DNS.  Si desencadena un reinicio gradual del entorno mediante el icono **Reiniciar**, el entorno aplica la nueva configuración de DNS. (El icono **Reiniciar** se encuentra en la parte superior de la hoja de administración de App Service Environment, en [Azure Portal][NewPortal]).

También se recomienda configurar de antemano los servidores DNS personalizados de la red virtual antes de crear una instancia de App Service Environment.  Si se cambia la configuración de DNS de una red virtual al crear una instancia de App Service Environment, se producirá un error en el proceso de creación de dicha instancia.  De forma similar, si hay un servidor DNS personalizado que no está accesible o disponible en el otro extremo de la puerta de enlace de VPN, también se generará un error al crear la instancia de App Service Environment.

## <a name="creating-a-network-security-group"></a>Creación de un grupo de seguridad de red
Para más información sobre cómo funcionan los grupos de seguridad de red, consulte la siguiente [información][NetworkSecurityGroups].  En el ejemplo de administración de servicios de Azure siguiente se tratan los aspectos destacados de los grupos de seguridad de red. En el ejemplo, se configura y aplica un grupo de seguridad de red a una subred que contiene una instancia de App Service Environment.

**Nota:** Los grupos de seguridad de red se pueden configurar gráficamente con [Azure Portal](https://portal.azure.com) o mediante Azure PowerShell.

Los grupos de seguridad de red se crean por primera vez como una entidad independiente asociada a una suscripción. Puesto que los grupos de seguridad de red se crean en una región de Azure, cree el grupo de seguridad de red en la misma región que la instancia de App Service Environment.

El siguiente comando muestra cómo se crea un grupo de seguridad de red:

```azurepowershell-interactive
New-AzureNetworkSecurityGroup -Name "testNSGexample" -Location "South Central US" -Label "Example network security group for an app service environment"
```

Una vez creado un grupo de seguridad de red, una o varias reglas de seguridad de red se agregan a él.  Dado que el conjunto de reglas puede cambiar con el tiempo, debe espaciar el esquema de numeración usado para las prioridades de las reglas. Esta práctica facilita la inserción de reglas adicionales a lo largo del tiempo.

En el ejemplo siguiente, una regla concede acceso de forma explícita a los puertos de administración necesarios para que la infraestructura de Azure administre y mantenga una instancia de App Service Environment.  Todo el tráfico de administración fluye a través de TLS y está protegido por certificados de cliente. Aunque los puertos están abiertos, son inaccesibles para cualquier entidad que no sea la infraestructura de administración de Azure.

```azurepowershell-interactive
Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "ALLOW AzureMngmt" -Type Inbound -Priority 100 -Action Allow -SourceAddressPrefix 'INTERNET'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '454-455' -Protocol TCP
```

Al bloquear el acceso a los puertos 80 y 443 para "ocultar" una instancia de App Service Environment tras dispositivos o servicios ascendentes, recuerde la dirección IP ascendente.  Por ejemplo, si usa un firewall de aplicaciones web (WAF), el WAF tendrá su propia dirección o direcciones IP. El WAF las usa al redirigir mediante proxy el tráfico a una instancia descendente de App Service Environment.  Deberá usar esta dirección IP en el parámetro *SourceAddressPrefix* de una regla de seguridad de red.

En el ejemplo siguiente, se permite explícitamente el tráfico entrante de una dirección IP ascendente específica.  La dirección *1.2.3.4* se usa como marcador de posición para la dirección IP de un WAF ascendente.  Cambie el valor para que coincida con la dirección usada por su dispositivo o servicio ascendente.

```azurepowershell-interactive
Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT HTTP" -Type Inbound -Priority 200 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '80' -Protocol TCP
Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT HTTPS" -Type Inbound -Priority 300 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '443' -Protocol TCP
```

Si se desea compatibilidad con FTP, use las reglas siguientes como plantilla para conceder acceso al puerto de control FTP y a los puertos del canal de datos.  Puesto que FTP es un protocolo con estado, es posible que no pueda enrutar el tráfico FTP a través de un dispositivo de firewall o proxy HTTP/HTTPS.  En este caso, deberá establecer *SourceAddressPrefix* en un valor diferente, como el intervalo de direcciones IP del desarrollador o los equipos de implementación en los que se ejecutan los clientes FTP. 

```azurepowershell-interactive
Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT FTPCtrl" -Type Inbound -Priority 400 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '21' -Protocol TCP
Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT FTPDataRange" -Type Inbound -Priority 500 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '10001-10020' -Protocol TCP
```

(**Nota:** puede cambiar el intervalo de puertos del canal de datos durante el período de vista previa).

Si se usa la depuración remota con Visual Studio, las reglas siguientes muestran cómo conceder acceso.  Puesto que cada versión usa un puerto diferente para la depuración remota, hay una regla distinta para cada versión compatible de Visual Studio.  Al igual que ocurre con el acceso FTP, es posible que el tráfico de depuración remoto no fluya correctamente a través de un dispositivo de proxy o WAF tradicional.  En su lugar, *SourceAddressPrefix* se puede establecer en el intervalo de direcciones IP de los equipos del desarrollador que ejecutan Visual Studio.

```azurepowershell-interactive
Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT RemoteDebuggingVS2012" -Type Inbound -Priority 600 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '4016' -Protocol TCP
Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT RemoteDebuggingVS2013" -Type Inbound -Priority 700 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '4018' -Protocol TCP
Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT RemoteDebuggingVS2015" -Type Inbound -Priority 800 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '4020' -Protocol TCP
```

## <a name="assigning-a-network-security-group-to-a-subnet"></a>Asignación de un grupo de seguridad de red a una subred
Un grupo de seguridad de red tiene una regla de seguridad predeterminada que deniega el acceso a todo el tráfico externo. Al combinar esta regla con las reglas de seguridad de red anteriores, solo el tráfico de los intervalos de direcciones de origen asociados a una acción *Permitir* podrá enviar tráfico a las aplicaciones que se ejecutan en una instancia de App Service Environment.

Una vez que se rellena un grupo de seguridad de red con las reglas de seguridad, asígnelo a la subred que contiene la instancia de App Service Environment.  El comando de asignación hace referencia a dos nombres: el nombre de la red virtual donde reside la instancia de App Service Environment y el nombre de la subred donde se creó dicha instancia.  

En el ejemplo siguiente se muestra la asignación de un grupo de seguridad de red a una subred y una red virtual:

```azurepowershell-interactive
Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityGroupToSubnet -VirtualNetworkName 'testVNet' -SubnetName 'Subnet-test'
```

La asignación es una operación de ejecución prolongada y puede tardar unos minutos en completarse. Una vez que la asignación del grupo de seguridad de red se realiza correctamente, solo el tráfico entrante que coincida con reglas *Permitir* llegará correctamente a las aplicaciones de la instancia de App Service Environment.

Por razones de integridad, en el ejemplo siguiente se muestra cómo quitar y desasociar el grupo de seguridad de red de la subred:

```azurepowershell-interactive
Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Remove-AzureNetworkSecurityGroupFromSubnet -VirtualNetworkName 'testVNet' -SubnetName 'Subnet-test'
```

## <a name="special-considerations-for-explicit-ip-ssl"></a>Consideraciones especiales para IP-SSL explícito
Si una aplicación está configurada con una dirección IP-SSL explícita (aplicable *solo* a los ASE que tienen una dirección VIP pública) en lugar de usar la dirección IP predeterminada de la instancia de App Service Environment, tanto el tráfico HTTP como el tráfico HTTPS fluyen a la subred a través de puertos distintos de los puertos 80 y 443.

Para encontrar el par individual de puertos que utiliza cada dirección IP-SSL, vaya al portal y consulte la hoja de experiencia de usuario de los detalles de la instancia de App Service Environment.  Seleccione **Toda la configuración** > **Direcciones IP**.  En la hoja **Direcciones IP** se muestra una tabla de todas las direcciones IP-SSL configuradas explícitamente para la instancia de App Service Environment. La hoja también muestra el par de puertos especial que se usa para enrutar el tráfico HTTP y HTTPS asociado a cada dirección IP-SSL.  Use este par de puertos para los parámetros de DestinationPortRange al configurar las reglas de un grupo de seguridad de red.

Cuando una aplicación de un ASE está configurada para usar IP-SSL, los clientes externos no verán la asignación del par de puertos especial ni tendrán que preocuparse por ello.  El tráfico a las aplicaciones fluirá con normalidad a la dirección IP-SSL configurada.  La traslación al par de puertos especial se realiza de manera automática e interna durante el tramo final del enrutamiento del tráfico en la subred que contiene el ASE. 

## <a name="getting-started"></a>Introducción
Para empezar a trabajar con las instancias de App Service Environment, vea [Introducción a App Service Environment][IntroToAppServiceEnvironment].

Para más información, vea [Conexión segura a los recursos de back-end desde App Service Environment][SecurelyConnecttoBackend].

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!-- LINKS -->
[virtualnetwork]: ../../virtual-network/virtual-networks-faq.md
[HowToCreateAnAppServiceEnvironment]: app-service-web-how-to-create-an-app-service-environment.md
[NetworkSecurityGroups]: ../../virtual-network/virtual-network-vnet-plan-design-arm.md
[IntroToAppServiceEnvironment]:  app-service-app-service-environment-intro.md
[SecurelyConnecttoBackend]:  app-service-app-service-environment-securely-connecting-to-backend-resources.md
[NewPortal]:  https://portal.azure.com  

<!-- IMAGES -->