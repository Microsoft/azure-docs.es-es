---
title: Conexión al back-end v1
description: Obtenga información acerca de cómo conectarse de forma segura a los recursos de back-end desde un entorno de App Service. Este documento solo se proporciona para los clientes que usan App Service Environment v1 heredado.
author: stefsch
ms.assetid: f82eb283-a6e7-4923-a00b-4b4ccf7c4b5b
ms.topic: article
ms.date: 10/04/2016
ms.author: stefsch
ms.custom: seodec18
ms.openlocfilehash: 9f8e288f771b9d584a0fd3430115f5fa60f68e47
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88961812"
---
# <a name="connect-securely-to-back-end-resources-from-an-app-service-environment"></a>Conexión segura a los recursos de back-end desde App Service Environment
Dado que una instancia de App Service Environment siempre se crea **o bien** en una red virtual de Azure Resource Manager **o** en una [red virtual][virtualnetwork] del modelo de implementación clásica, las conexiones salientes de dicho entorno a otros recursos de back-end solo pueden fluir a través de la red virtual. A partir de junio de 2016, las instancias de ASE también se pueden implementar en redes virtuales que usen intervalos de direcciones públicas o espacios de direcciones RFC1918 (direcciones privadas).  

Por ejemplo, puede haber un servidor SQL Server que se ejecute en un clúster de máquinas virtuales con el puerto 1433 bloqueado.  En el extremo puede incluirse una lista de control de acceso para permitir únicamente el acceso de otros recursos de la misma red virtual.  

Otro ejemplo: los puntos de conexión con información confidencial pueden ejecutarse localmente y conectarse a Azure mediante conexiones [Sitio a sitio][SiteToSite] o [Azure ExpressRoute][ExpressRoute].  Como resultado, solo los recursos de las redes virtuales conectadas a túneles ExpressRoute o De sitio a sitio podrían tener acceso a los extremos locales.

En todos estos escenarios, las aplicaciones que se ejecutan en App Service Environment pueden conectarse de forma segura a los distintos servidores y recursos. Si el tráfico saliente de las aplicaciones se ejecuta en App Service Environment hacia puntos de conexión privados de la misma red virtual (o conectados a la misma red virtual) solamente fluirá por la red virtual.  El tráfico saliente hacia puntos de conexión privados no fluirá por la red pública de Internet.

Un problema se aplica al tráfico saliente desde una instancia de App Service Environment hacia los puntos de conexión de una red virtual. Las instancias de App Service Environment no pueden acceder a los puntos de conexión de máquinas virtuales ubicadas en la **misma** subred que el App Service Environment. Normalmente, esta limitación no debería suponer ningún problema, siempre y cuando App Service Environment se implemente en una subred reservada para su uso exclusivo.

[!INCLUDE [app-service-web-to-api-and-mobile](../../../includes/app-service-web-to-api-and-mobile.md)]

## <a name="outbound-connectivity-and-dns-requirements"></a>Requisitos de DNS y conectividad saliente
Para que un Entorno de App Service funcione correctamente, necesita acceso de salida a varios puntos de conexión. Una lista completa de los puntos de conexión externos utilizados por un ASE en la sección "Conectividad de red necesaria" del artículo [Detalles de configuración de red para entornos del Servicio de aplicaciones con ExpressRoute](app-service-app-service-environment-network-configuration-expressroute.md#required-network-connectivity) .

Los Entornos de App Service requieren una infraestructura DNS válida configurada para la red virtual.  Si se cambia la configuración de DNS después de crear una instancia de App Service Environment, los desarrolladores pueden forzar que una instancia aplique la nueva configuración de DNS. En la parte superior de la hoja de administración de App Service Environment del portal, seleccione el icono **Reiniciar** para desencadenar el reinicio gradual del entorno, lo que provoca que el entorno aplique la nueva configuración de DNS.

También se recomienda configurar de antemano los servidores DNS personalizados de la red virtual antes de crear una instancia de App Service Environment.  Si se cambia la configuración de DNS de una red virtual al crear una instancia de App Service Environment, se generará un error en el proceso de creación. En el otro extremo de una puerta de enlace de VPN, si hay un servidor DNS personalizado que no está accesible o disponible, también se generará un error al crear la instancia de App Service Environment.

## <a name="connecting-to-a-sql-server"></a>Conexión a un servidor SQL Server
Una configuración común de SQL Server tiene un extremo que escucha en el puerto 1433:

![Extremo de SQL Server][SqlServerEndpoint]

Existen dos formas de restringir el tráfico a este extremo:

* [Listas de control de acceso de red][NetworkAccessControlLists] (ACL de red)
* [Grupos de seguridad de red][NetworkSecurityGroups]

## <a name="restricting-access-with-a-network-acl"></a>Restricción del acceso con una lista de control de acceso de red
El puerto 1433 se puede proteger mediante una lista de control de acceso de red.  En el ejemplo siguiente se incluyen en los permisos de la asignación las direcciones de cliente que proceden de una red virtual y se bloquea el acceso al resto de clientes.

![Ejemplo de lista de control de acceso de red][NetworkAccessControlListExample]

Cualquier aplicación que se ejecute en App Service Environment, en la misma red virtual que SQL Server, puede conectarse a la instancia de SQL Server. Use la dirección IP **interna de la red virtual** para la máquina virtual de SQL Server.  

La cadena de conexión de ejemplo siguiente hace referencia a SQL Server mediante su dirección IP privada.

`Server=tcp:10.0.1.6;Database=MyDatabase;User ID=MyUser;Password=PasswordHere;provider=System.Data.SqlClient`

Aunque la máquina virtual también tiene un punto de conexión público, los intentos de conexión para usar la dirección IP pública se rechazarán debido a la ACL de red. 

## <a name="restricting-access-with-a-network-security-group"></a>Restricción del acceso con un grupo de seguridad de red
Un enfoque alternativo para proteger el acceso consiste en usar un grupo de seguridad de red.  Los grupos de seguridad de red pueden aplicarse a máquinas virtuales individuales o a una subred que contenga máquinas virtuales.

En primer lugar, deberá crear un grupo de seguridad de red:

```azurepowershell-interactive
New-AzureNetworkSecurityGroup -Name "testNSGexample" -Location "South Central US" -Label "Example network security group for an app service environment"
```

Restringir el acceso exclusivamente al tráfico interno de la red virtual es sencillo con un grupo de seguridad de red.  Las reglas predeterminadas de un grupo de este tipo solo permiten el acceso de otros clientes de la misma red virtual.

Como resultado, el bloqueo del acceso a SQL Server es sencillo. Solo tiene que aplicar un grupo de seguridad de red con sus reglas predeterminadas a las máquinas virtuales que ejecutan SQL Server o a la subred que contiene las máquinas virtuales.

En el ejemplo siguiente se aplica un grupo de seguridad de red a la subred contenedora:

```azurepowershell-interactive
Get-AzureNetworkSecurityGroup -Name "testNSGExample" | Set-AzureNetworkSecurityGroupToSubnet -VirtualNetworkName 'testVNet' -SubnetName 'Subnet-1'
```

El resultado final es un conjunto de reglas de seguridad que bloquean el acceso externo y permiten el acceso interno a la red virtual:

![Reglas de seguridad de red predeterminadas][DefaultNetworkSecurityRules]

## <a name="getting-started"></a>Introducción
Para empezar a trabajar con los entornos de App Service, consulte [Introducción al entorno de App Service][IntroToAppServiceEnvironment]

Para más información sobre cómo controlar el tráfico entrante en App Service Environment, consulte el artículo sobre el [control del tráfico entrante en una instancia de App Service Environment][ControlInboundASE].

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!-- LINKS -->
[virtualnetwork]: ../../virtual-network/virtual-networks-faq.md
[ControlInboundTraffic]:  app-service-app-service-environment-control-inbound-traffic.md
[SiteToSite]: ../../vpn-gateway/vpn-gateway-multi-site.md
[ExpressRoute]: https://azure.microsoft.com/services/expressroute/
[NetworkAccessControlLists]: /previous-versions/azure/virtual-network/virtual-networks-acl
[NetworkSecurityGroups]: ../../virtual-network/virtual-network-vnet-plan-design-arm.md
[IntroToAppServiceEnvironment]:  app-service-app-service-environment-intro.md
[ControlInboundASE]:  app-service-app-service-environment-control-inbound-traffic.md

<!-- IMAGES -->
[SqlServerEndpoint]: ./media/app-service-app-service-environment-securely-connecting-to-backend-resources/SqlServerEndpoint01.png
[NetworkAccessControlListExample]: ./media/app-service-app-service-environment-securely-connecting-to-backend-resources/NetworkAcl01.png
[DefaultNetworkSecurityRules]: ./media/app-service-app-service-environment-securely-connecting-to-backend-resources/DefaultNetworkSecurityRules01.png