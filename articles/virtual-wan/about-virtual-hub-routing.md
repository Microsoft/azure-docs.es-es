---
title: Acerca del enrutamiento de centros virtuales
titleSuffix: Azure Virtual WAN
description: En este artículo se describe el enrutamiento de centros virtuales.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 06/29/2020
ms.author: cherylmc
ms.custom: fasttrack-edit
ms.openlocfilehash: f29a7e48fc1872f83b5a6ce127f38c1a559b2691
ms.sourcegitcommit: e2b36c60a53904ecf3b99b3f1d36be00fbde24fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88762324"
---
# <a name="about-virtual-hub-routing"></a>Acerca del enrutamiento de centros virtuales

Las funcionalidades de enrutamiento de centros virtuales se proporcionan mediante un enrutador que administra todo el enrutamiento entre puertas de enlace mediante el Protocolo de puerta de enlace de borde (BGP). Un centro virtual puede contener varias puertas de enlace, como una puerta de enlace de VPN de sitio a sitio, una puerta de enlace de ExpressRoute, una puerta de enlace de punto a sitio y Azure Firewall. Este enrutador también proporciona conectividad de tránsito entre las redes virtuales que se conectan a un centro virtual y pueden admitir hasta un rendimiento agregado de 50 Gbps. Estas funcionalidades de enrutamiento se aplican a los clientes de Virtual WAN estándar. 

Para configurar el enrutamiento, vea [Configuración del enrutamiento de centro virtual](how-to-virtual-hub-routing.md).

## <a name="routing-concepts"></a><a name="concepts"></a>Conceptos de enrutamiento

En las secciones siguientes se describen los conceptos clave del enrutamiento de centros virtuales.

### <a name="hub-route-table"></a><a name="hub-route"></a>Tabla de rutas del centro de conectividad

Una tabla de rutas de centros virtuales puede contener una o varias rutas. Una ruta incluye su nombre, una etiqueta, un tipo de destino, una lista de prefijos de destino y la información de próximo salto de un paquete que se va a enrutar. Una **conexión** normalmente tendrá una configuración de enrutamiento que se asocia o se propaga a una tabla de rutas.

### <a name="connection"></a><a name="connection"></a>Conexión

Las conexiones son recursos de Resource Manager que tienen una configuración de enrutamiento. Los cuatro tipos de conexiones son:

* **Conexión VPN**: conecta un sitio VPN a una puerta de enlace de VPN de centro virtual.
* **Conexión de ExpressRoute**: conecta un circuito de ExpressRoute a una puerta de enlace de ExpressRoute de centro virtual.
* **Conexión con la configuración de punto a sitio**: conecta una configuración de una VPN de usuario (punto a sitio) a una puerta de enlace de VPN de centro virtual (punto a sitio).
* **Conexión de red virtual del centro**: conecta redes virtuales a un centro virtual.

Puede configurar el enrutamiento de una conexión de red virtual durante la instalación. De forma predeterminada, todas las conexiones se asocian y propagan a la tabla de rutas predeterminada.

### <a name="association"></a><a name="association"></a>Asociación

Cada conexión está asociada a una tabla de rutas. La asociación de una conexión a una tabla de rutas permite que el tráfico se envíe al destino indicado como rutas en la tabla de rutas. La configuración de enrutamiento de la conexión mostrará la tabla de rutas asociada.  Se pueden asociar varias conexiones a la misma tabla de rutas. Todas las conexiones VPN, ExpressRoute y VPN de usuario están asociadas a la misma tabla de rutas (predeterminada).

De forma predeterminada, todas las conexiones están asociadas a una **tabla de rutas predeterminada** en un centro virtual. Cada centro virtual tiene su propia tabla de rutas predeterminada, que se puede editar para agregar una o más rutas estáticas. Las rutas agregadas estáticamente tienen prioridad sobre las rutas aprendidas de forma dinámica para los mismos prefijos.

:::image type="content" source="./media/about-virtual-hub-routing/concepts-association.png" alt-text="Asociación":::

### <a name="propagation"></a><a name="propagation"></a>Propagación

Las conexiones propagan dinámicamente las rutas a una tabla de rutas. Con una conexión VPN, una conexión ExpressRoute o una conexión de configuración de punto a sitio, las rutas se propagan desde el centro virtual al enrutador local mediante BGP. Las rutas se pueden propagar a una o varias tablas de rutas.

También está disponible una **tabla de rutas None** para cada centro virtual. La propagación a la tabla de rutas None significa que no se requiere la propagación de ninguna ruta desde la conexión. Las conexiones VPN, ExpressRoute y VPN de usuario propagan rutas al mismo conjunto de tablas de rutas.

:::image type="content" source="./media/about-virtual-hub-routing/concepts-propagation.png" alt-text="Propagación":::

### <a name="labels"></a><a name="static"></a>Etiquetas
Las etiquetas proporcionan un mecanismo para agrupar las tablas de rutas de forma lógica. Esto es especialmente útil durante la propagación de rutas desde las conexiones a varias tablas de rutas. Por ejemplo, la tabla de rutas predeterminada tiene una etiqueta integrada denominada "Default". Cuando los usuarios propagan las rutas de conexión a la etiqueta "Default", se aplica automáticamente a todas las tablas de rutas predeterminadas que se encuentran en todos los concentradores de Virtual WAN. 

### <a name="configuring-static-routes-in-a-virtual-network-connection"></a><a name="static"></a>Configuración de rutas estáticas en una conexión de red virtual

La configuración de rutas estáticas proporciona un mecanismo para dirigir el tráfico a través de una dirección IP de próximo salto, que podría ser de una aplicación virtual de red (NVA) aprovisionada en una red virtual radial conectada a un centro virtual. La ruta estática se compone de un nombre de ruta, una lista de prefijos de destino y una dirección IP de próximo salto.

## <a name="reset-hub"></a><a name="route"></a>Restablecimiento del centro
Esta opción solo está disponible en Azure Portal y proporciona al usuario una forma de devolver todos los recursos con errores, como las tablas de enrutamiento, el enrutador del centro o el propio recurso del centro virtual a su estado de aprovisionamiento correcto. Esta es una opción adicional que el usuario debe tener en cuenta antes de ponerse en contacto con Microsoft para recibir soporte técnico. Esta operación no restablece ninguna de las puertas de enlace de ningún centro virtual. 

## <a name="route-tables-in-basic-and-standard-virtual-wans-prior-to-the-feature-set-of-association-and-propagation"></a><a name="route"></a>Tablas de ruta en las versiones Básico y Estándar de Virtual WAN antes del conjunto de características de asociación y propagación

Las tablas de rutas ahora tienen características para la asociación y propagación. Una tabla de rutas preexistente es una tabla de enrutamiento que no tiene estas características. Si tiene rutas preexistentes en el enrutamiento del centro y desea usar las nuevas funcionalidades, tenga en cuenta lo siguiente:

* **Clientes de la versión Estándar de Virtual WAN con rutas preexistentes en un centro virtual**:

Si tiene rutas preexistentes en la sección Enrutamiento del centro de Azure Portal, primero debe eliminarlas y, luego, intentar crear tablas de rutas nuevas (disponibles en la sección Tablas de rutas del centro de Azure Portal).

* **Clientes de la versión Básica de Virtual WAN con rutas preexistentes en un centro virtual**: Si tiene rutas preexistentes en la sección Enrutamiento del centro de Azure Portal, primero debe eliminarlas y, después, **actualizar** la versión Básica de Virtual WAN a la versión Estándar. Consulte [Actualización de una virtual WAN de Básica a Estándar](upgrade-virtual-wan.md).

## <a name="virtual-wan-routing-considerations"></a><a name="considerations"></a>Consideraciones sobre el enrutamiento Virtual WAN

Tenga en cuenta lo siguiente al configurar el enrutamiento Virtual WAN:

* Todas las conexiones de rama (de punto a sitio, de sitio a sitio y de ExpressRoute) deben estar asociadas a la tabla de rutas predeterminada. De este modo, todas las ramas aprenderán los mismos prefijos.
* Todas las conexiones de rama deben propagar sus rutas al mismo conjunto de tablas de rutas. Por ejemplo, si decide que las rama deben propagarse a la tabla de rutas predeterminada, esta configuración debe ser coherente en todas las ramas. Como resultado, todas las conexiones asociadas a la tabla de rutas predeterminada podrán llegar a todas las ramas.
* De momento no se admite el flujo de rama a rama a través de Azure Firewall.
* Al usar Azure Firewall en varias regiones, todas las redes virtuales de radios deben estar asociadas a la misma tabla de rutas. Por ejemplo, no es posible tener un subconjunto de redes virtuales que pasen por Azure Firewall mientras otras redes virtuales se saltan Azure Firewall en el mismo centro virtual.
* Se puede configurar una única dirección IP de próximo salto por cada conexión de red virtual.
* El centro de conectividad virtual no admite la ruta estática para 0.0.0.0/0 ni la conexión de red virtual del próximo salto (o una IP de un dispositivo en la conexión de red virtual)

## <a name="next-steps"></a>Pasos siguientes

Para configurar el enrutamiento, vea [Configuración del enrutamiento de centro virtual](how-to-virtual-hub-routing.md).

Para más información sobre Virtual WAN, consulte las [preguntas más frecuentes](virtual-wan-faq.md).
