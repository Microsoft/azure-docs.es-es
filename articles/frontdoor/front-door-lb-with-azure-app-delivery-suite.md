---
title: 'Azure Front Door: Equilibrio de carga con el conjunto de entrega de aplicaciones de Azure | Microsoft Docs'
description: Este artículo le ayuda a saber cuándo Azure recomienda el equilibrio de carga con su conjunto de entrega de aplicaciones.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/10/2018
ms.author: duau
ms.openlocfilehash: 685ee9feaf057e4f2fae3cfe016624806f1ad00c
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89399470"
---
# <a name="load-balancing-with-azures-application-delivery-suite"></a>Equilibrio de carga con el conjunto de entrega de aplicaciones de Azure

## <a name="introduction"></a>Introducción
Microsoft Azure proporciona varios servicios globales y regionales para administrar cómo se distribuye el tráfico de red y se equilibra la carga: Traffic Manager, Front Door, Application Gateway y Load Balancer.  El uso de estos servicios juntos, en combinación con la arquitectura zonal y de varias regiones, le permite crear aplicaciones robustas, escalables y de alto rendimiento.

![Conjunto de entrega de aplicaciones ][1]
 
Estos servicios se dividen en dos categorías:
1. **Servicios de equilibrio de carga globales**, por ejemplo, Traffic Manager y Front Door distribuyen el tráfico de los usuarios finales entre los servidores back-end regionales, las nubes o incluso los servicios locales híbridos. El equilibrio de carga global enruta el tráfico al back-end de servicio más cercano y responde a los cambios en la confiabilidad o el rendimiento del servicio para mantener el rendimiento máximo siempre activo para los usuarios. 
2. **Servicios de equilibrio de carga regionales**, por ejemplo, Standard Load Balancer o Application Gateway ofrecen la posibilidad de distribuir el tráfico de las redes virtuales (VNET) entre las máquinas virtuales (VM) o los puntos de conexión de servicio zonales de una región.

La combinación de servicios globales y regionales en la aplicación proporciona una forma confiable, segura y efectiva de enrutar el tráfico hacia y desde los usuarios a los servicios IaaS, PaaS o locales. En la siguiente sección se describe cada uno de estos servicios.

## <a name="global-load-balancing"></a>Equilibrio de carga global
**Traffic Manager** proporciona equilibrio de carga de DNS global. Examina las solicitudes de DNS entrantes y responde con un back-end en buen estado, de acuerdo con la directiva de enrutamiento que el cliente ha seleccionado. Las opciones de los métodos de enrutamiento son:
- Enrutamiento de rendimiento, para enviar al solicitante al back-end más cercano en términos de latencia.
- Enrutamiento de prioridad, para dirigir todo el tráfico a un back-end, con otros servidores back-end como respaldo.
- Enrutamiento round-robin ponderado, que distribuye el tráfico según la ponderación asignada a cada back-end.
- Enrutamiento geográfico, para garantizar que los solicitantes ubicados en regiones geográficas específicas se dirigen a los servidores back-end asignados a esas regiones (por ejemplo, todas las solicitudes desde España deben dirigirse a la región de Azure Centro de Francia).
- Enrutamiento de subred, que permite asignar intervalos de direcciones IP a servidores back-end, de modo que las solicitudes que procedan de dichos servidores se enviarán al back-end especificado (por ejemplo, todos los usuarios que se conectan desde el intervalo de direcciones IP de la sede central corporativa obtendrán contenido web diferente al de los usuarios generales).

El cliente se conecta directamente a ese back-end. Azure Traffic Manager detecta cuándo un back-end está en mal estado y redirige a los clientes a otra instancia en buen estado. Consulte la documentación de [Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md) para más información sobre el servicio.

**Azure Front Door** proporciona aceleración dinámica de sitios web (DSA), que incluye equilibrio de carga global HTTP.  Examina las rutas de las solicitudes HTTP entrantes a la región o el back-end de servicio más cercanos para comprobar que existen el nombre de host, la ruta de la dirección URL y las reglas configuradas que se han especificado.  
Front Door termina las solicitudes HTTP en el perímetro de la red de Microsoft y las sondea activamente para detectar los cambios en la aplicación, el mantenimiento de la infraestructura o la latencia.  Front Door siempre enruta el tráfico al back-end más rápido y disponible (con un buen estado). Consulte la [arquitectura de enrutamiento](front-door-routing-architecture.md) y los [métodos de enrutamiento del tráfico](front-door-routing-methods.md) de Front Door para conocer más sobre el servicio.

## <a name="regional-load-balancing"></a>Equilibrio de carga regional
Application Gateway proporciona un controlador de entrega de aplicaciones (ADC) como servicio, que ofrece diversas funcionalidades de equilibrio de carga de nivel 7 para la aplicación. Permite a los clientes optimizar la productividad de las granjas de servidores web traspasando la carga de la terminación TLS con mayor actividad de CPU a Application Gateway. Otras funcionalidades de enrutamiento de nivel 7 son la distribución round robin del tráfico entrante, la afinidad de sesiones basada en cookies, el enrutamiento basado en ruta de acceso de URL y la posibilidad de hospedar varios sitios web tras una única instancia de Application Gateway. Application Gateway puede configurarse como una puerta de enlace accesible desde Internet, una puerta de enlace solo para uso interno o una combinación de las dos. Application Gateway está completamente administrada por Azure, es escalable y tiene una elevada disponibilidad. Cuenta con un amplio conjunto de funcionalidades de diagnóstico y registro, lo que facilita su administración.
Load Balancer, parte integral de la pila de Azure SDN, proporciona servicios de equilibrio de carga de capa 4 de alto rendimiento y baja latencia para todos los protocolos TCP y UDP. Administra conexiones entrantes y salientes. Puede configurar puntos de conexión con equilibrio de carga públicos e internos y definir reglas para asignar conexiones entrantes a destinos de grupo de back-end con opciones de sondeo de estado TCP y HTTP para administrar la disponibilidad del servicio.


## <a name="choosing-a-global-load-balancer"></a>Selección de un equilibrador de carga global
Al elegir un equilibrador de carga global entre Traffic Manager y Azure Front Door para el enrutamiento global, debe considerar las similitudes y las diferencias en los dos servicios.   Lo que proporcionan ambos servicios
- **Redundancia geográfica múltiple:** Si una región queda inactiva, el tráfico se dirige sin problemas a la siguiente región más cercana sin intervención del propietario de la aplicación.
- **Enrutamiento a la región más cercana:** El tráfico se enruta automáticamente a la región más cercana.

</br>En la tabla siguiente se describen las diferencias entre Traffic Manager y Azure Front Door:</br>

| Traffic Manager | Azure Front Door |
| --------------- | ------------------------ |
|**Cualquier protocolo:** Dado que Traffic Manager funciona en el nivel DNS, puede enrutar cualquier tipo de tráfico de red: HTTP, TCP, UDP, etc. | **Aceleración de HTTP:** Con Front Door, el tráfico se redirige mediante proxy en el perímetro de la red de Microsoft.  Por este motivo, las solicitudes HTTP(S) experimentan mejoras en la latencia y el rendimiento al reducirse la latencia en la negociación de TLS y usar las conexiones activas de AFD a la aplicación.|
|**Enrutamiento local:** Con el enrutamiento en una capa de DNS, el tráfico siempre va de punto a punto.  El enrutamiento desde una sucursal al centro de datos local puede tomar una ruta directa, incluso en su propia red con Traffic Manager. | **Escalabilidad independiente:** Como Front Door funciona con las solicitudes HTTP, las solicitudes a rutas de dirección URL diferentes se pueden enrutar a diferentes grupos de servidores back-end y servicios regionales (microservicios) en función de las reglas y el mantenimiento de cada microservicio de aplicación.|
|**Formato de facturación:** La facturación basada en DNS se escala con el número de usuarios y para servicios con más usuarios, con el fin de reducir el costo de un mayor uso. |**Seguridad alineada:** Front Door permite reglas como limitación de tasa y creación de listas ACL de IP para que pueda proteger los servidores back-end antes de que el tráfico llegue a la aplicación. 

</br>Debido a las ventajas de rendimiento, funcionamiento y seguridad para las cargas de trabajo HTTP con Front Door, se recomienda a los clientes usar el mencionado servicio con sus cargas de trabajo HTTP.    Traffic Manager y Front Door se pueden usar en paralelo para suministrar todo el tráfico de la aplicación. 

## <a name="building-with-azures-application-delivery-suite"></a>Creación con el conjunto de entrega de aplicaciones de Azure 
Se recomienda que todos los sitios web, API y servicios sean geográficamente redundantes y que entreguen el tráfico a sus usuarios desde la ubicación (latencia más baja) que les sea más cercana siempre que sea posible.  La combinación de servicios de Traffic Manager, Front Door, Application Gateway y Load Balancer le permite crear aplicaciones con redundancia geográfica y de zona para aumentar la confiabilidad, la escalabilidad y el rendimiento.

En el diagrama siguiente se describe un servicio de ejemplo que usa una combinación de todos estos servicios para crear un servicio web global.   En este caso, el arquitecto ha usado Traffic Manager a fin de enrutar los servidores back-end globales para la entrega de archivos y objetos, al tiempo que usa Front Door para enrutar globalmente las rutas de dirección URL que relacionan el patrón /store/* con el servicio que han migrado a App Service; mientras, el resto de las solicitudes se enruta a instancias regionales de Application Gateway.

En la región, y para el servicio IaaS, el desarrollador de aplicaciones ha decidido que las direcciones URL que coincidan con el patrón /images/* se sirvan desde un grupo dedicado de máquinas virtuales distintas del resto de la granja de servidores web.

Además, el grupo de máquinas virtuales predeterminado que sirve el contenido dinámico debe comunicarse con una base de datos de back-end hospedada en un clúster de alta disponibilidad. Toda la implementación está configurada mediante Azure Resource Manager.

En el siguiente diagrama se muestra la arquitectura de este escenario:

![Arquitectura detallada del conjunto de entrega de aplicaciones][2] 

> [!NOTE]
> Este ejemplo es solo una de las muchas configuraciones posibles de los servicios de equilibrio de carga que ofrece Azure. Traffic Manager, Front Door, Application Gateway y Load Balancer se pueden combinar de la forma que mejor se adapte a sus necesidades de equilibrio de carga. Por ejemplo, si la descarga de TLS/SSL o el procesamiento de Capa 7 no son necesarios, se puede usar Load Balancer en lugar de Application Gateway.


## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear una instancia de Front Door](quickstart-create-front-door.md).
- Más información acerca de cómo [funciona Front Door](front-door-routing-architecture.md).

<!--Image references-->
[1]: ./media/front-door-lb-with-azure-app-delivery-suite/application-delivery-figure1.png
[2]: ./media/front-door-lb-with-azure-app-delivery-suite/application-delivery-figure2.png
