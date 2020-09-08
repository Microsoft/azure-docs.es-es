---
title: ¿Qué es Azure Machine Learning Studio?
description: Azure Machine Learning Studio es un portal web para las áreas de trabajo de Azure Machine Learning. Studio combina las experiencias de los tipos sin código y código primero para crear una plataforma de ciencia de datos inclusiva.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: overview
author: peterclu
ms.author: peterlu
ms.date: 08/24/2020
ms.openlocfilehash: 4f6252ffbb880e347a89594cb267fb99d576dc7a
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89039331"
---
# <a name="what-is-azure-machine-learning-studio"></a>¿Qué es Azure Machine Learning Studio?

En este artículo, obtendrá información sobre Azure Machine Learning Studio, el portal web para desarrolladores y científicos de datos de [Azure Machine Learning](overview-what-is-azure-ml.md). Studio combina las experiencias de los tipos sin código y código primero para una plataforma de ciencia de datos inclusiva.

En este artículo, aprenderá lo siguiente:
>[!div class="checklist"]
> - Cómo [crear proyectos de aprendizaje automático](#author-machine-learning-projects) en Studio.
> - Cómo [administrar activos y recursos](#manage-assets-and-resources) en Studio.
> - Las diferencias entre [Azure Machine Learning Studio y Machine Learning Studio (clásico)](#ml-studio-classic-vs-azure-machine-learning-studio).


## <a name="author-machine-learning-projects"></a>Creación de proyectos de aprendizaje automático

Studio ofrece varias experiencias de creación en función del tipo de proyecto y el nivel de experiencia del usuario.

+ **Blocs de notas**

  Escriba y ejecute su propio código en servidores de [Jupyter Notebook](how-to-run-jupyter-notebooks.md) administrados que estén integrados directamente en Studio. 

+ **Diseñador de Azure Machine Learning (versión preliminar)**

  Use el diseñador para entrenar e implementar modelos de Machine Learning sin necesidad de escribir código. Arrastre y coloque conjuntos de datos y módulos para crear canalizaciones de Machine Learning. Pruebe el [tutorial del diseñador](tutorial-designer-automobile-price-train-score.md).

    ![Ejemplo del diseñador de Azure Machine Learning](media/concept-designer/designer-drag-and-drop.gif)

+ **Interfaz de usuario de aprendizaje automático automatizado (versión preliminar)**

  Aprenda a crear [experimentos de aprendizaje automático automatizado](tutorial-first-experiment-automated-ml.md) con una interfaz fácil de usar. 

  [![Panel de navegación de Azure Machine Learning Studio](./media/overview-what-is-azure-ml/azure-machine-learning-automated-ml-ui.jpg)](./media/overview-what-is-azure-ml/azure-machine-learning-automated-ml-ui.jpg)

+ **Etiquetado de datos**

    Use el [etiquetado de datos de Azure Machine Learning](how-to-create-labeling-projects.md) para coordinar eficazmente los proyectos de etiquetado de datos.

## <a name="manage-assets-and-resources"></a>Administración de activos y recursos

Administre los recursos de aprendizaje automático directamente en el explorador. Los recursos se comparten en la misma área de trabajo entre el SDK y Studio para ofrecer una experiencia sin problemas. Use Studio para administrar:

- Modelos
- Conjuntos de datos
- Almacenes de datos
- Recursos de proceso
- Cuaderno
- Experimentos
- Registros de ejecución
- Procesos 
- Puntos de conexión de canalización

Studio puede simplificar el modo en que se administran los recursos del área de trabajo incluso para desarrolladores experimentados.

## <a name="ml-studio-classic-vs-azure-machine-learning-studio"></a>Machine Learning Studio (clásico) en comparación con Azure Machine Learning Studio

**Machine Learning Studio (clásico)** se publicó en 2015 y fue nuestro primer generador de aprendizaje automático que usó el modelo de arrastrar y colocar. Es un servicio independiente que solo ofrece una experiencia visual. Studio (clásico) no interopera con Azure Machine Learning.

**Azure Machine Learning** es un servicio independiente y modernizado que ofrece una plataforma de ciencia de datos completa. Admite las experiencias de los tipos código primero y código bajo.

**Azure Machine Learning Studio** es un portal web *de* Azure Machine Learning que contiene las opciones de los tipos código bajo y sin código para la creación de proyectos y la administración de recursos. 

Es aconsejable que los nuevos usuarios elijan **Azure Machine Learning**, en lugar de Machine Learning Studio (clásico), para obtener la gama más reciente de herramientas de ciencia de datos.

### <a name="feature-comparison"></a>Comparación de características

En la tabla siguiente se resumen las principales diferencias entre Machine Learning Studio (clásico) y Azure Machine Learning.

| Característica | ML Studio (clásico) | Azure Machine Learning |
|---| --- | --- |
| Interfaz que permite arrastrar y colocar | Experiencia clásica | Experiencia actualizada: [diseñador de Azure Machine Learning (versión preliminar)](concept-designer.md) <br/>(requiere el área de trabajo de Enterprise) | 
| SDK de código | No compatible | Completamente integrado con los SDK de [Azure Machine Learning Python](https://docs.microsoft.com/python/api/overview/azure/ml/) y [R](tutorial-1st-r-experiment.md) |
| Experimento | Escalable (límite de datos de entrenamiento, 10 GB) | Escalar con destino de proceso |
| Entrenamiento de destinos de proceso | Destino de proceso propietario, solo admite CPU | Amplia gama de [destinos de proceso de entrenamiento](concept-compute-target.md#train) personalizables. Incluye compatibilidad con GPU y CPU | 
| Destinos de proceso de implementación | Formato del servicio web propietario, no personalizable | Amplia gama de [destinos de proceso de implementación](concept-compute-target.md#deploy) personalizables. Incluye compatibilidad con GPU y CPU |
| Canalización de Machine Learning | No compatible | Creación de [canalizaciones](concept-ml-pipelines.md) flexibles y modulares para automatizar flujos de trabajo |
| MLOps | Implementación y administración básicas de modelos | Control de versiones de entidades (modelo, datos, flujos de trabajo), automatización de flujos de trabajo, integración con herramientas de CICD [y más](concept-model-management-and-deployment.md) |
| Formato del modelo | Formato propietario, solo Studio (clásico) | Se admiten varios formatos en función del tipo de trabajo de entrenamiento |
| Entrenamiento de modelos y optimización de hiperparámetros automatizados |  No compatible | [Admitido](concept-automated-ml.md). Opciones de los tipos código primero y sin código. | 
| Detección de desfase de datos | No compatible | [Compatible](how-to-monitor-datasets.md) |
| Proyecto de etiquetado de datos | No compatible | [Compatible](how-to-create-labeling-projects.md) |


## <a name="next-steps"></a>Pasos siguientes

Visite [Studio](https://ml.azure.com) o explore las diferentes opciones de creación con estos tutoriales:  
  + [Uso de cuadernos de Python para entrenar e implementar modelos](tutorial-1st-experiment-sdk-setup.md)
  + [Uso del aprendizaje automático automatizado para entrenar e implementar modelos](tutorial-first-experiment-automated-ml.md)  
  + [Uso del diseñador para entrenar e implementar modelos](tutorial-designer-automobile-price-train-score.md)

