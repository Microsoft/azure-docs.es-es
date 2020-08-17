---
title: Punto de conexión de servicio de red virtual de Azure
titlesuffix: Azure Virtual Network
description: Aprenda a habilitar el acceso directo a los recursos de Azure desde una red virtual con puntos de conexión de servicio.
services: virtual-network
documentationcenter: na
author: sumeetmittal
ms.service: virtual-network
ms.devlang: NA
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/08/2019
ms.author: sumi
ms.custom: ''
ms.openlocfilehash: 6935ea505643f253daf4c0b7d818137f52583682
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132830"
---
# <a name="virtual-network-service-endpoints"></a>Puntos de conexión de servicio de red virtual

El punto de conexión de servicio de red virtual (VNet) proporciona conectividad directa y segura con los servicios de Azure por medio de una ruta optimizada a través de la red troncal de Azure. Los puntos de conexión permiten proteger los recursos de servicio de Azure críticos únicamente para las redes virtuales. Los puntos de conexión de servicio permiten a las direcciones IP privadas de la red virtual alcanzar el punto de conexión de un servicio de Azure sin necesidad de una dirección IP pública en la red virtual.

Esta característica está disponible para las siguientes regiones y servicios de Azure. El recurso *Microsoft.\** está entre paréntesis. Habilítelo desde la subred mientras configura los puntos de conexión del servicio:

**Disponibilidad general**

- **[Azure Storage](../storage/common/storage-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json#grant-access-from-a-virtual-network)** (*Microsoft.Storage*): disponibilidad general en todas las regiones de Azure.
- **[Azure SQL Database](../azure-sql/database/vnet-service-endpoint-rule-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)** (*Microsoft.Sql*): disponibilidad general en todas las regiones de Azure.
- **[Azure SQL Data Warehouse](../azure-sql/database/vnet-service-endpoint-rule-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)** (*Microsoft.Sql*): disponibilidad general en todas las regiones de Azure.
- **[Servidor de Azure Database for PostgreSQL](../postgresql/howto-manage-vnet-using-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)** (*Microsoft.Sql*): disponibilidad general en las regiones de Azure en las que el servicio de base de datos esté disponible.
- **[Servidor de Azure Database for MySQL](../mysql/howto-manage-vnet-using-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)** (*Microsoft.Sql*): disponibilidad general en las regiones de Azure en las que el servicio de base de datos esté disponible.
- **[Azure Database for MariaDB](https://docs.microsoft.com/azure/mariadb/concepts-data-access-security-vnet)** (*Microsoft.Sql*): disponibilidad general en las regiones de Azure en las que el servicio de base de datos esté disponible.
- **[Azure Cosmos DB](../cosmos-db/vnet-service-endpoint.md?toc=%2fazure%2fvirtual-network%2ftoc.json)** (*Microsoft.AzureCosmosDB*): disponibilidad general en todas las regiones de Azure.
- **[Azure Key Vault](../key-vault/general/overview-vnet-service-endpoints.md)** (*Microsoft.KeyVault*): disponibilidad general en todas las regiones de Azure.
- **[Azure Service Bus](../service-bus-messaging/service-bus-service-endpoints.md?toc=%2fazure%2fvirtual-network%2ftoc.json)** (*Microsoft.ServiceBus*): disponibilidad general en todas las regiones de Azure.
- **[Azure Event Hubs](../event-hubs/event-hubs-service-endpoints.md?toc=%2fazure%2fvirtual-network%2ftoc.json)** (*Microsoft.EventHub*): disponibilidad general en todas las regiones de Azure.
- **[Azure Data Lake Store Gen 1](../data-lake-store/data-lake-store-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json)** (*Microsoft.AzureActiveDirectory*): Disponible con carácter general en todas las regiones de Azure donde ADLS Gen1 está disponible.
- **[Azure App Service](https://docs.microsoft.com/azure/app-service/app-service-ip-restrictions)** (*Microsoft.Web*): disponible con carácter general en todas las regiones de Azure en que App Service esté disponible.

**Versión preliminar pública**

- **[Azure Container Registry](../container-registry/container-registry-vnet.md)** (*Microsoft.ContainerRegistry*): Hay una versión preliminar disponible en algunas de las regiones de Azure en que está disponible Azure Container Registry.

Para conocer las notificaciones más actualizadas sobre, consulte la página [Actualizaciones de Azure Virtual Network](https://azure.microsoft.com/updates/?product=virtual-network).

## <a name="key-benefits"></a>Ventajas principales

Los puntos de conexión de servicio proporcionan las siguientes ventajas:

- **Seguridad mejorada para los recursos de servicio de Azure**: Los espacios de direcciones privadas de una red virtual pueden superponerse. No se pueden usar espacios superpuestos para identificar de forma exclusiva el tráfico que parte de una red virtual. Los puntos de conexión de servicio ofrecen la capacidad de proteger los recursos de los servicios de Azure en la red virtual mediante la extensión de la identidad de la red virtual al servicio. Una vez que habilita puntos de conexión de servicio en su red virtual, puede agregar una regla de red virtual para proteger los recursos de los servicios de Azure en la red virtual. La adición de la regla proporciona una mayor seguridad, ya que elimina totalmente el acceso público de Internet a los recursos y solo permite el tráfico que procede de la red virtual.
- **Enrutamiento óptimo para el tráfico del servicio de Azure desde la red virtual**: hoy, las rutas de la red virtual que fuerzan el tráfico de Internet a las aplicaciones virtuales o locales también fuerzan al tráfico del servicio de Azure a que realice la misma ruta que el tráfico de Internet. Los puntos de conexión de servicio proporcionan un enrutamiento óptimo al tráfico de Azure. 

  Los puntos de conexión siempre toman el tráfico del servicio directamente de la red virtual al servicio en la red troncal de Microsoft Azure. Mantener el tráfico en la red troncal de Azure permite seguir auditando y supervisando el tráfico saliente de Internet desde las redes virtuales, a través de la tunelización forzada, sin que afecte al tráfico del servicio. Para más información sobre las rutas definidas por el usuario y la tunelización forzada, consulte [Enrutamiento del tráfico de redes virtuales de Azure](virtual-networks-udr-overview.md).
- **Fácil de configurar con menos sobrecarga de administración**: ya no necesita direcciones IP públicas y reservadas en sus redes virtuales para proteger los recursos de Azure a través de una dirección IP del firewall. No hay ningún dispositivo NAT (traducción de direcciones de red) o de puerta de enlace necesario para configurar los puntos de conexión de servicio. Estos se pueden configurar con un simple clic en una subred. El mantenimiento de los puntos de conexión no supone una sobrecarga adicional.

## <a name="limitations"></a>Limitaciones

- Esta característica solo está disponible en las redes virtuales implementadas con el modelo de implementación de Azure Resource Manager.
- Los puntos de conexión están habilitados en subredes configuradas en redes virtuales de Azure. Los puntos de conexión no se pueden usar para el tráfico que fluye entre las instalaciones y los servicios de Azure. Para más información, consulte [Protección del acceso de servicio de Azure desde el entorno local](#secure-azure-services-to-virtual-networks)
- Para Azure SQL, un punto de conexión de servicio solo se aplica al tráfico del servicio de Azure dentro de la región de una red virtual. En el caso de Azure Storage, los puntos de conexión también se amplían para incluir las regiones emparejadas en las que se implementa la red virtual para admitir el tráfico del almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) y el del almacenamiento con redundancia geográfica (GRS). Para más información, consulte [Regiones emparejadas de Azure](../best-practices-availability-paired-regions.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-paired-regions).
- En el caso de Azure Data Lake Storage (ADLS) Gen 1, la funcionalidad de integración de red virtual solo está disponible para las redes sociales que se encuentran dentro de la misma región. Tenga también en cuenta que la integración de la red virtual en ADLS Gen1 usa la seguridad del punto de conexión de servicio de red virtual entre la red virtual y Azure Active Directory (Azure AD) para generar notificaciones de seguridad adicionales en el token de acceso. Estas notificaciones se usan entonces para autenticar la red virtual en la cuenta de Data Lake Storage Gen1 y permitir el acceso. La etiqueta *Microsoft.AzureActiveDirectory* que aparece debajo de los servicios que admiten puntos de conexión de servicios se usa solo para admitir puntos de conexión de servicios en ADLS Gen 1. Azure AD no admite puntos de conexión de servicio de forma nativa. Para más información acerca de la integración de redes virtuales en Azure Data Lake Store Gen 1, consulte [Seguridad de red en Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="secure-azure-services-to-virtual-networks"></a>Protección de servicios de Azure en redes virtuales

- Un punto de conexión de servicio de red virtual proporciona la identidad de la red virtual al servicio de Azure. Una vez que habilita puntos de conexión de servicio en su red virtual, puede agregar una regla de red virtual para proteger los recursos de los servicios de Azure en la red virtual.
- Actualmente, el tráfico del servicio de Azure desde una red virtual usa direcciones IP públicas como direcciones IP de origen. Con los puntos de conexión de servicio, el tráfico del servicio cambia para usar direcciones privadas de red virtual como la direcciones IP de origen al acceder al servicio de Azure desde una red virtual. Este modificador permite acceder a los servicios sin necesidad de direcciones IP públicas y reservadas utilizadas en los firewalls IP.

   >[!NOTE]
   > Con los puntos de conexión de servicio, las direcciones IP de origen de las máquinas virtuales en la subred para el tráfico de servicio pasan de utilizar direcciones IPv4 públicas a utilizar direcciones IPv4 privadas. Las reglas existentes de firewall de servicio de Azure que utilizan direcciones IP públicas Azure dejarán de funcionar con este cambio. Asegúrese de que las reglas de firewall del servicio de Azure permiten este cambio antes de configurar los puntos de conexión de servicio. También es posible que experimente una interrupción temporal del tráfico de servicio desde esta subred mientras configura los puntos de conexión de servicio. 
 
## <a name="secure-azure-service-access-from-on-premises"></a>Protección del acceso de servicio de Azure desde el entorno local

  De forma predeterminada, a los recursos de los servicios de Azure protegidos en las redes virtuales no se puede acceder desde redes locales. Si quiere permitir el tráfico desde el entorno local, también debe permitir las direcciones IP públicas (normalmente NAT) desde el entorno local o ExpressRoute. Estas direcciones IP se pueden agregar mediante la configuración del firewall de IP para los recursos de los servicios de Azure.

  ExpressRoute: Si usa [ExpressRoute](../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json) para el emparejamiento público o el emparejamiento de Microsoft desde un entorno local, tendrá que identificar las direcciones IP de NAT que va a usar. En el caso del emparejamiento público, cada circuito ExpressRoute usa de forma predeterminada dos direcciones IP de NAT que se aplican al tráfico del servicio de Azure cuando entra en la red troncal de Microsoft Azure. En el caso del emparejamiento de Microsoft, las direcciones IP de NAT que se utilizan las proporcionan el cliente o el proveedor de servicios. Para permitir el acceso a los recursos de servicio, tiene que permitir estas direcciones IP públicas en la configuración del firewall de IP de recursos. Para encontrar las direcciones de IP de circuito de ExpressRoute de los pares públicos, [abra una incidencia de soporte técnico con ExpressRoute](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) en Azure Portal. Para más información sobre NAT tanto para el emparejamiento de Microsoft como para el emparejamiento público de ExpresRoute, consulte [Requisitos de NAT de ExpressRoute](../expressroute/expressroute-nat.md?toc=%2fazure%2fvirtual-network%2ftoc.json#nat-requirements-for-azure-public-peering).

![Protección de servicios de Azure para las redes virtuales](./media/virtual-network-service-endpoints-overview/VNet_Service_Endpoints_Overview.png)

### <a name="configuration"></a>Configuración

- Configure los puntos de conexión de servicio en una subred de una red virtual. Los puntos de conexión funcionan con cualquier tipo de instancias de proceso que se ejecute en esa subred.
- Puede configurar varios puntos de conexión de servicio para todos los servicios de Azure admitidos (por ejemplo, Azure Storage o Azure SQL Database) en una subred.
- Para Azure SQL Database, las redes virtuales deben estar en la misma región que el recurso de servicio de Azure. Si se usan cuentas de Azure Storage para GRS y RA-GRS, la cuenta principal debe encontrarse en la misma región que la red virtual. En el caso de los restantes servicios, los recursos de servicio de Azure se pueden proteger en las redes virtuales de cualquier región. 
- La red virtual donde se ha configurado el punto de conexión puede estar en la misma suscripción o en otra distinta del recurso de servicio de Azure. Para más información sobre los permisos necesarios para configurar los puntos de conexión y proteger los servicios de Azure, consulte [Aprovisionamiento](#provisioning).
- Para los servicios compatibles, puede proteger los nuevos recursos o los recursos existentes a las redes virtuales con puntos de conexión de servicio.

### <a name="considerations"></a>Consideraciones

- Después de habilitar un punto de conexión de servicio, las direcciones IP de origen pasan de utilizar las direcciones IPv4 públicas a usar su dirección IPv4 privada, cuando se comunican con el servicio desde dicha subred. Las conexiones TCP abiertas existentes en el servicio se cierran durante este cambio. Asegúrese de que no se esté ejecutando ninguna tarea crítica al habilitar o deshabilitar un punto de conexión de servicio en un servicio para una subred. Además, asegúrese de que las aplicaciones pueden conectarse automáticamente a los servicios de Azure después del cambio de dirección IP.

  El cambio de dirección IP solo afecta el tráfico del servicio desde la red virtual. No afecta a ningún otro tráfico entrante o saliente de las direcciones IPv4 públicas asignadas a las máquinas virtuales. Para los servicios de Azure, si tiene reglas de firewall existentes que usan direcciones IP públicas de Azure, estas reglas dejan de funcionar con el cambio a las direcciones privadas de red virtual.
- Con los puntos de conexión de servicio, las entradas DNS para los servicios de Azure permanecen tal cual en la actualidad y siguen resolviendo las direcciones IP públicas asignadas al servicio de Azure.

- Grupos de seguridad de red (NSG) con puntos de conexión de servicio:
  - De forma predeterminada, los grupos de seguridad de red no solo permiten el tráfico de Internet saliente, sino también el tráfico de su red virtual a los servicios de Azure. Este tráfico sigue funcionando con puntos de conexión de servicio tal cual. 
  - Si desea denegar todo el tráfico de Internet saliente y permitir únicamente el tráfico a servicios de Azure concretos, puede hacerlo mediante las [etiquetas de servicio](security-overview.md#service-tags) de sus grupos de seguridad de red. Puede especificar los servicios de Azure compatibles como destino en las reglas de grupos de seguridad de red y Azure también proporciona el mantenimiento de las direcciones IP subyacentes en cada etiqueta. Para más información, consulte las [etiquetas de servicio de Azure para grupos de seguridad de red](security-overview.md#service-tags). 

### <a name="scenarios"></a>Escenarios

- **Redes virtuales emparejadas, conectadas o múltiples**: para proteger los servicios de Azure de varias subredes dentro de una red virtual o entre varias redes virtuales, puede habilitar los puntos de conexión de servicio en cada una de las subredes de manera independiente y proteger los recursos del servicio de Azure a todas las subredes.
- **Filtrado de tráfico saliente desde una red virtual a los servicios de Azure**: Si desea inspeccionar o filtrar el tráfico enviado a un servicio de Azure desde una red virtual, puede implementar una aplicación virtual de red dentro de la red virtual. Después, puede aplicar los puntos de conexión de servicio a la subred donde se implementa la aplicación virtual de red y se protegen los recursos de servicio de Azure solo para esta subred. Este escenario puede resultar útil si desea usar el filtrado de la aplicación virtual de red para restringir el acceso de los servicios de Azure desde la red virtual solo a recursos concretos de Azure. Para más información, consulte el artículo sobre la [salida con las aplicaciones de redes virtuales](/azure/architecture/reference-architectures/dmz/nva-ha).
- **Protección de recursos de Azure con los servicios implementados directamente en redes virtuales**: puede implementar directamente varios servicios de Azure en subredes concretas de una red virtual. Puede proteger los recursos de servicio de Azure para subredes de [servicio administrado](virtual-network-for-azure-services.md) mediante la configuración de un punto de conexión de servicio en la subred de servicio administrada.
- **Tráfico de disco procedente de una máquina virtual de Azure**: el tráfico de disco de máquina virtual, tanto en discos administrados como no administrados, no resulta afectado por los puntos de conexión de servicio que enrutan los cambios en Azure Storage. Este tráfico no solo incluye la E/S de disco, sino también el montaje y desmontaje. Puede limitar el acceso de REST a los blobs en páginas para determinadas redes mediante puntos de conexión de servicio y [reglas de red de Azure Storage](../storage/common/storage-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json). 

### <a name="logging-and-troubleshooting"></a>Registro y solución de problemas

Una vez que configure los puntos de conexión de servicio en un servicio concreto, asegúrese de que la ruta del punto de conexión de servicio está en vigor de la forma siguiente: 
 
- Valide la dirección IP de origen de todas las solicitudes de servicio en los diagnósticos del servicio. Todas las nuevas solicitudes con puntos de conexión de servicio muestran la dirección IP de origen de la solicitud como la dirección de red virtual privada, asignada al cliente que realiza la solicitud desde la red virtual. Sin el punto de conexión, la dirección es una dirección IP pública de Azure.
- Visualice las rutas eficaces en cualquier interfaz de red de una subred. La ruta al servicio:
  - Muestra una ruta predeterminada más específica a los intervalos de prefijos de la dirección de cada servicio
  - Tiene una clase nextHopType de *VirtualNetworkServiceEndpoint*
  - Indica que está en vigor una conexión más directa al servicio, en comparación con cualquier ruta de tunelización forzada

>[!NOTE]
> Las rutas del punto de conexión de servicio invalidan las rutas BGP o UDR para la coincidencia del prefijo de dirección de un servicio de Azure. Para más información, consulte el artículo en el que se indica [cómo se solucionan problemas con redes eficaces](diagnose-network-routing-problem.md).

## <a name="provisioning"></a>Aprovisionamiento

Un usuario con acceso de escritura a una red virtual puede configurar puntos de conexión de servicio en redes virtuales de forma independiente. Para proteger los recursos de los servicios de Azure en una red virtual, el usuario debe tener permiso en *Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/action* para las subredes agregadas. Los roles de administrador de servicios integrados incluyen este permiso de manera predeterminada. El permiso se puede modificar mediante la creación de roles personalizados.

Para obtener más información sobre los roles integrados, consulte [Roles integrados en Azure](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Para obtener más información sobre la asignación de permisos concretos a roles personalizados, consulte [Roles personalizados de Azure](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

Las redes virtuales y los recursos de servicio de Azure pueden encontrarse en la misma o en diferentes suscripciones. Si los recursos de servicio de Azure y de red virtual están en distintas suscripciones, los recursos deben estar en el mismo inquilino de Active Directory (AD). 

## <a name="pricing-and-limits"></a>Precios y límites

No se realiza ningún cargo adicional por el uso de puntos de conexión de servicio. El modelo de precios vigente para los servicios de Azure (Azure Storage, Azure SQL Database, etc.) se aplica tal cual.

No hay límite en el número total de puntos de conexión de servicio que puede tener una red virtual.

Para determinados servicios de Azure (por ejemplo, cuentas de Azure Storage), puede aplicar límites en el número de subredes que se usan para proteger el recurso. Para más información, consulte la documentación de varios servicios en la sección [Pasos siguientes](#next-steps).

## <a name="vnet-service-endpoint-policies"></a>Directivas de punto de conexión de servicio de una red virtual 

Las directivas de punto de conexión de servicio de una red virtual permiten filtrar el tráfico de la red virtual a los servicios de Azure. Este filtro solo permite determinados recursos de servicio de Azure a través de puntos de conexión de servicio. Las directivas de punto de conexión de servicio ofrecen un control de acceso pormenorizado para el tráfico de red virtual a los servicios de Azure. Para más información, consulte [Directivas de punto de conexión de servicio de red virtual](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoint-policies-overview).

## <a name="faqs"></a>Preguntas más frecuentes

Consulte las [preguntas más frecuentes acerca de los puntos de conexión de servicio de red virtual](https://docs.microsoft.com/azure/virtual-network/virtual-networks-faq#virtual-network-service-endpoints).

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de los puntos de conexión de servicio de red virtual](tutorial-restrict-network-access-to-resources.md)
- [Protección de una cuenta de Azure Storage para una red virtual](../storage/common/storage-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Protección de una instancia de Azure SQL Database para una red virtual](../azure-sql/database/vnet-service-endpoint-rule-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Protección de una instancia de Azure SQL Data Warehouse para una red virtual](../azure-sql/database/vnet-service-endpoint-rule-overview.md?toc=%2fazure%2fsql-data-warehouse%2ftoc.json)
- [Integración de servicios de Azure en redes virtuales](virtual-network-for-azure-services.md)
- [Directivas de puntos de conexión de servicio de red virtual](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoint-policies-overview)
- [Plantilla de Azure Resource Manager](https://azure.microsoft.com/resources/templates/201-vnet-2subnets-service-endpoints-storage-integration)
