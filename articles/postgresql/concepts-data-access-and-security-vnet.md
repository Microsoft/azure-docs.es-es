---
title: 'Reglas de red virtual en Azure Database for PostgreSQL: servidor único'
description: 'Aprenda a usar los puntos de conexión de servicio de red virtual (vnet) para conectarse a Azure Database for PostgreSQL: servidor único.'
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 07/17/2020
ms.openlocfilehash: 544fabf9a32eaa7ef7457fc26ae9212b9fce9872
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87837217"
---
# <a name="use-virtual-network-service-endpoints-and-rules-for-azure-database-for-postgresql---single-server"></a>Uso de reglas y puntos de conexión de servicio de red virtual para Azure Database for PostgreSQL con un único servidor

Las *reglas de red virtual* son una característica de firewall que controla si el servidor de Azure Database for PostgreSQL acepta las comunicaciones que se envían desde subredes específicas en redes virtuales. En este artículo se explica por qué la característica de la regla de red virtual a veces es la mejor opción para permitir la comunicación de forma segura con el servidor de Azure Database for PostgreSQL.

Para crear una regla de red virtual, primero debe haber una [red virtual][vm-virtual-network-overview] (VNET) y un [punto de conexión de servicio de red virtual][vm-virtual-network-service-endpoints-overview-649d] para la regla a la que hacer referencia. En la imagen siguiente se muestra cómo funciona un punto de conexión de servicio de red virtual con Azure Database for PostgreSQL:

![Ejemplo de cómo funciona un punto de conexión de servicio de red virtual](media/concepts-data-access-and-security-vnet/vnet-concept.png)

> [!NOTE]
> Esta característica está disponible en todas las regiones de nube pública de Azure donde Azure Database for PostgreSQL se implementa para servidores de uso general y optimizados para memoria.
> En el caso de emparejamiento de VNET, si el tráfico fluye a través de una instancia de VNet Gateway común con puntos de conexión de servicio y se supone que lo hace al elemento del mismo nivel, cree una regla de ACL o red virtual para permitir que Microsoft Azure Virtual Machines en la red virtual de puerta de enlace acceda al servidor Azure Database for PostgreSQL.

También puede considerar la posibilidad de usar [Private Link](concepts-data-access-and-security-private-link.md) para las conexiones. Private Link proporciona una dirección IP privada en la red virtual para el servidor de Azure Database for PostgreSQL.

<a name="anch-terminology-and-description-82f"></a>
## <a name="terminology-and-description"></a>Terminología y descripción

**Red virtual:** puede tener redes virtuales asociadas a la suscripción de Azure.

**Subred:** una red virtual contiene **subredes**. Cualquier máquina virtual (VM) de Azure que tenga se asignará a las subredes. Una subred puede contener varias máquinas virtuales u otros nodos de proceso. Los nodos de proceso que se encuentran fuera de la red virtual no pueden tener acceso a su red virtual a menos que configure la seguridad para que permita el acceso.

**Punto de conexión de servicio de red virtual:** un [punto de conexión de servicio de red virtual][vm-virtual-network-service-endpoints-overview-649d] es una subred cuyos valores de propiedad incluyen uno o más nombres formales de tipo de servicio de Azure. En este artículo nos interesa el nombre de tipo de **Microsoft.Sql**, que hace referencia al servicio de Azure denominado SQL Database. Esta etiqueta de servicio también se aplica a los servicios Azure Database for PostgreSQL y MySQL. Es importante tener en cuenta que, al aplicar la etiqueta de servicio de **Microsoft.Sql** a un punto de conexión de servicio de red virtual, se configurará el tráfico de punto de conexión de servicio de todos los servidores de Azure SQL Database, Azure Database for PostgreSQL y Azure Database for MySQL de la subred. 

**Regla de red virtual:** una regla de red virtual para el servidor de Azure Database for PostgreSQL es una subred que se muestra en la lista de control de acceso (ACL) del servidor de Azure Database for PostgreSQL. Para estar en la ACL de su servidor de Azure Database for PostgreSQL, la subred debe contener el nombre de tipo **Microsoft.Sql**.

Una regla de red virtual indica a su servidor de Azure Database for PostgreSQL que acepte las comunicaciones procedentes de todos los nodos que están en la subred.

<a name="anch-details-about-vnet-rules-38q"></a>

## <a name="benefits-of-a-virtual-network-rule"></a>Ventajas de una regla de red virtual

Hasta que no tome medidas, las máquinas virtuales de sus subredes no pueden comunicarse con el servidor de Azure Database for PostgreSQL. Una acción que permite establecer la comunicación es la creación de una regla de red virtual. El motivo de la elección del enfoque de la regla de red virtual requiere un análisis de comparación y contraste relacionado con las opciones de seguridad competentes que ofrece el firewall.

### <a name="a-allow-access-to-azure-services"></a>A. Permitir el acceso a servicios de Azure

El panel de Seguridad de conexión tiene el botón **Activado/Desactivado** con la etiqueta **Permitir el acceso a servicios de Azure**. La configuración **Activado** permite las comunicaciones desde todas las direcciones IP de Azure y todas las subredes de Azure. Es posible que estas IP o subredes de Azure no le pertenezcan. La configuración **Activado** es probablemente más abierta de lo que le gustaría que fuera su instancia de Azure Database for PostgreSQL Database. La característica de la regla de red virtual ofrece un control mucho más granular.

### <a name="b-ip-rules"></a>B. Reglas IP

El firewall de Azure Database for PostgreSQL le permite especificar los intervalos de direcciones IP desde los que se aceptan las comunicaciones en Azure Database for PostgreSQL Database. Este enfoque es preciso para las direcciones IP estables que están fuera de la red privada de Azure. Sin embargo, muchos nodos de dentro de la red privada de Azure se configuran con direcciones IP *dinámicas*. Es posible que las direcciones IP dinámicas cambien, por ejemplo, cuando se reinicia la máquina virtual. Sería una locura especificar una dirección IP dinámica en una regla de firewall, en un entorno de producción.

Para recuperar la opción de IP, puede obtener una dirección IP *estática* para la máquina virtual. Para más información, consulte [Configuración de direcciones IP privadas para una máquina virtual mediante Azure Portal][vm-configure-private-ip-addresses-for-a-virtual-machine-using-the-azure-portal-321w].

Sin embargo, el enfoque de IP estática puede resultar difícil de administrar, y es costoso si se realiza a escala. Las reglas de red virtual son más sencillas de establecer y administrar.


<a name="anch-details-about-vnet-rules-38q"></a>

## <a name="details-about-virtual-network-rules"></a>Detalles sobre las reglas de red virtual

En esta sección se describen varios detalles acerca de las reglas de red virtual.

### <a name="only-one-geographic-region"></a>Solo una región geográfica

Cada punto de conexión de servicio de red virtual se aplica solo a una región de Azure. El punto de conexión no permite que otras regiones acepten la comunicación de la subred.

Cualquier regla de red virtual se limita a la región a la que se aplica su punto de conexión subyacente.

### <a name="server-level-not-database-level"></a>Nivel de servidor y no de base de datos

Cada regla de red virtual se aplica a todo el servidor de Azure Database for PostgreSQL y no solo a una base de datos determinada del servidor. En otras palabras, la regla de red virtual se aplica en el nivel de servidor y no en el nivel de base de datos.

#### <a name="security-administration-roles"></a>Roles de administración de la seguridad

Existe una separación de los roles de seguridad en la administración de puntos de conexión de servicio de red virtual. Se requiere una acción de cada uno de los roles siguientes:

- **Administrador de red:** &nbsp; se activa el punto de conexión.
- **Administrador de base de datos:** &nbsp; se actualiza la lista de control de acceso (ACL) que se va a agregar a la subred proporcionada en el servidor de Azure Database for PostgreSQL.

*Alternativa de RBAC:*

Los roles de administrador de red y de base de datos tienen más funcionalidades de las que se necesitan para administrar las reglas de red virtual. Solo se necesita un subconjunto de sus capacidades.

Si lo desea, puede optar por usar el [control de acceso basado en rol de Azure (Azure RBAC)][rbac-what-is-813s] en Azure para crear un rol personalizado único que tenga solo el subconjunto necesario de funcionalidades. Se podría usar el rol personalizado en lugar del administrador de red o el administrador de la base de datos. El área expuesta de la exposición de seguridad es inferior si agrega un usuario a un rol personalizado, en lugar de agregar el usuario a los otros dos roles de administrador principales.

> [!NOTE]
> En algunos casos, Azure Database for PostgreSQL y la subred de red virtual se encuentran en distintas suscripciones. En estos casos debe garantizar las siguientes configuraciones:
> - Ambas suscripciones deben estar en el mismo inquilino de Azure Active Directory.
> - El usuario tiene los permisos necesarios para iniciar operaciones como habilitar los puntos de conexión de servicio y agregar una subred de red virtual al servidor especificado.
> - Asegúrese de que ambas suscripciones tengan los proveedores de recursos **Microsoft.Sql** y **Microsoft.DBforPostgreSQL** registrados. Para más información, consulte [resource-manager-registration][resource-manager-portal].

## <a name="limitations"></a>Limitaciones

Para Azure Database for PostgreSQL, la característica de las reglas de red virtual tiene las siguientes limitaciones:

- Una aplicación web se puede asignar a una dirección IP privada en una red virtual o subred. Incluso si los puntos de conexión de servicio están ACTIVADOS desde la red virtual o subred proporcionada, las conexiones de la aplicación web con el servidor tendrán un origen IP público de Azure, no un origen de red virtual o subred. Para habilitar la conectividad desde una aplicación web con un servidor que tenga reglas de firewall de red virtual, debe permitir que los servicios de Azure accedan al servidor en el servidor.

- En el firewall de su instancia de Azure Database for PostgreSQL, cada regla de red virtual hace referencia a una subred. Todas estas subredes a las que se hace referencia deben estar hospedadas en la misma región geográfica que hospeda Azure Database for PostgreSQL.

- Cada servidor de Azure Database for PostgreSQL puede tener hasta 128 entradas de ACL para cualquier red virtual proporcionada.

- Las reglas de red virtual solo se aplican a las redes virtuales de Azure Resource Manager, y no a las redes del [modelo de implementación clásica][arm-deployment-model-568f].

- Al activar los puntos de conexión de servicio de red virtual en Azure Database for PostgreSQL con la etiqueta de servicio **Microsoft.Sql** también se habilitan los puntos de conexión de todos los servicios de Azure Database: Azure Database for MySQL, Azure Database for PostgreSQL, Azure SQL Database y Azure SQL Data Warehouse.

- La compatibilidad con puntos de conexión de servicio de red virtual solo existe para servidores de uso general y optimizados para memoria.

- Si **Microsoft.Sql** está habilitado en una subred, indica que solo desea usar reglas de red virtual para conectarse. Las [reglas de firewall que no sean de red virtual](concepts-firewall-rules.md) no funcionarán en esa subred.

- En el firewall, los intervalos de direcciones IP se aplican a los siguientes elementos de red, pero no las reglas de red virtual:
    - [Red privada virtual (VPN) de sitio a sitio (S2S)][vpn-gateway-indexmd-608y]
    - Entorno local a través de [ExpressRoute][expressroute-indexmd-744v]

## <a name="expressroute"></a>ExpressRoute

Si la red está conectada a la red de Azure mediante el uso de [ExpressRoute][expressroute-indexmd-744v], cada circuito está configurado con dos direcciones IP públicas en Microsoft Edge. Las dos direcciones IP se utilizan para conectarse a servicios Microsoft, como a Azure Storage, mediante el uso del emparejamiento público de Azure.

Para permitir la comunicación desde el circuito a Azure Database for PostgreSQL, debe crear reglas de red IP para direcciones IP públicas de los circuitos. Para encontrar las direcciones IP públicas del circuito de ExpressRoute, abra un vale de soporte con ExpressRoute mediante Azure Portal.

## <a name="adding-a-vnet-firewall-rule-to-your-server-without-turning-on-vnet-service-endpoints"></a>Adición de una regla de firewall de red virtual al servidor sin activar los puntos de conexión de servicio de la red virtual

Si solo establece una regla de firewall de red virtual, no tendrá el servidor protegido en la red virtual. Por lo tanto, también debe **activar** los puntos de conexión de servicio de red virtual para que la seguridad surta efecto. Al **activar** los puntos de conexión de servicio, la subred de la red virtual experimenta cierto tiempo de inactividad hasta que estos puntos se **activan** **totalmente**. Esto sucede especialmente en redes virtuales de gran tamaño. Puede usar la marca **IgnoreMissingServiceEndpoint** para reducir o eliminar el tiempo de inactividad durante la activación.

Puede establecer la marca **IgnoreMissingServiceEndpoint** mediante la CLI de Azure o el portal.

## <a name="related-articles"></a>Artículos relacionados
- [Redes virtuales de Azure][vm-virtual-network-overview]
- [Punto de conexión de servicio de red virtual de Azure][vm-virtual-network-service-endpoints-overview-649d]

## <a name="next-steps"></a>Pasos siguientes
Para ver artículos sobre cómo crear reglas de red virtual, consulte lo siguiente:
- [Creación y administración de reglas y puntos de conexión de servicio de red virtual de Azure Database for PostgreSQL mediante Azure Portal](howto-manage-vnet-using-portal.md)
- [Create and manage Azure Database for PostgreSQL VNet rules using Azure CLI](howto-manage-vnet-using-cli.md) (Creación y administración de reglas de red virtual de Azure Database for PostgreSQL mediante la CLI de Azure)


<!-- Link references, to text, Within this same GitHub repo. -->
[arm-deployment-model-568f]: ../azure-resource-manager/management/deployment-models.md

[vm-virtual-network-overview]: ../virtual-network/virtual-networks-overview.md

[vm-virtual-network-service-endpoints-overview-649d]: ../virtual-network/virtual-network-service-endpoints-overview.md

[vm-configure-private-ip-addresses-for-a-virtual-machine-using-the-azure-portal-321w]: ../virtual-network/virtual-networks-static-private-ip-arm-pportal.md

[rbac-what-is-813s]: ../active-directory/role-based-access-control-what-is.md

[vpn-gateway-indexmd-608y]: ../vpn-gateway/index.yml

[expressroute-indexmd-744v]: ../expressroute/index.yml

[resource-manager-portal]: ../azure-resource-manager/management/resource-providers-and-types.md
