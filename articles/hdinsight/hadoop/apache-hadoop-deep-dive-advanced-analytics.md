---
title: 'Profundización: análisis avanzado en Azure HDInsight'
description: Obtenga información sobre cómo el análisis avanzado usa algoritmos para procesar macrodatos en Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 01/01/2020
ms.openlocfilehash: ffc8fcf734877f55210507310849a5746a280c3d
ms.sourcegitcommit: 59ea8436d7f23bee75e04a84ee6ec24702fb2e61
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/07/2020
ms.locfileid: "89504356"
---
# <a name="deep-dive---advanced-analytics"></a>Profundización: análisis avanzado

## <a name="what-is-advanced-analytics-for-hdinsight"></a>¿Qué es el análisis avanzado de HDInsight?

HDInsight proporciona la capacidad de obtener información valiosa de grandes cantidades de datos estructurados, no estructurados y que cambian continuamente. El análisis avanzado consiste en el uso de arquitecturas muy escalables, modelos de aprendizaje automático y estadístico y paneles inteligentes para proporcionarle información significativa. El aprendizaje automático o el *análisis predictivo* usa algoritmos que identifican las relaciones de los datos, y aprenden a partir de estas, para realizar predicciones y tomar las decisiones.

## <a name="advanced-analytics-process"></a>Proceso de análisis avanzado

![Flujo de proceso de análisis avanzado](./media/apache-hadoop-deep-dive-advanced-analytics/hdinsight-analytic-process.png)

Una vez que haya identificado el problema empresarial y haya empezado a recopilar y procesar los datos, debe crear un modelo que represente la pregunta que quiera predecir. Su modelo usará uno o varios algoritmos de aprendizaje automático para realizar el tipo de predicción que mejor se adapte a sus necesidades empresariales.  Se debe usar la mayoría de los datos para entrenar el modelo, y la parte restante para probarlo o evaluarlo.

Después de crear, cargar, probar y evaluar el modelo, en el siguiente paso se debe implementar el modelo para que comience a proporcionar respuestas a sus preguntas. El último paso consiste en supervisar el rendimiento de su modelo y ajustarlo según sea necesario.

## <a name="common-types-of-algorithms"></a>Tipos comunes de algoritmos

Las soluciones de análisis avanzado proporcionan un conjunto de algoritmos de aprendizaje automático. Este es un resumen de las categorías de los algoritmos y los casos de uso empresariales comunes asociados.

![Resúmenes de las categorías de Machine Learning](./media/apache-hadoop-deep-dive-advanced-analytics/machine-learning-use-cases.png)

Además de seleccionar los algoritmos más adecuados, debe considerar si debe proporcionar o no datos para el aprendizaje. Los algoritmos de aprendizaje automático se clasifican como se indica a continuación:

* Supervisado: el algoritmo se debe entrenar con base en un conjunto de datos etiquetados para que pueda proporcionar resultados.
* El algoritmo supervisado puede aumentarse mediante destinos adicionales a través de la consulta interactiva por parte de un instructor, que no estaba disponible durante la fase inicial del entrenamiento
* No supervisado: el algoritmo no requiere datos de entrenamiento
* Refuerzo: el algoritmo usa agentes de software para determinar el comportamiento ideal dentro de un contexto específico (se usa a menudo en robótica).

| Categoría de algoritmo| Uso | Tipo de aprendizaje | Algoritmos |
| --- | --- | --- | -- |
| clasificación | Clasificar personas o cosas en grupos | Supervisado | Árboles de decisión, regresión logística y redes neurales |
| Agrupación en clústeres | Dividir un conjunto de ejemplos en grupos homogéneos | Sin supervisar | Agrupación en clústeres K-means |
| Detección de patrones | Identificar las asociaciones frecuentes en los datos | Sin supervisar | Reglas de asociación |
| Regresión | Predecir los resultados numéricos | Supervisado | Regresión lineal y redes neurales |
| Refuerzo | Determinar el comportamiento óptimo para los robots | Refuerzo | Simulaciones Monte Carlo y DeepMind |

## <a name="machine-learning-on-hdinsight"></a>Aprendizaje automático en HDInsight

HDInsight tiene varias opciones de aprendizaje automático para un flujo de trabajo de análisis avanzado:

* Machine Learning y Apache Spark
* R y ML Services
* Azure Machine Learning y Apache Hive
* Apache Spark y aprendizaje profundo

### <a name="machine-learning-and-apache-spark"></a>Machine Learning y Apache Spark

[HDInsight Spark](../spark/apache-spark-overview.md) es una oferta hospedada por Azure de [Apache Spark](https://spark.apache.org/). Consiste en un marco de procesamiento de datos paralelo de código abierto y unificado que usa el procesamiento en memoria para mejorar el análisis de macrodatos. El motor de procesamiento Spark se ha creado para ofrecer velocidad, facilidad de uso y análisis sofisticados. Las capacidades de cálculo distribuido en memoria de Spark lo convierten en una buena opción para algoritmos iterativos en los cálculos de gráficos y aprendizaje automático.

Hay tres bibliotecas de aprendizaje automático escalables que ofrecen las funcionalidades del modelado algorítmico a este entorno distribuido.

* [**MLlib**](https://spark.apache.org/docs/latest/ml-guide.html): MLlib contiene la API original que se basa en RDD de Spark.
* [**SparkML**](https://spark.apache.org/docs/1.2.2/ml-guide.html): SparkML es un paquete más reciente que proporciona una API de nivel más alto que se basa en DataFrames de Spark para construir canalizaciones de ML.
* [**MMLSpark**](https://github.com/Azure/mmlspark): la biblioteca de Microsoft Machine Learning para Apache Spark (MMLSpark) se ha diseñado para que los científicos de datos sean más productivos en Spark, aumenten la velocidad de experimentación y aprovechen las técnicas de aprendizaje automático de vanguardia, así como el aprendizaje profundo, en grandes conjuntos de datos. La biblioteca MMLSpark simplifica las tareas de modelado comunes para la compilación de modelos en PySpark.

### <a name="r-and-ml-services"></a>R y ML Services

Como parte de HDInsight, puede crear un clúster de HDInsight con [ML Services](../r-server/r-server-overview.md) listo para su uso con grandes conjuntos de datos y modelos. Esta nueva funcionalidad proporciona a los científicos de datos y a los estadísticos una interfaz conocida de R que se puede escalar a petición a través de HDInsight, sin la sobrecarga de mantenimiento y configuración del clúster.

### <a name="azure-machine-learning-and-apache-hive"></a>Azure Machine Learning y Apache Hive

[Azure Machine Learning Studio (clásico)](https://studio.azureml.net/) proporciona herramientas para el análisis predictivo y un servicio totalmente administrado que puede usar para implementar los modelos predictivos como servicios web listos para consumir. Azure Machine Learning proporciona herramientas para crear completas soluciones de análisis predictivos en la nube a fin de crear, probar, operacionalizar y administrar modelos predictivos rápidamente. Seleccione entre una biblioteca de algoritmos de gran tamaño, utilice un estudio basado en web para la compilación de modelos e implemente fácilmente el modelo como un servicio web.

### <a name="apache-spark-and-deep-learning"></a>Apache Spark y aprendizaje profundo

El [aprendizaje profundo](https://www.microsoft.com/research/group/dltc/) es una rama del aprendizaje automático que utiliza *redes neurales profundas* (DNN), inspiradas por los procesos biológicos del cerebro humano. Muchos investigadores ven el aprendizaje profundo como un enfoque prometedor para la inteligencia artificial. Algunos ejemplos del aprendizaje profundo son los intérpretes automáticos de idiomas, los sistemas de reconocimiento de imágenes y el razonamiento automático. Con el fin de realizar avances en su trabajo de aprendizaje profundo, Microsoft ha desarrollado el kit de herramientas [Microsoft Cognitive Toolkit](https://www.microsoft.com/en-us/cognitive-toolkit/) de código abierto, gratuito y fácil de usar. Actualmente, una amplia variedad de productos de Microsoft, compañías de todo el mundo con necesidad de implementar el aprendizaje profundo a escala y estudiantes interesados en los algoritmos y las técnicas más recientes usan de forma exhaustiva el kit de herramientas.

## <a name="scenario---score-images-to-identify-patterns-in-urban-development"></a>Escenario: puntuación de imágenes para identificar patrones de desarrollo urbano

Vamos a revisar un ejemplo de una canalización de aprendizaje automático de análisis avanzado con HDInsight.

En este escenario verá cómo DNN producido en un marco de aprendizaje profundo, Cognitive Toolkit de Microsoft (CNTK), se puede usar para puntuar grandes colecciones de imágenes almacenadas en una cuenta de Azure Blob Storage mediante PySpark en un clúster de HDInsight Spark. Este enfoque se aplica a un caso de uso común de DNN (la clasificación de imágenes aéreas) y puede usarse para identificar patrones recientes en desarrollo urbano.  Usará un modelo de clasificación de imágenes previamente entrenado. El modelo se entrenó previamente en el [conjunto de datos de CIFAR-10](https://www.cs.toronto.edu/~kriz/cifar.html) y se ha aplicado a 10 000 imágenes retenidas.

Hay tres tareas clave en este escenario de análisis avanzado:

1. Cree un clúster de Hadoop de Azure HDInsight con una distribución de Apache Spark 2.1.0.
2. Ejecute un script personalizado para instalar Microsoft Cognitive Toolkit en todos los nodos de un clúster de Azure HDInsight Spark.
3. Cargue un bloc de notas de Jupyter precompilado en el clúster de HDInsight Spark para aplicar un modelo de aprendizaje profundo de Microsoft Cognitive Toolkit entrenado a los archivos de una cuenta de Azure Blob Storage mediante la API de Spark Python (PySpark).

En este ejemplo se utiliza el conjunto de imágenes de CIFAR-10 que compilan y distribuyen Alex Krizhevsky, Vinod Nair y Geoffrey Hinton. El conjunto de datos de CIFAR-10 contiene 60 000 imágenes 32 x 32 que pertenecen a 10 clases mutuamente exclusivas de color:

![Imágenes de ejemplo de Machine Learning](./media/apache-hadoop-deep-dive-advanced-analytics/machine-learning-images.png)

Para más información sobre el conjunto de datos, vea [Learning Multiple Layers of Features from Tiny Images](https://www.cs.toronto.edu/~kriz/learning-features-2009-TR.pdf) (Entrenamiento de varias capas de características a partir de imágenes muy pequeñas) de Alex Krizhevsky.

El conjunto de datos se dividió en particiones para formar un conjunto de aprendizaje de 50 000 imágenes y un conjunto de pruebas de 10 000 imágenes. El primer conjunto se usó para entrenar un modelo de red residual (ResNet) convolucional de veinte capas de profundidad con Microsoft Cognitive Toolkit siguiendo los pasos de [este tutorial](https://github.com/Microsoft/CNTK/tree/master/Examples/Image/Classification/ResNet) del repositorio de GitHub de Cognitive Toolkit. Las 10 000 imágenes restantes se utilizaron para probar la exactitud del modelo. Aquí es donde entra en juego la computación distribuida: la tarea de procesamiento previo y la puntuación de las imágenes se puede paralelizar considerablemente. Con el modelo entrenado guardado actual, usamos:

* PySpark para distribuir las imágenes y el modelo entrenado a los nodos de trabajo del clúster.
* Python para procesar previamente las imágenes en cada nodo del clúster de HDInsight Spark.
* Cognitive Toolkit para cargar el modelo y puntuar las imágenes procesadas previamente en cada nodo.
* Jupyter Notebook para ejecutar el script de PySpark, agregar los resultados y usar [Matplotlib](https://matplotlib.org/) para visualizar el rendimiento del modelo.

El proceso completo del procesamiento previo o la puntuación de 10 000 imágenes tarda menos de un minuto en un clúster con 4 nodos de trabajo. El modelo predice con precisión las etiquetas de aproximadamente 9100 imágenes (91 %). Una matriz de confusión muestra los errores más comunes de clasificación. Por ejemplo, la matriz muestra que el etiquetado incorrecto de perros como gatos y viceversa se produce con mayor frecuencia que con otros pares de etiquetas.

![Gráfico de resultados de Machine Learning](./media/apache-hadoop-deep-dive-advanced-analytics/machine-learning-results.png)

### <a name="try-it-out"></a>¡Pruébelo!

Siga [este tutorial](../spark/apache-spark-microsoft-cognitive-toolkit.md) para implementar esta solución de un extremo a otro: configurar un clúster de HDInsight Spark, instalar Cognitive Toolkit y ejecutar el Jupyter Notebook que puntúa las imágenes de 10.000 CIFAR.

## <a name="next-steps"></a>Pasos siguientes

Apache Hive y Azure Machine Learning

* [Apache Hive y Azure Machine Learning de un extremo a otro](../../machine-learning/team-data-science-process/hive-walkthrough.md)
* [Uso de un clúster de Azure HDInsight Hadoop en un conjunto de información de 1 TB](../../machine-learning/team-data-science-process/hive-criteo-walkthrough.md)

Apache Spark y MLLib

* [Aprendizaje automático con Apache Spark en HDInsight](../../machine-learning/team-data-science-process/spark-overview.md)
* [Apache Spark con Machine Learning: uso de Apache Spark en HDInsight para analizar la temperatura de edificios con los datos del sistema de acondicionamiento de aire](../spark/apache-spark-ipython-notebook-machine-learning.md)
* [Apache Spark con Machine Learning: uso de Apache Spark en HDInsight para predecir los resultados de la inspección de alimentos](../spark/apache-spark-machine-learning-mllib-ipython.md)

Aprendizaje profundo, Cognitive Toolkit y otros

* [Máquina virtual de Azure de ciencia de datos](../../machine-learning/data-science-virtual-machine/overview.md)
* [Introducción a H2O.ai en Azure HDInsight](https://azure.microsoft.com/blog/introducing-h2o-ai-with-on-azure-hdinsight-to-bring-the-most-robust-ai-platform-for-enterprises/)
