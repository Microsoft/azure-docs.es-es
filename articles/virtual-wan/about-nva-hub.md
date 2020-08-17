---
title: 'Azure Virtual WAN: Acerca de la aplicación virtual de red en el centro de conectividad'
description: En este artículo, obtendrá información sobre las aplicaciones virtuales de red en el centro de conectividad de Virtual WAN.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 07/06/2020
ms.author: scottnap
Customer intent: As someone with a networking background, I want to learn about Network Virtual Appliances in the Virtual WAN hub.
ms.openlocfilehash: ad7c7fb5111ce700a5725336b7c9db788e178c4c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095289"
---
# <a name="about-network-virtual-appliance-in-an-azure-virtual-wan-hub-preview"></a>Acerca de una aplicación virtual de red en un centro de conectividad de Azure Virtual WAN (versión preliminar)

Azure Virtual WAN ha trabajado con asociados de red para crear una automatización que facilite la conexión de su equipo local del cliente (CPE) a una puerta de enlace de VPN de Azure que se encuentre en el centro de conectividad virtual. Azure trabaja con algunos asociados de red seleccionados para que los clientes implementen una aplicación virtual de red (NVA) de terceros directamente en el centro de conectividad virtual. Esto permite a los clientes conectar el CPE de su rama a la NVA de la misma marca en el centro de conectividad virtual para poder sacar provecho de todas las funcionalidades propias de SD-WAN.

Barracuda Networks es el primer asociado que proporciona una oferta de NVA que se puede implementar directamente en el centro de conectividad de Virtual WAN con su producto [Barracuda CloudGen WAN](https://www.barracuda.com/products/cloudgenwan). Azure está trabajando con más asociados, por lo que lo normal es que aparezcan otras ofertas.

> [!NOTE]
> Las únicas ofertas que se pueden implementar en el centro de Virtual WAN son las que están disponibles para ello. No se pueden implementar en una red virtual arbitraria de Azure.

## <a name="how-does-it-work"></a><a name="how"></a>¿Cómo funciona?

Las NVA que están disponibles para implementarse directamente en el centro de Azure Virtual WAN se han diseñado específicamente para su uso en el centro de conectividad virtual. La oferta de NVA se publica en Azure Marketplace como una aplicación administrada y los clientes pueden implementar la oferta directamente desde Azure Marketplace, o bien desde el centro de conectividad virtual a través de Azure Portal.

:::image type="content" source="./media/about-nva-hub/high-level-process.png" alt-text="Introducción al proceso":::

La oferta de NVA de cada asociado tendrá una experiencia y funcionalidad ligeramente diferente en función de sus requisitos de implementación. Sin embargo, hay algunas cosas que son comunes a las ofertas de WAN de todos los asociados en el centro de Virtual WAN.

* Una experiencia de aplicación administrada ofrecida a través de Azure Marketplace.
* Capacidad y facturación basados en unidad de la infraestructura de NVA.
* Las métricas de estado se muestran a través de Azure Monitor.

### <a name="managed-application"></a><a name="managed"></a>Aplicación administrada

Todas las ofertas de NVA que están disponibles para su implementación en el centro de Virtual WAN tendrán una **aplicación administrada** disponible en Azure Marketplace. Las aplicaciones administradas permiten a los asociados realizar las siguientes operaciones:

* Crear una experiencia de implementación personalizada para su NVA.
* Proporcionar una plantilla de Resource Manager especializada que les permita crear la NVA directamente en el centro de conectividad de Virtual WAN.
* Facturar los costos de las licencias de software directamente o a través de Azure Marketplace.
* Exponer las propiedades personalizadas y los medidores de recursos.

Los asociados de las NVA pueden crear diferentes recursos en función de la implementación de la aplicación, las licencias de configuración y las necesidades de administración. Cuando un cliente crea una NVA en el centro de conectividad de Virtual WAN, como en todas las aplicaciones administradas, se crearán dos grupos de recursos en su suscripción.

* **Grupo de recursos del cliente**: incluirá un marcador de posición de la aplicación para la aplicación administrada. Los asociados pueden usarlo para exponer las propiedades personalizadas que elijan.
* **Grupo de recursos administrados**: los clientes no pueden configurar ni cambiar los recursos de este grupo directamente, ya que es el editor de la aplicación administrada quien lo controla. Este grupo de recursos contendrá el recurso **NetworkVirtualAppliances**.

:::image type="content" source="./media/about-nva-hub/managed-app.png" alt-text="Grupos de recursos de la aplicación administrada":::

### <a name="nva-infrastructure-units"></a><a name="units"></a>Unidades de infraestructura de NVA

Al crear una NVA en el centro de conectividad de Virtual WAN, es preciso elegir el número de unidades de infraestructura de NVA con el que desea implementarla. Una **unidad de infraestructura de NVA** es una unidad de capacidad de ancho de banda agregado para una NVA del centro de conectividad de Virtual WAN. Una **unidad de infraestructura de NVA** es similar a una [unidad de escalado](pricing-concepts.md#scale-unit) de VPN en lo relativo a la capacidad y el dimensionamiento.

* Una unidad de infraestructura de NVA representa 500 Mbps de ancho de banda agregado para todas las conexiones del sitio de la rama que llegan a esta NVA, y ello tiene un costo de 0,25 USD por hora.
* Azure admite entre 1 y 80 unidades de infraestructura de NVA para la implementación de un centro de conectividad virtual de NVA determinado.
* Cada asociado puede ofrecer diferentes paquetes de unidades de infraestructura de NVA, los cuales constituyen un subconjunto de todas las configuraciones de las unidades de infraestructura de NVA compatibles.

De forma similar a las unidades de escalado de VPN, si elige que *1 unidad de escalado = 500 Mbps*, esto implica que se crearán dos instancias de redundancia, cada una con un rendimiento máximo de 500 Mbps. Por ejemplo, si tuviera cinco ramas, cada una con 10 Mbps en la rama, necesitará un agregado de 50 Mbps en el extremo principal. El planeamiento de la capacidad agregada de la NVA debe realizarse después de evaluar la capacidad necesaria para admitir el número de ramas en el centro de conectividad.

## <a name="network-virtual-appliance-configuration-process"></a><a name="configuration"></a>Proceso de configuración de la aplicación virtual de red

Los asociados se han esforzado para ofrecer una experiencia que configura la NVA de forma automática como parte del proceso de implementación. Una vez que la NVA se ha aprovisionado en el centro de conectividad virtual, cualquier otra configuración que necesite la NVA se debe realizar a través del portal de asociados o de la aplicación de administración de NVA. El acceso directo a la NVA no está disponible.

## <a name="site-and-connection-resources-with-nvas"></a><a name="resources"></a>Recursos de sitio y de conexión con NVA

A diferencia de las configuraciones de Azure VPN Gateway, no es preciso crear recursos de **sitio**, recursos de **conexión de sitio a sitio** o recursos de **conexión de punto a sitio** para conectar los sitios de la rama con su NVA en el centro de conectividad de Virtual WAN. Todo esto se administra a través del asociado de NVA.

Necesita crear conexiones del centro de conectividad a la red virtual para conectar el centro de conectividad de Virtual WAN a sus redes virtuales de Azure.

## <a name="supported-regions"></a><a name="regions"></a>Regiones admitidas

NVA en el centro de conectividad virtual está disponible en versión preliminar en las siguientes regiones:

|Región geopolítica | Regiones de Azure|
|---|---|
| Norteamérica| Oeste de EE. UU., Centro-sur de EE. UU. y Este de EE. UU. 2   |
| Sudamérica | Sur de Brasil |
| Europa | Oeste de Europa, Sur de Reino Unido|
|  Oriente Medio | Norte de Emiratos Árabes Unidos |
| Asia | Japón Oriental |
| Australia | Este de Australia |

## <a name="faq"></a>Preguntas más frecuentes

### <a name="i-am-a-network-appliance-partner-and-want-to-get-our-nva-in-the-hub--can-i-join-this-partner-program"></a>Soy asociado de dispositivos de red y quiero que nuestra NVA esté en el centro de conectividad.  ¿Puedo participar en este programa para asociados?

Desafortunadamente, no tenemos capacidad para incorporar nuevas ofertas para asociados en este momento. Vuelva a realizar la consulta en noviembre.

### <a name="can-i-deploy-any-nva-from-azure-marketplace-into-the-virtual-wan-hub"></a>¿Puedo implementar cualquier NVA desde Azure Marketplace en el centro de conectividad de Virtual WAN?

No. Actualmente, [Barracuda CloudGen WAN](https://aka.ms/BarracudaMarketPlaceOffer) es el único producto disponible para implementarse en el centro de conectividad de Virtual WAN.

### <a name="what-is-the-cost-of-the-nva"></a>¿Cuál es el costo de la NVA?

Debe comprar en Barracuda una licencia para la NVA Barracuda CloudGen WAN. Para más información sobre las licencias, consulte la [página de CloudGen WAN de Barracuda](https://www.barracuda.com/products/cloudgenwan). Además, Microsoft también puede cobrarle por las unidades de la infraestructura de la NVA que consuma, así como cualquier otro recurso que use. Para más información, consulte [Conceptos de precios](pricing-concepts.md).

### <a name="can-i-deploy-an-nva-to-a-basic-hub"></a>¿Se puede implementar una NVA en un centro de conectividad Básico?

No. Si desea implementar una NVA, debe usar un centro de conectividad Estándar.

### <a name="can-i-deploy-an-nva-into-a-secure-hub"></a>¿Se puede implementar una NVA en un centro de conectividad Seguro?

Sí. Barracuda CloudGen WAN se puede implementar en un centro de conectividad con Azure Firewall.

### <a name="can-i-connect-any-cpe-device-in-my-branch-office-to-barracuda-cloudgen-wan-nva-in-the-hub"></a>¿Se pueden conectar todos los dispositivos CPE de una sucursal a la NVA Barracuda CloudGen WAN del centro de conectividad?

No. Barracuda CloudGen WAN solo es compatible con dispositivos CPE de Barracuda. Para más información acerca de los requisitos de CloudGen WAN, consulte la [página de CloudGen WAN de Barracuda](https://www.barracuda.com/products/cloudgenwan).

### <a name="what-routing-scenarios-are-supported-with-nva-in-the-hub"></a>¿Qué escenarios de enrutamiento son compatibles con NVA en el centro de conectividad?

Todos los escenarios de enrutamiento compatibles con Virtual WAN son compatibles con las NVA del centro de conectividad.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Virtual WAN, consulte el artículo [Introducción a Virtual WAN](virtual-wan-about.md).