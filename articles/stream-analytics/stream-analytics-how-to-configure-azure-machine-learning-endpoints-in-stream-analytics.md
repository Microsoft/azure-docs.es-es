---
title: Uso de puntos de conexión de Machine Learning en Azure Stream Analytics
description: En este artículo se describe cómo utilizar las funciones definidas por el usuario de lenguaje máquina en Azure Stream Analytics.
author: jseb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 06/11/2019
ms.openlocfilehash: 33d1e73523ca85a21c032729b6537ee3297ea23e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87020838"
---
# <a name="azure-machine-learning-studio-classic-integration-in-stream-analytics-preview"></a>Integración de Azure Machine Learning Studio (clásico) en Stream Analytics (versión preliminar)
Stream Analytics admite funciones definidas por el usuario que llaman a puntos de conexión de Azure Machine Learning Studio (clásico). La compatibilidad con la API de REST para esta característica se detalla en la [biblioteca API de REST de Stream Analytics](https://msdn.microsoft.com/library/azure/dn835031.aspx). Este artículo proporciona información adicional necesaria para una implementación correcta de esta capacidad en Stream Analytics. También se ha publicado un tutorial, que está disponible [aquí](stream-analytics-machine-learning-integration-tutorial.md).

## <a name="overview-azure-machine-learning-studio-classic-terminology"></a>Introducción: terminología de Azure Machine Learning Studio (clásico)
Microsoft Azure Machine Learning Studio (clásico) proporciona una herramienta colaborativa de arrastrar y colocar que le permite crear, probar e implementar soluciones de análisis predictivo en sus datos. Esta herramienta de denomina *Azure Machine Learning Studio (clásico)* . El estudio se usará para interactuar con los recursos de Machine Learning, así como para compilar, probar e iterar fácilmente en su diseño. A continuación, se proporcionan estos recursos y sus definiciones.

* **Área de trabajo**: el *área de trabajo* es un contenedor con todos los demás recursos de Machine Learning en un solo lugar para la administración y el control.
* **Experimento**: los científicos de datos crean *experimentos* para utilizar conjuntos de datos y entrenar un modelo de Aprendizaje automático.
* **Punto de conexión**: los *puntos de conexión* son el objeto de Azure Machine Learning Studio (clásico) que se usa para tomar características como entradas, aplicar un modelo de aprendizaje automático especificado y devolver la salida puntuada.
* **Servicio web de puntuación**: un *servicio web de puntuación* es una colección de puntos de conexión, como se mencionó anteriormente.

Cada punto de conexión tiene varias API para la ejecución de lotes y la ejecución sincrónica. Stream Analytics usa la ejecución sincrónica. El servicio específico se denomina [servicio de solicitud y respuesta](../machine-learning/studio/consume-web-services.md) en Azure Machine Learning Studio (clásico).

## <a name="machine-learning-resources-needed-for-stream-analytics-jobs"></a>Recursos de Machine Learning necesarios para trabajos de Stream Analytics
Para el procesamiento de trabajos de Stream Analytics, para la correcta ejecución se necesitan un punto de conexión de solicitud/respuesta, una [apikey](https://docs.microsoft.com/azure/machine-learning/studio/consume-web-services)y una definición de Swagger. Stream Analytics tiene un punto de conexión adicional que construye la dirección URL de Swagger, busca en la interfaz y devuelve una definición de función definida por el usuario predeterminada al usuario.

## <a name="configure-a-stream-analytics-and-machine-learning-udf-via-rest-api"></a>Configuración de un Análisis de transmisiones y funciones definidas por el usuario de Machine Learning mediante la API de REST
Mediante las API de REST, puede configurar el trabajo para llamar a funciones de Aprendizaje automático de Azure. Los pasos son los siguientes:

1. Creación de un trabajo de Stream Analytics
2. Definición de una entrada
3. Definición de una salida
4. Creación de una función definida por el usuario
5. Creación de una transformación de Stream Analytics que llame a la función definida por el usuario
6. Inicio del trabajo

## <a name="creating-a-udf-with-basic-properties"></a>Creación de una función definida por el usuario con las propiedades básicas
Por ejemplo, el siguiente código de ejemplo crea una función definida por el usuario escalar denominada *newudf* que enlaza con un punto de conexión de Azure Machine Learning Studio (clásico). Tenga en cuenta que el *punto de conexión* (URI de servicio) se puede encontrar en la página de ayuda de API para el servicio seleccionado y la *apiKey* puede encontrarse en la página principal de servicios.

```
    PUT : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.StreamAnalytics/streamingjobs/<streamingjobName>/functions/<udfName>?api-version=<apiVersion>
```

Ejemplo del cuerpo de solicitud:

```json
    {
        "name": "newudf",
        "properties": {
            "type": "Scalar",
            "properties": {
                "binding": {
                    "type": "Microsoft.MachineLearning/WebService",
                    "properties": {
                        "endpoint": "https://ussouthcentral.services.azureml.net/workspaces/f80d5d7a77fb4b46bf2a30c63c078dca/services/b7be5e40fd194258796fb402c1958eaf/execute ",
                        "apiKey": "replacekeyhere"
                    }
                }
            }
        }
    }
```

## <a name="call-retrievedefaultdefinition-endpoint-for-default-udf"></a>Llamada al punto de conexión RetrieveDefaultDefinition para la función definida por el usuario predeterminada
Una vez creado el esqueleto de la función definida por el usuario, es necesaria la definición completa de la función definida por el usuario. El punto de conexión RetrieveDefaultDefinition ayuda a obtener la definición predeterminada de una función escalar enlazada a un punto de conexión de Azure Machine Learning Studio (clásico). La siguiente carga requiere obtener la definición de la función definida por el usuario predeterminada para una función escalar enlazada a un punto de conexión de Azure Machine Learning. No especifica el punto de conexión real, porque ya se ha proporcionado durante la solicitud PUT. Stream Analytics llamará al punto de conexión proporcionado en la solicitud si se proporciona explícitamente. De lo contrario, usará al que se hace referencia desde el principio. Aquí, la función definida por el usuario toma un parámetro de una sola cadena (una frase) y devuelve una única salida de tipo "string" que indica la etiqueta "sentiment" para esa frase.

```
POST : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.StreamAnalytics/streamingjobs/<streamingjobName>/functions/<udfName>/RetrieveDefaultDefinition?api-version=<apiVersion>
```

Ejemplo del cuerpo de solicitud:

```json
    {
        "bindingType": "Microsoft.MachineLearning/WebService",
        "bindingRetrievalProperties": {
            "executeEndpoint": null,
            "udfType": "Scalar"
        }
    }
```

Un ejemplo de salida tendría el aspecto siguiente.

```json
    {
        "name": "newudf",
        "properties": {
            "type": "Scalar",
            "properties": {
                "inputs": [{
                    "dataType": "nvarchar(max)",
                    "isConfigurationParameter": null
                }],
                "output": {
                    "dataType": "nvarchar(max)"
                },
                "binding": {
                    "type": "Microsoft.MachineLearning/WebService",
                    "properties": {
                        "endpoint": "https://ussouthcentral.services.azureml.net/workspaces/f80d5d7a77ga4a4bbf2a30c63c078dca/services/b7be5e40fd194258896fb602c1858eaf/execute",
                        "apiKey": null,
                        "inputs": {
                            "name": "input1",
                            "columnNames": [{
                                "name": "tweet",
                                "dataType": "string",
                                "mapTo": 0
                            }]
                        },
                        "outputs": [{
                            "name": "Sentiment",
                            "dataType": "string"
                        }],
                        "batchSize": 10
                    }
                }
            }
        }
    }
```

## <a name="patch-udf-with-the-response"></a>Revisión de la función definida por el usuario con la respuesta
Ahora se debe revisar la función definida por el usuario con la respuesta anterior, tal como se muestra a continuación.

```
PATCH : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.StreamAnalytics/streamingjobs/<streamingjobName>/functions/<udfName>?api-version=<apiVersion>
```

Cuerpo de la solicitud (salida de RetrieveDefaultDefinition):

```json
    {
        "name": "newudf",
        "properties": {
            "type": "Scalar",
            "properties": {
                "inputs": [{
                    "dataType": "nvarchar(max)",
                    "isConfigurationParameter": null
                }],
                "output": {
                    "dataType": "nvarchar(max)"
                },
                "binding": {
                    "type": "Microsoft.MachineLearning/WebService",
                    "properties": {
                        "endpoint": "https://ussouthcentral.services.azureml.net/workspaces/f80d5d7a77ga4a4bbf2a30c63c078dca/services/b7be5e40fd194258896fb602c1858eaf/execute",
                        "apiKey": null,
                        "inputs": {
                            "name": "input1",
                            "columnNames": [{
                                "name": "tweet",
                                "dataType": "string",
                                "mapTo": 0
                            }]
                        },
                        "outputs": [{
                            "name": "Sentiment",
                            "dataType": "string"
                        }],
                        "batchSize": 10
                    }
                }
            }
        }
    }
```

## <a name="implement-stream-analytics-transformation-to-call-the-udf"></a>Implementación de una transformación de Stream Analytics que llame a la función definida por el usuario
Ahora consulte la función definida por el usuario (aquí llamada scoreTweet) para cada evento de entrada y escriba una respuesta para ese evento en una salida.

```json
    {
        "name": "transformation",
        "properties": {
            "streamingUnits": null,
            "query": "select *,scoreTweet(Tweet) TweetSentiment into blobOutput from blobInput"
        }
    }
```


## <a name="get-help"></a>Obtener ayuda
Para más ayuda, pruebe nuestra [Página de preguntas y respuestas de Microsoft sobre Azure Stream Analytics](https://docs.microsoft.com/answers/topics/azure-stream-analytics.html)

## <a name="next-steps"></a>Pasos siguientes
* [Introducción a Azure Stream Analytics](stream-analytics-introduction.md)
* [Introducción al uso de Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Escalación de trabajos de Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Referencia del lenguaje de consulta de Azure Stream Analytics](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference)
* [Referencia de API de REST de administración de Azure Stream Analytics](https://msdn.microsoft.com/library/azure/dn835031.aspx)
