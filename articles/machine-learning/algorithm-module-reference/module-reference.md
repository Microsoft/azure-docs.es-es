---
title: Referencia de módulos y algoritmos
description: Conozca los módulos disponibles en el diseñador de Azure Machine Learning (versión preliminar).
titleSuffix: Azure Machine Learning
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 05/19/2020
ms.openlocfilehash: a588c58c74ab9632f3c3c61e223dd37704103714
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87048106"
---
# <a name="algorithm--module-reference-for-azure-machine-learning-designer-preview"></a>Referencia de módulo y algoritmo para el diseñador de Azure Machine Learning (versión preliminar)

Este contenido de referencia proporciona el contexto técnico de cada uno de los algoritmos y módulos de aprendizaje automático disponibles en el diseñador de Azure Machine Learning (versión preliminar).

Cada módulo representa un conjunto de código que puede ejecutarse de forma independiente y realizar una tarea de aprendizaje automático, dadas las entradas necesarias. Un módulo puede contener un algoritmo en particular o realizar una tarea que sea importante para el aprendizaje automático, como el reemplazo de un valor que falta o análisis estadísticos.

Para obtener ayuda con la elección de algoritmos, consulte 
* [Selección de algoritmos](../how-to-select-algorithms.md)
* [Hoja de características de los algoritmos de Machine Learning](../algorithm-cheat-sheet.md)

> [!TIP]
> En cualquier canalización del diseñador, puede obtener información sobre un módulo específico. Seleccione el vínculo **Más información** en la tarjeta del módulo al mantener el mouse sobre el módulo en la lista de módulos o en el panel derecho del módulo.

## <a name="data-preparation-modules"></a>Módulos de preparación de datos


| Funcionalidad | Descripción | módulo |
| --- |--- | --- |
| Entrada y salida de datos | Mueva los datos de los orígenes en la nube a la canalización. Escriba los resultados o los datos intermedios en Azure Storage, SQL Database o Hive, mientras ejecuta una canalización, o use el almacenamiento en la nube para intercambiar datos entre canalizaciones.  | [Introducción manual de datos](enter-data-manually.md) <br/> [Export Data](export-data.md) <br/> [Import Data](import-data.md) |
| Transformación de datos | Operaciones con los datos que son exclusivas del aprendizaje automático, como la normalización o discretización de datos, la reducción de la dimensionalidad y la conversión de datos entre barios formatos de archivo.| [Adición de columnas](add-columns.md) <br/> [Adición de filas](add-rows.md) <br/> [Aplicación de operación matemática](apply-math-operation.md) <br/> [Aplicación de transformaciones de SQL](apply-sql-transformation.md) <br/> [Clean Missing Data](clean-missing-data.md) (limpiar datos faltantes) <br/> [Recorte de valores](clip-values.md) <br/> [Conversión a CSV](convert-to-csv.md) <br/> [Conversión en conjunto de datos](convert-to-dataset.md) <br/> [Convertir en valores de indicador](convert-to-indicator-values.md) <br/> [Edición de metadatos](edit-metadata.md) <br/> [Agrupación de datos en intervalos](group-data-into-bins.md) <br/> [Combinación de datos](join-data.md) <br/> [Normalize Data](normalize-data.md) (normalizar datos) <br/> [Partición y ejemplo](partition-and-sample.md)  <br/> [Supresión de filas duplicadas](remove-duplicate-rows.md) <br/> [SMOTE](smote.md) <br/> [Selección de transformación de columnas](select-columns-transform.md) <br/> [Seleccionar columnas de conjunto de datos](select-columns-in-dataset.md) <br/> [División de datos](split-data.md) |
| Selección de características | Seleccione un subconjunto de características pertinentes y útiles para la creación de un modelo analítico. | [Selección de características basada en filtros](filter-based-feature-selection.md) <br/> [Importancia de la característica de permutación](permutation-feature-importance.md) |
| Funciones estadísticas | Proporcionar una amplia variedad de métodos estadísticos relacionados con la ciencia de datos. | [Resumen de datos](summarize-data.md)|

## <a name="machine-learning-algorithms"></a>Algoritmos de aprendizaje automático

| Funcionalidad | Descripción | módulo |
| --- |--- | --- |
| Regresión | Prediga un valor. | [Boosted Decision Tree Regression](boosted-decision-tree-regression.md) (Regresión del árbol de decisión ampliado) <br/> [Regresión de bosque de decisión](decision-forest-regression.md) <br/> [Regresión rápida de bosque por cuantiles](fast-forest-quantile-regression.md)  <br/> [Regresión lineal](linear-regression.md)  <br/> [Regresión de red neuronal](neural-network-regression.md)  <br/> [Regresión de Poisson](poisson-regression.md)  <br/>|
| Agrupación en clústeres | Agrupe datos.| [Agrupación en clústeres K-Means](k-means-clustering.md)
| clasificación | Prediga una clase.  Elija en algoritmos binarios (dos clases) o multiclase.| [Árbol de decisión ampliado multiclase](multiclass-boosted-decision-tree.md) <br/> [Bosque de decisión multiclase](multiclass-decision-forest.md) <br/> [Regresión logística multiclase](multiclass-logistic-regression.md)  <br/> [Red neuronal multiclase](multiclass-neural-network.md) <br/> [One vs. All Multiclass](one-vs-all-multiclass.md) (Multiclase uno frente a todos) <br/> [Perceptrón promedio de dos clases](two-class-averaged-perceptron.md) <br/>  [Two-Class Boosted Decision Tree](two-class-boosted-decision-tree.md) (Árbol de decisión promovido por dos clases)  <br/> [Bosque de decisión de dos clases](two-class-decision-forest.md) <br/>  [Regresión logística de dos clases](two-class-logistic-regression.md) <br/> [Red neuronal de dos clases](two-class-neural-network.md) <br/> [Two Class Support Vector Machine](two-class-support-vector-machine.md) (Máquina de vectores compatible con dos clases) | 

## <a name="modules-for-building-and-evaluating-models"></a>Módulos para compilar y evaluar modelos

| Funcionalidad | Descripción | módulo |
| --- |--- | --- |
| Entrenamiento del modelo | Ejecute datos a través del algoritmo. |  [Entrenamiento del modelo de agrupación en clústeres](train-clustering-model.md) <br/> [Train Model](train-model.md) (entrenar modelo) <br/> [Entrenamiento del modelo de PyTorch](train-pytorch-model.md) <br/> [Tune Model Hyperparameters](tune-model-hyperparameters.md) (Optimizar hiperparámetros del modelo) |
| Evaluación y puntuación del modelo | Mida la precisión del modelo entrenado. | [Aplicación de la transformación](apply-transformation.md) <br/> [Assign Data to Clusters](assign-data-to-clusters.md) (asignar datos a los clústeres) <br/> [Cross Validate Model](cross-validate-model.md) (Modelo de validación cruzada) <br/> [Evaluación de módulo](evaluate-model.md) <br/> [Puntuación del modelo de imagen](score-image-model.md) <br/> [Score Model](score-model.md) (puntuar modelo) |
| Lenguaje Python | Escriba código e insértelo en un módulo para integrar Python con la canalización. | [Creación de modelo Python](create-python-model.md) <br/> [Ejecución de script de Python](execute-python-script.md) |
| Lenguaje R | Escriba código e insértelo en un módulo para integrar R con la canalización. | [Ejecución script de R](execute-r-script.md) |
| Text Analytics | Proporcione herramientas de cálculo especializadas para trabajar con texto estructurado y no estructurado. |  [Conversión de palabra en vector](convert-word-to-vector.md) <br/> [Extracción de características de n-gramas a partir de texto](extract-n-gram-features-from-text.md) <br/> [Hash de características](feature-hashing.md) <br/> [Preprocesamiento de texto](preprocess-text.md) <br/> [Asignación de Dirichlet latente](latent-dirichlet-allocation.md) <br/> [Puntuación del modelo de Vowpal Wabbit](score-vowpal-wabbit-model.md) <br/> [Entrenamiento del modelo de Vowpal Wabbit](train-vowpal-wabbit-model.md)|
| Computer Vision | Módulos relacionados con el preprocesamiento de datos de imagen y el reconocimiento de imágenes. |  [Aplicación de transformación de imagen](apply-image-transformation.md) <br/> [Conversión al directorio de imagen](convert-to-image-directory.md) <br/> [Transformación de imagen init](init-image-transformation.md) <br/> [División de directorio de imagen](split-image-directory.md) <br/> [DenseNet](densenet.md) <br/> [ResNet](resnet.md) |
| Recomendación | Compile modelos de recomendación. | [Evaluate Recommender](evaluate-recommender.md) (Evaluar recomendador) <br/> [Score SVD Recommender](score-svd-recommender.md) (Puntuar recomendador de SVD) <br/> [Puntuación del recomendador ancho y profundo](score-wide-and-deep-recommender.md)<br/> [Train SVD Recommender](train-SVD-recommender.md) (Entrenar recomendador de SVD) <br/> [Entrenamiento del recomendador ancho y profundo](train-wide-and-deep-recommender.md)|
| Detección de anomalías | Compile modelos de detección de anomalías. | [Detección de anomalías basada en PCA](pca-based-anomaly-detection.md) <br/> [Train Anomaly Detection Model](train-anomaly-detection-model.md) (entrenar un modelo de detección de anomalías) |


## <a name="web-service"></a>Servicio web

Obtenga información sobre los [módulos del servicio web](web-service-input-output.md) que son necesarios para la inferencia en tiempo real en el diseñador de Azure Machine Learning.

## <a name="error-messages"></a>Mensajes de error

Conozca los [mensajes de error y códigos de excepción](designer-error-codes.md) que pueden surgir al usar los módulos del diseñador de Azure Machine Learning.

## <a name="next-steps"></a>Pasos siguientes

* [Tutorial: Creación de un modelo con el diseñador para predecir el precio de un automóvil](../tutorial-designer-automobile-price-train-score.md)
