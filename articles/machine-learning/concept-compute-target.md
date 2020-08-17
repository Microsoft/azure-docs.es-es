---
title: Qué son los destinos de proceso
titleSuffix: Azure Machine Learning
description: Defina dónde desea entrenar o implementar el modelo con Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sgilley
author: sdgilley
ms.date: 07/27/2020
ms.openlocfilehash: 27c129af9fbf3e76c6c57fbf084596876b51955b
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88141932"
---
#  <a name="what-are-compute-targets-in-azure-machine-learning"></a>¿Qué son los destinos de proceso en Azure Machine Learning? 

Un **destino de proceso** es un entorno o recurso de proceso en el que se ejecuta el script de entrenamiento o se hospeda la implementación del servicio web. Esta ubicación puede ser su equipo local o un recurso de proceso en la nube. El uso de los destinos de proceso le facilita cambiar posteriormente el entorno de proceso sin tener que cambiar el código.  

En un ciclo de vida de desarrollo de modelos típico, puede:
1. Comenzar desarrollando y experimentando con una pequeña cantidad de datos. En esta etapa, le recomendamos su entorno local (equipo local o máquina virtual basada en la nube) como destino de proceso. 
2. Escalar verticalmente a datos más grandes o realizar un entrenamiento distribuido mediante uno de estos [destinos de proceso de entrenamiento](#train).  
3. Una vez que el modelo esté listo, impleméntelo en un entorno de hospedaje web o en un dispositivo de IoT con uno de estos [destinos de proceso de implementación](#deploy).

Los recursos de proceso que use para los destinos de proceso están asociados a un [área de trabajo](concept-workspace.md). Los usuarios del área de trabajo comparten los recursos de proceso que no sean el equipo local.

## <a name="training-compute-targets"></a><a name="train"></a> Entrenamiento de destinos de proceso

Azure Machine Learning tiene distintas modalidades de soporte técnico en los diferentes recursos de proceso.  También puede adjuntar su propio recurso de proceso, aunque el soporte técnico para varios escenarios puede variar.

[!INCLUDE [aml-compute-target-train](../../includes/aml-compute-target-train.md)]

Obtenga más información sobre la [configuración y el uso de un destino de proceso para el entrenamiento del modelo](how-to-set-up-training-targets.md).

## <a name="deployment-targets"></a><a name="deploy"></a>Destinos de implementación

Los siguientes recursos de proceso se pueden usar hospedar la implementación del modelo.

[!INCLUDE [aml-compute-target-deploy](../../includes/aml-compute-target-deploy.md)]

Aprenda [dónde y cómo implementar el modelo en un destino de proceso](how-to-deploy-and-where.md).

<a name="amlcompute"></a>
## <a name="azure-machine-learning-compute-managed"></a>Proceso de Azure Machine Learning (administrado)

Azure Machine Learning crea y administra un recurso de proceso administrado. Dicho proceso está optimizado para cargas de trabajo de Machine Learning. Los clústeres de procesos y las [instancias de procesos](concept-compute-instance.md) de Azure Machine Learning son los únicos procesos administrados. En el futuro podrán agregarse recursos de proceso administrados adicionales.

Puede crear instancias de procesos o clústeres de procesos de Azure Machine Learning con cualquiera de las siguientes opciones:
* Azure Machine Learning Studio
* Azure portal
* SDK de Python [ComputeInstance](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.computeinstance(class)?view=azure-ml-py) y clases de [AmlCompute](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.amlcompute(class)?view=azure-ml-py)
* [SDK de R](https://azure.github.io/azureml-sdk-for-r/reference/index.html#section-compute-targets) (versión preliminar)
* Plantilla de Resource Manager. Para ver una plantilla de ejemplo, consulte [Creación de una plantilla de proceso de Azure Machine Learning](https://github.com/Azure/azure-quickstart-templates/tree/master/101-machine-learning-compute-create-amlcompute).
* Extensión de Machine learning [para la CLI de Azure](reference-azure-machine-learning-cli.md#resource-management).  

Cuando se crean, estos recursos de proceso forman parte automáticamente del área de trabajo, a diferencia de otros tipos de destinos de proceso.


|Capacidad  |Clúster de proceso  |Instancia de proceso  |
|---------|---------|---------|
|Clúster de uno o varios nodos     |    **&check;**       |         |
|Se escala automáticamente cada vez que se envía una ejecución     |     **&check;**      |         |
|Administración del clúster automático y programación de trabajos     |   **&check;**        |     **&check;**      |
|Es compatible con recursos de CPU y GPU     |  **&check;**         |    **&check;**       |


> [!NOTE]
> Cuando un clúster de proceso está inactivo, se escala automáticamente a 0 nodos, por lo que no paga cuando no se usa.  Sin embargo, una *instancia* de proceso está siempre activa y no se escala automáticamente.  Debe [detener la instancia de proceso](tutorial-1st-experiment-sdk-train.md#stop-the-compute-instance) cuando no la esté usando para evitar costos adicionales.

### <a name="supported-vm-series-and-sizes"></a>Series y tamaños de maquina virtual compatibles

Al seleccionar un tamaño de nodo para un recurso de proceso administrado en Azure Machine Learning, puede elegir entre varios tamaños de máquina virtual disponibles en Azure. Azure ofrece una variedad de tamaños para Windows y Linux para diferentes cargas de trabajo. Consulte aquí para obtener más información sobre los distintos [tamaños y tipos de máquinas virtuales](https://docs.microsoft.com/azure/virtual-machines/linux/sizes).

Hay algunas excepciones y limitaciones a la hora de elegir un tamaño de máquina virtual:
* Algunas series de máquinas virtuales no se admiten en Azure Machine Learning.
* Algunas series de máquinas virtuales están restringidas. Para usar una serie restringida, póngase en contacto con el soporte técnico y solicite un aumento de la cuota para la serie. Para información sobre cómo ponerse en contacto con el soporte técnico, consulte [Opciones de soporte técnico de Azure](https://azure.microsoft.com/support/options/).

Consulte la tabla siguiente para más información sobre las series admitidas y las restricciones. 

| **Series de maquinas virtuales compatibles**  | **Restricciones** |
|------------|------------|
| D | None |
| Dv2 | None |  
| DSv2 | None |  
| FSv2 | None | 
| HBv2 | Requiere aprobación |  
| HCS | Requiere aprobación |  
| M | Requiere aprobación |
| NC | None |    
| NCsv2 | Requiere aprobación |
| NCsv3 | Requiere aprobación |  
| NDs | Requiere aprobación |
| NDv2 | Requiere aprobación |
| NV | None |
| NVv3 | Requiere aprobación | 


Aunque Azure Machine Learning admite estas series de máquinas virtuales, puede que no estén disponibles en todas las regiones de Azure. Puede consultar la serie de máquinas virtuales disponibles aquí: [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines).

## <a name="unmanaged-compute"></a>Proceso no administrado

Azure Machine Learning *no*administra un destino de proceso no administrado. Cree este tipo de destino de proceso fuera de Azure Machine Learning y luego adjúntelo al área de trabajo. Estos recursos de proceso no administrados pueden requerir pasos adicionales para mantener o mejorar el rendimiento de las cargas de trabajo de Machine Learning.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo:
* [Configurar un destino de proceso para entrenar el modelo](how-to-set-up-training-targets.md)
* [Implementar el modelo para un destino de proceso](how-to-deploy-and-where.md)
