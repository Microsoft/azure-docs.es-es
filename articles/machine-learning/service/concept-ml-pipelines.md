---
title: 'Canalizaciones: optimización de los flujos de trabajo de Machine Learning'
titleSuffix: Azure Machine Learning service
description: En este artículo, obtenga información sobre las canalizaciones de Machine Learning que puede compilar con el SDK de Azure Machine Learning para Python y las ventajas de utilizar canalizaciones. Los científicos de datos usan las canalizaciones de Machine Learning (ML) para crear, optimizar y administrar los flujos de trabajo de aprendizaje automático.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: sanpil
author: sanpil
ms.date: 05/14/2019
ms.custom: seodec18
ms.openlocfilehash: ff693ac34382ea5673989ecb6cbb38e19e176ad3
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65801074"
---
# <a name="build-machine-learning-pipelines-with-the-azure-machine-learning-service"></a>Creación de canalizaciones de aprendizaje automático con el servicio Azure Machine Learning

En este artículo, aprenderá sobre las canalizaciones de aprendizaje automático que puede crear con el SDK de Azure Machine Learning para Python y las ventajas de usar canalizaciones.

## <a name="what-are-machine-learning-pipelines"></a>¿Qué son las canalizaciones de Machine Learning?

Mediante las canalizaciones de Machine Learning (ML), los científicos de datos, los ingenieros de datos y los profesionales de TI pueden colaborar en los pasos relacionados con las acciones siguientes:
+ Preparación de datos, como normalizaciones y transformaciones
+ Entrenamiento del modelo
+ Evaluación del modelo
+ Implementación 

El diagrama siguiente muestra un ejemplo de canalización:

![Canalizaciones de aprendizaje automático en el servicio Azure Machine Learning](./media/concept-ml-pipelines/pipelines.png)

### <a name="which-azure-pipeline-technology-should-i-use"></a>¿La tecnología de la canalización de Azure se debe usar?

La nube de Azure proporciona varias otras canalizaciones, cada uno con un propósito diferente. En la tabla siguiente se enumera las canalizaciones diferentes y lo que se usan para:

| Canalización | Qué hace | Canalización canónico |
| ---- | ---- | ---- |
| Canalizaciones de Azure Machine Learning | Define los flujos de trabajo que pueden utilizarse como plantilla para los escenarios de aprendizaje automático de aprendizaje de automático reutilizable. | Datos -> modelo |
| [Canalizaciones de Azure Data Factory](https://docs.microsoft.com/azure/data-factory/concepts-pipelines-activities) | Las actividades de control, transformación y movimiento de datos de los grupos necesitan para realizar una tarea.  | Datos -> datos |
| [Canalizaciones de Azure](https://azure.microsoft.com/services/devops/pipelines/) | La integración continua y entrega de la aplicación para cualquier plataforma/any en la nube  | Código -> aplicación/servicio |

## <a name="why-build-pipelines-with-azure-machine-learning"></a>Motivos para compilar canalizaciones con Azure Machine Learning

El [SDK de Azure Machine Learning para Python](#the-python-sdk-for-pipelines) puede usarse para crear canalizaciones de Machine Learning, así como para enviar y realizar el seguimiento de ejecuciones de canalizaciones individuales.

Con las canalizaciones, puede optimizar su flujo de trabajo con sencillez, velocidad, portabilidad y reutilización. Al crear canalizaciones mediante Azure Machine Learning, puede centrarse en lo que domina, el aprendizaje automático, en lugar de en la infraestructura.

El uso de distintos pasos permite volver a ejecutar solo aquellos que necesita, mientras ajusta y prueba el flujo de trabajo. Un paso es una unidad de proceso en la canalización. Como se muestra en el diagrama anterior, en la tarea de preparación de los datos pueden intervenir muchos pasos. Estos pasos incluyen, pero no se limitan a, normalización, transformación, validación y características. Los orígenes de datos y los datos intermedios se reutilizan en la canalización, lo que ahorra tiempo y recursos de proceso. 

Una vez diseñada la canalización, a menudo se aplican más ajustes en torno al bucle de aprendizaje de la canalización. Cuando se vuelve a ejecutar una canalización, la ejecución salta a los pasos necesarios para volver a ejecutarse, como un script de aprendizaje actualizado, y omite lo que no ha cambiado. El mismo paradigma se aplica a los scripts sin cambios utilizados para la ejecución del paso. 

Con Azure Machine Learning, puede usar diversos Kits de herramientas y marcos, como TensorFlow, o de PyTorch para cada paso en la canalización. Azure se coordina entre los distintos [destinos de proceso](concept-azure-machine-learning-architecture.md) que usa para que los datos intermedios se puedan compartir con los destinos de proceso de bajada fácilmente. 

También puede [realizar un seguimiento de las métricas de los experimentos de canalización](https://docs.microsoft.com/azure/machine-learning/service/how-to-track-experiments) directamente en Azure Portal. 

## <a name="key-advantages"></a>Ventajas clave

Las principales ventajas de la compilación de canalizaciones para los flujos de trabajo de aprendizaje automático son:

|Ventaja clave|DESCRIPCIÓN|
|:-------:|-----------|
|**Ejecuciones&nbsp;desatendidas**|Programe unos pasos para ejecutar en paralelo o en secuencia de manera confiable y desatendida. Dado que la preparación y el modelado de los datos pueden tardar días o semanas, ahora puede centrarse en otras tareas mientras se ejecuta la canalización. |
|**Proceso mixto y diverso**|Utilice varias canalizaciones coordinadas de forma confiable en procesos y almacenamientos heterogéneos y escalables. Puede ejecutar los pasos individuales de la canalización en diferentes destinos de proceso, como HDInsight, Data Science Virtual Machine de GPU y Databricks. De esta manera, se hace un uso eficiente de las opciones de proceso disponibles.|
|**Reusabilidad**|Se pueden crear plantillas de canalizaciones en escenarios específicos, como reentrenamiento y puntuación por lotes. Y puede desencadenarlas desde sistemas externos mediante llamadas REST sencillas.|
|**Seguimiento y control de versiones**|En lugar de hacer un seguimiento manual de los datos y los resultados conforme itera, use el SDK de canalizaciones para asignar de forma explícita un nombre y una versión a los orígenes de datos, las entradas y las salidas. También puede administrar scripts y datos por separado para aumentar la productividad.|

## <a name="the-python-sdk-for-pipelines"></a>SDK de Python para las canalizaciones

Use Python para crear sus canalizaciones de ML. El SDK de Azure Machine Learning ofrece construcciones imperativas para definir la secuencia y el paralelismo de los pasos descritos de las canalizaciones cuando no existen ninguna dependencia de datos. Puede interactuar con él en cuadernos de Jupyter, o en otro entorno de desarrollo integrado preferido. 

Con las dependencias de datos declarativas, puede optimizar sus tareas. El SDK incluye un marco de módulos compilados previamente para tareas comunes, como la transferencia de datos y la publicación de modelos. Puede ampliar el marco para modelar sus propias convenciones, mediante la implementación de pasos personalizados que se puedan reutilizar en las canalizaciones. También puede administrar los destinos de proceso y los recursos de almacenamiento directamente desde el SDK.

Puede guardar las canalizaciones como plantillas e implementarlas en un punto de conexión de REST, para así programar trabajos de puntuación por lotes o reentrenamiento.

Para ver cómo crear las suyas propias, consulte los [documentos de referencia del SDK de Python para canalizaciones](https://docs.microsoft.com/python/api/azureml-pipeline-core/?view=azure-ml-py) y el cuaderno de la sección siguiente.

## <a name="example-notebooks"></a>Cuadernos de ejemplo
 
Los cuadernos siguientes demuestran las canalizaciones con Azure Machine Learning: [how-to-use-azureml/machine-learning-pipelines](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines).
 
[!INCLUDE [aml-clone-in-azure-notebook](../../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>Pasos siguientes

Aprenda a [crear su primera canalización](how-to-create-your-first-pipeline.md).
