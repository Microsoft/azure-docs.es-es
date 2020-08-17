---
title: Ingesta de datos con Azure Data Factory
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo crear una canalización de ingesta de datos con Azure Data Factory.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: iefedore
author: eedorenko
manager: davete
ms.reviewer: larryfr
ms.date: 03/01/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: b756e83f1d810007e9e9ef6cf2987c3cf60b7f7d
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87852861"
---
# <a name="data-ingestion-with-azure-data-factory"></a>Ingesta de datos con Azure Data Factory

En este artículo, aprenderá a crear una canalización de ingesta de datos con Azure Data Factory (ADF). Esta canalización se utiliza para ingerir datos para su uso con Azure Machine Learning. Azure Data Factory permite extraer, transformar y cargar datos fácilmente. Una vez que los datos se han transformado y cargado en el almacenamiento, se pueden usar para entrenar los modelos de aprendizaje automático.

La transformación de datos simple se puede controlar con actividades y instrumentos de ADF nativos, como el [flujo de datos](https://docs.microsoft.com/azure/data-factory/control-flow-execute-data-flow-activity). Cuando se trata de escenarios más complicados, los datos se pueden procesar con código personalizado. Por ejemplo, código de Python o R.

Hay varias técnicas comunes de uso de Azure Data Factory para transformar datos durante la ingesta. Cada técnica tiene ventajas e inconvenientes que determinan si es una buena opción para un caso de uso específico:

| Técnica | Ventajas | Desventajas |
| ----- | ----- | ----- |
| ADF + Azure Functions | Baja latencia, proceso sin servidor.</br>Funciones con estado.</br>Funciones reutilizables. | Solo adecuado para el procesamiento de ejecución breve. |
| ADF + componente personalizado | Procesamiento en paralelo a gran escala.</br>Adecuado para algoritmos pesados. | Encapsulado de código en un archivo ejecutable.</br>Complejidad de la administración de dependencias y E/S. |
| ADF + cuaderno de Azure Databricks | Spark de Apache</br>Entorno de Python nativo. | Puede ser caro.</br>La creación inicial de clústeres lleva tiempo y agrega latencia.

## <a name="adf-with-azure-functions"></a>Uso con Azure Functions.

![adf-function](media/how-to-data-ingest-adf/adf-function.png)

Azure Functions permite ejecutar pequeños fragmentos de código (denominados "funciones") sin preocuparse por la infraestructura de la aplicación. En esta opción, los datos se procesan con código Python personalizado encapsulado en una función de Azure Functions. 

La función se invoca con la [actividad de las funciones de Azure en ADF](https://docs.microsoft.com/azure/data-factory/control-flow-azure-function-activity). Este enfoque es una buena opción para las transformaciones de datos ligeros. 

* Ventajas:
    * Los datos se procesan en un proceso sin servidor con una latencia relativamente baja.
    * La canalización de ADF puede invocar una [función Durable Functions de Azure](/azure/azure-functions/durable/durable-functions-overview) que puede implementar un flujo de transformación de datos sofisticado. 
    * Los detalles de la transformación de datos se abstraen mediante la función de Azure que se puede reutilizar e invocar desde otros lugares.
* Inconvenientes:
    * Las funciones de Azure Functions deben crearse antes de usarse con ADF.
    * Azure Functions es adecuado solo para el procesamiento de datos de ejecución breve.

## <a name="adf-with-custom-component-activity"></a>ADF con actividad de componentes personalizados

![adf-customcomponent](media/how-to-data-ingest-adf/adf-customcomponent.png)

En esta opción, los datos se procesan con código Python personalizado encapsulado en un archivo ejecutable. Se invoca con una [actividad de componente personalizado de ADF](https://docs.microsoft.com/azure/data-factory/transform-data-using-dotnet-custom-activity). Este enfoque es más adecuado para los datos de gran tamaño que la técnica anterior.

* Ventajas:
    * Los datos se procesan en un grupo de [Azure Batch](https://docs.microsoft.com/azure/batch/batch-technical-overview), lo que proporciona un procesamiento de alto rendimiento y paralelo a gran escala.
    * Se puede usar para ejecutar algoritmos pesados y procesar grandes cantidades de datos.
* Inconvenientes:
    * Se debe crear el grupo de Azure Batch antes de usarse con ADF.
    * Sobre ingeniería relacionada con el encapsulado de código de Python en un archivo ejecutable. Complejidad de la administración de dependencias y parámetros de entrada y salida.

## <a name="adf-with-azure-databricks-python-notebook"></a>ADF con Azure Databricks cuaderno de Python

![adf-databricks](media/how-to-data-ingest-adf/adf-databricks.png)

[Azure Databricks](https://azure.microsoft.com/services/databricks/) es una plataforma de análisis basada en Apache Spark y optimizada para Azure.

En esta técnica, un [cuaderno de Python](https://docs.microsoft.com/azure/data-factory/transform-data-using-databricks-notebook), que se ejecuta en un clúster de Azure Databricks, realiza la transformación de datos. Probablemente, el enfoque más común que aprovecha toda la eficacia de un servicio de Azure Databricks. Está diseñado para el procesamiento de datos distribuidos a escala.

* Ventajas:
    * Los datos se transforman en el servicio de Azure de procesamiento de datos más eficaz, del que se realiza una copia de seguridad en un entorno de Apache Spark.
    * Compatibilidad nativa con Python junto con los marcos de ciencia de datos y las bibliotecas, como TensorFlow, PyTorch y scikit-learn.
    * No es necesario incluir el código de Python en funciones o módulos ejecutables. El código funciona tal cual.
* Inconvenientes:
    * Se debe crear la infraestructura de Azure Databricks antes de usarse con ADF.
    * Puede ser costoso en función de la configuración de Azure Databricks.
    * La rotación de los clústeres de proceso desde el modo "frío" tarda un tiempo que agrega una latencia alta a la solución. 
    

## <a name="consuming-data-in-azure-machine-learning-pipelines"></a>Consumo de datos en canalizaciones de Azure Machine Learning

![aml-dataset](media/how-to-data-ingest-adf/aml-dataset.png)

Los datos transformados de la canalización de ADF se guardan en el almacenamiento de datos (como Azure Blob). Azure Machine Learning puede tener acceso a estos datos mediante [almacenes de datos](https://docs.microsoft.com/azure/machine-learning/how-to-access-data#create-and-register-datastores) y [conjuntos de datos](https://docs.microsoft.com/azure/machine-learning/how-to-create-register-datasets).

Cada vez que se ejecuta la canalización de ADF, los datos se guardan en una ubicación diferente en el almacenamiento. Para pasar la ubicación a Azure Machine Learning, la canalización de ADF llama a una canalización Azure Machine Learning. Al llamar a la canalización de ML, la ubicación de los datos y el identificador de ejecución se envían como parámetros. La canalización de ML puede crear un almacén de datos o un conjunto de datos mediante la ubicación de los datos. 

> [!TIP]
> Los conjuntos de datos [admiten el control de versiones](https://docs.microsoft.com/azure/machine-learning/how-to-version-track-datasets), por lo que la canalización de ML puede registrar una nueva versión del conjunto de datos que apunte a los datos más recientes de la canalización de ADF.

Una vez que se pueda acceder a los datos a través de un almacén de datos o un conjunto de datos, puede usarlos para entrenar un modelo de ML. El proceso de entrenamiento podría formar parte de la misma canalización de ML a la que se llama desde ADF. También puede ser un proceso independiente como experimentación en un cuaderno de Jupyter Notebook.

Dado que los conjuntos de datos admiten el control de versiones y cada ejecución de la canalización crea una nueva versión, es fácil entender qué versión de los datos se usó para entrenar un modelo.

## <a name="next-steps"></a>Pasos siguientes

* [Ejecución de un cuaderno de Databricks en Azure Data Factory](https://docs.microsoft.com/azure/data-factory/transform-data-using-databricks-notebook)
* [Acceso a los datos en los servicios de almacenamiento de Azure](https://docs.microsoft.com/azure/machine-learning/how-to-access-data#create-and-register-datastores)
* [Entrenamiento de modelos con conjuntos de datos en Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/how-to-train-with-datasets)
* [DevOps para una canalización de ingesta de datos](https://docs.microsoft.com/azure/machine-learning/how-to-cicd-data-ingestion)

