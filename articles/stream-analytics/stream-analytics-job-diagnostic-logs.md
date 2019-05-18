---
title: Solución de problemas de Azure Stream Analytics mediante registros de diagnóstico
description: En este artículo se describe cómo analizar los registros de diagnóstico en Azure Stream Analytics.
services: stream-analytics
author: jseb225
ms.author: jeanb
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 05/15/2019
ms.openlocfilehash: e784cfd2956479327cff9c97a09dd0ada6a154c2
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65826580"
---
# <a name="troubleshoot-azure-stream-analytics-by-using-diagnostics-logs"></a>Solución de problemas de Azure Stream Analytics mediante registros de diagnóstico

En ocasiones, un trabajo de Azure Stream Analytics deja de procesarse inesperadamente. Es importante poder solucionar este tipo de evento. Los errores pueden deberse a un resultado de consulta inesperado, la conectividad a los dispositivos o una interrupción inesperada del servicio. Los registros de diagnóstico de Stream Analytics pueden ayudarle a identificar la causa de los problemas cuando se producen y a reducir el tiempo de recuperación.

## <a name="log-types"></a>Tipos de registro

Stream Analytics ofrece dos tipos de registros:

* [Registros de actividad](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs) (siempre activados), que proporcionan información sobre las operaciones realizadas en los trabajos.

* [Registros de diagnóstico](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs) (configurables), que proporcionan información más completa sobre todo lo que ocurre con un trabajo. Comienzan cuando se crea el trabajo y finalizan cuando se elimina el trabajo. Abarcan los eventos de cuando se actualiza el trabajo y mientras se está ejecutando.

> [!NOTE]
> Puede usar servicios como Azure Storage, Azure Event Hubs, y los registros de Azure Monitor para analizar datos no conformes. Se le cobra según el modelo de precios existente para esos servicios.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="debugging-using-activity-logs"></a>Depuración con registros de actividad

Los registros de actividad están activados de forma predeterminada y proporcionan conclusiones de alto nivel sobre las operaciones realizadas por el trabajo de Stream Analytics. La información presente en los registros de actividad puede ayudar a encontrar la causa principal de los problemas que afectan a su trabajo. Siga los pasos a continuación para usar los registros de actividad en Stream Analytics:

1. Inicie sesión en Azure Portal y seleccione **Registro de actividad** en **Introducción**.

   ![Registro de actividad de Stream Analytics](./media/stream-analytics-job-diagnostic-logs/stream-analytics-menu.png)

2. Puede ver una lista de operaciones que se han realizado. Toda operación que haya producido un error en el trabajo tiene una burbuja de información de color rojo.

3. Haga clic en una operación para ver su vista de resumen. Esta información suele ser limitada. Para más información acerca de la operación, haga clic en **JSON**.

   ![Resumen de la operación de registro de actividad de Stream Analytics](./media/stream-analytics-job-diagnostic-logs/operation-summary.png)

4. Desplácese hacia abajo hasta la sección **Propiedades** del código JSON, que proporciona los detalles del error que provocó el error de la operación. En este ejemplo, el error se produjo debido a un error en tiempo de ejecución por valores fuera de los límites de latitud.

   ![Detalles del error en JSON](./media/stream-analytics-job-diagnostic-logs/error-details.png)

5. Puede tomar acciones correctivas según el mensaje de error en JSON. En este ejemplo, deben agregarse a la consulta comprobaciones para asegurar que el valor de latitud esté entre -90 grados y 90 grados.

6. Si el mensaje de error en los registros de actividad no es útil para identificar la causa raíz, habilitar los registros de diagnóstico y usar los registros de Azure Monitor.

## <a name="send-diagnostics-to-azure-monitor-logs"></a>Enviar diagnósticos a los registros de Azure Monitor

Se recomienda activar los registros de diagnóstico y los envía a los registros de Azure Monitor. Los registros de diagnóstico están **desactivados** de forma predeterminada. Para activar los registros de diagnóstico, siga estos pasos:

1.  Inicie sesión en Azure Portal y vaya al trabajo de Stream Analytics. En **Supervisión**, seleccione **Registros de diagnóstico**. Después, seleccione **Activar diagnósticos**.

    ![Navegación en la hoja a los registros de diagnóstico](./media/stream-analytics-job-diagnostic-logs/diagnostic-logs-monitoring.png)  

2.  Cree un **Nombre** en **Configuración de diagnóstico** y active la casilla junto a **Enviar a Log Analytics**. Luego, agregue un **área de trabajo de Log Analytics** existente o cree una. Active las casillas de **Ejecución** y **Creación** en **LOG**, y **AllMetrics** en **MÉTRICA**. Haga clic en **Save**(Guardar).

    ![Configuración de registros de diagnóstico](./media/stream-analytics-job-diagnostic-logs/diagnostic-settings.png)

3. Cuando se inicia el trabajo de Stream Analytics, los registros de diagnóstico se enrutan a su área de trabajo de Log Analytics. Navegue hasta el área de trabajo de Log Analytics y elija **Registros** bajo la sección **General**.

   ![Registros de Azure Monitor en la sección general](./media/stream-analytics-job-diagnostic-logs/log-analytics-logs.png)

4. Puede escribir [su propia consulta](../azure-monitor/log-query/get-started-portal.md) para buscar términos, identificar tendencias o analizar patrones y proporcionar conclusiones basadas en los datos. Por ejemplo, puede escribir una consulta para filtrar solo los registros de diagnóstico que tengan el mensaje "Error en el trabajo de streaming". Los registros de diagnóstico de Azure Stream Analytics se almacenan en la tabla **AzureDiagnostics**.

   ![Consulta y resultados de diagnóstico](./media/stream-analytics-job-diagnostic-logs/diagnostic-logs-query.png)

5. Cuando tenga una consulta que busque los registros correctos, guárdela seleccionando **Guardar** y proporcione un nombre y una categoría. A continuación, puede crear una alerta seleccionando **Nueva regla de alertas**. A continuación, especifique la condición de la alerta. Seleccione **Condición** y escriba el valor de umbral y la frecuencia a la que se evalúa esta búsqueda de registros personalizada.  

   ![Consulta de búsqueda de registros de diagnóstico](./media/stream-analytics-job-diagnostic-logs/search-query.png)

6. Elija el grupo de acciones y especifique los detalles de la alerta, como el nombre y descripción, antes de poder crear la regla de alertas. Puede enrutar los registros de diagnóstico de varios trabajos a la misma área de trabajo de Log Analytics. Esto le permite configurar las alertas una vez para que funcionen en todos los trabajos.  

## <a name="diagnostics-log-categories"></a>Categorías de registro de diagnóstico

Azure Stream Analytics captura dos categorías de registros de diagnóstico:

* **Creación**: Captura registros de eventos relacionados con operaciones de creación de trabajos, como creación de trabajos, adición y eliminación de entradas y salidas, adición y actualización de la consulta, e inicio y detención del trabajo.

* **Ejecución**: Captura los eventos que se producen durante la ejecución del trabajo.
    * Errores de conectividad
    * Errores de procesamiento de datos, como por ejemplo:
        * Eventos que no se ajustan a la definición de consulta (tipos y valores de campo no coincidentes, campos ausentes, etc.)
        * Errores de evaluación de expresión
    * Otros eventos y errores

## <a name="diagnostics-logs-schema"></a>Esquema de registros de diagnóstico

Todos los registros se almacenan en formato JSON. Cada entrada tiene los siguientes campos de cadena comunes:

NOMBRE | DESCRIPCIÓN
------- | -------
time | Marca de tiempo (en UTC) del registro.
resourceId | Identificador del recurso en el que tuvo lugar la operación, en mayúsculas. Incluye el identificador de suscripción, el grupo de recursos y el nombre del trabajo. Por ejemplo, **/SUBSCRIPTIONS/6503D296-DAC1-4449-9B03-609A1F4A1C87/RESOURCEGROUPS/MY-RESOURCE-GROUP/PROVIDERS/MICROSOFT.STREAMANALYTICS/STREAMINGJOBS/MYSTREAMINGJOB**.
category | La categoría del registro: **Ejecución** o **Creación**.
operationName | Nombre de la operación que se registra. Por ejemplo, **Envío de eventos: error de escritura de salida de SQL en mysqloutput**.
status | Estado de la operación. Por ejemplo, **Erróneo** o **Correcto**.
nivel | Nivel de registro. Por ejemplo, **Error**, **Advertencia** o **Información**.
properties | Detalle específico de entrada de registro, serializado como una cadena JSON. Para más información, consulte las siguientes secciones de este artículo.

### <a name="execution-log-properties-schema"></a>Esquema de propiedades de registros de ejecución

Los registros de ejecución contienen información sobre eventos que se produjeron durante la ejecución del trabajo de Stream Analytics. El esquema de propiedades varía dependiendo de si el evento es un error de datos o un evento genérico.

### <a name="data-errors"></a>Errores de datos

Cualquier error que se produce mientras el trabajo está procesando datos se puede incluir en esta categoría de registros. Estos registros se crean habitualmente durante las operaciones de lectura, serialización y escritura de datos. No incluyen errores de conectividad. Los errores de conectividad se tratan como eventos genéricos.

NOMBRE | DESCRIPCIÓN
------- | -------
Origen | Nombre de la entrada o salida del trabajo donde se produjo el error.
Message | Mensaje asociado al error.
Type | Tipo de error. Por ejemplo, **DataConversionError**, **CsvParserError** o **ServiceBusPropertyColumnMissingError**.
Datos | Contiene datos útiles para localizar con exactitud el origen del error. Sujeto a truncamiento dependiendo del tamaño.

En función del valor de **operationName**, los errores de datos tendrán el siguiente esquema:

* **Serializar eventos** se producen durante las operaciones de lectura de eventos. Tienen lugar cuando los datos en la entrada no satisfacen el esquema de consulta por una de las siguientes razones:

   * *Discordancia de tipos durante la (de)serialización de eventos*: identifica el campo que provoca el error.

   * *No se puede leer un evento, la serialización no es válida*: muestra información sobre la ubicación de los datos de entrada donde se produjo el error. Incluye el nombre de blob de la entrada de blob, el desplazamiento y una muestra de los datos.

* **Enviar eventos** se producen durante las operaciones de escritura. Identifican el evento de transmisión que produjo el error.

### <a name="generic-events"></a>Eventos genéricos

Los eventos genéricos incluyen todos los demás.

NOMBRE | DESCRIPCIÓN
-------- | --------
Error | (opcional) Información de error. Normalmente, es información de la excepción, si está disponible.
Message| Mensaje de registro.
Type | Tipo de mensaje. Se asigna a la categorización interna de errores. Por ejemplo, **JobValidationError** o **BlobOutputAdapterInitializationFailure**.
Id. de correlación | [GUID](https://en.wikipedia.org/wiki/Universally_unique_identifier) que identifica de manera única la ejecución del trabajo. Todas las entradas de registros de ejecución desde el momento en que se inicia el trabajo hasta que se detiene tienen el mismo valor de **Id. de correlación**.

## <a name="next-steps"></a>Pasos siguientes

* [¿Qué es Stream Analytics?](stream-analytics-introduction.md)
* [Introducción a Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Escalado de trabajos de Stream Analytics](stream-analytics-scale-jobs.md)
* [Referencia de lenguaje de consulta de Stream Analytics](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Referencia de API de REST de administración de Stream Analytics](https://msdn.microsoft.com/library/azure/dn835031.aspx)
