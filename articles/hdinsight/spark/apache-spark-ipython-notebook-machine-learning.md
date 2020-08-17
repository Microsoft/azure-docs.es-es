---
title: 'Tutorial: Creación de una aplicación de aprendizaje automático de Spark en Azure HDInsight'
description: 'Tutorial: Instrucciones detalladas acerca de cómo compilar una aplicación de aprendizaje automático de Apache Spark en clústeres Spark de HDInsight mediante Jupyter Notebook.'
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: tutorial
ms.custom: hdinsightactive,mvc, devx-track-python
ms.date: 04/07/2020
ms.openlocfilehash: 0029761f8a1c48ab595dfa5bc2a87fca12d90e3d
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87873932"
---
# <a name="tutorial-build-an-apache-spark-machine-learning-application-in-azure-hdinsight"></a>Tutorial: Compilación de aplicaciones de aprendizaje automático de Apache Spark en Azure HDInsight

En este tutorial, aprenderá a usar [Jupyter Notebook](https://jupyter.org/) para compilar una aplicación de aprendizaje de automático de [Azure Spark](./apache-spark-overview.md) para Apache HDInsight.

[MLlib](https://spark.apache.org/docs/latest/ml-guide.html) es una biblioteca de aprendizaje automático adaptable de Spark que consta de utilidades y algoritmos de aprendizaje comunes (clasificación, regresión, agrupación en clústeres, filtrado colaborativo y reducción de dimensionalidad, además de primitivas de optimización subyacentes).

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Desarrollar una aplicación de aprendizaje automático de Apache Spark

## <a name="prerequisites"></a>Prerrequisitos

* Un clúster de Apache Spark en HDInsight. Vea [Creación de un clúster de Apache Spark](./apache-spark-jupyter-spark-sql-use-portal.md).

* Experiencia en el uso de Jupyter Notebooks con Spark en HDInsight. Para más información, consulte [Carga de datos y ejecución de consultas en un clúster de Apache Spark en Azure HDInsight](./apache-spark-load-data-run-query.md).

## <a name="understand-the-data-set"></a>Información acerca del conjunto de datos

La aplicación usa los datos de ejemplo de **HVAC.csv**, que están disponibles en todos los clústeres de manera predeterminada. El archivo se encuentra en `\HdiSamples\HdiSamples\SensorSampleData\hvac`. Los datos muestran la temperatura objetivo y la temperatura real de algunos edificios que tienen sistemas de calefacción, ventilación y aire acondicionado instalados. La columna **System** representa el identificador del sistema y la columna **SystemAge**, el número de años que lleva el sistema HVAC instalado en el edificio. Se puede predecir si un edificio será más cálido o frío en función de la temperatura objetivo, dados un identificador del sistema y la antigüedad del sistema.

![Instantánea de los datos usados para el ejemplo de aprendizaje automático de Spark](./media/apache-spark-ipython-notebook-machine-learning/spark-machine-learning-understand-data.png "Instantánea de los datos usados para el ejemplo de aprendizaje automático de Spark")

## <a name="develop-a-spark-machine-learning-application-using-spark-mllib"></a>Desarrollo de una aplicación de aprendizaje automático de Spark mediante Spark MLlib

En esta aplicación se usa una [canalización de Machine Learning](https://spark.apache.org/docs/2.2.0/ml-pipeline.html) de Spark para realizar una clasificación de documentos. Las canalizaciones de Machine Learning proporcionan un conjunto uniforme de API de alto nivel basadas en DataFrames. Mediante DataFrames, los usuarios pueden crear y ajustar prácticas canalizaciones de aprendizaje automático. En la canalización, se divide el documento en palabras, se convierten las palabras en un vector numérico de característica y finalmente se genera un modelo de predicción que use los vectores de característica y las etiquetas. Realice los siguientes pasos para crear la aplicación:

1. Cree un cuaderno de Jupyter Notebook con el kernel de PySpark. Para las instrucciones, consulte [Creación de un cuaderno de Jupyter Notebook](./apache-spark-jupyter-spark-sql.md#create-a-jupyter-notebook).

1. Importe los tipos necesarios para este escenario. Pegue el siguiente fragmento de código en una celda vacía y presione **MAYÚS + ENTRAR**.

    ```PySpark
    from pyspark.ml import Pipeline
    from pyspark.ml.classification import LogisticRegression
    from pyspark.ml.feature import HashingTF, Tokenizer
    from pyspark.sql import Row

    import os
    import sys
    from pyspark.sql.types import *

    from pyspark.mllib.classification import LogisticRegressionWithSGD
    from pyspark.mllib.regression import LabeledPoint
    from numpy import array
    ```

1. Cargue los datos (hvac.csv), analícelos y úselos para entrenar el modelo.

    ```PySpark
    # Define a type called LabelDocument
    LabeledDocument = Row("BuildingID", "SystemInfo", "label")

    # Define a function that parses the raw CSV file and returns an object of type LabeledDocument
    def parseDocument(line):
        values = [str(x) for x in line.split(',')]
        if (values[3] > values[2]):
            hot = 1.0
        else:
            hot = 0.0

        textValue = str(values[4]) + " " + str(values[5])

        return LabeledDocument((values[6]), textValue, hot)

    # Load the raw HVAC.csv file, parse it using the function
    data = sc.textFile("/HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")

    documents = data.filter(lambda s: "Date" not in s).map(parseDocument)
    training = documents.toDF()
    ```

    En el fragmento de código, se define una función que compara la temperatura real con la temperatura de destino. Si la temperatura real es mayor, el edificio está cálido, lo que viene indicado por el valor **1.0**. De lo contrario, el edificio está frío, lo que se indica con el valor **0.0**.

1. Configure la canalización de aprendizaje automático de Spark, que consta de tres fases: tokenizer, hashingTF e lr.

    ```PySpark
    tokenizer = Tokenizer(inputCol="SystemInfo", outputCol="words")
    hashingTF = HashingTF(inputCol=tokenizer.getOutputCol(), outputCol="features")
    lr = LogisticRegression(maxIter=10, regParam=0.01)
    pipeline = Pipeline(stages=[tokenizer, hashingTF, lr])
    ```

    Para más información acerca de las canalizaciones y cómo funcionan, consulte [Canalización de aprendizaje automático de Apache Spark](https://spark.apache.org/docs/latest/ml-pipeline.html).

1. Ajuste la canalización al documento de formación.

    ```PySpark
    model = pipeline.fit(training)
    ```

1. Compruebe el documento de aprendizaje para controlar el progreso con la aplicación.

    ```PySpark
    training.show()
    ```

    La salida es parecida a esta:

    ```output
    +----------+----------+-----+
    |BuildingID|SystemInfo|label|
    +----------+----------+-----+
    |         4|     13 20|  0.0|
    |        17|      3 20|  0.0|
    |        18|     17 20|  1.0|
    |        15|      2 23|  0.0|
    |         3|      16 9|  1.0|
    |         4|     13 28|  0.0|
    |         2|     12 24|  0.0|
    |        16|     20 26|  1.0|
    |         9|      16 9|  1.0|
    |        12|       6 5|  0.0|
    |        15|     10 17|  1.0|
    |         7|      2 11|  0.0|
    |        15|      14 2|  1.0|
    |         6|       3 2|  0.0|
    |        20|     19 22|  0.0|
    |         8|     19 11|  0.0|
    |         6|      15 7|  0.0|
    |        13|      12 5|  0.0|
    |         4|      8 22|  0.0|
    |         7|      17 5|  0.0|
    +----------+----------+-----+
    ```

    La comparación de la salida con el archivo CSV sin procesar. Por ejemplo, la primera fila del archivo CSV tiene estos datos:

    ![Instantánea de los datos de salida para el ejemplo de aprendizaje automático de Spark](./media/apache-spark-ipython-notebook-machine-learning/spark-machine-learning-output-data.png "Instantánea de los datos de salida para el ejemplo de aprendizaje automático de Spark")

    Observe que la temperatura real es menor que la temperatura objetivo, lo que indica que el edificio está frío. El valor de **label** en la primera fila es **0.0**, lo que significa que la temperatura del edificio no es cálida.

1. Prepare un conjunto de datos con el que ejecutar el modelo entrenado. Para ello, pase un identificador del sistema y la antigüedad del sistema (indicada como **SystemInfo** en la salida de entrenamiento). El modelo predice si el edificio con ese identificador del sistema y esa antigüedad del sistema será más cálido (indicado por 1.0) o más fresco (indicado por 0.0).

    ```PySpark
    # SystemInfo here is a combination of system ID followed by system age
    Document = Row("id", "SystemInfo")
    test = sc.parallelize([(1L, "20 25"),
                    (2L, "4 15"),
                    (3L, "16 9"),
                    (4L, "9 22"),
                    (5L, "17 10"),
                    (6L, "7 22")]) \
        .map(lambda x: Document(*x)).toDF()
    ```

1. Por último, realice predicciones basadas en los datos de prueba.

    ```PySpark
    # Make predictions on test documents and print columns of interest
    prediction = model.transform(test)
    selected = prediction.select("SystemInfo", "prediction", "probability")
    for row in selected.collect():
        print row
    ```

    La salida es parecida a esta:

    ```output  
    Row(SystemInfo=u'20 25', prediction=1.0, probability=DenseVector([0.4999, 0.5001]))
    Row(SystemInfo=u'4 15', prediction=0.0, probability=DenseVector([0.5016, 0.4984]))
    Row(SystemInfo=u'16 9', prediction=1.0, probability=DenseVector([0.4785, 0.5215]))
    Row(SystemInfo=u'9 22', prediction=1.0, probability=DenseVector([0.4549, 0.5451]))
    Row(SystemInfo=u'17 10', prediction=1.0, probability=DenseVector([0.4925, 0.5075]))
    Row(SystemInfo=u'7 22', prediction=0.0, probability=DenseVector([0.5015, 0.4985]))
    ```

   Observe la primera fila de la predicción. En un sistema HVAC con el identificador 20 y una antigüedad de 25 años, el edificio tiene una temperatura cálida (**prediction=1.0**). El primer valor de DenseVector (0.49999) corresponde a la predicción 0.0 y el segundo, (0.5001), corresponde a la predicción 1.0. En la salida, aunque el segundo valor solo es levemente superior, el modelo muestra **prediction=1.0**.

1. Cierre el cuaderno para liberar los recursos. Para ello, en el menú **File** (Archivo) del cuaderno y seleccione **Close and Halt** (Cerrar y detener). Con esta acción se cerrará el cuaderno.

## <a name="use-anaconda-scikit-learn-library-for-spark-machine-learning"></a>Uso de la biblioteca scikit-learn de Anaconda para el aprendizaje automático de Spark

Los clústeres Apache Spark de HDInsight incluyen bibliotecas de Anaconda. También incluyen la biblioteca **scikit-learn** para el aprendizaje automático. La biblioteca también contiene diversos conjuntos de datos que puede usar para crear aplicaciones de ejemplo directamente a partir de un cuaderno de Jupyter. Para obtener ejemplos sobre el uso de la biblioteca scikit-learn, consulte [https://scikit-learn.org/stable/auto_examples/index.html](https://scikit-learn.org/stable/auto_examples/index.html).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, elimine el clúster que creó mediante los siguientes pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. En el cuadro **Búsqueda** en la parte superior, escriba **HDInsight**.

1. Seleccione **Clústeres de HDInsight** en **Servicios**.

1. En la lista de clústeres de HDInsight que aparece, seleccione el signo **...** situado junto al clúster que ha creado para este tutorial.

1. Seleccione **Eliminar**. Seleccione **Sí**.

![Azure Portal elimina un clúster de HDInsight](./media/apache-spark-ipython-notebook-machine-learning/hdinsight-azure-portal-delete-cluster.png "Eliminación de un clúster de HDInsight")

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a usar Jupyter Notebook para compilar una aplicación de aprendizaje de automático de Azure Spark para Apache HDInsight. Para aprender a usar IntelliJ IDEA para trabajos de Spark, pase al siguiente tutorial.

> [!div class="nextstepaction"]
> [Creación de una aplicación de Scala Maven mediante IntelliJ](./apache-spark-create-standalone-application.md)
