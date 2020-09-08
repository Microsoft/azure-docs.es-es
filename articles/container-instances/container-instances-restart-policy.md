---
title: Reiniciar de la directiva para tareas de una sola ejecución
description: Aprenda a usar Azure Container Instances para ejecutar tareas hasta su terminación, como compilaciones, pruebas o trabajos de representación de imágenes.
ms.topic: article
ms.date: 08/11/2020
ms.openlocfilehash: 336a31a03cdc9dfdfebe79ef47b59ef90053f523
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88798948"
---
# <a name="run-containerized-tasks-with-restart-policies"></a>Ejecución de tareas en contenedores con directivas de reinicio

La facilidad y rapidez con que se implementan contenedores convierten a Azure Container Instances en una plataforma convincente para la ejecución de tareas que se ejecutan una única vez, como compilaciones, pruebas y representaciones de imágenes en una instancia de contenedor.

Gracias a una directiva de reinicio configurable, puede especificar que los contenedores se detengan cuando sus procesos se hayan completado. Dado que las instancias de contenedor se facturan por segundo, solo se le cobra por los recursos de proceso usados mientras el contenedor que ejecuta su tarea está en funcionamiento.

En los ejemplos presentados en este artículo se usa la CLI de Azure. Debe tener [instalada localmente][azure-cli-install] la CLI de Azure, versión 2.0.21 o superior, o usar la CLI de [Azure Cloud Shell](../cloud-shell/overview.md).

## <a name="container-restart-policy"></a>Directiva de reinicio de contenedor

Cuando se crea un [grupo de contenedores](container-instances-container-groups.md) en Azure Container Instances, se puede especificar una de tres configuraciones de directiva de reinicio.

| Directiva de reinicio   | Descripción |
| ---------------- | :---------- |
| `Always` | Siempre se reinician los contenedores del grupo de contenedores. Este es el valor de configuración **predeterminado** aplicado cuando no se especifica ninguna directiva de reinicio durante la creación del contenedor. |
| `Never` | Nunca se reinician los contenedores del grupo de contenedores. Los contenedores se ejecutan al menos una vez. |
| `OnFailure` | Los contenedores del grupo de contenedores se reinician solo cuando se produce un error en el proceso ejecutado en el contenedor (cuando se cierra con un código de salida distinto de cero). Los contenedores se ejecutan al menos una vez. |

[!INCLUDE [container-instances-restart-ip](../../includes/container-instances-restart-ip.md)]

## <a name="specify-a-restart-policy"></a>Especificación de una directiva de reinicio

El modo en que especifique una directiva de reinicio depende de cómo cree sus instancias de contenedor, es decir, con la CLI de Azure, los cmdlets de Azure PowerShell o Azure Portal. En la CLI de Azure, especifique el parámetro `--restart-policy` cuando llame a [az container create][az-container-create].

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainer \
    --image mycontainerimage \
    --restart-policy OnFailure
```

## <a name="run-to-completion-example"></a>Ejemplo de ejecución hasta terminación

Para ver la directiva de reinicio en acción, cree una instancia de contenedor a partir de la imagen [aci-wordcount][aci-wordcount-image] de Microsoft y especifique la directiva de reinicio `OnFailure`. Este contenedor de ejemplo ejecuta un script de Python que, de forma predeterminada, analiza el texto de la obra [Hamlet](http://shakespeare.mit.edu/hamlet/full.html) de Shakespeare, escribe las 10 palabras más comunes en STDOUT y, a continuación, se cierra.

Ejecute el contenedor de ejemplo con el siguiente comando [az container create][az-container-create]:

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainer \
    --image mcr.microsoft.com/azuredocs/aci-wordcount:latest \
    --restart-policy OnFailure
```

Azure Container Instances inicia el contenedor y, luego, lo detiene cuando su aplicación (o script, en este caso) se cierra. Cuando Azure Container Instances detiene un contenedor cuya directiva de reinicio es `Never` o `OnFailure`, el estado del contenedor se establece en **Terminado**. Puede comprobar el estado de un contenedor con el comando [az container show][az-container-show]:

```azurecli-interactive
az container show \
    --resource-group myResourceGroup \
    --name mycontainer \
    --query containers[0].instanceView.currentState.state
```

Salida de ejemplo:

```bash
"Terminated"
```

Una vez que el estado del contenedor de ejemplo muestra *Terminado*, puede ver la salida de las tareas en los registros del contenedor. Ejecute el comando [az container logs][az-container-logs] para ver la salida del script:

```azurecli-interactive
az container logs --resource-group myResourceGroup --name mycontainer
```

Salida:

```bash
[('the', 990),
 ('and', 702),
 ('of', 628),
 ('to', 610),
 ('I', 544),
 ('you', 495),
 ('a', 453),
 ('my', 441),
 ('in', 399),
 ('HAMLET', 386)]
```

En este ejemplo se muestra la salida que el script envía a STDOUT. No obstante, puede que las tareas en contenedor escriban su salida en almacenamiento persistente para su posterior recuperación. Por ejemplo, en un [recurso compartido de archivos de Azure](./container-instances-volume-azure-files.md).

## <a name="next-steps"></a>Pasos siguientes

Los escenarios basados en tareas, como el procesamiento por lotes de un gran conjunto de datos con varios contenedores, pueden aprovechar las [variables de entorno](container-instances-environment-variables.md) o las [líneas de comando](container-instances-start-command.md) personalizadas en tiempo de ejecución.

Para más información acerca de cómo conservar la salida de los contenedores que se ejecutan hasta completarse, consulte [Montaje de un recurso compartido de archivos de Azure en Azure Container Instances](./container-instances-volume-azure-files.md).

<!-- LINKS - External -->
[aci-wordcount-image]: https://hub.docker.com/_/microsoft-azuredocs-aci-wordcount

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container?view=azure-cli-latest#az-container-create
[az-container-logs]: /cli/azure/container?view=azure-cli-latest#az-container-logs
[az-container-show]: /cli/azure/container?view=azure-cli-latest#az-container-show
[azure-cli-install]: /cli/azure/install-azure-cli
