---
title: Representación de una escena de Blender con Azure Batch y Batch Explorer
description: 'Tutorial: Representación de varios fotogramas de una escena de Blender mediante la aplicación cliente de Azure Batch y Batch Explorer'
author: mscurrell
ms.author: markscu
ms.date: 08/02/2018
ms.topic: tutorial
ms.openlocfilehash: 65baff827417cebe2289cc821df94acd38a1ae61
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "85964811"
---
# <a name="tutorial-render-a-blender-scene-using-batch-explorer"></a>Tutorial: Representación de una escena de Blender con Batch Explorer

Este tutorial muestra cómo representar varios fotogramas de una escena de demostración de Blender. Blender se usa en este tutorial ya que es gratuito para el cliente y para las máquinas virtuales de representación, pero el proceso es muy parecido si se usaran otras aplicaciones como Maya o 3ds Max.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Cargar una escena de Blender en Azure Storage
> * Crear un grupo de Batch con varios nodos para realizar la representación
> * Representar varios fotogramas
> * Ver y descargar los archivos de fotogramas representados

## <a name="prerequisites"></a>Prerequisites

Necesita una suscripción de pago por uso u otra opción de compra de Azure para usar aplicaciones de representación en Batch sobre una base de pago por uso. Las licencias de pago por uso no se admiten si usa una oferta gratuita de Azure que proporciona un crédito monetario.

Necesita una cuenta de Azure Batch con una cuenta de almacenamiento asociada.  Consulte cualquiera de las guías de inicio rápido de Batch como el [artículo sobre la CLI](./quick-create-cli.md) para crear una cuenta de Batch.

Se requiere una cuota de núcleos de baja prioridad de al menos 50 núcleos para el tamaño de máquina virtual y el número de máquinas virtuales especificado en este tutorial. Se puede usar la cuota predeterminada, pero tendrá que usar un tamaño de máquina virtual más pequeño, lo cual hará que se tarde más tiempo en representar las imágenes. El proceso para solicitar una mayor cuota de núcleos se detalla en [este artículo](./batch-quota-limit.md).

Finalmente, se debe instalar [Batch Explorer](https://azure.github.io/BatchExplorer/). Está disponible para Windows, OSX y Linux. Es opcional, pero si [Blender](https://www.blender.org/download/) está instalado, se puede ver el archivo de modelo de ejemplo.

## <a name="download-the-demo-scene"></a>Descarga de la escena de demostración

Descargue el archivo ZIP de demostración denominado "Class room" de la [página web de archivos de demostración de Blender](https://www.blender.org/download/demo-files/) y descomprímalo en una unidad local.

## <a name="upload-a-scene-to-azure-storage"></a>Cargar una escena en Azure Storage

Cree un contenedor de cuentas de almacenamiento para los archivos de la escena de demostración:

* Inicie Batch Explorer.
* En el menú principal del lado izquierdo, seleccione el elemento de menú "Datos".
* Asegúrese de que "Grupos de archivos" está seleccionado en la lista desplegable.
* Seleccione el botón "+" y cree un nuevo "grupo de archivos vacío" denominado "blender-classroom"
  * Un grupo de archivos es simplemente un contenedor de blobs de Azure Storage con un prefijo "fgrp-"; es una convención que se usa para filtrar los otros contenedores de la cuenta de almacenamiento

![Grupo de archivos para archivos de escenas](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_scene_filegroup.png)

Cargue los archivos de escenas:

* Seleccione el nuevo contenedor y arrastre y coloque el contenido de la carpeta "classroom" en el contenedor de Batch Explorer.

![Archivos de escena cargados](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_scene_filegroup_uploaded.png)

## <a name="create-azure-storage-container-for-output-images"></a>Creación del contenedor de Azure Storage para las imágenes de salida

Cree un contenedor de cuentas de almacenamiento para los archivos de salida de la escena de demostración:

* En el menú principal del lado izquierdo, seleccione el elemento de menú "Datos".
* Seleccione el botón "+" y cree un nuevo "grupo de archivos vacío" denominado "render-output"

![Grupo de archivos para archivos de salida](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_output_filegroup.png)

## <a name="create-a-pool-of-vms-for-rendering"></a>Creación de un grupo de máquinas virtuales para representación

Cree un grupo de Batch mediante la representación de imágenes de máquina virtual de Azure Marketplace que contiene la aplicación Blender:

* En el menú principal del lado izquierdo, seleccione el elemento de menú "Galería".
* Seleccione el elemento 'Blender' para obtener la lista de elementos de la aplicación.
* Seleccione los elementos para representar los fotogramas en Windows Server.
* Opcionalmente, se puede seleccionar el icono de vínculo en el lado derecho del elemento para ver los archivos de plantilla que se usarán para crear un grupo y un trabajo.

![Elementos de la galería de Blender](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_gallery_item.png)

* Seleccione el botón "Create pool for later use" (Crear grupo para su uso posterior)
  * Mantenga el nombre del grupo como "blender-windows"
  * Establezca "Dedicated vm count" (Recuento de VM dedicadas) en "0"
  * Establezca "Low priority vm count" (Recuento de VM de prioridad baja) en "3"
  * Establezca "Tamaño del nodo" en "Standard_F16": se puede seleccionar otro tamaño de máquina virtual, pero tenga en cuenta que el tiempo de representación de un fotograma dependerá principalmente del número de núcleos.
* Seleccione el botón verde para crear el grupo
  * El grupo se creará casi de inmediato, pero se tardará algunos minutos en asignar e iniciar las máquinas virtuales.

![Plantilla de grupo de Blender](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_pool_template.png)

> [!WARNING]
> Tenga en cuenta que cuando hay máquinas virtuales en un grupo, el costo de esas máquinas virtuales se cobra a su suscripción de Azure. Deberá eliminar el grupo o las máquinas virtuales para detener los gastos. Elimine el grupo como se indica al final de este tutorial para evitar gastos continuos.

Se puede supervisar el estado del grupo y de las máquinas virtuales en la vista "Grupos". En el siguiente ejemplo se muestra que se han asignado las tres máquinas virtuales, se han iniciado dos y están inactivas y una se está todavía iniciando: ![Mapa térmico del grupo](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_pool_heatmap.png)

## <a name="create-a-rendering-job"></a>Creación de un trabajo de representación

Cree un trabajo de representación para representar algunos fotogramas mediante el grupo que creó:
* En el menú principal del lado izquierdo, seleccione el elemento de menú "Galería".
* Seleccione el elemento 'Blender' para obtener la lista de elementos de la aplicación.
* Seleccione los elementos para representar los fotogramas en Windows Server.
* Seleccione el botón "Run job with existing pool" (Ejecutar trabajo con el grupo existente)
* Seleccione el grupo "blender-windows"
* Establezca el "nombre del trabajo" en "blender-render-tutorial1"
* Seleccione "fgrp-blender-classroom" como "datos de entrada"
* Seleccione el icono de archivo del archivo "Blend" y seleccione "classroom.blend"
* Mantenga el "Fotograma de inicio" en "1" y establezca el "fotograma de final" en "5"
* Establezca las "salidas" en "fgrp-render-output"
* Seleccione el botón verde para crear el trabajo; se creará un trabajo con cinco tareas, una para cada fotograma

![Plantilla de trabajo de Blender](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_job_template.png)

Una vez que se ha creado el trabajo y todas las tareas, el trabajo aparecerá junto con las tareas: ![Lista de tareas del trabajo](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_task_list.png)

Cuando una tarea empieza a ejecutarse por primera vez en una máquina virtual de un grupo, se ejecuta una tarea de preparación de trabajo por lotes que copia los archivos de escena del grupo de archivos de almacenamiento en la máquina virtual para que Blender pueda acceder a ellos.
El estado de la representación se puede determinar visualizando el archivo de registro stdout.txt que genera Blender.  Seleccione una tarea, aparecerán las "salidas de la tarea" de forma predeterminada y se puede seleccionar el archivo "stdout.txt" y visualizarlo.
![archivo stdout](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_stdout.png)

Si se selecciona el grupo "blender-windows", el grupo de máquinas virtuales aparecerá en un estado de ejecución: ![Mapa térmico del grupo con los nodos en ejecución](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_pool_heatmap_running.png)

Las imágenes representadas tardarán varios minutos en generarse según el tamaño de máquina virtual seleccionado.  Con el uso de la máquina virtual F16 especificada anteriormente, los fotogramas tardaron aproximadamente 16 minutos en representarse.

## <a name="view-the-rendering-output"></a>Visualización de la salida de representación

Cuando los fotogramas terminan de representarse, esas tareas aparecen como completadas: ![Finalización de tareas](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_tasks_complete.png)

La imagen representada se escribe primero en la máquina virtual y se puede visualizar seleccionando la carpeta "wd": ![Imagen representada en el nodo de grupo](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_output_image.png)

La plantilla de trabajo especifica también que el fotograma de salida y los archivos de registro se escriban de nuevo en un grupo de archivos de la cuenta de Azure Storage que especificó al crear el trabajo.  La interfaz de usuario de "Datos" puede usarse para ver los registros y archivos de salida. También se puede usar para descargar los archivos: ![Imagen representada en un grupo de archivos de almacenamiento](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_output_image_storage.png)

Cuando se hayan completado todas las tareas, el trabajo se marcará como completo: ![Trabajo y tareas completadas](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_job_alltasks_complete.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

> [!WARNING]
> Se debe eliminar el grupo (también se puede cambiar de tamaño a cero nodos) para evitar que los gastos se acumulen en la suscripción de Azure.

* Seleccione "Grupos"
* Seleccione el grupo "blender-windows"
* Haga clic con el botón derecho y seleccione "Eliminar", o bien seleccione el icono de papelera situado encima del grupo

## <a name="next-steps"></a>Pasos siguientes
* En la sección "Galería", explore las aplicaciones de representación disponibles a través de Batch Explorer.
* Para cada aplicación, hay varias plantillas disponibles, que se expandirán con el tiempo.  Por ejemplo, para Blender existen plantillas que dividen una sola imagen en mosaicos, por lo que las partes de una imagen se pueden representar en paralelo.
* Para obtener una descripción completa de las funcionalidades de representación, consulte el conjunto de artículos de [aquí](./batch-rendering-service.md).
