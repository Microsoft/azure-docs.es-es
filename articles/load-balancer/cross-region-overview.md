---
title: Equilibrador de carga entre regiones (versión preliminar)
titleSuffix: Azure Load Balancer
description: Información general del nivel del equilibrador de carga entre regiones para Azure Load Balancer.
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/01/2020
ms.author: allensu
ms.custom: references_regions
ms.openlocfilehash: 11184b2e9770ba6fd7c4ec9d04862c0890ac94c6
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931975"
---
# <a name="cross-region-load-balancer-preview"></a>Equilibrador de carga entre regiones (versión preliminar)

Azure Load Balancer distribuye el tráfico de entrada que llega al front-end del equilibrador de carga a las instancias del grupo de servidores back-end.

Azure Standard Load Balancer admite el equilibrio de carga entre regiones que habilita escenarios de alta disponibilidad con redundancia geográfica como, por ejemplo:

* Tráfico entrante que se origina desde varias regiones.
* [Conmutación por error global instantánea](#regional-redundancy) a la siguiente implementación regional óptima.
* Distribución de la carga entre regiones en la región de Azure más cercana con [latencia ultra](#ultra-low-latency).
* Capacidad de [escalar verticalmente o reducir verticalmente](#ability-to-scale-updown-behind-a-single-endpoint) detrás de un solo punto de conexión.
* [Dirección IP estática](#static-ip)
* [Conservación de la dirección IP de cliente](#client-ip-preservation)
* [Compilación en la solución del equilibrador de carga existente](#build-cross-region-solution-on-existing-azure-load-balancer) sin curva de aprendizaje

> [!IMPORTANT]
> El equilibrador de carga entre regiones se encuentra actualmente en versión preliminar y no está disponible con carácter general.  Para tener acceso a la versión preliminar del equilibrador de carga entre regiones, póngase en contacto con: [crossregionlb@microsoft.com](mailto:crossregionlb@microsoft.com). </br> </br>
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

El equilibrio de carga entre regiones ofrece las mismas ventajas de alto rendimiento y baja latencia que la instancia de Standard Load Balancer regional. 

La configuración de direcciones IP de front-end del equilibrador de carga entre regiones es estática y se anuncia a través de la [mayoría de las regiones de Azure](#participating-regions).

:::image type="content" source="./media/cross-region-overview/cross-region-load-balancer.png" alt-text="Diagrama del equilibrador de carga entre regiones." border="true":::

> [!NOTE]
> El puerto de carga de back-end donde la regla de equilibrio del equilibrador de carga entre regiones debe coincidir con el puerto de front-end de la regla de equilibrio de carga o la regla NAT de entrada de la instancia de Standard Load Balancer regional. 

### <a name="regional-redundancy"></a>Redundancia regional

Configure la redundancia regional agregando una dirección IP pública de front-end global a los equilibradores de carga existentes. 

Si se produce un error en una región, el tráfico se enruta al siguiente equilibrador de carga regional correcto más cercano.  

El sondeo de estado del equilibrador de carga entre regiones recopila información sobre la disponibilidad cada 20 segundos. Si la disponibilidad de un equilibrador de carga regional baja a 0, el equilibrador de carga entre regiones detectará el error. A continuación, el equilibrador de carga regional se excluye de la rotación. 

:::image type="content" source="./media/cross-region-overview/global-region-view.png" alt-text="Diagrama de la vista del tráfico de la región global." border="true":::

### <a name="ultra-low-latency"></a>Latencia ultrabaja

El algoritmo de equilibrio de carga de proximidad geográfica se basa en la ubicación geográfica de los usuarios y las implementaciones regionales. 

El tráfico iniciado desde un cliente alcanzará la región participante más cercana y circulará por la red troncal global de Microsoft para llegar a la implementación regional más cercana. 

Por ejemplo, tiene un equilibrador de carga entre regiones con instancias de Standard Load Balancer en las regiones de Azure:

* Oeste de EE. UU.
* Norte de Europa

Si se inicia un flujo desde Seattle, el tráfico entra en Oeste de EE. UU. Esta región es la región participante más cercana de Seattle. El tráfico se enruta al equilibrador de carga de la región más cercana, que es Oeste de EE. UU.

El equilibrador de carga entre regiones de Azure usa el algoritmo de equilibrio de carga de proximidad geográfica para la decisión de enrutamiento. 

El modo de distribución de carga configurado de los equilibradores de carga regionales se usa para tomar la decisión de enrutamiento final al usarse varios equilibradores de carga regionales para la proximidad geográfica.

Para más información, consulte [Configurar el modo de distribución para Azure Load Balancer](https://docs.microsoft.com/azure/load-balancer/load-balancer-distribution-mode).


### <a name="ability-to-scale-updown-behind-a-single-endpoint"></a>Capacidad de escalar verticalmente o reducir verticalmente detrás de un solo punto de conexión

Cuando exponga el punto de conexión global de un equilibrador de carga entre regiones a los clientes, puede agregar o quitar implementaciones regionales detrás del punto de conexión global sin impacto en los clientes. 

<!---To learn about how to add or remove a regional deployment from the backend, read more [here](TODO: Insert CLI doc here).--->

### <a name="static-ip"></a>IP estática
El equilibrador de carga entre regiones viene con una dirección pública estática, que garantiza que la dirección IP siga siendo la misma. Para obtener más información sobre la dirección IP estática, lea más [aquí](https://docs.microsoft.com/azure/virtual-network/public-ip-addresses#allocation-method).

### <a name="client-ip-preservation"></a>Conservación de la dirección IP de cliente
El equilibrador de carga entre regiones es un equilibrador de carga de red de tránsito de nivel 4. Este tránsito conserva la dirección IP original del paquete.  La dirección IP original está disponible para el código que se ejecuta en la máquina virtual. Esta conservación permite aplicar lógica específica de una dirección IP.

## <a name="build-cross-region-solution-on-existing-azure-load-balancer"></a>Compilación de la solución entre regiones en la instancia de Azure Load Balancer existente
El grupo de back-end del equilibrador de carga entre regiones contiene uno o varios equilibradores de carga regionales. 

Agregue las implementaciones del equilibrador de carga existentes a un equilibrador de carga entre regiones para una implementación entre regiones altamente disponible.

La **región principal** es aquella en la que se implementa el equilibrador de carga entre regiones. Esta región no afecta al modo en que se enrutará el tráfico. Si una región principal deja de funcionar, no afectará al flujo de tráfico.

### <a name="home-regions"></a>Regiones principales
* Este de EE. UU. 2
* Oeste de EE. UU.
* Oeste de Europa
* Sudeste Asiático
* Centro de EE. UU.
* Norte de Europa
* Este de Asia

> [!NOTE]
> Solo puede implementar el equilibrador de carga entre regiones en una de las 8 regiones anteriores.

Una **región participante** es aquella en la que la dirección IP pública global del equilibrador de carga está disponible. 

El tráfico iniciado por el usuario se dirigirá a la región participante más cercana a través de la red principal de Microsoft. 

El equilibrador de carga entre regiones enruta el tráfico al equilibrador de carga regional adecuado.

### <a name="participating-regions"></a>Regiones participantes
* Este de EE. UU. 
* Oeste de Europa 
* Centro de EE. UU. 
* Este de EE. UU. 2 
* Oeste de EE. UU. 
* Norte de Europa 
* Centro-sur de EE. UU. 
* Oeste de EE. UU. 2 
* Sur de Reino Unido 2 
* Sudeste de Asia 
* Centro-Norte de EE. UU 
* Japón Oriental 
* Este de Asia 
* Centro-Oeste de EE. UU. 
* Sudeste de Australia 
* Este de Australia 
* Centro de la India 

## <a name="limitations"></a>Limitaciones

* Las configuraciones de direcciones IP de front-end entre regiones son solo públicas. Actualmente no se admite un front-end interno.

* No se puede agregar un equilibrador de carga interno o privado al grupo de back-end del equilibrador de carga entre regiones 

* Las configuraciones de direcciones IP de front-end IPv6 entre regiones no se admiten. 

* Un sondeo de estado no se puede configurar actualmente. Un sondeo de estado predeterminado recopila de forma automática información de disponibilidad sobre el equilibrador de carga regional cada 20 segundos. 

## <a name="pricing-and-sla"></a>Precios y contrato de nivel de servicio
El equilibrador de carga entre regiones comparte el [Acuerdo de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/load-balancer/v1_0/ ) de la instancia de Standard Load Balancer.

 
## <a name="next-steps"></a>Pasos siguientes

- Consulte el artículo sobre cómo [crear una instancia de Standard Load Balancer pública](quickstart-load-balancer-standard-public-portal.md) para empezar a usar un equilibrador de carga.
- Más información sobre [Azure Load Balancer](load-balancer-overview.md).
- [Preguntas frecuentes](load-balancer-faqs.md) sobre Load Balancer
