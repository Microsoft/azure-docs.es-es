---
title: 'Tutorial: Lista de comprobación de red'
description: Requisitos previos de red y detalles sobre la conectividad de red y los puertos de red
ms.topic: tutorial
ms.date: 08/21/2020
ms.openlocfilehash: aba5d7767e420b3ade6238621487884e44fbb6e2
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88750417"
---
# <a name="networking-checklist-for-azure-vmware-solution"></a>Lista de comprobación de redes para Azure VMware Solution 

Azure VMware Solution ofrece un entorno de nube privada de VMware, que es accesible para usuarios y aplicaciones desde recursos o entornos locales y basados en Azure. La conectividad se proporciona mediante servicios de red como conexiones VPN y Azure ExpressRoute, y requerirá algunos intervalos de direcciones de red y puertos de firewall específicos para habilitar los servicios. En este artículo se proporciona la información necesaria para configurar correctamente las redes a fin de que funcionen con Azure VMware Solution.

En este tutorial, aprenderá lo siguiente:

> [!div class="checklist"]
> * Requisitos de conectividad de red
> * DHCP en Azure VMware Solution

## <a name="virtual--network-and-expressroute-circuit--considerations"></a>Consideraciones sobre la red virtual y el circuito ExpressRoute
Al crear una conexión desde una red virtual de la suscripción, el circuito ExpressRoute se establece por medio del emparejamiento, usa una clave de autorización y un identificador de emparejamiento que se solicita en Azure Portal. El emparejamiento es una conexión privada de uno a uno entre la nube privada y la red virtual.

> [!NOTE] 
> El circuito ExpressRoute no forma parte de una implementación de nube privada. El circuito de ExpressRoute local está fuera del ámbito de este documento. Si necesita una conectividad desde el entorno local a su nube privada, puede utilizar uno de los circuitos de ExpressRoute existentes o comprar uno en Azure Portal.

Al implementar una nube privada, se reciben direcciones IP de vCenter y NSX-T Manager. Para acceder a esas interfaces de administración, es necesario crear recursos adicionales en una red virtual de la suscripción. En los tutoriales puede encontrar los procedimientos para crear esos recursos y establecer el emparejamiento privado de ExpressRoute.

Las redes lógicas de nube privada incluyen NSX-T previamente aprovisionado. Se aprovisionan previamente de forma automática una puerta de enlace de nivel 0 y una de nivel 1. Puede crear un segmento y asociarlo a la puerta de enlace de nivel 1 existente o hacerlo a una nueva puerta de enlace de nivel 1 que puede definir. Los componentes de redes lógicas NSX-T proporcionan conectividad horizontal de derecha a izquierda entre cargas de trabajo y proporcionan conectividad vertical de arriba abajo a Internet y a los servicios de Azure.

## <a name="routing-and-subnet-considerations"></a>Consideraciones sobre el enrutamiento y la subred
La nube privada de AVS se conecta a la red virtual de Azure mediante una conexión de Azure ExpressRoute. Esta conexión de alto ancho de banda y baja latencia permite acceder a los servicios que se ejecutan en la suscripción de Azure desde el entorno de nube privada. El enrutamiento se basa en el Protocolo de puerta de enlace de borde (BGP), que se aprovisiona y habilita automáticamente de forma predeterminada para cada implementación de nube privada. 

Las nubes privadas AVS requieren un mínimo de un bloque de direcciones de red CIDR `/22` para las subredes, que se muestran a continuación. Esta red complementa las redes locales. El bloque de direcciones no debe superponerse a los bloques de direcciones usados en otras redes virtuales ubicadas en la suscripción y las redes locales. Dentro de este bloque de direcciones, la administración, el aprovisionamiento y las redes de vMotion se aprovisionan automáticamente.

Un ejemplo de bloque de direcciones de red CIDR `/22` es `10.10.0.0/22`

Las subredes:

| Uso de red             | Subnet | Ejemplo        |
| ------------------------- | ------ | -------------- |
| Administración de la nube privada  | `/24`  | `10.10.0.0/24` |
| Red vMotion           | `/24`  | `10.10.1.0/24` |
| Cargas de trabajo de máquinas virtuales              | `/24`  | `10.10.2.0/24` |
| Emparejamiento de ExpressRoute      | `/24`  | `10.10.3.8/30` |


### <a name="network-ports-required-to-communicate-with-the-service"></a>Puertos de red necesarios para comunicarse con el servicio

| Source | Destination | Protocolo | Port | Descripción  | 
| ------ | ----------- | :------: | :---:| ------------ | 
| Servidor DNS de la nube privada | Servidor DNS local | UDP | 53 | Cliente DNS: solicitudes de reenvío desde el vCenter de PC para cualquier consulta de DNS local (consulte la sección DNS siguiente) |  
| Servidor DNS local   | Servidor DNS de la nube privada | UDP | 53 | Cliente DNS: solicitudes de reenvío desde servicios locales a servidores DNS de la nube privada (consulte la sección DNS siguiente) |  
| Red local  | Servidor vCenter de nube privada  | TCP(HTTP)  | 80 | vCenter Server requiere el puerto 80 para las conexiones HTTP directas. El puerto 80 redirige las solicitudes al puerto HTTPS 443. Este redireccionamiento ayuda si usa `http://server` en lugar de `https://server`.  <br><br>WS-Management (también requiere que el puerto 443 esté abierto) <br><br>Si utiliza una base de datos de Microsoft SQL personalizada y no la base de datos agrupada de SQL Server 2008 en vCenter Server, los servicios de SQL Reporting usan el puerto 80. Al instalar vCenter Server, el instalador le pide que cambie el puerto HTTP para vCenter Server. Cambie el puerto HTTP de vCenter Server a un valor personalizado para asegurarse de que la instalación se realiza correctamente. Microsoft Internet Information Services (IIS) también usa el puerto 80. Consulte Conflicto entre vCenter Server e IIS para el puerto 80. |  
| Red de administración de la nube privada | Active Directory local  | TCP  | 389 | Este puerto debe estar abierto en todas las instancias remotas y locales de vCenter Server. Es el número de puerto LDAP de Servicios de directorio para el grupo de vCenter Server. El sistema vCenter Server debe enlazar con el puerto 389, incluso si no va a unir esta instancia de vCenter Server a un grupo en Linked Mode. Si se está ejecutando otro servicio en este puerto, es preferible quitarlo o cambiar su puerto por otro. Puede ejecutar el servicio LDAP en cualquier puerto desde el 1025 al 65 535.  Si esta instancia actúa como Active Directory de Microsoft Windows, cambie el número de puerto 389 por uno disponible entre 1025 y 65 535. Este puerto es opcional, para configurar AD local como un origen de identidad en el servidor vCenter de nube privada. |  
| Red local  | Servidor vCenter de nube privada  | TCP(HTTPS)  | 443 | Este puerto le permite acceder a vCenter desde la red local. El puerto predeterminado que el sistema vCenter Server usa para escuchar las conexiones desde el cliente de vSphere. Para permitir que el sistema vCenter Server reciba datos del cliente de vSphere, abra el puerto 443 en el firewall. El sistema vCenter Server también utiliza el puerto 443 para supervisar la transferencia de datos desde los clientes del SDK. Este puerto también se utiliza para los siguientes servicios: WS-Management (también requiere que el puerto 80 esté abierto). Acceso del cliente de vSphere a vSphere Update Manager. Conexiones de cliente de administración de redes de terceros a vCenter Server. Los clientes de administración de redes de terceros acceden a los hosts. |  
| Explorador web  | Hybrid Cloud Manager  | TCP(HTTPS) | 9443 | Interfaz de administración de aplicaciones virtuales de Hybrid Cloud Manager para la configuración del sistema de Hybrid Cloud Manager. |
| Red de administración  | Hybrid Cloud Manager | SSH | 22 | Acceso SSH de administrador a Hybrid Cloud Manager. |
| HCM | Puerta de enlace en la nube | TCP(HTTPS) | 8123 | Se envían instrucciones del servicio de replicación basadas en host a Hybrid Cloud Gateway. |
| HCM | Puerta de enlace en la nube | HTTP TCP(HTTPS) | 9443 | Se envían instrucciones de administración a la instancia de Hybrid Cloud Gateway local mediante la API REST. |
| Puerta de enlace en la nube | L2C | TCP(HTTPS) | 443 | Se envían instrucciones de administración de Cloud Gateway a L2C si L2C usa la misma ruta de acceso que Hybrid Cloud Gateway. |
| Puerta de enlace en la nube | Hosts de ESXi | TCP | 80,902 | Administración e implementación de OVF. |
| Puerta de enlace en la nube (local)| Puerta de enlace en la nube (remota) | UDP | 4500 | Necesario para IPSEC<br>   Intercambio de claves por red (IKEv2) para encapsular las cargas de trabajo para el túnel bidireccional. También se admite la traducción transversal de direcciones de red (NAT-T). |
| Puerta de enlace en la nube (local) | Puerta de enlace en la nube (remota)  | UDP | 500 | Obligatorio para IPSEC<br> Intercambio de claves por red (ISAKMP) para el túnel bidireccional. |
| Red vCenter local | Red de administración de la nube privada | TCP | 8000 |  vMotion de máquinas virtuales desde vCenter local a vCenter de nube privada   |     

## <a name="dhcp-and-dns-resolution-considerations"></a>Consideraciones sobre DHCP y la resolución de DNS
Las aplicaciones y cargas de trabajo que se ejecutan en un entorno de nube privada requieren servicios de resolución de nombres y DHCP para la búsqueda y asignación de direcciones IP. Para proporcionar estos servicios se requiere una infraestructura de DHCP y DNS adecuada. Puede configurar una máquina virtual para proporcionar estos servicios en el entorno de nube privada.  

Se recomienda usar el servicio DHCP integrado en NSX o el uso de un servidor DHCP local en la nube privada en lugar de enrutar el tráfico DHCP de difusión a través de la WAN de nuevo al entorno local.


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió sobre lo siguiente:

> [!div class="checklist"]
> * Requisitos de conectividad de red
> * DHCP en Azure VMware Solution

Una vez que tenga la red adecuada en su lugar, continúe con el siguiente tutorial para crear la nube privada de Azure VMware Solution.

> [!div class="nextstepaction"]
> [Tutorial: Creación de una nube privada de Azure VMware Solution](tutorial-create-private-cloud.md)
