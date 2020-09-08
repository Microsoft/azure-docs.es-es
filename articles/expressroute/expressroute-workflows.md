---
title: 'Azure ExpressRoute: Flujos de trabajo de configuración de circuito'
description: En esta página se muestran los flujos de trabajo para configurar el circuito ExpressRoute y los emparejamientos
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 08/24/2020
ms.author: duau
ms.custom: contperfq1
ms.openlocfilehash: 14a61d33ef2e4d4fc80770f5c86e33d34ec860cd
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401442"
---
# <a name="expressroute-workflows-for-circuit-provisioning-and-circuit-states"></a>Flujos de trabajo de ExpressRoute para aprovisionamiento de circuitos y estados de circuitos de ExpressRoute

Este artículo le guiará a través del aprovisionamiento de servicios y de los flujos de trabajo de configuración del enrutamiento a alto nivel. En las secciones siguientes se describen las tareas para aprovisionar un circuito ExpressRoute de un extremo a otro.

## <a name="workflow-steps"></a>Pasos del flujo de trabajo

### <a name="1-prerequisites"></a>1. Prerrequisitos

Asegúrese de que se cumplen los requisitos previos. Para obtener una lista completa, consulte [Requisitos previos y lista de comprobación](expressroute-prerequisites.md).

* Se ha creado una suscripción de Azure.
* La conectividad física se estableció con el asociado de ExpressRoute o se configuró mediante ExpressRoute Direct. Revise la ubicación, consulte [Ubicaciones y asociados](expressroute-locations-providers.md#partners) para ver el asociado de ExpressRoute y la conectividad de ExpressRoute Direct a través de las ubicaciones de emparejamiento.

### <a name="2-order-connectivity-or-configure-expressroute-direct"></a>2. Solicitud de conectividad o configuración de ExpressRoute Direct

Solicite conectividad desde el proveedor de servicios o configure ExpressRoute Direct.

#### <a name="expressroute-partner-model"></a>Modelo de asociado de ExpressRoute

Solicite conectividad al proveedor de servicios. Este proceso varía. Para obtener más información sobre cómo solicitar conectividad, póngase en contacto con su proveedor de conectividad.

* Seleccione el asociado de ExpressRoute
* Seleccione la ubicación de emparejamiento
* Seleccione el ancho de banda
* Seleccione el modelo de facturación
* Seleccione el complemento Estándar o Premium

#### <a name="expressroute-direct-model"></a>Modelo de ExpressRoute Direct

* Vea la capacidad de ExpressRoute Direct disponible en las ubicaciones de emparejamiento.
* Reserve puertos mediante la creación de recursos de ExpressRoute Direct en su suscripción de Azure.
* Solicite y reciba la carta de autorización y solicite conexiones cruzadas físicas del proveedor de la ubicación de emparejamiento.
* Habilite el estado de administración y vea los niveles de luz y el vínculo físico mediante [Azure Monitor](expressroute-monitoring-metrics-alerts.md#expressroute-direct-metrics).

### <a name="3-create-an-expressroute-circuit"></a>3. Creación de un circuito ExpressRoute

#### <a name="expressroute-partner-model"></a>Modelo de asociado de ExpressRoute

Compruebe que el asociado de ExpressRoute está listo para aprovisionar la conectividad. El circuito ExpressRoute se factura a partir del momento en que se emite una clave de servicio. Use las instrucciones que aparecen en [Creación de un circuito ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md) para crear un circuito.

#### <a name="expressroute-direct-model"></a>Modelo de ExpressRoute Direct

Asegúrese de que el vínculo físico y el estado de administración estén habilitados en ambos vínculos. Consulte [Configuración de ExpressRoute Direct](how-to-expressroute-direct-portal.md) para obtener instrucciones. El circuito ExpressRoute se factura a partir del momento en que se emite una clave de servicio. Use las instrucciones que aparecen en [Creación de un circuito ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md) para crear un circuito.

### <a name="4-service-provider-provisions-connectivity"></a>4. El proveedor de servicios aprovisiona la conectividad

Esta sección solo se aplica al modelo de conectividad de asociados de ExpressRoute:

* Proporcione la clave de servicio al proveedor de conectividad.
* Proporcione la información adicional necesaria para el proveedor de conectividad (por ejemplo, el id. de VPN).
* Si el proveedor administra la configuración de enrutamiento, proporcione los datos necesarios.

Puede asegurarse de que el circuito se ha aprovisionado correctamente al comprobar el estado de aprovisionamiento del circuito ExpressRoute a través de PowerShell, Azure Portal o la CLI.

### <a name="5-configure-routing-domains"></a>5. Configuración de los dominios de enrutamiento

Configure los dominios de enrutamiento. Si el proveedor de conectividad administra la configuración de nivel 3, configurará el enrutamiento del circuito. Si el proveedor de conectividad solo ofrece servicios de nivel 2, o si está usando ExpressRoute Direct, debe configurar el enrutamiento según las instrucciones que se describen en las páginas de [requisitos de enrutamiento](expressroute-routing.md) y [configuración de enrutamiento](expressroute-howto-routing-classic.md).

#### <a name="for-azure-private-peering"></a>En el caso del emparejamiento privado de Azure

Habilite el emparejamiento privado para conectarse a VM y servicios en la nube implementados en la red virtual de Azure.

* Subred de emparejamiento para la ruta de acceso 1 (/30)
* Subred de emparejamiento para la ruta de acceso 2 (/30)
* Id. de VLAN para emparejamiento
* ASN para emparejamiento
* ASN de ExpressRoute: 12076
* Hash MD5 (opcional)

#### <a name="for-microsoft-peering"></a>En el caso del emparejamiento de Microsoft

Habilite esta opción para acceder a servicios en línea de Microsoft, como Office 365. Además, se puede acceder a todos los servicios de PaaS de Azure a través del emparejamiento de Microsoft. Debe asegurarse de que usa un proxy/borde independiente para conectarse a Microsoft, distinto del que usa para Internet. Si usa el mismo borde para ExpressRoute e Internet se producirá un enrutamiento asimétrico y causará interrupciones en la conectividad de la red.

* Subred de emparejamiento para la ruta de acceso 1 (/30): debe ser una dirección IP pública
* Subred de emparejamiento para la ruta de acceso 2 (/30): debe ser una dirección IP pública
* Id. de VLAN para emparejamiento
* ASN para emparejamiento
* Prefijos anunciados: deben ser prefijos de direcciones IP públicas
* ASN de cliente (opcional si es diferente del ASN de emparejamiento)
* RIR/IRR para la validación de IP y ASN
* ASN de ExpressRoute: 12076
* Hash MD5 (opcional)

### <a name="6-start-using-the-expressroute-circuit"></a>6. Uso del circuito ExpressRoute

* Puede vincular las redes virtuales de Azure al circuito ExpressRoute para habilitar la conectividad del entorno local a la red virtual de Azure. Consulte el artículo [Vinculación de una red virtual a un circuito](expressroute-howto-linkvnet-arm.md) para obtener instrucciones. Dichas redes virtuales pueden estar en la misma suscripción de Azure que el circuito ExpressRoute, o bien pueden estar en una suscripción diferente.
* Conéctese a los servicios de Azure y a los servicios en la nube de Microsoft a través del emparejamiento de Microsoft.

##  <a name="expressroute-partner-circuit-provisioning-states"></a><a name="expressroute-circuit-provisioning-states"></a>Estados de aprovisionamiento de circuitos de los asociados de ExpressRoute

En la siguiente sección se describen los diferentes estados del circuito ExpressRoute para el modelo de conectividad de asociados de ExpressRoute.
Cada circuito de asociado de ExpressRoute tiene dos estados:

* **ServiceProviderProvisioningState** representa el estado del lado del proveedor de conectividad. Puede ser *No aprovisionado*, *Aprovisionando* o *Aprovisionado*. Para configurar el emparejamiento, el circuito ExpressRoute debe estar en un estado aprovisionado. **Este estado solo se aplica a los circuitos de asociados de ExpressRoute y no se muestra en las propiedades de un circuito ExpressRoute Direct**.

* **Status** representa el estado de aprovisionamiento de Microsoft. Esta propiedad se establece en Enabled al crear un circuito ExpressRoute.

### <a name="possible-states-of-an-expressroute-circuit"></a>Posibles estados de un circuito ExpressRoute

En esta sección se describen los posibles estados de un circuito ExpressRoute creado con el modelo de conectividad de asociados de ExpressRoute.

**En tiempo de creación**

El circuito ExpressRoute notificará los estados siguientes durante la creación de recursos.

```output
ServiceProviderProvisioningState : NotProvisioned
Status                           : Enabled
```

**Cuando el proveedor de conectividad está en el proceso de aprovisionamiento del circuito**

El circuito ExpressRoute notificará los estados siguientes mientras el proveedor de conectividad trabaja para aprovisionar el circuito.

```output
ServiceProviderProvisioningState : Provisioning
Status                           : Enabled
```

**Cuando el proveedor de conectividad ha completado el proceso de aprovisionamiento**

El circuito ExpressRoute notificará los estados siguientes una vez que el proveedor de conectividad haya aprovisionado correctamente el circuito.

```output
ServiceProviderProvisioningState : Provisioned
Status                           : Enabled
```

**Cuando el proveedor de conectividad está desaprovisionando el circuito**

Si es necesario cancelar el aprovisionamiento del circuito ExpressRoute, el circuito notificará los estados siguientes una vez que el proveedor de servicios haya completado el proceso de desaprovisionamiento.

```output
ServiceProviderProvisioningState : NotProvisioned
Status                           : Enabled
```

Si es necesario, puede volver a habilitarlo, o bien ejecutar los cmdlets de PowerShell para eliminar el circuito.  

> [!IMPORTANT]
> No se puede eliminar un circuito cuando el valor de ServiceProviderProvisioningState sea Aprovisionamiento o Aprovisionado. El proveedor de conectividad debe desaprovisionar el circuito antes de que se pueda eliminar. Microsoft seguirá facturando el circuito hasta que se elimine el recurso del circuito ExpressRoute en Azure.
> 

## <a name="routing-session-configuration-state"></a>Estado de configuración de sesión de enrutamiento

El estado de aprovisionamiento de BGP notifica si la sesión BGP se ha habilitado en el perímetro de Microsoft. El estado debe estar habilitado para usar el emparejamiento privado o de Microsoft.

Es importante comprobar el estado de sesión de BGP, especialmente para la configuración entre pares de Microsoft. Además del estado de aprovisionamiento de BGP, hay otro estado denominado *estado de prefijos públicos anunciados*. El estado de los prefijos públicos anunciados debe estar en *configurado*, tanto para que la sesión BGP esté activa como para que el enrutamiento funcione de un extremo a otro. 

Si el estado de los prefijos públicos anunciados se establece en el estado de *validación necesaria* , la sesión BGP no se habilita, ya que los prefijos anunciados no coincidían con el número AS en ninguno de los registros de enrutamiento.

> [!IMPORTANT]
> Si el estado de los prefijos públicos anunciados es *validación manual*, tendrá que abrir una incidencia de soporte técnico en el [servicio de soporte técnico de Microsoft](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) y aportar una evidencia de que posee la dirección IP anunciada, junto con el número del sistema autónomo asociado.
> 
> 

## <a name="next-steps"></a>Pasos siguientes

* Configure su conexión ExpressRoute.
  
  * [Creación de un circuito ExpressRoute](expressroute-howto-circuit-arm.md)
  * [Configuración del enrutamiento](expressroute-howto-routing-arm.md)
  * [Vinculación de redes virtuales a circuitos ExpressRoute](expressroute-howto-linkvnet-arm.md)