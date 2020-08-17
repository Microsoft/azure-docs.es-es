---
title: Archivos y directorios de Azure Batch
description: Obtenga información sobre archivos y directorios y cómo se usan en un flujo de trabajo de Azure Batch desde el punto de vista del desarrollo.
ms.topic: conceptual
ms.date: 08/03/2020
ms.openlocfilehash: eafea6c234c3b261521f8a791b7a03e25388f02a
ms.sourcegitcommit: 1b2d1755b2bf85f97b27e8fbec2ffc2fcd345120
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87552650"
---
# <a name="files-and-directories-in-azure-batch"></a>Archivos y directorios de Azure Batch

En Azure Batch, cada tarea tiene un directorio de trabajo en el que se pueden crear archivos y directorios. Este directorio de trabajo se puede usar para almacenar el programa que va a ejecuta la tarea, los datos que procesa y el resultado del procesamiento que realiza. Todos los archivos y directorios de una tarea son propiedad del usuario de la tarea.

El servicio Batch expone parte del sistema de archivos en un nodo como *directorio raíz*. Este directorio raíz se encuentra en la unidad de almacenamiento temporal de la máquina virtual, no directamente en la unidad del sistema operativo.

Las tareas pueden acceder al directorio raíz haciendo referencia a la variable de entorno `AZ_BATCH_NODE_ROOT_DIR` . Para obtener más información sobre el uso de variables de entorno, consulte [Configuración del entorno para las tareas](jobs-and-tasks.md#environment-settings-for-tasks).

## <a name="root-directory-structure"></a>Estructura del directorio raíz

El directorio raíz contiene la siguiente estructura de directorio:

![Captura de pantalla de la estructura de directorio del nodo de proceso.](media\files-and-directories\node-folder-structure.png)

- **applications**: contiene información acerca de los detalles de los paquetes de aplicación instalados en el nodo de ejecución. Las tareas pueden acceder a este directorio haciendo referencia a la variable de entorno `AZ_BATCH_APP_PACKAGE` .

- **fsmounts**: el directorio contiene cualquier sistema de archivos montado en un nodo de ejecución. Las tareas pueden acceder a este directorio haciendo referencia a la variable de entorno `AZ_BATCH_NODE_MOUNTS_DIR` . Para más información, consulte [Montaje de un sistema de archivos virtual en un grupo de Batch](virtual-file-mount.md).

- **shared**: este directorio proporciona acceso de lectura y escritura a *todas* las tareas que se ejecutan en un nodo. Toda tarea que se ejecute en el nodo puede crear, leer, actualizar y eliminar archivos en este directorio. Las tareas pueden acceder a este directorio haciendo referencia a la variable de entorno `AZ_BATCH_NODE_SHARED_DIR` .

- **startup**: una tarea de inicio usa este directorio como directorio de trabajo. Aquí se almacenan todos los archivos que la tarea de inicio descarga en el nodo. La tarea de inicio puede crear, leer, actualizar y eliminar archivos en este directorio. Las tareas pueden acceder a este directorio haciendo referencia a la variable de entorno `AZ_BATCH_NODE_STARTUP_DIR` .

- **volatile**: este directorio es para fines internos. No hay ninguna garantía de que los archivos de este directorio o el propio directorio existan en el futuro.

- **workitems**: este directorio contiene los directorios de los trabajos y sus tareas en el nodo de ejecución.

    En el directorio **workitems**, se crea un directorio **Tasks** para cada tarea que se ejecuta en el nodo. Se puede acceder a este directorio haciendo referencia a la variable de entorno `AZ_BATCH_TASK_DIR`.

    Dentro de cada directorio **Tasks**, el servicio Batch crea un directorio de trabajo (`wd`) cuya ruta de acceso único se especifica con la variable de entorno `AZ_BATCH_TASK_WORKING_DIR`. Este directorio proporciona acceso de lectura y escritura a la tarea. La tarea puede crear, leer, actualizar y eliminar archivos en este directorio. Este directorio se conserva en función de la restricción *RetentionTime* especificada para la tarea.

    Los archivos `stdout.txt` y `stderr.txt` se escriben en la carpeta **Tasks** durante la ejecución de esta.

> [!IMPORTANT]
> Cuando se quita un nodo del grupo, se quitan todos los archivos que están almacenados en el nodo.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre el [control y la detección de errores](error-handling.md) en Azure Batch.