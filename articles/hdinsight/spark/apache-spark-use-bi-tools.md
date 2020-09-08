---
title: 'Tutorial: Análisis de los datos de Apache Spark en Azure HDInsight con Power BI'
description: 'Tutorial: Uso de Microsoft Power BI para visualizar datos de Apache Spark almacenados en clústeres de HDInsight'
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: tutorial
ms.custom: hdinsightactive,mvc,seoapr2020
ms.date: 04/21/2020
ms.openlocfilehash: 4eb10298ac683c991835f86582d82fb952b314b2
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "82195118"
---
# <a name="tutorial-analyze-apache-spark-data-using-power-bi-in-hdinsight"></a>Tutorial: Análisis de datos de Apache Spark mediante Power BI en HDInsight

En este tutorial, aprenderá a utilizar Microsoft Power BI para visualizar datos en un clúster de Apache Spark en Azure HDInsight.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Visualizar datos de Spark mediante Power BI

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

* Completar el artículo [Tutorial: Carga de datos y ejecución de consultas en un clúster de Apache Spark en Azure HDInsight](./apache-spark-load-data-run-query.md).

* [Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/).

* Opcional: [Suscripción de evaluación de Power BI](https://app.powerbi.com/signupredirect?pbi_source=web).

## <a name="verify-the-data"></a>Comprobación de los datos

La instancia de [Jupyter Notebook](https://jupyter.org/) que creó en el [tutorial anterior](apache-spark-load-data-run-query.md) incluye código para crear una tabla `hvac`. Esta tabla se basa en el archivo CSV en todos los clústeres de Spark de HDInsight en `\HdiSamples\HdiSamples\SensorSampleData\hvac\hvac.csv`. Use el siguiente procedimiento para comprobar los datos.

1. Desde el cuaderno de Jupyter, pegue el siguiente código y presione **MAYÚS + ENTRAR**. El código comprueba la existencia de las tablas.

    ```PySpark
    %%sql
    SHOW TABLES
    ```

    El resultado tendrá una apariencia similar a la siguiente:

    ![Se muestran tablas en Spark](./media/apache-spark-use-bi-tools/apache-spark-show-tables.png)

    Si ha cerrado el bloc de notas antes de iniciar este tutorial, `hvactemptable` se limpia, por lo que no se incluye en los resultados.  Desde las herramientas de BI, solo se puede acceder a las tablas de Hive almacenadas en Metastore (indicadas como **False** en la columna **isTemporary**). En este tutorial, se conecta a la tabla **hvac** que ha creado.

2. Pegue el siguiente código en una celda vacía y presione **MAYÚS + ENTRAR**. El código comprueba los datos de la tabla.

    ```PySpark
    %%sql
    SELECT * FROM hvac LIMIT 10
    ```

    El resultado tendrá una apariencia similar a la siguiente:

    ![Se muestran las filas de la tabla hvac en Spark](./media/apache-spark-use-bi-tools/apache-spark-select-limit.png)

3. En el menú **File** (Archivo) del cuaderno, seleccione **Close and Halt** (Cerrar y detener). Cierre el cuaderno para liberar los recursos.

## <a name="visualize-the-data"></a>Visualización de los datos

En esta sección, se usa Power BI para crear visualizaciones, informes y paneles de datos a partir de los datos de clúster de Spark.

### <a name="create-a-report-in-power-bi-desktop"></a>Creación de un informe en Power BI Desktop

Los primeros pasos para trabajar con Spark pasan por conectarse al clúster de Power BI Desktop, cargar datos del clúster y crear una visualización básica basada en dichos datos.

1. Abra Power BI Desktop. Cierre la pantalla de presentación inicial si se abre.

2. En la pestaña **Inicio**, vaya a **Obtener datos** > **Más...** .

    ![Obtención de datos en Power BI Desktop desde HDInsight Apache Spark](./media/apache-spark-use-bi-tools/hdinsight-spark-power-bi-desktop-get-data.png "Obtención de datos en Power BI desde Apache Spark BI")

3. Escriba `Spark` en el cuadro de búsqueda, seleccione **Azure HDInsight Spark** y, luego, seleccione **Conectar**.

    ![Obtención de datos en Power BI desde Apache Spark BI](./media/apache-spark-use-bi-tools/apache-spark-bi-import-data-power-bi.png "Obtención de datos en Power BI desde Apache Spark BI")

4. Escriba la dirección URL del clúster (en el formulario `mysparkcluster.azurehdinsight.net`) en el cuadro de texto **Servidor**.

5. En **Modo de conectividad de datos**, seleccione **DirectQuery**. Después, seleccione **Aceptar**.

    Puede usar cualquier modo de conectividad de datos con Spark. Si usa DirectQuery, los cambios se reflejan en los informes sin tener que actualizar el conjunto de datos completo. Si importa los datos, deberá actualizar el conjunto de datos para ver los cambios. Para obtener más información sobre cómo y cuándo se debe usar DirectQuery, consulte [Uso de DirectQuery en Power BI](https://powerbi.microsoft.com/documentation/powerbi-desktop-directquery-about/).

6. Escriba la información de la cuenta de inicio de sesión de HDInsight y seleccione **Conectar**. El nombre de cuenta predeterminado es *admin*.

7. Seleccione la tabla `hvac`, espere para obtener una vista previa de los datos y seleccione **Cargar**.

    ![Nombre de usuario y contraseña del clúster de Spark](./media/apache-spark-use-bi-tools/apache-spark-bi-select-table.png "Nombre de usuario y contraseña del clúster de Spark")

    Power BI Desktop tiene toda la información necesaria para conectarse a los datos de carga y al clúster de Spark desde la tabla `hvac`. La tabla y las columnas que la forman se muestran en el panel **Campos**.

8. Visualice la variación entre la temperatura objetivo y la real para cada edificio:

    1. En el panel **VISUALIZACIONES**, seleccione **Gráfico de áreas**.

    2. Arrastre el campo **BuildingID** a **Eje** y arrastre los campos **ActualTemp** y **TargetTemp** a **Valor**.

        ![Incorporación de columnas de valor](./media/apache-spark-use-bi-tools/apache-spark-bi-add-value-columns.png "Incorporación de columnas de valor")

        El diagrama tiene el siguiente aspecto:

        ![Suma en gráfico de áreas](./media/apache-spark-use-bi-tools/apache-spark-bi-area-graph-sum.png "Suma en gráfico de áreas")

        De manera predeterminada, la visualización muestra la suma de **ActualTemp** y **TargetTemp**. Seleccione la flecha abajo junto a **ActualTemp** y **TragetTemp** en el panel Visualizaciones; puede ver que **Suma** se ha seleccionado.

    3. Seleccione las flechas abajo junto a **ActualTemp** y **TragetTemp** en el panel Visualizaciones, seleccione **Media** para obtener un promedio de temperaturas reales y objetivo para cada edificio.

        ![Media de valores](./media/apache-spark-use-bi-tools/apache-spark-bi-average-of-values.png "Media de valores")

        La visualización de datos debe parecerse a la que se muestra en la captura de pantalla. Mueva el cursor sobre la visualización para obtener información sobre herramientas con datos relevantes.

        ![Gráfico de áreas](./media/apache-spark-use-bi-tools/apache-spark-bi-area-graph.png "Gráfico de áreas")

9. Vaya a **Archivo** > **Guardar**, escriba el nombre `BuildingTemperature` para el archivo y, a continuación, seleccione **Guardar**.

### <a name="publish-the-report-to-the-power-bi-service-optional"></a>Publicar el informe en el servicio Power BI (opcional)

El servicio Power BI le permite compartir informes y paneles a través de su organización. En esta sección, primero publica el conjunto de datos y el informe. A continuación, puede anclar el informe a un panel. Los paneles suelen usarse para centrarse en un subconjunto de datos de un informe. Solo tiene una visualización del informe, pero sigue siendo útil seguir los pasos.

1. Abra Power BI Desktop.

1. Desde la pestaña **Inicio**, seleccione **Publicar**.

    ![Publicar desde Power BI Desktop](./media/apache-spark-use-bi-tools/apache-spark-bi-publish.png "Publicar desde Power BI Desktop")

1. Seleccione un área de trabajo en la que publicar el conjunto de datos y el informe, y seleccione **Seleccionar**. En la siguiente imagen, está seleccionado el valor predeterminado **Mi área de trabajo**.

    ![Selección del área de trabajo donde se va a publicar el conjunto de datos y el informe](./media/apache-spark-use-bi-tools/apache-spark-bi-select-workspace.png "Selección del área de trabajo donde se va a publicar el conjunto de datos y el informe")

1. Después de que la publicación se haya realizado correctamente, seleccione **Abrir 'BuildingTemperature.pbix' en Power BI**.

    ![Publicación correcta, clic para escribir las credenciales](./media/apache-spark-use-bi-tools/apache-spark-bi-publish-success.png "Publicación correcta, clic para escribir las credenciales")

1. En el servicio Power BI, seleccione **Escribir credenciales**.

    ![Introducción de credenciales en el servicio Power BI](./media/apache-spark-use-bi-tools/apache-spark-bi-enter-credentials.png "Introducción de credenciales en el servicio Power BI")

1. Seleccione **Editar credenciales**.

    ![Edición de credenciales en el servicio Power BI](./media/apache-spark-use-bi-tools/apache-spark-bi-edit-credentials.png "Edición de credenciales en el servicio Power BI")

1. Escriba la información de la cuenta de inicio de sesión de HDInsight y seleccione **Iniciar sesión**. El nombre de cuenta predeterminado es *admin*.

    ![Inicio de sesión en el clúster de Spark](./media/apache-spark-use-bi-tools/apache-spark-bi-sign-in.png "Inicio de sesión en el clúster de Spark")

1. En el panel izquierdo, vaya a **Áreas de trabajo** > **Mi área de trabajo** > **INFORMES** y seleccione **BuildingTemperature**.

    ![Informe incluido en los informes del panel izquierdo](./media/apache-spark-use-bi-tools/apache-spark-bi-service-left-pane.png "Informe incluido en los informes del panel izquierdo")

    También verá **BuildingTemperature** en el panel izquierdo, debajo de **CONJUNTOS DE DATOS**.

    Ahora el objeto visual creado en Power BI Desktop está disponible en el servicio Power BI.

1. Mantenga el cursor sobre la visualización y seleccione el icono de anclaje en la esquina superior derecha.

    ![Informe en el servicio Power BI](./media/apache-spark-use-bi-tools/apache-spark-bi-service-report.png "Informe en el servicio Power BI")

1. Seleccione "Nuevo panel", escriba el nombre `Building temperature` y después seleccione **Anclar**.

    ![Anclaje al nuevo panel](./media/apache-spark-use-bi-tools/apache-spark-bi-pin-dashboard.png "Anclaje al nuevo panel")

1. En el informe, seleccione **Ir al panel**.

El objeto visual se ancla al panel. Puede agregar otros elementos visuales al informe y anclarlos al mismo panel. Para más información acerca de los informes y paneles, consulte [Informes de Power BI](https://powerbi.microsoft.com/documentation/powerbi-service-reports/) y [Paneles de Power BI](https://powerbi.microsoft.com/documentation/powerbi-service-dashboards/).

## <a name="clean-up-resources"></a>Limpieza de recursos

Después de completar el tutorial, puede ser conveniente eliminar el clúster. Con HDInsight, los datos se almacenan en Azure Storage, por lo que puede eliminar un clúster de forma segura cuando no se esté usando. Los clústeres de HDInsight se cobran aunque no se estén usando. Como en muchas ocasiones los cargos por el clúster son mucho más elevados que los cargos por el almacenamiento, desde el punto de vista económico tiene sentido eliminar clústeres cuando no se usen.

Para eliminar un clúster, consulte [Eliminación de un clúster de HDInsight con el explorador, PowerShell o la CLI de Azure](../hdinsight-delete-cluster.md).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a utilizar Microsoft Power BI para visualizar datos en un clúster de Apache Spark en Azure HDInsight. Vaya al siguiente artículo para saber cómo crear una aplicación de Machine Learning.

> [!div class="nextstepaction"]
> [Creación de una aplicación de Machine Learning](./apache-spark-ipython-notebook-machine-learning.md)