---
title: Configuración de Azure ExpressRoute v1
description: Configuración de red para instancias de App Service Environment para PowerApps con Azure ExpressRoute. Este documento solo se proporciona para los clientes que usan App Service Environment v1 heredado.
author: stefsch
ms.assetid: 34b49178-2595-4d32-9b41-110c96dde6bf
ms.topic: article
ms.date: 10/14/2016
ms.author: stefsch
ms.custom: seodec18
ms.openlocfilehash: 3fa2677f7c71b118ec3f7da717e29d5879ee982b
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88961829"
---
# <a name="network-configuration-details-for-app-service-environment-for-powerapps-with-azure-expressroute"></a>Detalles de configuración de red para entornos de App Service para PowerApps con Azure ExpressRoute

Los clientes pueden conectar un circuito de [Azure ExpressRoute][ExpressRoute] a su infraestructura de red virtual para ampliar la red local a Azure. App Service Environment se crea en una subred de la infraestructura de [red virtual][virtualnetwork]. Las aplicaciones que se ejecutan en App Service Environment pueden establecer conexiones seguras con los recursos de back-end a los que solo se puede tener acceso través de la conexión de ExpressRoute.  

Se puede crear una instancia de App Service Environment en estos escenarios:
- Redes virtuales de Azure Resource Manager.
- Redes virtuales con el modelo de implementación clásica.
- Redes virtuales que usen intervalos de direcciones públicas o espacios de direcciones de RFC1918 (es decir, direcciones privadas). 

[!INCLUDE [app-service-web-to-api-and-mobile](../../../includes/app-service-web-to-api-and-mobile.md)]

## <a name="required-network-connectivity"></a>Conectividad de red necesaria

App Service Environment tiene unos requisitos de conectividad de red que inicialmente podrían no cumplirse en una red virtual conectada a ExpressRoute.

App Service Environment requiere los siguientes valores de conectividad de red para funcionar correctamente:

* Conectividad de red saliente a los puntos de conexión de Azure Storage distribuidos por todo el mundo en los puertos 80 y 443. Estos puntos de conexión están ubicados en la misma región que App Service Environment y también en otras regiones de Azure. Los puntos de conexión de Azure Storage se resuelven en los siguientes dominios de DNS: table.core.windows.net, blob.core.windows.net, queue.core.windows.net y file.core.windows.net.  

* Conectividad de red saliente al servicio Archivos de Azure en el puerto 445.

* Conectividad de red saliente a los puntos de conexión de Azure SQL Database ubicados en la misma región que App Service Environment. Los puntos de conexión de SQL Database se resuelven en el dominio database.windows.net, lo que requiere un acceso abierto a los puertos 1433, 11000 a 11999 y 14000 a 14999. Para más información sobre el uso de puertos de SQL Database V12, consulte [Puertos más allá de 1433 para ADO.NET 4.5](../../azure-sql/database/adonet-v12-develop-direct-route-ports.md).

* Conectividad de red saliente a los puntos de conexión del plano de administración de Azure (los del modelo de implementación clásica de Azure y los de Azure Resource Manager). La conectividad a esos puntos de conexión incluye los dominios management.core.windows.net y management.azure.com. 

* Conectividad de red saliente a los dominios ocsp.msocsp.com, mscrl.microsoft.com y crl.microsoft.com. La conectividad con estos dominios es necesaria para admitir la funcionalidad TLS.

* La configuración de DNS para la red virtual debe ser capaz de resolver todos los puntos de conexión y dominios mencionados en este artículo. Si no se pueden resolver los puntos de conexión, se producirá un error en la creación de la instancia de App Service Environment. Todas las instancias de App Service Environment se marcarán como incorrectas.

* Se requiere acceso saliente en el puerto 53 para establecer la comunicación con los servidores DNS.

* Si existe un servidor DNS personalizado en el otro punto de conexión de una puerta de enlace de VPN, el servidor DNS debe estar accesible desde la subred que contiene App Service Environment. 

* La ruta de acceso de la red saliente no puede atravesar los servidores proxy corporativos internos, ni puede forzar la tunelización a local. Si lo hiciera, se cambiaría la dirección NAT en vigor del tráfico de red saliente de App Service Environment. Los cambios en la dirección NAT del tráfico de red de salida de un entorno de App Service Environment provocarán errores de conectividad a muchos de los puntos de conexión. Se producirá un error en la creación de la instancia de App Service Environment. Todas las instancias de App Service Environment se marcarán como incorrectas.

* El acceso de red entrante a los puertos requeridos para las instancias de App Service Environment debe estar permitido. Para más información, consulte [Cómo controlar el tráfico de entrada a App Service Environment][requiredports].

Para cumplir los requisitos de DNS, asegúrese de que se configura y se mantiene una infraestructura DNS válida para la red virtual. Si se cambia la configuración de DNS después de haber creado una instancia de App Service Environment, los desarrolladores pueden forzar a esta para que recoja la nueva configuración de DNS. Puede desencadenar un reinicio gradual del entorno mediante el icono **Reiniciar** en la administración de App Service Environment en [Azure Portal][NewPortal]. El reinicio hace que el entorno recoja la nueva configuración de DNS.

Para cumplir los requisitos de acceso de la red entrante, configure un [grupo de seguridad de red (NSG)][NetworkSecurityGroups] en la subred de App Service Environment. Este grupo permite el acceso necesario [para controlar el tráfico entrante en App Service Environment][requiredports].

## <a name="outbound-network-connectivity"></a>Conectividad de red saliente

De forma predeterminada, un circuito de ExpressRoute recién creado anuncia una ruta predeterminada que permite la conectividad saliente de Internet. App Service Environment puede usar esta configuración para conectarse a otros puntos de conexión de Azure.

Una configuración habitual de cliente es definir su propia ruta predeterminada (0.0.0.0/0) que fuerza a que el tráfico saliente de Internet fluya a nivel local. El flujo de tráfico interrumpe invariablemente App Service Environment. El tráfico saliente se bloqueará de forma local o NAT tendrá un conjunto de direcciones irreconocibles que ya no funcionarán con varios puntos de conexión de Azure.

La solución consiste en definir una (o varias) rutas definidas por el usuario en la subred que contiene la instancia de App Service Environment. Una ruta definida por el usuario define las rutas de subred específica que se respetarán en lugar de la ruta predeterminada.

Si es posible, use la siguiente configuración:

* La configuración de ExpressRoute recomienda 0.0.0.0/0. De forma predeterminada, la configuración obliga a los túneles a impulsar todo el tráfico saliente de manera local.
* La ruta definida por el usuario aplicada a la subred que contiene App Service Environment define 0.0.0.0/0 con un tipo de próximo salto de Internet. Más adelante en este artículo se describe un ejemplo de esta configuración.

El efecto combinado de esta configuración es que la ruta definida por el usuario en el nivel de la subred tiene prioridad sobre la tunelización forzada de ExpressRoute. Se garantiza el acceso al tráfico de Internet saliente desde App Service Environment.

> [!IMPORTANT]
> Las rutas definidas en una UDR tienen que ser lo suficientemente específicas para que tengan prioridad sobre cualquier otra ruta anunciada por la configuración de ExpressRoute. El ejemplo descrito en la sección siguiente usa el intervalo de direcciones 0.0.0.0/0 amplio. Los anuncios de ruta con intervalos de direcciones más específicos pueden reemplazar accidentalmente a este intervalo.
> 
> Las instancias de App Service Environment no son compatibles con las configuraciones de ExpressRoute que anuncian rutas entre la ruta de acceso de emparejamiento público y la ruta de acceso de emparejamiento privado. Las configuraciones de ExpressRoute con el emparejamiento público configurado reciben anuncios de ruta de Microsoft para un amplio conjunto de intervalos de direcciones IP de Microsoft Azure. Si los intervalos de direcciones se anuncian en la ruta de acceso de emparejamiento privado, se fuerza la tunelización de todos los paquetes de red salientes de la subred de App Service Environment a la infraestructura de red local de un cliente. Actualmente, este flujo de red no es compatible con las instancias de App Service Environment. Una solución consiste en dejar de anunciar las rutas entre la ruta de acceso de emparejamiento público y la de emparejamiento privado.
> 
> 

Para más información sobre las rutas definidas por el usuario, consulte [Enrutamiento del tráfico de redes virtuales][UDROverview].  

Para aprender a crear y configurar rutas definidas por el usuario, consulte [Enrutamiento del tráfico de red con una tabla de rutas mediante PowerShell][UDRHowTo].

## <a name="udr-configuration"></a>Configuración de una ruta definida por el usuario (UDR)

Esta sección muestra un ejemplo de configuración de UDR para App Service Environment.

### <a name="prerequisites"></a>Prerrequisitos

* Instale Azure PowerShell desde la [página de descargas de Azure][AzureDownloads]. Elija una descarga con una fecha de junio de 2015 u otra posterior. En **Herramientas de línea de comandos** > **Windows PowerShell**, seleccione **Instalar** para instalar los cmdlets de PowerShell más recientes.

* Cree una subred única para uso exclusivo de App Service Environment. Esta garantiza que las rutas definidas por el usuario aplicadas a la subred solo abrirán el tráfico saliente para App Service Environment.

> [!IMPORTANT]
> Implemente App Service Environment solo después de completar los pasos de configuración. Los pasos que se indican a continuación garantizan que la conectividad de red saliente esté disponible antes de intentar implementar App Service Environment.

### <a name="step-1-create-a-route-table"></a>Paso 1: Creación de una tabla de rutas

Cree una tabla de rutas llamada **DirectInternetRouteTable** en la región Oeste de EE. UU. como se indica en este fragmento de código:

`New-AzureRouteTable -Name 'DirectInternetRouteTable' -Location uswest`

### <a name="step-2-create-routes-in-the-table"></a>Paso 2: Creación de rutas en la tabla

Agregue rutas a la tabla de rutas para permitir el acceso saliente a Internet.  

Configure el acceso saliente a Internet. Defina una ruta para 0.0.0.0/0 como se muestra en este fragmento de código:

`Get-AzureRouteTable -Name 'DirectInternetRouteTable' | Set-AzureRoute -RouteName 'Direct Internet Range 0' -AddressPrefix 0.0.0.0/0 -NextHopType Internet`

0.0.0.0/0 es un intervalo de direcciones amplio. Los intervalos de direcciones que anuncia ExpressRoute son más específicos y reemplazan a este intervalo. Se debe usar una UDR con una ruta 0.0.0.0/0 junto con una configuración de ExpressRoute que solo anuncia 0.0.0.0/0. 

Como alternativa, descargue una lista actual y completa de los intervalos CIDR que usa Azure. El archivo XML para todos los intervalos de direcciones IP de Azure está disponible en el [Centro de descarga de Microsoft][DownloadCenterAddressRanges].  

> [!NOTE]
>
> Los intervalos de direcciones IP de Azure cambian con el tiempo. Las rutas definidas por el usuario necesitan actualizaciones periódicas manuales para mantener la sincronización.
>
> Una UDR individual tiene un límite superior predeterminado de 100 rutas. Es preciso "resumir" los intervalos de direcciones IP de Azure para que entren en ese límite. Las UDR deben ser más específicas que las rutas que se anuncian mediante la conexión de ExpressRoute.
> 

### <a name="step-3-associate-the-table-to-the-subnet"></a>Paso 3: Asociación de la tabla a la subred

Asocie la tabla de rutas a la subred en la que quiere implementar App Service Environment. El comando siguiente asocia la tabla **DirectInternetRouteTable** a la subred **ASESubnet** que contendrá la instancia de App Service Environment.

`Set-AzureSubnetRouteTable -VirtualNetworkName 'YourVirtualNetworkNameHere' -SubnetName 'ASESubnet' -RouteTableName 'DirectInternetRouteTable'`

### <a name="step-4-test-and-confirm-the-route"></a>Paso 4: Prueba y confirmación de la ruta

Una vez que la tabla de rutas esté enlazada a la subred, pruebe y confirme la ruta.

Implemente una máquina virtual en la subred y confirme estas condiciones:

* El tráfico saliente hacia los puntos de conexión de Azure, y los que no son de Azure, que se han descrito en este artículo **no** fluye de manera descendente al circuito de ExpressRoute. Si el tráfico saliente de la subred se produce por tunelización forzada a local, la creación de la instancia de App Service Environment siempre producirá un error.
* Las búsquedas de DNS para los puntos de conexión que se han descrito en este artículo se resolverán todas correctamente. 

Después de completar los pasos de configuración y confirmar la ruta, elimine la máquina virtual. La subred debe estar "vacía" cuando se cree la instancia de App Service Environment.

Ahora ya está listo para implementar App Service Environment.

## <a name="next-steps"></a>Pasos siguientes

Para empezar a trabajar con App Service Environment para PowerApps, consulte [Introducción a App Service Environment][IntroToAppServiceEnvironment].

<!-- LINKS -->
[virtualnetwork]: https://azure.microsoft.com/services/virtual-network/ 
[ExpressRoute]: https://azure.microsoft.com/services/expressroute/ 
[requiredports]: app-service-app-service-environment-control-inbound-traffic.md 
[NetworkSecurityGroups]: ../../virtual-network/virtual-network-vnet-plan-design-arm.md
[UDROverview]: ../../virtual-network/virtual-networks-udr-overview.md
<!-- Old link -- [UDRHowTo]: https://azure.microsoft.com/documentation/articles/virtual-networks-udr-how-to/ -->

[UDRHowTo]: ../../virtual-network/tutorial-create-route-table-powershell.md
[HowToCreateAnAppServiceEnvironment]: app-service-web-how-to-create-an-app-service-environment.md 
[AzureDownloads]: https://azure.microsoft.com/downloads/ 
[DownloadCenterAddressRanges]: https://www.microsoft.com/download/details.aspx?id=41653 
[NetworkSecurityGroups]: ../../virtual-network/virtual-network-vnet-plan-design-arm.md
[IntroToAppServiceEnvironment]:  app-service-app-service-environment-intro.md 
[NewPortal]:  https://portal.azure.com 


<!-- IMAGES -->