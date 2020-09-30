---
title: Creación de un entrenamiento e implementación de los procesos (Studio)
titleSuffix: Azure Machine Learning
description: Uso de Studio para crear recursos de proceso de entrenamiento e implementación (destinos de proceso) para el aprendizaje automático
services: machine-learning
author: sdgilley
ms.author: sgilley
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.date: 08/06/2020
ms.topic: conceptual
ms.custom: how-to, contperfq1
ms.openlocfilehash: 5345fd81e41bbb354e11e1be23329c3130d4d0c2
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90898085"
---
# <a name="create-compute-targets-for-model-training-and-deployment-in-azure-machine-learning-studio"></a>Creación de destinos de proceso para la implementación y el entrenamiento de modelos en Azure Machine Learning Studio

En este artículo, aprenderá a crear y administrar destinos de proceso en Azure Machine Studio.  También puede crear y administrar destinos de proceso con:

* [SDK de Azure Machine Learning](how-to-create-attach-compute-sdk.md) 
* La [extensión de la CLI](reference-azure-machine-learning-cli.md#resource-management) para Azure Machine Learning
* La [extensión de VS Code](how-to-manage-resources-vscode.md#compute-clusters) para Azure Machine Learning.


## <a name="prerequisites"></a>Requisitos previos

* Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe hoy mismo la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).
* Un [área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

## <a name="whats-a-compute-target"></a>¿Qué es un destino de proceso? 

Con Azure Machine Learning, puede entrenar el modelo en una variedad de recursos o entornos, denominados colectivamente [__destinos de proceso__](concept-azure-machine-learning-architecture.md#compute-targets). Un destino de proceso puede ser una máquina local o un recurso de nube, como una instancia de proceso de Azure Machine Learning, Azure HDInsight o una máquina virtual remota.  También puede crear destinos de proceso para la implementación de modelos tal como se describe en ["Cómo y dónde implementar los modelos"](how-to-deploy-and-where.md).

## <a name="view-compute-targets"></a><a id="portal-view"></a>Visualización de destinos de proceso

Para ver todos los destinos de proceso del área de trabajo, siga estos pasos:

1. Vaya a [Azure Machine Learning Studio](https://ml.azure.com).
 
1. En __Administrar__, seleccione __Proceso__.

1. Seleccione las pestañas de la parte superior para mostrar cada tipo de destino de proceso.

    :::image type="content" source="media/how-to-create-attach-studio/view-compute-targets.png" alt-text="Visualización de la lista de destinos de proceso":::

## <a name="create-compute-target"></a><a id="portal-create"></a>Crear el destino de proceso

Siga los pasos anteriores para ver la lista de destinos de proceso. A continuación, siga estos pasos para crear un destino de proceso:

1. Seleccione la pestaña de la parte superior correspondiente al tipo de proceso que va a crear.

1. Si no tiene ningún destino de proceso, seleccione **Crear** en el área central de la página.
  
    :::image type="content" source="media/how-to-create-attach-studio/create-compute-target.png" alt-text="Crear el destino de proceso":::

1. Si ve una lista de recursos de proceso, seleccione **+ Nuevo** encima de la lista.

    :::image type="content" source="media/how-to-create-attach-studio/select-new.png" alt-text="Seleccionar Nuevo":::


1. Rellene el formulario para el tipo de proceso:

  * [Instancia de proceso](#compute-instance)
  * [Clústeres de proceso](#amlcompute)
  * [Clústeres de inferencia](#inference-clusters)
  * [Proceso adjunto](#attached-compute)

1. Seleccione __Crear__.

1. Vea el estado de la operación de creación seleccionando el destino de proceso en la lista:

    :::image type="content" source="media/how-to-create-attach-studio/view-list.png" alt-text="Visualización del estado del proceso en una lista":::


### <a name="compute-instance"></a>Instancia de proceso

Utilice los [pasos anteriores](#portal-create) para crear la instancia de proceso.  Después, rellene el formulario de la manera siguiente:

:::image type="content" source="media/concept-compute-instance/create-compute-instance.png" alt-text="Crear una nueva instancia de proceso":::


|Campo  |Descripción  |
|---------|---------|
|Nombre del proceso     |  <li>El nombre es obligatorio y debe tener una longitud de entre 3 y 24 caracteres.</li><li>Los caracteres válidos son mayúsculas y minúsculas, dígitos y el carácter **-** .</li><li>El nombre debe empezar con una letra</li><li>El nombre debe ser único en todos los procesos existentes dentro de una región de Azure. Verá una alerta si el nombre elegido no es único</li><li>Si se usa el carácter **-** , debe ir seguido de al menos una letra más adelante en el nombre</li>     |
|Tipo de máquina virtual |  Elija CPU o GPU. Este tipo no se puede cambiar después de la creación     |
|Tamaño de la máquina virtual     |  Los tamaños de máquina virtual admitidos podrían estar restringidos en su región. Comprobación de la [lista de disponibilidad](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines)     |
|Habilitación o deshabilitación del acceso SSH     |   El acceso SSH está deshabilitado de forma predeterminada.  El acceso SSH no se puede. cambiar después de la creación. Asegúrese de habilitar el acceso si tiene previsto depurar de forma interactiva con [VS Code Remote](how-to-set-up-vs-code-remote.md)   |
|Configuración avanzada     |  Opcional. Configurar una red virtual. Especifique el **Grupo de recursos**, **Red virtual** y **Subred** para crear la instancia de proceso dentro de una Azure Virtual Network (vnet). Para más información, consulte estos [requisitos de red](how-to-enable-virtual-network.md#compute-instance) para la red virtual.  |

### <a name="compute-clusters"></a><a name="amlcompute"></a> Clústeres de proceso

Cree un clúster de proceso de uno o varios nodos para el entrenamiento, la inferencia por lotes o el refuerzo de las cargas de trabajo de aprendizaje. Utilice los [pasos anteriores](#portal-create) para crear el clúster de proceso.  Después, rellene el formulario de la manera siguiente:


|Campo  |Descripción  |
|---------|---------|
|Nombre del proceso     |  <li>El nombre es obligatorio y debe tener una longitud de entre 3 y 24 caracteres.</li><li>Los caracteres válidos son mayúsculas y minúsculas, dígitos y el carácter **-** .</li><li>El nombre debe empezar con una letra</li><li>El nombre debe ser único en todos los procesos existentes dentro de una región de Azure. Verá una alerta si el nombre elegido no es único</li><li>Si se usa el carácter **-** , debe ir seguido de al menos una letra más adelante en el nombre</li>     |
|Tipo de máquina virtual |  Elija CPU o GPU. Este tipo no se puede cambiar después de la creación     |
|Prioridad de la máquina virtual | Elija **Dedicado** o **Prioridad baja**.  Las máquinas virtuales de prioridad baja son más económicas, pero no garantizan nodos de proceso. Es posible que se dé prioridad a su trabajo.
|Tamaño de la máquina virtual     |  Los tamaños de máquina virtual admitidos podrían estar restringidos en su región. Comprobación de la [lista de disponibilidad](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines)     |
|Número mínimo de nodos | Número mínimo de nodos que quiere aprovisionar. Si quiere un número de nodos dedicado, establezca este número aquí. Ahorre dinero estableciendo el valor mínimo en 0, por lo que no pagará por ningún nodo cuando el clúster esté inactivo. |
|Número máximo de nodos | Número máximo de nodos que quiere aprovisionar. El proceso se escala automáticamente hasta este número máximo de nodos cuando se envía un trabajo. |
|Configuración avanzada     |  Opcional. Configurar una red virtual. Especifique el **Grupo de recursos**, **Red virtual** y **Subred** para crear la instancia de proceso dentro de una Azure Virtual Network (vnet). Para más información, consulte estos [requisitos de red](how-to-enable-virtual-network.md#compute-instance) para la red virtual.   Adjunte también [identidades administradas](#managed-identity) para conceder acceso a los recursos.     |

#### <a name="set-up-managed-identity"></a><a id="managed-identity"></a> Configuración de la identidad administrada

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-managed-identity-intro.md)]

Durante la creación del clúster de proceso o la edición de los detalles del clúster de proceso, en **Configuración avanzada**, cambie a **Assign a managed identity** (Asignar una identidad administrada) y especifique una identidad asignada por el sistema o una identidad asignada por el usuario.

#### <a name="managed-identity-usage"></a>Uso de la identidad administrada

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-managed-identity-default.md)]

### <a name="inference-clusters"></a>Clústeres de inferencia

> [!IMPORTANT]
> El uso de Azure Kubernetes Service con Azure Machine Learning tiene varias opciones de configuración. Algunos escenarios, como las redes, requieren instalación y configuración adicionales. Para obtener más información sobre el uso de AKS con Azure ML, consulte [Creación y conexión de un clúster de Azure Kubernetes Service](how-to-create-attach-kubernetes.md).

Cree o adjunte un clúster de Azure Kubernetes Service (AKS) para la inferencia a gran escala. Utilice los [pasos anteriores](#portal-create) para crear el clúster de AKS.  Después, rellene el formulario de la manera siguiente:


|Campo  |Descripción  |
|---------|---------|
|Nombre del proceso     |  <li>Se requiere el nombre. Debe tener entre 2 y 16 caracteres. </li><li>Los caracteres válidos son mayúsculas y minúsculas, dígitos y el carácter **-** .</li><li>El nombre debe empezar con una letra</li><li>El nombre debe ser único en todos los procesos existentes dentro de una región de Azure. Verá una alerta si el nombre elegido no es único</li><li>Si se usa el carácter **-** , debe ir seguido de al menos una letra más adelante en el nombre</li>     |
|Kubernetes Service | Seleccione **Crear nuevo** y rellene el resto del formulario.  O seleccione **Usar existente** y, a continuación, seleccione un clúster de AKS existente de su suscripción.
|Region |  Seleccione la región en la que se creará el clúster. |
|Tamaño de la máquina virtual     |  Los tamaños de máquina virtual admitidos podrían estar restringidos en su región. Comprobación de la [lista de disponibilidad](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines)     |
|Propósito del clúster  | Seleccione **Producción** o **Desarrollo/pruebas**. |
|Número de nodos | El número de nodos multiplicado por el número de núcleos de máquina virtual (vCPU) tiene que ser mayor o igual que 12. |
| Network configuration (Configuración de red) | Seleccione **Avanzada** para crear el proceso en una red virtual existente. Para más información sobre AKS en una red virtual, consulte [Aislamiento de red durante el entrenamiento y la inferencia con puntos de conexión privados y redes virtuales](how-to-enable-virtual-network.md#aksvnet). |
| Habilitación de la configuración de SSL | Use esta opción para configurar el certificado SSL en el proceso. |

### <a name="attached-compute"></a>Proceso adjunto

Para usar destinos de proceso creados fuera del área de trabajo de Azure Machine Learning, debe adjuntarlos. Adjuntar un destino de proceso hace que esté disponible para el área de trabajo.  Use **Attached compute** (Proceso adjunto) para adjuntar un destino de proceso para el **entrenamiento**.  Use **Inference clusters** (Clústeres de inferencia) para adjuntar un clúster de AKS para la **inferencia**.

Use los [pasos anteriores](#portal-create) para adjuntar un proceso.  Después, rellene el formulario de la manera siguiente:

1. Escriba un nombre para el destino de proceso. 
1. Seleccione el tipo de proceso que va a adjuntar. No todos los tipos de proceso se pueden adjuntar desde Azure Machine Learning Studio. Los tipos de proceso que se pueden adjuntar actualmente para entrenamiento incluyen:
    * Una máquina virtual remota
    * Azure Databricks (para su uso en las canalizaciones de aprendizaje automático)
    * Azure Data Lake Analytics (para su uso en las canalizaciones de aprendizaje automático)
    * HDInsight de Azure

1. Rellene el formulario y proporcione valores para las propiedades necesarias.

    > [!NOTE]
    > Microsoft recomienda que use claves SSH, que son más seguras que las contraseñas. Las contraseñas son vulnerables a ataques por fuerza bruta. Las claves SSH se basan en las firmas criptográficas. Para información sobre cómo crear claves SSH para usarlas con Azure Virtual Machines, consulte los siguientes documentos:
    >
    > * [Creación y uso de claves SSH en Linux o macOS](https://docs.microsoft.com/azure/virtual-machines/linux/mac-create-ssh-keys)
    > * [Creación y uso de claves SSH en Windows](https://docs.microsoft.com/azure/virtual-machines/linux/ssh-from-windows)

1. Seleccione __Adjuntar__. 


## <a name="next-steps"></a>Pasos siguientes

Después de crear un destino y de adjuntarlo al área de trabajo, lo usará en la [configuración de ejecución](how-to-set-up-training-targets.md) con un objeto `ComputeTarget`:

```python
from azureml.core.compute import ComputeTarget
myvm = ComputeTarget(workspace=ws, name='my-vm-name')
```

* Use el recurso de proceso para [enviar una ejecución de entrenamiento](how-to-set-up-training-targets.md).
* [Tutorial: Entrenamiento de un modelo](tutorial-train-models-with-aml.md) utiliza un destino de proceso administrado para entrenar un modelo.
* Obtenga información sobre cómo [ajustar los hiperparámetros eficazmente](how-to-tune-hyperparameters.md) para crear modelos mejores.
* Cuando tenga un modelo entrenado, aprenda [cómo y dónde implementar los modelos](how-to-deploy-and-where.md).
* [Uso de Azure Machine Learning con Azure Virtual Networks](how-to-enable-virtual-network.md)
