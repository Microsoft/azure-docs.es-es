---
title: 'Inicio rápido de Azure: ejecución del primer trabajo de Batch en Azure Portal'
description: Aprenda a utilizar Azure Portal para crear una cuenta de Batch, un grupo de nodos de proceso y un trabajo que ejecute tareas básicas en el grupo.
ms.topic: quickstart
ms.date: 08/17/2020
ms.custom: mvc
ms.openlocfilehash: cf65a681764a848f8132ec44b8ba313ef1a83235
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88511378"
---
# <a name="quickstart-run-your-first-batch-job-in-the-azure-portal"></a>Inicio rápido: ejecute su primer trabajo de Batch en Azure Portal

Para empezar a trabajar con Azure Batch, utilice Azure Portal para crear una cuenta de Batch, un grupo de nodos de proceso (máquinas virtuales) y un trabajo que ejecute tareas en el grupo. Tras completar este inicio rápido, conocerá los conceptos clave del servicio Batch y estará listo para probar dicho servicio con cargas de trabajo más realistas y a mayor escala.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="create-a-batch-account"></a>Crear una cuenta de Batch

Siga estos pasos para crear una cuenta de Batch de ejemplo para hacer pruebas. Necesita una cuenta de Batch para crear grupos y trabajos. Como se muestra aquí, cualquier cuenta de Azure Storage se puede vincular con la cuenta de Batch. Aunque no es necesaria para esta guía de inicio rápido, la cuenta de almacenamiento es útil para implementar aplicaciones y almacenar los datos de entrada y salida de la mayoría de las cargas de trabajo reales.

1. En [Azure Portal](https://portal.azure.com), seleccione **Crear un recurso** > **Proceso** > **Servicio de Batch**. 

   :::image type="content" source="media/quick-create-portal/marketplace-batch.png" alt-text="Captura de pantalla del servicio Batch en Azure Marketplace.":::

1. En el campo **Grupo de recursos**, seleccione **Crear nuevo** y escriba un nombre para el grupo de recursos.

1. Escriba un valor en **Nombre de cuenta**. El nombre debe ser único en la **ubicación** de Azure seleccionada. Solo puede contener letras minúsculas y número, y debe tener entre 3 y 24 caracteres.

1. En **Cuenta de almacenamiento**, seleccione una cuenta de almacenamiento existente o cree una.

1. No cambie los restantes valores. Seleccione **Revisar y crear**, y, después, seleccione **Crear** para crear la cuenta de Batch.

Cuando aparezca el mensaje **Implementación correcta**, vaya a la cuenta de Batch que creó.

## <a name="create-a-pool-of-compute-nodes"></a>Creación de un grupo de nodos de proceso

Ahora que tiene una cuenta de Batch, cree un grupo de ejemplo de nodos de proceso de Windows para realizar pruebas. El grupo de este ejemplo rápido consta de dos nodos en los que se ejecuta una imagen de Windows Server 2019 de Azure Marketplace.

1. En la cuenta de Batch, seleccione **Grupos** > **Agregar**.

1. En **Identificador del grupo**, escriba *mypool*.

1. En **Sistema operativo**, seleccione la siguiente configuración (puede explorar otras opciones).
  
   |Configuración  |Value  |
   |---------|---------|
   |**Tipo de imagen**|Marketplace|
   |**Publicador**     |microsoftwindowsserver|
   |**Oferta**     |windowsserver|
   |**Sku**     |2019-datacenter-core-smalldisk|

1. Desplácese hacia abajo para especificar los valores de **Tamaño de nodo** y **Escala**. El tamaño del nodo que se sugiere ofrece un buen equilibrio entre rendimiento y costo para este ejemplo rápido.
  
   |Configuración  |Value  |
   |---------|---------|
   |**Plan de tarifa del nodo**     |Estándar A1|
   |**Nodos dedicados de destino**     |2|

1. En el resto mantenga los valores predeterminados y seleccione **Aceptar** para crear el grupo.

Batch crea el grupo inmediatamente, pero tarda unos minutos en asignar e iniciar los nodos de proceso. Durante de este tiempo, el valor de **Estado de asignación** del grupo es **Cambio de tamaño**. Puede crear un trabajo y tareas mientras se cambia el tamaño del grupo.

Tras unos minutos, el estado de asignación cambia a **Estable** y se inician los nodos. Para comprobar el estado de los nodos, seleccione el grupo y, después, seleccione **Nodos**. Cuando el estado de un nodo es **Inactivo**, está listo para ejecutar las tareas.

## <a name="create-a-job"></a>Creación de un trabajo

Ahora que tiene un grupo, cree un trabajo para que se ejecute en él. Un trabajo de Batch es un grupo lógico de una o varias tareas. Un trabajo incluye valores comunes para las tareas, como la prioridad y el grupo en el que se ejecutan las tareas. Inicialmente, el trabajo no tiene tareas.

1. En la vista de cuentas de Batch, seleccione **Trabajos** > **Agregar**.

1. En **Id. de trabajo**, escriba *myjob*. En **Grupo**, seleccione *mypool*. Mantenga los valores predeterminados de los valores de configuración restantes y seleccione **Aceptar**.

## <a name="create-tasks"></a>Creación de tareas

Ahora, seleccione el trabajo para abrir la página **Tareas**. Aquí es donde va a crear las tareas de ejemplo que se van a ejecutar en el trabajo. Lo habitual es que cree varias tareas que Batch se encarga de poner en cola y distribuir para ejecutarse en los nodos de proceso. En este ejemplo se crean dos tareas idénticas. En cada una de ellas se ejecuta una línea de comandos que muestra las variables de entorno de Batch de un nodo de proceso y, después, espera 90 segundos.

Cuando se usa Batch, la línea de comandos es el lugar en el que se especifica la aplicación o el script. Batch proporciona varias formas de implementar aplicaciones y scripts en nodos de proceso.

Para crear la primera tarea:

1. Seleccione **Agregar**.

1. En **Id. de tarea**, escriba *mytask*.

1. En **Línea de comandos**, escriba `cmd /c "set AZ_BATCH & timeout /t 90 > NUL"`. Mantenga los valores predeterminados en el resto de las opciones y seleccione **Enviar**.

Después de crear una tarea, Batch la pone en cola para ejecutarla en el grupo. Cuando un nodo esté disponible para ejecutarla, la tarea se ejecuta.

Para crear una segunda tarea, repita los pasos anteriores. Escriba otro valor en **Id. de tarea**, pero especifique la misma línea de comandos. Si la primera tarea se sigue ejecutando, Batch inicia la segunda tarea en el otro nodo del grupo.

## <a name="view-task-output"></a>Visualización de la salida de la tarea

Las tareas de ejemplo que ha creado se completarán en un par de minutos. Para ver la salida de una tarea completada, seleccione la tarea y, después, seleccione **Archivos en el nodo**. Seleccione el archivo `stdout.txt` para ver la salida estándar de la tarea. El contenido es similar al siguiente:

:::image type="content" source="media/quick-create-portal/task-output.png" alt-text="Captura de pantalla de la salida de una tarea completada.":::

El contenido muestra las variables de entorno de Azure Batch que se definen en el nodo. Al crear sus propios trabajos y tareas de Batch, puede hacer referencia a estas variables de entorno tanto en las líneas de comandos de la tarea como en las aplicaciones y los scripts que ejecutan las líneas de comandos.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si desea continuar con los ejemplos y tutoriales de Batch, utilice la cuenta de Batch y la cuenta de almacenamiento vinculada que se ha creado en esta guía de inicio rápido. Por la propia cuenta de Batch no se cobra nada.

Se cobran el grupo mientras que los nodos estén en ejecución, aunque no haya trabajos programados. Cuando no necesite el grupo, elimínelo. En la vista de cuenta, seleccione **Grupos** y el nombre del grupo. A continuación, seleccione **Eliminar**.  Al eliminar el grupo, las salidas de tarea de los nodos también se eliminan.

Cuando no los necesite, elimine el grupo de recursos, la cuenta de Batch y todos los recursos relacionados. Para ello, seleccione el grupo de recursos de la cuenta de Batch y seleccione **Eliminar grupo de recursos**.

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado una cuenta de Batch, un grupo de Batch y un trabajo de Batch. El trabajo ha ejecutado tareas de ejemplo y se ha visto la salida que se ha creado en uno de los nodos. Ahora que conoce los conceptos clave del servicio Batch, ya esta listo para probar dicho servicio con cargas de trabajo más realistas y a mayor escala. Para más información acerca de Azure Batch, continúe con los tutoriales de Azure Batch.

> [!div class="nextstepaction"]
> [Tutoriales de Azure Batch](./tutorial-parallel-dotnet.md)
