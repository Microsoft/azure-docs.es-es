---
title: 'Azure ExpressRoute: Tablas de ARP: solución de problemas: clásico'
description: En esta página se proporcionan instrucciones para obtener tablas ARP para un circuito ExpressRoute (modelo de implementación clásico).
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: troubleshooting
ms.date: 01/30/2017
ms.author: duau
ms.openlocfilehash: 7c83b6d25936e184c70d4bf0a51636a3986bf4b2
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89394846"
---
# <a name="getting-arp-tables-in-the-classic-deployment-model"></a>Obtención de tablas ARP en el modelo de implementación clásica
> [!div class="op_single_selector"]
> * [PowerShell: administrador de recursos](expressroute-troubleshooting-arp-resource-manager.md)
> * [PowerShell: clásico](expressroute-troubleshooting-arp-classic.md)
> 
> 

Este artículo le guiará a través de los pasos para obtener las tablas del Protocolo de resolución de direcciones (ARP) para su circuito Azure ExpressRoute.

> [!IMPORTANT]
> Este documento está pensado para ayudarle a diagnosticar y corregir problemas sencillos. No pretende sustituir al soporte técnico de Microsoft. Si no puede solucionar el problema mediante las siguientes instrucciones, abra una solicitud de soporte técnico con [Ayuda y soporte técnico de Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).
> 
> 

## <a name="address-resolution-protocol-arp-and-arp-tables"></a>Protocolo de resolución de direcciones (ARP) y tablas ARP
ARP es un protocolo de nivel 2 que se define en [RFC 826](https://tools.ietf.org/html/rfc826). ARP se usa para asignar una dirección Ethernet (dirección MAC) a una dirección IP.

Una tabla ARP proporciona una asignación de la dirección IPv4 y la dirección MAC para un emparejamiento determinado. La tabla ARP de un emparejamiento de circuito ExpressRoute proporciona la siguiente información para cada interfaz (principal y secundaria):

1. Asignación de una dirección IP de la interfaz del enrutador local a la dirección MAC
2. Asignación de la dirección IP de la interfaz del enrutador de ExpressRoute a la dirección MAC
3. La antigüedad de la asignación

Las tablas ARP pueden ayudar a validar la configuración de capa 2 y a solucionar los problemas básicos de conectividad de nivel 2.

A continuación, se muestra un ejemplo de una tabla ARP:

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           10.0.0.1   ffff.eeee.dddd
  0 Microsoft         10.0.0.2   aaaa.bbbb.cccc
```


En la siguiente sección se proporciona información sobre cómo puede ver las tablas ARP que ven los enrutadores de borde de ExpressRoute.

## <a name="prerequisites-for-using-arp-tables"></a>Requisitos previos para usar las tablas ARP
Asegúrese de que tiene lo siguiente antes de continuar:

* Un circuito ExpressRoute válido configurado con al menos un emparejamiento. El circuito debe estar completamente configurado por el proveedor de conectividad. Usted (o su proveedor de conectividad) debe configurar al menos uno de los emparejamientos (privado de Azure, público de Azure o Microsoft) en este circuito.
* Intervalos de direcciones IP que se usan para configurar los emparejamientos (privado de Azure, público de Azure y Microsoft). Revise los ejemplos de asignación de dirección IP en [Requisitos de enrutamiento de ExpressRoute](expressroute-routing.md) para comprender cómo se asignan las direcciones IP a las interfaces en su sitio y en el sitio de ExpressRoute. Puede obtener información sobre la configuración de emparejamiento en la página [Creación y modificación del enrutamiento de un circuito ExpressRoute](expressroute-howto-routing-classic.md).
* Información del equipo de red o del proveedor de conectividad sobre las direcciones MAC de las interfaces que se usan con estas direcciones IP.
* El módulo más reciente de Windows PowerShell para Azure (versión 1.50 o superior).

## <a name="arp-tables-for-your-expressroute-circuit"></a>Tablas ARP para el circuito ExpressRoute
En esta sección se proporcionan instrucciones sobre cómo ver las tablas ARP para cada tipo de emparejamiento mediante PowerShell. Antes de continuar, usted o su proveedor de conectividad necesita configurar el emparejamiento. Cada circuito tiene dos rutas de acceso (principal y secundaria). Puede comprobar en la tabla ARP cada ruta de acceso de forma independiente.

### <a name="arp-tables-for-azure-private-peering"></a>Tablas ARP para el emparejamiento privado de Azure
El siguiente cmdlet proporciona el emparejamiento privado de Azure para las tablas ARP:

```azurepowershell
# Required variables
$ckt = "<your Service Key here>

# ARP table for Azure private peering--primary path
Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Private -Path Primary

# ARP table for Azure private peering--secondary path
Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Private -Path Secondary
```

A continuación, se muestra el resultado de ejemplo de una de las rutas de acceso:

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           10.0.0.1   ffff.eeee.dddd
  0 Microsoft         10.0.0.2   aaaa.bbbb.cccc
```


### <a name="arp-tables-for-azure-public-peering"></a>Tablas ARP para el emparejamiento público de Azure:
El siguiente cmdlet proporciona las tablas ARP para el emparejamiento público de Azure:

```azurepowershell
# Required variables
$ckt = "<your Service Key here>

# ARP table for Azure public peering--primary path
Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Public -Path Primary

# ARP table for Azure public peering--secondary path
Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Public -Path Secondary
```

A continuación, se muestra el resultado de ejemplo de una de las rutas de acceso:

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           10.0.0.1   ffff.eeee.dddd
  0 Microsoft         10.0.0.2   aaaa.bbbb.cccc
```


A continuación, se muestra el resultado de ejemplo de una de las rutas de acceso:

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           64.0.0.1   ffff.eeee.dddd
  0 Microsoft         64.0.0.2   aaaa.bbbb.cccc
```


### <a name="arp-tables-for-microsoft-peering"></a>Tablas ARP para el emparejamiento de Microsoft
El siguiente cmdlet proporciona las tablas ARP para el emparejamiento de Microsoft:

```azurepowershell
# ARP table for Microsoft peering--primary path
Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Microsoft -Path Primary

# ARP table for Microsoft peering--secondary path
Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Microsoft -Path Secondary
```


A continuación se muestra el resultado de ejemplo de una de las rutas de acceso:

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           65.0.0.1   ffff.eeee.dddd
  0 Microsoft         65.0.0.2   aaaa.bbbb.cccc
```


## <a name="how-to-use-this-information"></a>Uso de esta información
La tabla ARP de un emparejamiento se puede usar para validar la configuración y la conectividad de nivel 2. En esta sección se proporciona información general sobre la apariencia de las tablas ARP en distintos escenarios.

### <a name="arp-table-when-a-circuit-is-in-an-operational-expected-state"></a>Tabla ARP cuando un circuito está en un estado operativo (esperado)
* La tabla ARP tiene una entrada para el lado local con una dirección IP válida y la dirección MAC, y una entrada similar para el lado de Microsoft.
* El último octeto de la dirección IP local siempre será un número impar.
* El último octeto de la dirección IP de Microsoft siempre será un número par.
* La misma dirección MAC aparece en el lado de Microsoft para los tres emparejamientos (principales o secundarios).

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           65.0.0.1   ffff.eeee.dddd
  0 Microsoft         65.0.0.2   aaaa.bbbb.cccc
```

### <a name="arp-table-when-its-on-premises-or-when-the-connectivity-provider-side-has-problems"></a>Tabla ARP cuando es local o cuando el lado del proveedor de conectividad tiene problemas
 Solo aparece una entrada en la tabla ARP. Muestra la asignación entre la dirección MAC y la dirección IP que se usa en el lado de Microsoft.

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
   0 Microsoft         65.0.0.2  aaaa.bbbb.cccc
```

> [!NOTE]
> Si experimenta un problema como este, abra una solicitud de soporte con su proveedor de conectividad para resolverlo.
> 
> 

### <a name="arp-table-when-the-microsoft-side-has-problems"></a>Tabla ARP cuando el lado de Microsoft tiene problemas
* Cuando hay problemas en el lado de Microsoft, no verá una tabla ARP para un emparejamiento.
* Abra una solicitud de soporte con [Ayuda y soporte técnico de Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade). Especifique que tiene un problema con la conectividad de nivel 2.

## <a name="next-steps"></a>Pasos siguientes
* Validar las configuraciones de nivel 3 para el circuito ExpressRoute:
  * Obtenga un resumen de ruta para determinar el estado de las sesiones BGP.
  * Obtenga una tabla de rutas para determinar qué prefijos se anuncian a través de ExpressRoute.
* Validar la transferencia de datos revisando los bytes de entrada y de salida.
* Si sigue teniendo problemas, abra una solicitud de soporte técnico en [Ayuda y soporte técnico de Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) .

