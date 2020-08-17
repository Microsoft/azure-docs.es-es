---
title: Configuración del enrutamiento de centro virtual
titleSuffix: Azure Virtual WAN
description: En este artículo se describe cómo configurar el enrutamiento de centro virtual
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 07/07/2020
ms.author: cherylmc
ms.openlocfilehash: 634967fbe50c74770bdc3b526d68896728e4ee07
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87513067"
---
# <a name="how-to-configure-virtual-hub-routing"></a>Configuración del enrutamiento de centro virtual

Un centro virtual puede contener varias puertas de enlace, como una puerta de enlace de VPN de sitio a sitio, una puerta de enlace de ExpressRoute, una puerta de enlace de punto a sitio y Azure Firewall. Las funcionalidades de enrutamiento del centro virtual se proporcionan mediante un enrutador que administra todo el enrutamiento, incluido el enrutamiento del tránsito, entre las puertas de enlace mediante el Protocolo de puerta de enlace de borde (BGP). Este enrutador también proporciona conectividad de tránsito entre las redes virtuales que se conectan a un centro virtual y pueden admitir hasta un rendimiento agregado de 50 Gbps. Estas funcionalidades de enrutamiento se aplican a los clientes de Virtual WAN estándar.

Para obtener más información, consulte [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).

> [!NOTE]
> Es posible que algunas de estas funcionalidades todavía se estén implementando. Se espera que la implementación se haya completado el 17 de agosto.
>

## <a name="create-a-route-table"></a><a name="create-table"></a>Creación de una tabla de rutas

1. En Azure Portal, vaya al centro virtual.
2. En **Conectividad**, seleccione **Enrutamiento**. En la página Enrutamiento, verá las tablas de rutas **Predeterminada** y **Ninguna**.

   :::image type="content" source="./media/how-to-virtual-hub-routing/routing.png" alt-text="Página Enrutamiento":::
3. Seleccione **+ Crear tabla de rutas** para abrir la página **Crear tabla de rutas**.
4. En la pestaña **Aspectos básicos** de la página Crear tabla de rutas, rellene los campos siguientes.

   :::image type="content" source="./media/how-to-virtual-hub-routing/basics.png" alt-text="Pestaña Aspectos básicos":::

   * **Nombre**
   * **Rutas**
   * **Nombre de ruta**
   * **Tipo de destino**
   * **Prefijo de destino**: puede agregar prefijos. Por ejemplo: Red virtual 1: 10.1.0.0/24 y red virtual 2: 10.1.1.0/24 se pueden agregar como 10.1.0.0/16. Las rutas de **Rama** se aplican a todos los sitios VPN conectados, circuitos ExpressRoute y conexiones VPN de usuario.
   * **Próximo salto**: lista de conexiones de red virtual o Azure Firewall.

     Si selecciona una conexión de red virtual, verá **Configurar rutas estáticas**. Esta configuración es opcional. Para más información, consulte [Configuración de rutas estáticas](about-virtual-hub-routing.md#static).

      :::image type="content" source="./media/how-to-virtual-hub-routing/next-hop.png" alt-text="Próximo salto":::

5. Seleccione la pestaña **Etiquetas** para configurar los nombres de etiqueta. Las etiquetas proporcionan un mecanismo para agrupar las tablas de rutas de forma lógica.

    :::image type="content" source="./media/how-to-virtual-hub-routing/labels.png" alt-text="Configurar nombres de etiqueta":::

6. Seleccione la pestaña **Asociaciones** para asociar conexiones a la tabla de rutas.
Verá las secciones **Ramas**, **Redes virtuales** y **Configuración actual** de las conexiones.

    :::image type="content" source="./media/how-to-virtual-hub-routing/associations.png" alt-text="Conexiones de asociación a la tabla de rutas":::

7. Seleccione la pestaña **Propagaciones** para propagar las rutas de las conexiones a la tabla de rutas.

    :::image type="content" source="./media/how-to-virtual-hub-routing/propagations.png" alt-text="Propagar rutas":::

8. Seleccione **Crear** para crear la tabla de rutas.

## <a name="to-edit-a-route-table"></a><a name="edit-table"></a>Para editar una tabla de rutas

En Azure Portal, busque la tabla de rutas del centro virtual. Seleccione la tabla de rutas para editar cualquier información.

## <a name="to-delete-a-route-table"></a><a name="delete-table"></a>Para eliminar una tabla de rutas

En Azure Portal, busque la tabla de rutas del centro virtual. No se puede eliminar una tabla de rutas Predeterminada o Ninguna. Sin embargo, puede eliminar todas las tablas de rutas personalizadas. Haga clic en **"..."** y, a continuación, seleccione **Eliminar**.

## <a name="to-view-effective-routes"></a><a name="view-routes"></a>Para ver rutas eficaces

En Azure Portal, busque la tabla de rutas del centro virtual. Haga clic en **"..."** y seleccione **Rutas eficaces** para ver las rutas que ha aprendido la tabla de rutas seleccionada. Las rutas propagadas desde la conexión a la tabla de rutas se rellenan automáticamente en **Rutas eficaces** de la tabla de rutas. Para obtener más información, consulte [Acerca de las rutas eficaces](effective-routes-virtual-hub.md).

:::image type="content" source="./media/how-to-virtual-hub-routing/effective.png" alt-text="Ver rutas eficaces" lightbox="./media/how-to-virtual-hub-routing/effective-expand.png":::

## <a name="to-set-up-routing-configuration-for-a-virtual-network-connection"></a><a name="routing-configuration"></a>Para establecer la configuración de enrutamiento de una conexión de red virtual

1. En Azure Portal, vaya a Virtual WAN y, en **Conectividad**, seleccione **Conexiones de red virtual**.
1. Seleccione **+Agregar conexión**.
1. Seleccione la red virtual en la lista desplegable.
1. Configure la configuración de enrutamiento que se va a asociar a una tabla de rutas. En **Asociar tabla de rutas**, seleccione la tabla de rutas de la lista desplegable.
1. Configure la configuración de enrutamiento que se va a propagar a una o varias tablas de rutas. En **Propagar a tabla de rutas**, seleccione una opción de la lista desplegable.
1. En **Rutas estáticas**, configure las rutas estáticas de la aplicación virtual de red (si es aplicable). Virtual WAN admite una única dirección IP de próximo salto para la ruta estática en una conexión de red virtual. Por ejemplo, si tiene una aplicación virtual independiente para flujos de tráfico de entrada y salida, sería mejor tener las aplicaciones virtuales en redes virtuales independientes y conectar las redes virtuales al centro virtual.


:::image type="content" source="./media/how-to-virtual-hub-routing/routing-configuration.png" alt-text="Definir la configuración de enrutamiento" lightbox="./media/how-to-virtual-hub-routing/routing-configuration-expand.png":::

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el enrutamiento de centros virtuales, vea [Acerca del enrutamiento de centros virtuales](about-virtual-hub-routing.md).
Para más información sobre Virtual WAN, consulte las [preguntas más frecuentes](virtual-wan-faq.md).
