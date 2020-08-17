---
title: 'Tutorial: Creación de un segmento de red de NSX-T en Azure VMware Solution (AVS)'
description: En este tutorial, ha creado los segmentos de red de NSX-T que se usan para las máquinas virtuales en vCenter.
ms.topic: tutorial
ms.date: 07/16/2020
ms.openlocfilehash: 5654fbb6a063d4dfeb541c20407f9a09dff1509f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87093974"
---
# <a name="tutorial-create-an-nsx-t-network-segment-in-azure-vmware-solution-avs"></a>Tutorial: Creación de un segmento de red de NSX-T en Azure VMware Solution (AVS)

Los segmentos de red creados en NSX-T Manager se usan como redes para máquinas virtuales (VM) en vCenter. Las máquinas virtuales creadas en vCenter se colocan en los segmentos de red creados en NSX-T y son visibles en vCenter.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Desplazarse en NSX-T Manager para agregar segmentos de red
> * Agregar un nuevo segmento de red
> * Observar el nuevo segmento de red en vCenter

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, se requiere una nube privada AVS con acceso a las interfaces de administración de vCenter y NSX-T Manager. Consulte [Tutorial: Configuración de redes para la nube privada de VMware en Azure](tutorial-configure-networking.md).

## <a name="provision-a-network-segment-in-nsx-t"></a>Aprovisionamiento de un segmento de red en NSX-T

1. En vCenter para la nube privada, seleccione **SDDC-Datacenter > Networks** (Redes ) y observe que todavía no hay redes.

   :::image type="content" source="media/nsxt/vcenter-without-ls01.png" alt-text="Seleccione SDDC-Datacenter > Networks"::: (Redes)

1. En el administrador de NSX-T para su nube privada, seleccione **Networking** (Redes).

   :::image type="content" source="media/nsxt/nsxt-network-overview.png" alt-text="Seleccione Networking para ir a la vista de información general de red de NSX-T Manager.":::

1. Seleccione **Segments** (Segmentos).

   :::image type="content" source="media/nsxt/nsxt-select-segments.png" alt-text="Seleccione Segments (Segmentos) en la página de información general de red.":::

1. En la página de información general de segmentos de NSX-T, seleccione **ADD SEGMENT** (AGREGAR SEGMENTO). Se crean tres segmentos como parte del aprovisionamiento de la nube privada y no se pueden usar para las máquinas virtuales.  Tendrá que agregar un nuevo segmento de red para este fin.

   :::image type="content" source="media/nsxt/nsxt-segments-overview.png" alt-text="Seleccione Add Segment (Agregar segmento) en la página Network Segment overview (Información general del segmento de red).":::

1. Asigne un nombre al segmento; en **Connected Gateway** (Puerta de enlace conectada), elija la puerta de enlace Tier1 preconfigurada (TNTxx-T1); deje **Type** (Tipo) como Flexible; en **Transport Zone** (zona de transporte), elija la superposición preconfigurada (TNTxx-OVERLAY-TZ); y, por último, seleccione Set Subnets (Establecer subredes). El resto de la configuración de esta sección así como **PORTS**  (PUERTOS) y **SEGMENT PROFILES** (PERFILES DE SEGMENTO) pueden permanecer en el valor predeterminado, al igual que la configuración.

   :::image type="content" source="media/nsxt/nsxt-create-segment-specs.png" alt-text="Establezca el nombre del segmento, la puerta de enlace conectada y el tipo y la zona de transporte; a continuación, seleccione Set Subnets (Establecer subredes).":::

1. Establezca la dirección IP de la puerta de enlace para el nuevo segmento y, a continuación, seleccione **ADD** (AGREGAR). La dirección IP que use debe estar en un bloque de direcciones RFC1918 que no se superponga, lo que garantiza que pueda conectarse a las máquinas virtuales en el nuevo segmento.

   :::image type="content" source="media/nsxt/nsxt-create-segment-gateway.png" alt-text="Especifique la dirección IP de la puerta de enlace de segmento en la notación CIDR y seleccione ADD (AGREGAR).":::

1. Aplique el nuevo segmento de red; para ello, seleccione **APPLY** (APLICAR) y, después, guarde la configuración con **SAVE** (GUARDAR).

   :::image type="content" source="media/nsxt/nsxt-create-segment-apply.png" alt-text="Aplique el nuevo segmento de red a la configuración de NSX-T con APPLY (APLICAR).":::

   :::image type="content" source="media/nsxt/nsxt-create-segment-save.png" alt-text="Guarde el nuevo segmento de red en la configuración de NSX-T con SAVE (GUARDAR).":::

1. Ahora se ha creado el nuevo segmento de red y rechazará la opción para continuar configurando el segmento si selecciona **NO**.

   :::image type="content" source="media/nsxt/nsxt-create-segment-continue-no.png" alt-text="Para seguir configurando el segmento de red recién creado, seleccione NO.":::

1. Confirme que el nuevo segmento de red está presente en NSX-T; para ello, seleccione **Networking > Segments** (Redes > Segmentos) y compruebe que el nuevo segmento se encuentra en la lista (en este caso, "ls01").

   :::image type="content" source="media/nsxt/nsxt-new-segment-overview-2.png" alt-text="Confirme que el nuevo segmento de red esté presente en NSX-T.":::

1. Confirme que el nuevo segmento de red esté presente en vCenter; para ello, seleccione **Networking > SDDC-Datacenter** (Redes > SDDC-Datacenter) y observe que el nuevo segmento esté en la lista (en este caso, "ls01").

   :::image type="content" source="media/nsxt/vcenter-with-ls01-2.png" alt-text="Confirme que el nuevo segmento de red está presente en vCenter.":::

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado los segmentos de red de NSX-T que se usan para las máquinas virtuales en vCenter. Ahora puede usar el [Tutorial: Creación de una biblioteca de contenido para implementar máquinas virtuales en Azure VMware Solution (AVS)](tutorial-deploy-vm-content-library.md) para crear una biblioteca de contenido en vCenter y aprovisionar una máquina virtual en la red que creó en este tutorial.

<!-- LINKS - external-->

<!-- LINKS - internal -->
