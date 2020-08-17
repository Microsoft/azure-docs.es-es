---
title: Actualización de modelos de Azure Machine Learning Studio (clásico) mediante Azure Data Factory
description: En este artículo se describe cómo crear canalizaciones predictivas con Azure Data Factory y Azure Machine Learning Studio (clásico)
services: data-factory
documentationcenter: ''
author: djpmsft
ms.author: daperlov
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 07/16/2020
ms.openlocfilehash: 83950c2d3c5439886ff787b69d9da4d0c214de31
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87092548"
---
# <a name="update-azure-machine-learning-studio-classic-models-by-using-update-resource-activity"></a>Actualización de los modelos de Azure Machine Learning Studio (clásico) con la actividad de actualización de recursos

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Este artículo complementa el artículo principal de integración de Azure Data Factory - Azure Machine Learning Studio (clásico): [Creación de canalizaciones predictivas con Azure Machine Learning Studio (clásico) y Azure Data Factory](transform-data-using-machine-learning.md). Si aún no lo ha hecho, revise el artículo principal antes de leer este artículo.

## <a name="overview"></a>Información general
Como parte del proceso de operacionalización de modelos de Azure Machine Learning Studio (clásico), el modelo se debe entrenar y guardar. Posteriormente, podrá usarlo para crear un servicio web predictivo. A continuación, el servicio web se puede consumir en sitios web, paneles y aplicaciones móviles.

Normalmente, los modelos que se crean con Azure Machine Learning Studio (clásico) no son estáticos. Cuando hay nuevos datos disponibles o cuando el consumidor de la API tiene sus propios datos, el modelo debe volver a entrenarse. 

El reentrenamiento puede producirse con frecuencia. Con las actividades de ejecución por lotes y de actualización de recursos, puede operacionalizar el modelo de Azure Machine Learning Studio (clásico) entrenando de nuevo y actualizando el servicio web predictivo con Data Factory.

La siguiente imagen muestra la relación entre los servicios web de entrenamiento y predictivo.

![SERVICIOS WEB](./media/update-machine-learning-models/web-services.png)

## <a name="azure-machine-learning-studio-classic-update-resource-activity"></a>Actividad de actualización de recursos de Azure Machine Learning Studio (clásico)

El siguiente fragmento JSON define una actividad de ejecución de lotes de Azure Machine Learning Studio (clásico).

```json
{
    "name": "amlUpdateResource",
    "type": "AzureMLUpdateResource",
    "description": "description",
    "linkedServiceName": {
        "type": "LinkedServiceReference",
        "referenceName": "updatableScoringEndpoint2"
    },
    "typeProperties": {
        "trainedModelName": "ModelName",
        "trainedModelLinkedServiceName": {
                    "type": "LinkedServiceReference",
                    "referenceName": "StorageLinkedService"
                },
        "trainedModelFilePath": "ilearner file path"
    }
}
```

| Propiedad                      | Descripción                              | Obligatorio |
| :---------------------------- | :--------------------------------------- | :------- |
| name                          | Nombre de la actividad en la canalización     | Sí      |
| description                   | Texto que describe para qué se usa la actividad.  | No       |
| type                          | Para la actividad de actualización de recurso de Azure Machine Learning Studio (clásico), el tipo de actividad es **AzureMLUpdateResource**. | Sí      |
| linkedServiceName             | Servicio vinculado de Azure Machine Learning Studio (clásico) que contiene la propiedad updateResourceEndpoint. | Sí      |
| trainedModelName              | Nombre del módulo del modelo entrenado del experimento de servicio web que se actualizará | Sí      |
| trainedModelLinkedServiceName | Nombre del servicio vinculado de Azure Storage que contiene el archivo ilearner cargado por la operación de actualización | Sí      |
| trainedModelFilePath          | La ruta de acceso relativa en trainedModelLinkedService para representar el archivo ilearner cargado por la operación de actualización | Sí      |

## <a name="end-to-end-workflow"></a>Flujo de trabajo de un extremo a otro

El proceso completo de operacionalización, volviendo a entrenar un modelo y actualizando los servicios web predictivos, implica los pasos siguientes:

- Invocar el **servicio web de entrenamiento** mediante la **actividad de ejecución por lotes**. Invocar un servicio web de entrenamiento es lo mismo que invocar un servicio web predictivo descrito en [Creación de canalizaciones predictivas con Azure Machine Learning Studio (clásico) y la actividad de ejecución por lotes de Data Factory](transform-data-using-machine-learning.md). La salida del servicio web de entrenamiento es un archivo iLearner que puede usar para actualizar el servicio web de predicción.
- Invocar el **punto de conexión de actualización de recurso** del **servicio web de predicción** mediante el uso de la **actividad de actualización de recurso** para actualizar el servicio web con el modelo nuevamente entrenado.

## <a name="azure-machine-learning-studio-classic-linked-service"></a>Servicio vinculado de Azure Machine Learning Studio (clásico)

Para que funcione el flujo de trabajo de un extremo a otro mencionado anteriormente, debe crear dos servicios vinculados de Azure Machine Learning Studio (clásico):

1. Un servicio vinculado de Azure Machine Learning Studio (clásico) al servicio web de entrenamiento; este servicio vinculado se usa en la actividad de ejecución por lotes de la misma manera que se menciona en [Creación de canalizaciones predictivas con Azure Machine Learning Studio (clásico) y la actividad de ejecución por lotes de Data Factory](transform-data-using-machine-learning.md). La diferencia es que la salida del servicio web de entrenamiento es un archivo iLearner, que se usa en la actividad de actualización de recursos para actualizar el servicio web predictivo.
2. Un servicio vinculado de Azure Machine Learning Studio (clásico) al punto de conexión de actualización de recursos del servicio web predictivo. La actividad de actualización de recurso utiliza este servicio vinculado para actualizar el servicio web de predicción con el archivo iLearner devuelto en el paso anterior.

Para el segundo servicio vinculado de Azure Machine Learning Studio (clásico), la configuración es diferente si el servicio web de Azure Machine Learning Studio (clásico) es un servicio web clásico o un servicio web nuevo. Las diferencias se tratan por separado en las secciones siguientes.

## <a name="web-service-is-new-azure-resource-manager-web-service"></a>El servicio web es un nuevo servicio web de Azure Resource Manager

Si el servicio web es el nuevo tipo de servicio web que expone un punto de conexión de Azure Resource Manager, no es preciso agregar el segundo punto de conexión **no predeterminado**. En el servicio vinculado, **updateResourceEndpoint** tiene el formato:

```
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resource-group-name}/providers/Microsoft.MachineLearning/webServices/{web-service-name}?api-version=2016-05-01-preview
```

Puede obtener valores para los marcadores de posición en la dirección URL al consultar el servicio web en el [Portal de servicios web Azure Machine Learning Studio (clásico)](https://services.azureml.net/).

El nuevo tipo de punto de conexión de actualización de recurso requiere autenticación de entidad de servicio. Para usar autenticación de entidad de servicio, registre una entidad de aplicación en Azure Active Directory (Azure AD) y concédale el rol **Colaborador** o **Propietario** de la suscripción o el grupo de recursos al que pertenece el servicio web. Consulte [Cómo crear una entidad de servicio y asignar permisos para administrar recursos de Azure](../active-directory/develop/howto-create-service-principal-portal.md). Anote los siguientes valores; los usará para definir el servicio vinculado:

- Identificador de aplicación
- Clave de la aplicación
- Id. de inquilino

Esta es una definición de un servicio vinculado de ejemplo:

```json
{
    "name": "AzureMLLinkedService",
    "properties": {
        "type": "AzureML",
        "description": "The linked service for AML web service.",
        "typeProperties": {
            "mlEndpoint": "https://ussouthcentral.services.azureml.net/workspaces/0000000000000000  000000000000000000000/services/0000000000000000000000000000000000000/jobs?api-version=2.0",
            "apiKey": {
                "type": "SecureString",
                "value": "APIKeyOfEndpoint1"
            },
            "updateResourceEndpoint": "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resource-group-name}/providers/Microsoft.MachineLearning/webServices/{web-service-name}?api-version=2016-05-01-preview",
            "servicePrincipalId": "000000000-0000-0000-0000-0000000000000",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "servicePrincipalKey"
            },
            "tenant": "mycompany.com"
        }
    }
}
```

El escenario siguiente proporciona más detalles. Contiene un ejemplo para volver a entrenar y actualizar modelos de Azure Machine Learning Studio (clásico) a partir de una canalización de Azure Data Factory.


## <a name="sample-retraining-and-updating-an-azure-machine-learning-studio-classic-model"></a>Sample: Volver a entrenar y actualizar un modelo de Azure Machine Learning Studio (clásico)

Esta sección proporciona una canalización de ejemplo que usa la **actividad de ejecución por lotes de Azure Machine Learning Studio (clásico)** para volver a entrenar un modelo. La canalización usa también la **actividad de actualización de recursos de Azure Machine Learning Studio (clásico)** para actualizar el modelo en el servicio web de puntuación. La sección también proporciona fragmentos JSON para todos los servicios vinculados, conjuntos de datos y canalización en el ejemplo.

### <a name="azure-blob-storage-linked-service"></a>Servicio vinculado de almacenamiento de blobs de Azure:
Azure Storage contiene los siguientes datos:

* datos de aprendizaje. Los datos de entrada del servicio web de entrenamiento de Azure Machine Learning Studio (clásico).
* archivo iLearner. La salida del servicio web de entrenamiento de Azure Machine Learning Studio (clásico). Este archivo también es la entrada para la actividad Actualizar recurso.

Esta es la definición de JSON de ejemplo del servicio vinculado:

```JSON
{
    "name": "StorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=name;AccountKey=key"
        }
    }
}
```

### <a name="linked-service-for-azure-machine-learning-studio-classic-training-endpoint"></a>Servicio vinculado al punto de conexión de entrenamiento de Azure Machine Learning Studio (clásico)
El siguiente fragmento JSON define un servicio vinculado de Azure Machine Learning Studio (clásico) que apunta al punto de conexión predeterminado del servicio web de entrenamiento.

```JSON
{
    "name": "trainingEndpoint",
    "properties": {
        "type": "AzureML",
        "typeProperties": {
            "mlEndpoint": "https://ussouthcentral.services.azureml.net/workspaces/xxx/services/--training experiment--/jobs",
            "apiKey": "myKey"
        }
    }
}
```

En **Azure Machine Learning Studio (clásico)** , haga lo siguiente para obtener los valores de **mlEndpoint** y **apiKey**:

1. Haga clic en **SERVICIOS WEB** en el menú de la izquierda.
2. En la lista de servicios web, haga clic en el **servicio web de entrenamiento** .
3. Haga clic en Copiar junto al cuadro de texto **Clave de API** . Pegue la clave del portapapeles en el editor de JSON de Data Factory.
4. En **Azure Machine Learning Studio (clásico)** , haga clic en el vínculo **EJECUCIÓN DE LOTES**.
5. Copie el **URI de solicitud** de la sección **Solicitar** y péguelo en el editor de JSON de Data Factory.

### <a name="linked-service-for-azure-machine-learning-studio-classic-updatable-scoring-endpoint"></a>Servicio vinculado al punto de conexión de puntuación actualizable de Azure Machine Learning Studio (clásico):
El siguiente fragmento JSON define un servicio vinculado de Azure Machine Learning Studio (clásico) que apunta al punto de conexión actualizable del servicio web de puntuación.

```JSON
{
    "name": "updatableScoringEndpoint2",
    "properties": {
        "type": "AzureML",
        "typeProperties": {
            "mlEndpoint": "https://ussouthcentral.services.azureml.net/workspaces/00000000eb0abe4d6bbb1d7886062747d7/services/00000000026734a5889e02fbb1f65cefd/jobs?api-version=2.0",
            "apiKey": "sooooooooooh3WvG1hBfKS2BNNcfwSO7hhY6dY98noLfOdqQydYDIXyf2KoIaN3JpALu/AKtflHWMOCuicm/Q==",
            "updateResourceEndpoint": "https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Default-MachineLearning-SouthCentralUS/providers/Microsoft.MachineLearning/webServices/myWebService?api-version=2016-05-01-preview",
            "servicePrincipalId": "fe200044-c008-4008-a005-94000000731",
            "servicePrincipalKey": "zWa0000000000Tp6FjtZOspK/WMA2tQ08c8U+gZRBlw=",
            "tenant": "mycompany.com"
        }
    }
}
```

### <a name="pipeline"></a>Canalización
La canalización tiene dos actividades: **AzureMLBatchExecution** y **AzureMLUpdateResource**. La actividad Ejecución por lotes toma los datos de entrenamiento como entrada y genera como salida un archivo iLearner. La actividad de actualización de recurso, a continuación, toma este archivo iLearner y lo usa para actualizar el servicio web de predicción.

```JSON
{
    "name": "LookupPipelineDemo",
    "properties": {
        "activities": [
            {
                "name": "amlBEGetilearner",
                "description": "Use AML BES to get the ileaner file from training web service",
                "type": "AzureMLBatchExecution",
                "linkedServiceName": {
                    "referenceName": "trainingEndpoint",
                    "type": "LinkedServiceReference"
                },
                "typeProperties": {
                    "webServiceInputs": {
                        "input1": {
                            "LinkedServiceName":{
                                "referenceName": "StorageLinkedService",
                                "type": "LinkedServiceReference"
                            },
                            "FilePath":"azuremltesting/input"
                        },
                        "input2": {
                            "LinkedServiceName":{
                                "referenceName": "StorageLinkedService",
                                "type": "LinkedServiceReference"
                            },
                            "FilePath":"azuremltesting/input"
                        }
                    },
                    "webServiceOutputs": {
                        "output1": {
                            "LinkedServiceName":{
                                "referenceName": "StorageLinkedService",
                                "type": "LinkedServiceReference"
                            },
                            "FilePath":"azuremltesting/output"
                        }
                    }
                }
            },
            {
                "name": "amlUpdateResource",
                "type": "AzureMLUpdateResource",
                "description": "Use AML Update Resource to update the predict web service",
                "linkedServiceName": {
                    "type": "LinkedServiceReference",
                    "referenceName": "updatableScoringEndpoint2"
                },
                "typeProperties": {
                    "trainedModelName": "ADFV2Sample Model [trained model]",
                    "trainedModelLinkedServiceName": {
                        "type": "LinkedServiceReference",
                        "referenceName": "StorageLinkedService"
                    },
                    "trainedModelFilePath": "azuremltesting/output/newModelForArm.ilearner"
                },
                "dependsOn": [
                    {
                        "activity": "amlbeGetilearner",
                        "dependencyConditions": [ "Succeeded" ]
                    }
                ]
            }
        ]
    }
}
```
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
