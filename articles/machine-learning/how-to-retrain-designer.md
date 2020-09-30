---
title: Volver a entrenar modelos mediante el diseñador de Azure Machine Learning
titleSuffix: Azure Machine Learning
description: Aprenda a volver a entrenar modelos con canalizaciones publicadas en el diseñador de Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: keli19
author: likebupt
ms.date: 04/06/2020
ms.topic: conceptual
ms.custom: how-to, designer
ms.openlocfilehash: d43bea855d9ac3dc34b8e72adcd9577e5933e52c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90905670"
---
# <a name="retrain-models-with-azure-machine-learning-designer"></a>Volver a entrenar modelos con el diseñador de Azure Machine Learning


Siga los pasos de este artículo para aprender a usar el diseñador de Azure Machine Learning para volver a entrenar un modelo de aprendizaje automático. Usará canalizaciones publicadas para automatizar el flujo de trabajo y establecer los parámetros para entrenar el modelo con nuevos datos. 

En este artículo aprenderá a:

> [!div class="checklist"]
> * Entrenar un modelo de Machine Learning.
> * Crear un parámetro de canalización.
> * Publicar la canalización de entrenamiento.
> * Volver a entrenar el modelo con nuevos parámetros.

## <a name="prerequisites"></a>Prerrequisitos

* Un área de trabajo de Azure Machine Learning
* Complete la parte 1 de esta serie de procedimientos, [Transformación de datos en el diseñador](how-to-designer-transform-data.md).

[!INCLUDE [machine-learning-missing-ui](../../includes/machine-learning-missing-ui.md)]

En este artículo se da por hecho que tiene algunos conocimientos sobre la creación de canalizaciones en el diseñador. Para una introducción guiada, complete el [tutorial](tutorial-designer-automobile-price-train-score.md). 

### <a name="sample-pipeline"></a>Canalización de ejemplo

La canalización usada en este artículo es una versión modificada de la canalización de ejemplo [Predicción de ingresos](samples-designer.md#classification) de la página principal del diseñador. La canalización usa el módulo [Import Data](algorithm-module-reference/import-data.md) (Importación de datos) en lugar del conjunto de datos de ejemplo para mostrarle cómo entrenar un modelo con sus propios datos.

![Captura de pantalla que muestra la canalización de ejemplo modificada con un cuadro que resalta el módulo Importación de datos](./media/how-to-retrain-designer/modified-sample-pipeline.png)

## <a name="create-a-pipeline-parameter"></a>Creación de un parámetro de canalización

Cree parámetros de canalización para establecer variables de forma dinámica en tiempo de ejecución. En este ejemplo, cambiará la ruta de acceso a los datos de entrenamiento de un valor fijo a un parámetro, de modo que pueda volver a entrenar el modelo con datos diferentes.

1. Seleccione el módulo **Importación de datos**.

    > [!NOTE]
    > En este ejemplo se usa el módulo Import Data (Importación de datos) para acceder a los datos de un almacén de datos registrado. Sin embargo, puede seguir pasos similares si usa patrones de acceso a datos alternativos.

1. En el panel de detalles del módulo, situado a la derecha del lienzo, seleccione el origen de datos.

1. Especifique la ruta de acceso a los datos. También puede seleccionar **Browse path** (Examinar ruta de acceso) para examinar el árbol de archivos. 

1. Mantenga el puntero sobre el campo **Path** (Ruta de acceso) y seleccione los puntos suspensivos que aparecen sobre el campo **Path** (Ruta de acceso).

    ![Captura de pantalla que muestra cómo crear un parámetro de canalización](media/how-to-retrain-designer/add-pipeline-parameter.png)

1. Seleccione **Add to pipeline parameter** (Agregar al parámetro de canalización).

1. Proporcione un nombre de parámetro y un valor predeterminado.

   > [!NOTE]
   > Puede inspeccionar y editar los parámetros de canalización; para ello, seleccione el icono de engranaje de **Configuración** situado junto al título del borrador de la canalización. 

1. Seleccione **Guardar**.

1. Envíe la ejecución de la canalización.

## <a name="find-a-trained-model"></a>Búsqueda de un modelo entrenado

El diseñador guarda todas las salidas de la canalización, incluidos los modelos entrenados, en la cuenta de almacenamiento del área de trabajo predeterminada. También puede acceder a los modelos entrenados directamente en el diseñador:

1. Espere a que finalice la ejecución de la canalización.
1. Seleccione el módulo **Train Model** (Entrenar modelo).
1. En el panel de detalles del módulo, situado a la derecha del lienzo, seleccione **Outputs + logs** (Salidas y registros).
1. Puede encontrar el modelo en **Other outputs** (Otras salidas) junto con los registros de ejecución.
1. Como alternativa, seleccione el icono **View output** (Ver salida). A partir de aquí, puede seguir las instrucciones del cuadro de diálogo para ir directamente al almacén de datos. 

> [!div class="mx-imgBorder"]
> ![Captura de pantalla que muestra cómo descargar el modelo entrenado](./media/how-to-retrain-designer/trained-model-view-output.png)

## <a name="publish-a-training-pipeline"></a>Publicación de una canalización de entrenamiento

Publique una canalización en un punto de conexión de canalización para volver a usar fácilmente las canalizaciones en el futuro. Un punto de conexión de canalización crea un punto de conexión REST para invocar a la canalización en el futuro. En este ejemplo, el punto de conexión de canalización le permite reutilizar la canalización para volver a entrenar un modelo con datos diferentes.

1. Seleccione la opción **Publicar** situada sobre el lienzo del diseñador.
1. Seleccione o cree un punto de conexión de canalización.

   > [!NOTE]
   > Puede publicar varias canalizaciones en un solo punto de conexión. Cada canalización de un punto de conexión determinado recibe un número de versión, que se puede especificar cuando se llama al punto de conexión de canalización.

1. Seleccione **Publicar**.

## <a name="retrain-your-model"></a>Volver a entrenar el modelo

Ahora que tiene una canalización de entrenamiento publicada, puede usarla para volver a entrenar el modelo con nuevos datos. Puede enviar ejecuciones desde un punto de conexión de canalización desde el área de trabajo de Studio o mediante programación.

### <a name="submit-runs-by-using-the-studio-portal"></a>Envío de ejecuciones mediante el portal de Studio

Realice los pasos siguientes para enviar una ejecución de punto de conexión de canalización con parámetros desde el portal de Studio:

1. Vaya a la página **Endpoints** (Puntos de conexión) del área de trabajo de Studio.
1. Seleccione la pestaña **Puntos de conexión de la canalización**. Seleccione el punto de conexión de canalización.
1. Seleccione la pestaña **Canalizaciones publicadas**. A continuación, seleccione la versión de canalización que desea ejecutar.
1. Seleccione **Submit** (Enviar).
1. En el cuadro de diálogo de instalación, puede especificar los valores de los parámetros para la ejecución. En este ejemplo, actualice la ruta de acceso a los datos para entrenar el modelo con un conjunto de datos que no es de Estados Unidos.

![Captura de pantalla que muestra cómo configurar una ejecución de canalización parametrizada en el diseñador](./media/how-to-retrain-designer/published-pipeline-run.png)

### <a name="submit-runs-by-using-code"></a>Envío de ejecuciones mediante código

Puede encontrar el punto de conexión REST de una canalización publicada en el panel de información general. Mediante una llamada al punto de conexión, puede volver a entrenar la canalización publicada.

Para hacer una llamada a REST, necesitará un encabezado de autenticación de tipo portador de OAuth 2.0. Para obtener información sobre cómo configurar la autenticación en el área de trabajo y realizar una llamada a REST con parámetros, consulte el tutorial [Compilación de una canalización de Azure Machine Learning para la puntuación por lotes](tutorial-pipeline-batch-scoring-classification.md#publish-and-run-from-a-rest-endpoint).

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a crear un punto de conexión de canalización de entrenamiento con parámetros mediante el diseñador.

Para obtener un tutorial completo sobre cómo implementar un modelo para realizar predicciones, consulte [Tutorial del diseñador](tutorial-designer-automobile-price-train-score.md) para entrenar e implementar un modelo de regresión.
