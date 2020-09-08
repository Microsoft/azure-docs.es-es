---
title: Implementación de Traffic Manager para equilibrar las cargas de trabajo de Azure VMware Solution (AVS)
description: Aprenda a integrar Traffic Manager con Azure VMware Solution (AVS) para equilibrar las cargas de trabajo de aplicaciones en varios puntos de conexión de distintas regiones.
ms.topic: how-to
ms.date: 08/14/2020
ms.openlocfilehash: dc3107c6a237273b103fe52a91b3569d9e694e1a
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88642554"
---
# <a name="deploy-traffic-manager-to-balance-azure-vmware-solution-avs-workloads"></a>Implementación de Traffic Manager para equilibrar las cargas de trabajo de Azure VMware Solution (AVS)

Este artículo lo guiará por el proceso de integración de Traffic Manager con Azure VMware Solution (AVS) para equilibrar las cargas de trabajo de aplicaciones en varios puntos de conexión. Veremos un escenario en el que Traffic Manager dirigirá el tráfico entre tres puertas de enlace de aplicación que abarcan varias regiones de AVS: Oeste de EE. UU., Oeste de Europa y servidor local en la región Este de EE. UU. 

Azure Traffic Manager es un equilibrador de carga de tráfico basado en DNS que le permite distribuir el tráfico de forma óptima a servicios de regiones de Azure globales. Equilibrará la carga del tráfico de aplicación entre las cargas de trabajo en ejecución de Azure y los puntos de conexión públicos externos. Para más información sobre Traffic Manager, consulte [¿Qué es Traffic Manager?](../traffic-manager/traffic-manager-overview.md)

Revise primero los [requisitos previos](#prerequisites); a continuación, analizaremos los procedimientos para lograr lo siguiente:

> [!div class="checklist"]
> * Comprobación de la configuración de las puertas de enlace de aplicación
> * Comprobación de la configuración del segmento NSX-T
> * Creación de un perfil de Traffic Manager
> * Incorporación de puntos de conexión externos en el perfil de Traffic Manager

## <a name="topology"></a>Topología

Como se muestra en la figura siguiente, Azure Traffic Manager proporciona equilibrio de carga para las aplicaciones en el nivel de DNS entre los puntos de conexión regionales. Las puertas de enlace de aplicación tienen miembros del grupo de back-end configurados como servidores IIS y se hace referencia a ellos como puntos de conexión externos de AVS.

La conexión a través de la red virtual entre las dos regiones de la nube privada de AVS, Oeste de EE. UU. y Oeste de Europa, y un servidor local en la región Este de EE. UU., usa una puerta de enlace de ExpressRoute.   

![Integración de Traffic Manager con AVS](media/traffic-manager/traffic-manager-topology.png)
 
## <a name="prerequisites"></a>Requisitos previos

- Tres máquinas virtuales configuradas como servidores de Microsoft IIS que se ejecutan en regiones de AVS distintas: Oeste de EE. UU., Oeste de Europa y servidor local. 

- Una puerta de enlace de aplicación con puntos de conexión externos en las regiones Oeste de EE. UU., Oeste de Europa y servidor local.

- Hospede con conectividad de Internet para la comprobación. 

## <a name="verify-configuration-of-your-application-gateways"></a>Comprobación de la configuración de las puertas de enlace de aplicación

[Azure Application Gateway](https://azure.microsoft.com/services/application-gateway/) es un equilibrador de carga de tráfico web de capa 7 que permite administrar el tráfico a las aplicaciones web. Para más información sobre Application Gateway, consulte [¿Qué es Azure Application Gateway?](../application-gateway/overview.md) 

En este escenario, se configuran tres instancias de puerta de enlace de aplicación como puntos de conexión de AVS externos. Las puertas de enlace de aplicación tienen máquinas virtuales de AVS configuradas como miembros del grupo de back-end para equilibrar la carga de las solicitudes de capa 7 entrantes. (Para aprender a configurar Application Gateway con máquinas virtuales de AVS como grupos de back-end, consulte [Uso de Azure Application Gateway para proteger aplicaciones web en Azure VMware Solution](protect-avs-web-apps-with-app-gateway.md)).  

En los pasos siguientes se comprueba la configuración correcta de las puertas de enlace de aplicación.

1. Abra Azure Portal y seleccione **Puertas de enlace de aplicación** para ver una lista de las puertas de enlace de aplicación actuales. 

    En este escenario, configuramos tres puertas de enlace de aplicación:
    - AVS-GW-WUS
    - AVS-GW-EUS (local)
    - AVS-GW-WEU

    :::image type="content" source="media/traffic-manager/app-gateways-list-1.png" alt-text="Lista de puertas de enlace de aplicación." lightbox="media/traffic-manager/app-gateways-list-1.png":::

2. Seleccione una de las puertas de enlace de aplicación previamente implementadas. Se abre una ventana que muestra diversa información sobre la puerta de enlace de aplicación. Seleccione **Grupos de back-end** para comprobar la configuración de uno de los grupos de back-end.

   :::image type="content" source="media/traffic-manager/backend-pool-config.png" alt-text="Detalles de la puerta de enlace de aplicación." lightbox="media/traffic-manager/backend-pool-config.png":::
 
3. En este caso, vemos un miembro de un grupo de back-end de máquina virtual configurado como servidor web con una dirección IP de 172.29.1.10.
 
     :::image type="content" source="media/traffic-manager/backend-pool-ip-address.png" alt-text="Edite el grupo de back-end.":::

    De manera similar, puede comprobar la configuración de las otras puertas de enlace de aplicación y los miembros del grupo de back-end. 

## <a name="verify-configuration-of-the-nsx-t-segment"></a>Comprobación de la configuración del segmento NSX-T

Los segmentos de red creados en NSX-T Manager se usan como redes para máquinas virtuales en vCenter. Para más información, consulte el tutorial [Creación de un segmento de red de NSX-T en Azure VMware Solution (AVS)](tutorial-nsx-t-network-segment.md).

En nuestro escenario, se configura un segmento NSX-T en el entorno de AVS al que está conectada la máquina virtual miembro del grupo de back-end.

1. Seleccione **Segmentos** para ver los segmentos configurados. En este caso, vemos que Contoso-segment1 está conectado a la puerta de enlace Contoso-T01, un enrutador flexible de nivel 1.

    :::image type="content" source="media/traffic-manager/nsx-t-segment-avs.png" alt-text="Perfiles de segmento en NSX-T Manager.":::    

2. Seleccione **Tier-1 Gateways** (Puertas de enlace de nivel 1) para ver una lista de las puertas de enlace de nivel 1 con el número de segmentos vinculados. Seleccione el segmento vinculado a Contoso-T01. Se abre una ventana que muestra la interfaz lógica configurada en el enrutador de nivel 01. Esto sirve como puerta de enlace a la máquina virtual miembro del grupo de back-end conectada al segmento.

   :::image type="content" source="media/traffic-manager/nsx-t-segment-linked-2.png" alt-text="Segmentos vinculados.":::    

3. En el cliente de vSphere de la máquina virtual, seleccione la máquina virtual para ver sus detalles. Tenga en cuenta que su dirección IP coincide con lo que vimos en el paso 3 de la sección anterior: 172.29.1.10.

    :::image type="content" source="media/traffic-manager/nsx-t-vm-details.png" alt-text="Detalles de la máquina virtual.":::    

4. Seleccione la máquina virtual y haga clic en **ACCIONES > Editar configuración** para comprobar la conexión con el segmento NSX-T.

## <a name="create-your-traffic-manager-profile"></a>Creación de un perfil de Traffic Manager

1. Inicie sesión en [Azure Portal](https://rc.portal.azure.com/#home). En **Servicios de Azure > Redes**, seleccione **Perfiles de Traffic Manager**.

2. Seleccione **+ Agregar** para crear un perfil nuevo de Traffic Manager.
 
3. Proporcione el nombre del perfil, el método de enrutamiento (en este escenario, se usará Ponderado; consulte [Métodos de enrutamiento de Traffic Manager](../traffic-manager/traffic-manager-routing-methods.md)), la suscripción y el grupo de recursos, y seleccione **Crear**.

## <a name="add-external-endpoints-into-the-traffic-manager-profile"></a>Incorporación de puntos de conexión externos en el perfil de Traffic Manager

1. Seleccione el perfil de Traffic Manager en el panel de resultados de la búsqueda, seleccione **Puntos de conexión** y, a continuación, **+ Agregar**.

2. Escriba los detalles necesarios: Tipo, Nombre, Nombre de dominio completo (FQDN) o Dirección IP y Peso (en este escenario, asignamos un peso de 1 a cada punto de conexión). Seleccione **Agregar**.

    :::image type="content" source="media/traffic-manager/traffic-manager-profile.png" alt-text="Perfil de Traffic Manager: agregar punto de conexión.":::  
 
    Esto crea el punto de conexión externo. El estado del monitor debe ser **En línea**. 

    Repita los mismos pasos para crear dos puntos de conexión externos, uno en una región distinta y el otro, en el entorno local. Una vez creados, los tres se mostrarán en el perfil de Traffic Manager y el estado de los tres deben ser **En línea**.

3. Seleccione **Información general**. Copie la dirección URL en **Nombre DNS**.

    :::image type="content" source="media/traffic-manager/traffic-manager-endpoints.png" alt-text="Información general de Traffic Manager con el nombre DNS."::: 

4. Pegue la dirección URL del nombre DNS en un explorador. En la captura de pantalla siguiente se muestra el tráfico que se dirige a la región Oeste de Europa.

    :::image type="content" source="media/traffic-manager/traffic-to-west-europe.png" alt-text="Tráfico enrutado a la región Oeste de Europa."::: 

5. Actualice el explorador. En la captura de pantalla siguiente se muestra el tráfico que ahora se dirige a otro conjunto de miembros del grupo de back-end en la región Oeste de EE. UU.

    :::image type="content" source="media/traffic-manager/traffic-to-west-us.png" alt-text="Tráfico enrutado a la región Oeste de EE. UU."::: 

6. Vuelva a actualizar el explorador. En la captura de pantalla siguiente se muestra el tráfico que ahora se dirige al conjunto final de miembros del grupo de back-end en el entorno local.

    :::image type="content" source="media/traffic-manager/traffic-to-on-premises.png" alt-text="Tráfico enrutado al servidor local.":::

## <a name="next-steps"></a>Pasos siguientes

Más información sobre:

- [Uso de Azure Application Gateway en Azure VMware Solution (AVS)](protect-avs-web-apps-with-app-gateway.md)
- [Métodos de enrutamiento de Traffic Manager](../traffic-manager/traffic-manager-routing-methods.md)
- [Combinación de servicios de equilibrio de carga en Azure](../traffic-manager/traffic-manager-load-balancing-azure.md)
- [Medición del rendimiento de Traffic Manager](../traffic-manager/traffic-manager-performance-considerations.md)
