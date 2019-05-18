---
title: Límites de recursos para Azure NetApp Files| Microsoft Docs
description: Describe los límites de recursos para Azure NetApp Files, incluidos los límites de las cuentas de NetApp, los grupos de capacidad, los volúmenes, instantáneas y la subred delegada.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: b-juche
ms.openlocfilehash: b55467d77beb8f97b8e392b72682268ae0407e54
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65826372"
---
# <a name="resource-limits-for-azure-netapp-files"></a>Límites de recursos para Azure NetApp Files

Entender los límites de recursos de Azure NetApp Files ayuda a administrar los volúmenes.

## <a name="resource-limits"></a>Límites de recursos

En la tabla siguiente se describe los límites de recursos de Azure Files de NetApp:

|  Recurso  |  Límite predeterminado  |  Ajustable a través de la solicitud de soporte técnico  |
|----------------|---------------------|--------------------------------------|
|  Número de cuentas de NetApp por suscripción de Azure   |  10    |  Sí   |
|  Número de grupos de capacidad por cuenta de NetApp   |    25     |   Sí   |
|  Número de volúmenes por grupo de capacidad     |    500   |    Sí     |
|  Número de instantáneas por volumen       |    255     |    No        |
|  Número de subredes que se delegan a Azure Files de NetApp (Microsoft.NetApp/volumes) por cada red Virtual de Azure    |   1   |    No    |
|  Número máximo de máquinas virtuales (incluye redes virtuales emparejadas) que le permitan conectarse a un volumen     |    1000   |    No   |
|  Tamaño mínimo de un grupo de capacidades único   |  4 TiB     |    No  |
|  Tamaño máximo de un grupo de capacidades único    |  500 TiB   |   No   |
|  Tamaño mínimo de un único volumen    |    100 GiB    |    No    |
|  Máximo de un único volumen * la cuota asignada   |   92 TiB   |    No   |
|  Tamaño máximo de un volumen único *     |    100 TB    |    No       |

* Un volumen se puede crear o cambiar el tamaño máximo 92 TiB manualmente. Sin embargo, un volumen puede crecer hasta 100 TB en un escenario por encima del límite. Consulte [modelo de costos de Azure Files de NetApp](azure-netapp-files-cost-model.md) para obtener más información sobre por encima del límite de capacidad. 

## <a name="request-limit-increase"></a>Solicitar aumento del límite 

Puede crear una solicitud de soporte técnico de Azure para aumentar los límites de la tabla anterior ajustables. 

Desde el plano de navegación de Azure portal: 

1. Haga clic en **ayuda y soporte técnico**.
2. Haga clic en **+ nueva solicitud de soporte técnico**.
3. En la pestaña aspectos básicos, proporcione la siguiente información: 
    1. Tipo de problema: Seleccione **límites de servicio y suscripción (cuotas)**.
    2. Suscripciones: Seleccione la suscripción para el recurso que necesite aumentar la cuota.
    3. Tipo de cuota: Seleccione **almacenamiento: Los archivos de NetApp Azure limita**.
    4. Haga clic en **Siguiente: Soluciones**.
4. En la pestaña Detalles:
    1. En el cuadro Descripción, proporcione la siguiente información para el tipo de recurso correspondiente:

        |  Recurso  |    Recursos primarios      |    Nuevos límites solicitados     |    Motivo de aumento de cuota       |
        |----------------|------------------------------|---------------------------------|------------------------------------------|
        |  Cuenta |  *Subscription ID*   |  *Nuevo máximo solicitado **cuenta** número*    |  *¿Qué escenario o caso de uso se le solicite la solicitud?*  |
        |  Grupo    |  *Identificador de suscripción, el URI de la cuenta*  |  *Nuevo máximo solicitado **grupo** número*   |  *¿Qué escenario o caso de uso se le solicite la solicitud?*  |
        |  Volumen  |  *Identificador de suscripción, URI, de la cuenta grupo URI*   |  *Nuevo máximo solicitado **volumen** número*     |  *¿Qué escenario o caso de uso se le solicite la solicitud?*  |

    2. Especifique admite el método adecuado y proporcione la información de contrato.

    3. Haga clic en **Siguiente: Revisar y crear** para crear la solicitud. 


## <a name="next-steps"></a>Pasos siguientes  

- [Información sobre la jerarquía del almacenamiento de Azure NetApp Files](azure-netapp-files-understand-storage-hierarchy.md)
- [Modelo de costos para Azure Files de NetApp](azure-netapp-files-cost-model.md)
