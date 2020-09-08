---
title: Programación de los trabajos
description: Utilice la programación de trabajos para administrar las tareas.
ms.topic: how-to
ms.date: 02/20/2020
ms.custom: seodec18
ms.openlocfilehash: 7da3c78e00f5d7e41a5396603cf4885a50cb6e5c
ms.sourcegitcommit: d7352c07708180a9293e8a0e7020b9dd3dd153ce
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2020
ms.locfileid: "89146358"
---
# <a name="schedule-jobs-for-efficiency"></a>Programación de trabajos para mejorar la eficacia

La programación de trabajos de Batch le permite priorizar los trabajos que desea ejecutar en primer lugar y tener en cuenta las tareas que tienen dependencias en otras tareas. Mediante la programación de los trabajos, puede asegurarse de que se usa la menor cantidad de recursos. Los nodos se pueden retirar cuando no son necesarios, las tareas que dependen de otras tareas se inician Just-in-time para optimizar los flujos de trabajo. Solo se ejecuta un trabajo cada vez. No se iniciará uno nuevo hasta que se complete el anterior. Puede establecer el trabajo en modo Autocompletar. 

## <a name="benefit-of-job-scheduling"></a>Ventajas de la programación de trabajos

La ventaja de la programación de trabajos es que se puede especificar una programación para la creación de trabajos. Las tareas que se programan mediante la tarea del administrador de trabajos están asociadas a un trabajo. La tarea del administrador de trabajos creará las tareas para el trabajo. Para ello, la tarea del administrador de trabajos debe autenticarse con la cuenta de Batch. Use el token de acceso AZ_BATCH_AUTHENTICATION_TOKEN. El token le permitirá el acceso al resto del trabajo. 

## <a name="use-the-portal-to-schedule-a-job"></a>Uso del portal para programar un trabajo

   1. Inicie sesión en el [portal de Azure](https://portal.azure.com/).

   2. Seleccione la cuenta de Batch en la que desea programar los trabajos.

   3. Seleccione **Agregar** para crear una nueva programación de trabajo y rellene el **formulario básico**.



![Programación de un trabajo][1]

**Identificador de programación de trabajo**: identificador único de esta programación de trabajo.

**Nombre para mostrar**: no es necesario que el nombre para mostrar del trabajo sea único, pero tiene una longitud máxima de 1024 caracteres.

**No ejecutar hasta**: especifica la hora más temprana a la que se ejecutará el trabajo. Si no establece esta configuración, la programación está lista para ejecutar trabajos inmediatamente.

**No ejecutar después de**: no se ejecutarán trabajos después de la hora que establezca aquí. Si no se especifica una hora, se crea una programación de trabajo periódica que permanece activa hasta que se finalice explícitamente.

**Intervalo de periodicidad**: puede especificar la cantidad de tiempo entre trabajos. Solo puede tener un trabajo programado a la vez, por lo que si es el momento de crear un nuevo trabajo en una programación de trabajo, pero el trabajo anterior todavía está en ejecución, el servicio Batch no creará el nuevo trabajo hasta que finalice el trabajo anterior.  

**Ventana de inicio**: aquí se especifica el intervalo de tiempo, a partir de la hora en que la programación indica que se debe crear un trabajo, hasta que se debe completar. Si el trabajo actual no se completa durante su ventana, no se iniciará el siguiente trabajo.

En la parte inferior del formulario básico, debe especificar el grupo en el que desea que se ejecute el trabajo. Para buscar la información del identificador de grupo, seleccione **Actualizar**. 

![Especificar grupo][2]


**Identificador de grupo**: especifique el grupo en el que se va a ejecutar el trabajo.

**Tarea de configuración del trabajo**: seleccione **Actualizar** para asignar un nombre a la tarea del administrador de trabajos, así como a las tareas de preparación y liberación del trabajo, si las usa.

**Prioridad**: asigne una prioridad al trabajo.

**Tiempo de reloj máximo**: establezca la cantidad máxima de tiempo que el trabajo puede estar en ejecución. Si no se completa dentro del período de tiempo, Batch finaliza el trabajo. Si no establece esta configuración, no hay límite de tiempo para el trabajo.

**Número máximo de reintentos de tareas**: especifique el número de veces que se puede volver a intentar una tarea, hasta un máximo de cuatro veces. Esto no es lo mismo que el número de reintentos que puede tener una llamada de API.

**Cuando se completan todas las tareas**: El valor predeterminado es ninguna acción.

**Cuando se produce un error en una tarea**: El valor predeterminado es ninguna acción. Se produce un error en una tarea si se agota el número de reintentos o se produce un error al iniciar la tarea. 

Después de seleccionar **Guardar**, si va a **Programaciones de trabajos** en el panel de navegación izquierdo, puede realizar un seguimiento de la ejecución del trabajo; para ello, seleccione **Información de ejecución**.


## <a name="for-more-information"></a>Para obtener más información

Para administrar un trabajo mediante la CLI de Azure, consulte [az batch job-schedule](/cli/azure/batch/job-schedule).

## <a name="next-steps"></a>Pasos siguientes

[Creación de dependencias de tareas para ejecutar las tareas que dependan de otras tareas](batch-task-dependencies.md).





[1]: ./media/batch-job-schedule/add_job_schedule-02.png
[2]: ./media/batch-job-schedule/add_job_schedule-03.png


