---
title: Recurso compartido de archivos de Azure para grupos de Azure Batch
description: Cómo montar un recurso compartido de archivos de Azure a partir de nodos de proceso en un grupo de Linux o Windows en Azure Batch.
ms.topic: how-to
ms.date: 05/24/2018
ms.openlocfilehash: 6bbaba20512a17de563e74ba095057c5c3f71f6b
ms.sourcegitcommit: 25bb515efe62bfb8a8377293b56c3163f46122bf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87986430"
---
# <a name="use-an-azure-file-share-with-a-batch-pool"></a>Uso de un recurso compartido de archivos con un grupo de Batch

[Azure Files](../storage/files/storage-files-introduction.md) ofrece recursos compartidos de archivos completamente administrados en la nube, a los que se puede acceder mediante el protocolo de Bloque de mensajes del servidor (SMB). En este artículo se proporciona información y ejemplos de código para montar y usar un recurso compartido de archivos de Azure en un grupo de nodos de proceso.

## <a name="considerations-for-use-with-batch"></a>Consideraciones sobre el uso con Batch

* Podría usar un recurso compartido de archivos de Azure cuando tiene grupos que ejecutan un número de tareas en paralelo relativamente bajo si usa Azure Files no Premium. Revise los [objetivos de rendimiento y escala](../storage/files/storage-files-scale-targets.md) para determinar si se debe usar Azure Files (que usa una cuenta de Azure Storage), dado el tamaño esperado del grupo y el número de archivos de recursos. 

* Los recursos compartidos de archivos de Azure son [rentables](https://azure.microsoft.com/pricing/details/storage/files/) y se pueden configurar con la replicación de datos en otra región, así que son redundantes de forma global. 

* Se puede montar al mismo tiempo un recurso compartido de archivos de Azure desde un equipo local. Sin embargo, asegúrese de que comprende las [implicaciones de simultaneidad](../storage/common/storage-concurrency.md) especialmente al usar las API REST.

* Consulte también las [consideraciones de planeación](../storage/files/storage-files-planning.md) sobre los recursos compartidos de archivos de Azure.


## <a name="create-a-file-share"></a>Creación de un recurso compartido de archivos

[Cree un recurso compartido de archivos](../storage/files/storage-how-to-create-file-share.md) en una cuenta de almacenamiento que esté vinculada a la cuenta Batch o en una cuenta de almacenamiento independiente.

## <a name="mount-an-azure-file-share-on-a-batch-pool"></a>Montaje de un recurso compartido de archivos de Azure con un grupo de Batch

Consulte la documentación sobre cómo [montar un sistema de archivos virtual en un grupo de Batch](virtual-file-mount.md).

## <a name="next-steps"></a>Pasos siguientes

* Para obtener información sobre otras opciones de lectura y escritura de datos en Batch, consulte [Trabajo persistente y resultado de la tarea](batch-task-output.md).
* Consulte también el kit de herramientas [Batch Shipyard](https://github.com/Azure/batch-shipyard), que incluye [recetas de Shipyard](https://github.com/Azure/batch-shipyard/tree/master/recipes) para implementar sistemas de archivos para cargas de trabajo de contenedor de Batch.
