---
title: Entornos mantenidos
titleSuffix: Azure Machine Learning
description: Colección de los entornos mantenidos disponibles en Azure Machine Learning
services: machine-learning
author: luisquintanilla
ms.author: luquinta
ms.reviewer: luquinta
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
ms.date: 09/03/2020
ms.openlocfilehash: 4ae96976f81aab9a0949594551c82d3a3fec4f0a
ms.sourcegitcommit: f8d2ae6f91be1ab0bc91ee45c379811905185d07
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89662072"
---
# <a name="azure-machine-learning-curated-environments"></a>Entornos mantenidos de Azure Machine Learning

En este artículo se muestran los entornos mantenidos en Azure Machine Learning y los paquetes y canales que tienen preinstalados. Los entornos mantenidos los proporciona Azure Machine Learning y están disponibles en el área de trabajo de forma predeterminada. Están respaldados por imágenes de Docker en caché, lo que reduce el costo de preparación de la ejecución y permite un tiempo de implementación más rápido. Use estos entornos para empezar a trabajar rápidamente con varios marcos de aprendizaje automático.

> [!NOTE]
> Esta lista se ha actualizado en septiembre de 2020. Use el SDK de Python para obtener la lista más actualizada. Para más información, consulte el [artículo sobre los entornos](./how-to-use-environments.md#use-a-curated-environment).

## <a name="azureml-automl"></a>AzureML-AutoML

**Canales de paquetes:**

* anaconda
* conda-forge
* pytorch

**Paquetes Conda:**

* Python
* numpy
* scikit-learn
* pandas
* py-xgboost
* fbprophet
* vacaciones
* setuptools-git
* psutil

**Paquetes Pip:**

* azureml-core
* azureml-pipeline-core
* azureml-telemetry
* azureml-defaults
* azureml-interpret
* azureml-explain-model
* azureml-automl-core
* azureml-automl-runtime
* azureml-train-automl-client
* azureml-train-automl-runtime
* inference-schema
* py-cpuinfo

## <a name="azureml-automl-dnn"></a>AzureML-AutoML-DNN

**Canales de paquetes:**

* anaconda
* conda-forge
* pytorch

**Paquetes Conda:**

* Python
* numpy
* scikit-learn
* pandas
* py-xgboost
* fbprophet
* vacaciones
* setuptools-git
* pytorch
* cudatoolkit
* psutil

**Paquetes Pip:**

* azureml-core
* azureml-pipeline-core
* azureml-telemetry
* azureml-defaults
* azureml-interpret
* azureml-explain-model
* azureml-automl-core
* azureml-automl-runtime
* azureml-train-automl-client
* azureml-train-automl-runtime
* inference-schema
* pytorch-transformers
* spacy
* en_core_web_sm
* py-cpuinfo

## <a name="azureml-automl-dnn-gpu"></a>AzureML-AutoML-DNN-GPU

**Canales de paquetes:**

* anaconda
* conda-forge
* pytorch

**Paquetes Conda:**

* Python
* numpy
* scikit-learn
* pandas
* fbprophet
* vacaciones
* setuptools-git
* pytorch
* cudatoolkit
* psutil

**Paquetes Pip:**

* azureml-core
* azureml-pipeline-core
* azureml-telemetry
* azureml-defaults
* azureml-interpret
* azureml-explain-model
* azureml-automl-core
* azureml-automl-runtime
* azureml-train-automl-client
* azureml-train-automl-runtime
* inference-schema
* horovod
* pytorch-transformers
* spacy
* en_core_web_sm
* py-cpuinfo

## <a name="azureml-automl-dnn-vision-gpu"></a>AzureML-AutoML-DNN-Vision-GPU

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-dataset-runtime
* azureml-contrib-dataset
* azureml-telemetry
* azureml-automl-core
* azureml-automl-runtime
* azureml-train-automl-client
* azureml-defaults
* azureml-interpret
* azureml-explain-model
* azureml-train-automl-runtime
* azureml-train-automl
* azureml-contrib-automl-dnn-vision

## <a name="azureml-automl-gpu"></a>AzureML-AutoML-GPU

**Canales de paquetes:**

* anaconda
* conda-forge
* pytorch

**Paquetes Conda:**

* Python
* numpy
* scikit-learn
* pandas
* fbprophet
* vacaciones
* setuptools-git
* psutil

**Paquetes Pip:**

* azureml-core
* azureml-pipeline-core
* azureml-telemetry
* azureml-defaults
* azureml-interpret
* azureml-explain-model
* azureml-automl-core
* azureml-automl-runtime
* azureml-train-automl-client
* azureml-train-automl-runtime
* inference-schema
* py-cpuinfo

## <a name="azureml-chainer-510-cpu"></a>AzureML-Chainer-5.1.0-CPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* chainer
* mpi4py

## <a name="azureml-chainer-510-gpu"></a>AzureML-Chainer-5.1.0-GPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* chainer
* cupy-cuda90
* mpi4py

## <a name="azureml-dask-cpu"></a>AzureML-Dask-CPU

**Canales de paquetes:**

* conda-forge
* pytorch
* parámetros predeterminados

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* adlfs
* azureml-core
* azureml-dataset-runtime
* dask[complete]
* dask-ml[complete]
* distributed
* fastparquet
* fsspec
* joblib
* jupyterlab
* lz4
* mpi4py
* notebook
* pyarrow

## <a name="azureml-dask-gpu"></a>AzureML-Dask-GPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python
* matplotlib

**Paquetes Pip:**

* azureml-defaults
* adlfs
* azureml-core
* dask[complete]
* dask-ml[complete]
* distributed
* fastparquet
* fsspec
* joblib
* jupyterlab
* lz4
* mpi4py
* notebook
* pyarrow

## <a name="azureml-hyperdrive-forecastdnn"></a>AzureML-Hyperdrive-ForecastDNN

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-pipeline-core
* azureml-telemetry
* azureml-defaults
* azureml-automl-core
* azureml-automl-runtime
* azureml-train-automl-client
* azureml-train-automl-runtime
* azureml-contrib-automl-dnn-forecasting

## <a name="azureml-minimal"></a>AzureML-Minimal

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults

## <a name="azureml-pyspark-mmlspark-015"></a>AzureML-PySpark-MmlSpark-0.15

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core

## <a name="azureml-pytorch-10-cpu"></a>AzureML-PyTorch-1.0-CPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod

## <a name="azureml-pytorch-10-gpu"></a>AzureML-PyTorch-1.0-GPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod

## <a name="azureml-pytorch-11-cpu"></a>AzureML-PyTorch-1.1-CPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-11-gpu"></a>AzureML-PyTorch-1.1-GPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-12-cpu"></a>AzureML-PyTorch-1.2-CPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-12-gpu"></a>AzureML-PyTorch-1.2-GPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-13-cpu"></a>AzureML-PyTorch-1.3-CPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-13-gpu"></a>AzureML-PyTorch-1.3-GPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-14-cpu"></a>AzureML-PyTorch-1.4-CPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-14-gpu"></a>AzureML-PyTorch-1.4-GPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-15-cpu"></a>AzureML-PyTorch-1.5-CPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-15-gpu"></a>AzureML-PyTorch-1.5-GPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-16-cpu"></a>AzureML-PyTorch-1.6-CPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-16-gpu"></a>AzureML-PyTorch-1.6-GPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-scikit-learn-0203"></a>AzureML-Scikit-learn-0.20.3

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* scikit-learn
* scipy
* joblib

## <a name="azureml-tensorflow-110-cpu"></a>AzureML-TensorFlow-1.10-CPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow
* horovod

## <a name="azureml-tensorflow-110-gpu"></a>AzureML-TensorFlow-1.10-GPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow-gpu
* horovod

## <a name="azureml-tensorflow-112-cpu"></a>AzureML-TensorFlow-1.12-CPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow
* horovod

## <a name="azureml-tensorflow-112-gpu"></a>AzureML-TensorFlow-1.12-GPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow-gpu
* horovod

## <a name="azureml-tensorflow-113-cpu"></a>AzureML-TensorFlow-1.13-CPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow
* horovod

## <a name="azureml-tensorflow-113-gpu"></a>AzureML-TensorFlow-1.13-GPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow-gpu
* horovod

## <a name="azureml-tensorflow-20-cpu"></a>AzureML-TensorFlow-2.0-CPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow
* horovod

## <a name="azureml-tensorflow-20-gpu"></a>AzureML-TensorFlow-2.0-GPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow-gpu
* horovod

## <a name="azureml-tensorflow-21-cpu"></a>AzureML-TensorFlow-2.1-CPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow
* horovod

## <a name="azureml-tensorflow-21-gpu"></a>AzureML-TensorFlow-2.1-GPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow-gpu
* horovod

## <a name="azureml-tensorflow-22-cpu"></a>AzureML-TensorFlow-2.2-CPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow
* horovod

## <a name="azureml-tensorflow-22-gpu"></a>AzureML-TensorFlow-2.2-GPU

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow-gpu
* horovod

## <a name="azureml-tutorial"></a>AzureML-Tutorial

**Canales de paquetes:**

* anaconda
* conda-forge

**Paquetes Conda:**

* Python
* Pandas
* numpy
* tqdm
* scikit-learn
* matplotlib

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* azureml-widgets
* azureml-pipeline-core
* azureml-pipeline-steps
* azureml-opendatasets
* azureml-automl-core
* azureml-automl-runtime
* azureml-train-automl-client
* azureml-train-automl-runtime
* azureml-train-automl
* azureml-train
* azureml-sdk
* azureml-interpret
* azureml-tensorboard
* azureml-mlflow
* mlflow
* sklearn-pandas

## <a name="azureml-vowpalwabbit-880"></a>AzureML-VowpalWabbit-8.8.0

**Canales de paquetes:**

* conda-forge

**Paquetes Conda:**

* Python

**Paquetes Pip:**

* azureml-core
* azureml-defaults
* azureml-dataset-runtime[fuse,pandas]
