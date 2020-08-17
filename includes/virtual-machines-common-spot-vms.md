---
title: archivo de inclusión
description: archivo de inclusión
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 07/20/2020
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: df78133f602466681da64d2666a311e1649c598f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87028809"
---
El uso de máquinas virtuales de Spot permite aprovechar las ventajas de nuestra capacidad no utilizada con un importante ahorro en los costos. Siempre que Azure necesite recuperar la capacidad, su infraestructura expulsará las máquinas virtuales de Spot. Por lo tanto, estas son excelentes para cargas de trabajo que soportan interrupciones, como los trabajos de procesamiento por lotes, los entornos de desarrollo y pruebas, las grandes cargas de trabajo de proceso, etc.

La cantidad de capacidad sin usar disponible varía, por ejemplo, en función del tamaño, la región o la hora del día. Al implementar máquinas virtuales de Spot, Azure las asigna si hay capacidad disponible, pero no hay un Acuerdo de Nivel de Servicio para ellas. Una máquina virtual de Spot no ofrece garantías de alta disponibilidad. Siempre que Azure necesite recuperar la capacidad, su infraestructura expulsará las máquinas virtuales de Spot previo aviso 30 segundos antes. 


## <a name="eviction-policy"></a>Directiva de expulsión

Las máquinas virtuales se pueden expulsar en función de la capacidad o del precio máximo establecido. Al crear VM de Spot, puede establecer la directiva de expulsión en *Deallocate* (Desasignar) (valor predeterminado) o *Delete* (Eliminar). 

La directiva *Deallocate* (Desasignar) mueve las VM al estado stopped-deallocated, lo que le permite volver a implementarlas más tarde. Sin embargo, no hay ninguna garantía de que la asignación se realizará correctamente. Las VM desasignadas se siguen teniendo en cuenta en la cuota y se cobrarán los costos de almacenamiento de los discos subyacentes. 

Si quiere que la VM se elimine al expulsarse, puede establecer la directiva de expulsión en *delete* (eliminar). Las VM expulsadas se eliminan junto con sus discos subyacentes y, por tanto, no se le cobrará el almacenamiento. 

Puede optar por recibir notificaciones en las VM mediante [Azure Scheduled Events](../articles/virtual-machines/linux/scheduled-events.md). De este modo se le notificará que se van a expulsar las máquinas virtuales y tendrá 30 segundos para terminar los trabajos y cerrar las tareas antes de que esto ocurra. 


| Opción | Resultado |
|--------|---------|
| El precio máximo se establece en un valor igual o mayor que el precio actual. | La máquina virtual se implementa si hay capacidad y cuota disponibles. |
| El precio máximo se establece en un valor menor que el precio actual. | La máquina virtual no se implementa. Obtendrá un mensaje de error que indicará que el precio máximo debe ser igual o mayor que el precio actual. |
| Reinicio de una máquina virtual detenida o desasignada si el precio máximo es igual o mayor que el precio actual | Si hay capacidad y cuota, la máquina virtual se implementa. |
| Reinicio de una máquina virtual detenida o desasignada si el precio máximo es menor que el precio actual | Obtendrá un mensaje de error que indicará que el precio máximo debe ser igual o mayor que el precio actual. | 
| El precio de la máquina virtual ha aumentado y ahora es mayor que el precio máximo. | La máquina virtual se expulsa. Recibirá una notificación 30 segundos antes de la expulsión. | 
| Después, el precio de la máquina virtual volverá a ser menor que el precio máximo. | La máquina virtual no se reiniciará automáticamente. Puede reiniciar la máquina virtual por su cuenta y se le cobrará el precio actual. |
| Si el precio máximo está establecido en `-1` | La máquina virtual no se expulsará por motivos de precio. El precio máximo será el precio actual, hasta como máximo el precio de las máquinas virtuales estándar. No se le cobrará por encima del precio estándar.| 
| Cambio del precio máximo | Debe desasignar la máquina virtual para cambiar el precio máximo. Desasigne la máquina virtual, establezca un nuevo precio máximo y actualícela. |


## <a name="limitations"></a>Limitaciones

No se admiten los siguientes tamaños de máquina virtual para Spot:
 - Serie B
 - Versiones de promoción de cualquier tamaño (como los tamaños de promoción Dv2, NV, NC, H)

Las máquinas virtuales de Spot se pueden implementar en cualquier región, excepto Microsoft Azure China 21Vianet.

<a name="channel"></a>

Actualmente se admiten los siguientes [tipos de ofertas](https://azure.microsoft.com/support/legal/offer-details/):

-   Contrato Enterprise
-   Pago por uso
-   Patrocinados
- Para obtener un proveedor de servicios en la nube (CSP), póngase en contacto con su asociado.


## <a name="pricing"></a>Precios

Los precios de las máquinas virtuales de Spot varían en función de la región y la SKU. Para más información, consulte precios de las máquinas virtuales para [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) y [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/). 


La variabilidad en los precios permite establecer un precio máximo, en dólares estadounidenses (USD), con un máximo de 5 decimales. Por ejemplo, el valor `0.98765` correspondería a un precio máximo de 0,98765 USD por hora. Si establece el precio máximo en `-1`, la máquina virtual no se expulsará por precio. El precio de la máquina virtual será el actual de Spot o el de una máquina virtual estándar, el menor de los dos, siempre que haya capacidad y cuota disponibles.


##  <a name="frequently-asked-questions"></a>Preguntas más frecuentes

**P:** Una vez creadas, ¿son las máquinas virtuales de Spot iguales que las estándar normales?

**R:** Sí, salvo que no hay ningún Acuerdo de Nivel de Servicio para las máquinas virtuales de Spot y se pueden expulsar en cualquier momento.


**P:** ¿Qué se debe hacer si se produce la expulsión pero aún se necesita capacidad?

**R:** Si necesita capacidad de inmediato, se recomienda usar máquinas virtuales estándar en lugar de máquinas virtuales de Spot.


**P:** ¿Cómo se administra la cuota para las máquinas virtuales de Spot?

**R:** Las máquinas virtuales de Spot tienen un grupo de cuotas independiente. La cuota de Spot se compartirá entre las máquinas virtuales y los conjuntos de escalado. Para más información, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits).


**P:** ¿Puedo solicitar una cuota adicional para Spot?

**R:** Sí, podrá enviar la solicitud para aumentar su cuota para las máquinas virtuales de Spot mediante el [proceso de solicitud de cuota estándar](https://docs.microsoft.com/azure/azure-portal/supportability/per-vm-quota-requests).


**P:** ¿Dónde puedo publicar preguntas?

**R:** Puede publicar y etiquetar la pregunta con `azure-spot` en [Q&A](https://docs.microsoft.com/answers/topics/azure-spot.html) (Preguntas y respuestas). 



