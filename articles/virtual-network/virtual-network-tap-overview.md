---
title: Introducción al TAP de Virtual Network | Microsoft Docs
description: Obtenga más información sobre Virtual Network TAP. Virtual Network TAP proporciona una copia en profundidad del tráfico de red de la máquina virtual que se puede transmitir a un recopilador de paquetes.
services: virtual-network
documentationcenter: na
author: karthikananth
manager: ganesr
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/14/2019
ms.author: kaanan
ms.openlocfilehash: 7013c8ed338e727dd79a3845ff3b85749c0f5cee
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836095"
---
# <a name="virtual-network-tap"></a>TAP de red virtual
> [!IMPORTANT]
> La versión preliminar de TAP de red virtual actualmente está en espera en todas las regiones de Azure. Puede enviarnos un correo electrónico a <azurevnettap@microsoft.com> con su identificador de suscripción y le notificaremos las actualizaciones futuras sobre la versión preliminar. Mientras tanto, puede usar soluciones basadas en agentes o NVA que proporcionen funcionalidad de visibilidad de red o TAP a través de las [soluciones de partners de agentes de paquetes](#virtual-network-tap-partner-solutions) disponibles en las [ofertas de Azure Marketplace](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/category/networking?page=1&subcategories=appliances%3Ball&search=Network%20Traffic&filters=partners).

Azure Virtual Network TAP (punto de acceso del terminal) permite transmitir continuamente el tráfico de red de la máquina virtual a un recopilador de paquetes de red o a una herramienta de análisis. Un asociado de la [aplicación virtual de red](https://azure.microsoft.com/solutions/network-appliances/) proporciona el recopilador o la herramienta de análisis de la herramienta. Para ver una lista de soluciones de asociados que estén validadas para funcionar con Virtual Network TAP, consulte las [soluciones de asociados](#virtual-network-tap-partner-solutions).
En la siguiente imagen se muestra cómo funciona Virtual Network TAP. Puede agregar una configuración de TAP en una [interfaz de red](virtual-network-network-interface.md) adjunta a una máquina virtual implementada en la red virtual. El destino es una dirección IP de red virtual en la misma red virtual que la interfaz de red supervisada o una red [virtual emparejada](virtual-network-peering-overview.md). La solución del recopilador para el punto de acceso de terminal de red virtual se puede implementar detrás de un equilibrador de carga interno de Azure para lograr alta disponibilidad.
![Cómo funciona Virtual Network TAP](./media/virtual-network-tap/architecture.png)

## <a name="prerequisites"></a>Prerrequisitos

Antes de crear un Virtual Network TAP, debe haber recibido un correo electrónico de confirmación de su inscripción a la versión preliminar y debe haber creado una o más máquinas virtuales usando el modelo de implementación de [Azure Resource Manager](../azure-resource-manager/management/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) y una solución de un partner para agregar el tráfico de TAP en la misma región de Azure. Si no tiene ninguna solución de partner en la red virtual, consulte [soluciones de partners](#virtual-network-tap-partner-solutions) para implementar una. Puede usar el mismo recurso de Virtual Red TAP para agregar tráfico de diferentes interfaces de red en la misma suscripción o en suscripciones distintas. Si las interfaces de red supervisadas se encuentran en suscripciones distintas, ambas suscripciones deben estar asociadas al mismo inquilino de Azure Active Directory. Además, las interfaces de red supervisadas y el punto de conexión de destino para agregar tráfico del TAP pueden estar en redes virtuales emparejadas en la misma región. Si usa este modelo de implementación, asegúrese de que el [emparejamiento de redes virtuales](virtual-network-peering-overview.md) está habilitada antes de configurar Virtual Network TAP.

## <a name="permissions"></a>Permisos

Las cuentas que utiliza para aplicar la configuración del TAP en interfaces de red deben estar asignadas al rol de [colaborador de red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) o a un [rol personalizado](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que tenga asignadas las acciones necesarias de la tabla siguiente:

| Acción | Nombre |
|---|---|
| Microsoft.Network/virtualNetworkTaps/* | Necesaria para crear, actualizar, leer y eliminar un recurso de Virtual Network TAP |
| Microsoft.Network/networkInterfaces/read | Necesaria para leer el recurso de la interfaz de red en que el TAP se configurará |
| Microsoft.Network/tapConfigurations/* | Necesaria para crear, actualizar, leer y eliminar la configuración del TAP en una interfaz de red |


## <a name="virtual-network-tap-partner-solutions"></a>Soluciones de los partners de Virtual Network TAP

### <a name="network-packet-brokers"></a>Agentes de paquetes de red

- [GigaSECURE de Gigamon](https://blog.gigamon.com/2018/09/13/why-microsofts-new-vtap-service-works-even-better-with-gigasecure-for-azure)
- [CloudLens de Ixia](https://www.ixiacom.com/cloudlens/cloudlens-azure)
- [Prisms de Nubeva](https://www.nubeva.com/azurevtap)
- [Big Monitoring Fabric de Big Switch](https://www.bigswitch.com/products/big-monitoring-fabric/public-cloud/microsoft-azure)

### <a name="security-analytics-networkapplication-performance-management"></a>Análisis de seguridad, administración del rendimiento de red o de la aplicación

- [Awake Security](https://awakesecurity.com/technology-partners/microsoft-azure/)
- [Cisco Stealthwatch Cloud](https://blogs.cisco.com/security/cisco-stealthwatch-cloud-and-microsoft-azure-reliable-cloud-infrastructure-meets-comprehensive-cloud-security)
- [Darktrace](https://www.darktrace.com/en/azure/)
- [Reveal(x) de ExtraHop](https://www.extrahop.com/partners/tech-partners/microsoft/)
- [Cybersecurity de Fidelis](https://www.fidelissecurity.com/technology-partners/microsoft-azure )
- [Flowmon](https://www.flowmon.com/blog/azure-vtap)
- [NetFort LANGuardian](https://www.netfort.com/languardian/solutions/visibility-in-azure-network-tap/)
- [vSTREAM de Netscout]( https://www.netscout.com/technology-partners/microsoft/azure-vtap)
- [Riverbed SteelCentral AppResponse]( https://www.riverbed.com/products/steelcentral/steelcentral-appresponse-11.html)
- [NetWitness® Platform de RSA](https://www.rsa.com/azure)
- [Cognito de Vectra](https://vectra.ai/microsoftazure)



## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo [crear un Virtual Network TAP](tutorial-tap-virtual-network-cli.md).
