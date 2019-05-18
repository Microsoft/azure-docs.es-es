---
title: Selección y optimización automáticas del algoritmo de Machine Learning
titleSuffix: Azure Machine Learning service
description: Obtenga información sobre cómo Azure Machine Learning Service puede elegir de forma automática un algoritmo y generar un modelo a partir de él para ahorrar tiempo, mediante los parámetros y criterios que proporcione con el fin de seleccionar el mejor algoritmo para el modelo.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: jmartens
author: nacharya1
ms.author: nilesha
ms.date: 05/02/2019
ms.custom: seodec18
ms.openlocfilehash: 9736cc3ab20d43cc3731bc237bed9eb9b5370cb4
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65800779"
---
# <a name="what-is-automated-machine-learning"></a>Descripción del aprendizaje automático

Automatizadas machine learning, también se denomina AutoML, permite a los desarrolladores, analistas y científicos de datos crear modelos de aprendizaje automático con alta escala, la eficiencia y productividad al mismo tiempo que mantiene la calidad del modelo. 

ML automatizado compila un conjunto de modelos de aprendizaje automático automáticamente, de forma inteligente selecciona modelos para el entrenamiento y, a continuación, recomienda lo mejor para usted. Desarrollo de modelos de aprendizaje automático tradicional consume muchos recursos que requieren un conocimiento ni dominio significativo y tiempo para generar y comparar docenas de modelos. Con ML automatizada, podrá acelerar el tiempo necesario para ponerse en modelos de aprendizaje automático listos para producción con eficacia y facilidad excelente.

En segundo plano, los datos de entrenamiento es realizados con una característica de destino definido e inteligentemente recorre en iteración en las combinaciones de algoritmos de aprendizaje automático y selecciones de características. A continuación, según las puntuaciones de entrenamiento, el mejor modelo ajustado es identificado y se recomienda. 

Todavía tiene control sobre el experimento y transparencia en lo que sucede. Puede definir restricciones y experimentar objetivos en función de tiempo, precisión o número de iteraciones, por ejemplo. Puede ver cada modelo que se generó para el experimento, el flujo de entrenamiento para cada iteración y las características más influyentes para un modelo determinado.

## <a name="how-automated-ml-works"></a>Funciona de forma automatizada ML

Uso de **servicio Azure Machine Learning**, puede diseñar y ejecutar sus experimentos de aprendizaje automatizados ML con estos pasos:

1. **Identificar el problema de Machine Learning** resolverse: clasificación, previsión o regresión
   
1. **Especificar el origen y el formato de los datos de entrenamiento etiquetado**: Matrices de Numpy o Pandas dataframe

1. **Configurar el destino de proceso de entrenamiento del modelo**, como su [equipo local, Azure Machine Learning calcula, las máquinas virtuales remotas o Azure Databricks](how-to-set-up-training-targets.md).  Obtenga información sobre aprendizaje automatizado [en un recurso remoto](how-to-auto-train-remote.md).

1. **Configurar los parámetros de aprendizaje de automático automatizado** que determinan cuántas iteraciones a través de diferentes modelos, configuración de hiperparámetros, avanzada y características de preprocesamiento y qué métricas se deben observar al determinar el mejor modelo.  Puede configurar los valores para el experimento de entrenamiento automático [en Azure portal](how-to-create-portal-experiments.md) o [con el SDK de](how-to-configure-auto-train.md).

1. **Enviar la ejecución de aprendizaje.** 


[![Aprendizaje automático automatizado](./media/how-to-automated-ml/automated-machine-learning.png)](./media/how-to-automated-ml/automated-machine-learning.png#lightbox)

Durante el aprendizaje, el servicio de Azure Machine Learning crea una serie de canalizaciones que prueban distintos parámetros y algoritmos. Se detendrá una vez que llega a los criterios de salida definidos en el experimento. 

También puede inspeccionar la información de ejecución registrada, que contiene las métricas que se recopilan durante la ejecución. La ejecución de aprendizaje genera un objeto serializado de Python (`.pkl` archivo) que contiene el modelo y preprocesamiento de datos.


Mientras se automatiza la creación del modelo, también puede [conocer cómo importante o relevantes características son](how-to-configure-auto-train.md#explain) para los modelos generados. 

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE2Xc9t]

<a name="preprocess"></a>

## <a name="preprocessing"></a>Preprocesamiento

En cada experimento de aprendizaje automático automatizada, los datos se preprocesan utilizando los métodos predeterminados y, opcionalmente, a través de opciones avanzadas de preprocesamiento.

### <a name="automatic-preprocessing-standard"></a>Operación automática de preprocesamiento (estándar)
En cada experimento de aprendizaje automático automatizada, los datos automáticamente se escala o normalizados para ayudar a los algoritmos funcionan bien.  Durante el entrenamiento del modelo, una de las siguientes técnicas de escalado o normalización se aplicará a cada modelo.

|Escalado&nbsp;&&nbsp;normalización| DESCRIPCIÓN |
| ------------- | ------------- |
| [StandardScaleWrapper](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.StandardScaler.html)  | Estandarizar características quitando la media y escalado a la varianza de la unidad  |
| [MinMaxScalar](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.MinMaxScaler.html)  | Transforma las características por cada característica por esa columna como mínimo y máximo de escalado  |
| [MaxAbsScaler](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.MaxAbsScaler.html#sklearn.preprocessing.MaxAbsScaler) |    Escalar cada característica, su valor absoluto máximo |  
| [RobustScalar](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.RobustScaler.html) |   Esta características Scaler por su rango intercuartil |
| [PCA](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.PCA.html) | Reducción de dimensionalidad lineal mediante descomposición en valores singulares de los datos al proyecto, a un espacio dimensional inferior | 
| [TruncatedSVDWrapper](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.TruncatedSVD.html) |    Esta clase transformer realiza la reducción de dimensionalidad lineal por medio de descomposición en valores singulares truncados (SVD). Al contrario que el PCA, este programa de estimación no centrar los datos antes de calcular la descomposición en valores singulares. Esto significa que pueda trabajar eficazmente con matrices scipy.sparse | 
| [SparseNormalizer](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.Normalizer.html) | Cada ejemplo (es decir, cada fila de la matriz de datos) con al menos un componente distinto de cero se vuelve a escalar con independencia de otras muestras para que su norma (l1 o l2) es igual a uno | 

### <a name="advanced-preprocessing-optional-featurization"></a>Advanced preprocesamiento: características opcionales

Características y preprocesamiento avanzadas adicionales también están disponibles, como la falta de imputación de valores, la codificación y las transformaciones. [Más información sobre qué características se incluye](how-to-create-portal-experiments.md#preprocess). Habilite a esta opción con:
+ Azure Portal: Seleccionar el **preprocesamiento** casilla de verificación en la **configuración avanzada** [con estos pasos](how-to-create-portal-experiments.md). 
+ SDK de Python: Especificar `"preprocess": True` para el [ `AutoMLConfig` clase](https://docs.microsoft.com/python/api/azureml-train-automl/azureml.train.automl.automlconfig?view=azure-ml-py).

## <a name="ensemble-models"></a>Los modelos

Puede entrenar los modelos con aprendizaje automático automatizada con la [Caruana algoritmo de selección de conjunto con la inicialización de conjunto ordenada](http://www.niculescu-mizil.org/papers/shotgun.icml04.revised.rev2.pdf). Aprendizaje conjunto mejora el rendimiento predictivo y los resultados de machine learning mediante la combinación de varios modelos en lugar de usar modelos de únicos. La iteración conjunto aparece como la última iteración de la ejecución.

## <a name="use-with-onnx-in-c-apps"></a>Uso con ONNX en C# aplicaciones

Con Azure Machine Learning, puede usar para generar un modelo de Python y convertirlo al formato ONNX ML automatizada. Es compatible con el tiempo de ejecución ONNX C#, por lo que puede usar el modelo creado automáticamente en su C# aplicaciones sin necesidad de volver a codificar o cualquiera de las latencias de red que presentan los extremos REST. Pruebe un ejemplo de este flujo [en este cuaderno de Jupyter](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/classification-with-onnx/auto-ml-classification-with-onnx.ipynb).

## <a name="automated-ml-across-microsoft"></a>ML automatizada a través de Microsoft

ML automatizada también está disponible en otras soluciones de Microsoft, como:
+ En las aplicaciones de .NET con Visual Studio y Visual Studio Code con [ML.NET](https://docs.microsoft.com/dotnet/machine-learning/automl-overview)
+ [En HDInsight](../../hdinsight/spark/apache-spark-run-machine-learning-automl.md), donde escala horizontalmente sus trabajos de aprendizaje de ML automatizados en Spark en clústeres de HDInsight en paralelo. 
+ [En Power BI](https://docs.microsoft.com/power-bi/service-machine-learning-automated)

## <a name="next-steps"></a>Pasos siguientes

Vea ejemplos y aprenda cómo generar modelos mediante Automated Machine Learning:

+ Siga el [Tutorial: Automatically train a classification model with Azure Automated Machine Learning](tutorial-auto-train-models.md) (Tutorial: Entrenamiento automático de un modelo de clasificación con Azure Machine Learning automatizado)

+ Configurar el experimento de aprendizaje automático: 
   + En la interfaz de portal de Azure, [siga estos pasos](how-to-create-portal-experiments.md).
   + Con el SDK de Python, [siga estos pasos](how-to-configure-auto-train.md).
  
 + Obtenga información sobre cómo auto entrenar con datos de series temporales, [siga estos pasos](how-to-auto-train-forecast.md).

+ Pruebe [ejemplos de Jupyter Notebook](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/)
