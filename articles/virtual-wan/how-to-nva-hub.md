---
title: 'Azure Virtual WAN: Creación de una aplicación virtual de red (NVA) en el centro de conectividad'
description: En este tutorial, aprenderá a implementar una aplicación virtual de red en el centro de conectividad de Virtual WAN.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 07/14/2020
ms.author: cherylmc
Customer intent: As someone with a networking background, I want to create a Network Virtual Appliance (NVA) in my Virtual WAN hub.
ms.openlocfilehash: c5f120d6d9d80db4eaa9cf6f68e8dd27d45c8aea
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095283"
---
# <a name="how-to-create-a-network-virtual-appliance-in-an-azure-virtual-wan-hub-preview"></a>Creación de una aplicación virtual de red en un centro de conectividad de Azure Virtual WAN (versión preliminar)

En este tutorial se muestra cómo usar Virtual WAN para conectarse a recursos de Azure mediante una **aplicación virtual de red** (NVA) en Azure. Este tipo de conexión requiere un dispositivo VPN local que tenga una dirección IP pública asignada. Para más información sobre Virtual WAN, consulte la [Introducción a Virtual WAN](virtual-wan-about.md).

Los pasos que se describen en este artículo le ayudarán a crear una aplicación virtual de red **Barracuda CloudGen WAN** en el centro de conectividad de Virtual WAN. Para completar este ejercicio, debe tener un dispositivo local de la nube de Barracuda Cloud Premise (CPE) y una licencia de la aplicación Barracuda CloudGen WAN que implementará en el centro antes de empezar.

## <a name="before-you-begin"></a>Antes de empezar

Antes de comenzar con la configuración, compruebe que se cumplen los criterios siguientes:

* Obtenga una licencia de la puerta de enlace de Barracuda CloudGen WAN. Para más información sobre cómo hacerlo, consulte la [documentación de Barracuda CloudGen WAN](https://www.barracuda.com/products/cloudgenwan)

* Tiene una red virtual a la que quiere conectarse. Compruebe que ninguna de las subredes de sus redes locales se superpone a las redes virtuales a las que quiere conectarse. Para crear una red virtual en Azure Portal consulte este [Inicio rápido](../virtual-network/quick-create-portal.md).

* Su red virtual no tiene ninguna puerta de enlace de red virtual. Si la red virtual tiene alguna puerta de enlace (ya sea VPN o ExpressRoute), tiene que quitarla. Esta configuración requiere que las redes virtuales estén conectadas a la puerta de enlace del centro de conectividad de Virtual WAN.

* Obtenga un intervalo de direcciones IP para la región del concentrador. El centro de conectividad es una red virtual que Virtual WAN crea y usa. El intervalo de direcciones que especifique para el centro de conectividad no se puede superponer a ninguna de las redes virtuales existentes a las que ya esté conectado. Igualmente no se puede superponer a los intervalos de direcciones con las que esté conectadas en el entorno local. Si no está familiarizado con los intervalos de direcciones IP ubicados en la configuración de la red local, póngase de acuerdo con alguien que pueda proporcionarle estos detalles.

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="create-a-virtual-wan"></a><a name="openvwan"></a>Creación de una instancia de Virtual WAN

[!INCLUDE [Create virtual WAN](../../includes/virtual-wan-create-vwan-include.md)]

## <a name="create-a-hub"></a><a name="hub"></a>Creación de un centro de conectividad

Un centro de conectividad es una red virtual que puede contener puertas de enlace para las funcionalidades de sitio a sitio, ExpressRoute, de punto a sitio o aplicación virtual de red. Cuando se crea el concentrador, se le cobrará por él, aunque no esté asociado a ningún sitio. Si elige crear una puerta de enlace de VPN de sitio a sitio, la operación tardará 30 minutos. A diferencia de las funcionalidades de sitio a sitio, ExpressRoute o de punto a sitio, el centro de conectividad debe ser lo primero que debe crearse, antes de poder implementar una aplicación virtual de red en la red virtual del centro de conectividad.

1. Ubique la red WAN virtual que ha creado. En la página **Virtual WAN**, en la sección **Conectividad**, seleccione **Centros**.
1. En la página **Centros**, seleccione + Nuevo centro de conectividad para abrir la página **Crear centro de conectividad virtual**.

   :::image type="content" source="./media/how-to-nva-hub/vwan-hub.png" alt-text="Conceptos básicos":::
1. En la página **Crear centro de conectividad virtual**, en la pestaña **Aspectos básicos** rellene los siguientes campos:

   **Detalles del proyecto**

   * Región (anteriormente se conocía como ubicación)
   * Nombre
   * Espacio de direcciones privadas del centro de conectividad. El espacio de direcciones mínimo es/24 para crear un centro de conectividad, lo que implica que cualquier intervalo de/25 a/32 generará un error durante la creación. Azure Virtual WAN, un servicio administrado por Microsoft, crea las subredes adecuadas en el centro de conectividad virtual para las diferentes puertas de enlace y servicios (por ejemplo: aplicaciones virtuales de red, puertas de enlace de VPN, puertas de enlace de ExpressRoute, puertas de enlace de punto a sitio/VPN de usuario, firewall, enrutamiento, etc.). No es preciso que el usuario planee explícitamente el espacio de direcciones de subred para los servicios en el centro de conectividad virtual, ya que Microsoft lo hace como parte del servicio.
1. Seleccione **Revisar y crear** para validar.
1. Seleccione **Crear** para crear un centro de conectividad.

## <a name="create-the-network-virtual-appliance-in-the-hub"></a>Creación de la aplicación virtual de red en el centro de conectividad

En este paso, creará una aplicación virtual de red en el centro de conectividad. El procedimiento de cada NVA será diferente para cada producto del asociado de NVA. En este ejemplo, vamos a crear una puerta de enlace de Barracuda CloudGen WAN.

1. Busque el centro de conectividad de Virtual WAN que creó en el paso anterior y ábralo.

   :::image type="content" source="./media/how-to-nva-hub/nva-hub.png" alt-text="Centro de conectividad virtual":::
1. Busque el icono de las aplicaciones virtuales de red y seleccione el vínculo **Crear**.
1. En la hoja **Network Virtual Appliance** (Aplicación virtual de red), seleccione **Barracuda CloudGen WAN** y, después, seleccione el botón **Crear**.

   :::image type="content" source="./media/how-to-nva-hub/select-nva.png" alt-text="Seleccione NVA":::
1. Pasará a la oferta que hay en Azure Marketplace de la puerta de enlace de Barracuda CloudGen WAN. Lea los términos y, luego, seleccione el botón **Crear** cuando esté listo.

   :::image type="content" source="./media/how-to-nva-hub/barracuda-create-basics.png" alt-text="Aspectos básicos de la NVA de Barracuda":::
1. En la página **Aspectos básicos** tendrá que especificar la siguiente información:

   * **Suscripción**: elija la suscripción que usó para implementar Virtual WAN y el centro de conectividad.
   * **Grupo de recursos**: elija el mismo grupo de recursos que usó para implementar Virtual WAN y el centro de conectividad.
   * **Región**: elija la misma región en la que se encuentra el recurso del centro de conectividad virtual.
   * **Nombre de la aplicación**: Barracuda NextGen WAN es una aplicación administrada. Elija un nombre que facilite la identificación de este recurso, ya que así es como se le va a llamar cuando aparezca en su suscripción.
   * **Grupo de recursos administrados**: es el nombre del grupo de recursos administrados en el que Barracuda implementará los recursos que administren. El nombre se debe rellenar previamente en este caso.
1. Seleccione el botón **Siguiente: puerta de enlace de CloudGen WAN**.

   :::image type="content" source="./media/how-to-nva-hub/barracuda-cloudgen-wan.png" alt-text="Puerta de enlace de CloudGen WAN":::
1. Especifique aquí la siguiente información:

   * **Centro de conectividad de Virtual WAN**: el centro de conectividad de Virtual WAN en que desea implementar esta NVA.
   * **Unidades de infraestructura de NVA**: indique el número de unidades de infraestructura de NVA en las que desea implementar esta NVA. Elija la capacidad de ancho de banda agregado que desea proporcionar a través de todos los sitios de rama que se conectarán a este centro mediante esta NVA.
   * **Token**: Barracuda requiere que especifique aquí un token de autenticación para identificarse como un usuario registrado de este producto. Deberá obtener ese token de Barracuda.
1. Seleccione el botón **Revisar y crear** para continuar.
1. En esta página se le pedirá que acepte los términos del contrato de acceso de coadministrador. Esto es algo estándar en las aplicaciones administradas en las que el editor va a tener acceso a algunos recursos de esta implementación. Seleccione el cuadro **Acepto los términos y condiciones anteriores** y, después, seleccione **Crear**.

## <a name="connect-the-vnet-to-the-hub"></a><a name="vnet"></a>Conexión de la red virtual al centro de conectividad

[!INCLUDE [Connect](../../includes/virtual-wan-connect-vnet-hub-include.md)]

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre Virtual WAN, consulte la página [Introducción a Virtual WAN](virtual-wan-about.md).
* Para más información sobre las NVA de un centro de conectividad de Virtual WAN, consulte el artículo [Acerca de la aplicación virtual de red en el centro de conectividad de Virtual WAN (versión preliminar)](about-nva-hub.md).