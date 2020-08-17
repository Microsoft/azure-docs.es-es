---
title: Solución de problemas de Azure Stream Analytics mediante registros de recursos
description: En este artículo se describe cómo analizar los registros de recursos en Azure Stream Analytics.
author: jseb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: troubleshooting
ms.date: 06/18/2020
ms.openlocfilehash: 6d0a778dee31d93244479c08c7bb7b6f37cf49cb
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87319361"
---
# <a name="troubleshoot-azure-stream-analytics-by-using-resource-logs"></a>Solución de problemas de Azure Stream Analytics mediante registros de recursos

En ocasiones, un trabajo de Azure Stream Analytics deja de procesarse inesperadamente. Es importante poder solucionar este tipo de evento. Los errores pueden deberse a un resultado de consulta inesperado, la conectividad a los dispositivos o una interrupción inesperada del servicio. Los registros de recursos de Stream Analytics pueden ayudarle a identificar la causa de los problemas cuando se producen y a reducir el tiempo de recuperación.

Es muy recomendable habilitar los registros de recursos para todos los trabajos, ya que esto contribuirá en gran medida a la depuración y la supervisión.

## <a name="log-types"></a>Tipos de registro

Stream Analytics ofrece dos tipos de registros:

* [Registros de actividad](../azure-monitor/platform/platform-logs-overview.md) (siempre activados), que proporcionan información sobre las operaciones realizadas en los trabajos.

* [Registros de recursos](../azure-monitor/platform/platform-logs-overview.md) (configurables), que proporcionan información más completa sobre todo lo que ocurre con un trabajo. Comienzan cuando se crea el trabajo y finalizan cuando se elimina el trabajo. Abarcan los eventos de cuando se actualiza el trabajo y mientras se está ejecutando.

> [!NOTE]
> Puede usar servicios como, por ejemplo, Azure Storage, Azure Event Hubs y registros de Azure Monitor, para analizar los datos no conformes. Se le cobra según el modelo de precios existente para esos servicios.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="debugging-using-activity-logs"></a>Depuración con registros de actividad

Los registros de actividad están activados de forma predeterminada y proporcionan conclusiones de alto nivel sobre las operaciones realizadas por el trabajo de Stream Analytics. La información presente en los registros de actividad puede ayudar a encontrar la causa principal de los problemas que afectan a su trabajo. Siga los pasos a continuación para usar los registros de actividad en Stream Analytics:

1. Inicie sesión en Azure Portal y seleccione **Registro de actividad** en **Introducción**.

   ![Registro de actividad de Stream Analytics](./media/stream-analytics-job-diagnostic-logs/stream-analytics-menu.png)

2. Puede ver una lista de operaciones que se han realizado. Toda operación que haya producido un error en el trabajo tiene una burbuja de información de color rojo.

3. Haga clic en una operación para ver su vista de resumen. Esta información suele ser limitada. Para más información acerca de la operación, haga clic en **JSON**.

   ![Resumen de la operación de registro de actividad de Stream Analytics](./media/stream-analytics-job-diagnostic-logs/operation-summary.png)

4. Desplácese hacia abajo hasta la sección **Propiedades** del código JSON, que proporciona los detalles del error que provocó el error de la operación. En este ejemplo, el error se produjo debido a un error en tiempo de ejecución por valores fuera de los límites de latitud. Una discrepancia en los datos que se procesan mediante un trabajo de Stream Analytics produce un error de datos. Puede obtener información sobre diferentes [errores de entrada y salida de datos y por qué se producen](https://docs.microsoft.com/azure/stream-analytics/data-errors).

   ![Detalles del error en JSON](./media/stream-analytics-job-diagnostic-logs/error-details.png)

5. Puede tomar acciones correctivas según el mensaje de error en JSON. En este ejemplo, deben agregarse a la consulta comprobaciones para asegurar que el valor de latitud esté entre -90 grados y 90 grados.

6. Si el mensaje de error de los registros de actividad no ayudan a detectar la causa principal, habilite los registros de recursos y use los registros de Azure Monitor.

## <a name="send-diagnostics-to-azure-monitor-logs"></a>Envío de diagnósticos a los registros de Azure Monitor

Es muy recomendable activar los registros de recursos y enviarlos a los registros de Azure Monitor. De forma predeterminada están **desactivados**. Para activarlos, siga estos pasos:

1.  Cree un área de trabajo de Log Analytics, en caso de que no tenga una. Se recomienda disponer de un área de trabajo de Log Analytics en la misma región que el trabajo de Stream Analytics.

2.  Inicie sesión en Azure Portal y vaya al trabajo de Stream Analytics. En **Supervisión**, seleccione **Registros de diagnóstico**. Después, seleccione **Activar diagnósticos**.

    ![Exploración de los registros de recursos en la hoja](./media/stream-analytics-job-diagnostic-logs/diagnostic-logs-monitoring.png)  

2.  Proporcione un **nombre** en el campo **Nombre de configuración de diagnóstico** y active las casillas **Ejecución** y **Creación** en **registro**, y **AllMetrics** en **métrica**. Después, seleccione **Enviar a Log Analytics** y elija su área de trabajo. Haga clic en **Save**(Guardar).

    ![Configuración de registros de recursos](./media/stream-analytics-job-diagnostic-logs/logs-setup.png)

3. Cuando se inicia el trabajo de Stream Analytics, los registros de recursos se enrutan a su área de trabajo de Log Analytics. Para ver los registros de recursos del trabajo, seleccione **Registros** en la sección **Supervisión**.

   ![Registros de recursos en Supervisión](./media/stream-analytics-job-diagnostic-logs/diagnostic-logs.png)

4. Stream Analytics proporciona consultas predefinidas que le permiten buscar fácilmente los registros que le interesan. Puede seleccionar las consultas predefinidas en el panel izquierdo y, a continuación, seleccionar **Ejecutar**. Verá los resultados de la consulta en el panel inferior. 

   ![Registros de recursos en Supervisión](./media/stream-analytics-job-diagnostic-logs/logs-example.png)

## <a name="resource-log-categories"></a>Categorías del registro de recursos

Azure Stream Analytics captura dos categorías de registros de recursos:

* **Creación**: Captura registros de eventos relacionados con operaciones de creación de trabajos, como creación de trabajos, adición y eliminación de entradas y salidas, adición y actualización de la consulta, e inicio y detención del trabajo.

* **Ejecución**: Captura los eventos que se producen durante la ejecución del trabajo.
    * Errores de conectividad
    * Errores de procesamiento de datos, como por ejemplo:
        * Eventos que no se ajustan a la definición de consulta (tipos y valores de campo no coincidentes, campos ausentes, etc.)
        * Errores de evaluación de expresión
    * Otros eventos y errores

## <a name="resource-logs-schema"></a>Esquema de registros de recurso

Todos los registros se almacenan en formato JSON. Cada entrada tiene los siguientes campos de cadena comunes:

Nombre | Descripción
------- | -------
time | Marca de tiempo (en UTC) del registro.
resourceId | Identificador del recurso en el que tuvo lugar la operación, en mayúsculas. Incluye el identificador de suscripción, el grupo de recursos y el nombre del trabajo. Por ejemplo, **/SUBSCRIPTIONS/6503D296-DAC1-4449-9B03-609A1F4A1C87/RESOURCEGROUPS/MY-RESOURCE-GROUP/PROVIDERS/MICROSOFT.STREAMANALYTICS/STREAMINGJOBS/MYSTREAMINGJOB**.
category | La categoría del registro: **Ejecución** o **Creación**.
operationName | Nombre de la operación que se registra. Por ejemplo, **Envío de eventos: error de escritura de salida de SQL en mysqloutput**.
status | Estado de la operación. Por ejemplo, **Erróneo** o **Correcto**.
level | Nivel de registro. Por ejemplo, **Error**, **Advertencia** o **Información**.
properties | Detalle específico de entrada de registro, serializado como una cadena JSON. Para más información, consulte las siguientes secciones de este artículo.

### <a name="execution-log-properties-schema"></a>Esquema de propiedades de registros de ejecución

Los registros de ejecución contienen información sobre eventos que se produjeron durante la ejecución del trabajo de Stream Analytics. El esquema de propiedades varía en función de si el evento es un error de datos o un evento genérico.

### <a name="data-errors"></a>Errores de datos

Cualquier error que se produce mientras el trabajo está procesando datos se puede incluir en esta categoría de registros. Estos registros se crean habitualmente durante las operaciones de lectura, serialización y escritura de datos. No incluyen errores de conectividad. Los errores de conectividad se tratan como eventos genéricos. Puede aprender más sobre la causa de los distintos [errores de datos de entrada y salida](https://docs.microsoft.com/azure/stream-analytics/data-errors).

Nombre | Descripción
------- | -------
Source | Nombre de la entrada o salida del trabajo donde se produjo el error.
Message | Mensaje asociado al error.
Tipo | Tipo de error. Por ejemplo, **DataConversionError**, **CsvParserError** o **ServiceBusPropertyColumnMissingError**.
data | Contiene datos útiles para localizar con exactitud el origen del error. Sujeto a truncamiento dependiendo del tamaño.

En función del valor de **operationName**, los errores de datos tendrán el siguiente esquema:

* Los **eventos de serialización** se producen durante las operaciones de lectura de eventos. Tienen lugar cuando los datos en la entrada no satisfacen el esquema de consulta por una de las siguientes razones:

   * *Discordancia de tipos durante la (de)serialización de eventos*: identifica el campo que provoca el error.

   * *No se puede leer un evento, la serialización no es válida*: muestra información sobre la ubicación de los datos de entrada donde se produjo el error. Incluye el nombre de blob de la entrada de blob, el desplazamiento y una muestra de los datos.

* Los **eventos de envío** se producen durante las operaciones de escritura. Identifican el evento de transmisión que produjo el error.

### <a name="generic-events"></a>Eventos genéricos

Los eventos genéricos incluyen todos los demás.

Nombre | Descripción
-------- | --------
Error | (opcional) Información de error. Normalmente, es información de la excepción, si está disponible.
Message| Mensaje de registro.
Tipo | Tipo de mensaje. Se asigna a la categorización interna de errores. Por ejemplo, **JobValidationError** o **BlobOutputAdapterInitializationFailure**.
Id. de correlación | GUID que identifica de manera única la ejecución del trabajo. Todas las entradas de registros de ejecución desde el momento en que se inicia el trabajo hasta que se detiene tienen el mismo valor de **Id. de correlación**.

## <a name="next-steps"></a>Pasos siguientes

* [Errores de datos de Stream Analytics](https://docs.microsoft.com/azure/stream-analytics/data-errors)
* [Referencia de lenguaje de consulta de Stream Analytics](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference)
