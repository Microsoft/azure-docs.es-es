---
title: 'Guía para mantenimiento predictivo para el sector aeroespacial: proceso de ciencia de datos en equipos'
description: Una guía técnica sobre la plantilla de solución orientada al mantenimiento predictivo en los sectores aeroespacial, de servicios públicos y de transporte.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: previous-author=fboylu, previous-ms.author=fboylu
ms.openlocfilehash: 9520369861623e60a0118baa20a7871437433a4b
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87290709"
---
# <a name="technical-guide-to-the-solution-template-for-predictive-maintenance-in-aerospace"></a>Guía técnica sobre la plantilla de solución orientada al mantenimiento predictivo en el sector aeroespacial

> [!Important]
> Este artículo está en desuso. El debate sobre el mantenimiento predictivo en el sector aeroespacial sigue siendo relevante, pero consulte la [introducción a la solución para audiencias empresariales](https://github.com/Azure/cortana-intelligence-predictive-maintenance-aerospace) para obtener información actualizada.


Las plantillas de solución están diseñadas para acelerar el proceso de creación de una demostración E2E. Una plantilla implementada aprovisiona la suscripción con los componentes necesarios y crea las relaciones entre ellos. También propaga la canalización de datos con datos de ejemplo generados a partir de una aplicación de generador de datos que descarga e instala en la máquina local después de implementar la plantilla de la solución. Los datos generados a partir del generador hidratan la canalización de datos e iniciarán la generación de predicciones de aprendizaje automático que podrán verse a continuación en el panel de Power BI.

El proceso de implementación le guía a través de varios pasos para configurar las credenciales de la solución. Asegúrese de que registra estas credenciales: nombre de la solución, nombre de usuario y contraseña que proporcionó durante la implementación. 


Los objetivos de este artículo son los siguientes:
- Describir la arquitectura de referencia y los componentes aprovisionados en la suscripción.
- Demostrar cómo reemplazar los datos de ejemplo con sus propios datos. 
- Mostrar cómo modificar la plantilla de solución.  

## <a name="overview"></a>Información general
![Arquitectura de mantenimiento predictivo](./media/predictive-maintenance-technical-guide/predictive-maintenance-architecture.png)

Al implementar la solución, activa los servicios de Azure (es decir, Event Hubs, Stream Analytics, HDInsight, Data Factory y Machine Learning). El diagrama de arquitectura anterior muestra cómo se construye el mantenimiento predictivo para la plantilla de solución aeroespacial. Puede investigar estos servicios en Azure Portal haciendo clic en ellos en el diagrama de la plantilla de solución creada con la implementación de la solución (con la excepción de HDInsight, que se aprovisiona a petición cuando las actividades de canalización relacionadas se requieren para la ejecución y se eliminan a continuación).
Descargue una [versión en tamaño completo del diagrama](https://download.microsoft.com/download/1/9/B/19B815F0-D1B0-4F67-AED3-A40544225FD1/ca-topologies-maintenance-prediction.png).

En las secciones siguientes se describen los elementos de la solución.

## <a name="data-source-and-ingestion"></a>Origen e ingesta de datos
### <a name="synthetic-data-source"></a>Origen de datos sintéticos
Para esta plantilla, el origen de datos utilizado se genera desde una aplicación de escritorio que se descarga y ejecuta localmente tras una implementación correcta.

Si quiere encontrar las instrucciones para descargar e instalar esta aplicación, seleccione el primer nodo denominado "Predictive Maintenance Data Generator" en el diagrama de la plantilla de solución. Las instrucciones se encuentran en la barra de propiedades. Esta aplicación alimenta el servicio [Azure Event Hubs](#azure-event-hub) con puntos de datos o eventos, que se usan en el flujo de solución restante. Este origen de datos deriva de los datos disponibles públicamente en el [repositorio de datos de la NASA](https://c3.nasa.gov/dashlink/resources/139/) utilizando [Turbofan Engine Degradation Simulation Data Set](https://ti.arc.nasa.gov/tech/dash/groups/pcoe/prognostic-data-repository/#turbofan).

La aplicación de generación de eventos rellena los recursos de Azure Event Hubs solo mientras se esté ejecutando en el equipo.  

### <a name="azure-event-hub"></a>Centro de eventos de Azure  
El servicio [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/) es el destinatario de la entrada proporcionada por el origen de datos sintéticos.

## <a name="data-preparation-and-analysis"></a>Preparación y análisis de datos  
### <a name="azure-stream-analytics"></a>Azure Stream Analytics
Use [Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics/) para proporcionar un análisis casi en tiempo real del flujo de entrada del servicio [Azure Event Hubs](#azure-event-hub). Después, publique los resultados en un panel de [Power BI](https://powerbi.microsoft.com) y archive todos los eventos entrantes sin formato en el servicio [Azure Storage](https://azure.microsoft.com/services/storage/) para procesarse posteriormente mediante [Azure Data Factory](https://azure.microsoft.com/documentation/services/data-factory/).

### <a name="hdinsight-custom-aggregation"></a>Agregación personalizada de HDInsight
Ejecute los scripts de [Hive](https://docs.microsoft.com/archive/blogs/uk_faculty_connection/getting-started-with-microsoft-big-data-hive-hdinsight-jump-start) (organizados mediante Azure Data Factory) mediante HDInsight para proporcionar las agregaciones en los eventos sin procesar que se almacenaron con el recurso de Azure Stream Analytics.

### <a name="azure-machine-learning"></a>Azure Machine Learning
Realice predicciones sobre la vida útil restante (RUL) de un motor de avión determinado dadas las entradas que se reciben con el servicio [Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/) (organizado por Azure Data Factory). 

## <a name="data-publishing"></a>Publicación de datos
### <a name="azure-sql-database"></a>Azure SQL Database
Utilice [Azure SQL Database](https://azure.microsoft.com/services/sql-database/) para almacenar las predicciones recibidas por Azure Machine Learning que se usan en el panel de [Power BI](https://powerbi.microsoft.com).

## <a name="data-consumption"></a>Consumo de datos
### <a name="power-bi"></a>Power BI
Utilice [Power BI](https://powerbi.microsoft.com) para mostrar un panel que contiene las agregaciones y alertas que proporciona [Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics/), así como las predicciones sobre la vida útil restante almacenadas en [Azure SQL Database](https://azure.microsoft.com/services/sql-database/) que se hayan producido al utilizar [Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/).

## <a name="how-to-bring-in-your-own-data"></a>Incorporación de sus propios datos
Esta sección describe cómo traer sus propios datos a Azure y qué áreas necesitarían cambios para los datos que ha traído a esta arquitectura.

Es improbable que el conjunto de datos coincida con el conjunto de datos utilizado por el [Turbofan Engine Degradation Simulation Data Set](https://ti.arc.nasa.gov/tech/dash/groups/pcoe/prognostic-data-repository/#turbofan) empleado en esta plantilla de solución. Entender bien los datos y los requisitos es fundamental a la hora de modificar esta plantilla para que funcione con sus propios datos. 

En las secciones siguientes se describen las secciones de la plantilla que requieren modificaciones cuando se introduce un nuevo conjunto de datos.

### <a name="azure-event-hub"></a>Centro de eventos de Azure
El servicio Azure Event Hubs es genérico; los datos se pueden publicar datos en el centro en formato CSV o JSON. No se produce ningún procesamiento especial en el Centro de eventos de Azure, pero es importante que comprenda los datos que se introducen en él.

Este documento no describe cómo introducir los datos, pero puede enviar fácilmente eventos o datos a una instancia de Azure Event Hubs mediante las API de Event Hubs.

### <a name="azure-stream-analytics"></a><a name="azure-stream-analytics-1"></a>Azure Stream Analytics
Utilice el recurso de Azure Stream Analytics para proporcionar análisis casi en tiempo real mediante la lectura de flujos de datos y el envío de datos a cualquier número de orígenes.

Para el mantenimiento predictivo de la plantilla de solución aeroespacial, la consulta del servicio Azure Stream Analytics consta de cuatro subconsultas, y cada una de ellas consume eventos del servicio Centro de eventos de Azure y tiene salidas a cuatro ubicaciones distintas. Estas salidas constan de tres conjuntos de datos de Power BI y una ubicación de Azure Storage.

La consulta de Azure Stream Analytics puede encontrarse al:

* Conexión a Azure Portal
* Buscar los trabajos de Stream Analytics ![Icono de Stream Analytic](./media/predictive-maintenance-technical-guide/icon-stream-analytics.png)s que se generaron cuando se implementó la solución (*por ejemplo*, **maintenancesa02asapbi** y **maintenancesa02asablob** para la solución de mantenimiento predictivo)
* Seleccionar
  
  * ***INPUTS*** para ver la entrada de consulta
  * ***QUERY*** para ver la consulta en sí
  * ***OUTPUTS*** para ver las diferentes salidas

Puede encontrar información acerca de la construcción de consultas de Análisis de transmisiones de Azure en [Stream Analytics Query Reference (Referencia de consultas en Análisis de transmisiones)](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference) en MSDN.

En esta solución, las consultas envían tres conjuntos de datos con información de análisis casi en tiempo real acerca del flujo de datos entrante a un panel de Power BI que se proporciona como parte de esta plantilla de solución. Dado que hay un conocimiento implícito acerca del formato de datos entrantes, estas consultas deben modificarse según su formato de datos.

La consulta del segundo trabajo de Stream Analytics, **maintenancesa02asablob**, simplemente envía todos los eventos de [Event Hubs](https://azure.microsoft.com/services/event-hubs/) a [Azure Storage](https://azure.microsoft.com/services/storage/) y, por tanto, no necesita ninguna modificación, independientemente del formato de los datos, ya que toda la información del evento se transmitió al almacenamiento.

### <a name="azure-data-factory"></a>Azure Data Factory
El servicio [Azure Data Factory](https://azure.microsoft.com/documentation/services/data-factory/) organiza el traslado y procesamiento de los datos. En el mantenimiento predictivo para la plantilla de solución aeroespacial, la factoría de datos se compone de tres [canalizaciones](../../data-factory/concepts-pipelines-activities.md) que mueven y procesan los datos mediante varias tecnologías.  Acceda a la factoría de datos si abre el nodo Data Factory en la parte inferior del diagrama de la plantilla de solución que se creó con la implementación de la solución. Los errores en los conjuntos de datos se deben a que la factoría de datos se implementaron antes de iniciarse el generador de datos. Esos errores se pueden omitir y no impiden que la factoría de datos funcione.

![Errores de conjuntos de datos de Data Factory](./media/predictive-maintenance-technical-guide/data-factory-dataset-error.png)

Esta sección describe las [canalizaciones y actividades](../../data-factory/concepts-pipelines-activities.md) necesarias contenidas en [Azure Data Factory](https://azure.microsoft.com/documentation/services/data-factory/). A continuación se muestra la vista de diagrama de la solución.

![Azure Data Factory](./media/predictive-maintenance-technical-guide/azure-data-factory.png)

Dos de las canalizaciones de esta factoría contienen scripts de [Hive](https://docs.microsoft.com/archive/blogs/uk_faculty_connection/getting-started-with-microsoft-big-data-hive-hdinsight-jump-start) que se utilizan para crear particiones y agregar los datos. Cuando los anote, los scripts se encontrarán en la cuenta de [Azure Storage](https://azure.microsoft.com/services/storage/) creada durante la instalación. Su ubicación es: maintenancesascript\\\\script\\\\hive\\\\ (o https://[Nombre de la solución].blob.core.windows.net/maintenancesascript).

Al igual que las consultas de [Azure Stream Analytics](#azure-stream-analytics-1), los scripts de [Hive](https://docs.microsoft.com/archive/blogs/uk_faculty_connection/getting-started-with-microsoft-big-data-hive-hdinsight-jump-start) tienen un conocimiento implícito sobre el formato de los datos entrantes y deben modificarse en función del formato de los datos.

#### <a name="aggregateflightinfopipeline"></a>*AggregateFlightInfoPipeline*
Esta [canalización](../../data-factory/concepts-pipelines-activities.md) contiene una sola actividad: una actividad de [HDInsightHive](../../data-factory/transform-data-using-hadoop-hive.md) que utiliza un servicio [HDInsightLinkedService](https://msdn.microsoft.com/library/azure/dn893526.aspx) que ejecuta un script de [Hive](https://docs.microsoft.com/archive/blogs/uk_faculty_connection/getting-started-with-microsoft-big-data-hive-hdinsight-jump-start) para crear una partición de los datos colocados en el [Azure Storage](https://azure.microsoft.com/services/storage/) durante el trabajo de [Stream Analytics de Azure](https://azure.microsoft.com/services/stream-analytics/).

El script de [Hive](https://docs.microsoft.com/archive/blogs/uk_faculty_connection/getting-started-with-microsoft-big-data-hive-hdinsight-jump-start) para esta tarea de creación de particiones es ***AggregateFlightInfo.hql***.

#### <a name="mlscoringpipeline"></a>*MLScoringPipeline*
Esta [canalización](../../data-factory/concepts-pipelines-activities.md) contiene varias actividades cuyo resultado final son las predicciones puntuadas a partir del experimento de [Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/) asociado a esta plantilla de solución.

Las actividades que se incluyen son:

* Una actividad de [HDInsightHive](../../data-factory/transform-data-using-hadoop-hive.md) que utiliza un servicio de [HDInsightLinkedService](https://msdn.microsoft.com/library/azure/dn893526.aspx) que ejecuta un script de [Hive](https://docs.microsoft.com/archive/blogs/uk_faculty_connection/getting-started-with-microsoft-big-data-hive-hdinsight-jump-start) para realizar las agregaciones y la ingeniería de características necesarias para el experimento de [Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/).
  El script de [Hive](https://docs.microsoft.com/archive/blogs/uk_faculty_connection/getting-started-with-microsoft-big-data-hive-hdinsight-jump-start) para esta tarea de creación de particiones es ***PrepareMLInput.hql***.
* Una actividad de [copia](https://msdn.microsoft.com/library/azure/dn835035.aspx) que mueve los resultados de la actividad [HDInsightHive](../../data-factory/transform-data-using-hadoop-hive.md) a un solo blob de [Azure Storage](https://azure.microsoft.com/services/storage/) al que se puede acceder mediante la actividad [AzureMLBatchScoring](https://msdn.microsoft.com/library/azure/dn894009.aspx).
* [AzureMLBatchScoring](https://msdn.microsoft.com/library/azure/dn894009.aspx) que llama al experimento de [Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/) que hace que los resultados se coloquen en un único blob de [Azure Storage](https://azure.microsoft.com/services/storage/).

#### <a name="copyscoredresultpipeline"></a>*CopyScoredResultPipeline*
Esta [canalización](../../data-factory/concepts-pipelines-activities.md) contiene una sola actividad: una actividad de[copia](https://msdn.microsoft.com/library/azure/dn835035.aspx) que mueve los resultados del experimento de [Azure Machine Learning](#azure-machine-learning) desde la canalización ***MLScoringPipeline*** a la instancia de [Azure SQL Database](https://azure.microsoft.com/services/sql-database/) que se aprovisionó como parte de la instalación de la plantilla de solución.

### <a name="azure-machine-learning"></a>Azure Machine Learning
El experimento de [Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/) utilizado para esta plantilla de solución proporciona la vida útil restante (RUL) de un motor de avión. El experimento es específico del conjunto de datos consumido y requiere la modificación o el reemplazo específico de los datos que se han aportado.

## <a name="monitor-progress"></a>Supervisión de progreso
Una vez iniciado el generador de datos, la canalización comienza a deshidratarse y los distintos componentes de la solución empiezan a entrar en acción siguiendo los comandos emitidos por la factoría de datos. Hay dos maneras de supervisar la canalización.

* Uno de los trabajos de Stream Analytics escribe los datos entrantes sin procesar en Blob Storage. Si hace clic en el componente Blob Storage de su solución desde la pantalla donde implementó correctamente la solución y después hace clic en Abrir en el panel derecho, irá a [Azure Portal](https://portal.azure.com/). Una vez allí, haga clic en Blobs. En el siguiente panel, verá una lista de Containers. Haga clic en **maintenancesadata**. En el siguiente panel, verá la carpeta **rawdata**. Dentro de la carpeta rawdata, verá carpetas con nombres como date=17 y hour=18. La presencia de estas carpetas significa que los datos sin procesar se generan en el equipo y se almacenan en un almacenamiento de blobs. Debería ver archivos csv con tamaños finitos en MB en esas carpetas.
* El último paso de la canalización consiste en escribir los datos (por ejemplo, predicciones del aprendizaje automático) en SQL Database. Es posible que deba esperar un máximo de tres horas para los datos aparezcan en SQL Database. Puede supervisar la cantidad de datos disponibles en SQL Database a través de [Azure Portal](https://portal.azure.com/). En el panel izquierdo, busque y haga clic en el ![icono SQL](./media/predictive-maintenance-technical-guide/icon-SQL-databases.png) de BASES DE DATOS SQL. Después, busque la base de datos **pmaintenancedb** y haga clic en ella. En la parte inferior de la página siguiente, haga clic en ADMINISTRAR.
   
    ![Icono de Administrar](./media/predictive-maintenance-technical-guide/icon-manage.png)
   
    Aquí, puede hacer clic en Nueva consulta y consultar el número de filas (por ejemplo, select count(*) from PMResult). A medida que la base de datos aumente de tamaño, debe incrementar el número de filas de la tabla.

## <a name="power-bi-dashboard"></a>Panel de Power BI

Configure el panel de Power BI para visualizar los datos de Azure Stream Analytics (ruta de acceso activa), así como los resultados de predicción por lotes de Azure Machine Learning (ruta de acceso inactiva).

### <a name="set-up-the-cold-path-dashboard"></a>Configuración del panel de ruta de acceso inactiva
En la canalización de datos de la ruta de acceso inactiva, el objetivo es obtener la predicción de la RUL (vida útil restante) de cada motor del avión cuando finaliza un vuelo (ciclo). El resultado de la predicción se actualiza cada 3 horas para poder predecir la vida útil restante de los motores de los aviones que hayan terminado un vuelo durante las últimas 3 horas.

Power BI se conecta a una base de datos de Azure SQL como su origen de datos, donde se almacenan los resultados de predicción. 

Nota: 
1.    Al implementar la solución, aparecerá una predicción real en la base de datos dentro de 3 horas. El archivo pbix incluido con la descarga del generador, contiene algunos datos de inicialización para que pueda crear el panel de Power BI inmediatamente. 
2.    En este paso, el requisito previo es descargar e instalar gratuitamente el software [Power BI Desktop](https://docs.microsoft.com/power-bi/fundamentals/desktop-get-the-desktop).

Los siguientes pasos lo guiarán a la hora de conectar el archivo pbix con SQL Database que estaba en marcha en el momento de la implementación de la solución que contiene los datos (por ejemplo, resultados de predicción) para su visualización.

1. Obtenga las credenciales de la base de datos.
   
   Antes de continuar con los pasos siguientes, necesitará el **nombre del servidor de la base de datos, el nombre de la base de datos, el nombre de usuario y la contraseña** . Estos son los pasos que le ayudarán a encontrarlos.
   
   * Una vez que la opción **Azure SQL Database** se muestre en color verde en el diagrama de la plantilla de solución, haga clic en ella y, después, en **Abrir**.
   * Verá una nueva pestaña o ventana del explorador que muestra la página de Azure Portal. Haga clic en **"Grupos de recursos"** en el panel izquierdo.
   * Seleccione la suscripción que va a utilizar para implementar la solución y, después, seleccione **nombreDeLaSolución\_grupoDeRecursos**.
   * En el nuevo panel emergente, haga clic en el ![icono de SQL](./media/predictive-maintenance-technical-guide/icon-sql.png) para acceder a la base de datos. El nombre de la base de datos se sitúa junto a este icono (por ejemplo, **pmaintenancedb**), y el **nombre del servidor de bases de datos** aparece bajo la propiedad de nombre de servidor y se parece a **nombreDeLaSolución.database.windows.net**.
   * El **nombre de usuario** y la **contraseña** de la base de datos son los mismos que los registrados anteriormente durante la implementación de la solución.
2. Actualice el origen de datos del archivo de informe de la ruta de acceso inactiva con Power BI Desktop.
   
   * En la carpeta donde ha descargado y descomprimido el archivo del generador, haga doble clic en el archivo **PowerBI\\PredictiveMaintenanceAerospace.pbix**. Si ve algún mensaje de advertencia al abrir el archivo, ignórelo. En la parte superior del archivo, haga clic en **"Editar consultas"** .
     
     ![Editar consultas](./media/predictive-maintenance-technical-guide/edit-queries.png)
   * Verá dos tablas, **RemainingUsefulLife** y **PMResult**. Seleccione la primera tabla y haga clic en ![Icono de Configuración de la consulta](./media/predictive-maintenance-technical-guide/icon-query-settings.png) junto a **Origen** en **PASOS APLICADOS** en el panel de la derecha **Configuración de la consulta**. Ignore los mensajes de advertencia que aparezcan.
   * En la ventana emergente, reemplace **Servidor** y **Base de datos** por sus propios nombres de servidor y base de datos y, después, haga clic en **Aceptar**. Para el nombre del servidor, asegúrese de especificar el puerto 1433 (**SuNombreDeSolución.database.windows.net, 1433**). Deje el campo Base de datos como **pmaintenancedb**. Ignore los mensajes de advertencia que aparezcan en la pantalla.
   * En la siguiente ventana emergente, verá dos opciones en el panel izquierdo (**Windows** y **Base de datos**). Haga clic en **Base de datos**, escriba su **nombre de usuario** y **contraseña** (que es el nombre de usuario y la contraseña que escribió la primera vez que implementó la solución y creó una base de datos de Azure SQL). En ***Seleccionar en qué nivel hay que aplicar estos valores***, active la opción de nivel de base de datos. A continuación, haga clic en **"Conectar"** .
   * Haga clic en la segunda tabla **PMResult**; después, en ![Icono de Navegación](./media/predictive-maintenance-technical-guide/icon-navigation.png) junto a la opción **Origen** de **PASOS APLICADOS** en el panel de la derecha **Configuración de la consulta**. Luego, actualice los nombres del servidor y de la base de datos como en los pasos anteriores y, por último, haga clic en Aceptar.
   * Una vez que se le dirija a la página anterior, cierre la ventana. Se muestra un mensaje y haga clic **Aplicar**. Finalmente, haga clic en el botón **Guardar** para guardar los cambios. El archivo de Power BI ha establecido ya la conexión con el servidor. Si las visualizaciones están vacías, asegúrese de borrar todas las selecciones para poder visualizar todos los datos haciendo clic en el icono de borrador de la esquina superior derecha de las leyendas. Utilice el botón Actualizar para reflejar los nuevos datos en las visualizaciones. Inicialmente, solo verá los datos de inicialización en las visualizaciones ya que la factoría de datos está programada para actualizarse cada tres horas. Después de 3 horas, verá nuevas predicciones reflejadas en las visualizaciones al actualizar los datos.
3. (Opcional) Publique el panel de la ruta de acceso en frío en [Power BI en línea](https://www.powerbi.com/). Este paso necesita una cuenta de Power BI (o la cuenta de Office 365).
   
   * Haga clic en **Publicar** y, unos segundos más tarde, aparecerá una ventana con un mensaje que indicará que la publicación en Power BI se ha realizado correctamente y con una marca de verificación verde. Haga clic en el vínculo "Abrir PredictiveMaintenanceAerospace.pbix en Power BI". Para obtener instrucciones detalladas, consulte [Publicar desde Power BI Desktop](https://support.powerbi.com/knowledgebase/articles/461278-publish-from-power-bi-desktop).
   * Para crear un nuevo panel: haga clic en el signo **+** junto a la sección **Paneles** del panel izquierdo. Escriba el nombre "Demo de mantenimiento predictivo" para este nuevo panel.
   * Una vez abierto el informe, haga clic en ![Icono de PIN](./media/predictive-maintenance-technical-guide/icon-pin.png) para anclar todas las visualizaciones al panel. Para obtener instrucciones detalladas, consulte [Anclaje de un icono a un panel de Power BI desde un informe](https://support.powerbi.com/knowledgebase/articles/430323-pin-a-tile-to-a-power-bi-dashboard-from-a-report).
     Vaya a la página del panel y ajuste el tamaño y la ubicación de las visualizaciones y edite los títulos. Para obtener instrucciones detalladas sobre cómo editar los iconos, consulte [Edición de un icono: cambiar el tamaño, mover, cambiar el nombre, anclar, eliminar, agregar un hipervínculo](https://powerbi.microsoft.com/documentation/powerbi-service-edit-a-tile-in-a-dashboard/#rename). Este es un panel de ejemplo con algunas visualizaciones de la ruta de acceso inactiva ancladas.  Dependiendo de cuánto tiempo tarde en ejecutarse el generador de datos, los números en las visualizaciones pueden ser diferentes.
     <br/>
     ![Vista final](./media/predictive-maintenance-technical-guide/final-view.png)
     <br/>
   * Para programar la actualización de los datos, mantenga el puntero sobre el conjunto de datos **PredictiveMaintenanceAerospace**, haga clic en ![Icono de puntos suspensivos](./media/predictive-maintenance-technical-guide/icon-elipsis.png) y, después, elija **Programar actualización**.
     <br/>
     > [!NOTE]
     > Si ve un mensaje de advertencia, haga clic en **Editar credenciales** y asegúrese de que las credenciales de la base de datos son las mismas que las descritas en el paso 1.
     <br/>
     ![Programar la actualización](./media/predictive-maintenance-technical-guide/schedule-refresh.png)
     <br/>
   * Expanda la sección **Programar actualización** . Active "Mantener los datos actualizados".
     <br/>
   * Programe la actualización según sus necesidades. Para obtener más información, consulte [Actualizar datos en Power BI](https://support.powerbi.com/knowledgebase/articles/474669-data-refresh-in-power-bi).

### <a name="setup-hot-path-dashboard"></a>Configuración del panel de ruta de acceso activa
Los siguientes pasos lo guiarán en el proceso para visualizar la salida de datos de los trabajos de Stream Analytics que se generaron en el momento de la implementación de la solución. Se necesita una cuenta de [Power BI en línea](https://www.powerbi.com/) para realizar los pasos siguientes. Si no tiene una cuenta, puede [crear una](https://powerbi.microsoft.com/pricing).

1. Agregue una salida de Power BI en Azure Stream Analytics (ASA).
   
   * Debe seguir las instrucciones descritas en [Azure Stream Analytics y Power BI: panel de análisis en tiempo real de flujo de datos](../../stream-analytics/stream-analytics-power-bi-dashboard.md) para configurar la salida del trabajo de Azure Stream Analytics como panel de Power BI.
   * La consulta ASA tiene tres salidas, que son **aircraftmonitor**, **aircraftalert** y **flightsbyhour**. Puede ver la consulta haciendo clic en la pestaña de consulta. Debe agregar a ASA la salida correspondiente a cada una de estas tablas. Cuando se agrega la primera salida (**aircraftmonitor**) asegúrese de que el **alias de salida**, el **nombre del conjunto de datos** y el **nombre de la tabla** son los mismos (**aircraftmonitor**). Repita los pasos para agregar las salidas de **aircraftalert** y **flightsbyhour**. Una vez haya agregado las tres tablas de salida e iniciado el trabajo de ASA, recibirá un mensaje de confirmación ("Se ha iniciado correctamente el trabajo de Stream Analytics maintenancesa02asapbi").
2. Inicie sesión en [Power BI en línea](https://www.powerbi.com)
   
   * En la sección Conjuntos de datos del panel izquierdo de Mi área de trabajo, deben aparecer los nombres de los ***CONJUNTOS DE DATOS*** **aircraftmonitor**, **aircraftalert** y **flightsbyhour**. Son los datos de transmisión que insertó desde Azure Stream Analytics en el paso anterior. El conjunto de datos **flightsbyhour** puede no aparecer al mismo tiempo que los otros dos conjuntos de datos debido a la naturaleza de la consulta SQL que hay detrás. Sin embargo, deben aparecer después de una hora.
   * Asegúrese de que el panel ***Visualizaciones*** está abierto y se muestra en el lado derecho de la pantalla.
3. Una vez que tenga los datos fluyendo en Power BI, puede comenzar a visualizar los datos de streaming. Este es un panel de ejemplo con algunas visualizaciones de la ruta de acceso en caliente ancladas. Puede crear otros iconos de panel basados en conjuntos de datos adecuados. Dependiendo de cuánto tiempo tarde en ejecutarse el generador de datos, los números en las visualizaciones pueden ser diferentes.

    ![Vista de panel](media/predictive-maintenance-technical-guide/dashboard-view.png)

1. Estos son algunos pasos para crear uno de los iconos anteriores: "Fleet View of Sensor 11 vs. Threshold 48.26":
   
   * Haga clic en el conjunto de datos **aircraftmonitor** en el panel izquierdo de la sección Conjuntos de datos.
   * Haga clic en el icono **Gráfico de líneas** .
   * Haga clic en la opción **Procesados** del panel **Campos** para que aparezca en Eje en el panel **Visualizaciones**.
   * Haga clic en s11 y s11\_alert para que ambos aparezcan en Valores. Haga clic en la flecha pequeña situada junto a **s11** y **s11\_alert**, y cambie Suma a Media.
   * Haga clic en **GUARDAR** en la parte superior y asigne el nombre "aircraftmonitor" al informe. El informe denominado "aircraftmonitor" se muestra en la sección **Informes** del panel **Navegador** de la izquierda.
   * Haga clic en el icono **Anclar visualización** situado en la esquina superior derecha de este gráfico de líneas. Puede aparecer una ventana llamada "Anclar en el panel" que le permitirá elegir un panel. Seleccione "Demostración de mantenimiento predictivo" y luego haga clic en "Anclar".
   * Mantenga el mouse sobre este icono en el panel, haga clic en el icono "Editar" en la esquina superior derecha para cambiar el título a "Fleet View of Sensor 11 vs. Threshold 48.26" y el subtítulo a "Average across fleet over time".

## <a name="delete-your-solution"></a>Eliminación de la solución
Asegúrese de detener el generador de datos cuando no se use activamente la solución, ya que de lo contrario incurrirá en costos elevados. Elimine la solución si no la está usando. Al eliminar la solución se eliminan todos los componentes aprovisionados en su suscripción cuando se implementó la solución. Para eliminar la solución, haga clic en el nombre de la solución en el panel izquierdo de la plantilla de soluciones y haga clic en **Eliminar**.

## <a name="cost-estimation-tools"></a>Herramientas de estimación de costos
Las dos herramientas siguientes están disponibles para ayudarlo a comprender mejor los costos totales implicados en la ejecución del mantenimiento predictivo para la plantilla de solución aeroespacial en su suscripción:

* [Herramienta de estimación de costos de Microsoft Azure (en línea)](https://azure.microsoft.com/pricing/calculator/)
* [Herramienta de estimación de costos de Microsoft Azure (escritorio)](https://www.microsoft.com/download/details.aspx?id=43376)

