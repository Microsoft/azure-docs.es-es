---
title: 'ExpressRoute: Conexión de una red virtual con un circuito: Portal de Azure'
description: Aprenda a crear una conexión para vincular una red virtual con un circuito de Azure ExpressRoute mediante Azure Portal.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 09/17/2019
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: 56508503c199b1f822ce8f181689a236f3a0af18
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89395832"
---
# <a name="connect-a-virtual-network-to-an-expressroute-circuit-using-the-portal"></a>Conexión de una red virtual con un circuito de ExpressRoute mediante el portal
> [!div class="op_single_selector"]
> * [Azure Portal](expressroute-howto-linkvnet-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-linkvnet-arm.md)
> * [CLI de Azure](howto-linkvnet-cli.md)
> * [Vídeo: Azure Portal](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-connection-between-your-vpn-gateway-and-expressroute-circuit)
> * [PowerShell (clásico)](expressroute-howto-linkvnet-classic.md)
> 

Este artículo le ayuda a crear una conexión para vincular una red virtual con un circuito de Azure ExpressRoute mediante Azure Portal. Las redes virtuales que se conectan al circuito de Azure ExpressRoute pueden estar en la misma suscripción o formar parte de una suscripción diferente.

## <a name="before-you-begin"></a>Antes de empezar

* Revise los [requisitos previos](expressroute-prerequisites.md), los [requisitos de enrutamiento](expressroute-routing.md) y los [flujos de trabajo](expressroute-workflows.md) antes de comenzar la configuración.

* Tiene que tener un circuito ExpressRoute activo.
  * Siga las instrucciones para [crear un circuito ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md) y habilite el circuito mediante el proveedor de conectividad.
  * Asegúrese de que dispone de un emparejamiento privado de Azure configurado para el circuito. Consulte el artículo [Creación y modificación del emparejamiento de un circuito ExpressRoute](expressroute-howto-routing-portal-resource-manager.md) para conocer las instrucciones de emparejamiento y enrutamiento.
  * Asegúrese de que el emparejamiento privado de Azure está configurado y el emparejamiento BGP entre la red y Microsoft está activo para habilitar la conectividad de extremo a extremo.
  * Asegúrese de que ha creado y aprovisionado totalmente una red virtual y una puerta de enlace de red virtual. Siga las instrucciones para [crear una puerta de enlace de red virtual en ExpressRoute](expressroute-howto-add-gateway-resource-manager.md). Una puerta de enlace de red virtual para ExpressRoute usa el GatewayType "ExpressRoute", no VPN.

* Es posible vincular hasta 10 redes virtuales a un circuito ExpressRoute estándar. Todas las redes virtuales deben pertenecer a la misma región geopolítica al utilizar un circuito de ExpressRoute estándar.

* Una red virtual solo se puede vincular a cuatro circuitos ExpressRoute como máximo. Use el procedimiento siguiente para crear un nuevo objeto de conexión para cada circuito ExpressRoute al que quiere conectarse. Los circuitos ExpressRoute pueden estar en la misma suscripción, en suscripciones diferentes o en una combinación de ambas.

* Puede vincular una red virtual fuera de la región geopolítica del circuito de ExpressRoute, o bien conectar un mayor número de redes virtuales a este si habilitó el complemento premium de ExpressRoute. Consulte las [preguntas más frecuentes](expressroute-faqs.md) para obtener más detalles sobre el complemento premium.

* Puede [ver un vídeo](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-connection-between-your-vpn-gateway-and-expressroute-circuit) antes de comenzar para entender mejor los pasos.

## <a name="connect-a-vnet-to-a-circuit---same-subscription"></a>Conexión de una red virtual con un circuito: misma suscripción

> [!NOTE]
> La información de la configuración de BGP no aparecerá si el proveedor de nivel 3 configuró los emparejamientos. Si el circuito tiene un estado aprovisionado, ya puede crear conexiones.
>

### <a name="to-create-a-connection"></a>Para crear una conexión

1. Asegúrese de que el circuito ExpressRoute y el emparejamiento privado de Azure estén correctamente configurados. Siga las instrucciones de [Creación de un circuito ExpressRoute](expressroute-howto-circuit-arm.md) y de [Creación y modificación del emparejamiento de un circuito ExpressRoute](expressroute-howto-routing-arm.md). El circuito ExpressRoute debe tener un aspecto similar a la imagen que aparece a continuación:

   [![Captura de pantalla de un circuito de ExpressRoute](./media/expressroute-howto-linkvnet-portal-resource-manager/routing1.png "Ver circuito")](./media/expressroute-howto-linkvnet-portal-resource-manager/routing1-exp.png#lightbox)
2. Ahora puede iniciar el aprovisionamiento de una conexión para vincular la puerta de enlace de red virtual con el circuito ExpressRoute. Haga clic en **Conexión** > **Agregar** para abrir la página **Agregar conexión** y configure los valores.

   [![Captura para agregar conexión](./media/expressroute-howto-linkvnet-portal-resource-manager/samesub1.png "Captura para agregar conexión")](./media/expressroute-howto-linkvnet-portal-resource-manager/samesub1-exp.png#lightbox)
3. Una vez que la conexión esté correctamente configurada, el objeto de conexión mostrará la información de la conexión.

   ![Captura de pantalla de objeto de conexión](./media/expressroute-howto-linkvnet-portal-resource-manager/samesub2.png)


## <a name="connect-a-vnet-to-a-circuit---different-subscription"></a>Conexión de una red virtual con un circuito: otra suscripción

Puede compartir un circuito ExpressRoute entre varias suscripciones. En la ilustración siguiente se muestra un sencillo esquema de cómo funciona el uso compartido de circuitos ExpressRoute entre varias suscripciones.

![Conectividad entre suscripciones](./media/expressroute-howto-linkvnet-portal-resource-manager/cross-subscription.png)

- Cada una de las nubes más pequeñas dentro de la nube de gran tamaño se usa para representar las suscripciones que pertenecen a diferentes departamentos dentro de una organización.
- Cada departamento de la organización puede usar su propia suscripción para implementar sus servicios, pero puede compartir un único circuito ExpressRoute para volver a conectarse a la red local.
- Un solo departamento (en este ejemplo: TI) puede ser el propietario de ExpressRoute. Otras suscripciones de la organización pueden usar el circuito ExpressRoute y autorizaciones asociadas con dicho circuito, incluidas las suscripciones vinculadas a otros inquilinos de Azure Active Directory e inscripciones al Contrato Enterprise.

  > [!NOTE]
  > Los cargos de conectividad y ancho de banda de un circuito dedicado recaerán en el propietario del circuito ExpressRoute. Todas las redes virtuales comparten el mismo ancho de banda.
  >
  >

### <a name="administration---about-circuit-owners-and-circuit-users"></a>Administración: acerca de los propietarios y los usuarios del circuito

El "propietario del circuito" es un usuario avanzado autorizado del recurso de circuito ExpressRoute. Puede crear autorizaciones que los "propietarios del circuito", a su vez, pueden canjear. Los usuarios del circuito son propietarios de puertas de enlace de red virtual que no se incluyen en la misma suscripción que el circuito ExpressRoute. Los usuarios del circuito pueden canjear autorizaciones (una autorización por red virtual).

El propietario del circuito tiene la capacidad de modificar y revocar las autorizaciones en cualquier momento. La revocación de una autorización dará como resultado la eliminación de todas las conexiones de vínculos de la suscripción cuyo acceso se haya revocado.

### <a name="circuit-owner-operations"></a>Operaciones del propietario del circuito

**Creación de una autorización de conexión**

El propietario del circuito crea una autorización. Esto da lugar a la creación de una clave de autorización que puede usar un usuario del circuito para conectar sus puertas de enlace de red virtual al circuito ExpressRoute. Una autorización solo es válida para una conexión.

> [!NOTE]
> Cada conexión requiere una autorización independiente.
>

1. En la página de ExpressRoute, haga clic en **Autorizaciones**, escriba el nombre de la autorización en **Nombre** y haga clic en **Guardar**.

   ![Autorizaciones](./media/expressroute-howto-linkvnet-portal-resource-manager/authorization.png)
2. Una vez guardada la configuración, copie los valores de **Identificador de recurso** y **Clave de autorización**.

   ![Clave de autorización](./media/expressroute-howto-linkvnet-portal-resource-manager/authkey.png)

**Eliminación de una autorización de conexión**

Para eliminar una conexión, seleccione el icono **Eliminar** en la página correspondiente a su conexión.

### <a name="circuit-user-operations"></a>Operaciones del usuario del circuito

El usuario del circuito necesita el identificador del recurso y una clave de autorización del propietario del circuito.

**Canje de una autorización de conexión**

1. Haga clic en el botón **+Nuevo**.

   ![Haga clic en Nuevo](./media/expressroute-howto-linkvnet-portal-resource-manager/Connection1.png)
2. Busque el nombre **"Conexión"** en Marketplace, selecciónelo y haga clic en **Crear**.

   ![Búsqueda de conexión](./media/expressroute-howto-linkvnet-portal-resource-manager/Connection2.png)
3. Asegúrese de que **Tipo de conexión** está establecido en "ExpressRoute".
4. Especifique los datos y haga clic en **Aceptar** en la página Datos básicos.

   ![Página Datos básicos](./media/expressroute-howto-linkvnet-portal-resource-manager/Connection3.png)
5. En la página **Configuración**, seleccione una opción en **Puerta de enlace de red virtual** y active la casilla **Canjear autorización**.
6. Escriba un valor en **Clave de autorización** y en **Peer circuit URI** (Emparejar URI de circuito) y asigne un nombre a la conexión. Haga clic en **OK**. El **URI de circuito del mismo nivel** es el identificador de recurso del circuito ExpressRoute (que puede encontrar en el panel de configuración de las propiedades del circuito ExpressRoute).

   ![Página de configuración](./media/expressroute-howto-linkvnet-portal-resource-manager/Connection4.png)
7. Revise la información de la página **Resumen** y haga clic en **Aceptar**.

**Liberación de una autorización de conexión**

Puede liberar una autorización eliminando la conexión que vincula el circuito ExpressRoute a la red virtual.

## <a name="delete-a-connection-to-unlink-a-vnet"></a>Eliminación de una conexión para desvincular una red virtual

Puede eliminar una conexión y desvincular la red virtual de un circuito de ExpressRoute; para ello, seleccione el icono **Eliminar** en la página de la conexión.

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información acerca de ExpressRoute, consulte [P+F de ExpressRoute](expressroute-faqs.md).
