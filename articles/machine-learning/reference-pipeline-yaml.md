---
title: 'Canalización de aprendizaje automático: YAML'
titleSuffix: Azure Machine Learning
description: Aprenda a definir una canalización de aprendizaje automático mediante un archivo YAML. Las definiciones de canalización YAML se usan con la extensión de aprendizaje automático de la CLI de Azure.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
ms.reviewer: larryfr
ms.author: nilsp
author: NilsPohlmann
ms.date: 07/31/2020
ms.custom: devx-track-python
ms.openlocfilehash: 079341fe1b22db83ded4b11941d8bdf809adfb8d
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87845542"
---
# <a name="define-machine-learning-pipelines-in-yaml"></a>Definición de canalizaciones de aprendizaje automático en YAML

Aprenda a definir las canalizaciones de aprendizaje automático en [YAML](https://yaml.org/). Al usar la extensión de aprendizaje automático de la CLI de Azure, muchos de los comandos relacionados con la canalización esperan un archivo YAML que define la canalización.

En la tabla siguiente se muestra lo que se admite y no se admite actualmente al definir una canalización en YAML:

| Tipo de paso | Se admite? |
| ----- | :-----: |
| PythonScriptStep | Sí |
| ParallelRunStep | Sí |
| AdlaStep | Sí |
| AzureBatchStep | Sí |
| DatabricksStep | Sí |
| DataTransferStep | Sí |
| AutoMLStep | No |
| HyperDriveStep | No |
| ModuleStep | Sí |
| MPIStep | No |
| EstimatorStep | No |

## <a name="pipeline-definition"></a>Definición de la canalización

Una definición de canalización utiliza las claves siguientes, que corresponden a la clase [Pipelines](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipeline.pipeline?view=azure-ml-py):

| Clave de YAML | Descripción |
| ----- | ----- |
| `name` | La descripción de la canalización. |
| `parameters` | Parámetro(s) para la canalización. |
| `data_reference` | Define cómo y dónde deben estar disponibles los datos en una ejecución. |
| `default_compute` | Destino de proceso predeterminado en el que se ejecutan todos los pasos de la canalización. |
| `steps` | Los pasos que se usan en la canalización. |

## <a name="parameters"></a>Parámetros

En la sección `parameters` se usan las claves siguientes, que corresponden a la clase [PipelineParameter](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelineparameter?view=azure-ml-py):

| Clave de YAML | Descripción |
| ---- | ---- |
| `type` | El tipo de valor del parámetro. Los tipos válidos son `string`, `int`, `float`, `bool` o `datapath`. |
| `default` | El valor predeterminado. |

Cada parámetro tiene su nombre. Por ejemplo, el siguiente fragmento de código YAML define tres parámetros llamados `NumIterationsParameter`, `DataPathParameter` y `NodeCountParameter`:

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        NumIterationsParameter:
            type: int
            default: 40
        DataPathParameter:
            type: datapath
            default:
                datastore: workspaceblobstore
                path_on_datastore: sample2.txt
        NodeCountParameter:
            type: int
            default: 4
```

## <a name="data-reference"></a>Referencia de datos

En la sección `data_references` se usan las siguientes claves, que corresponden a la clase [DataReference](https://docs.microsoft.com/python/api/azureml-core/azureml.data.data_reference.datareference?view=azure-ml-py):

| Clave de YAML | Descripción |
| ----- | ----- |
| `datastore` | El almacén de datos al que se hace referencia. |
| `path_on_datastore` | La ruta de acceso relativa del almacenamiento de seguridad para la referencia de datos. |

Cada referencia de datos está contenida en una clave. Por ejemplo, el siguiente fragmento de código YAML define una referencia de datos almacenada en la clave denominada `employee_data`:

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        PipelineParam1:
            type: int
            default: 3
    data_references:
        employee_data:
            datastore: adftestadla
            path_on_datastore: "adla_sample/sample_input.csv"
```

## <a name="steps"></a>Pasos

Los pasos definen un entorno computacional, junto con los archivos que se ejecutarán en el entorno. Para definir el tipo de un paso, use la clave `type`:

| Tipo de paso | Descripción |
| ----- | ----- |
| `AdlaStep` | Ejecuta un script U-SQL con Azure Data Lake Analytics. Corresponde a la clase [AdlaStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.adlastep?view=azure-ml-py). |
| `AzureBatchStep` | Ejecuta trabajos mediante Azure Batch. Corresponde a la clase [AzureBatchStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.azurebatchstep?view=azure-ml-py). |
| `DatabricsStep` | Agrega un bloc de notas de Databricks, un script de Python o un archivo JAR. Corresponde a la clase [DatabricksStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.databricksstep?view=azure-ml-py). |
| `DataTransferStep` | Transfiere datos entre opciones de almacenamiento. Corresponde a la clase [DataTransferStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.datatransferstep?view=azure-ml-py). |
| `PythonScriptStep` | Ejecuta un script de Python. Corresponde a la clase [PythonScriptStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.python_script_step.pythonscriptstep?view=azure-ml-py). |
| `ParallelRunStep` | Ejecuta un script de Python para procesar grandes cantidades de datos de forma asincrónica y en paralelo. Corresponde a la clase [ParallelRunStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.parallel_run_step.parallelrunstep?view=azure-ml-py). |

### <a name="adla-step"></a>Paso de ADLA

| Clave de YAML | Descripción |
| ----- | ----- |
| `script_name` | Nombre del script de U-SQL (relativo a `source_directory`). |
| `compute_target` | El destino de proceso de Azure Data Lake que se usará en este paso. |
| `parameters` | [Parámetros](#parameters) para la canalización. |
| `inputs` | Las entradas pueden ser [InputPortBinding](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.inputportbinding?view=azure-ml-py), [Reference](#data-reference), [PortDataReference](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.portdatareference?view=azure-ml-py), [PipelineData](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?view=azure-ml-py), [DataSet](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset%28class%29?view=azure-ml-py), [DatasetDefinition](https://docs.microsoft.com/python/api/azureml-core/azureml.data.dataset_definition.datasetdefinition?view=azure-ml-py)o [PipelineDataset](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedataset?view=azure-ml-py). |
| `outputs` | Las salidas pueden ser [PipelineData](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?view=azure-ml-py) o [OutputPortBinding](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.outputportbinding?view=azure-ml-py). |
| `source_directory` | Directorio que contiene el script, los ensamblados, etc. |
| `priority` | El valor de prioridad que se usará en el trabajo actual. |
| `params` | Diccionario de pares de nombre-valor. |
| `degree_of_parallelism` | El grado de paralelismo que se va a usar en este trabajo. |
| `runtime_version` | La versión en tiempo de ejecución del motor de Data Lake Analytics. |
| `allow_reuse` | Determina si el paso debe volver a usar los resultados anteriores cuando se ejecute de nuevo con la misma configuración. |

El ejemplo siguiente contiene una definición de paso de ADLA:

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        PipelineParam1:
            type: int
            default: 3
    data_references:
        employee_data:
            datastore: adftestadla
            path_on_datastore: "adla_sample/sample_input.csv"
    default_compute: adlacomp
    steps:
        Step1:
            runconfig: "D:\\Yaml\\default_runconfig.yml"
            parameters:
                NUM_ITERATIONS_2:
                    source: PipelineParam1
                NUM_ITERATIONS_1: 7
            type: "AdlaStep"
            name: "MyAdlaStep"
            script_name: "sample_script.usql"
            source_directory: "D:\\scripts\\Adla"
            inputs:
                employee_data:
                    source: employee_data
            outputs:
                OutputData:
                    destination: Output4
                    datastore: adftestadla
                    bind_mode: mount
```

### <a name="azure-batch-step"></a>Paso de Azure Batch

| Clave de YAML | Descripción |
| ----- | ----- |
| `compute_target` | El destino de proceso de Azure Batch que se usará en este paso. |
| `inputs` | Las entradas pueden ser [InputPortBinding](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.inputportbinding?view=azure-ml-py), [Reference](#data-reference), [PortDataReference](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.portdatareference?view=azure-ml-py), [PipelineData](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?view=azure-ml-py), [DataSet](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset%28class%29?view=azure-ml-py), [DatasetDefinition](https://docs.microsoft.com/python/api/azureml-core/azureml.data.dataset_definition.datasetdefinition?view=azure-ml-py)o [PipelineDataset](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedataset?view=azure-ml-py). |
| `outputs` | Las salidas pueden ser [PipelineData](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?view=azure-ml-py) o [OutputPortBinding](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.outputportbinding?view=azure-ml-py). |
| `source_directory` | Directorio que contiene los binarios del módulo, los archivos ejecutables, los ensamblados, etc. |
| `executable` | Nombre del comando o ejecutable que se ejecutará como parte de este trabajo. |
| `create_pool` | Marca booleana que indica si se debe crear el grupo antes de ejecutar el trabajo. |
| `delete_batch_job_after_finish` | Marca booleana que indica si se debe eliminar el trabajo de la cuenta de Batch una vez finalizado. |
| `delete_batch_pool_after_finish` | Marca booleana que indica si se debe eliminar el grupo después de que finalice el trabajo. |
| `is_positive_exit_code_failure` | Marca booleana que indica si la tarea finaliza con un código positivo en caso de que el trabajo genere un error. |
| `vm_image_urn` | Si `create_pool` es `True` y la máquina virtual usa `VirtualMachineConfiguration`. |
| `pool_id` | El identificador del grupo en el que se ejecutará el trabajo. |
| `allow_reuse` | Determina si el paso debe volver a usar los resultados anteriores cuando se ejecute de nuevo con la misma configuración. |

El ejemplo siguiente contiene una definición de paso de Azure Batch:

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        PipelineParam1:
            type: int
            default: 3
    data_references:
        input:
            datastore: workspaceblobstore
            path_on_datastore: "input.txt"
    default_compute: testbatch
    steps:
        Step1:
            runconfig: "D:\\Yaml\\default_runconfig.yml"
            parameters:
                NUM_ITERATIONS_2:
                    source: PipelineParam1
                NUM_ITERATIONS_1: 7
            type: "AzureBatchStep"
            name: "MyAzureBatchStep"
            pool_id: "MyPoolName"
            create_pool: true
            executable: "azurebatch.cmd"
            source_directory: "D:\\scripts\\AureBatch"
            allow_reuse: false
            inputs:
                input:
                    source: input
            outputs:
                output:
                    destination: output
                    datastore: workspaceblobstore
```

### <a name="databricks-step"></a>Paso de Databricks

| Clave de YAML | Descripción |
| ----- | ----- |
| `compute_target` | El destino de proceso de Azure Databricks que se usará en este paso. |
| `inputs` | Las entradas pueden ser [InputPortBinding](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.inputportbinding?view=azure-ml-py), [Reference](#data-reference), [PortDataReference](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.portdatareference?view=azure-ml-py), [PipelineData](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?view=azure-ml-py), [DataSet](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset%28class%29?view=azure-ml-py), [DatasetDefinition](https://docs.microsoft.com/python/api/azureml-core/azureml.data.dataset_definition.datasetdefinition?view=azure-ml-py)o [PipelineDataset](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedataset?view=azure-ml-py). |
| `outputs` | Las salidas pueden ser [PipelineData](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?view=azure-ml-py) o [OutputPortBinding](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.outputportbinding?view=azure-ml-py). |
| `run_name` | El nombre en Databricks de esta ejecución. |
| `source_directory` | Directorio que contiene el script y otros archivos. |
| `num_workers` | El número estático de trabajos para el clúster de ejecución de Databricks. |
| `runconfig` | La ruta de acceso a un archivo `.runconfig`. Este archivo es una representación YAML de la clase [RunConfiguration](https://docs.microsoft.com/python/api/azureml-core/azureml.core.runconfiguration?view=azure-ml-py). Para más información sobre la estructura de este archivo, consulte [runconfigschema.json](https://github.com/microsoft/MLOps/blob/b4bdcf8c369d188e83f40be8b748b49821f71cf2/infra-as-code/runconfigschema.json). |
| `allow_reuse` | Determina si el paso debe volver a usar los resultados anteriores cuando se ejecute de nuevo con la misma configuración. |

El ejemplo siguiente contiene un paso de Databricks:

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        PipelineParam1:
            type: int
            default: 3
    data_references:
        adls_test_data:
            datastore: adftestadla
            path_on_datastore: "testdata"
        blob_test_data:
            datastore: workspaceblobstore
            path_on_datastore: "dbtest"
    default_compute: mydatabricks
    steps:
        Step1:
            runconfig: "D:\\Yaml\\default_runconfig.yml"
            parameters:
                NUM_ITERATIONS_2:
                    source: PipelineParam1
                NUM_ITERATIONS_1: 7
            type: "DatabricksStep"
            name: "MyDatabrickStep"
            run_name: "DatabricksRun"
            python_script_name: "train-db-local.py"
            source_directory: "D:\\scripts\\Databricks"
            num_workers: 1
            allow_reuse: true
            inputs:
                blob_test_data:
                    source: blob_test_data
            outputs:
                OutputData:
                    destination: Output4
                    datastore: workspaceblobstore
                    bind_mode: mount
```

### <a name="data-transfer-step"></a>Paso de transferencia de datos

| Clave de YAML | Descripción |
| ----- | ----- |
| `compute_target` | El destino de proceso de Azure Data Factory que se usará en este paso. |
| `source_data_reference` | Conexión de entrada que actúa como origen de las operaciones de transferencia de datos. Los valores admitidos son [InputPortBinding](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.inputportbinding?view=azure-ml-py), [Reference](#data-reference), [PortDataReference](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.portdatareference?view=azure-ml-py), [PipelineData](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?view=azure-ml-py), [DataSet](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset%28class%29?view=azure-ml-py), [DatasetDefinition](https://docs.microsoft.com/python/api/azureml-core/azureml.data.dataset_definition.datasetdefinition?view=azure-ml-py)o [PipelineDataset](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedataset?view=azure-ml-py). |
| `destination_data_reference` | Conexión de entrada que actúa como destino de las operaciones de transferencia de datos. Los valores admitidos son [PipelineData](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?view=azure-ml-py) y [OutputPortBinding](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.outputportbinding?view=azure-ml-py). |
| `allow_reuse` | Determina si el paso debe volver a usar los resultados anteriores cuando se ejecute de nuevo con la misma configuración. |

El ejemplo siguiente contiene un paso de transferencia de datos:

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        PipelineParam1:
            type: int
            default: 3
    data_references:
        adls_test_data:
            datastore: adftestadla
            path_on_datastore: "testdata"
        blob_test_data:
            datastore: workspaceblobstore
            path_on_datastore: "testdata"
    default_compute: adftest
    steps:
        Step1:
            runconfig: "D:\\Yaml\\default_runconfig.yml"
            parameters:
                NUM_ITERATIONS_2:
                    source: PipelineParam1
                NUM_ITERATIONS_1: 7
            type: "DataTransferStep"
            name: "MyDataTransferStep"
            adla_compute_name: adftest
            source_data_reference:
                adls_test_data:
                    source: adls_test_data
            destination_data_reference:
                blob_test_data:
                    source: blob_test_data
```

### <a name="python-script-step"></a>Paso de script de Python

| Clave de YAML | Descripción |
| ----- | ----- |
| `inputs` | Las entradas pueden ser [InputPortBinding](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.inputportbinding?view=azure-ml-py), [Reference](#data-reference), [PortDataReference](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.portdatareference?view=azure-ml-py), [PipelineData](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?view=azure-ml-py), [DataSet](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset%28class%29?view=azure-ml-py), [DatasetDefinition](https://docs.microsoft.com/python/api/azureml-core/azureml.data.dataset_definition.datasetdefinition?view=azure-ml-py)o [PipelineDataset](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedataset?view=azure-ml-py). |
| `outputs` | Las salidas pueden ser [PipelineData](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?view=azure-ml-py) o [OutputPortBinding](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.outputportbinding?view=azure-ml-py). |
| `script_name` | El nombre del script de Python (relativo a `source_directory`). |
| `source_directory` | Directorio que contiene el script, el entorno de Conda, etc. |
| `runconfig` | La ruta de acceso a un archivo `.runconfig`. Este archivo es una representación YAML de la clase [RunConfiguration](https://docs.microsoft.com/python/api/azureml-core/azureml.core.runconfiguration?view=azure-ml-py). Para más información sobre la estructura de este archivo, consulte [runconfig.json](https://github.com/microsoft/MLOps/blob/b4bdcf8c369d188e83f40be8b748b49821f71cf2/infra-as-code/runconfigschema.json). |
| `allow_reuse` | Determina si el paso debe volver a usar los resultados anteriores cuando se ejecute de nuevo con la misma configuración. |

El ejemplo siguiente contiene un paso de script de Python:

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        PipelineParam1:
            type: int
            default: 3
    data_references:
        DataReference1:
            datastore: workspaceblobstore
            path_on_datastore: testfolder/sample.txt
    default_compute: cpu-cluster
    steps:
        Step1:
            runconfig: "D:\\Yaml\\default_runconfig.yml"
            parameters:
                NUM_ITERATIONS_2:
                    source: PipelineParam1
                NUM_ITERATIONS_1: 7
            type: "PythonScriptStep"
            name: "MyPythonScriptStep"
            script_name: "train.py"
            allow_reuse: True
            source_directory: "D:\\scripts\\PythonScript"
            inputs:
                InputData:
                    source: DataReference1
            outputs:
                OutputData:
                    destination: Output4
                    datastore: workspaceblobstore
                    bind_mode: mount
```

### <a name="parallel-run-step"></a>Paso de ejecución en paralelo

| Clave de YAML | Descripción |
| ----- | ----- |
| `inputs` | Las entradas pueden ser [Dataset](https://docs.microsoft.com/python/api/azureml-core/azureml.core.dataset%28class%29?view=azure-ml-py), [DatasetDefinition](https://docs.microsoft.com/python/api/azureml-core/azureml.data.dataset_definition.datasetdefinition?view=azure-ml-py) o [PipelineDataset](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedataset?view=azure-ml-py). |
| `outputs` | Las salidas pueden ser [PipelineData](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?view=azure-ml-py) o [OutputPortBinding](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.outputportbinding?view=azure-ml-py). |
| `script_name` | El nombre del script de Python (relativo a `source_directory`). |
| `source_directory` | Directorio que contiene el script, el entorno de Conda, etc. |
| `parallel_run_config` | La ruta de acceso a un archivo `parallel_run_config.yml`. Este archivo es una representación YAML de la clase [ParallelRunConfig](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.parallelrunconfig?view=azure-ml-py). |
| `allow_reuse` | Determina si el paso debe volver a usar los resultados anteriores cuando se ejecute de nuevo con la misma configuración. |

El ejemplo siguiente contiene un paso de ejecución en paralelo:

```yaml
pipeline:
    description: SamplePipelineFromYaml
    default_compute: cpu-cluster
    data_references:
        MyMinistInput:
            dataset_name: mnist_sample_data
    parameters:
        PipelineParamTimeout:
            type: int
            default: 600
    steps:        
        Step1:
            parallel_run_config: "yaml/parallel_run_config.yml"
            type: "ParallelRunStep"
            name: "parallel-run-step-1"
            allow_reuse: True
            arguments:
            - "--progress_update_timeout"
            - parameter:timeout_parameter
            - "--side_input"
            - side_input:SideInputData
            parameters:
                timeout_parameter:
                    source: PipelineParamTimeout
            inputs:
                InputData:
                    source: MyMinistInput
            side_inputs:
                SideInputData:
                    source: Output4
                    bind_mode: mount
            outputs:
                OutputDataStep2:
                    destination: Output5
                    datastore: workspaceblobstore
                    bind_mode: mount
```

### <a name="pipeline-with-multiple-steps"></a>Canalización con varios pasos 

| Clave de YAML | Descripción |
| ----- | ----- |
| `steps` | Secuencia de una o más definiciones de PipelineStep. Tenga en cuenta que las claves de `destination` de `outputs` de un paso se convierten en las claves de `source` de `inputs` del siguiente paso.| 

```yaml
pipeline:
    name: SamplePipelineFromYAML
    description: Sample multistep YAML pipeline
    data_references:
        TitanicDS:
            dataset_name: 'titanic_ds'
            bind_mode: download
    default_compute: cpu-cluster
    steps:
        Dataprep:
            type: "PythonScriptStep"
            name: "DataPrep Step"
            compute: cpu-cluster
            runconfig: ".\\default_runconfig.yml"
            script_name: "prep.py"
            arguments:
            - '--train_path'
            - output:train_path
            - '--test_path'
            - output:test_path
            allow_reuse: True
            inputs:
                titanic_ds:
                    source: TitanicDS
                    bind_mode: download
            outputs:
                train_path:
                    destination: train_csv
                    datastore: workspaceblobstore
                test_path:
                    destination: test_csv
        Training:
            type: "PythonScriptStep"
            name: "Training Step"
            compute: cpu-cluster
            runconfig: ".\\default_runconfig.yml"
            script_name: "train.py"
            arguments:
            - "--train_path"
            - input:train_path
            - "--test_path"
            - input:test_path
            inputs:
                train_path:
                    source: train_csv
                    bind_mode: download
                test_path:
                    source: test_csv
                    bind_mode: download

```

## <a name="schedules"></a>Programaciones

Al definir la programación de una canalización, se puede desencadenar por el almacén de datos o de forma periódica según un intervalo de tiempo. Estas son las claves que se usan para definir una programación:

| Clave de YAML | Descripción |
| ----- | ----- |
| `description` | Una descripción de la programación. |
| `recurrence` | Contiene la configuración de periodicidad, si la programación es periódica. |
| `pipeline_parameters` | Cualquier parámetro necesario para la canalización. |
| `wait_for_provisioning` | Indica si se va a esperar a que se complete el aprovisionamiento de la programación. |
| `wait_timeout` | El número de segundos que se va a esperar antes de que se agote el tiempo de espera. |
| `datastore_name` | El almacén de objetos que se va a supervisar para los blobs modificados o agregados. |
| `polling_interval` | La cantidad de tiempo, en minutos, entre el sondeo de blobs modificados o agregados. Valor predeterminado: 5 minutos. Solo se admite para las programaciones del almacén de datos. |
| `data_path_parameter_name` | El nombre del parámetro de canalización de la ruta de acceso de datos que se va a establecer con la ruta de acceso del blob modificado. Solo se admite para las programaciones del almacén de datos. |
| `continue_on_step_failure` | Indica si se continúa la ejecución del resto de pasos en el elemento PipelineRun enviado si se produce un error en un paso. Si se proporciona, invalidará el valor `continue_on_step_failure` de la canalización.
| `path_on_datastore` | Opcional. La ruta de acceso del almacén de datos que se supervisará en los blobs modificados o agregados. La ruta de acceso está en el contenedor del almacén de datos, por lo que la ruta de acceso real que la programación supervisa es container/`path_on_datastore`. Si no hay ninguna, se supervisa el contenedor del almacén de datos. Las adiciones y modificaciones realizadas en una subcarpeta de `path_on_datastore` no se supervisan. Solo se admite para las programaciones del almacén de datos. |

El ejemplo siguiente contiene la definición de una programación desencadenada por un almacén de datos:

```yaml
Schedule: 
      description: "Test create with datastore" 
      recurrence: ~ 
      pipeline_parameters: {} 
      wait_for_provisioning: True 
      wait_timeout: 3600 
      datastore_name: "workspaceblobstore" 
      polling_interval: 5 
      data_path_parameter_name: "input_data" 
      continue_on_step_failure: None 
      path_on_datastore: "file/path" 
```

Al definir una **programación periódica**, use las claves que hay en `recurrence`:

| Clave de YAML | Descripción |
| ----- | ----- |
| `frequency` | Frecuencia con la que se repite la programación. Los valores válidos son `"Minute"`, `"Hour"`, `"Day"`, `"Week"` o `"Month"`. |
| `interval` | Frecuencia con la que se activa la programación. El valor entero es el número de unidades de tiempo que hay que esperar hasta que se vuelve a desencadenar la programación. |
| `start_time` | La hora de inicio de la programación. El formato de cadena del valor es `YYYY-MM-DDThh:mm:ss`. Si no se proporciona ninguna hora de inicio, la primera carga de trabajo se ejecuta de forma instantánea y las cargas de trabajo futuras se ejecutan en función de la programación. Si la hora de inicio pertenece al pasado, la primera carga de trabajo se ejecuta en el siguiente tiempo de ejecución calculado. |
| `time_zone` | La zona horaria correspondiente a la hora de inicio. Si no se proporciona ninguna zona horaria, se usa la hora UTC. |
| `hours` | Si `frequency` es `"Day"` o `"Week"`, puede especificar uno o varios enteros de 0 a 23, separados por comas, como las horas del día en que se debe ejecutar la canalización. Solo se pueden usar `time_of_day` o `hours` y `minutes`. |
| `minutes` | Si `frequency` es `"Day"` o `"Week"`, puede especificar uno o varios enteros de 0 a 59, separados por comas, como los minutos de la hora en que se debe ejecutar la canalización. Solo se pueden usar `time_of_day` o `hours` y `minutes`. |
| `time_of_day` | Si `frequency` es `"Day"` o `"Week"`, puede especificar una hora del día para que se ejecute la programación. El formato de cadena del valor es `hh:mm`. Solo se pueden usar `time_of_day` o `hours` y `minutes`. |
| `week_days` | Si `frequency` es `"Week"`, puede especificar uno o varios días, separados por comas, en que se debe ejecutar la programación. Los valores válidos son `"Monday"`, `"Tuesday"`, `"Wednesday"`, `"Thursday"`, `"Friday"`, `"Saturday"` y `"Sunday"`. |

El ejemplo siguiente contiene la definición de una programación periódica:

```yaml
Schedule: 
    description: "Test create with recurrence" 
    recurrence: 
        frequency: Week # Can be "Minute", "Hour", "Day", "Week", or "Month". 
        interval: 1 # how often fires 
        start_time: 2019-06-07T10:50:00 
        time_zone: UTC 
        hours: 
        - 1 
        minutes: 
        - 0 
        time_of_day: null 
        week_days: 
        - Friday 
    pipeline_parameters: 
        'a': 1 
    wait_for_provisioning: True 
    wait_timeout: 3600 
    datastore_name: ~ 
    polling_interval: ~ 
    data_path_parameter_name: ~ 
    continue_on_step_failure: None 
    path_on_datastore: ~ 
```

## <a name="next-steps"></a>Pasos siguientes

Aprenda a [usar la extensión de la CLI para Azure Machine Learning](reference-azure-machine-learning-cli.md).
