---
title: Creación de conjuntos de datos con Azure Open Datasets
titleSuffix: Azure Open Datasets
description: Aprenda a crear un conjunto de datos de Azure Machine Learning en Azure Open Datasets.
ms.service: open-datasets
ms.topic: conceptual
ms.author: nibaccam
author: nibaccam
ms.date: 08/05/2020
ms.custom: how-to, tracking-python
ms.openlocfilehash: 00718e15b069e9be25abdaf0e4460eae20c6afd0
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998082"
---
# <a name="create-azure-machine-learning-datasets-from-azure-open-datasets"></a>Creación de conjuntos de datos de Azure Machine Learning en Azure Open Datasets
[!INCLUDE [aml-applies-to-basic-enterprise-sku](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo, aprenderá a incorporar datos de enriquecimiento mantenidos en sus experimentos de aprendizaje automático locales o remotos con conjuntos de datos de [Azure Machine Learning](../machine-learning/overview-what-is-azure-ml.md) y [Azure Open Datasets](https://docs.microsoft.com/azure/open-datasets/). 

Mediante la creación de un [conjunto de datos de Azure Machine Learning](../machine-learning/how-to-create-register-datasets.md), creará una referencia a la ubicación del origen de datos, junto con una copia de sus metadatos. Dado que los conjuntos de datos se evalúan de forma diferida y los datos permanecen en su ubicación existente, usted:
* No generará ningún costo de almacenamiento adicional.
* No se arriesgará de forma no intencionada cambiando sus orígenes de datos originales. 
* Mejorará las velocidades de rendimiento del flujo de trabajo de ML.

Para comprender el lugar de los almacenes de datos en el flujo de trabajo global de acceso a datos de Azure Machine Learning, consulte el artículo [Acceso seguro a los datos](../machine-learning/concept-data.md#data-workflow).

Azure Open Datasets son conjuntos de datos públicos mantenidos que puede usar para agregar características de escenarios específicos con el fin de enriquecer sus soluciones predictivas y mejorar su precisión. Consulte el [Catálogo Open Datasets](https://azure.microsoft.com/en-in/services/open-datasets/catalog/) para encontrar los datos de dominio público que pueden ayudarle a entrenar modelos de Machine Learning, como:

* [tiempo](https://azure.microsoft.com/services/open-datasets/catalog/noaa-integrated-surface-data/)
* [censo](https://azure.microsoft.com/services/open-datasets/catalog/us-decennial-census-zip/)
* [vacaciones](https://azure.microsoft.com/services/open-datasets/catalog/public-holidays/)
* [seguridad pública](https://azure.microsoft.com/services/open-datasets/catalog/chicago-safety-data/)
* ubicación

Los conjuntos de datos abiertos están en la nube en Microsoft Azure y se incluyen tanto en el [SDK de Azure Machine Learning para Python](#create-datasets-with-the-sdk) como en [Azure Machine Learning Studio](#create-datasets-with-the-studio).


## <a name="prerequisites"></a>Prerrequisitos

Para realizar este artículo, necesitará lo siguiente:

* Suscripción a Azure. Si no tiene una, cree una cuenta gratuita antes de empezar. Pruebe la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

* Un [área de trabajo de Azure Machine Learning](../machine-learning/how-to-manage-workspace.md).

* El [SDK de Azure Machine Learning para Python instalado](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py), que incluye el paquete `azureml-datasets`.

    * Cree una [instancia de proceso de Azure Machine Learning](../machine-learning/concept-compute-instance.md#managing-a-compute-instance) que sea un entorno de desarrollo completamente configurado y administrado que incluya cuadernos integrados y el SDK ya instalado.

    **OR**

    * Trabaje en su propio entorno de Python e instale el SDK con [estas instrucciones](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py).

> [!NOTE]
> Algunas clases de conjunto de tipos tienen dependencias en el paquete [azureml-dataprep](https://docs.microsoft.com/python/api/azureml-dataprep/?view=azure-ml-py), que solo es compatible con Python de 64 bits. Para los usuarios de Linux, estas clases solo se admiten en las siguientes distribuciones:  Red Hat Enterprise Linux (7, 8), Ubuntu (14.04, 16.04, 18.04), Fedora (27, 28), Debian (8, 9) y CentOS (7).

## <a name="create-datasets-with-the-sdk"></a>Creación de conjuntos de datos con el SDK

Para crear conjuntos de datos de Azure Machine Learning mediante clases de Azure Open Datasets en el SDK de Python, el paquete se debe haber instalado con `pip install azureml-opendatasets`. Cada conjunto de datos discretos se representa mediante su propia clase en el SDK y ciertas clases están disponibles como Azure Machine Learning [`TabularDataset`, `FileDataset`](../machine-learning/how-to-create-register-datasets.md#dataset-types), o ambos. Consulte la [documentación de referencia](https://docs.microsoft.com/python/api/azureml-opendatasets/azureml.opendatasets?view=azure-ml-py) para obtener una lista completa de clases `opendatasets`.

Puede recuperar ciertas clases `opendatasets` como `TabularDataset` o `FileDataset`, lo que le permite manipular y descargar los archivos directamente. Otras clases **solo** pueden obtener un conjunto de datos mediante las funciones `get_tabular_dataset()` o `get_file_dataset()` de la clase `Dataset` en el SDK de Python.

En el código siguiente se muestra que la clase `opendatasets` de MNIST puede devolver `TabularDataset` o `FileDataset`. 


```python
from azureml.core import Dataset
from azureml.opendatasets import MNIST

# MNIST class can return either TabularDataset or FileDataset
tabular_dataset = MNIST.get_tabular_dataset()
file_dataset = MNIST.get_file_dataset()
```

En este ejemplo, la clase `opendatasets` de Diabetes solo está disponible como `TabularDataset`, de ahí el uso de `get_tabular_dataset()`.

```python

from azureml.opendatasets import Diabetes
from azureml.core import Dataset

# Diabetes class can return ONLY TabularDataset and must be called from the static function
diabetes_tabular = Diabetes.get_tabular_dataset()
```
## <a name="register-datasets"></a>Registro de conjuntos de datos

Registre un conjunto de datos de Azure Machine Learning con el área de trabajo, con el fin de que pueda compartirlos con otros usuarios y reutilizarlos en varios experimentos en el área de trabajo. Cuando se registra un conjunto de datos de Azure Machine Learning creado desde Open Datasets, los datos no se descargan inmediatamente, pero se tendrá acceso a ellos más adelante cuando se soliciten (durante el entrenamiento, por ejemplo) desde una ubicación de almacenamiento central.

Para registrar sus conjuntos de datos con un área de trabajo, use el método [`register()`](https://docs.microsoft.com/python/api/azureml-core/azureml.data.abstract_dataset.abstractdataset?view=azure-ml-py#register-workspace--name--description-none--tags-none--create-new-version-false-). 
```Python
titanic_ds = titanic_ds.register(workspace=workspace,
                                 name='titanic_ds',
                                 description='titanic training data')
```

## <a name="create-datasets-with-the-studio"></a>Creación de conjuntos de datos con Studio

También puede crear conjuntos de datos de Azure Machine Learning a partir de Azure Open Datasets con [Azure Machine Learning Studio](https://ml.azure.com), una interfaz web consolidada que incluye herramientas de aprendizaje automático para llevar a la práctica escenarios de ciencia de datos para los profesionales de ciencia de datos de todos los niveles de conocimiento.

> [!Note]
> Los conjuntos de datos creados mediante Azure Machine Learning Studio se registran automáticamente en el área de trabajo.

1. En el área de trabajo, seleccione la pestaña **Conjunto de datos** en **Recursos**. En el menú desplegable **Crear conjunto de datos**, seleccione **From Open Datasets** (Desde Open Datasets).

    ![Abrir un conjunto de datos con la interfaz de usuario](./media/how-to-create-dataset-from-open-dataset/open-datasets-1.png)

1. Seleccione un conjunto de datos seleccionando su icono (tiene la opción de filtrar mediante la barra de búsqueda). Seleccione **Next** (Siguiente).

    ![Elegir conjunto de datos](./media/how-to-create-dataset-from-open-dataset/open-datasets-2.png)

1. Elija el nombre con que va a registrar el conjunto de datos y, opcionalmente, filtre los datos mediante los filtros disponibles. En este caso, para el conjunto de datos de **días festivos público**, filtre el período a un año y el código de país a solo EE. UU. Consulte el [Catálogo de Azure Open Datasets](https://azure.microsoft.com/services/op[en-datasets/catalog/) para ver datos detallados como, por ejemplo, descripciones del campo e intervalos de fechas. Seleccione **Crear**.

    ![Establecer parámetros del conjunto de datos y crear conjunto de datos](./media/how-to-create-dataset-from-open-dataset/open-datasets-3.png)

    El conjunto de valores ahora está disponible en su área de trabajo, en **Conjunto de datos**. Puede usarlo de la misma manera que otros conjuntos de valores que haya creado.


## <a name="access-datasets-for-your-experiments"></a>Acceso a los conjuntos de datos de los experimentos

Use sus conjuntos de datos en los experimentos de aprendizaje automático para entrenar modelos de aprendizaje automático. [Obtenga más información sobre cómo entrenar con conjuntos de datos](../machine-learning/how-to-train-with-datasets.md).

## <a name="example-notebooks"></a>Cuadernos de ejemplo

Puede encontrar ejemplos y demostraciones de la funcionalidad de Open Datasets en estos [cuadernos de ejemplo](samples.md).

## <a name="next-steps"></a>Pasos siguientes

* [Entrenamiento del primer modelo de Machine Learning](../machine-learning/tutorial-1st-experiment-sdk-train.md).

* [Entrenamiento con conjuntos de datos](../machine-learning/how-to-train-with-datasets.md).

* [Creación de un conjunto de datos de Azure Machine Learning](../machine-learning/how-to-create-register-datasets.md).



