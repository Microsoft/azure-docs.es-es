---
title: Capacidad de interpretación de modelos en Azure Machine Learning (versión preliminar)
titleSuffix: Azure Machine Learning
description: Aprenda a explicar los motivos por los que un modelo realiza predicciones mediante el SDK de Azure Machine Learning. Se puede usar durante el entrenamiento y la inferencia para saber cómo realiza las predicciones el modelo.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.author: mithigpe
author: minthigpen
ms.reviewer: Luis.Quintanilla
ms.date: 07/09/2020
ms.openlocfilehash: 757646a1630443f297d4b7c538721e9facdc130c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90897429"
---
# <a name="model-interpretability-in-azure-machine-learning-preview"></a>Capacidad de interpretación de modelos en Azure Machine Learning (versión preliminar)


## <a name="overview-of-model-interpretability"></a>Explicación de la interpretación del modelo

La interpretación es fundamental para los científicos de datos, los auditores y los responsables de la toma de decisiones empresariales para garantizar el cumplimiento de las directivas de la empresa, los estándares del sector y las regulaciones gubernamentales:

+ Los científicos de datos necesitan la capacidad de explicar sus modelos a ejecutivos y partes interesadas para que puedan reconocer el valor y la precisión de sus hallazgos. También deben tener la capacidad de interpretarse para depurar los modelos y tomar decisiones informadas sobre cómo mejorarlos. 

+ Los auditores legales requieren herramientas para validar los modelos en relación con el cumplimiento normativo y supervisar el impacto de las decisiones de los modelos en los usuarios. 

+ Los responsables de la toma de decisiones empresariales necesitan generar armonía al tener la capacidad de ofrecer transparencia a los usuarios finales. Esto les permite ganar y mantener la confianza.


La habilitación de la capacidad de explicar un modelo de aprendizaje automático es importante durante dos fases principales del desarrollo del modelo:
+ En la fase de entrenamiento, tanto los evaluadores como los diseñadores de modelos pueden usar el resultado de la interoperabilidad de un modelo para comprobar las hipótesis y crear relaciones de confianza con las partes interesadas. También usan la conclusiones sobre el modelo para depurarlo, para validar que el comportamiento del modelo coincide con sus objetivos y para comprobar si hay características insignificantes o sesgadas.

+ Durante la fase de inferencia, como la transparencia en torno a los modelos implementados, los ejecutivos pueden comprender "cuando se implementan" cómo funciona el modelo y cómo sus decisiones tratan y afectan a las personas en la vida real. 

## <a name="interpretability-with-azure-machine-learning"></a>Interpretación con Azure Machine Learning

Las clases de interpretabilidad están disponibles través de varios paquetes de SDK: (Obtenga información sobre cómo [instalar paquetes de SDK para Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py&preserve-view=true))

* `azureml.interpret`, el paquete principal, que contiene funcionalidades compatibles con Microsoft.

* `azureml.contrib.interpret`, versión preliminar, y las funcionalidades experimentales que puede probar.

Use `pip install azureml-interpret` y `pip install azureml-contrib-interpret` para el uso general.


> [!IMPORTANT]
> El contenido del espacio de nombres `contrib` no es totalmente compatible. A medida que maduran las funcionalidades experimentales, se mueven gradualmente al espacio de nombres principal.
.



## <a name="how-to-interpret-your-model"></a>Interpretación del modelo

Mediante las clases y métodos del SDK, puede hacer lo siguiente:
+ Explicar la predicción del modelo al generar valores de importancia de la característica para el modelo completo o para los puntos de datos individuales. 
+ Lograr la interpretabilidad del modelo en los conjuntos de datos del mundo real a escala, durante el entrenamiento y la inferencia.
+ Usar un panel de visualizaciones interactivas para detectar de patrones en los datos y explicaciones durante el entrenamiento.


En el aprendizaje automático, las **características** son los campos de datos que se usan para predecir un punto de datos de destino. Por ejemplo, para predecir el riesgo de crédito, se pueden usar los campos de datos de la edad, el tamaño de la cuenta y la antigüedad de la cuenta. En este caso, la edad, el tamaño de la cuenta y la antigüedad de la cuenta son **características**. La importancia de las características le indica la forma en que cada campo de datos ha afectado las predicciones del modelo. Por ejemplo, es posible que la edad se use mucho en la predicción, mientras que la antigüedad y el tamaño de la cuenta no afectan a los valores de la predicción significativamente. Este proceso permite a los científicos de datos explicar las predicciones resultantes, con el fin de que partes interesadas puedan ver qué características son las más importantes del modelo.

Obtenga información sobre las técnicas de interpretación admitidas, los modelos de Machine Learning admitidos y los entornos de ejecución admitidos en este artículo.


## <a name="supported-interpretability-techniques"></a>Técnicas de interpretación admitidas

 `azureml-interpret` usa las técnicas de interpretación desarrolladas en la [Comunidad de Interpret](https://github.com/interpretml/interpret-community/), un paquete Python de código abierto para entrenar modelos que se pueden interpretar y para ayudar a explicar los sistemas de IA de caja negra. [Interpretar-Comunidad ](https://github.com/interpretml/interpret-community/)actúa como el host de los Explicadores admitidos por este SDK y actualmente admite las siguientes técnicas de interpretación:

|Técnicas de interpretación|Descripción|Tipo|
|--|--|--------------------|
|SHAP Tree Explainer| El explicador de árboles de [SHAP](https://github.com/slundberg/shap), que se centra en el algoritmo de cálculo rápido de valores SHAP en tiempo polinómico específico para **árboles y conjuntos de árboles**.|Específico del modelo|
|SHAP Deep Explainer| Según la explicación de SHAP, Deep Explainer "es un algoritmo de aproximación de alta velocidad para valores SHAP en modelos de aprendizaje profundo que se basa en una conexión con DeepLIFT descrita en el documento [SHAP NIPS](https://papers.nips.cc/paper/7062-a-unified-approach-to-interpreting-model-predictions). Se admiten tanto los modelos de **TensorFlow** como los modelos de **Keras** que usan el back-end de TensorFlow (también hay compatibilidad preliminar con PyTorch)".|Específico del modelo|
|SHAP Linear Explainer| SHAP Linear Explainer calcula los valores SHAP para un **modelo lineal**, considerando de forma opcional las correlaciones entre funciones.|Específico del modelo|
|SHAP Kernel Explainer| SHAP Kernel Explainer usa una regresión lineal local ponderada de forma especial para calcular los valores SHAP de **cualquier modelo**.|Independiente del modelo|
|Mimic Explainer (suplente global)| El explicador Mimic se basa en la idea de entrenar [modelos globales de sustitución](https://christophm.github.io/interpretable-ml-book/global.html) para imitar los modelos de caja negra. Un modelo suplente global es un modelo interpretable de forma intrínseca que está entrenado para aproximarse a las predicciones de **cualquier modelo de caja negra** de la forma más precisa posible. Los científicos de datos pueden interpretar el modelo suplente para extraer conclusiones acerca del modelo de caja negra. Cualquiera de los siguientes modelos se puede usar como modelo suplente: LightGBM (LGBMExplicableModel), Regresión lineal (LinearExplicableModel), Modelo explicable de descenso de gradiente estocástico (SGDExplicableModel) y Árbol de decisión (DecisionTreeExplicableModel).|Independiente del modelo|
|Permutation Feature Importance Explainer (PFI)| La importancia de características de permutación es una técnica que se usa para explicar los modelos de clasificación y regresión que está inspirada en el [documento acerca de bosques aleatorios de Breiman](https://www.stat.berkeley.edu/~breiman/randomforest2001.pdf) (consulte la sección 10). En un alto nivel, la forma en que funciona es cuando se barajan aleatoriamente los datos de una característica para todo el conjunto de datos y se calcula en qué medida cambia la métrica de rendimiento que interesa. Cuanto mayor sea el cambio, más importante es la característica. PFI puede explicar el comportamiento general de **cualquier modelo subyacente**, pero no explica las predicciones individuales. |Independiente del modelo|




Además de las técnicas de interpretación descritas anteriormente, se admite otro explicador basado en SHAP, denominado `TabularExplainer`. En función del modelo, `TabularExplainer` usa uno de los explicadores de SHAP admitidos:

* TreeExplainer para todos los modelos basados en árboles
* DeepExplainer para los modelos de DNN
* LinearExplainer para los modelos lineales
* KernelExplainer para todos los demás modelos

`TabularExplainer` también realizó mejoras significativas en las características y en el rendimiento con respecto a los Explicadores directos de la SHAP:

* **Resumen del conjunto de datos de inicialización**. En los casos en que la velocidad de la explicación es lo más importante, resumimos el conjunto de datos de inicialización y generamos un pequeño conjunto de ejemplos representativos, lo que acelera la generación de valores generales y de importancia de la característica individual.
* **Muestreo del conjunto de datos de evaluación**. Si el usuario pasa un gran conjunto de ejemplos de evaluación, pero no necesita que se evalúen todos, el parámetro de muestreo puede establecerse en true para acelerar el cálculo de las explicaciones de modelo generales.

El siguiente diagrama muestra la estructura actual de los explicadores admitidos.

[![Arquitectura de interpretabilidad de Machine Learning](./media/how-to-machine-learning-interpretability/interpretability-architecture.png)](./media/how-to-machine-learning-interpretability/interpretability-architecture.png#lightbox)


## <a name="supported-machine-learning-models"></a>Modelos de Machine Learning admitidos

El paquete `azureml.interpret` del SDK admite los modelos entrenados con los siguientes formatos de conjunto de datos:
- `numpy.array`
- `pandas.DataFrame`
- `iml.datatypes.DenseData`
- `scipy.sparse.csr_matrix`

Las funciones de explicación aceptan como entrada tanto modelos como canalizaciones. Si se proporciona un modelo, este debe implementar la función de predicción `predict` o `predict_proba` que se ajuste a la convención de Scikit. Si el modelo no lo admite, puede encapsularlo en una función que genere el mismo resultado que `predict` o `predict_proba` en Scikit y usar esa función contenedora con el explicador seleccionado. Si se proporciona una canalización, la función de la explicación da por supuesto que el script de canalización de ejecución devuelve una predicción. Con esta técnica de encapsulado, `azureml.interpret` puede admitir modelos entrenados mediante marcos de aprendizaje profundo de PyTorch, TensorFlow y Keras, así como modelos de Machine Learning clásicos.

## <a name="local-and-remote-compute-target"></a>Destino de proceso local y remoto

El paquete `azureml.interpret` está diseñado para trabajar con destinos de proceso locales y remotos. Si se ejecutan localmente, las funciones el SDK no entrarán en contacto con ningún servicio de Azure. 

Puede ejecutar una explicación de forma remota en Proceso de Azure Machine Learning y registrar la información de la explicación en el servicio de historial de ejecución de Azure Machine Learning. Una vez que esta información se registra, tanto los informes como las visualizaciones de la explicación están disponibles en Azure Machine Learning Studio (clásico) para que los pueda analizar el usuario.


## <a name="next-steps"></a>Pasos siguientes

- Vea las [instrucciones](how-to-machine-learning-interpretability-aml.md) para habilitar la interpretación de entrenamiento de modelos tanto localmente como en los recursos de computación remota de Azure Machine Learning. 
- Consulte los [cuadernos de muestras](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/explain-model) para ver otros escenarios. 
- Si le interesa la interpretación de los escenarios de texto, consulte [Interpret-text](https://github.com/interpretml/interpret-text), un repositorio de código abierto relacionado con [Interpret-Community](https://github.com/interpretml/interpret-community/), para conocer las técnicas de interpretación de NLP. El paquete `azureml.interpret` no es compatible actualmente con estas técnicas, pero puede empezar a trabajar con un [cuaderno de ejemplo sobre la clasificación de texto](https://github.com/interpretml/interpret-text/blob/master/notebooks/text_classification/text_classification_classical_text_explainer.ipynb).
