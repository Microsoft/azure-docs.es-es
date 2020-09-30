---
title: Qué es un área de trabajo
titleSuffix: Azure Machine Learning
description: El área de trabajo es el recurso de nivel superior de Azure Machine Learning. Conserva un historial de todas las ejecuciones del entrenamiento, que incluye registros, métricas, resultados y una instantánea de sus scripts. Esta información se usa para determinar qué ejecución de entrenamiento crea el mejor modelo.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sgilley
author: sdgilley
ms.date: 09/22/2020
ms.openlocfilehash: 7185adf559f429feb0ada60fef65e1edb106da66
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907630"
---
# <a name="what-is-an-azure-machine-learning-workspace"></a>¿Qué es un área de trabajo de Azure Machine Learning?

El área de trabajo es el recurso de nivel superior para Azure Machine Learning, que proporciona un lugar centralizado para trabajar con todos los artefactos que crea al usar Azure Machine Learning.  El área de trabajo conserva un historial de las ejecuciones del entrenamiento, que incluye registros, métricas, resultados y una instantánea de sus scripts. Esta información se usa para determinar qué ejecución de entrenamiento crea el mejor modelo.  

Una vez que tenga un modelo que le guste, regístrelo con el área de trabajo. Después, usará el modelo registrado y los scripts de puntuación para implementar en Azure Container Instances, en Azure Kubernetes Service o en una matriz de puerta programable en el campo (FPGA) como un punto de conexión HTTP basado en REST. También puede implementar el modelo en un dispositivo de Azure IoT Edge como un módulo.

## <a name="taxonomy"></a>Taxonomía 

El diagrama siguiente es una taxonomía del área de trabajo:

[![Taxonomía del área de trabajo](./media/concept-workspace/azure-machine-learning-taxonomy.png)](./media/concept-workspace/azure-machine-learning-taxonomy.png#lightbox)

En el diagrama se muestran los siguientes componentes de un área de trabajo:

+ Un área de trabajo puede incluir [instancias de proceso de Azure Machine Learning](concept-compute-instance.md), recursos en la nube configurados con el entorno de Python necesario para ejecutar Azure Machine Learning.

+ Los [roles de usuario](how-to-assign-roles.md) le permiten compartir su área de trabajo con otros usuarios, equipos o proyectos.
+ Los [destinos de proceso](concept-azure-machine-learning-architecture.md#compute-targets) se usan para ejecutar sus experimentos.
+ Al crear el área de trabajo, los [recursos asociados](#resources) también se crean automáticamente.
+ Los [experimentos](concept-azure-machine-learning-architecture.md#experiments) son ejecuciones de entrenamiento que usa para entrenar sus modelos.  
+ Las [canalizaciones](concept-azure-machine-learning-architecture.md#ml-pipelines) son flujos de trabajo reutilizables para entrenar el modelo y repetir dicho proceso.
+ Los [conjuntos de datos](concept-azure-machine-learning-architecture.md#datasets-and-datastores) ayudan en la administración de los datos que usa para el entrenamiento del modelo y la creación de canalizaciones.
+ Una vez que tenga un modelo que desee implementar, cree un modelo registrado.
+ Use el modelo registrado y un script de puntuación para crear un [punto de conexión de implementación](concept-azure-machine-learning-architecture.md#endpoints).

## <a name="tools-for-workspace-interaction"></a>Herramientas para la interacción con el área de trabajo

Puede interactuar con el área de trabajo de las siguientes formas:

> [!IMPORTANT]
> Las herramientas marcadas (versión preliminar) a continuación se encuentran actualmente en versión preliminar pública.
> Se ofrece la versión preliminar sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

+ En la Web:
    + [Azure Machine Learning Studio](https://ml.azure.com) 
    + [Diseñador de Azure Machine Learning](concept-designer.md) 
+ En cualquier entorno de Python con el [SDK de Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py&preserve-view=true).
+ En cualquier entorno de R con el [SDK de Azure Machine Learning para R (versión preliminar)](https://azure.github.io/azureml-sdk-for-r/reference/index.html).
+ En la línea de comandos con la [extensión de la CLI](https://docs.microsoft.com/azure/machine-learning/reference-azure-machine-learning-cli) de Azure Machine Learning
+ [Extensión Azure Machine Learning para VS Code](how-to-manage-resources-vscode.md#workspaces)


## <a name="machine-learning-with-a-workspace"></a>Aprendizaje automático con un área de trabajo

Las tareas de aprendizaje automático leen o escriben artefactos en el área de trabajo.

+ Ejecute un experimento para entrenar un modelo: escribe los resultados de la ejecución del experimento en el área de trabajo.
+ Use Machine Learning automatizado para entrenar un modelo: escribe los resultados del entrenamiento en el área de trabajo.
+ Registre un modelo en el área de trabajo.
+ Implemente un modelo: usa el modelo registrado para crear una implementación.
+ Cree y ejecute flujos de trabajo reutilizables.
+ Vea artefactos de aprendizaje automático como experimentos, canalizaciones, modelos o implementaciones.
+ Realice un seguimiento de los modelos y supervíselos.

## <a name="workspace-management"></a>Administración de áreas de trabajo

También puede realizar las siguientes tareas de administración de áreas de trabajo:

| Tarea de administración de áreas de trabajo   | Portal              | Estudio | SDK para Python/SDK para R       | CLI        | Código de VS
|---------------------------|---------|---------|------------|------------|------------|
| Crear un área de trabajo        | **&check;**     | | **&check;** | **&check;** | **&check;** |
| Administración del acceso al área de trabajo    | **&check;**   || |  **&check;**    ||
| Creación y administración de recursos de proceso    | **&check;**   | **&check;** | **&check;** |  **&check;**   ||
| Creación de una máquina virtual de Notebook |   | **&check;** | |     ||

> [!WARNING]
> No se admite mover el área de trabajo de Azure Machine Learning a otra suscripción ni mover la suscripción propietaria a un nuevo inquilino. Si lo hace, pueden producirse errores.

## <a name="create-a-workspace"></a><a name='create-workspace'></a> Creación de un área de trabajo

Hay varias maneras de crear un área de trabajo:  

* Use [Azure Portal](how-to-manage-workspace.md) si quiere utilizar una interfaz de apuntar y hacer clic que le guíe por cada paso.
* Use el [SDK de Azure Machine Learning para Python](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py#&preserve-view=trueworkspace) si quiere crear un área de trabajo al momento a partir de scripts de Python o cuadernos de Jupyter.
* Use una [plantilla de Azure Resource Manager](how-to-create-workspace-template.md) o la [CLI de Azure Machine Learning](reference-azure-machine-learning-cli.md) cuando necesite automatizar o personalizar la creación con estándares de seguridad corporativos.
* Si trabaja en Visual Studio Code, use la [extensión de VS Code](how-to-manage-resources-vscode.md#create-a-workspace).

> [!NOTE]
> El nombre del área de trabajo no distingue mayúsculas de minúsculas.

## <a name="associated-resources"></a><a name="resources"></a> Recursos asociados

Al crear una nueva área de trabajo, se crean automáticamente varios recursos de Azure que el área de trabajo utiliza:

+ [Cuenta de Azure Storage](https://azure.microsoft.com/services/storage/): se usa como almacén de datos predeterminado para el área de trabajo.  Las instancias de Jupyter Notebook que se usan con las instancias de proceso de Azure Machine Learning también se almacenan aquí. 
  
  > [!IMPORTANT]
  > De manera predeterminada, la cuenta de almacenamiento es una cuenta de uso general v1. Puede [actualizarla a una de uso general v2](https://docs.microsoft.com/azure/storage/common/storage-account-upgrade) una vez creada el área de trabajo. No habilite el espacio de nombres jerárquico en la cuenta de almacenamiento después de actualizar a la versión v2 de uso general.

  Para usar una cuenta de Azure Storage existente, no puede ser una cuenta Premium (Premium_LRS o Premium_GRS). Tampoco puede tener un espacio de nombres jerárquico (se usa con Azure Data Lake Storage Gen2). No se admite Premium Storage ni el espacio de nombres jerárquico con la cuenta de almacenamiento _predeterminada_ del área de trabajo. Puede usar Premium Storage o el espacio de nombres jerárquico con cuentas de almacenamiento _no predeterminadas_.
  
+ [Azure Container Registry](https://azure.microsoft.com/services/container-registry/): registra los contenedores de docker que usa durante el entrenamiento y al implementar un modelo. Para reducir los costos, ACR se **carga de forma diferida** hasta que se crean imágenes de la implementación.

+ [Azure Application Insights](https://azure.microsoft.com/services/application-insights/): almacena información sobre la supervisión de los modelos.

+ [Azure Key Vault](https://azure.microsoft.com/services/key-vault/): almacena secretos que usan los destinos de proceso y otra información confidencial que el área de trabajo necesita.

> [!NOTE]
> Además de crear nuevas versiones, también puede usar los servicios de Azure existentes.

<a name="wheres-enterprise"></a>

## <a name="what-happened-to-enterprise-edition"></a>Qué le ha ocurrido a Enterprise Edition

A partir de septiembre de 2020, todas las capacidades que estaban disponibles en las áreas de trabajo de Enterprise Edition también están disponibles en las áreas de trabajo de Basic Edition. Ya no se pueden crear nuevas áreas de trabajo de Enterprise.  Cualquier llamada de SDK, CLI o Azure Resource Manager que use el parámetro `sku` continuará funcionando, pero se aprovisionará un área de trabajo de Basic.

A partir del 21 de diciembre, todas las áreas de trabajo de Enterprise Edition se establecerán automáticamente en Basic Edition, que tiene las mismas funcionalidades. No se producirá ningún tiempo de inactividad durante este proceso. El 1 de enero de 2021, Enterprise Edition se retirará formalmente. 

En cualquiera de las ediciones, los clientes son responsables de los costos de los recursos de Azure consumidos y no tendrán que pagar cargos adicionales por Azure Machine Learning. Para obtener más información, consulte la [página de precios de Azure Machine Learning](https://azure.microsoft.com/pricing/details/machine-learning/).

## <a name="next-steps"></a>Pasos siguientes

Para una introducción a Azure Machine Learning, consulte:

+ [Información general de Azure Machine Learning](overview-what-is-azure-ml.md)
+ [Creación de un área de trabajo](how-to-manage-workspace.md)
+ [Administración de un área de trabajo](how-to-manage-workspace.md)
+ [Tutorial: Creación del primer experimento de ML con el SDK de Python](tutorial-1st-experiment-sdk-setup.md)
+ [Tutorial: Introducción a Azure Machine Learning con el SDK de R](tutorial-1st-r-experiment.md)
+ [Tutorial: Creación del primer modelo de clasificación con el aprendizaje automático automatizado](tutorial-first-experiment-automated-ml.md) 
+ [Tutorial: Predicción del precio de un automóvil con el diseñador](tutorial-designer-automobile-price-train-score.md)
