---
title: archivo de inclusión
description: archivo de inclusión
services: batch
author: JnHs
ms.service: batch
ms.topic: include
ms.date: 06/03/2020
ms.author: jenhayes
ms.custom: include file
ms.openlocfilehash: 2a35180e4c5ca26d53be135718e345f9657af6a2
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88655148"
---
| **Recurso** | **Límite predeterminado** | **Límite máximo** |
| --- | --- | --- |
| Cuentas de Azure Batch por región y suscripción | 1-3 |50 |
| Núcleos dedicados por cuenta de Batch | 90-900 | Ponerse en contacto con soporte técnico |
| Núcleos de baja prioridad por cuenta de Batch | 10-100 | Ponerse en contacto con soporte técnico |
| Trabajos **[activos](https://docs.microsoft.com/rest/api/batchservice/job/get#jobstate)** y programaciones de trabajos por cuenta de Batch (los trabajos **completados** no tienen ningún límite) | 100-300 | 1000<sup>1</sup> |
| Grupos por cuenta de Batch | 20-100 | 500<sup>1</sup> |

<sup>1</sup> Si quiere solicitar un aumento de este límite, póngase en contacto con el soporte técnico de Azure.

> [!IMPORTANT]
> Estamos cambiando la forma de solicitar y administrar la cuota dedicada.  El número total de CPU virtuales dedicadas es el valor que se aplica actualmente, pero pronto aplicaremos una cuota dedicada por serie de VM. La cuota de prioridad baja se seguirá aplicando según el límite total; no se aplicará por serie de VM.

> [!NOTE]
> Los límites predeterminados varían según el tipo de suscripción que se use para crear una cuenta de Batch. Las cuotas de núcleos mostradas son para las cuentas de Batch del modo de servicio Batch. [Vea las cuotas de su cuenta de Batch](../articles/batch/batch-quota-limit.md#view-batch-quotas).

> [!IMPORTANT]
> Para ayudarnos a administrar mejor la capacidad durante la pandemia sanitaria global, las cuotas de núcleo predeterminadas para las nuevas cuentas de Batch en algunas regiones y para algunos tipos de suscripción se han reducido con respecto al intervalo de valores anterior, en algunos casos a cero núcleos. Al crear una nueva cuenta de Batch, debe [comprobar la cuota de núcleos](../articles/batch/batch-quota-limit.md#view-batch-quotas) y [solicitar un aumento de la cuota de núcleos](../articles/batch/batch-quota-limit.md#increase-a-quota) si es necesario. Como alternativa, considere la posibilidad de volver a usar aquellas cuentas de Batch que ya tengan suficiente cuota. 
