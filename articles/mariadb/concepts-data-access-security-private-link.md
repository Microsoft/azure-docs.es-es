---
title: 'Private Link: Azure Database for MariaDB'
description: Obtenga información sobre cómo funciona Private Link para Azure Database for MariaDB.
author: kummanish
ms.author: manishku
ms.service: mariadb
ms.topic: conceptual
ms.date: 03/10/2020
ms.openlocfilehash: 20add4859b272b6d121666cde9c56296119d41e4
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836537"
---
# <a name="private-link-for-azure-database-for-mariadb"></a>Private Link para Azure Database for MariaDB

Private Link le permite crear puntos de conexión privados para Azure Database for MariaDB y, por tanto, incorpora los servicios de Azure en su red privada virtual (VNet). El punto de conexión privado expone una dirección IP privada que puede usar para conectarse a su servidor de base de datos de Azure Database for MariaDB como cualquier otro recurso de la red virtual.

Para una lista de los servicios PaaS que admiten la funcionalidad Private Link, consulte la página de [documentación](https://docs.microsoft.com/azure/private-link/index) de Private Link. Un punto de conexión privado es una dirección IP privada dentro de una [red virtual](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview) y una subred específicas.

> [!NOTE]
> La característica de vínculo privado solo está disponible para servidores de Azure Database for MariaDB en los planes de tarifa De uso general u Optimizado para memoria. Asegúrese de que el servidor de bases de datos esté incluido en uno de estos planes de tarifa.

## <a name="data-exfiltration-prevention"></a>Prevención de la filtración de datos

La filtración de datos en Azure Database for MariaDB tiene lugar cuando un usuario autorizado, como un administrador de base de datos, puede extraer datos de un sistema y moverlos a otra ubicación o sistema que esté fuera de la organización. Por ejemplo, el usuario mueve los datos a una cuenta de almacenamiento propiedad de un tercero.

Considere un escenario con un usuario que ejecuta MariaDB Workbench dentro de una máquina virtual de Azure que se conecta a una instancia de Azure Database for MariaDB. Esta instancia de MariaDB se encuentra en el centro de datos de Oeste de EE. UU. En el ejemplo siguiente se muestra cómo limitar el acceso con puntos de conexión públicos en la instancia de Azure Database for MariaDB mediante controles de acceso a la red.

* Deshabilite todo el tráfico de los servicios de Azure a la instancia de Azure Database for MariaDB mediante el punto de conexión público. Para ello, establezca la opción Allow Azure Services (Permitir servicios de Azure) en Desactivada. Asegúrese de que ningún intervalo o dirección IP tenga permitido el acceso al servidor a través de las [reglas de firewall](https://docs.microsoft.com/azure/mariadb/concepts-firewall-rules) o los [puntos de conexión de servicio de red virtual](https://docs.microsoft.com/azure/mariadb/concepts-data-access-security-vnet).

* Permita solo el tráfico a Azure Database for MariaDB mediante la dirección IP privada de la máquina virtual. Para más información, consulte los artículos acerca del [punto de conexión de servicio](concepts-data-access-security-vnet.md) y de las [reglas de firewall de la red virtual](howto-manage-vnet-portal.md).

* En la máquina virtual de Azure, restrinja el ámbito de la conexión saliente mediante el uso de grupos de seguridad de red (NSG) y etiquetas de servicio, como se indica a continuación:

    * Especifique una regla de NSG para permitir el tráfico en la etiqueta de servicio SQL.WestUs (solo se permite la conexión a la instancia de Azure Database for MariaDB en Oeste de EE. UU.).
    * Especifique una regla de NSG (con una prioridad mayor) para denegar el tráfico a la etiqueta de servicio SQL (se deniegan las conexiones a una base de datos MariaDB en todas las regiones).</br></br>

Al final de esta configuración, la máquina virtual de Azure solo puede conectarse a la instancia de Azure Database for MariaDB de la región Oeste de EE. UU. Sin embargo, la conectividad no está restringida a una sola instancia de Azure Database for MariaDB. La máquina virtual también puede conectarse a todas las instancias de Azure Database for MariaDB de la región Oeste de EE. UU., incluidas las bases de datos que no forman parte de la suscripción. Aunque en el escenario anterior se ha reducido el ámbito de la filtración de datos a una región concreta, no se ha eliminado por completo.</br>

Con Private Link, ahora puede configurar controles de acceso a la red como grupos de seguridad de red a fin de restringir el acceso al punto de conexión privado. Los recursos PaaS de Azure individuales se asignan a puntos de conexión privados concretos. Un usuario interno malintencionado solo puede acceder al recurso PaaS asignado (por ejemplo, una instancia de Azure Database for MariaDB), pero no a los recursos restantes.

## <a name="on-premises-connectivity-over-private-peering"></a>Conectividad local a través del emparejamiento privado

Cuando se conecta al punto de conexión público desde máquinas locales, es necesario agregar su dirección IP al firewall basado en IP mediante una regla de firewall a nivel de servidor. Aunque este modelo funciona bien para permitir el acceso a equipos individuales para las cargas de trabajo de desarrollo o de prueba, es difícil de administrar en los entornos de producción.

Con Private Link, puede habilitar el acceso entre locales al punto de conexión privado mediante [ExpressRoute](https://azure.microsoft.com/services/expressroute/), el emparejamiento privado o el [túnel de VPN](https://docs.microsoft.com/azure/vpn-gateway/). Posteriormente, puede deshabilitar todo el acceso a través de un punto de conexión público y no usar el firewall basado en IP.

> [!NOTE]
> En algunos casos, Azure Database for MariaDB y la subred de red virtual se encuentran en distintas suscripciones. En estos casos debe garantizar las siguientes configuraciones:
> - Asegúrese de que ambas suscripciones tengan registrado el proveedor de recursos **Microsoft.DBforMariaDB**. Para más información, consulte [resource-manager-registration][resource-manager-portal].

## <a name="configure-private-link-for-azure-database-for-mariadb"></a>Configuración de Private Link para Azure Database for MariaDB

### <a name="creation-process"></a>Proceso de creación

Los puntos de conexión privados son necesarios para habilitar Private Link. Se puede realizar mediante cualquiera de las guías paso a paso que se indican a continuación.

* [Azure Portal](https://docs.microsoft.com/azure/mariadb/howto-configure-privatelink-portal)
* [CLI](https://docs.microsoft.com/azure/mariadb/howto-configure-privatelink-cli)

### <a name="approval-process"></a>Proceso de aprobación

Después de crear el punto de conexión privado (PE), el administrador puede administrar la conexión de punto de conexión privado (PEC) a la instancia de Azure Database for MariaDB. Esta separación de tareas entre el administrador de red y el administrador de base de datos es útil para la administración de la conectividad de Azure Database for MariaDB. 

* Vaya al recurso de servidor de Azure Database for MariaDB en Azure Portal. 
    * Seleccione las conexiones del punto de conexión privado en el panel izquierdo.
    * Muestra una lista de todas las conexiones del punto de conexión privado (PEC).
    * Punto de conexión privado (PE) correspondiente creado.

![Selección del portal del punto de conexión privado](media/concepts-data-access-and-security-private-link/select-private-link-portal.png)

* Seleccione una conexión del punto de conexión privado en la lista.

![Selección del punto de conexión privado pendiente de aprobación](media/concepts-data-access-and-security-private-link/select-private-link.png)

* El administrador de servidor de MariaDB puede optar por aprobar o rechazar un punto de conexión privado y, además, tiene la opción de agregar una respuesta con un texto breve.

![Selección del mensaje del punto de conexión privado](media/concepts-data-access-and-security-private-link/select-private-link-message.png)

* Después de la aprobación o el rechazo, la lista reflejará el estado apropiado, junto con el texto de respuesta.

![Selección del estado final del punto de conexión privado](media/concepts-data-access-and-security-private-link/show-private-link-approved-connection.png)

## <a name="use-cases-of-private-link-for-azure-database-for-mariadb"></a>Casos de uso de Private Link para Azure Database for MariaDB

Los clientes se pueden conectar al punto de conexión privado desde la misma red virtual, desde una red virtual emparejada de la misma región o a través de una conexión entre redes virtuales de distintas regiones. Además, los clientes pueden conectarse de forma local mediante ExpressRoute, emparejamiento privado o tunelización de VPN. A continuación, puede ver un diagrama simplificado que muestra los casos de uso habituales.

![Información general sobre la selección del punto de conexión privado](media/concepts-data-access-and-security-private-link/show-private-link-overview.png)

### <a name="connecting-from-an-azure-vm-in-peered-virtual-network-vnet"></a>Conexión desde una máquina virtual de Azure en una red virtual emparejada (VNet)
Configure el [Emparejamiento de VNET](https://docs.microsoft.com/azure/virtual-network/tutorial-connect-virtual-networks-powershell) para establecer la conectividad con Azure Database for MariaDB desde una máquina virtual de Azure en una red virtual emparejada.

### <a name="connecting-from-an-azure-vm-in-vnet-to-vnet-environment"></a>Conexión desde una máquina virtual de Azure en un entorno de red virtual a red virtual
Configure la [conexión de la puerta de enlace de VPN entre redes virtuales](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal) para establecer conectividad con una instancia de Azure Database for MariaDB desde una máquina virtual de Azure de otra región o suscripción.

### <a name="connecting-from-an-on-premises-environment-over-vpn"></a>Conexión desde un entorno local a través de VPN
Para establecer la conectividad desde un entorno local a Azure Database for MariaDB, elija e implemente una de las siguientes opciones:

* [Conexión de punto a sitio](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps)
* [Conexión VPN de sitio a sitio](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell)
* [Circuito de ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-howto-linkvnet-portal-resource-manager)

## <a name="private-link-combined-with-firewall-rules"></a>Combinación de Private Link con las reglas de firewall

Las situaciones y resultados que se muestran a continuación son posibles cuando se usa Private Link en combinación con las reglas de firewall:

* Si no configura ninguna regla de firewall, ningún tráfico podrá tener acceso de forma predeterminada a la instancia de Azure Database for MariaDB.

* Si configura el tráfico público o un punto de conexión de servicio y crea puntos de conexión privados, los distintos tipos de tráfico entrante estarán autorizados por el tipo de regla de firewall correspondiente.

* Si no configura ningún tráfico público ni punto de conexión de servicio y crea puntos de conexión privados, Azure Database for MariaDB solo será accesible a través de los puntos de conexión privados. Si no configura ningún tráfico público ni punto de conexión de servicio, después de que se rechacen o eliminen todos los puntos de conexión privados aprobados, ningún tráfico podrá tener acceso a la instancia de Azure Database for MariaDB.

## <a name="deny-public-access-for-azure-database-for-mariadb"></a>Denegación del acceso público para Azure Database for MariaDB

Si desea depender completamente de puntos de conexión privados para acceder a sus instancias de Azure Database for MariaDB, puede deshabilitar la configuración de todos los puntos de conexión públicos ([reglas de firewall](concepts-firewall-rules.md) y [puntos de conexión de servicio de red virtual](concepts-data-access-security-vnet.md)). Para ello, configure **Deny Public Network Access** (Denegar el acceso a la red pública) en el servidor de base de datos. 

Si esta opción está establecida en *SÍ*, solo se permiten las conexiones mediante puntos de conexión privados a su instancia de Azure Database for MariaDB. Si esta opción está establecida en *NO*, los clientes pueden conectarse a su instancia de Azure Database for MariaDB en función de la configuración del firewall o del servicio de red virtual. Además, una vez establecido el valor de acceso a la red privada, los clientes no pueden agregar ni actualizar las "reglas de firewall" ni la "regla de punto de conexión de servicio de red virtual" existentes.

> [!Note]
> Esta característica está disponible en todas las regiones de Azure donde Azure Database for PostgreSQL: servidor único admite los planes de tarifa de uso general y optimizados para memoria.
>
> Esta configuración no afecta a las configuraciones de SSL y TLS de su instancia de Azure Database for MariaDB.

Para obtener información sobre cómo establecer **Deny Public Network Access** (Denegar el acceso a la red pública) para su instancia de Azure Database for MariaDB desde Azure Portal, consulte [Cómo configurar la denegación del acceso a una red pública](howto-deny-public-network-access.md).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre las características de seguridad de Azure Database for MariaDB, consulte los siguientes artículos:

* Para configurar un firewall para Azure Database for MariaDB, consulte [Compatibilidad con firewalls](https://docs.microsoft.com/azure/mariadb/concepts-firewall-rules).

* Para aprender a configurar un punto de conexión de servicio de red virtual para su instancia de Azure Database for MariaDB, consulte [Configuración del acceso desde redes virtuales](https://docs.microsoft.com/azure/mariadb/concepts-data-access-security-vnet).

* Para obtener información general sobre la conectividad de Azure Database for MariaDB, consulte [Arquitectura de la conectividad en Azure Database for MariaDB](https://docs.microsoft.com/azure/MariaDB/concepts-connectivity-architecture).

<!-- Link references, to text, Within this same GitHub repo. -->
[resource-manager-portal]: ../azure-resource-manager/management/resource-providers-and-types.md