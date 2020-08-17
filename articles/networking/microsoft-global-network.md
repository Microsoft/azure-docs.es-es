---
title: 'Red global de Microsoft: Azure'
description: Aprenda cómo Microsoft crea y opera una de las redes troncales más grandes del mundo y por qué es fundamental para ofrecer una excelente experiencia en la nube.
services: networking
documentationcenter: ''
author: KumudD
manager: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/13/2019
ms.author: kumud
ms.reviewer: ypitsch
ms.openlocfilehash: 9420d52cf7d88ff84fd23928f92cdc62e9627619
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87280909"
---
# <a name="microsoft-global-network"></a>Red global de Microsoft

Microsoft posee y opera una de las mayores redes troncales del mundo. Esta arquitectura global y sofisticada, que abarca más de 160 000 millas, conecta nuestros centros de datos y clientes. 
 
Cada día, clientes de todo el mundo se conectan y pasan billones de solicitudes a Microsoft Azure, Bing, Dynamics 365, Office 365, XBox y muchos otros. Con independencia del tipo, los clientes esperan la confiabilidad y capacidad de respuesta instantánea de nuestros servicios. 
 
La [red global de Microsoft](https://azure.microsoft.com/global-infrastructure/global-network/) (WAN) es una parte fundamental de la oferta de una experiencia de nube excelente. Nuestra red global, que conecta los [centros de datos](https://azure.microsoft.com/global-infrastructure/) de Microsoft en 54 regiones de Azure y una gran red de nodos perimetrales estratégicamente ubicados en todo el mundo, ofrece la disponibilidad, capacidad y flexibilidad para satisfacer cualquier demanda.

![Red global de Microsoft](./media/microsoft-global-network/microsoft-global-wan.png)
 
## <a name="get-the-premium-cloud-network"></a>Obtención de la red en la nube Premium
 
Optar por la [mejor experiencia posible](https://www.sdxcentral.com/articles/news/azure-tops-aws-gcp-in-cloud-performance-says-thousandeyes/2018/11/) es fácil cuando se usa la nube de Microsoft. Desde el momento en que el tráfico de cliente entra en nuestra red global a través de nuestros nodos perimetrales estratégicamente ubicados, los datos viajan a través de rutas optimizadas prácticamente a la velocidad de la luz. Esto garantiza una latencia óptima para obtener el mejor rendimiento. Estos nodos perimetrales, todos conectados entre sí a más de 3500 asociados de Internet únicos (homólogos) mediante miles de conexiones en más de 165 ubicaciones, proporcionan la base de nuestra estrategia de interconexión. 
 
Con independencia de que se conecte desde Londres a Tokio o desde Washington DC a Los Ángeles, el rendimiento de la red se cuantifica y se ve afectado por aspectos como la latencia, la vibración, la pérdida de paquetes y el rendimiento.  En Microsoft, se prefieren y usan interconexiones directas en lugar de conexiones de tránsito, lo que hace que el tráfico de respuesta sea simétrico y ayuda a reducir la duración y la complejidad de los saltos, las entidades de emparejamiento y las rutas de acceso. 

Por ejemplo, si un usuario en Londres intenta acceder a un servicio de Tokio, el tráfico de Internet entra en uno de nuestros servidores perimetrales en Londres, recorre la WAN de Microsoft a través de Francia, nuestras rutas de acceso de Trans-Arabia entre Europa e India, y después llega a Japón, donde se hospeda el servicio. El tráfico de respuesta es simétrico. Esto se conoce a veces como [enrutamiento de patata fría](https://en.wikipedia.org/wiki/Hot-potato_and_cold-potato_routing), lo que significa que el tráfico permanece en la red de Microsoft el máximo tiempo posible antes de entregarlo.  
  
Por tanto, ¿se refiere a todo el tráfico cuando se usan los servicios de Microsoft? Sí, todo el tráfico entre centros datos, dentro de Microsoft Azure o entre los servicios de Microsoft, como máquinas virtuales, Microsoft 365, XBox, bases de datos SQL, Storage y redes virtuales se enruta dentro de nuestra red mundial y nunca sobre Internet, para garantizar un rendimiento y una integridad óptimos.  
 
Las grandes inversiones en capacidad de fibra y diversidad a través de rutas metropolitanas, terrestres y submarinas son fundamentales para poder mantener un alto nivel de servicio coherente al tiempo que se impulsa el crecimiento extremo de nuestros servicios en la nube y en línea. Las adiciones recientes a la red son nuestro cable submarino [MAREA](https://www.submarinecablemap.com/#/submarine-cable/marea), el primer sistema de líneas abiertas (OLS) submarino del sector (entre Bilbao, España y Virginia Beach, Virginia, EE. UU.), así como el [AEC](https://www.submarinecablemap.com/#/submarine-cable/aeconnect-1) entre Nueva York, Estados Unidos y Dublín, Irlanda, y [New Cross Pacific (NCP)](https://www.submarinecablemap.com/#/submarine-cable/new-cross-pacific-ncp-cable-system) entre Tokio, Japón y Portland, Oregon, Estados Unidos. 
 

## <a name="our-network-is-your-network"></a>Nuestra red es la suya

Para garantizar un rendimiento óptimo en todo momento, hemos incluido en la red dos décadas de experiencia junto con grandes inversiones. Las empresas pueden aprovechar al máximo nuestros recursos de red sobre los que crear arquitecturas de superposición avanzadas. 
 
Microsoft Azure ofrece la mayor cartera de servicios y funciones, lo que permite a los clientes crear, expandir y cumplir los requisitos de red en cualquier lugar, de forma rápida y sencilla. Nuestra familia de servicios de conectividad abarca el emparejamiento de redes virtuales entre regiones, arquitecturas híbridas y de punto a sitio y de sitio a sitio en la nube, así como escenarios de tránsito IP globales.  Para las empresas que buscan conectar su propio centro de datos o red a Azure, o bien para los clientes con necesidades de tránsito o ingesta de datos masivos, [ExpressRoute](../expressroute/expressroute-introduction.md) y [ExpressRoute Direct](../expressroute/expressroute-erdirect-about.md) proporcionan opciones de hasta 100 Gbps de ancho de banda, directamente en la red global de Microsoft en ubicaciones de emparejamiento de todo el mundo.  
 
[Global Reach de ExpressRoute](../expressroute/expressroute-global-reach.md) está diseñado para complementar la implementación de WAN de su proveedor de servicios y conectar los sitios locales en todo el mundo. Si ejecuta una operación global, puede usar Global Reach de ExpressRoute junto con sus proveedores de servicios locales preferidos para conectar todos los sitios globales mediante la red global de Microsoft. La expansión de la nueva red en la nube (WAN) para abarcar grandes números de sitios de sucursales se puede realizar a través de Azure Virtual WAN, que ofrece la posibilidad de conectar las sucursales sin problemas a la red global de Microsoft con dispositivos SDWAN y VPN (es decir, Equipo local del cliente o CPE) con facilidad de uso integrada y administración automatizada de la conectividad y la configuración. 
 
El [emparejamiento de VNet global](../virtual-network/virtual-network-peering-overview.md) permite a los clientes conectar sin problemas dos o más redes virtuales de Azure entre regiones. Una vez emparejadas, las redes virtuales aparecen como una sola. El tráfico entre las máquinas virtuales de las redes virtuales emparejadas se enruta a través de la infraestructura de la red troncal de Microsoft, de forma muy parecida al enrutamiento del tráfico entre máquinas virtuales de la misma red virtual únicamente a través de direcciones IP privadas. 
 

## <a name="well-managed-using-software-defined-innovation"></a>Administración correcta con innovación definida por software

Con una de las nubes líderes a nivel mundial, Microsoft ha obtenido mucha información y experiencia sobre la creación y administración de una infraestructura global de alto rendimiento.  
 
Seguimos un conjunto sólido de principios operativos: 
 
- Uso del mejor hardware de conmutación entre los distintos niveles de la red.  
- Implementación de nuevas características sin impacto para los usuarios finales.  
- Implementación de actualizaciones de forma segura y confiable en toda la flota, tan rápido como sea posible. Horas en lugar de semanas.  
- Uso de telemetría profunda de escala de nube y mitigación de errores totalmente automatizada.  
- Uso de tecnología de red unificada y definida por software para controlar todos los elementos de hardware de la red.  Eliminación de la duplicación y reducción de los errores. 
 
Estos principios se aplican a todas las capas de la red: desde la interfaz de red del host, la plataforma de conmutación, las funciones del centro de datos como los equilibradores de carga, hasta llegar a la WAN con nuestra plataforma de ingeniería de tráfico y nuestras redes ópticas.  
 
El crecimiento exponencial de Azure y su red ha alcanzado un punto donde finalmente hemos asumido que ya no es posible confiar en la intuición humana para administrar las operaciones de red globales. Para satisfacer la necesidad de validar los cambios en la red a corto, medio y largo plazo, hemos desarrollado una plataforma que imite de forma sintética nuestra red de producción. La capacidad de crear entornos reflejados y ejecutar millones de simulaciones nos permite probar los cambios de hardware y software y su impacto, antes de confirmarlos en nuestra plataforma y red de producción. 

## <a name="next-steps"></a>Pasos siguientes
- [Más información sobre los servicios de red proporcionados en Azure](https://azure.microsoft.com/product-categories/networking/)
