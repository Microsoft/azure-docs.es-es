---
title: 'Tutorial: Administración de procesos con Azure Functions'
description: Cómo usar Azure Functions para administrar el proceso del grupo de SQL en Azure Synapse Analytics.
services: synapse-analytics
author: julieMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 04/27/2018
ms.author: jrasnick
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: 60e79ecd4148829c38b237c0e28d60796e84ac01
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543663"
---
# <a name="use-azure-functions-to-manage-compute-resources-in-azure-synapse-analytics-sql-pool"></a>Uso de Azure Functions para administrar recursos de proceso en el grupo de SQL de Azure Synapse Analytics

Cómo usar Azure Functions para administrar los recursos de proceso para un grupo de SQL en Azure Synapse Analytics.

Para poder usar la aplicación Azure Function con el grupo de SQL, debe crear una [cuenta de la entidad de servicio](../../active-directory/develop/howto-create-service-principal-portal.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) con acceso de colaborador en la misma suscripción que la instancia del grupo de SQL.

## <a name="deploy-timer-based-scaling-with-an-azure-resource-manager-template"></a>Implementación de la escala basada en el temporizador con una plantilla de Azure Resource Manager

Para implementar la plantilla, necesitará la siguiente información:

- Nombre del grupo de recursos donde se encuentra el grupo de SQL
- Nombre del servidor en que se encuentra la instancia del grupo de SQL
- Nombre de la instancia del grupo de SQL
- Id. del inquilino (identificación de directorio) de Azure Active Directory
- Id. de suscripción
- Id. de aplicación de la entidad de servicio
- Clave secreta de entidad de servicio

Una vez que tenga la información anterior, implemente esta plantilla:

[![Imagen que muestra un botón con la etiqueta "Implementar en Azure".](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2Fsql-data-warehouse-samples%2Fmaster%2Farm-templates%2FsqlDwTimerScaler%2Fazuredeploy.json)

Una vez implementada la plantilla, encontrará tres nuevos recursos: un plan gratuito de Azure App Service, un consumo basado en el plan de Function App y una cuenta de almacenamiento que controlará el registro y la cola de operaciones. Seguir leyendo las demás secciones para saber cómo modificar las funciones implementadas para que se ajusten a sus necesidades.

## <a name="change-the-compute-level"></a>Cambio del nivel de proceso

1. Vaya a Function App Service. Si ha implementado la plantilla con los valores predeterminados, este servicio debe denominarse *DWOperations*. Una vez se abre Function App, observará cinco funciones implementadas para Function App Service.

   ![Funciones que se implementan con la plantilla](./media/manage-compute-with-azure-functions/five-functions.png)

2. Seleccione *DWScaleDownTrigger* o *DWScaleUpTrigger* dependiendo de si desea cambiar el tiempo para reducir verticalmente o escalar horizontalmente. En el menú desplegable, seleccione Integrar.

   ![Seleccione Integrar por función](./media/manage-compute-with-azure-functions/select-integrate.png)

3. En este momento, el valor mostrado debería indicar *%ScaleDownTime%* o *%ScaleUpTime%* . Estos valores indican que la programación se basa en los valores definidos en la [Configuración de la aplicación](../../azure-functions/functions-how-to-use-azure-function-app-settings.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). Por ahora, puede omitir este valor y cambiar la programación para el momento que prefiera en función de los siguientes pasos.

4. En el área de programación, agregue el tiempo de la expresión CRON que prefiera para reflejar la frecuencia con la que desea que SQL Data Warehouse se reduzca verticalmente.

   ![Cambiar la programación de la función](./media/manage-compute-with-azure-functions/change-schedule.png)

   El valor de `schedule` es una [expresión CRON](https://en.wikipedia.org/wiki/Cron#CRON_expression) que incluye estos seis campos:

   ```json
   {second} {minute} {hour} {day} {month} {day-of-week}
   ```

   Por ejemplo, *"0 30 9 * * 1-5"* reflejaría un desencadenador a las 9:30 a. m. de cada día de la semana. Para más información, visite los [ejemplos de programación](../../azure-functions/functions-bindings-timer.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#example) de Azure Functions.

## <a name="change-the-time-of-the-scale-operation"></a>Cambio de la hora de la operación de escalado

1. Vaya a Function App Service. Si ha implementado la plantilla con los valores predeterminados, este servicio debe denominarse *DWOperations*. Una vez se abre Function App, observará cinco funciones implementadas para Function App Service.

2. Seleccione *DWScaleDownTrigger* o *DWScaleUpTrigger* dependiendo de si desea cambiar el valor del proceso para reducir verticalmente o escalar horizontalmente. Al seleccionar las funciones, el panel debe mostrar el archivo *index.js*.

   ![Cambiar el nivel de proceso del desencadenador de función](././media/manage-compute-with-azure-functions/index-js.png)

3. Cambie el valor de *ServiceLevelObjective* al nivel que desee y guarde. Este valor es el nivel de proceso al que la instancia de almacenamiento de datos escalará según la programación definida en la sección Integrar.

## <a name="use-pause-or-resume-instead-of-scale"></a>Use la pausa o la reanudación en lugar de la escala

Actualmente, las funciones predeterminadas son *DWScaleDownTrigger* y *DWScaleUpTrigger*. Si desea que usar las funcionalidades de pausa o reanudación en su lugar, puede habilitar *DWPauseTrigger* o *DWResumeTrigger*.

1. Navegue hasta el panel Funciones.

   ![Panel Funciones](./media/manage-compute-with-azure-functions/functions-pane.png)

2. Haga clic en el control deslizante de alternancia para habilitar los desencadenadores que desee.

3. Navegue hasta las pestañas *Integrar* pestañas para cambiar la programación de los respectivos desencadenadores.

   > [!NOTE]
   > La diferencia funcional entre los desencadenadores de escalado y los desencadenadores de pausar y reanudar es el mensaje que se envía a la cola. Para obtener más información, consulte [Agregar una nueva función de desencadenador](manage-compute-with-azure-functions.md#add-a-new-trigger-function).

## <a name="add-a-new-trigger-function"></a>Añadir una nueva función de desencadenador

En este momento, hay solo dos funciones de escalado incluidas en la plantilla. Con estas funciones, en el transcurso de un día, solo puede reducir verticalmente y escalar horizontalmente una sola vez. Para un control más granular, como escalar horizontalmente varias veces al día o tener un comportamiento de escalado diferente durante los fines de semana, tendrá que agregar otro desencadenador.

1. Creación de una nueva función en blanco. Seleccione el botón *+* situado cerca de la ubicación Funciones para que aparezca el panel de plantillas de función.

   ![Crear una nueva función](./media/manage-compute-with-azure-functions/create-new-function.png)

2. En Lenguaje, seleccione *Javascript* y, a continuación, seleccione *TimerTrigger*.

   ![Crear una nueva función](./media/manage-compute-with-azure-functions/timertrigger-js.png)

3. Nombre la función y establezca la programación. La imagen muestra cómo se puede desencadenar la función cada sábado a medianoche (viernes por la noche).

   ![Reducir verticalmente el sábado](./media/manage-compute-with-azure-functions/scale-down-saturday.png)

4. Copiar el contenido del *index.js* de una de las demás funciones del desencadenador.

   ![Copiar index.js](././media/manage-compute-with-azure-functions/index-js.png)

5. Establecer la variable de operación para el comportamiento deseado como se indica a continuación:

   ```JavaScript
   // Resume the SQL pool instance
   var operation = {
       "operationType": "ResumeDw"
   }

   // Pause the SQL pool instance
   var operation = {
       "operationType": "PauseDw"
   }

   // Scale the SQL pool instance to DW600
   var operation = {
       "operationType": "ScaleDw",
       "ServiceLevelObjective": "DW600"
   }
   ```

## <a name="complex-scheduling"></a>Programación compleja

En esta sección se muestra brevemente qué es necesario para obtener una programación más compleja de la pausa, la reanudación y las funcionalidades del escalado.

### <a name="example-1"></a>Ejemplo 1

Escalar horizontalmente todos los días a las 8 a.m. a DW600 y reducir verticalmente a las 8 p.m. a DW200.

| Función  | Programación     | Operación                                |
| :-------- | :----------- | :--------------------------------------- |
| Function1 | 0 0 8 * * *  | `var operation = {"operationType": "ScaleDw",    "ServiceLevelObjective": "DW600"}` |
| Function2 | 0 0 20 * * * | `var operation = {"operationType": "ScaleDw", "ServiceLevelObjective": "DW200"}` |

### <a name="example-2"></a>Ejemplo 2

Escalar horizontalmente todos los días a las 8 a. m. a DW1000, reducir verticalmente una vez a DW600 a las 4 p.m. y reducir verticalmente a las 10 p. m. a DW200.

| Función  | Programación     | Operación                                |
| :-------- | :----------- | :--------------------------------------- |
| Function1 | 0 0 8 * * *  | `var operation = {"operationType": "ScaleDw",    "ServiceLevelObjective": "DW1000"}` |
| Function2 | 0 0 16 * * * | `var operation = {"operationType": "ScaleDw", "ServiceLevelObjective": "DW600"}` |
| Function3 | 0 0 22 * * * | `var operation = {"operationType": "ScaleDw", "ServiceLevelObjective": "DW200"}` |

### <a name="example-3"></a>Ejemplo 3

Escalar horizontalmente a las 8 a.m. a DW1000 y reducir verticalmente una vez a DW600 a las 4 p.m. los días laborables. Se pausa el viernes a las 11 p.m., se reanuda a las 7 a.m. del lunes.

| Función  | Programación       | Operación                                |
| :-------- | :------------- | :--------------------------------------- |
| Function1 | 0 0 8 * * 1-5  | `var operation = {"operationType": "ScaleDw",    "ServiceLevelObjective": "DW1000"}` |
| Function2 | 0 0 16 * * 1-5 | `var operation = {"operationType": "ScaleDw", "ServiceLevelObjective": "DW600"}` |
| Function3 | 0 0 23 * * 5   | `var operation = {"operationType": "PauseDw"}` |
| Function4 | 0 0 7 * * 0    | `var operation = {"operationType": "ResumeDw"}` |

## <a name="next-steps"></a>Pasos siguientes

Más información sobre funciones de Azure [desencadenadas mediante temporizador](../../azure-functions/functions-create-scheduled-function.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).

Consulte el [repositorio de ejemplos](https://github.com/Microsoft/sql-data-warehouse-samples) del grupo de SQL.
