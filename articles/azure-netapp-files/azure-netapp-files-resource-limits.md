---
title: Límites de recursos para Azure NetApp Files| Microsoft Docs
description: Describe los límites para los recursos de Azure NetApp Files y cómo solicitar un aumento del límite de recursos.
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
ms.date: 08/21/2020
ms.author: b-juche
ms.openlocfilehash: 9facbc1629b8e1330c6bbafb4444d5bfc237d16f
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88752299"
---
# <a name="resource-limits-for-azure-netapp-files"></a>Límites de recursos para Azure NetApp Files

Entender los límites de recursos de Azure NetApp Files ayuda a administrar los volúmenes.

## <a name="resource-limits"></a>Límites de recursos

En la tabla siguiente se describen los límites de recursos de Azure NetApp Files:

|  Recurso  |  Límite predeterminado  |  Ajustable a través de la solicitud de soporte técnico  |
|----------------|---------------------|--------------------------------------|
|  Número de cuentas de NetApp por región de Azure   |  10    |  Sí   |
|  Número de grupos de capacidad por cuenta de NetApp   |    25     |   Sí   |
|  Número de volúmenes por suscripción   |    500     |   Sí   |
|  Número de volúmenes por grupo de capacidad     |    500   |    Sí     |
|  Número máximo de instantáneas por volumen       |    255     |    No        |
|  Número de subredes que se delegan a Azure NetApp Files (Microsoft.NetApp/volumes) por cada red virtual de Azure    |   1   |    No    |
|  Número de direcciones IP en uso en una red virtual (incluidas las redes virtuales emparejadas inmediatamente) con Azure NetApp Files   |    1000   |    No   |
|  Tamaño mínimo de un único grupo de capacidades   |  4 TiB     |    No  |
|  Tamaño máximo de un único grupo de capacidades    |  500 TiB   |   No   |
|  Tamaño mínimo de un único volumen    |    100 GiB    |    No    |
|  Tamaño máximo de un único volumen     |    100 TiB    |    No    |
|  Tamaño máximo de un archivo individual     |    16 TiB    |    No    |    
|  Tamaño máximo de los metadatos de directorio en un solo directorio      |    320 MB    |    No    |    
|  Número máximo de archivos ([maxfiles](#maxfiles)) por volumen     |    100 millones    |    Sí    |    

Para más información, consulte [Preguntas más frecuentes sobre la administración de la capacidad](azure-netapp-files-faqs.md#capacity-management-faqs).

## <a name="maxfiles-limits"></a>Límites de número máximo de archivos <a name="maxfiles"></a> 

Los volúmenes de Azure NetApp Files tienen un límite denominado *maxfiles*. El límite de maxfiles es el número de archivos que puede contener un volumen. El límite de maxfiles de un volumen de Azure NetApp Files se indexa en función del tamaño (cuota) del volumen. El límite de maxfiles de un volumen aumenta o disminuye a la velocidad de 20 millones de archivos por TiB del tamaño del volumen aprovisionado. 

El servicio ajusta dinámicamente el límite de maxfiles de un volumen en función de su tamaño aprovisionado. Por ejemplo, un volumen configurado inicialmente con un tamaño de 1 TiB tendría un límite de maxfiles de 20 millones. Los cambios posteriores en el tamaño del volumen provocarán un reajuste automático del límite de maxfiles en función de las siguientes reglas: 

|    Tamaño del volumen (cuota)     |  Reajuste automático del límite de maxfiles    |
|----------------------------|-------------------|
|    < 1 TiB                 |    20 millones     |
|    > = 1 TiB, pero < 2 TiB    |    40 millones     |
|    > = 2 TiB, pero < 3 TiB    |    60 millones     |
|    > = 3 TiB, pero < 4 TiB    |    80 millones     |
|    >= 4 TiB                |    100 millones    |

Si ya ha asignado al menos 4 TiB de cuota para un volumen, puede iniciar una [solicitud de soporte](#limit_increase) para aumentar el límite de maxfiles más allá de 100 millones.

## <a name="request-limit-increase"></a>Solicitud de aumento del límite<a name="limit_increase"></a> 

Puede crear una solicitud de soporte técnico de Azure para aumentar los límites ajustables de la tabla anterior. 

Desde el plano de navegación de Azure Portal: 

1. Haga clic en **Ayuda y soporte técnico**.
2. Haga clic en **Nueva solicitud de soporte técnico**.
3. En la pestaña Básico, especifique la siguiente información: 
    1. Tipo de problema: Seleccione **Límites de servicio y suscripción (cuotas)** .
    2. Suscripciones: Seleccione la suscripción del recurso que necesite aumentar la cuota.
    3. Tipo de cuota: Seleccione **Storage: Límites de Azure NetApp Files**.
    4. Haga clic en **Siguiente: Soluciones**.
4. Haga clic en la pestaña Detalles:
    1. En el cuadro Descripción, proporcione la siguiente información para el tipo de recurso correspondiente:

        |  Resource  |    Recursos primarios      |    Nuevos límites solicitados     |    Motivo de aumento de cuota       |
        |----------------|------------------------------|---------------------------------|------------------------------------------|
        |  Cuenta |  *Subscription ID*   |  *Nuevo número máximo de **cuentas** solicitado*    |  *¿Qué escenario o caso de uso motivó la solicitud?*  |
        |  grupo    |  *Identificador de suscripción y URI de la cuenta*  |  *Nuevo número máximo de **grupos** solicitado*   |  *¿Qué escenario o caso de uso motivó la solicitud?*  |
        |  Volumen  |  *Identificador de suscripción, URI y URI de grupo*   |  *Nuevo número máximo de **volúmenes** solicitado*     |  *¿Qué escenario o caso de uso motivó la solicitud?*  |
        |  Maxfiles  |  *Id. de suscripción, URI de cuenta, URI de grupo, URI de volumen*   |  *Nuevo número máximo de **maxfiles** solicitado*     |  *¿Qué escenario o caso de uso motivó la solicitud?*  |    

    2. Especifique admite el método de soporte técnico adecuado y proporcione la información del contrato.

    3. Haga clic en **Siguiente: Revisar y crear** para crear la solicitud. 


## <a name="next-steps"></a>Pasos siguientes  

- [Información sobre la jerarquía del almacenamiento de Azure NetApp Files](azure-netapp-files-understand-storage-hierarchy.md)
- [Modelo de costo de Azure NetApp Files](azure-netapp-files-cost-model.md)
