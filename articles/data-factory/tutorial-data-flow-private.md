---
title: Transformación de datos con un flujo de datos de asignación de red virtual administrado por Azure Data Factory
description: Este tutorial proporciona instrucciones detalladas para usar Azure Data Factory con el objetivo de transformar los datos con un flujo de datos de asignación.
author: djpmsft
ms.author: daperlov
ms.reviewer: makromer
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 05/19/2019
ms.openlocfilehash: 5515f6c4dfbc5d3c0e391373e763597d7fdc89ed
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86528321"
---
# <a name="transform-data-securely-using-mapping-data-flows"></a>Transformación de datos mediante flujos de datos de asignación de forma segura

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Si no está familiarizado con Azure Data Factory, consulte [Introducción a Azure Data Factory](https://docs.microsoft.com/azure/data-factory/introduction).

En este tutorial, se usará la interfaz de usuario de Azure Data Factory (UX) para crear una canalización que copie y transforme los datos **desde un origen de Azure Data Lake Storage (ADLS) Gen2 hasta un receptor de ADLS Gen2 (ambos permitiendo el acceso solo a las redes seleccionadas)** mediante un flujo de datos de asignación en una [red virtual administrada por Azure Data Factory](managed-virtual-network-private-endpoint.md). El patrón de configuración de este tutorial se puede expandir al transformar los datos mediante el flujo de datos de asignación.

En este tutorial, realizará los siguientes pasos:

> [!div class="checklist"]
>
> * Creación de una factoría de datos.
> * Creación de una canalización con una actividad de Data Flow.
> * Cree un flujo de datos de asignación con cuatro transformaciones.
> * Realización de la serie de pruebas de la canalización.
> * Supervisión de una actividad de Data Flow

## <a name="prerequisites"></a>Requisitos previos
* **Suscripción de Azure**. Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* **Cuenta de Azure Storage**. El almacenamiento ADLS se puede usar como almacén de datos de *origen* y *receptor*. Si no tiene una cuenta de almacenamiento, consulte [Crear una cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/common/storage-account-create?tabs=azure-portal) para crear una. **Asegúrese de que la cuenta de almacenamiento solo permita el acceso desde las "Redes seleccionadas".** 

El archivo que se está transformando en este tutorial es MoviesDB. csv, que se puede encontrar [aquí](https://raw.githubusercontent.com/djpmsft/adf-ready-demo/master/moviesDB.csv). Para recuperar el archivo de GitHub, copie el contenido en un editor de texto de su elección para guardarlo localmente como un archivo. csv. Para cargar el archivo en la cuenta de almacenamiento, vea [Carga de blobs con Azure Portal](https://docs.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-portal). Los ejemplos harán referencia a un contenedor denominado "Sample-Data".

## <a name="create-a-data-factory"></a>Crear una factoría de datos

En este paso, creará una factoría de datos y abrirá la interfaz de usuario de Data Factory para crear una canalización en la factoría de datos.

1. Abra **Microsoft Edge** o **Google Chrome**. Actualmente, la interfaz de usuario de Data Factory solo se admite en los exploradores web Microsoft Edge y Google Chrome.
2. En el menú de la izquierda, seleccione **Crear un recurso** > **Analytics** > **Data Factory**.
3. En la página **Nueva factoría de datos**, en **Nombre**, escriba **ADFTutorialDataFactory**.

   El nombre de la instancia de Azure Data Factory debe ser *único de forma global*. Si recibe un mensaje de error sobre el valor de nombre, escriba un nombre diferente para la factoría de datos. (Por ejemplo, utilice SuNombreADFTutorialDataFactory). Para conocer las reglas de nomenclatura de los artefactos de Data Factory, consulte [Azure Data Factory: reglas de nomenclatura](naming-rules.md).

4. Seleccione la **suscripción** de Azure en la que quiere crear la factoría de datos.
5. Para **Grupo de recursos**, realice uno de los siguientes pasos:

    a. Seleccione en primer lugar **Usar existente**y después un grupo de recursos de la lista desplegable.

    b. Seleccione **Crear nuevo**y escriba el nombre de un grupo de recursos. 
         
    Para más información sobre los grupos de recursos, consulte [Uso de grupos de recursos para administrar los recursos de Azure](../azure-resource-manager/management/overview.md). 
6. En **Versión**, seleccione **V2**.
7. En **Ubicación**, seleccione la ubicación de la factoría de datos. En la lista desplegable solo se muestran las ubicaciones que se admiten. Los almacenes de datos (por ejemplo, Azure Storage y SQL Database) y los procesos (por ejemplo, Azure HDInsight) que la factoría de datos usa pueden estar en otras regiones.

8. Seleccione **Crear**.

9. Una vez finalizada la creación, verá el aviso en el centro de notificaciones. Seleccione **Ir al recurso** para ir a la página de Data Factory.
10. Haga clic en **Author & Monitor** (Creación y supervisión) para iniciar la interfaz de usuario de Data Factory en una pestaña independiente.

## <a name="create-an-azure-integration-runtime-in-adf-managed-virtual-network"></a>Creación de una instancia de Azure Integration Runtime en una red virtual administrada por ADF
En este paso se crea una instancia de Azure Integration Runtime y se habilita una red virtual administrada.

1. En el portal de ADF, vaya a **Administrar centro** y haga clic en **Nuevo** para crear una instancia nueva de Azure Integration Runtime.
   ![Creación de una instancia nueva de Azure Integration Runtime](./media/tutorial-copy-data-portal-private/create-new-azure-ir.png)
2. Elija crear una instancia de **Azure** Integration Runtime.
   ![Instancia nueva de Azure Integration Runtime](./media/tutorial-copy-data-portal-private/azure-ir.png)
3. Habilite **Red virtual**.
   ![Instancia nueva de Azure Integration Runtime](./media/tutorial-copy-data-portal-private/enable-managed-vnet.png)
4. Seleccione **Crear**.

## <a name="create-a-pipeline-with-a-data-flow-activity"></a>Creación de una canalización con una actividad de Data Flow

En este paso, creará una canalización que contiene una actividad de Data Flow.

1. En la página **Let's get started** (Introducción) seleccione **Create pipeline** (Crear canalización).

   ![Creación de una canalización](./media/doc-common-process/get-started-page.png)

1. En el panel **Propiedades** de la canalización, escriba **TransformMovies** como **Nombre** de la canalización.
1. En la barra superior de Data Factory, deslice el control deslizante **Depuración de Data Flow** para activarlo. El modo de depuración permite realizar pruebas interactivas de la lógica de transformación en un clúster de Spark activo. Los clústeres de Data Flow tardan de 5 a 7 minutos en prepararse y se recomienda que los usuarios activen primero la depuración si planean realizar el desarrollo de Data Flow. Para más información, consulte [Modo de depuración](https://docs.microsoft.com/azure/data-factory/concepts-data-flow-debug-mode).

    ![Depuración de un flujo de datos](media/tutorial-data-flow-private/dataflow-debug.png)
1. En el panel **Actividades** expanda el acordeón **Movimiento y transformación**. Arrastre y coloque la actividad **Data Flow** del panel al lienzo de la canalización.

1. En el elemento emergente **Adding data flow** (Agregar flujo de datos), seleccione **Create new Data Flow** (Crear un nuevo flujo de datos) y, luego, **Flujo de datos de asignación**. Pulse **OK** (Aceptar) cuando haya terminado.

    ![Flujo de datos de asignación](media/tutorial-data-flow-private/mapping-dataflow.png)

1. Asigne al flujo de datos el nombre **TransformMovies** en el panel Propiedades.

## <a name="build-transformation-logic-in-the-data-flow-canvas"></a>Generación de la lógica de transformación en el lienzo de flujo de datos

Una vez creado el flujo de datos, se le enviará automáticamente al lienzo flujo de datos. En este paso, creará un flujo de datos que toma el archivo moviesDB. csv en el almacenamiento de ADLS y agrega el promedio de clasificación de Comedias de 1910 a 2000. Después, volverá a escribir este archivo en el almacenamiento de ADLS.

### <a name="add-the-source-transformation"></a>Incorporación de la transformación de origen

En este paso, se configura Azure Data Lake Storage Gen2 como origen.

1. Para agregar un lienzo de flujo de datos, haga clic en el cuadro **Agregar origen** y agregue un origen.

1. Asigne un nombre al origen **MoviesDB**. Haga clic en **Nuevo** para crear un conjunto de datos de origen nuevo.

1. Elija **Azure Data Lake Storage Gen2**. Haga clic en Continue.

1. Elija **DelimitedText**. Haga clic en Continue.

1. Asigne un nombre al conjunto de datos **MoviesDB**. En la lista desplegable de servicios vinculados, elija **Nuevo**.

1. En la pantalla de creación de un servicio vinculado, asigne el nombre **ADLSGen2** al servicio vinculado ADLS gen2 y especifique el método de autenticación. A continuación, escriba las credenciales de conexión. En este tutorial, vamos a usar la clave de cuenta para conectarse a la cuenta de almacenamiento. 

1. Asegúrese de habilitar **Interactive Authoring** (Creación interactiva). Puede tardar aproximadamente 1 minuto en habilitarse.

    ![Creación interactiva](./media/tutorial-data-flow-private/interactive-authoring.png)

1. Seleccione **Probar conexión**, se debería producir un error, ya que la cuenta de almacenamiento no permite el acceso a ella sin la creación y aprobación de un punto de conexión privado. En el mensaje de error, debería ver un vínculo para crear un **punto de conexión privado** que puede usar para crear un punto de conexión privado administrado. *Una alternativa es ir directamente a la pestaña Administrar y seguir las instrucciones de [esta sección](#create-a-managed-private-endpoint) para crear un punto de conexión privado administrado*.

1. Mantenga abierto el cuadro de diálogo y, después, vaya a la cuenta de almacenamiento seleccionada anteriormente.

1. Siga las instrucciones de [esta sección](#approval-of-a-private-link-in-storage-account) para aprobar el vínculo privado.

1. Vuelva al cuadro de diálogo. Seleccione de nuevo **Prueba de conexión** y seleccione **Crear** para implementar el servicio vinculado.

1. Una vez que vuelva a la pantalla de creación del conjunto de datos, escriba la ubicación del archivo en el campo **Ruta de acceso de archivo**. En este tutorial, el archivo moviesDB.csv se encuentra en el contenedor sample-data. Como el archivo tiene encabezados, active **First row as header** (Primera fila como encabezado). Seleccione **From Connection/Store** (Desde la conexión o almacén) para importar el esquema de encabezado directamente desde el archivo en el almacenamiento. Haga clic en Aceptar cuando haya terminado.

    ![Ruta de origen](media/tutorial-data-flow-private/source-file-path.png)

1. Si se ha iniciado el clúster de depuración, vaya a la pestaña **Vista previa de los datos** de la transformación de origen y haga clic en **Actualizar** para obtener una instantánea de los datos. Puede usar la vista previa de los datos para comprobar que la transformación está configurada correctamente.

    ![Vista previa de datos](media/tutorial-data-flow-private/data-preview.png)

#### <a name="create-a-managed-private-endpoint"></a>Creación de un punto de conexión privado administrado

Si no ha hecho clic en el hipervínculo al probar la conexión anterior, siga la ruta de acceso que hay a continuación. Ahora debe crear un punto de conexión privado administrado que conectará al servicio vinculado creado anteriormente.

1. Vaya a la pestaña Administrar.
> [!NOTE]
> Es posible que esta no esté disponible para todas las instancias de Data Factory. Si no la ve, también puede acceder a los puntos de conexión privados mediante la pestaña "**Crear** --> "**Conexiones** --> "**Punto de conexión privado**".
1. Vaya a la sección Puntos de conexión privados administrados.
1. Seleccione **+ Nuevo** en Puntos de conexión privados administrados.

    ![Nuevo punto de conexión privado administrado](./media/tutorial-data-flow-private/new-managed-private-endpoint.png) 

1. Seleccione el icono de Azure Data Lake Storage Gen2 de la lista y seleccione **Continuar**.
1. Escriba el nombre de la cuenta de almacenamiento que creó anteriormente.
1. Seleccione **Crear**.
1. Después de esperar algunos segundos, debería ver que el vínculo privado creado necesita una aprobación.
1. Seleccione el punto de conexión privado que creó anteriormente. Puede ver un hipervínculo que le llevará a la página para aprobar el punto de conexión privado en el nivel de cuenta de almacenamiento.

    ![Administración de un punto de conexión privado](./media/tutorial-data-flow-private/manage-private-endpoint.png) 

#### <a name="approval-of-a-private-link-in-storage-account"></a>Aprobación de un vínculo privado en una cuenta de almacenamiento

1. En la cuenta de almacenamiento, vaya a **Conexiones de punto de conexión privado** en la sección Configuración.

1. Marque el punto de conexión privado que se ha creado anteriormente y seleccione **Aprobar**.

    ![Aprobación de un punto de conexión privado](./media/tutorial-data-flow-private/approve-private-endpoint.png)

1. Agregue una descripción y haga clic en **Sí**.
1. Vuelva a la sección **Puntos de conexión privados administrados** de la pestaña **Administrar** de Azure Data Factory.
1. La aprobación tardará aproximadamente 1 minuto en verse reflejada para el punto de conexión privado.

### <a name="add-the-filter-transformation"></a>Incorporación de la transformación filtrar

1. Junto al nodo de origen en el lienzo de flujo de datos, haga clic en el icono de signo más para agregar una nueva transformación. La primera transformación que va a agregar es un **filtro**.

    ![Adición de filtro](media/tutorial-data-flow-private/add-filter.png)
1. Denomine **FilterYears** a la transformación de filtro. Haga clic en el cuadro de expresión junto a **Filtro en** para abrir el generador de expresiones. Aquí especificará la condición de filtrado.

    ![Filtro de años](media/tutorial-data-flow-private/filter-years.png)
1. El generador de expresiones de flujo de datos le permite compilar de forma interactiva expresiones para utilizarlas en varias transformaciones. Las expresiones pueden incluir funciones integradas, columnas del esquema de entrada y parámetros definidos por el usuario. Para más información sobre cómo compilar expresiones, vea [Generador de expresiones de Mapping Data Flow](https://docs.microsoft.com/azure/data-factory/concepts-data-flow-expression-builder).

    * En este tutorial, desea filtrar las películas del género de comedia que se estrenaron entre los años 1910 y 2000. Dado que el año es actualmente una cadena, debe convertirlo en un entero mediante la función ```toInteger()```. Use los operadores mayor o igual que (> =) y menor o igual que (< =) para comparar con los valores de año literal 1910 y 200-. Una estas expresiones junto con el operador and (&&). La expresión aparece como:

        ```toInteger(year) >= 1910 && toInteger(year) <= 2000```

    * Para averiguar qué películas son comedias, puede usar la función ```rlike()``` para buscar el patrón " comedia" en la columna de géneros. Una la expresión rlike con la comparación de año para obtener:

        ```toInteger(year) >= 1910 && toInteger(year) <= 2000 && rlike(genres, 'Comedy')```

    * Si tiene un clúster de depuración activo, puede comprobar la lógica; para ello, haga clic en **Actualizar** para ver la salida de la expresión en comparación con las entradas usadas. Hay más de una respuesta correcta sobre cómo puede realizar esta lógica mediante el lenguaje de expresiones de flujo de datos.

        ![Expresión de filtro](media/tutorial-data-flow-private/filter-expression.png)

    * Haga clic en **Guardar y finalizar** una vez que haya terminado con la expresión.

1. Capture una **Vista previa de datos** para comprobar que el filtro funciona correctamente.

    ![Filtro de datos, versión preliminar](media/tutorial-data-flow-private/filter-data.png)

### <a name="add-the-aggregate-transformation"></a>Incorporación de la transformación de agregado

1. La transformación siguiente que se va a agregar es una transformación de **agregado** en **Schema Modifier** (Modificador de esquema).

    ![Incorporación de agregado](media/tutorial-data-flow-private/add-aggregate.png)
1. Denomine **AggregateComedyRatings** a la transformación de agregado. En la pestaña **Agrupar por**, seleccione **year** (año) en la lista desplegable para agrupar las agregaciones por el año en que apareció la película.

    ![Grupo agregado](media/tutorial-data-flow-private/group-by-year.png)
1. Vaya a la pestaña **Agregados**. En el cuadro de texto de la izquierda, asigne a la columna agregada el nombre **AverageComedyRating**. Haga clic en el cuadro de la expresión derecha para especificar la expresión de agregado a través del generador de expresiones.

    ![Nombre de columna agregada](media/tutorial-data-flow-private/name-column.png)
1. Para obtener el promedio de la columna **Rating** (Clasificación), use la función de agregado ```avg()```. Como **Rating** (Clasificación) es una cadena y ```avg()``` toma una entrada numérica, debemos convertir el valor en un número a través de la función ```toInteger()```. Se trata de una expresión similar a la siguiente:

    ```avg(toInteger(Rating))```

    Haga clic en **Guardar y finalizar** cuando haya terminado.

    ![Guardado de agregado](media/tutorial-data-flow-private/save-aggregate.png)
1. Vaya a la pestaña **Vista previa de datos** para ver la salida de la transformación. Observe que solo hay dos columnas, **year** y **AverageComedyRating**.

### <a name="add-the-sink-transformation"></a>Incorporación de la transformación de receptor

1. A continuación, desea agregar una transformación de **receptor** en **Destino**.

    ![Incorporación de receptor](media/tutorial-data-flow-private/add-sink.png)
1. Asigne un nombre al receptor **Sink** (Receptor). Haga clic en **Nuevo** para crear un conjunto de datos de receptor.

    ![Creación de receptor](media/tutorial-data-flow-private/create-sink.png)
1. En la página Nuevo conjunto de datos, elija **Azure Data Lake Storage Gen2**. Haga clic en Continue.

1. En la página Seleccionar formato, elija **DelimitedText**. Haga clic en Continue.

1. Asigne el nombre **MoviesSink** al conjunto de datos de receptor. Para el servicio vinculado, elija el mismo servicio vinculado ADLSGen2 que se ha creado para la transformación del origen. Escriba una carpeta de salida en la que escribir los datos. En este tutorial, vamos a escribir en la carpeta "output" (salida) en el contenedor "sample-data" (datos de ejemplo). No es necesario que la carpeta exista de antemano y se puede crear dinámicamente. Establezca **Usar la primera fila como encabezado** en true y seleccione **Ninguna** en **Importar esquema**. Haga clic en Aceptar.

    ![Ruta de acceso del receptor](media/tutorial-data-flow-private/sink-file-path.png)

Ahora ha terminado de crear el flujo de datos. Está preparado para ejecutarlo en la canalización.

## <a name="running-and-monitoring-the-data-flow"></a>Ejecución y supervisión del flujo de datos

Puede depurar una canalización antes de publicarla. En este paso, va a desencadenar una ejecución de depuración de la canalización de flujo de datos. Aunque la vista previa de los datos no escribe datos, una ejecución de depuración escribirá datos en el destino del receptor.

1. Vaya al lienzo de la canalización. Haga clic en **Depurar** para desencadenar una ejecución de depuración.

1. La depuración de canalización de actividades de Data Flow usa el clúster de depuración activo, pero sigue tardando al menos un minuto en inicializarse. Puede realizar un seguimiento del progreso a través de la pestaña **Salida**. Una vez que la ejecución se realice correctamente, haga clic en el icono de anteojos para ver los detalles.

1. En el panel Detalles, puede ver el número de filas y el tiempo invertido en cada paso de transformación.

    ![Ejecución de la supervisión](media/tutorial-data-flow-private/run-details.png)
1. Haga clic en una transformación para obtener información detallada sobre las columnas y las particiones de los datos.

Si siguió correctamente este tutorial, debe haber escrito 83 filas y 2 columnas en la carpeta del receptor. Puede comprobar el almacenamiento de blobs para confirmar que los datos son correctos.

## <a name="summary"></a>Resumen

En este tutorial, usará la interfaz de usuario de Azure Data Factory (UX) para crear una canalización que copie y transforme los datos **desde un origen de Azure Data Lake Storage (ADLS) Gen2 hasta un receptor de ADLS Gen2 (ambos permitiendo el acceso solo a las redes seleccionadas)** mediante un flujo de datos de asignación en una [red virtual administrada por Azure Data Factory](managed-virtual-network-private-endpoint.md).
