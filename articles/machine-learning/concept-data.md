---
title: Acceso seguro a datos en la nube
titleSuffix: Azure Machine Learning
description: Obtenga información acerca de cómo conectarse de forma segura a los datos en Azure Machine Learning y cómo usar conjuntos de datos y almacenes de datos para tareas de Machine Learning. Los almacenes de datos pueden almacenar datos de blobs de Azure, Azure Data Lake Gen 1 y 2, bases de datos SQL y Azure Databricks.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: nibaccam
author: nibaccam
ms.author: nibaccam
ms.date: 08/31/2020
ms.custom: devx-track-python
ms.openlocfilehash: 85aa78223b8db76cfb430341b19878bc6bbe87d7
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89651172"
---
# <a name="secure-data-access-in-azure-machine-learning"></a>Acceso seguro a datos en Azure Machine Learning

Azure Machine Learning facilita la conexión con los datos en la nube.  Proporciona una capa de abstracción en el servicio de almacenamiento subyacente, por lo que puede acceder de forma segura a los datos y trabajar con ellos sin tener que escribir código específico para su tipo de almacenamiento. Azure Machine Learning también ofrece las siguientes funcionalidades de datos:

*    Interoperabilidad con DataFrames de Pandas y Spark
*    Control de versiones y seguimiento del linaje de datos
*    Etiquetado de datos 
*    Supervisión del desfase de datos
    
## <a name="data-workflow"></a>Flujo de trabajo de datos

Cuando esté listo para usar los datos de la solución de almacenamiento basada en la nube, se recomienda el siguiente flujo de trabajo de entrega de datos. En este flujo de trabajo se supone que tiene una [cuenta de Azure Storage ](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account?tabs=azure-portal) y datos en un servicio de almacenamiento basado en la nube en Azure. 

1. Cree un [almacén de datos de Azure Machine Learning](#datastores) para almacenar la información de conexión en Azure Storage.

2. En ese almacén de datos, cree un [conjunto de datos de Azure Machine Learning](#datasets) para que apunte a un archivo o archivos específicos del almacenamiento subyacente. 

3. Para usar ese conjunto de datos en el experimento de aprendizaje automático; puede:
    1. montarlo en el destino de proceso del experimento para el entrenamiento del modelo

        **OR** 

    1. consumirlo directamente en soluciones de Azure Machine Learning como las ejecuciones de experimentos de aprendizaje automático automatizado (ML automatizado), las canalizaciones de aprendizaje automático o el [diseñador de Azure Machine Learning](concept-designer.md).

4. Cree [monitores de conjunto de datos](#data-drift) para el conjunto de datos de salida del modelo con el fin de detectar desfases de datos. 

5. Si se detecta un desfase de datos, actualice el conjunto de datos de entrada y vuelva a entrenar el modelo como corresponde.

En el siguiente diagrama se ofrece una demostración visual de este flujo de trabajo recomendado.

![Diagrama de concepto de datos](./media/concept-data/data-concept-diagram.svg)

## <a name="datastores"></a>Almacenes de datos

Los almacenes de datos de Azure Machine Learning conservan de forma segura la información de conexión en Azure Storage, por lo que no tiene que codificarla en los scripts. [Registre y cree un almacén de datos](how-to-access-data.md) para conectarse fácilmente a su cuenta de almacenamiento y acceder a los datos del servicio de almacenamiento de Azure subyacente. 

A continuación se indican los servicios de almacenamiento basados en la nube de Azure que se pueden registrar como almacenes de datos:

+ Azure Blob Container
+ Recurso compartido de archivos de Azure
+ Azure Data Lake
+ Azure Data Lake Gen2
+ Azure SQL Database
+ Azure Database for PostgreSQL
+ Sistema de archivos de Databricks
+ Azure Database for MySQL

## <a name="datasets"></a>Conjuntos de datos

Los conjuntos de datos de Azure Machine Learning son referencias que apuntan a los datos del servicio de almacenamiento. No son copias de sus datos. Mediante la creación de un conjunto de datos de Azure Machine Learning, creará una referencia a la ubicación del origen de datos, junto con una copia de sus metadatos. 

Dado que los conjuntos de datos se evalúan de forma diferida y los datos permanecen en su ubicación existente, usted:

* No generará ningún costo de almacenamiento adicional.
* No se arriesgará de forma no intencionada cambiando sus orígenes de datos originales.
* Mejorará las velocidades de rendimiento del flujo de trabajo de ML.

Para interactuar con los datos del almacenamiento, [cree un conjunto de datos](how-to-create-register-datasets.md) para empaquetar los datos en un objeto consumible para las tareas de aprendizaje automático. Registre el conjunto de datos en el área de trabajo para compartirlo y reutilizarlo en distintos experimentos sin las complejidades de la ingesta de datos.

Los conjuntos de datos se pueden crear a partir de archivos locales, direcciones URL públicas, instancias de [Azure Open Datasets](https://azure.microsoft.com/services/open-datasets/) o servicios de almacenamiento de Azure mediante almacenes de datos. 

Hay 2 tipos de conjuntos de datos: 

+ [FileDataset](https://docs.microsoft.com/python/api/azureml-core/azureml.data.file_dataset.filedataset?view=azure-ml-py&preserve-view=true) hace referencia a uno o varios archivos de los almacenes de datos o direcciones URL públicas. Si los datos ya están limpios y listos para su uso en experimentos de entrenamiento, puede [descargar o montar archivos](how-to-train-with-datasets.md#mount-files-to-remote-compute-targets) a los que hacen referencia los objetos FileDataset en el destino de proceso.

+ [TabularDataset](https://docs.microsoft.com/python/api/azureml-core/azureml.data.tabulardataset?view=azure-ml-py&preserve-view=true) representa los datos en formato tabular mediante el análisis del archivo o la lista de archivos proporcionados. Puede cargar un objeto TabularDataset en un dataframe de Pandas o Spark para manipularlo o limpiarlo mejor. Para obtener una lista completa de formatos de datos a partir de los cuales se pueden crear objetos TabularDataset, consulte la [clase TabularDatasetFactory](https://aka.ms/tabulardataset-api-reference).

En la siguiente documentación se pueden encontrar funcionalidades adicionales de los conjuntos de datos:

+ [Versión y seguimiento](how-to-version-track-datasets.md) del linaje del conjunto de datos.
+ [Supervisión del conjunto de datos](how-to-monitor-datasets.md) para ayudar con la detección de desfases de datos.    

## <a name="work-with-your-data"></a>Trabajo con los datos

Con los conjuntos de datos puede realizar una serie de tareas de aprendizaje automático, gracias a la perfecta integración con las características de Azure Machine Learning. 

+ Crear un [proyecto de etiquetado de datos](#label).
+ Entrenar modelos de Machine Learning:
     + [experimentos de ML automatizado](how-to-use-automated-ml-for-ml-models.md)
     + el [diseñador](tutorial-designer-automobile-price-train-score.md#import-data)
     + [notebooks](how-to-train-with-datasets.md)
     + [Canalizaciones de Azure Machine Learning](how-to-create-your-first-pipeline.md)
+ Acceder a los conjuntos de datos para la puntuación con la [inferencia por lotes](how-to-use-parallel-run-step.md) en [canalizaciones de aprendizaje automático](how-to-create-your-first-pipeline.md).
+ Configurar un monitor de conjunto de datos para la detección de [desfase de datos](#drift).

<a name="label"></a>

## <a name="data-labeling"></a>Etiquetado de datos

El etiquetado de grandes cantidades de datos a menudo ha resultado un dolor de cabeza en los proyectos de aprendizaje automático. Los que incluyen un componente de Computer Vision, como la clasificación de imágenes o la detección de objetos, normalmente requieren miles de imágenes y sus etiquetas correspondientes.

Azure Machine Learning le proporciona una ubicación central para crear, administrar y supervisar proyectos de etiquetado. Los proyectos de etiquetado ayudan a coordinar los datos, las etiquetas y los miembros del equipo, lo que le permite administrar de forma más eficaz las tareas de etiquetado. Actualmente, las tareas admitidas son la clasificación de imágenes (de varias etiquetas y de varias clases) y la identificación de objetos mediante cuadros de límite.

Cree un [proyecto de etiquetado de datos](how-to-create-labeling-projects.md) y genere un conjunto de datos para su uso en experimentos de aprendizaje automático.

<a name="drift"></a>

## <a name="data-drift"></a>Desfase de datos

En el contexto de aprendizaje automático, el desfase de datos es el cambio en los datos de entrada del modelo que conduce a la degradación del rendimiento del modelo. Es uno de los principales motivos por los que la precisión del modelo se degrada con el tiempo. Por lo tanto, la supervisión del desfase de datos ayuda a detectar problemas de rendimiento del modelo.

Consulte el artículo sobre [creación de monitores de conjunto de datos](how-to-monitor-datasets.md) para obtener información sobre cómo detectar y alertar sobre el desfase de datos en los nuevos datos de un conjunto de datos.

## <a name="next-steps"></a>Pasos siguientes 

+ Cree un conjunto de datos en Azure Machine Learning Studio o con el SDK de Python [mediante estos pasos](how-to-create-register-datasets.md).
+ Pruebe los ejemplos de entrenamiento del conjunto de datos con nuestros [cuadernos de ejemplo](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/work-with-data/).
