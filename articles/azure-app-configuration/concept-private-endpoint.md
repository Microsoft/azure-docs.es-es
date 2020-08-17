---
title: Uso de puntos de conexión privados en Azure App Configuration
description: Proteja el almacén de App Configuration mediante puntos de conexión privados.
services: azure-app-configuration
author: lisaguthrie
ms.service: azure-app-configuration
ms.topic: conceptual
ms.date: 3/12/2020
ms.author: lcozzens
ms.openlocfilehash: 3ec2a0e38f3bead5fbab8a119099bb5bbc3ded2e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87042125"
---
# <a name="using-private-endpoints-for-azure-app-configuration"></a>Uso de puntos de conexión privados en Azure App Configuration

Puede usar [puntos de conexión privados](../private-link/private-endpoint-overview.md) en Azure App Configuration para que los clientes de una red virtual puedan acceder de forma segura a los datos a través de un [vínculo privado](../private-link/private-link-overview.md). El punto de conexión privado usa una dirección IP del espacio de direcciones de la red virtual para el almacén de App Configuration. El tráfico de red entre los clientes de la red virtual y la cuenta de App Configuration atraviesa la red virtual usando un vínculo privado en la red troncal de Microsoft, lo que elimina la exposición a la red pública de Internet.

El uso de puntos de conexión privados en el almacén de App Configuration permite:
- Proteger los detalles de configuración de la aplicación mediante la configuración del firewall para bloquear todas las conexiones a App Configuration en el punto de conexión público.
- Aumentar la seguridad de la red virtual, lo que garantiza que los datos no escapen de ella.
- Conectarse de forma segura al almacén de App Configuration desde las redes locales que se conectan a la red virtual mediante [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md) o [ExpressRoute](../expressroute/expressroute-locations.md) con emparejamiento privado.

## <a name="conceptual-overview"></a>Información general conceptual

Un punto de conexión privado es una interfaz de red especial para un servicio de Azure de la [red virtual](../virtual-network/virtual-networks-overview.md). Cuando se crea un punto de conexión privado en el almacén de App Configuration, se consigue una conectividad segura entre los clientes de la red virtual y el almacén de configuración. Al punto de conexión privado se le asigna una dirección IP del intervalo de direcciones IP de la red virtual. La conexión entre el punto de conexión privado y el almacén de configuración usa un vínculo privado seguro.

Las aplicaciones de la red virtual se pueden conectar al almacén de configuración a través del punto de conexión privado **mediante las mismas cadenas de conexión y mecanismos de autorización que se usarían en cualquier otro caso**. Los puntos de conexión privados se pueden usar con todos los protocolos que se admiten en el almacén de App Configuration.

Aunque App Configuration no admite puntos de conexión de servicio, se pueden crear puntos de conexión privados en subredes que usen [puntos de conexión de servicio](../virtual-network/virtual-network-service-endpoints-overview.md). Los clientes de una subred pueden conectarse de forma segura a un almacén de App Configuration mediante el punto de conexión privado, al mismo tiempo que usan puntos de conexión de servicio para acceder a otros.  

Cuando se crea un punto de conexión privado para un servicio de la red virtual, se envía una solicitud de consentimiento para su aprobación al propietario de la cuenta de servicio. Si el usuario que solicita la creación del punto de conexión privado también es propietario de la cuenta, esta solicitud de consentimiento se aprueba automáticamente.

Los propietarios de cuentas de servicio pueden administrar las solicitudes de consentimiento y los puntos de conexión privados desde la pestaña `Private Endpoints` del almacén de configuración en [Azure Portal](https://portal.azure.com).

### <a name="private-endpoints-for-app-configuration"></a>Puntos de conexión privados en Azure App Configuration 

Al crear un punto de conexión privado, debe especificar el almacén de App Configuration al que se conecta. Si tiene varias instancias de App Configuration en una cuenta, necesita un punto de conexión privado independiente para cada almacén.

### <a name="connecting-to-private-endpoints"></a>Conexión a puntos de conexión privados

Azure se basa en la resolución de DNS para enrutar las conexiones desde la red virtual al almacén de configuración a través de un vínculo privado. Puede buscar rápidamente cadenas de conexiones en Azure Portal seleccionando el almacén de App Configuration y, después, **Configuración** > **Claves de acceso**.  

> [!IMPORTANT]
> Use la misma cadena de conexión para conectarse al almacén de App Configuration con puntos de conexión privados que usaría con un punto de conexión público. No se conecte al almacén mediante la dirección URL de su subdominio `privatelink`.

## <a name="dns-changes-for-private-endpoints"></a>Cambios de DNS en puntos de conexión privados

Al crear un punto de conexión privado, el registro del recurso CNAME de DNS del almacén de configuración se actualiza a un alias de un subdominio con el prefijo `privatelink`. Azure también crea una [zona DNS privada](../dns/private-dns-overview.md), que se corresponde con el subdominio `privatelink`, con los registros de recursos A de DNS para los puntos de conexión privados.

Cuando se resuelve la dirección URL del punto de conexión en la red virtual que hospeda el punto de conexión privado, lo hace en el punto de conexión privado del almacén. Cuando se resuelve desde fuera de la red virtual, lo hace en el punto de conexión público. Cuando se crea un punto de conexión privado, el punto de conexión público se deshabilita.

Si va a usar un servidor DNS personalizado en la red, los clientes deben ser capaces de resolver el nombre de dominio completo (FQDN) del punto de conexión de servicio en la dirección IP del punto de conexión privado. Configure el servidor DNS para delegar el subdominio del vínculo privado en la zona DNS privada de la red virtual o configure los registros A para `AppConfigInstanceA.privatelink.azconfig.io` con la dirección IP del punto de conexión privado.

> [!TIP]
> Cuando use un servidor DNS personalizado o local, debe configurarlo para resolver el nombre del almacén del subdominio `privatelink` en la dirección IP del punto de conexión privado. Para ello, puede delegar el subdominio `privatelink` en la zona DNS privada de la red virtual, o bien configurar la zona DNS en el servidor DNS y agregar los registros A de DNS.

## <a name="pricing"></a>Precios

La habilitación de puntos de conexión privados requiere un almacén de App Configuration de [nivel Estándar](https://azure.microsoft.com/pricing/details/app-configuration/).  Para más información sobre los precios de vínculos privados, consulte [Precios de Azure Private Link](https://azure.microsoft.com/pricing/details/private-link).

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre la creación de un punto de conexión privado para el almacén de App Configuration en los siguientes artículos:

- [Creación de un punto de conexión privado mediante Private Link Center en Azure Portal](../private-link/create-private-endpoint-portal.md)
- [Creación de un punto de conexión privado mediante la CLI de Azure](../private-link/create-private-endpoint-cli.md)
- [Creación de un punto de conexión privado mediante Azure PowerShell](../private-link/create-private-endpoint-powershell.md)

Aprenda a configurar el servidor DNS con puntos de conexión privados:

- [Resolución de nombres de recursos en redes virtuales de Azure](/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances#name-resolution-that-uses-your-own-dns-server)
- [Configuración de DNS para puntos de conexión privados](/azure/private-link/private-endpoint-overview#dns-configuration)
