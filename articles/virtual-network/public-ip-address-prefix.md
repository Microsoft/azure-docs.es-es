---
title: Prefijo de dirección IP pública de Azure
description: Obtenga información sobre qué es un prefijo de dirección IP pública de Azure y cómo puede ayudarle a asignar direcciones IP públicas predecibles a los recursos.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/29/2020
ms.author: allensu
ms.openlocfilehash: 53dd6d2dda762b3cbf53f4aaec6cd3692a9656e9
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87432580"
---
# <a name="public-ip-address-prefix"></a>Prefijo de dirección IP pública

Un prefijo de dirección IP pública es un rango reservado de direcciones IP en Azure. Azure proporciona a las suscripciones un rango contiguo de direcciones en función de la cantidad de ellas que se especifiquen. 

Si no está familiarizado con las direcciones públicas, consulte [Direcciones IP públicas](virtual-network-ip-addresses-overview-arm.md#public-ip-addresses).

Las direcciones IP públicas se asignan desde un grupo de direcciones en cada región de Azure. Puede [descargar](https://www.microsoft.com/download/details.aspx?id=56519) la lista de intervalos que Azure usa para cada región. Por ejemplo, 40.121.0.0/16 es uno de los más de 100 intervalos que Azure usa en la región Este de EE. UU. El intervalo incluye las direcciones utilizables de 40.121.0.1 a 40.121.255.254.

Especifique un nombre y cuántas direcciones desea que incluya el prefijo para crear un prefijo de dirección IP pública en una región de Azure y una suscripción. 

Los rangos de direcciones IP públicas se asignan con el prefijo que elija. Si crea un prefijo de /28, Azure proporciona16 direcciones IP de uno de sus rangos.

No se sabe qué intervalo asignará Azure hasta que se crea el intervalo, pero las direcciones son contiguas. 

Los prefijos de direcciones IP públicas tienen una cuota; para más información, consulte los [precios de las direcciones IP públicas](https://azure.microsoft.com/pricing/details/ip-addresses).

## <a name="why-create-a-public-ip-address-prefix"></a>¿Por qué crear un prefijo de dirección IP pública?

Cuando se crean recursos de dirección IP pública, Azure asigna una dirección IP pública disponible de cualquiera de los rangos que se usan en esa región. 

Hasta que Azure asigne la dirección IP, no se conocerá la IP exacta. Este proceso puede resultar problemático cuando se crean reglas de firewall que permiten direcciones IP concretas. Para cada dirección IP agregada, se debe agregar una regla de firewall correspondiente.

Si se asignan direcciones a los recursos desde un prefijo de dirección IP pública, no es preciso actualizar las reglas de firewall. Todo el rango se agrega a la regla.

## <a name="benefits"></a>Ventajas

- Los recursos de dirección IP pública se crean desde un rango conocido.
- Configuración de reglas de firewall con rangos que incluyen direcciones IP públicas que ya se han asignado y direcciones que no se han asignado aún. Esta configuración elimina la necesidad de cambiar las reglas de firewall cuando se asignan direcciones IP a los nuevos recursos.
- El tamaño predeterminado de un intervalo que se puede crear es /28 o 16 direcciones IP.
- No hay límites en cuanto al número de rangos que se pueden crear. Hay límites en el número máximo de direcciones IP públicas estáticas que puede haber en una suscripción de Azure. El número de rangos que se crean no puede abarcar más direcciones IP públicas de las que se pueden tener en una suscripción. Para más información, consulte [Límites de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).
- Las direcciones que se crean con direcciones del prefijo se pueden asignar a cualquier recurso de Azure al que se pueda asignar una dirección IP pública.
- Puede ver fácilmente qué direcciones IP se han proporcionado y cuáles aún no se han proporcionado dentro del rango.

## <a name="scenarios"></a>Escenarios
Puede asociar los siguientes recursos a una dirección IP pública estática desde un prefijo:

|Resource|Escenario|Pasos|
|---|---|---|
|Máquinas virtuales| La asociación de direcciones IP públicas desde un prefijo a las máquinas virtuales en Azure reduce la sobrecarga de administración que se produce cuando se agregan direcciones IP a una lista de direcciones IP permitidas en el firewall. Puede agregar un prefijo completo con una única regla de firewall. A medida que escala con máquinas virtuales de Azure, puede asociar direcciones IP del mismo prefijo para ahorrar costos, tiempo y sobrecarga de administración.| Para asociar direcciones IP de un prefijo a la máquina virtual: </br> 1. [Cree un prefijo.](manage-public-ip-address-prefix.md) </br> 2. [Cree una dirección IP del prefijo.](manage-public-ip-address-prefix.md) </br> 3. [Asocie la dirección IP a la interfaz de red de la máquina virtual.](virtual-network-network-interface-addresses.md#add-ip-addresses) </br> También puede [asociar las direcciones IP a un conjunto de escalado de máquinas virtuales](https://azure.microsoft.com/resources/templates/101-vmms-with-public-ip-prefix/).
| Equilibradores de carga estándar | La asociación de direcciones IP públicas de un prefijo a la configuración IP de front-end o a la regla de salida de un equilibrador de carga garantiza la simplificación del espacio de direcciones IP públicas de Azure. Para simplificar su escenario, limpie las conexiones salientes de un rango de direcciones IP contiguas. | Para asociar las direcciones IP de un prefijo a un equilibrador de carga: </br> 1. [Cree un prefijo.](manage-public-ip-address-prefix.md) </br> 2. [Cree una dirección IP del prefijo.](manage-public-ip-address-prefix.md) </br> 3. Al crear el equilibrador de carga, seleccione o actualice la dirección IP que creó en el paso 2 como dirección IP de front-end del equilibrador de carga. |
| Azure Firewall | Puede usar una dirección IP pública de un prefijo para la conexión SNAT de salida. Todo el tráfico de red virtual saliente se traslada a la dirección IP pública de [Azure Firewall](../firewall/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). | Para asociar una dirección IP de un prefijo a su firewall: </br> 1. [Cree un prefijo.](manage-public-ip-address-prefix.md) </br> 2. [Cree una dirección IP del prefijo.](manage-public-ip-address-prefix.md) </br> 3. Cuando [implemente el firewall de Azure](../firewall/tutorial-firewall-deploy-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json#deploy-the-firewall), no olvide seleccionar la dirección IP que previamente proporcionó desde el prefijo.|
| Application Gateway v2 | Puede usar una dirección IP pública de un prefijo para el escalado automático y Application Gateway v2 con redundancia de zona. | Para asociar una dirección IP de un prefijo a su puerta de enlace: </br> 1. [Cree un prefijo.](manage-public-ip-address-prefix.md) </br> 2. [Cree una dirección IP del prefijo.](manage-public-ip-address-prefix.md) </br> 3. Cuando [implemente Application Gateway](../application-gateway/quick-create-portal.md#create-an-application-gateway), asegúrese de seleccionar la dirección IP que proporcionó previamente desde el prefijo.|

## <a name="constraints"></a>Restricciones

- No se pueden especificar las direcciones IP del prefijo. Azure proporciona las direcciones IP para el prefijo, en función del tamaño que especifique.
- De manera predeterminada puede crear un prefijo de hasta 16 direcciones IP o un /28. Para más información, consulte [Solicitudes de aumento de los límites de redes](https://docs.microsoft.com/azure/azure-portal/supportability/networking-quota-requests) y [Límites de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).
- Una vez que se ha creado el prefijo no se puede cambiar el intervalo.
- Solo se pueden asignar direcciones IP públicas estáticas del intervalo del prefijo creadas con la SKU Estándar. Para más información sobre las SKU de direcciones IP públicas, consulte [Direcciones IP publicas](virtual-network-ip-addresses-overview-arm.md#public-ip-addresses).
- Las direcciones del intervalo solo se pueden asignar a recursos de Azure Resource Manager. En el modelo de implementación clásico no se pueden asignar direcciones a recursos.
- Todas las direcciones IP públicas creadas a partir del prefijo deben existir en la misma región y suscripción de Azure que el prefijo. Las direcciones se deben asignar a recursos de la misma región y suscripción.
- No se puede eliminar un prefijo si tiene direcciones asignadas a recursos de dirección IP públicas asociados a un recurso. En primer lugar, debe desasociar todos los recursos de dirección IP pública que tienen asignadas direcciones IP del prefijo.


## <a name="next-steps"></a>Pasos siguientes

- [Creación](manage-public-ip-address-prefix.md) de un prefijo de dirección IP pública
