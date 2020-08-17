---
title: Ejecución de predicciones por lotes mediante el diseñador de Azure Machine Learning (versión preliminar)
titleSuffix: Azure Machine Learning
description: Aprenda a entrenar un modelo y a configurar una canalización de predicción por lotes mediante el diseñador. Implemente la canalización como servicio web con parámetros que se pueda desencadenar desde cualquier biblioteca HTTP.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: peterlu
author: peterclu
ms.date: 02/24/2020
ms.topic: conceptual
ms.custom: how-to, designer
ms.openlocfilehash: 61465210c9ad476c6d8d2987330498aa0efa39d4
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87319616"
---
# <a name="run-batch-predictions-using-azure-machine-learning-designer-preview"></a>Ejecución de predicciones por lotes mediante el diseñador de Azure Machine Learning (versión preliminar)
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-enterprise-sku.md)]

En este artículo, aprenderá a usar el diseñador para crear una canalización de predicción por lotes. La predicción por lotes permite puntuar continuamente grandes conjuntos de valores a petición mediante un servicio web que se puede desencadenar desde cualquier biblioteca HTTP.

En esta guía paso a paso aprenderá a realizar las tareas siguientes:

> [!div class="checklist"]
> * Crear y publicar una canalización de inferencias por lotes
> * Consumir un punto de conexión de canalización
> * Administrar las versiones del punto de conexión

Para aprender a configurar los servicios de puntuación por lotes mediante el SDK, consulte las [instrucciones](how-to-run-batch-predictions.md) adjuntas.

## <a name="prerequisites"></a>Prerrequisitos

En estas instrucciones se da por hecho que ya tiene una canalización de entrenamiento. Para ver una introducción guiada sobre el diseñador, complete la [parte uno del tutorial](tutorial-designer-automobile-price-train-score.md). 

[!INCLUDE [machine-learning-missing-ui](../../includes/machine-learning-missing-ui.md)]

## <a name="create-a-batch-inference-pipeline"></a>Creación de una canalización de inferencias por lotes

Las canalizaciones de entrenamiento deben ejecutarse al menos una vez para poder crear una canalización de inferencias.

1. Vaya a la pestaña **Diseñador** del área de trabajo.

1. Seleccione la canalización de entrenamiento que entrene el modelo que quiere usar para realizar la predicción.

1. **Envíe** la canalización.

    ![Enviar la canalización](./media/how-to-run-batch-predictions-designer/run-training-pipeline.png)

Ahora que se ha ejecutado la canalización de entrenamiento, puede crear una canalización de inferencias por lotes.

1. Al lado de **Submit** (Enviar), seleccione la nueva lista desplegable **Create inference pipeline** (Crear canalización de inferencia).

1. Seleccione **Canalización de inferencias por lotes predeterminada**.

    ![Creación de una canalización de inferencias por lotes](./media/how-to-run-batch-predictions-designer/create-batch-inference.png)
    
El resultado es una canalización de inferencias por lotes predeterminada. 

### <a name="add-a-pipeline-parameter"></a>Incorporación de un parámetro de canalización

Para crear predicciones en los nuevos datos, puede conectarse manualmente a otro conjunto de datos de esta vista de borrador de la canalización, o bien crear un parámetro para el conjunto de datos. Con los parámetros puede cambiar el comportamiento del proceso de inferencia por lotes en el runtime.

En esta sección, se crea un parámetro de conjunto de datos para especificar otro conjunto de datos en el que realizar predicciones.

1. Seleccione el módulo del conjunto de datos.

1. Aparecerá un panel a la derecha del lienzo, en cuya parte inferior debe seleccionar **Set as pipeline parameter** (Establecer como parámetro de canalización).
   
    Escriba el nombre del parámetro o acepte el valor predeterminado.

## <a name="publish-your-batch-inferencing-pipeline"></a>Publicación de una canalización de inferencias por lotes

Ya está listo para implementar la canalización de inferencias. Así se implementará la canalización y se pondrá a disposición de otros usuarios.

1. Seleccione el botón **Publicar**.

1. En el cuadro de diálogo que aparece, expanda la lista desplegable de **PipelineEndpoint** y seleccione **New PipelineEndpoint** (PipelineEndpoint nuevo).

1. Especifique el nombre del punto de conexión y, si lo desea, una descripción del mismo.

    Cerca de la parte inferior del cuadro de diálogo, puede ver el parámetro cuyo valor predeterminado es el identificador del conjunto de datos que usó durante el entrenamiento.

1. Seleccione **Publicar**.

![Publicar una canalización](./media/how-to-run-batch-predictions-designer/publish-inference-pipeline.png)


## <a name="consume-an-endpoint"></a>Consumo de un punto de conexión

Ya tiene una canalización publicada con un parámetro de conjunto de datos. La canalización usará el modelo entrenado creado en la canalización de entrenamiento para puntuar el conjunto de datos que especifique como parámetro.

### <a name="submit-a-pipeline-run"></a>Ejecución de una canalización 

En esta sección, configurará la ejecución manual de una canalización y modificará el parámetro de canalización para puntuar nuevos datos. 

1. Una vez finalizada la implementación, vaya a la sección **Endpoints** (Puntos de conexión).

1. Seleccione **Pipeline endpoints** (Puntos de conexión de canalización).

1. Seleccione el nombre del punto de conexión que ha creado.

![Vínculo al punto de conexión](./media/how-to-run-batch-predictions-designer/manage-endpoints.png)

1. Seleccione **Published pipelines** (Canalizaciones publicadas).

    En esta pantalla se muestran todas las canalizaciones publicadas en este punto de conexión.

1. Seleccione la canalización que ha publicado.

    En la página de detalles de la canalización se muestra el historial de ejecuciones detallado y la información de la cadena de conexión de la canalización. 
    
1. Seleccione **Run** (Ejecutar) para crear una ejecución manual de la canalización.

    ![Detalles de la canalización](./media/how-to-run-batch-predictions-designer/submit-manual-run.png)
    
1. Cambie el parámetro para usar otro conjunto de datos.
    
1. Seleccione **Submit** (Enviar) para ejecutar la canalización.

### <a name="use-the-rest-endpoint"></a>Uso del punto de conexión de REST

Puede encontrar información sobre cómo consumir puntos de conexión de canalización y la canalización publicada en la sección **Puntos de conexión**.

El punto de conexión de REST de un punto de conexión de canalización en el panel de información general de la ejecución. Al llamar al punto de conexión, consume su canalización publicada predeterminada.

También puede consumir una canalización publicada en la página **Published pipelines** (Canalizaciones publicadas). Seleccione una canalización publicada y busque su punto de conexión de REST. 

![Detalles de punto de conexión de REST](./media/how-to-run-batch-predictions-designer/rest-endpoint-details.png)

Para hacer una llamada a REST, necesitará un encabezado de autenticación de tipo portador de OAuth 2.0. Consulte la siguiente [sección de tutorial](tutorial-pipeline-batch-scoring-classification.md#publish-and-run-from-a-rest-endpoint) para más información sobre la configuración de la autenticación en el área de trabajo y la realización de una llamada a REST con parámetros.

## <a name="versioning-endpoints"></a>Puntos de conexión de control de versiones

El diseñador asigna una versión a cada una de las canalizaciones posteriores que publique en un punto de conexión. Puede especificar la versión de la canalización que desea ejecutar como parámetro en su llamada a REST. Si no especifica ningún número de versión, el diseñador usará la canalización predeterminada.

Si publica una canalización, puede elegir que sea la nueva canalización predeterminada para el punto de conexión.

![Establecimiento de la canalización predeterminada](./media/how-to-run-batch-predictions-designer/set-default-pipeline.png)

También puede establecer una canalización predeterminada nueva en la pestaña **Published pipelines** (Canalizaciones publicadas) del punto de conexión.

![Establecimiento de la canalización predeterminada](./media/how-to-run-batch-predictions-designer/set-new-default-pipeline.png)

## <a name="next-steps"></a>Pasos siguientes

Siga el [tutorial](tutorial-designer-automobile-price-train-score.md) del diseñador para entrenar e implementar un modelo de regresión.
''