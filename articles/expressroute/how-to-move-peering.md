---
title: 'Azure ExpressRoute: Cambiar un emparejamiento público a emparejamiento de Microsoft'
description: En este artículo se detallan los pasos para cambiar un emparejamiento público a emparejamiento de Microsoft en ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 12/12/2019
ms.author: duau
ms.openlocfilehash: 5671be5e54a9bb789e349a4da6d0f1809807d974
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401425"
---
# <a name="move-a-public-peering-to-microsoft-peering"></a>Cambiar un emparejamiento público a emparejamiento de Microsoft

Este artículo le servirá para cambiar una configuración de emparejamiento público a emparejamiento de Microsoft sin sufrir ningún tiempo de inactividad. ExpressRoute admite el uso del emparejamiento de Microsoft con filtros de ruta para los servicios de PaaS de Azure, como Azure Storage y Azure SQL Database. Ahora solo se necesita un dominio de enrutamiento para tener acceso a PaaS de Microsoft y a los servicios de SaaS. Puede usar los filtros de ruta para anunciar los prefijos del servicio de PaaS de forma selectiva en las regiones de Azure que quiere que se usen.

El emparejamiento público de Azure tiene 1 dirección IP de NAT asociada a cada sesión BGP. El emparejamiento de Microsoft le permite configurar sus propias asignaciones de NAT, así como usar filtros de ruta para los anuncios de prefijo selectivo. El emparejamiento público es un servicio unidireccional en el que la conectividad siempre se inicia desde la WAN a los servicios de Microsoft Azure. Servicios de Microsoft Azure no podrá iniciar conexiones en la red a través de este dominio de enrutamiento.

Una vez que se habilita el emparejamiento público, podrá conectarse a todos los servicios de Azure. No se permite elegir de forma selectiva servicios para los que anunciemos rutas. Por su parte, el emparejamiento de Microsoft es una conectividad bidireccional donde la conexión puede iniciarse desde el servicio de Microsoft Azure, junto con la WAN. Para más información sobre cómo enrutar emparejamientos y dominios de enrutamiento, vea [Circuitos ExpressRoute y dominios de enrutamiento](expressroute-circuit-peerings.md).

## <a name="before-you-begin"></a><a name="before"></a>Antes de empezar

Para conectarse al emparejamiento de Microsoft, necesita configurar y administrar NAT. Puede que su proveedor de conectividad haya configurado y administrado NAT como un servicio administrado. Si tiene previsto acceder a servicios de PaaS y SaaS de Azure en el emparejamiento de Microsoft, es importante cambiar correctamente el tamaño del grupo de direcciones IP de NAT. Para más información sobre NAT para ExpressRoute, vea [Requisitos NAT para el emparejamiento de Microsoft](expressroute-nat.md#nat-requirements-for-microsoft-peering). Cuando se conecta a Microsoft mediante Azure ExpressRoute (emparejamiento de Microsoft), tendrá varios vínculos a Microsoft. Un vínculo es la conexión existente de Internet y el otro se realiza a través de ExpressRoute. Una parte del tráfico a Microsoft puede ir a través de Internet pero volver a través de ExpressRoute o viceversa.

![Conectividad bidireccional](./media/how-to-move-peering/bidirectional-connectivity.jpg)

> [!Warning]
> El grupo de direcciones IP NAT anunciado a Microsoft no se debe anunciar a Internet. Esto interrumpirá la conectividad con otros servicios de Microsoft.

Consulte [Enrutamiento asimétrico con varias rutas de acceso de red](https://docs.microsoft.com/azure/expressroute/expressroute-asymmetric-routing) para ver advertencias de enrutamiento asimétrico antes de configurar el emparejamiento de Microsoft.

* Si usa el emparejamiento público y actualmente tiene reglas de red IP para las direcciones IP públicas que se usan para acceder a [Azure Storage](../storage/common/storage-network-security.md) o [Azure SQL Database](../azure-sql/database/vnet-service-endpoint-rule-overview.md), tiene que asegurarse de que el grupo de IP de NAT configurado con el emparejamiento de Microsoft se incluye en la lista de direcciones IP públicas de la cuenta de Azure Storage o Azure SQL.<br>
* Para cambiar a un emparejamiento de Microsoft sin sufrir tiempos de inactividad, realice los pasos de este artículo en el orden en el que aparecen.

## <a name="1-create-microsoft-peering"></a><a name="create"></a>1. Crear el emparejamiento de Microsoft

Si no ha creado el emparejamiento de Microsoft, use cualquiera de los siguientes artículos para crearlo. Si su proveedor de conectividad presta servicios administrados de nivel 3, puede pedirle que habilite el emparejamiento de Microsoft en su circuito.

Si es usted quien administra el nivel 3, la siguiente información es necesaria antes de continuar:

* Una subred /30 para el vínculo principal. Debe ser un prefijo de IPv4 público válido que sea de su propiedad y esté registrado en un Registro regional de Internet (RIR) o un Registro de enrutamiento de Internet (IRR). Desde esta subred asignará la primera dirección IP utilizable para el enrutador, ya que Microsoft usa la segunda dirección IP utilizable para su enrutador.<br>
* Una subred /30 para el vínculo secundario. Debe ser un prefijo de IPv4 público válido que sea de su propiedad y esté registrado en un Registro regional de Internet (RIR) o un Registro de enrutamiento de Internet (IRR). Desde esta subred asignará la primera dirección IP utilizable para el enrutador, ya que Microsoft usa la segunda dirección IP utilizable para su enrutador.<br>
* Un identificador VLAN válido para establecer esta configuración entre pares. Asegúrese de que ninguna otra configuración entre pares en el circuito usa el mismo identificador de VLAN. Para los vínculos principal y secundario, debe usar el mismo identificador de VLAN.<br>
* Número de sistema autónomo (AS) para la configuración entre pares. Puede usar 2 bytes o 4 bytes como números AS.<br>
* Prefijos anunciados: tiene que proporcionar una lista de todos los prefijos que planea anunciar en la sesión BGP. Se aceptan solo prefijos de direcciones IP públicas. Si tiene pensado enviar un conjunto de prefijos, puede enviar una lista separada por comas. Estos prefijos tienen que estar registrados a su nombre en un Registro regional de Internet (RIR) o un Registro de enrutamiento de Internet (IRR).<br>
* Nombre del enrutamiento del Registro: puede especificar el RIR o TIR en el que están registrados el número AS y los prefijos.

* **Opcional**: ASN de cliente: si anuncia prefijos que no están registrados en el número de AS de emparejamiento, puede especificar el número de AS en el que están registrados.<br>
* **Opcional**: un hash MD5 si elige usar uno.

En los siguientes artículos encontrará instrucciones detalladas para habilitar el emparejamiento de Microsoft:

* [Creación del emparejamiento de Microsoft con Azure Portal](expressroute-howto-routing-portal-resource-manager.md#msft)<br>
* [Creación del emparejamiento de Microsoft con Azure PowerShell](expressroute-howto-routing-arm.md#msft)<br>
* [Creación del emparejamiento de Microsoft con la CLI de Azure](howto-routing-cli.md#msft)

## <a name="2-validate-microsoft-peering-is-enabled"></a><a name="validate"></a>2. Confirmar que el emparejamiento de Microsoft está habilitado

Compruebe que el emparejamiento de Microsoft está habilitado y que los prefijos públicos anunciados están en estado configurado.

* [Azure Portal](expressroute-howto-routing-portal-resource-manager.md#getmsft)<br>
* [Azure PowerShell](expressroute-howto-routing-arm.md#getmsft)<br>
* [CLI de Azure](howto-routing-cli.md#getmsft)

## <a name="3-configure-and-attach-a-route-filter-to-the-circuit"></a><a name="routefilter"></a>3. Configurar un filtro de ruta y adjuntarlo al circuito

De forma predeterminada, los emparejamientos de Microsoft nuevos no publican ningún prefijo hasta que se adjunta un filtro de ruta al circuito. Cuando se crea una regla de filtro de ruta, puede especificar la lista de comunidades de servicio correspondientes a las regiones de Azure que quiere usar para los servicios de PaaS de Azure. Esto le proporciona flexibilidad para filtrar las rutas según sus necesidades, como se muestra en la siguiente captura de pantalla:

![Combinar emparejamientos públicos](./media/how-to-move-peering/routefilter.jpg)

Para configurar filtros de ruta, use cualquiera de los siguientes artículos:

* [Configuración de filtros de ruta para el emparejamiento de Microsoft: Azure Portal](how-to-routefilter-portal.md)<br>
* [Configuración de filtros de ruta para el emparejamiento de Microsoft: PowerShell](how-to-routefilter-powershell.md)<br>
* [Configuración de filtros de ruta para el emparejamiento de Microsoft: CLI de Azure](how-to-routefilter-cli.md)

## <a name="4-delete-the-public-peering"></a><a name="delete"></a>4. Eliminar el emparejamiento público

Tras constatar que el emparejamiento de Microsoft está configurado y que los prefijos que se van a usar se anuncian correctamente en el emparejamiento de Microsoft, puede eliminar el emparejamiento público. Para eliminar el emparejamiento público, use cualquiera de los siguientes artículos:

* [Eliminación del emparejamiento público de Azure con Azure PowerShell](about-public-peering.md#powershell)
* [Eliminación del emparejamiento público de Azure con la CLI](about-public-peering.md#cli)
  
## <a name="5-view-peerings"></a><a name="view"></a>5. Visualización de emparejamientos
  
Puede ver una lista de todos los circuitos y emparejamientos de ExpressRoute en Azure Portal. Para obtener más información, consulte [Visualización de detalles del emparejamiento de Microsoft](expressroute-howto-routing-portal-resource-manager.md#getmsft).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de ExpressRoute, consulte [P+F de ExpressRoute](expressroute-faqs.md).
