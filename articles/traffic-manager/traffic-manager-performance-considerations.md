---
title: Consideraciones de rendimiento de Azure Traffic Manager | Microsoft Docs
description: Descripción del rendimiento en el Administrador de tráfico y cómo probar el rendimiento de su sitio web al usar el Administrador de tráfico
services: traffic-manager
documentationcenter: ''
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/16/2017
ms.author: duau
ms.openlocfilehash: b72126933c5454b8fabe19dda08ad5eb66edd1ba
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89392677"
---
# <a name="performance-considerations-for-traffic-manager"></a>Consideraciones de rendimiento sobre el Administrador de tráfico

En esta página se explican las consideraciones de rendimiento relacionadas con el uso de Administrador de tráfico. Considere el caso siguiente:

Tiene instancias de su sitio web en las regiones Oeste de EE. UU. y Este de Asia. Una de ellas no ha superado la comprobación de estado para el sondeo de Traffic Manager. El tráfico de aplicaciones se dirige a la región en buen estado. Se esperaba esta conmutación por error, pero el rendimiento puede ser un problema en función de la latencia del tráfico que ahora se desplaza a una región lejana.

## <a name="performance-considerations-for-traffic-manager"></a>Consideraciones de rendimiento sobre el Administrador de tráfico

El único impacto en el rendimiento que puede tener Traffic Manager sobre su sitio web es en la búsqueda de DNS inicial. Una solicitud de DNS del nombre de su perfil de Traffic Manager se procesa mediante el servidor de raíz DNS de Microsoft que hospeda la zona trafficmanager.net. Traffic Manager rellena y actualiza con regularidad los servidores de raíz DNS de Microsoft en función de la directiva de Traffic Manager y los resultados del sondeo. Por tanto, incluso durante la búsqueda de DNS inicial, no se envía ninguna consulta de DNS a Traffic Manager.

Traffic Manager se compone de varios elementos: servidores de nombres DNS, un servicio de API, la capa de almacenamiento y un servicio de supervisión de puntos de conexión. Si se produce un error en un componente del servicio Traffic Manager, no surte ningún efecto en el nombre DNS asociado a su perfil de Traffic Manager. No se modificarán los registros en los servidores DNS de Microsoft. Sin embargo, la supervisión de puntos de conexión y la actualización de DNS no se llevan a cabo. Por lo tanto, Traffic Manager no es capaz de actualizar DNS para que señale a su sitio de conmutación por error cuando el sitio principal deja de funcionar.

La resolución de nombres DNS es rápida y los resultados se almacenan en la memoria caché. La velocidad de la búsqueda de DNS inicial depende de los servidores DNS que el cliente use para la resolución de nombres. Por lo general, un cliente puede completar una búsqueda de DNS en aproximadamente 50 ms. Los resultados de la búsqueda se almacenan en la memoria caché durante el período de vida (TTL) de DNS. El TTL predeterminado para Traffic Manager es 300 segundos.

El tráfico NO fluye a través del Administrador de tráfico. Una vez completada la búsqueda de DNS, el cliente tiene una dirección IP para una instancia de su sitio web. El cliente se conecta directamente a esa dirección y no pasa a través de Traffic Manager. La directiva de Traffic Manager que elija no influye en el rendimiento de DNS. Sin embargo, un método de enrutamiento de rendimiento puede repercutir negativamente en el uso de la aplicación. Por ejemplo, si la directiva redirige el tráfico de Norteamérica a una instancia hospedada en Asia, la latencia de red para esas sesiones podría causar problemas de rendimiento.

## <a name="measuring-traffic-manager-performance"></a>Medición del rendimiento de Traffic Manager

Existen varios sitios web que sirven para comprender el rendimiento y el comportamiento de un perfil de Traffic Manager. Muchos de ellos son gratuitos, pero pueden tener limitaciones. Algunos sitios ofrecen supervisión mejorada y elaboración de informes por una cuota.

Las herramientas de estos sitios miden las latencias de DNS y muestran las direcciones IP resueltas para ubicaciones de cliente en el mundo. La mayoría de estas herramientas no almacenan en caché los resultados de DNS. Por lo tanto, muestran la búsqueda de DNS completa cada vez que se ejecuta una prueba. Cuando hace la prueba desde su propio cliente, solo experimenta el rendimiento de la búsqueda de DNS completa una vez durante el TTL.

## <a name="sample-tools-to-measure-dns-performance"></a>Herramientas de muestra para medir el rendimiento de DNS

* [SolveDNS](https://www.solvedns.com/dns-comparison/)

    SolveDNS ofrece muchas herramientas de rendimiento. La herramienta de comparación de DNS puede mostrarle cuánto tiempo se tarda en resolver el nombre DNS y lo compara con otros proveedores de servicios DNS.

* [WebSitePulse](https://www.websitepulse.com/help/tools.php)

    Una de las herramientas más sencillas es WebSitePulse. Escriba la dirección URL para ver el tiempo de resolución de DNS, el primer byte, el último byte y otras estadísticas de rendimiento. Puede elegir entre tres ubicaciones de prueba diferentes. En este ejemplo verá que la primera ejecución muestra que la búsqueda de DNS tarda 0,204 s.

    ![pulse1](./media/traffic-manager-performance-considerations/traffic-manager-web-site-pulse.png)

    Como los resultados se almacenan en caché, en la segunda prueba para el mismo punto de conexión de Traffic Manager, la búsqueda de DNS tarda 0,002 s.

    ![pulse2](./media/traffic-manager-performance-considerations/traffic-manager-web-site-pulse2.png)

* [CA App Synthetic Monitor](https://asm.ca.com/en/checkit.php)

    Antes conocido como la herramienta Watch-mouse Check Website, este sitio muestra el tiempo de resolución de DNS desde varias regiones geográficas simultáneamente. Escriba la dirección URL para ver el tiempo de resolución de DNS, el tiempo de conexión y la velocidad desde varias ubicaciones geográficas. Use esta prueba para ver qué servicio hospedado se devuelve para diferentes ubicaciones del mundo.

    ![pulse1](./media/traffic-manager-performance-considerations/traffic-manager-web-site-watchmouse.png)

* [Pingdom](https://tools.pingdom.com/)

    Esta herramienta proporciona estadísticas de rendimiento para cada elemento de una página web. La pestaña de análisis de página muestra el porcentaje de tiempo invertido en la búsqueda de DNS.

* [What's My DNS?](https://www.whatsmydns.net/)

    Este sitio realiza una búsqueda de DNS desde 20 ubicaciones distintas y muestra los resultados en un mapa.

* [Dig Web Interface](https://www.digwebinterface.com)

    Este sitio muestra información de DNS más detallada, incluidos los registros A y CNAME. Asegúrese de activar las opciones "Colorize output" (Colorear salida) y "Stats" (Estadísticas), y de seleccionar todos los servidores de nombres.

## <a name="next-steps"></a>Pasos siguientes

[Información acerca de los métodos de enrutamiento del tráfico del Administrador de tráfico](traffic-manager-routing-methods.md)

[Pruebe la configuración del Administrador de tráfico](traffic-manager-testing-settings.md)

[Operaciones del Administrador de tráfico (referencia de la API de REST)](https://go.microsoft.com/fwlink/?LinkId=313584)

[Cmdlets del Administrador de tráfico de Azure](https://docs.microsoft.com/powershell/module/az.trafficmanager)

