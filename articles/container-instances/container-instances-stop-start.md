---
title: Detener o iniciar un grupo de contenedores manualmente
description: Obtenga información sobre cómo detener o iniciar un grupo de contenedores manualmente en Azure Container Instances.
ms.topic: article
ms.date: 08/11/2020
ms.openlocfilehash: f697f18459959cf40e71d3ca90cd8b42aaa74239
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88799084"
---
# <a name="manually-stop-or-start-containers-in-azure-container-instances"></a>Detener o iniciar contenedores manualmente en Azure Container Instances

La opción [directiva de reinicio](container-instances-restart-policy.md) configuración de un grupo de contenedores determina cómo las instancias de contenedor se inician o detienen de forma predeterminada. Puede invalidar la configuración predeterminada deteniendo o iniciando manualmente un grupo de contenedores.

[!INCLUDE [container-instances-restart-ip](../../includes/container-instances-restart-ip.md)]

## <a name="stop"></a>Stop

Detenga manualmente un grupo de contenedores en ejecución, por ejemplo, mediante el comando [az container stop][az-container-stop] o Azure Portal. En ciertas cargas de trabajo de contenedor puede que quiera detener un grupo de contenedores de ejecución prolongada después de un período definido para ahorrar en costos. 

*Cuando un grupo de contenedores entra en estado detenido, finaliza y recicla todos los contenedores del grupo. No conserva el estado de los contenedores.*

Cuando se reciclan los contenedores, los [recursos](container-instances-container-groups.md#resource-allocation) se desasignan y la facturación se detiene para el grupo de contenedores.

La acción de detención no tiene ningún efecto si el grupo de contenedores ya ha finalizado (se encuentre en estado Correcto o Erróneo). Por ejemplo, un grupo de contenedores con tareas de contenedor de una sola ejecución que se haya ejecutado correctamente finaliza con el estado Correcto. Los intentos por detener el grupo en ese estado no cambian el estado. 

## <a name="start"></a>Start

Cuando un grupo de contenedores está detenido, ya sea porque los contenedores finalizaron por sí mismos o porque usted detuvo manualmente el grupo, puede iniciar los contenedores. Por ejemplo, use el comando [az container start][az-container-start] o Azure Portal para iniciar manualmente los contenedores del grupo. Si se actualiza la imagen de contenedor de cualquier contenedor, se extrae una nueva imagen. 

Cuando se inicia un grupo de contenedores comienza una nueva implementación con la misma configuración de contenedor. Esta acción puede ayudarle a volver a utilizar rápidamente una configuración de grupo de contenedores conocida que funciona de la manera prevista. No tiene que crear un nuevo grupo de contenedores para ejecutar la misma carga de trabajo.

Todos los contenedores en un grupo de contenedores se inician mediante esta acción. No se puede iniciar un contenedor específico en el grupo.

Después de iniciar o reiniciar un grupo de contenedores manualmente, este se ejecuta según la configuración de la directiva de reinicio.
  
## <a name="restart"></a>Reinicio

Puede reiniciar un grupo de contenedores mientras se está ejecutando, por ejemplo, con el comando [az container restart][az-container-restart]. Esta acción permite reiniciar todos los contenedores del grupo. Si se actualiza la imagen de contenedor de cualquier contenedor, se extrae una nueva imagen. 

El reinicio de un grupo de contenedores es útil cuando desea solucionar un problema de implementación. Por ejemplo, si una limitación de recursos temporal impide que los contenedores se ejecuten correctamente, reiniciar el grupo puede solucionar el problema.

Todos los contenedores en un grupo de contenedores se reinician mediante esta acción. No se puede reiniciar un contenedor específico en el grupo.

Después de reiniciar un grupo de contenedores manualmente, este se ejecuta según la configuración de la directiva de reinicio.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre la [configuración de la directiva de reinicio](container-instances-restart-policy.md) en Azure Container Instances.

Además de detener e iniciar manualmente un grupo de contenedores con la configuración existente, puede [actualizar la configuración](container-instances-update.md) de un grupo de contenedores en ejecución.

<!-- LINKS - External -->

<!-- LINKS - Internal -->
[az-container-restart]: /cli/azure/container?view=azure-cli-latest#az-container-restart
[az-container-start]: /cli/azure/container?view=azure-cli-latest#az-container-start
[az-container-stop]: /cli/azure/container?view=azure-cli-latest#az-container-stop
