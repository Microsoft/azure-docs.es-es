---
title: Prefijo de dirección IP pública de Azure | Microsoft Docs
description: Obtenga información sobre qué es un prefijo de dirección IP pública de Azure y cómo puede ayudarle a asignar direcciones IP públicas predecibles a los recursos.
services: virtual-network
documentationcenter: na
author: anavinahar
manager: narayan
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/24/2018
ms.author: anavin
ms.openlocfilehash: 23cd77d4a2d0c8203670039dd44c878bf7217fd3
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65799120"
---
# <a name="public-ip-address-prefix"></a>Prefijo de dirección IP pública

Un prefijo de dirección IP pública es un intervalo reservado de direcciones IP para los puntos de conexión públicos de Azure. Azure asigna un intervalo contiguo de direcciones a la suscripción en función del número especificado. Si no está familiarizado con las direcciones públicas, consulte [Direcciones IP públicas](virtual-network-ip-addresses-overview-arm.md#public-ip-addresses).

Las direcciones IP públicas se asignan desde un grupo de direcciones en cada región de Azure. Puede [descargar](https://www.microsoft.com/download/details.aspx?id=56519) la lista de intervalos que Azure usa para cada región. Por ejemplo, 40.121.0.0/16 es uno de los más de 100 intervalos que Azure usa en la región Este de EE. UU. El intervalo incluye las direcciones utilizables de 40.121.0.1 a 40.121.255.254.

Especifique un nombre y cuántas direcciones desea que incluya el prefijo para crear un prefijo de dirección IP pública en una región de Azure y una suscripción. Por ejemplo, si crea un prefijo de dirección IP pública /28, Azure asigna 16 direcciones de uno de los intervalos. No se sabe qué intervalo asignará Azure hasta que se crea el intervalo, pero las direcciones son contiguas. Los prefijos de direcciones IP públicas tienen una cuota. Para más información, consulte [Precios de direcciones IP públicas](https://azure.microsoft.com/pricing/details/ip-addresses).

## <a name="why-create-a-public-ip-address-prefix"></a>¿Por qué crear un prefijo de dirección IP pública?

Al crear recursos de dirección IP pública, Azure asigna una dirección IP pública disponible de cualquiera de los intervalos usados en una región. Una vez que Azure asigna la dirección, se sabe cuál es la dirección, pero hasta que Azure asigna la dirección, no se sabe qué dirección podría asignar. Esto puede ser problemático cuando, por ejemplo, usted o sus socios comerciales configuran reglas de firewall para permitir direcciones IP específicas. Cada vez que se asigna una nueva dirección IP pública a un recurso, la dirección se debe agregar a la regla de firewall. Al asignar direcciones a los recursos desde un prefijo de dirección IP pública, no es necesario actualizar las reglas de firewall cada vez que se asigna una de las direcciones, porque se puede agregar todo el intervalo a una regla.

## <a name="benefits"></a>Ventajas

- Puede crear recursos de dirección IP pública desde un intervalo conocido.
- Usted o sus socios comerciales pueden crear reglas de firewall con intervalos que incluyen direcciones IP públicas que ha asignado actualmente, así como direcciones que no se han asignado todavía. Esto elimina la necesidad de cambiar las reglas de firewall cuando se asignan direcciones IP a los nuevos recursos.
- El tamaño predeterminado de un intervalo que se puede crear es /28 o 16 direcciones IP.
- No existen límites sobre cuántos intervalos se pueden crear, sin embargo, hay límites en el número máximo de direcciones IP públicas estáticas que puede tener en una suscripción de Azure. Como resultado, el número de intervalos que se creen no puede abarcar más direcciones IP públicas de las que puede tener en la suscripción. Para más información, consulte [Límites de Azure](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).
- Las direcciones que se crean con direcciones del prefijo se pueden asignar a cualquier recurso de Azure al que se pueda asignar una dirección IP pública.
- Puede ver fácilmente qué direcciones IP se han asignado y las que aún no se han asignado dentro del intervalo.

## <a name="scenarios"></a>Escenarios
Puede asociar los siguientes recursos a una dirección IP pública estática desde un prefijo:

|Recurso|Escenario|Pasos|
|---|---|---|
|Virtual Machines| La asociación de direcciones IP públicas de un prefijo a las máquinas virtuales de Azure reduce la sobrecarga de administración en cuanto a la inclusión de direcciones IP en la lista de permitidos de un firewall. Puede simplemente incluir en la lista de permitidos un prefijo completo con una única regla de firewall. A medida que escala con máquinas virtuales de Azure, puede asociar direcciones IP del mismo prefijo para ahorrar costos, tiempo y sobrecarga de administración.| Para asociar direcciones IP de un prefijo a la máquina virtual: 1. [Cree un prefijo.](manage-public-ip-address-prefix.md) 2. [Cree una dirección IP del prefijo.](manage-public-ip-address-prefix.md) 3. [Asocie la dirección IP a la interfaz de red de la máquina virtual.](virtual-network-network-interface-addresses.md#add-ip-addresses)
| Equilibradores de carga | La asociación de direcciones IP públicas de un prefijo a la configuración IP de front-end o a la regla de salida de un equilibrador de carga garantiza la simplificación del espacio de direcciones IP públicas de Azure. Puede simplificar su escenario haciendo que las conexiones salientes se originen desde un intervalo de direcciones IP contiguas definido por el prefijo de direcciones IP públicas. | Para asociar direcciones IP de un prefijo al equilibrador de carga: 1. [Cree un prefijo.](manage-public-ip-address-prefix.md) 2. [Cree una dirección IP del prefijo.](manage-public-ip-address-prefix.md) 3. Al crear el equilibrador de carga, seleccione o actualice la dirección IP que creó en el paso 2 anterior como la dirección IP de front-end del equilibrador de carga. |
| Azure Firewall | Puede usar una dirección IP pública de un prefijo para la conexión SNAT de salida. Esto significa que todo tráfico de red virtual saliente se traduce a la dirección IP pública de [Azure Firewall](../firewall/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Puesto que esta dirección IP procede de un prefijo predeterminado, es muy fácil saber de antemano el aspecto que tendrá la superficie IP pública en Azure. | 1. [Cree un prefijo.](manage-public-ip-address-prefix.md) 2. [Cree una dirección IP del prefijo.](manage-public-ip-address-prefix.md) 3. Cuando [implemente Azure Firewall](../firewall/tutorial-firewall-deploy-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json#deploy-the-firewall), no olvide seleccionar la dirección IP asignada previamente desde el prefijo.|

## <a name="constraints"></a>Restricciones

- No se pueden especificar las direcciones IP del prefijo. Azure asigna las direcciones IP para el prefijo, según el tamaño que especifique.
- Puede crear un prefijo de hasta 16 direcciones IP o una de/28. Para más información, consulte [Límites de Azure](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).
- Una vez que se ha creado el prefijo no se puede cambiar el intervalo.
- El intervalo es solo para direcciones IPv4. El intervalo no contiene direcciones IPv6.
- Solo se pueden asignar direcciones IP públicas estáticas del intervalo del prefijo creadas con la SKU Estándar. Para más información sobre las SKU de direcciones IP públicas, consulte [Direcciones IP publicas](virtual-network-ip-addresses-overview-arm.md#public-ip-addresses).
- Las direcciones del intervalo solo se pueden asignar a recursos de Azure Resource Manager. Las direcciones no se pueden asignar a recursos creados mediante el modelo de implementación clásica.
- Todas las direcciones IP públicas creadas a partir del prefijo deben existir en la misma región y suscripción de Azure que el prefijo y se deben asignar a recursos de la misma región y suscripción.
- No se puede eliminar un prefijo si tiene direcciones asignadas a recursos de dirección IP públicas asociados a un recurso. En primer lugar, debe desasociar todos los recursos de dirección IP pública que tienen asignadas direcciones IP del prefijo.


## <a name="next-steps"></a>Pasos siguientes

- [Creación](manage-public-ip-address-prefix.md) de un prefijo de dirección IP pública
