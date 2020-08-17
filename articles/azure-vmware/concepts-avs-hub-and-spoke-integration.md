---
title: 'Concepto: integración de una implementación de Azure VMware Solution (AVS) en una arquitectura radial'
description: Obtenga información sobre las recomendaciones para la integración de una implementación de Azure VMware Solution (AVS) en una arquitectura radial existente o nueva en Azure.
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: 0d95ed81c5188eab0dc508f5320549c4a402e151
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87062934"
---
# <a name="integrate-azure-vmware-solution-avs-in-a-hub-and-spoke-architecture"></a>Integración de Azure VMware Solution (AVS) en una arquitectura radial

En este artículo, se ofrecen recomendaciones para la integración de una implementación de Azure VMware Solution (AVS) en una [arquitectura radial](/azure/architecture/reference-architectures/hybrid-networking/shared-services) existente o nueva en Azure. 

El escenario radial presupone un entorno de nube híbrida con cargas de trabajo en:

* Azure nativo con servicios IaaS o PaaS
* AVS 
* vSphere local

## <a name="architecture"></a>Architecture

El *Centro* es una red virtual de Azure que actúa como punto central de conectividad para el entorno local y la nube privada AVS. Los *radios* son redes virtuales emparejadas con el centro para permitir la comunicación entre redes virtuales.

El tráfico entre el centro de datos local, la nube privada de AVS y el centro pasa a través de las conexiones ExpressRoute. Las redes virtuales de radios suelen contener cargas de trabajo basadas en IaaS, pero pueden tener servicios PaaS como [App Service Environment](../app-service/environment/intro.md), que tiene integración directa con Virtual Network u otros servicios PaaS con [Azure Private Link](../private-link/index.yml) habilitado. 

En el diagrama se muestra un ejemplo de una implementación radial en Azure conectada al entorno local y AVS a través de ExpressRoute.

:::image type="content" source="./media/hub-spoke/avs-hub-and-spoke-deployment.png" alt-text="Implementación de la integración radial de AVS":::




La arquitectura consta de los siguientes componentes principales:

-   **Sitio local:** centros de datos locales de clientes conectados a Azure a través de una conexión ExpressRoute.

-   **Nube privada de AVS:** AVS SDDC formado por uno o varios clústeres de vSphere, cada uno con un máximo de 16 nodos.

-   **Puerta de enlace de ExpressRoute:** permite la comunicación entre la nube privada de AVS, la red local, los servicios compartidos en la red virtual de centro y las cargas de trabajo que se ejecutan en redes virtuales de radios.

    > [!NOTE]
    > **Consideraciones sobre VPN S2S:** En el caso de las implementaciones de producción de AVS, no se admite Azure S2S debido a los requisitos de red para HCX. Sin embargo, se puede usar para una implementación de PoC o de no producción que no requiera HCX.

-   **Red virtual de centro:** actúa como el punto central de conectividad para la red local y la nube privada de AVS.

-   **Red virtual de radios**

    -   **Radio de IaaS:** un radio de IaaS hospedará las cargas de trabajo basadas en Azure IaaS, incluidos los conjuntos de disponibilidad de máquina virtual y los conjuntos de escalado de máquinas virtuales, así como los componentes de red correspondientes.

    -   **Radio de PaaS:** un radio de PaaS hospeda los servicios de Azure PaaS mediante el direccionamiento privado, gracias al [punto de conexión privado](../private-link/private-endpoint-overview.md) y a [Private Link](../private-link/private-link-overview.md).

-   **Azure Firewall:** actúa como la pieza central para segmentar el tráfico entre los radios, el entorno local y AVS.

-   **Application Gateway:** expone y protege las aplicaciones web que se ejecutan en Azure IaaS/PaaS o en las máquinas virtuales (VM) de AVS. Se integra con otros servicios como API Management.

## <a name="network-and-security-considerations"></a>Consideraciones sobre red y seguridad

Las conexiones ExpressRoute permiten que el tráfico fluya entre el entorno local, AVS y el tejido de red de Azure. AVS usa [Global Reach de ExpressRoute](../expressroute/expressroute-global-reach.md) para implementar esta conectividad.

La conectividad local también puede usar Global Reach de ExpressRoute, pero no es obligatorio.

* **Flujo de tráfico del entorno local a AVS**

  :::image type="content" source="media/hub-spoke/on-prem-to-avs-traffic-flow.png" alt-text="Flujo de tráfico del entorno local a AVS":::


* **Flujo de tráfico de AVS a VNET de centro**

  :::image type="content" source="media/hub-spoke/avs-to-hub-vnet-traffic-flow.png" alt-text="Flujo de tráfico de AVS a la red virtual de centro":::


Puede encontrar más detalles sobre los conceptos de interconectividad y redes de AVS en la [documentación del producto AVS](./concepts-networking.md).

### <a name="traffic-segmentation"></a>Segmentación de tráfico

[Azure Firewall](../firewall/index.yml) es la pieza central de la topología radial implementada en la red virtual de centro. Use Azure Firewall u otra aplicación virtual de red compatible con Azure para establecer reglas de tráfico y segmentar la comunicación entre los distintos radios, entornos locales y cargas de trabajo de AVS.

Cree tablas de rutas para dirigir el tráfico a Azure Firewall.  En el caso de las redes virtuales de radios, cree una ruta que establezca la ruta predeterminada a la interfaz interna de Azure Firewall, de esta manera cuando una carga de trabajo de la red virtual deba alcanzar el espacio de direcciones de AVS, el firewall puede evaluarla y aplicar la regla de tráfico correspondiente para permitirla o denegarla.  

:::image type="content" source="media/hub-spoke/create-route-table-to-direct-traffic.png" alt-text="Cree tablas de rutas para dirigir el tráfico a Azure Firewall":::


> [!IMPORTANT]
> No se admite una ruta con el prefijo de dirección 0.0.0.0/0 en el valor **GatewaySubnet**.

Establezca rutas para redes específicas en la tabla de rutas correspondiente. Por ejemplo, las rutas para llegar a los prefijos IP de administración y cargas de trabajo de AVS desde el entorno local y viceversa, enrutando todo el tráfico desde el entorno local a la nube privada AVS a través de Azure Firewall.

:::image type="content" source="media/hub-spoke/specify-gateway-subnet-for-route-table.png" alt-text="Establezca rutas para redes específicas en la tabla de rutas correspondiente":::

Un segundo nivel de segmentación del tráfico que usa los grupos de seguridad de red dentro de la red radial para crear una directiva de tráfico más pormenorizada. 


### <a name="application-gateway"></a>Application Gateway

Se ha probado Azure Application Gateway V1 y V2 con aplicaciones web que se ejecutan en máquinas virtuales AVS como grupo de back-end. Application Gateway es actualmente el único método admitido para exponer las aplicaciones web que se ejecutan en máquinas virtuales AVS en Internet. También puede exponer las aplicaciones a los usuarios internos de forma segura.

:::image type="content" source="media/hub-spoke/avs-second-level-traffic-segmentation.png" alt-text="Segundo nivel de segmentación del tráfico con los grupos de seguridad de red":::


### <a name="jumpbox-and-azure-bastion"></a>Jumpbox y Azure Bastion

Acceda al entorno de AVS con Jumpbox, que es una máquina virtual de Windows 10 o Windows Server implementada en la subred de servicio compartida dentro de la red virtual de centro.

Como práctica recomendada de seguridad, implemente el servicio [Microsoft Azure Bastion](../bastion/index.yml) dentro de la red virtual de centro. Azure Bastion proporciona el acceso RDP y SSH sin problemas a las máquinas virtuales implementadas en Azure sin necesidad de aprovisionar direcciones IP públicas a esos recursos. Una vez que aprovisione el servicio de Azure Bastion, puede acceder a la máquina virtual seleccionada desde Azure Portal. Después de establecer la conexión, se abre una nueva pestaña que muestra el escritorio de Jumpbox y, desde ese escritorio, puede acceder al plano de administración de la nube privada de AVS.

> [!IMPORTANT]
> No asigne una dirección IP pública a la máquina virtual Jumpbox ni exponga el puerto 3389/TCP a la red pública de Internet. 


:::image type="content" source="media/hub-spoke/azure-bastion-hub-vnet.png" alt-text="Red virtual de centro de Azure Bastion":::


## <a name="azure-dns-resolution-considerations"></a>Consideraciones sobre la resolución Azure DNS

Para la resolución de Azure DNS hay dos opciones disponibles:

-   Use los controladores de dominio de Azure Active Directory (Azure AD) implementados en el centro (descritos en [Consideraciones de identidad](#identity-considerations)) como servidores de nombres.

-   Implemente y configure una zona privada de Azure DNS.

El mejor enfoque consiste en combinar ambos para proporcionar una resolución de nombres confiable para AVS, el entorno local y Azure.

Como recomendación de diseño general, use la infraestructura de Azure DNS existente (en este caso, DNS integrado en Active Directory) implementada en al menos dos máquinas virtuales de Azure implementadas en la red virtual de centro y configuradas en las redes virtuales de radios para usar esos servidores de Azure DNS en la configuración de DNS.

Todavía se puede usar el DNS privado de Azure en el que la zona de DNS privado de Azure está vinculada a las redes virtuales y los servidores DNS se usan como resoluciones híbridas con reenvío condicional a los nombres DNS locales o AVS utilizando la infraestructura de DNS privado de Azure.

Hay varias consideraciones que se deben tener en cuenta para las zonas privadas de Azure DNS:

* El registro automático se debe habilitar para que Azure DNS también administre automáticamente el ciclo de vida de los registros DNS de las máquinas virtuales implementadas en redes virtuales de radios.
* El número máximo de zonas DNS privadas a las que se puede vincular una red virtual con el registro automático habilitado es solo una.
* El número máximo de zonas DNS privadas a las que se puede vincular una red virtual sin el registro automático habilitado es 1000.

Los servidores locales y AVS se pueden configurar con reenviadores condicionales a las máquinas virtuales de resolución en Azure para la zona de DNS privado de Azure.

## <a name="identity-considerations"></a>Consideraciones de identidad

En lo que respecta a la identidad, el mejor enfoque consiste en implementar al menos un controlador de dominio de AD en el centro, mediante la subred de servicio compartida, idealmente dos de ellos en el modo distribuido por zonas o en un conjunto de disponibilidad de máquina virtual. Consulte [Centro de arquitectura de Azure](/azure/architecture/reference-architectures/identity/adds-extend-domain) para extender el dominio de AD local a Azure.

Además, implemente otro controlador de dominio en el lado AVS para que actúe como identidad y origen DNS dentro del entorno de vSphere.

En el caso de vCenter y SSO, establezca un origen de identidad en Azure Portal, en **Administrar \> Identidad \> Orígenes de identidad**.

Como práctica recomendada, integre el [dominio de AD con Azure Active Directory](/azure/architecture/reference-architectures/identity/azure-ad).

<!-- LINKS - external -->
[Azure Architecture Center]: /azure/architecture/

[Hub & Spoke topology]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke

[Azure networking documentation]: ../networking/index.yml

<!-- LINKS - internal -->
