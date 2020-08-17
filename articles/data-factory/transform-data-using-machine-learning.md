---
title: Creación de canalizaciones de datos predictivos
description: 'Obtenga información sobre cómo crear una canalización predictiva mediante el uso de Azure Machine Learning Studio (clásico): actividad de ejecución de lotes en Azure Data Factory.'
author: nabhishek
ms.author: abnarain
manager: shwang
services: data-factory
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 07/16/2020
ms.openlocfilehash: dabb7b8cd8023fe88a8c8d6dc507a09623bd11dd
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86537687"
---
# <a name="create-a-predictive-pipeline-using-azure-machine-learning-studio-classic-and-azure-data-factory"></a>Creación de canalizaciones predictivas con Azure Machine Learning Studio (clásico) y Azure Data Factory

> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](v1/data-factory-azure-ml-batch-execution-activity.md)
> * [Versión actual](transform-data-using-machine-learning.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

[Azure Machine Learning Studio (clásico)](https://azure.microsoft.com/documentation/services/machine-learning/) permite compilar, probar e implementar soluciones de análisis predictivo. Desde una perspectiva general, esto se realiza en tres pasos:

1. **Crear un experimento de entrenamiento**. Este paso se lleva a cabo utilizando Azure Machine Learning Studio (clásico). Azure Machine Learning Studio (clásico) es un entorno de desarrollo visual de colaboración que se emplea para entrenar y probar un modelo de análisis predictivo con datos de entrenamiento.
2. **Convertirlo en un experimento predictivo**. Una vez que el modelo se ha entrenado con datos existentes y está listo para usarlo para puntuar nuevos datos, debe preparar y simplificar el experimento para la puntuación.
3. **Implementarlo como un servicio web**. Puede publicar el experimento de puntuación como un servicio web de Azure. Los usuarios pueden enviar datos al modelo a través de este punto de conexión de servicio web y recibir las predicciones de resultado para el modelo.

### <a name="data-factory-and-azure-machine-learning-studio-classic-together"></a>Data Factory y Azure Machine Learning Studio (clásico) juntos
Azure Data Factory permite crear fácilmente canalizaciones que usan un servicio web de [Azure Machine Learning Studio (clásico)](https://azure.microsoft.com/documentation/services/machine-learning) publicado para realizar análisis predictivos. Con la **actividad de ejecución de lotes** en una canalización de Azure Data Factory, puede invocar un servicio web de Azure Machine Learning Studio (clásico) para realizar predicciones sobre los datos en el lote.

Pasado algún tiempo, los modelos predictivos en los experimentos de puntuación de Azure Machine Learning Studio (clásico) tienen que volver a entrenarse con nuevos conjuntos de datos de entrada. Puede volver a entrenar un modelo de una canalización de Data Factory realizando los pasos siguientes:

1. Publicar el experimento de entrenamiento (experimento no predictivo) como un servicio web. Tiene que llevar a cabo este paso en Azure Machine Learning Studio (clásico), tal como hizo para exponer el experimento predictivo como un servicio web en el escenario anterior.
2. Use la actividad de ejecución de lotes de Azure Machine Learning Studio (clásico) para invocar el servicio web para el experimento de entrenamiento. Básicamente, puede emplear la actividad de ejecución de lotes de Azure Machine Learning Studio (clásico) para invocar el servicio web de entrenamiento y el servicio web de puntuación.

Cuando haya terminado el reentrenamiento, actualice el servicio web de puntuación (experimento predictivo expuesto como servicio web) con el modelo recién entrenado utilizando la **actividad de actualización de recurso de Azure Machine Learning Studio (clásico)** . Para obtener más información, consulte el artículo [Updating models using Update Resource Activity](update-machine-learning-models.md) (Actualización de modelos mediante la actividad de recursos de actualización).

## <a name="azure-machine-learning-studio-classic-linked-service"></a>Servicio vinculado de Azure Machine Learning Studio (clásico)

Un servicio vinculado de **Azure Machine Learning Studio (clásico)** se crea para vincular un servicio web de Azure Machine Learning Studio (clásico) a Azure Data Factory. El servicio vinculado lo usan la actividad de ejecución de Batch de Azure Machine Learning Studio (clásico) y la [actividad de actualización de recurso](update-machine-learning-models.md).

```JSON
{
    "type" : "linkedServices",
    "name": "AzureMLLinkedService",
    "properties": {
        "type": "AzureML",
        "typeProperties": {
            "mlEndpoint": "URL to Azure ML Predictive Web Service",
            "apiKey": {
                "type": "SecureString",
                "value": "api key"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

Vea en el artículo [Compute linked services](compute-linked-services.md) (Servicios vinculados de proceso) una descripción de las propiedades en la definición de JSON.

Azure Machine Learning Studio (clásico) admite los servicios web clásicos y los nuevos servicios web para el experimento predictivo. Puede elegir el servicio más adecuado desde Data Factory. Para obtener la información necesaria para crear el servicio vinculado de Azure Machine Learning Studio (clásico), vaya a https://services.azureml.net, donde se indican todos los servicios web (nuevos) y los servicios web clásicos. Seleccione el servicio web al que le gustaría tener acceso y haga clic en la página **Consumir**. Copie **Clave principal** en la propiedad **apiKey** y **Batch Requests** (Solicitudes por lotes) en la propiedad **mlEndpoint**.

![Servicios web de Azure Machine Learning Studio (clásico)](./media/transform-data-using-machine-learning/web-services.png)

## <a name="azure-machine-learning-studio-classic-batch-execution-activity"></a>Actividad de ejecución de lotes de Azure Machine Learning Studio (clásico)

El siguiente fragmento JSON define una actividad de ejecución de lotes de Azure Machine Learning Studio (clásico). La definición de actividad tiene una referencia al servicio vinculado de Azure Machine Learning Studio (clásico) que ha creado anteriormente.

```JSON
{
    "name": "AzureMLExecutionActivityTemplate",
    "description": "description",
    "type": "AzureMLBatchExecution",
    "linkedServiceName": {
        "referenceName": "AzureMLLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "webServiceInputs": {
            "<web service input name 1>": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService1",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"path1"
            },
            "<web service input name 2>": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService1",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"path2"
            }
        },
        "webServiceOutputs": {
            "<web service output name 1>": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService2",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"path3"
            },
            "<web service output name 2>": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService2",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"path4"
            }
        },
        "globalParameters": {
            "<Parameter 1 Name>": "<parameter value>",
            "<parameter 2 name>": "<parameter 2 value>"
        }
    }
}
```

| Propiedad          | Descripción                              | Obligatorio |
| :---------------- | :--------------------------------------- | :------- |
| name              | Nombre de la actividad en la canalización     | Sí      |
| description       | Texto que describe para qué se usa la actividad.  | No       |
| type              | Para la actividad de U-SQL de Data Lake Analytics, el tipo de actividad es **AzureMLBatchExecution**. | Sí      |
| linkedServiceName | Servicios vinculados al servicio vinculado de Azure Machine Learning Studio (clásico). Para obtener más información sobre este servicio vinculado, vea el artículo [Compute linked services](compute-linked-services.md) (Servicios vinculados de procesos). | Sí      |
| webServiceInputs  | Pares clave-valor que asignan los nombres de entradas del servicio web de Azure Machine Learning Studio (clásico). La clave debe coincidir con los parámetros de entrada definidos en el servicio web publicado de Azure Machine Learning Studio (clásico). El valor es un par de propiedades FilePath y servicios vinculados de Azure Storage que especifica las ubicaciones del blob de entrada. | No       |
| webServiceOutputs | Pares clave-valor que asignan los nombres de salidas del servicio web de Azure Machine Learning Studio (clásico). La clave debe coincidir con los parámetros de salida definidos en el servicio web publicado de Azure Machine Learning Studio (clásico). El valor es un par de propiedades FilePath y servicios vinculados de Azure Storage que especifica las ubicaciones del blob de salida. | No       |
| globalParameters  | Pares clave-valor que se pasan al punto de conexión del servicio de ejecución por lotes de Azure Machine Learning Studio (clásico). Las claves tienen que coincidir con los nombres de los parámetros del servicio web definidos en el servicio web publicado de Azure Machine Learning Studio (clásico). Los valores se pasan en la propiedad GlobalParameters de la solicitud de ejecución por lotes de Azure Machine Learning Studio (clásico) | No       |

### <a name="scenario-1-experiments-using-web-service-inputsoutputs-that-refer-to-data-in-azure-blob-storage"></a>Escenario 1: experimentos mediante entradas y salidas de servicios web que hacen referencia a datos de Azure Blob Storage

En este escenario, el servicio web de Azure Machine Learning Studio (clásico) realiza predicciones mediante datos de un archivo de un almacenamiento de blobs de Azure y almacena los resultados de predicción en el almacenamiento de blobs. El siguiente JSON define una canalización de Data Factory con una actividad AzureMLBatchExecution. Se hace referencia a los datos de entrada y salida en Azure Blob Storage mediante un par LinkedName y FilePath. En el servicio vinculado de ejemplo las entradas y salidas son diferentes, puede usar servicios vinculados diferentes para cada una de las entradas o salidas, de modo que Data Factory pueda seleccionar los archivos correctos y enviarlos al servicio web de Azure Machine Learning Studio (clásico).

> [!IMPORTANT]
> En el experimento de Azure Machine Learning Studio (clásico), los puertos de entrada y salida del servicio web y los parámetros globales tienen nombres predeterminados ("input1", "input2") que se pueden personalizar. Los nombres que se utilizan para la configuración de globalParameters, webServiceOutputs y webServiceInputs deben coincidir exactamente con los de los experimentos. Puede ver la carga útil de la solicitud de ejemplo en la página de ayuda de ejecución por lotes del punto de conexión de Azure Machine Learning Studio (clásico) para comprobar la asignación esperada.


```JSON
{
    "name": "AzureMLExecutionActivityTemplate",
    "description": "description",
    "type": "AzureMLBatchExecution",
    "linkedServiceName": {
        "referenceName": "AzureMLLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "webServiceInputs": {
            "input1": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService1",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"amltest/input/in1.csv"
            },
            "input2": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService1",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"amltest/input/in2.csv"
            }
        },
        "webServiceOutputs": {
            "outputName1": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService2",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"amltest2/output/out1.csv"
            },
            "outputName2": {
                "LinkedServiceName":{
                    "referenceName": "AzureStorageLinkedService2",
                    "type": "LinkedServiceReference"
                },
                "FilePath":"amltest2/output/out2.csv"
            }
        }
    }
}
```
### <a name="scenario-2-experiments-using-readerwriter-modules-to-refer-to-data-in-various-storages"></a>Escenario 2: experimentos mediante módulos lector y escritor para hacer referencia a datos de varios almacenamientos
Otro escenario común al crear experimentos de Microsoft Azure Machine Learning Studio (clásico) es usar módulos Importar datos y Exportar datos. El módulo Importar datos se usa para cargar datos en un experimento y el módulo Exportar datos se usa para guardar los datos de los experimentos. Para obtener más información sobre los módulos Importar datos y Exportar datos, vea los temas [Importar datos](https://msdn.microsoft.com/library/azure/dn905997.aspx) y [Exportar datos](https://msdn.microsoft.com/library/azure/dn905984.aspx) en MSDN Library.

Al usar los módulos Importar datos y Exportar datos, es recomendable emplear un parámetro de servicio web para cada propiedad de estos módulos. Estos parámetros web permiten configurar los valores en tiempo de ejecución. Por ejemplo, puede crear un experimento con un módulo de importación de datos que usa una base de datos de Azure SQL: XXX.database.windows.net. Una vez implementado el servicio web, quiere habilitar los consumidores del servicio web con el fin de especificar otro servidor SQL lógico denominado `YYY.database.windows.net`. Puede usar un parámetro de servicio web para permitir que se configure este valor.

> [!NOTE]
> Las entradas y salidas de servicio web son diferentes de los parámetros de servicio web. En el primer escenario, ha visto cómo pueden especificarse una entrada y una salida para un servicio web de Azure Machine Learning Studio (clásico). En este escenario, se pasan parámetros para un servicio web que corresponden a las propiedades de los módulos Importar datos y Exportar datos.

Echemos un vistazo a un escenario de uso de parámetros de servicio web. Tiene un servicio web de Azure Machine Learning Studio (clásico) implementado que usa un módulo lector para leer datos de uno de los orígenes de datos compatibles con Azure Machine Learning Studio (clásico) (por ejemplo: Azure SQL Database). Después de realizar la ejecución de lotes, los resultados se escriben con un módulo escritor (Azure SQL Database).  No hay entradas ni salidas de servicio web definidas en los experimentos. En este caso, se recomienda que configure los parámetros de servicio web pertinentes para los módulos lector y escritor. De esta forma, se podrán configurar los módulos lector y escritor cuando se use la actividad AzureMLBatchExecution. Los parámetros de servicio web se especifican en la sección **globalParameters** de la actividad JSON como se indica a continuación.

```JSON
"typeProperties": {
    "globalParameters": {
        "Database server name": "<myserver>.database.windows.net",
        "Database name": "<database>",
        "Server user account name": "<user name>",
        "Server user account password": "<password>"
    }
}
```

> [!NOTE]
> Los parámetros de servicio web distinguen entre mayúsculas y minúsculas para garantizar que los nombres que especifica en JSON de actividad coinciden con los que muestra el servicio web.

Cuando haya terminado el reentrenamiento, actualice el servicio web de puntuación (experimento predictivo expuesto como servicio web) con el modelo recién entrenado utilizando la **actividad de actualización de recurso de Azure Machine Learning Studio (clásico)** . Para obtener más información, consulte el artículo [Updating models using Update Resource Activity](update-machine-learning-models.md) (Actualización de modelos mediante la actividad de recursos de actualización).

## <a name="next-steps"></a>Pasos siguientes
Vea los siguientes artículos, en los que se explica cómo transformar datos de otras maneras:

* [Actividad de U-SQL](transform-data-using-data-lake-analytics.md)
* [Actividad de Hive](transform-data-using-hadoop-hive.md)
* [Actividad de Pig](transform-data-using-hadoop-pig.md)
* [Actividad de MapReduce](transform-data-using-hadoop-map-reduce.md)
* [Actividad de streaming de Hadoop](transform-data-using-hadoop-streaming.md)
* [Actividad de Spark](transform-data-using-spark.md)
* [Actividad personalizada de .NET](transform-data-using-dotnet-custom-activity.md)
* [Actividad de procedimiento almacenado](transform-data-using-stored-procedure.md)
