---
title: Configuración de vRealize Operations para Azure VMware Solution
description: Aprenda a configurar vRealize Operations para una nube privada de Azure VMware Solution.
ms.topic: how-to
ms.date: 08/06/2020
ms.openlocfilehash: 729ee5c64776d7d04f702af62451175f7c53421b
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88750401"
---
# <a name="set-up-vrealize-operations-for-azure-vmware-solution"></a>Configuración de vRealize Operations para Azure VMware Solution


vRealize Operations Manager es una plataforma de administración de operaciones que permite a los administradores de infraestructura de VMware supervisar recursos del sistema. Estos recursos del sistema podrían ser objetos de nivel de aplicación o de nivel de infraestructura (ambos físicos y virtuales). Históricamente, la mayoría de los administradores han usado vRealize Operations para supervisar y administrar los componentes de nube privada de VMware: vCenter, ESXi, NSX-T, vSAN e Hybrid Cloud Extension (HCX). Cada nube privada de Azure VMware Solution se aprovisiona con una implementación dedicada de vCenter, NSX-T, vSAN y HCX. 

Revise primero detenidamente las secciones [Antes de comenzar](#before-you-begin) y [Requisitos previos](#prerequisites). A continuación, lo guiaremos a través de las dos topologías de implementación típicas para vRealize Operations Manager con Azure VMware Solution:

> [!div class="checklist"]
> * [vRealize Operations locales que administran la implementación de Azure VMware Solution](#on-premises-vrealize-operations-managing-azure-vmware-solution-deployment)
> * [vRealize Operations que se ejecutan en la implementación de Azure VMware Solution](#vrealize-operations-running-on-azure-vmware-solution-deployment)

## <a name="before-you-begin"></a>Antes de empezar
* Revise la [documentación del producto vRealize Operations Manager](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.vapp.doc/GUID-7FFC61A0-7562-465C-A0DC-46D092533984.html) para obtener más información sobre la implementación de vRealize Operations. 
* Revise la [serie de tutoriales](tutorial-network-checklist.md) básicos de Software Defined Datacenter (SDDC) de Azure VMware Solution.
* Opcionalmente, revise la documentación del producto [vRealize Operations Remote Controller](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.vapp.doc/GUID-263F9219-E801-4383-8A59-E84F3D01ED6B.html) para la opción de administración de vRealize Operations local de la implementación de Azure VMware Solution. 



## <a name="prerequisites"></a>Requisitos previos
* Se deben configurar VPN o Azure ExpressRoute entre el SDDC de Azure VMware Solution y el entorno local.
* Una nube privada de Azure VMware Solution se ha implementado en Azure.



## <a name="on-premises-vrealize-operations-managing-azure-vmware-solution-deployment"></a>vRealize Operations locales que administran la implementación de Azure VMware Solution
La mayoría de los clientes tienen una implementación local ya existente de vRealize Operations que se usa para administrar uno o varios dominios vCenters locales. Cuando los clientes aprovisionan una nueva nube privada de Azure VMware Solution en Azure, habitualmente conectan su entorno local con Azure VMware Solution mediante una instancia de Azure ExpressRoute o una solución VPN de nivel 3, como se muestra a continuación.   

:::image type="content" source="media/vrealize-operations-manager/vrealize-operations-deployment-option-1.png" alt-text="vRealize Operations locales que administran la implementación de Azure VMware Solution"  border="false":::

Para ampliar las funcionalidades de vRealize Operations a la nube privada de Azure VMware Solution, cree una [ instancia del adaptador para los recursos de una nube privada de Azure VMware Solution](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.config.doc/GUID-640AD750-301E-4D36-8293-1BFEB67E2600.html) para recopilar datos de la nube privada de Azure VMware Solution e incluirlos en vRealize Operations local. La instancia de vRealize Operations Manager local puede conectarse directamente a vCenter y NSX-T Manager en Azure VMware Solution, o bien, opcionalmente, puede implementar una instancia de vRealize Operations Remote Collector en la nube privada de Azure VMware Solution. Una instancia de vRealize Operations Remote Collector comprime y cifra los datos recopilados de la nube privada de Azure VMware Solution antes de su envío a través de la red VPN o de ExpressRoute a la instancia de vRealize Operations Manager que se ejecuta en el entorno local. 

> [!TIP]
> Consulte la [documentación de VMware](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.vapp.doc/GUID-7FFC61A0-7562-465C-A0DC-46D092533984.html) y obtenga una guía paso a paso de cómo instalar vRealize Operations Manager. 



## <a name="vrealize-operations-running-on-azure-vmware-solution-deployment"></a>vRealize Operations que se ejecutan en la implementación de Azure VMware Solution

Otra opción de implementación es implementar una instancia de vRealize Operations Manager en uno de los clústeres de vSphere de la nube privada de Azure VMware Solution, como se muestra a continuación. 

:::image type="content" source="media/vrealize-operations-manager/vrealize-operations-deployment-option-2.png" alt-text="vRealize Operations que se ejecutan en Azure VMware Solution" border="false":::

Después de implementar la instancia de Azure VMware Solution de vRealize Operations, puede configurar vRealize Operations para recopilar datos de vCenter, ESXi, NSX-T, vSAN y HCX. 

> [!TIP]
> Consulte la [documentación de VMware](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.vapp.doc/GUID-7FFC61A0-7562-465C-A0DC-46D092533984.html) y obtenga una guía paso a paso de cómo instalar vRealize Operations Manager.




<!-- LINKS - external -->


<!-- LINKS - internal -->




