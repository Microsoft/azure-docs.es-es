---
title: Automatización e ingesta de datos
titleSuffix: Azure Machine Learning
description: Obtenga información sobre las opciones de ingesta de datos para entrenar los modelos de aprendizaje automático.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: nibaccam
author: nibaccam
ms.author: nibaccam
ms.date: 02/26/2020
ms.custom: devx-track-python
ms.openlocfilehash: 18bbecbe811a9f0bc6a56194830c7e92d8770979
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90890174"
---
# <a name="data-ingestion-options-for-azure-machine-learning-workflows"></a>Opciones de ingesta de datos para flujos de trabajo de Azure Machine Learning

En este artículo, obtendrá información sobre las ventajas y desventajas de las opciones de ingesta de datos disponibles con Azure Machine Learning. 

Elija entre las siguientes opciones:
+ [Azure Data Factory](#azure-data-factory) canalizaciones, creadas específicamente para extraer, cargar y transformar datos.

+ [SDK de Python de Azure Machine Learning](#azure-machine-learning-python-sdk), proporciona una solución de código personalizada para las tareas de ingesta de datos.

+ Una combinación de ambas opciones.

La ingesta de datos es el proceso en el que los datos no estructurados se extraen de uno o varios orígenes y, a continuación, se preparan para entrenar modelos de Machine Learning. También requiere mucho tiempo, especialmente si se realiza manualmente, y si tiene grandes cantidades de datos de varios orígenes. La automatización de este esfuerzo libera recursos y garantiza que los modelos utilicen los datos más recientes y aplicables.

## <a name="azure-data-factory"></a>Azure Data Factory

[Azure Data Factory](https://docs.microsoft.com/azure/data-factory/introduction) ofrece compatibilidad nativa para la supervisión de orígenes de datos y desencadenadores para las canalizaciones de ingesta de datos.  

En la tabla siguiente se resumen las ventajas y desventajas del uso de Azure Data Factory para los flujos de trabajo de ingesta de datos.

|Ventajas|Desventajas
---|---
Diseñado específicamente para extraer, cargar y transformar datos.|Actualmente ofrece un conjunto limitado de tareas de canalización de Azure Data Factory. 
Le permite crear flujos de trabajo controlados por datos para orquestar el movimiento y la transformación de los datos a gran escala.|Su construcción y mantenimiento son costosos. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/data-factory/data-pipeline/) de Azure Data Factory para obtener más información.
Se integra con varias herramientas de Azure, como [Azure Databricks](https://docs.microsoft.com/azure/data-factory/transform-data-using-databricks-notebook) y [Azure Functions](https://docs.microsoft.com/azure/data-factory/control-flow-azure-function-activity). | No ejecuta scripts de forma nativa; en su lugar, se basa en procesos independientes para la ejecución de scripts. 
Admite de forma nativa la ingesta de datos desencadena por orígenes de datos.| 
Los procesos de preparación de datos y entrenamiento del modelo son independientes.|
Capacidad de linaje de datos integrada para flujo de datos de Azure Data Factory.|
Proporciona una [interfaz de usuario](https://docs.microsoft.com/azure/data-factory/quickstart-create-data-factory-portal) de experiencia de código bajo para enfoques que no son de scripting. |

Estos pasos y el siguiente diagrama sirven para ilustrar el flujo de trabajo de la ingesta de datos de Azure Data Factory.

1. Extraer los datos de sus orígenes.
1. Transformar y guardar los datos en un contenedor de blobs de salida, que sirve como almacenamiento de datos para Azure Machine Learning.
1. Con los datos almacenados preparados, la canalización de Azure Data Factory invoca una canalización de Machine Learning de entrenamiento que recibe los datos preparados para el entrenamiento del modelo.


    ![Ingesta de datos de ADF](media/concept-data-ingestion/data-ingest-option-one.svg)
    
Aprenda a crear una canalización de ingesta de datos para Machine Learning con [Azure Data Factory](how-to-data-ingest-adf.md).

## <a name="azure-machine-learning-python-sdk"></a>SDK de Python de Azure Machine Learning 

Con el [SDK de Python](https://docs.microsoft.com/python/api/overview/azure/ml), puede incorporar las tareas de ingesta de datos en un paso de [canalización de Azure Machine Learning](how-to-create-your-first-pipeline.md).

En la siguiente tabla se resumen las ventajas y desventajas del uso del SDK y un paso de canalizaciones de ML para las tareas de ingesta de datos.

Ventajas| Desventajas
---|---
Configuración de sus propios scripts de Python. | No admite de forma nativa el desencadenamiento de cambios en el origen de datos. Requiere implementaciones de Logic App o Azure Function.
Preparación de datos como parte de cada ejecución de entrenamiento del modelo.|Requiere aptitudes de desarrollo para crear un script de ingesta de datos.
Admite scripts de preparación de datos en varios destinos de procesos, incluido el [proceso de Azure Machine Learning](concept-compute-target.md#azure-machine-learning-compute-managed). |No proporciona una interfaz de usuario para crear el mecanismo de ingesta.

En el siguiente diagrama, la canalización de Azure Machine Learning consta de dos pasos: la ingesta de datos y el entrenamiento del modelo. El paso de ingesta de datos abarca tareas que se pueden realizar con bibliotecas de Python y el SDK de Python, como la extracción de datos de orígenes web o locales, y transformaciones de datos, como la imputación de valores que faltan. En el paso de entrenamiento se usan datos preparados como entrada del script de entrenamiento para entrenar el modelo de Machine Learning. 

![Canalización de Azure + ingesta de datos del SDK](media/concept-data-ingestion/data-ingest-option-two.png)

## <a name="next-steps"></a>Pasos siguientes

Siga estos artículos de procedimientos:
* [Creación de una canalización de ingesta de datos con Azure Data Factory](how-to-data-ingest-adf.md)

* [Automatización y administración de canalizaciones de ingesta de datos con Azure Pipelines](how-to-cicd-data-ingestion.md).
