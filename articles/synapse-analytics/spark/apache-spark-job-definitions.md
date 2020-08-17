---
title: 'Tutorial: Creación de una definición de trabajo de Apache Spark en Synapse Studio'
description: 'Tutorial: Uso de Azure Synapse Analytics para crear definiciones de trabajos de Spark y enviarlas a un grupo de Apache Spark para Azure Synapse Analytics.'
author: hrasheed-msft
ms.author: jejiang
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: tutorial
ms.subservice: ''
ms.date: 04/15/2020
ms.openlocfilehash: ac3e163ffefcb7b164860b0c4fa42edc866227e3
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87065621"
---
# <a name="tutorial-create-apache-spark-job-definition-in-synapse-studio"></a>Tutorial: Creación de una definición de trabajo de Apache Spark en Synapse Studio

En este tutorial se muestra cómo usar Azure Synapse Studio para crear definiciones de trabajos de Apache Spark y enviarlas luego a un grupo de Apache Spark.

En este tutorial se describen las tareas siguientes:

* Creación de una definición de trabajo de Apache Spark para PySpark (Python)
* Creación de una definición de trabajo de Apache Spark para Spark (Scala)
* Creación de una definición de trabajo de Apache Spark para .NET Spark (C#/F#)
* Envío de una definición de trabajo de Apache Spark como un trabajo por lotes
* Adición de una definición de trabajo de Apache Spark en la canalización

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar este tutorial, asegúrese de que se cumplen los requisitos siguientes:

* Un área de trabajo de Azure Synapse Analytics. Para obtener instrucciones, consulte [Creación de un área de trabajo de Azure Synapse Analytics](../../machine-learning/how-to-manage-workspace.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json#create-a-workspace).
* Un grupo de Apache Spark
* Una cuenta de almacenamiento de ADLS Gen2. Es necesario ser el propietario de los datos del blob de almacenamiento del sistema de archivos de ADLS Gen2 con el que quiere trabajar. Si no lo es, debe agregar el permiso manualmente.

## <a name="create-an-apache-spark-job-definition-for-pyspark-python"></a>Creación de una definición de trabajo de Apache Spark para PySpark (Python)

En esta sección, creará una definición de trabajo de Apache Spark para PySpark (Python).

1. Abra [Azure Synapse Studio](https://web.azuresynapse.net/).

2. Puede ir a [Sample files for creating Apache Spark job definitions](https://github.com/Azure-Samples/Synapse/tree/master/Spark/Python) (Archivos de ejemplo para crear definiciones de trabajos de Apache Spark) para descargar **wordcount.zip** y **shakespear.txt**. Luego, cargue estos archivos en Azure Storage: Haga clic en **Datos**, seleccione **Cuentas de almacenamiento** y cargue los archivos relacionados en el sistema de archivos de ADLS Gen2. Si los archivos ya están en Azure Storage, omita este paso. 

     ![Carga del archivo de Python](./media/apache-spark-job-definitions/upload-python-file.png)

3. Haga clic en el centro de conectividad **Desarrollar**, seleccione **Definición de trabajos de Spark** en el panel izquierdo, haga clic en el nodo de acción "…" junto a **Definición de trabajos de Spark** y, luego, seleccione **New Spark job definition** (Nueva definición de trabajo de Spark) en el menú contextual.

     ![Creación de una definición para Python](./media/apache-spark-job-definitions/create-new-definition.png)

4. Seleccione **PySpark (Python)** en la lista desplegable de lenguajes de la ventana principal de definiciones de trabajos de Apache Spark.

5. Rellene la información de la definición de trabajo de Apache Spark. Puede copiar la información de ejemplo.

     |  Propiedad   | Descripción   |  
     | ----- | ----- |  
     |Job definition name (Nombre de la definición de trabajo)| Proporcione un nombre para la definición de trabajo de Apache Spark. Este nombre se puede actualizar en cualquier momento hasta que se publique. Ejemplo: `job definition sample`|
     |Main definition file (Archivo de definición principal)| Archivo principal usado para el trabajo. Seleccione un archivo PY en el almacenamiento. Puede seleccionar **Upload file** (Cargar archivo) para cargar un archivo en una cuenta de almacenamiento. Ejemplo: `abfss://…/path/to/wordcount.py`|
     |Argumentos de la línea de comandos| Argumentos opcionales del trabajo. Ejemplo: `abfss://…/path/to/shakespeare.txt abfss://…/path/to/result`|
     |Reference files (Archivos de referencia)| Archivos adicionales usados como referencia en el archivo de definición principal. Puede seleccionar **Upload file** (Cargar archivo) para cargar un archivo en una cuenta de almacenamiento. |
     |Grupo de Spark| El trabajo se enviará al grupo de Apache Spark seleccionado.|
     |Versión de Spark| Versión de Apache Spark que ejecuta el grupo de Apache Spark.|
     |Ejecutores| Número de ejecutores que se van a proporcionar en el grupo de Apache Spark especificado para el trabajo.|
     |Executor size (Tamaño del ejecutor)| Número de núcleos y memoria que se van a usar para los ejecutores proporcionados en el grupo de Apache Spark especificado para el trabajo.|  
     |Driver size (Tamaño del controlador)| Número de núcleos y memoria que se van a usar para el controlador proporcionado en el grupo de Apache Spark especificado para el trabajo.|

     ![Establecimiento del valor de la definición de trabajo de Spark para Python](./media/apache-spark-job-definitions/create-py-definition.png)

6. Seleccione **Publicar** para guardar la definición de trabajo de Apache Spark.

     ![Publicación de la definición de Python](./media/apache-spark-job-definitions/publish-py-definition.png)

## <a name="create-an-apache-spark-job-definition-for-apache-sparkscala"></a>Creación de una definición de trabajo de Apache Spark para Apache Spark (Scala)

En esta sección, creará una definición de trabajo de Apache Spark para Apache Spark (Scala).

 1. Abra [Azure Synapse Studio](https://web.azuresynapse.net/).

 2. Puede ir a [Sample files for creating Apache Spark job definitions](https://github.com/Azure-Samples/Synapse/tree/master/Spark/Scala) (Archivos de ejemplo para crear definiciones de trabajos de Apache Spark) para descargar **wordcount.jar** y **shakespear.txt**. Luego, cargue estos archivos en Azure Storage: Haga clic en **Datos**, seleccione **Cuentas de almacenamiento** y cargue los archivos relacionados en el sistema de archivos de ADLS Gen2. Si los archivos ya están en Azure Storage, omita este paso. 
 
     ![Preparación de la estructura de Scala](./media/apache-spark-job-definitions/prepare-scala-structure.png)

 3. Haga clic en el centro de conectividad **Desarrollar**, seleccione **Definición de trabajos de Spark** en el panel izquierdo, haga clic en el nodo de acción "…" junto a **Definición de trabajos de Spark** y, luego, seleccione **New Spark job definition** (Nueva definición de trabajo de Spark) en el menú contextual.
     ![Creación de una definición para Scala](./media/apache-spark-job-definitions/create-new-definition.png)

 4. Seleccione **Spark (Scala)** en la lista desplegable de lenguajes de la ventana principal de definiciones de trabajos de Apache Spark.

 5. Rellene la información de la definición de trabajo de Apache Spark. Puede copiar la información de ejemplo.

     |  Propiedad   | Descripción   |  
     | ----- | ----- |  
     |Job definition name (Nombre de la definición de trabajo)| Proporcione un nombre para la definición de trabajo de Apache Spark. Este nombre se puede actualizar en cualquier momento hasta que se publique. Ejemplo: `job definition sample`|
     |Main definition file (Archivo de definición principal)| Archivo principal usado para el trabajo. Seleccione un archivo JAR del almacenamiento. Puede seleccionar **Upload file** (Cargar archivo) para cargar un archivo en una cuenta de almacenamiento. Ejemplo: `abfss://…/path/to/wordcount.jar`|
     |Nombre de clase principal| Identificador completo o clase principal que se encuentra en el archivo de definición principal. Ejemplo: `WordCount`|
     |Argumentos de la línea de comandos| Argumentos opcionales del trabajo. Ejemplo: `abfss://…/path/to/shakespeare.txt abfss://…/path/to/result`|
     |Reference files (Archivos de referencia)| Archivos adicionales usados como referencia en el archivo de definición principal. Puede seleccionar **Upload file** (Cargar archivo) para cargar un archivo en una cuenta de almacenamiento.|
     |Grupo de Spark| El trabajo se enviará al grupo de Apache Spark seleccionado.|
     |Versión de Spark| Versión de Apache Spark que ejecuta el grupo de Apache Spark.|
     |Ejecutores| Número de ejecutores que se van a proporcionar en el grupo de Apache Spark especificado para el trabajo.|  
     |Executor size (Tamaño del ejecutor)| Número de núcleos y memoria que se van a usar para los ejecutores proporcionados en el grupo de Apache Spark especificado para el trabajo.|
     |Driver size (Tamaño del controlador)| Número de núcleos y memoria que se van a usar para el controlador proporcionado en el grupo de Apache Spark especificado para el trabajo.|

     ![Establecimiento del valor de la definición de trabajo de Spark para Scala](./media/apache-spark-job-definitions/create-scala-definition.png)

 6. Seleccione **Publicar** para guardar la definición de trabajo de Apache Spark.

     ![Publicación de la definición de Scala](./media/apache-spark-job-definitions/publish-scala-definition.png)


## <a name="create-an-apache-spark-job-definition-for-net-sparkcf"></a>Creación de una definición de trabajo de Apache Spark para .NET Spark (C#/F#)

En esta sección, creará una definición de trabajo de Apache Spark para .NET Spark (C#/F#).
 1. Abra [Azure Synapse Studio](https://web.azuresynapse.net/).

 2. Puede ir a [Sample files for creating Apache Spark job definitions](https://github.com/Azure-Samples/Synapse/tree/master/Spark/DotNET) (Archivos de ejemplo para crear definiciones de trabajos de Apache Spark) para descargar **wordcount.zip** y **shakespear.txt**. Luego, cargue estos archivos en Azure Storage: Haga clic en **Datos**, seleccione **Cuentas de almacenamiento** y cargue los archivos relacionados en el sistema de archivos de ADLS Gen2. Si los archivos ya están en Azure Storage, omita este paso. 

     ![Preparación de la estructura de dotnet](./media/apache-spark-job-definitions/prepare-scala-structure.png)

 3. Haga clic en el centro de conectividad **Desarrollar**, seleccione **Definición de trabajos de Spark** en el panel izquierdo, haga clic en el nodo de acción "…" junto a **Definición de trabajos de Spark** y, luego, seleccione **New Spark job definition** (Nueva definición de trabajo de Spark) en el menú contextual.

     ![Creación de una definición para dotnet](./media/apache-spark-job-definitions/create-new-definition.png)

 4. Seleccione **.NET Spark(C#/F#)** en la lista desplegable de lenguajes de la ventana principal de definición de trabajos de Apache Spark.

 5. Rellene la información de la definición de trabajo de Apache Spark. Puede copiar la información de ejemplo.
    
     |  Propiedad   | Descripción   |  
     | ----- | ----- |  
     |Job definition name (Nombre de la definición de trabajo)| Proporcione un nombre para la definición de trabajo de Apache Spark. Este nombre se puede actualizar en cualquier momento hasta que se publique. Ejemplo: `job definition sample`|
     |Main definition file (Archivo de definición principal)| Archivo principal usado para el trabajo. Seleccione un archivo ZIP que contenga la aplicación .NET para Apache Spark (es decir, el archivo ejecutable principal, los archivos DLL que contienen las funciones definidas por el usuario y otros archivos necesarios) del almacenamiento. Puede seleccionar **Upload file** (Cargar archivo) para cargar un archivo en una cuenta de almacenamiento. Ejemplo: `abfss://…/path/to/wordcount.zip`|
     |Main executable file (Archivo ejecutable principal)| Archivo ejecutable principal del archivo ZIP de la definición principal. Ejemplo: `WordCount`|
     |Argumentos de la línea de comandos| Argumentos opcionales del trabajo. Ejemplo: `abfss://…/path/to/shakespeare.txt abfss://…/path/to/result`|
     |Reference files (Archivos de referencia)| Archivos adicionales necesarios para que los nodos de trabajo ejecuten la aplicación de .NET para Apache Spark y que no se incluyen en el archivo ZIP de la definición principal (es decir, archivos JAR dependientes, archivos DLL adicionales de funciones definidas por el usuario y otros archivos de configuración). Puede seleccionar **Upload file** (Cargar archivo) para cargar un archivo en una cuenta de almacenamiento.|
     |Grupo de Spark| El trabajo se enviará al grupo de Apache Spark seleccionado.|
     |Versión de Spark| Versión de Apache Spark que ejecuta el grupo de Apache Spark.|
     |Ejecutores| Número de ejecutores que se van a proporcionar en el grupo de Apache Spark especificado para el trabajo.|  
     |Executor size (Tamaño del ejecutor)| Número de núcleos y memoria que se van a usar para los ejecutores proporcionados en el grupo de Apache Spark especificado para el trabajo.|
     |Driver size (Tamaño del controlador)| Número de núcleos y memoria que se van a usar para el controlador proporcionado en el grupo de Apache Spark especificado para el trabajo.|

     ![Establecimiento del valor de la definición de trabajo de Spark para dotnet](./media/apache-spark-job-definitions/create-net-definition.png)

 6. Seleccione **Publicar** para guardar la definición de trabajo de Apache Spark.

      ![Publicación de la definición de dotnet](./media/apache-spark-job-definitions/publish-net-definition.png)

## <a name="submit-an-apache-spark-job-definition-as-a-batch-job"></a>Envío de una definición de trabajo de Apache Spark como un trabajo por lotes

Después de crear una definición de trabajo de Apache Spark, puede enviarla a un grupo de Apache Spark. Asegúrese de ser el propietario de los datos del blob de almacenamiento del sistema de archivos de ADLS Gen2 con el que quiere trabajar. Si no lo es, debe agregar el permiso manualmente.

### <a name="scenario-1-submit-apache-spark-job-definition"></a>Escenario 1: Envío de la definición de trabajo de Apache Spark
 1. Haga clic en una ventana de definición de trabajo de Apache Spark para abrirla.

      ![Abrir definición de trabajo de Spark para enviar ](./media/apache-spark-job-definitions/open-spark-definition.png)

 2. Haga clic en el icono **Enviar** para enviar el proyecto al grupo de Apache Spark seleccionado. Puede hacer clic en la pestaña **Spark monitoring URL** (Dirección URL de supervisión de Spark) para ver la consulta de registro de la aplicación de Apache Spark.

    ![Clic en el botón Enviar para enviar la definición de trabajo de Spark](./media/apache-spark-job-definitions/submit-spark-definition.png)

    ![Cuadro de diálogo de envío de Spark](./media/apache-spark-job-definitions/submit-definition-result.png)

### <a name="scenario-2-view-apache-spark-job-running-progress"></a>Escenario 2: Visualización del progreso de ejecución del trabajo de Apache Spark

 1. Haga clic en **Monitor** (Supervisar) y, a continuación, seleccione la opción **Spark applications** (Aplicaciones Spark). Podrá encontrar la aplicación de Apache Spark enviada.

     ![Ver aplicación Spark](./media/apache-spark-job-definitions/view-spark-application.png)

 2. Luego, haga clic en la aplicación de Apache Spark para mostrar la ventana **Log Query** (Consulta de registro). Puede ver el progreso de la ejecución del trabajo desde **Log Query** (Consulta del registro).
     
     ![Ver consulta del registro de la aplicación Spark](./media/apache-spark-job-definitions/view-job-log-query.png)

### <a name="scenario-3-check-output-file"></a>Escenario 3: Comprobación del archivo de salida

 1. Haga clic en **Data** (Datos) y, a continuación, seleccione **Storage accounts** (Cuentas de almacenamiento). Después de una ejecución correcta, puede ir al almacenamiento de ADLS Gen2 y comprobar que se han generado las salidas.

     ![Ver archivo de salida](./media/apache-spark-job-definitions/view-output-file.png)

## <a name="add-an-apache-spark-job-definition-into-pipeline"></a>Adición de una definición de trabajo de Apache Spark en la canalización

En esta sección, agregará una definición de trabajo de Apache Spark en la canalización.

 1. Abra una definición de trabajo de Apache Spark existente.

 2. Haga clic en el icono de la parte superior derecha de la definición de trabajo de Apache Spark, elija **Existing Pipeline** (Canalización existente) o **New pipeline** (Nueva canalización). Puede consultar la página Pipeline (Canalización) para más información.

     ![Agregar a la canalización](./media/apache-spark-job-definitions/add-to-pipeline01.png)

     ![Agregar a la canalización](./media/apache-spark-job-definitions/add-to-pipeline02.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial se muestra cómo usar Azure Synapse Studio para crear definiciones de trabajos de Apache Spark y enviarlas luego a un grupo de Apache Spark. Luego, puede usar Azure Synapse Studio para crear conjuntos de datos de Power BI y administrar los datos de Power BI.

