---
title: 'Concepto: integración de una implementación de Azure VMware Solution en una arquitectura en estrella tipo hub-and-spoke'
description: Conozca las recomendaciones para la integración de una implementación de Azure VMware Solution en una arquitectura en estrella tipo hub-and-spoke existente o nueva en Azure.
ms.topic: conceptual
ms.date: 08/20/2020
ms.openlocfilehash: deb2756f7e83250ff58836098dc4954ec482fbda
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88684524"
---
# <a name="integrate-azure-vmware-solution-in-a-hub-and-spoke-architecture"></a>Integración de Azure VMware Solution en una arquitectura en estrella tipo hub-and-spoke

En este artículo, se ofrecen recomendaciones para la integración de una implementación de Azure VMware Solution en una [arquitectura en estrella tipo hub-and-spoke](/azure/architecture/reference-architectures/hybrid-networking/shared-services) existente o nueva en Azure. 

El escenario radial presupone un entorno de nube híbrida con cargas de trabajo en:

* Azure nativo con servicios IaaS o PaaS
* Azure VMware Solution 
* vSphere local

## <a name="architecture"></a>Architecture

El *concentrador* (hub) es una red virtual de Azure que actúa como punto central de conectividad al entorno local y la nube privada de Azure VMware Solution. Los *radios* son redes virtuales emparejadas con el centro para permitir la comunicación entre redes virtuales.

El tráfico entre el centro de datos local, la nube privada de Azure VMware Solution y el concentrador pasa por las conexiones ExpressRoute. Las redes virtuales de radios suelen contener cargas de trabajo basadas en IaaS, pero pueden tener servicios PaaS como [App Service Environment](../app-service/environment/intro.md), que tiene integración directa con Virtual Network u otros servicios PaaS con [Azure Private Link](../private-link/index.yml) habilitado. 

En el diagrama se muestra un ejemplo de una implementación en estrella tipo hub-and-spoke en Azure conectada al entorno local y a Azure VMware Solution a través de ExpressRoute.

:::image type="content" source="./media/hub-spoke/avs-hub-and-spoke-deployment.png" alt-text="Implementación de la integración de la arquitectura en estrella tipo hub-and-spoke de Azure VMware Solution" border="false":::

La arquitectura consta de los siguientes componentes principales:

-   **Sitio local:** centros de datos locales de clientes conectados a Azure a través de una conexión ExpressRoute.

-   **Nube privada de Azure VMware Solution:** SDDC de Azure VMware Solution formado por uno o varios clústeres de vSphere, cada uno con un máximo de 16 nodos.

-   **Puerta de enlace de ExpressRoute:** permite la comunicación entre la nube privada de Azure VMware Solution, la red local, los servicios compartidos en la red virtual de concentrador y las cargas de trabajo que se ejecutan en redes virtuales de radio.

    > [!NOTE]
    > **Consideraciones sobre VPN S2S:** en el caso de las implementaciones de producción de Azure VMware Solution, no se admite Azure S2S debido a los requisitos de red para HCX. Sin embargo, se puede usar para una implementación de PoC o de no producción que no requiera HCX.

-   **Red virtual de centro:** actúa como punto central de conectividad a la red local y la nube privada de Azure VMware Solution.

-   **Red virtual de radios**

    -   **Radio de IaaS:** un radio de IaaS hospeda las cargas de trabajo basadas en Azure IaaS, incluidos los conjuntos de disponibilidad de máquina virtual y los conjuntos de escalado de máquinas virtuales, así como los componentes de red correspondientes.

    -   **Radio de PaaS:** un radio de PaaS hospeda los servicios de Azure PaaS mediante el direccionamiento privado, gracias al [punto de conexión privado](../private-link/private-endpoint-overview.md) y a [Private Link](../private-link/private-link-overview.md).

-   **Azure Firewall:** actúa como pieza central para segmentar el tráfico entre los radios, el entorno local y Azure VMware Solution.

-   **Application Gateway:** expone y protege las aplicaciones web que se ejecutan en Azure IaaS/PaaS o en máquinas virtuales (VM) de Azure VMware Solution. Se integra con otros servicios como API Management.

## <a name="network-and-security-considerations"></a>Consideraciones sobre red y seguridad

Las conexiones ExpressRoute permiten que el tráfico fluya entre el entorno local, Azure VMware Solution y el tejido de red de Azure. Azure VMware Solution emplea [Global Reach de ExpressRoute](../expressroute/expressroute-global-reach.md) para implementar esta conectividad.

La conectividad local también puede usar Global Reach de ExpressRoute, pero no es obligatorio.

* **Flujo de tráfico del entorno local a Azure VMware Solution**

  :::image type="content" source="media/hub-spoke/on-prem-to-avs-traffic-flow.png" alt-text="Flujo de tráfico del entorno local a Azure VMware Solution" border="false":::


* **Flujo de tráfico de Azure VMware Solution a la red virtual de concentrador**

  :::image type="content" source="media/hub-spoke/avs-to-hub-vnet-traffic-flow.png" alt-text="Flujo de tráfico de Azure VMware Solution a la red virtual de concentrador" border="false":::


Puede encontrar más detalles sobre los conceptos de interconectividad y redes de Azure VMware Solution en la [documentación del producto de Azure VMware Solution](./concepts-networking.md).

### <a name="traffic-segmentation"></a>Segmentación de tráfico

[Azure Firewall](../firewall/index.yml) es la pieza central de la topología radial implementada en la red virtual de centro. Use Azure Firewall u otra aplicación virtual de red compatible con Azure para establecer reglas de tráfico y segmentar la comunicación entre los distintos radios, el entorno local y las cargas de trabajo de Azure VMware Solution.

Cree tablas de rutas para dirigir el tráfico a Azure Firewall.  En el caso de las redes virtuales de radio, cree una ruta que establezca la ruta predeterminada a la interfaz interna de Azure Firewall; de este modo, cuando una carga de trabajo de la red virtual deba alcanzar el espacio de direcciones de Azure VMware Solution, el firewall puede evaluarla y aplicar la regla de tráfico correspondiente para permitirla o denegarla.  

:::image type="content" source="media/hub-spoke/create-route-table-to-direct-traffic.png" alt-text="Cree tablas de rutas para dirigir el tráfico a Azure Firewall":::


> [!IMPORTANT]
> No se admite una ruta con el prefijo de dirección 0.0.0.0/0 en el valor **GatewaySubnet**.

Establezca rutas para redes específicas en la tabla de rutas correspondiente. Por ejemplo, rutas para llegar a los prefijos IP de administración y cargas de trabajo de Azure VMware Solution desde el entorno local y viceversa, donde todo el tráfico se enruta desde el entorno local a la nube privada de Azure VMware Solution mediante Azure Firewall.

:::image type="content" source="media/hub-spoke/specify-gateway-subnet-for-route-table.png" alt-text="Establezca rutas para redes específicas en la tabla de rutas correspondiente":::

Un segundo nivel de segmentación del tráfico que usa los grupos de seguridad de red dentro de la red radial para crear una directiva de tráfico más pormenorizada. 


### <a name="application-gateway"></a>Application Gateway

Se ha probado Azure Application Gateway V1 y V2 con aplicaciones web que se ejecutan en máquinas virtuales de Azure VMware Solution como grupo de back-end. Application Gateway es actualmente el único método admitido para exponer las aplicaciones web que se ejecutan en máquinas virtuales de Azure Application Gateway a Internet. También puede exponer las aplicaciones a los usuarios internos de forma segura.

:::image type="content" source="media/hub-spoke/avs-second-level-traffic-segmentation.png" alt-text="Segundo nivel de segmentación del tráfico con los grupos de seguridad de red" border="false":::


### <a name="jumpbox-and-azure-bastion"></a>Jumpbox y Azure Bastion

Acceda al entorno de Azure VMware Solution con Jumpbox, que es una máquina virtual de Windows 10 o Windows Server implementada en la subred de servicio compartida dentro de la red virtual de concentrador.

Como práctica recomendada de seguridad, implemente el servicio [Microsoft Azure Bastion](../bastion/index.yml) dentro de la red virtual de centro. Azure Bastion proporciona el acceso RDP y SSH sin problemas a las máquinas virtuales implementadas en Azure sin necesidad de aprovisionar direcciones IP públicas a esos recursos. Una vez que aprovisione el servicio de Azure Bastion, puede acceder a la máquina virtual seleccionada desde Azure Portal. Después de establecer la conexión, se abre una nueva pestaña que muestra el escritorio de Jumpbox y, desde ese escritorio, puede acceder al plano de administración de la nube privada de Azure VMware Solution.

> [!IMPORTANT]
> No asigne una dirección IP pública a la máquina virtual Jumpbox ni exponga el puerto 3389/TCP a la red pública de Internet. 


:::image type="content" source="media/hub-spoke/azure-bastion-hub-vnet.png" alt-text="Red virtual de centro de Azure Bastion" border="false":::


## <a name="azure-dns-resolution-considerations"></a>Consideraciones sobre la resolución Azure DNS

Para la resolución de Azure DNS hay dos opciones disponibles:

-   Use los controladores de dominio de Azure Active Directory (Azure AD) implementados en el centro (descritos en [Consideraciones de identidad](#identity-considerations)) como servidores de nombres.

-   Implemente y configure una zona privada de Azure DNS.

El mejor enfoque consiste en combinar ambos para proporcionar una resolución de nombres confiable para Azure VMware Solution, el entorno local y Azure.

Como recomendación de diseño general, use la infraestructura de Azure DNS existente (en este caso, DNS integrado en Active Directory) implementada en al menos dos máquinas virtuales de Azure implementadas en la red virtual de centro y configuradas en las redes virtuales de radios para usar esos servidores de Azure DNS en la configuración de DNS.

Todavía se puede usar el DNS privado de Azure en el que la zona de DNS privado de Azure está vinculada a las redes virtuales y los servidores DNS se usan como resolución híbrida con reenvío condicional al entorno local o a Azure VMware Solution que ejecutan DNS aprovechando la infraestructura de DNS privado de Azure.

Hay varias consideraciones que se deben tener en cuenta para las zonas privadas de Azure DNS:

* El registro automático se debe habilitar para que Azure DNS también administre automáticamente el ciclo de vida de los registros DNS de las máquinas virtuales implementadas en redes virtuales de radios.
* El número máximo de zonas DNS privadas a las que se puede vincular una red virtual con el registro automático habilitado es solo una.
* El número máximo de zonas DNS privadas a las que se puede vincular una red virtual sin el registro automático habilitado es 1000.

Los servidores locales y de Azure VMware Solution se pueden configurar con reenviadores condicionales a las máquinas virtuales de resolución de Azure para la zona de DNS privado de Azure.

## <a name="identity-considerations"></a>Consideraciones de identidad

En lo que respecta a la identidad, el mejor enfoque consiste en implementar al menos un controlador de dominio de AD en el centro, mediante la subred de servicio compartida, idealmente dos de ellos en el modo distribuido por zonas o en un conjunto de disponibilidad de máquina virtual. Consulte [Centro de arquitectura de Azure](/azure/architecture/reference-architectures/identity/adds-extend-domain) para extender el dominio de AD local a Azure.

Además, implemente otro controlador de dominio en el lado de Azure VMware Solution para que actúe como identidad y origen DNS dentro del entorno de vSphere.

En el caso de vCenter y SSO, establezca un origen de identidad en Azure Portal, en **Administrar \> Identidad \> Orígenes de identidad**.

Como práctica recomendada, integre el [dominio de AD con Azure Active Directory](/azure/architecture/reference-architectures/identity/azure-ad).

<!-- LINKS - external -->
[Azure Architecture Center]: /azure/architecture/

[Hub & Spoke topology]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke

[Azure networking documentation]: ../networking/index.yml

<!-- LINKS - internal -->
