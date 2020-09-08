---
title: Creación y administración de DHCP
description: En este artículo se explica cómo administrar DHCP en Azure VMware Solution.
ms.topic: conceptual
ms.date: 05/04/2020
ms.openlocfilehash: 3fc3de228179925afdf1b7c1015c577fd9c4c924
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88752161"
---
# <a name="how-to-create-and-manage-dhcp-in-azure-vmware-solution"></a>Creación y administración de DHCP en Azure VMWare Solution

NSX-T le proporciona la capacidad de configurar DHCP para su nube privada. Si tiene previsto usar NSX-T para hospedar el servidor DHCP, consulte [Creación de un servidor DHCP](#create-dhcp-server). En caso contrario, si tiene un servidor DHCP externo de terceros en la red y desea retransmitir las solicitudes a ese servidor DHCP, consulte [Creación de un servicio de retransmisión DHCP](#create-dhcp-relay-service).

## <a name="create-dhcp-server"></a>Creación de un servidor DHCP

Para configurar un servidor DHCP en NSX-T, siga los pasos indicados a continuación.

En el administrador de NSX, vaya a la pestaña **Networking** (Redes) y seleccione **DHCP** en **IP Management** (Administración de IP). Seleccione el botón **ADD SERVER** (Agregar servidor). Proporcione el nombre del servidor y su dirección IP. Cuando haya terminado, seleccione **Save** (Guardar).

:::image type="content" source="./media/manage-dhcp/dhcp-server-settings.png" alt-text="agregar un servidor DHCP" border="true":::

### <a name="connect-dhcp-server-to-the-tier-1-gateway"></a>Conecte el servidor DHCP a la puerta de enlace de nivel 1.

1. Seleccione **Tier 1 Gateways** (Puertas de enlace de nivel 1), luego, la puerta de enlace y, a continuación, **Edit** (Editar).

   :::image type="content" source="./media/manage-dhcp/edit-tier-1-gateway.png" alt-text="seleccionar la puerta de enlace que se va a usar" border="true":::

1. Para agregar una subred, seleccione **No IP Allocation Set** (No hay conjuntos de asignación de direcciones IP).

   :::image type="content" source="./media/manage-dhcp/add-subnet.png" alt-text="agregar una subred" border="true":::

1. En la siguiente pantalla, seleccione **DHCP Local Server** (Servidor local DHCP) en la lista desplegable **Type** (Tipo). En **DHCP Server** (Servidor DHCP), seleccione **Default DHCP** (DHCP predeterminado) y, luego, **Save** (Guardar).

   :::image type="content" source="./media/manage-dhcp/set-ip-address-management.png" alt-text="selección de opciones para el servidor DHCP" border="true":::

1. En la ventana **Tier -1 Gateway** (Puerta de enlace de nivel 1), seleccione **Save** (Guardar). En la siguiente pantalla verá **Changes Saved** (Cambios guardados), seleccione **Close Editing** (Cerrar edición) para finalizar.

### <a name="add-a-network-segment"></a>Incorporación de un segmento de red

Una vez que haya creado el servidor DHCP, deberá agregarle segmentos de red.

1. En NSX-T, seleccione la pestaña **Networking** (Redes) y, luego, **Segments** (Segmentos) en **Connectivity** (Conectividad). Seleccione **ADD SEGMENT** (AGREGAR SEGMENTO). Asigne un nombre al segmento y a la conexión en la puerta de enlace de nivel 1. A continuación, seleccione **Set Subnets** (Establecer subredes) para configurar una nueva subred. 

   :::image type="content" source="./media/manage-dhcp/add-segment.png" alt-text="agregar nuevo segmento de red" border="true":::

1. En la ventana **Set Subnets** (Establecer subredes), seleccione **ADD SUBNET** (AGREGAR SUBRED). Escriba la dirección IP de la puerta de enlace y el intervalo de DHCP, y seleccione **Add** (Agregar) y, después, **APPLY** (APLICAR).

   :::image type="content" source="./media/manage-dhcp/add-subnet-segment.png" alt-text="agregar segmento de red" border="true":::

1. Cuando haya terminado, seleccione **Save** (Guardar) para terminar de agregar el segmento de red.

   :::image type="content" source="./media/manage-dhcp/segments-complete.png" alt-text="segmentos terminados" border="true":::

## <a name="create-dhcp-relay-service"></a>Creación de un servicio de retransmisión DHCP

1. En la ventana de NXT-T, seleccione la pestaña **Networking** (Redes) y, en **IP Management** (Administración de direcciones IP), elija **DHCP**. Seleccione **ADD SERVER** (AGREGAR SERVIDOR). Elija DHCP Relay (Retransmisión DHCP) para **Server Type** (Tipo de servidor) y escriba el nombre del servidor y la dirección IP del servidor de retransmisión. Haga clic en **Guardar** para guardar los cambios.

   :::image type="content" source="./media/manage-dhcp/create-dhcp-relay.png" alt-text="crear servidor de retransmisión DHCP" border="true":::

1. Seleccione **Tier-1 Gateways** (Puertas de enlace de nivel 1) en **Connectivity** (Conectividad). Seleccione los puntos suspensivos verticales en la puerta de enlace de nivel 1 y elija **Edit** (Editar).

   :::image type="content" source="./media/manage-dhcp/edit-tier-1-gateway-relay.png" alt-text="editar puerta de enlace de nivel 1" border="true":::

1. Seleccione **No IP Allocation Set** (No hay conjuntos de asignación de direcciones IP) para definir la asignación de direcciones IP.

   :::image type="content" source="./media/manage-dhcp/edit-ip-address-allocation.png" alt-text="editar asignación de direcciones IP" border="true":::

1. En el cuadro de diálogo, para **Type** (Tipo), seleccione **DHCP Relay Server** (Servidor de retransmisión DHCP). En la lista desplegable **DHCP Relay** (Retransmisión DHCP), seleccione el servidor de retransmisión DHCP. Cuando termine, seleccione **Save** (Guardar).

   :::image type="content" source="./media/manage-dhcp/set-ip-address-management-relay.png" alt-text="configurar administración de direcciones IP" border="true":::

## <a name="specify-a-dhcp-range-ip-on-segment"></a>Definición de una dirección IP del intervalo de DHCP en el segmento

> [!NOTE]
> Esta configuración es necesaria para realizar la funcionalidad de retransmisión DHCP en el segmento del cliente DHCP. 

1. En **Connectivity** (Conectividad), seleccione **Segments** (Segmentos). Seleccione los puntos suspensivos verticales y, luego, **Edit** (Editar). Si desea agregar un nuevo segmento, en su lugar, puede seleccionar **Add Segment** (Agregar segmento) para crear un nuevo segmento.

   :::image type="content" source="./media/manage-dhcp/edit-segments.png" alt-text="editar una subred de red" border="true":::

1. Agregue detalles sobre el segmento. Seleccione el valor en **Subnets** (Subredes) o **Set Subnets** (Establecer subredes) para agregar o modificar la subred.

   :::image type="content" source="./media/manage-dhcp/network-segments.png" alt-text="segmentos de red" border="true":::

1. Seleccione los puntos suspensivos verticales y elija **Edit** (Editar). Si necesita crear una nueva subred, seleccione **Add Subnet** (Agregar subred) para crear una puerta de enlace y configurar un intervalo de DHCP. Proporcione el intervalo del grupo de direcciones IP y seleccione **Apply** (Aplicar) y, después, **Save** (Guardar).

   :::image type="content" source="./media/manage-dhcp/edit-subnet.png" alt-text="editar subredes" border="true":::

1. Ahora se asigna un grupo de servidores DHCP al segmento.

   :::image type="content" source="./media/manage-dhcp/assigned-to-segment.png" alt-text="grupo de servidores DHCP asignado al segmento" border="true":::
