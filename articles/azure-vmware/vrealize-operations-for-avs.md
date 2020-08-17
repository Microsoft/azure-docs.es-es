---
title: Configuración de vRealize Operations para Azure VMware Solution (AVS)
description: ''
ms.topic: how-to
ms.date: 08/06/2020
ms.openlocfilehash: 57b513b681c4d2522b1c92946aab80fe8ba95746
ms.sourcegitcommit: 5f7b75e32222fe20ac68a053d141a0adbd16b347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87475893"
---
# <a name="set-up-vrealize-operations-for-azure-vmware-solution-avs"></a>Configuración de vRealize Operations para Azure VMware Solution (AVS)


vRealize Operations Manager es una plataforma de administración de operaciones que permite a los administradores de infraestructura de VMware supervisar recursos del sistema. Estos recursos del sistema podrían ser objetos de nivel de aplicación o de nivel de infraestructura (ambos físicos y virtuales). Históricamente, la mayoría de los administradores han usado vRealize Operations para supervisar y administrar los componentes de nube privada de VMware: vCenter, ESXi, NSX-T, vSAN e Hybrid Cloud Extension (HCX). Cada nube privada de AVS se aprovisiona con una implementación dedicada de vCenter, NSX-T, vSAN y HCX. 

Revise primero detenidamente las secciones [Antes de comenzar](#before-you-begin) y [Requisitos previos](#prerequisites). A continuación, le guiaremos a través de las dos topologías de implementación típicas para vRealize Operations Manager con AVS:

> [!div class="checklist"]
> * [Administración de vRealize Operations local de la implementación de AVS](#on-premises-vrealize-operations-managing-avs-deployment)
> * [Ejecución de vRealize Operations en la implementación de AVS](#vrealize-operations-running-on-avs-deployment)

## <a name="before-you-begin"></a>Antes de empezar
* Revise la [documentación del producto vRealize Operations Manager](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.vapp.doc/GUID-7FFC61A0-7562-465C-A0DC-46D092533984.html) para obtener más información sobre la implementación de vRealize Operations. 
* Revise la [serie de tutoriales](tutorial-network-checklist.md) básicos de Software Defined Datacenter (SDDC) de AVS.
* Opcionalmente, revise la documentación del producto [vRealize Operations Remote Controller](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.vapp.doc/GUID-263F9219-E801-4383-8A59-E84F3D01ED6B.html) para la opción de administración de vRealize Operations local de la implementación de AVS. 



## <a name="prerequisites"></a>Requisitos previos
* Se deben configurar VPN o Azure ExpressRoute entre el SDDC de AVS y el entorno local.
* Una nube privada de AVS se ha implementado en Azure.



## <a name="on-premises-vrealize-operations-managing-avs-deployment"></a>Administración de vRealize Operations local de la implementación de AVS
La mayoría de los clientes tienen una implementación local ya existente de vRealize Operations que se usa para administrar uno o varios dominios vCenters locales. Cuando los clientes aprovisionan una nueva nube privada de AVS en Azure, normalmente conectan su entorno local con AVS mediante una instancia de Azure ExpressRoute o una solución VPN de nivel 3, como se muestra a continuación.   

:::image type="content" source="media/vrealize-operations-manager/vrealize-operations-deployment-option-1.png" alt-text="Administración de vRealize Operations local de la implementación de AVS"  border="false":::

Para ampliar las funcionalidades de vRealize Operations a la nube privada de AVS, cree una [ instancia del adaptador para los recursos de una nube privada de AVS](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.config.doc/GUID-640AD750-301E-4D36-8293-1BFEB67E2600.html) para recopilar datos de la nube privada de AVS e incluirlos en vRealize Operations local. La instancia de vRealize Operations Manager local puede conectarse directamente a vCenter y NSX-T Manager en AVS, o bien, opcionalmente, puede implementar una instancia de vRealize Operations Remote Collector en la nube privada de AVS. Una instancia de vRealize Operations Remote Collector comprime y cifra los datos recopilados de la nube privada de AVS antes de su envío a través de la red VPN o de ExpressRoute a la instancia de vRealize Operations Manager que se ejecuta en el entorno local. 

> [!TIP]
> Consulte la [documentación de VMware](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.vapp.doc/GUID-7FFC61A0-7562-465C-A0DC-46D092533984.html) y obtenga una guía paso a paso de cómo instalar vRealize Operations Manager. 



## <a name="vrealize-operations-running-on-avs-deployment"></a>Ejecución de vRealize Operations en la implementación de AVS

Otra opción de implementación es implementar una instancia de vRealize Operations Manager en uno de los clústeres de vSphere de la nube privada de AVS, como se muestra a continuación. 

:::image type="content" source="media/vrealize-operations-manager/vrealize-operations-deployment-option-2.png" alt-text="Ejecución de vRealize Operations en AVS" border="false":::

Después de implementar la instancia de AVS de vRealize Operations, puede configurar vRealize Operations para recopilar datos de vCenter, ESXi, NSX-T, vSAN y HCX. 

> [!TIP]
> Consulte la [documentación de VMware](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.vapp.doc/GUID-7FFC61A0-7562-465C-A0DC-46D092533984.html) y obtenga una guía paso a paso de cómo instalar vRealize Operations Manager.




<!-- LINKS - external -->


<!-- LINKS - internal -->




