---
title: Compilación de modelos de Machine Learning con el diseñador (versión preliminar)
titleSuffix: Azure Machine Learning
description: Más información sobre los términos, los conceptos y el flujo de trabajo que componen el diseñador para Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: peterlu
author: peterclu
ms.date: 06/28/2020
ms.custom: designer
ms.openlocfilehash: 181f0d62f160a6644e0423be052012521752525f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87012951"
---
# <a name="what-is-azure-machine-learning-designer-preview"></a>¿Qué es el diseñador de Azure Machine Learning (versión preliminar)? 
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-enterprise-sku.md)]

El diseñador de Azure Machine Learning le permite conectar visualmente [conjuntos de datos](#datasets) y [módulos](#module) en un lienzo interactivo para crear modelos de aprendizaje automático. Para más información sobre cómo empezar a trabajar con el diseñador, consulte [Tutorial: Predicción del precio de un automóvil con el diseñador](tutorial-designer-automobile-price-train-score.md)

![Ejemplo del diseñador de Azure Machine Learning](./media/concept-designer/designer-drag-and-drop.gif)

El diseñador usa el [área de trabajo](concept-workspace.md) de Azure Machine Learning para organizar recursos compartidos, por ejemplo:

+ [Canalizaciones](#pipeline)
+ [Conjuntos de datos](#datasets)
+ [Recursos de proceso](#compute)
+ [Modelos registrados](concept-azure-machine-learning-architecture.md#models)
+ [Canalizaciones publicadas](#publish)
+ [Puntos de conexión en tiempo real](#deploy)

## <a name="model-training-and-deployment"></a>Entrenamiento e implementación de modelos

El diseñador proporciona un lienzo visual para compilar, probar e implementar modelos de aprendizaje automático. Con el diseñador puede:

+ Arrastrar [conjuntos de datos](#datasets) y [módulos](#module) y colocarlos en el lienzo.
+ Conectar los módulos para crear un [borrador de canalización](#pipeline-draft).
+ Enviar una [ejecución de canalización](#pipeline-run) mediante los recursos de proceso del área de trabajo de Azure Machine Learning.
+ Convertir las **canalizaciones de entrenamiento** en **canalizaciones de inferencia**.
+ [Publicar](#publish) las canalizaciones en un **punto de conexión de canalización** de REST para enviar una nueva canalización que se ejecute con distintos parámetros y conjuntos de datos.
    + Publique una **canalización de entrenamiento** para reutilizar una sola canalización para el entrenamiento de varios modelos y, al mismo tiempo, cambiar los parámetros y conjuntos de datos.
    + Publique una **canalización de inferencia por lotes** para hacer predicciones sobre nuevos datos mediante un modelo entrenado previamente.
+ [Implementar](#deploy) una **canalización de inferencia en tiempo real** en un punto de conexión en tiempo real para realizar predicciones sobre nuevos datos en tiempo real.

![Diagrama de flujo de trabajo para entrenamiento, inferencia por lotes e inferencia en tiempo real en el diseñador](./media/concept-designer/designer-workflow-diagram.png)

## <a name="pipeline"></a>Canalización

Una [canalización](concept-azure-machine-learning-architecture.md#ml-pipelines) consta de conjuntos de datos y módulos analíticos, que se conectan. Las canalizaciones tienen muchos usos: puede crear una canalización que entrene a un modelo único o una que entrene a varios modelos. Puede crear una canalización que realice predicciones en tiempo real o por lotes, o crear una canalización que solo limpie los datos. Las canalizaciones le permiten reutilizar el trabajo y organizar los proyectos.

### <a name="pipeline-draft"></a>Borrador de canalización

A medida que edita una canalización en el diseñador, el progreso se guarda como un **borrador de canalización**. Puede editar un borrador de canalización en cualquier momento y agregar o quitar módulos, configurar destinos de proceso, crea parámetros, etc.

Una canalización válida tiene estas características:

* Los conjuntos de datos solo pueden conectarse a módulos.
* Los módulos solo se pueden conectar a conjuntos de datos o a otros módulos.
* Todos los puertos de entrada de los módulos deben tener alguna conexión al flujo de datos.
* Deben establecerse todos los parámetros necesarios para cada módulo.

Cuando esté listo para ejecutar el borrador de la canalización, envíe una ejecución de canalización.

### <a name="pipeline-run"></a>Ejecución de la canalización

Cada vez que se ejecuta una canalización, la configuración de esta y sus resultados se almacenan en el área de trabajo como una **ejecución de canalización**. Puede regresar a cualquier ejecución de canalización para inspeccionarla con fines de solución de problemas o auditoría. **Clone** una ejecución de canalización para crear un borrador de canalización y editarlo.

Las ejecuciones de canalización se agrupan en [experimentos](concept-azure-machine-learning-architecture.md#experiments) para organizar el historial de ejecuciones. Puede establecer el experimento de cada ejecución de canalización. 

## <a name="datasets"></a>Conjuntos de datos

Un conjuntos de datos de aprendizaje automático permite acceder a los datos y trabajar con ellos fácilmente. En el diseñador se incluyen muchos conjuntos de valores de ejemplo para experimentar con ellos. Puede [registrar](how-to-create-register-datasets.md) más conjuntos de datos a medida que los necesite.

## <a name="module"></a>módulo

Un módulo es un algoritmo que puede aplicar sobre sus datos. El diseñador tiene varios módulos que van desde las funciones de entrada de datos hasta los procesos de entrenamiento, puntuación y validación.

Un módulo puede tener un conjunto de parámetros que puede utilizar para configurar los algoritmos internos del módulo. Al seleccionar un módulo en el lienzo, los parámetros del módulo se muestran en el panel Propiedades a la derecha del lienzo. Puede modificar los parámetros en ese panel para ajustar su modelo. Puede establecer los recursos de proceso para módulos individuales del diseñador. 

:::image type="content" source="./media/concept-designer/properties.png"alt-text="Propiedades del módulo":::


Si desea ayuda para desplazarse por la biblioteca de algoritmos de aprendizaje automático disponibles, consulte [Información general sobre la referencia de módulos y algoritmos](algorithm-module-reference/module-reference.md). Para obtener ayuda para elegir un algoritmo, vea la [hoja de referencia rápida de algoritmos de Azure Machine Learning](algorithm-cheat-sheet.md).

## <a name="compute-resources"></a><a name="compute"></a>Recursos de proceso

Use recursos de proceso del área de trabajo para ejecutar la canalización y hospedar los modelos implementados como puntos de conexión en tiempo real o como puntos de conexión de canalización (para la inferencia por lotes). Los destinos de proceso admitidos son los siguientes:

| Destino de proceso | Cursos | Implementación |
| ---- |:----:|:----:|
| Proceso de Azure Machine Learning | ✓ | |
| Instancia de proceso de Azure Machine Learning | ✓ | |
| Azure Kubernetes Service | | ✓ |

Los destinos de proceso están conectados a su [área de trabajo de Azure Machine Learning](concept-workspace.md). Los destinos de proceso se administran en el área de trabajo de [Azure Machine Learning Studio](https://ml.azure.com).

## <a name="deploy"></a>Implementación

Para realizar la inferencia en tiempo real, debe implementar una canalización como un **punto de conexión en tiempo real**. El punto de conexión en tiempo real crea una interfaz entre una aplicación externa y el modelo de puntuación. Una llamada a un punto de conexión en tiempo real devuelve resultados de predicción a la aplicación externa en tiempo real. Para realizar una llamada a un punto de conexión en tiempo real, es necesario pasar la clave de API que se creó al implementar el punto de conexión. El punto de conexión se basa en REST, una arquitectura muy usada para proyectos de programación web.

Los puntos de conexión en tiempo real se deben implementar en un clúster de Azure Kubernetes Service.

Para aprender a implementar el modelo, consulte [Tutorial: Implemente un modelo de aprendizaje automático con el diseñador](tutorial-designer-automobile-price-deploy.md).

## <a name="publish"></a>Publicar

También puede publicar una canalización en un **punto de conexión de canalización**. Igual que un punto de conexión en tiempo real, un punto de conexión de canalización permite enviar nuevas ejecuciones de canalización desde aplicaciones externas mediante llamadas REST. Sin embargo, no se pueden enviar ni recibir datos en tiempo real.

Las canalizaciones publicadas son flexibles; se pueden usar para entrenar o volver a entrenar modelos, [realizar la inferencia por lotes](how-to-run-batch-predictions-designer.md), procesar nuevos datos y mucho más. Puede publicar varias canalizaciones en un único punto de conexión de canalización y especificar la versión de canalización que se va a ejecutar.

Una canalización publicada se ejecuta en los recursos de proceso que se definen en el borrador de canalización de cada módulo.

El diseñador crea el mismo objeto [PublishedPipeline](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.publishedpipeline?view=azure-ml-py) como SDK.

## <a name="next-steps"></a>Pasos siguientes

* Aprenda los aspectos básicos del análisis predictivo y del aprendizaje automático con el [Tutorial: Predicción del precio de un automóvil con el diseñador](tutorial-designer-automobile-price-train-score.md)
* Aprenda a modificar los [ejemplos existentes del diseñador](samples-designer.md) para adaptarlos a sus necesidades.
