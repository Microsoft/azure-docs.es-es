---
title: Cuadernos de Jupyter de ejemplo con datos de NOAA
titleSuffix: Azure Open Datasets
description: Use los cuadernos de Jupyter de ejemplo para Azure Open Datasets para obtener información sobre cómo cargar los conjuntos de datos abiertos y usarlos para enriquecer los datos de demostración. Las técnicas incluyen el uso de Spark y Pandas para procesar los datos.
ms.service: open-datasets
ms.topic: sample
author: cjgronlund
ms.author: cgronlun
ms.date: 05/06/2020
ms.openlocfilehash: c24363caac1db8dd8ce21b690ef989b2beb97f2d
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88506063"
---
# <a name="example-jupyter-notebooks-show-how-to-enrich-data-with-open-datasets"></a>Los cuadernos de Jupyter de ejemplo muestran cómo enriquecer los datos con Open Datasets 
Los cuadernos de Jupyter de ejemplo para Azure Open Datasets le muestran cómo cargar conjuntos de datos abiertos y usarlos para enriquecer los datos de demostración. Las técnicas incluyen el uso de Apache Spark y Pandas para procesar los datos.

>[!IMPORTANT]
>Al trabajar en un entorno distinto de Spark, Open Datasets permite descargar un solo mes de datos a la vez, con determinadas clases con el fin de evitar MemoryError con grandes conjuntos de datos.

## <a name="load-noaa-integrated-surface-database-isd-data"></a>Cargar datos de Integrated Surface Database (ISD) de NOAA 
|Notebook        | Descripción                                    |
|----------------|------------------------------------------------|
|[Cargar un mes reciente de datos meteorológicos en un dataframe de Pandas](https://github.com/Azure/OpenDatasetsNotebooks/blob/master/tutorials/data-access/02-weather-to-pandas-dataframe.ipynb) | Obtenga información sobre cómo cargar los datos meteorológicos históricos en su dataframe de Pandas favorito. |
|[Cargar un mes reciente de datos meteorológicos en un dataframe de Spark](https://github.com/Azure/OpenDatasetsNotebooks/blob/master/tutorials/data-access/01-weather-to-spark-dataframe.ipynb) | Obtenga información sobre cómo cargar los datos meteorológicos históricos en su dataframe de Spark favorito.  |

## <a name="join-demo-data-with-noaa-isd-data"></a>Unir datos de demostración con datos de ISD de NOAA 
|Notebook        | Descripción                                    |
|----------------|------------------------------------------------|
|[Combinar datos de demostración con datos meteorológicos: Pandas](https://github.com/Azure/OpenDatasetsNotebooks/blob/master/tutorials/data-join/02-weather-join-in-pandas.ipynb) | Una un conjunto de datos de demostración de un mes de ubicaciones de sensores con lecturas meteorológicas en un dataframe de Pandas.  |
|[Unir datos de demostración con datos meteorológicos: Spark](https://github.com/Azure/OpenDatasetsNotebooks/blob/master/tutorials/data-join/01-weather-join-in-spark.ipynb) | Una un conjunto de datos de demostración de ubicaciones de sensores con lecturas meteorológicas en un dataframe de Spark. |

## <a name="join-nyc-taxi-data-with-noaa-isd-data"></a>Unir datos de taxis de Nueva York con datos de ISD de NOAA 
|Notebook        | Descripción                                    |
|----------------|------------------------------------------------|
|[Datos de viajes de taxis, enriquecidos con datos meteorológicos: Pandas](https://github.com/Azure/OpenDatasetsNotebooks/blob/master/tutorials/data-join/04-nyc-taxi-join-weather-in-pandas.ipynb) | Cargue datos de taxis verdes de la ciudad de Nueva York (más de un mes) y enriquézcalos con datos meteorológicos en un dataframe de Pandas. Este ejemplo reemplaza el método `get_pandas_limit` y equilibra el rendimiento de carga de datos con la cantidad de datos.|
|[Datos de viajes de taxis enriquecidos con datos meteorológicos: Spark](https://github.com/Azure/OpenDatasetsNotebooks/blob/master/tutorials/data-join/03-nyc-taxi-join-weather-in-spark.ipynb) | Cargue datos de taxis verdes de la ciudad de Nueva York y enriquézcalos con datos meteorológicos en un dataframe de Spark.  |

## <a name="next-steps"></a>Pasos siguientes

* [Tutorial: Regression modeling with automated machine learning and an open dataset](/azure/machine-learning/tutorial-auto-train-models?context=azure/open-datasets/context/open-datasets-context) (Tutorial: Modelado de regresión con aprendizaje automático automatizado y un conjunto de datos abierto)
* [SDK de Python para Open Datasets](/python/api/azureml-opendatasets/azureml.opendatasets?view=azure-ml-py)
* [Catálogo de Azure Open Datasets](https://azure.microsoft.com/services/open-datasets/catalog/)
* [Creación de conjuntos de datos de Azure Machine Learning a partir de Open Dataset](how-to-create-azure-machine-learning-dataset-from-open-dataset.md)