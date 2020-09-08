---
title: Creación de un cliente para el modelo implementado como servicio web
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo llamar a un punto de conexión de servicio web generado al implementar un modelo desde Azure Machine Learning. El punto de conexión expone una API de REST, a la que puede llamar para realizar una inferencia con el modelo. Cree clientes para esta API mediante el lenguaje de programación que quiera.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: aashishb
author: aashishb
ms.reviewer: larryfr
ms.date: 06/17/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, devx-track-csharp
ms.openlocfilehash: 56cd2117a352626cf59023d62ea8c931401389c5
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018100"
---
# <a name="consume-an-azure-machine-learning-model-deployed-as-a-web-service"></a>Consumir un modelo de Azure Machine Learning que está implementado como un servicio web
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

La implementación de un modelo de Azure Machine Learning como un servicio web crea un punto de conexión de la API de REST. Puede enviar datos a este punto de conexión y recibir la predicción que devuelve el modelo. En este documento aprenderá a crear clientes para el servicio web usando C #, Go, Java y Python.

Al implementar un modelo en el entorno local, en Azure Container Instances, en Azure Kubernetes Service o en matrices de puertas programables (FPGA) creará un servicio web. Recuperará el URI que se usa para obtener acceso al servicio web gracias al [SDK de Azure Machine Learning](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py). Si la autenticación está habilitada, también puede usar el SDK para obtener las claves o tokens de autenticación.

El flujo de trabajo general al crear un cliente que usa un servicio web de Machine Learning es:

1. Usar el SDK para obtener la información de conexión.
1. Determinar el tipo de datos de solicitud que usa el modelo.
1. Crear una aplicación que llame al servicio web.

> [!TIP]
> Los ejemplos de este documento se crean manualmente sin el uso de las especificaciones de OpenAPI (Swagger). Si ha habilitado una especificación de OpenAPI para la implementación, puede usar herramientas como [swagger-codegen](https://github.com/swagger-api/swagger-codegen) para crear bibliotecas de cliente para el servicio.

## <a name="connection-information"></a>Información sobre la conexión

> [!NOTE]
> Use el SDK de Azure Machine Learning para obtener la información del servicio web. Este es un SDK de Python. Puede usar cualquier lenguaje para crear un cliente para el servicio.

La clase [azureml.core.Webservice](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice(class)?view=azure-ml-py) proporciona la información necesaria para crear un cliente. Las siguientes propiedades `Webservice` son útiles para crear una aplicación cliente:

* `auth_enabled`: si la autenticación de la clave está habilitada, `True`; de lo contrario, `False`.
* `token_auth_enabled`: si la autenticación del token está habilitada, `True`; de lo contrario, `False`.
* `scoring_uri`: dirección de la API REST.
* `swagger_uri`: la dirección de la especificación OpenAPI. Este identificador URI estará disponible si habilitó la generación automática de esquemas. Para más información, consulte [Implementación de modelos con Azure Machine Learning](how-to-deploy-and-where.md).

Existen tres formas de recuperar esta información para los servicios web implementados:

* Cuando implementa un modelo, se devuelve un objeto `Webservice` con información sobre el servicio:

    ```python
    service = Model.deploy(ws, "myservice", [model], inference_config, deployment_config)
    service.wait_for_deployment(show_output = True)
    print(service.scoring_uri)
    print(service.swagger_uri)
    ```

* Puede usar `Webservice.list` para recuperar una lista de servicios web implementados para modelos en su área de trabajo. Puede agregar filtros para limitar la lista de información devuelta. Para obtener más información sobre lo que se puede filtrar, consulte la documentación de referencia de [Webservice.list](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.webservice.webservice?view=azure-ml-py).

    ```python
    services = Webservice.list(ws)
    print(services[0].scoring_uri)
    print(services[0].swagger_uri)
    ```

* Si conoce el nombre del servicio implementado, puede crear una nueva instancia de `Webservice` y proporcionar el espacio de trabajo y el nombre del servicio como parámetros. El nuevo objeto contiene información sobre el servicio implementado.

    ```python
    service = Webservice(workspace=ws, name='myservice')
    print(service.scoring_uri)
    print(service.swagger_uri)
    ```

### <a name="secured-web-service"></a>Servicio web protegido

Si ha protegido el servicio web implementado mediante un certificado TLS/SSL, puede usar [HTTPS](https://en.wikipedia.org/wiki/HTTPS) para conectarse al servicio mediante la puntuación o el URI de Swagger. HTTPS le ayuda a proteger las comunicaciones entre un cliente y un servicio web mediante el cifrado de las comunicaciones entre los dos. El cifrado usa [Seguridad de la capa de transporte (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security). TLS se conoce a veces todavía como *Capa de sockets seguros* (SSL), que fue su predecesor.

> [!IMPORTANT]
> Los servicios web implementados a través de Azure Machine Learning solo admiten la versión 1.2 de TLS. Al crear una aplicación cliente, asegúrese de que es compatible con esta versión.

Para obtener más información, vea [Uso de TLS para proteger un servicio web mediante Azure Machine Learning](how-to-secure-web-service.md).

### <a name="authentication-for-services"></a>Autenticación para servicios

Azure Machine Learning proporciona dos formas de controlar el acceso a los servicios web.

|Método de autenticación|ACI|AKS|
|---|---|---|
|Clave|Deshabilitado de forma predeterminada| Habilitado de forma predeterminada|
|Token| No disponible| Deshabilitado de forma predeterminada |

Al enviar una solicitud a un servicio protegido con una clave o un token, use el encabezado __Autorización__ para pasar la clave o el token. La clave o el token deben tener el formato `Bearer <key-or-token>`, donde `<key-or-token>` es el valor de la clave o del token.

La principal diferencia entre las claves y los tokens es que las **claves son estáticas y se pueden volver a generar manualmente** y los **tokens deben actualizarse tras la expiración**. La autenticación basada en claves es compatible con la instancia de Azure Container Instance y los servicios web implementados por el servicio Kubernetes de Azure, y la autenticación basada en token está **solo** disponible para las implementaciones del servicio Kubernetes de Azure. Consulte el [procedimiento](how-to-setup-authentication.md#web-service-authentication) en la autenticación para más información y ejemplos de código específicos.


#### <a name="authentication-with-keys"></a>Autenticación con claves

Al habilitar la autenticación para una implementación, se crean automáticamente las claves de autenticación.

* La autenticación se habilita de manera predeterminada cuando se implementa en Azure Kubernetes Service.
* La autenticación se deshabilita de manera predeterminada cuando se implementa en instancias de Azure Container.

Para controlar la autenticación, use el parámetro `auth_enabled` cuando cree o actualice una implementación.

Si la autenticación está habilitada, puede usar el método `get_keys` para recuperar una clave de autenticación primaria y secundaria:

```python
primary, secondary = service.get_keys()
print(primary)
```

> [!IMPORTANT]
> Si necesita regenerar una clave, use [`service.regen_key`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice(class)?view=azure-ml-py).

#### <a name="authentication-with-tokens"></a>Autenticación con tokens

Cuando se habilita la autenticación por tokens para un servicio web, el usuario debe proporcionar un token JWT de Azure Machine Learning al servicio web para tener acceso a él. 

* La autenticación por tokens se deshabilita de manera predeterminada cuando se implementa en Azure Kubernetes Service.
* La autenticación por tokens no se admite cuando se implementa en Azure Container Instances.

Para controlar la autenticación por tokens, use el parámetro `token_auth_enabled` cuando cree o actualice una implementación.

Si la autenticación por tokens está habilitada, puede usar el método `get_token` para recuperar un token de portador y la hora de expiración de los tokens:

```python
token, refresh_by = service.get_token()
print(token)
```

Si tiene la [CLI de Azure y la extensión de Machine Learning](reference-azure-machine-learning-cli.md), puede usar el siguiente comando para obtener un token:

```azurecli
az ml service get-access-token -n <service-name>
```

> [!IMPORTANT]
> Actualmente, la única manera de recuperar el token es mediante el SDK de Azure Machine Learning o la extensión de Machine Learning de la CLI de Azure.

Tendrá que solicitar un nuevo token después de la hora del token `refresh_by`. 

## <a name="request-data"></a>Datos de la solicitud

La API REST espera que el cuerpo de la solicitud sea un documento JSON con la siguiente estructura:

```json
{
    "data":
        [
            <model-specific-data-structure>
        ]
}
```

> [!IMPORTANT]
> La estructura de los datos debe coincidir con lo que esperan el script y el modelo de puntuación en el servicio. El script de puntuación podría modificar los datos antes de pasarlos al modelo.

### <a name="binary-data"></a>Datos binarios

Para obtener información sobre cómo habilitar la compatibilidad con datos binarios en un servicio, consulte [Datos binarios](how-to-deploy-advanced-entry-script.md#binary-data).

> [!TIP]
> La habilitación de la compatibilidad con datos binarios se produce en el archivo score.py que usa el modelo implementado. En el cliente, use la funcionalidad HTTP del lenguaje de programación. Por ejemplo, el siguiente fragmento de código envía el contenido de un archivo JPG a un servicio web:
>
> ```python
> import requests
> # Load image data
> data = open('example.jpg', 'rb').read()
> # Post raw data to scoring URI
> res = request.post(url='<scoring-uri>', data=data, headers={'Content-Type': 'application/> octet-stream'})
> ```

### <a name="cross-origin-resource-sharing-cors"></a>Uso compartido de recursos entre orígenes (CORS)

Para obtener información sobre cómo habilitar la compatibilidad con CORS en un servicio, consulte [Uso compartido de recursos de varios orígenes](how-to-deploy-advanced-entry-script.md#cors).

## <a name="call-the-service-c"></a>Llamada al servicio (C#)

En este ejemplo se muestra cómo usar C# para llamar al servicio web creado a partir del ejemplo [Train within notebook](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training-with-deep-learning/how-to-use-estimator/notebook_example.ipynb):

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Net.Http;
using System.Net.Http.Headers;
using Newtonsoft.Json;

namespace MLWebServiceClient
{
    // The data structure expected by the service
    internal class InputData
    {
        [JsonProperty("data")]
        // The service used by this example expects an array containing
        //   one or more arrays of doubles
        internal double[,] data;
    }
    class Program
    {
        static void Main(string[] args)
        {
            // Set the scoring URI and authentication key or token
            string scoringUri = "<your web service URI>";
            string authKey = "<your key or token>";

            // Set the data to be sent to the service.
            // In this case, we are sending two sets of data to be scored.
            InputData payload = new InputData();
            payload.data = new double[,] {
                {
                    0.0199132141783263,
                    0.0506801187398187,
                    0.104808689473925,
                    0.0700725447072635,
                    -0.0359677812752396,
                    -0.0266789028311707,
                    -0.0249926566315915,
                    -0.00259226199818282,
                    0.00371173823343597,
                    0.0403433716478807
                },
                {
                    -0.0127796318808497, 
                    -0.044641636506989, 
                    0.0606183944448076, 
                    0.0528581912385822, 
                    0.0479653430750293, 
                    0.0293746718291555, 
                    -0.0176293810234174, 
                    0.0343088588777263, 
                    0.0702112981933102, 
                    0.00720651632920303
                }
            };

            // Create the HTTP client
            HttpClient client = new HttpClient();
            // Set the auth header. Only needed if the web service requires authentication.
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", authKey);

            // Make the request
            try {
                var request = new HttpRequestMessage(HttpMethod.Post, new Uri(scoringUri));
                request.Content = new StringContent(JsonConvert.SerializeObject(payload));
                request.Content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
                var response = client.SendAsync(request).Result;
                // Display the response from the web service
                Console.WriteLine(response.Content.ReadAsStringAsync().Result);
            }
            catch (Exception e)
            {
                Console.Out.WriteLine(e.Message);
            }
        }
    }
}
```

Los resultados devueltos son similares al siguiente documento JSON:

```json
[217.67978776218715, 224.78937091757172]
```

## <a name="call-the-service-go"></a>Llamada al servicio (Go)

En este ejemplo se muestra cómo usar Go para llamar al servicio web creado a partir del ejemplo [Train within notebook](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training-with-deep-learning/how-to-use-estimator/notebook_example.ipynb):

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
)

// Features for this model are an array of decimal values
type Features []float64

// The web service input can accept multiple sets of values for scoring
type InputData struct {
    Data []Features `json:"data",omitempty`
}

// Define some example data
var exampleData = []Features{
    []float64{
        0.0199132141783263, 
        0.0506801187398187, 
        0.104808689473925, 
        0.0700725447072635, 
        -0.0359677812752396, 
        -0.0266789028311707, 
        -0.0249926566315915, 
        -0.00259226199818282, 
        0.00371173823343597, 
        0.0403433716478807,
    },
    []float64{
        -0.0127796318808497, 
        -0.044641636506989, 
        0.0606183944448076, 
        0.0528581912385822, 
        0.0479653430750293, 
        0.0293746718291555, 
        -0.0176293810234174, 
        0.0343088588777263, 
        0.0702112981933102, 
        0.00720651632920303,
    },
}

// Set to the URI for your service
var serviceUri string = "<your web service URI>"
// Set to the authentication key or token (if any) for your service
var authKey string = "<your key or token>"

func main() {
    // Create the input data from example data
    jsonData := InputData{
        Data: exampleData,
    }
    // Create JSON from it and create the body for the HTTP request
    jsonValue, _ := json.Marshal(jsonData)
    body := bytes.NewBuffer(jsonValue)

    // Create the HTTP request
    client := &http.Client{}
    request, err := http.NewRequest("POST", serviceUri, body)
    request.Header.Add("Content-Type", "application/json")

    // These next two are only needed if using an authentication key
    bearer := fmt.Sprintf("Bearer %v", authKey)
    request.Header.Add("Authorization", bearer)

    // Send the request to the web service
    resp, err := client.Do(request)
    if err != nil {
        fmt.Println("Failure: ", err)
    }

    // Display the response received
    respBody, _ := ioutil.ReadAll(resp.Body)
    fmt.Println(string(respBody))
}
```

Los resultados devueltos son similares al siguiente documento JSON:

```json
[217.67978776218715, 224.78937091757172]
```

## <a name="call-the-service-java"></a>Llamada al servicio (Java)

En este ejemplo se muestra cómo usar Java para llamar al servicio web creado a partir del ejemplo [Train within notebook ](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training-with-deep-learning/how-to-use-estimator/notebook_example.ipynb):

```java
import java.io.IOException;
import org.apache.http.client.fluent.*;
import org.apache.http.entity.ContentType;
import org.json.simple.JSONArray;
import org.json.simple.JSONObject;

public class App {
    // Handle making the request
    public static void sendRequest(String data) {
        // Replace with the scoring_uri of your service
        String uri = "<your web service URI>";
        // If using authentication, replace with the auth key or token
        String key = "<your key or token>";
        try {
            // Create the request
            Content content = Request.Post(uri)
            .addHeader("Content-Type", "application/json")
            // Only needed if using authentication
            .addHeader("Authorization", "Bearer " + key)
            // Set the JSON data as the body
            .bodyString(data, ContentType.APPLICATION_JSON)
            // Make the request and display the response.
            .execute().returnContent();
            System.out.println(content);
        }
        catch (IOException e) {
            System.out.println(e);
        }
    }
    public static void main(String[] args) {
        // Create the data to send to the service
        JSONObject obj = new JSONObject();
        // In this case, it's an array of arrays
        JSONArray dataItems = new JSONArray();
        // Inner array has 10 elements
        JSONArray item1 = new JSONArray();
        item1.add(0.0199132141783263);
        item1.add(0.0506801187398187);
        item1.add(0.104808689473925);
        item1.add(0.0700725447072635);
        item1.add(-0.0359677812752396);
        item1.add(-0.0266789028311707);
        item1.add(-0.0249926566315915);
        item1.add(-0.00259226199818282);
        item1.add(0.00371173823343597);
        item1.add(0.0403433716478807);
        // Add the first set of data to be scored
        dataItems.add(item1);
        // Create and add the second set
        JSONArray item2 = new JSONArray();
        item2.add(-0.0127796318808497);
        item2.add(-0.044641636506989);
        item2.add(0.0606183944448076);
        item2.add(0.0528581912385822);
        item2.add(0.0479653430750293);
        item2.add(0.0293746718291555);
        item2.add(-0.0176293810234174);
        item2.add(0.0343088588777263);
        item2.add(0.0702112981933102);
        item2.add(0.00720651632920303);
        dataItems.add(item2);
        obj.put("data", dataItems);

        // Make the request using the JSON document string
        sendRequest(obj.toJSONString());
    }
}
```

Los resultados devueltos son similares al siguiente documento JSON:

```json
[217.67978776218715, 224.78937091757172]
```

## <a name="call-the-service-python"></a>Llamada al servicio (Python)

En este ejemplo muestra cómo utilizar Python para llamar al servicio web creado a partir del ejemplo [Train within notebook](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training-with-deep-learning/how-to-use-estimator/notebook_example.ipynb):

```python
import requests
import json

# URL for the web service
scoring_uri = '<your web service URI>'
# If the service is authenticated, set the key or token
key = '<your key or token>'

# Two sets of data to score, so we get two results back
data = {"data":
        [
            [
                0.0199132141783263,
                0.0506801187398187,
                0.104808689473925,
                0.0700725447072635,
                -0.0359677812752396,
                -0.0266789028311707,
                -0.0249926566315915,
                -0.00259226199818282,
                0.00371173823343597,
                0.0403433716478807
            ],
            [
                -0.0127796318808497,
                -0.044641636506989,
                0.0606183944448076,
                0.0528581912385822,
                0.0479653430750293,
                0.0293746718291555,
                -0.0176293810234174,
                0.0343088588777263,
                0.0702112981933102,
                0.00720651632920303]
        ]
        }
# Convert to JSON string
input_data = json.dumps(data)

# Set the content type
headers = {'Content-Type': 'application/json'}
# If authentication is enabled, set the authorization header
headers['Authorization'] = f'Bearer {key}'

# Make the request and display the response
resp = requests.post(scoring_uri, input_data, headers=headers)
print(resp.text)
```

Los resultados devueltos son similares al siguiente documento JSON:

```JSON
[217.67978776218715, 224.78937091757172]
```


## <a name="web-service-schema-openapi-specification"></a>Esquema de servicio web (especificación de OpenAPI)

Si ha usado la generación automática de esquemas con la implementación, puede obtener la dirección de la especificación de OpenAPI para el servicio mediante la [propiedad swagger_uri](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.local.localwebservice?view=azure-ml-py#swagger-uri). (Por ejemplo, `print(service.swagger_uri)`). Use una solicitud GET o abra el URI en un explorador para recuperar la especificación.

El siguiente documento JSON es un ejemplo de un esquema (especificación de OpenAPI) generado para una implementación:

```json
{
    "swagger": "2.0",
    "info": {
        "title": "myservice",
        "description": "API specification for Azure Machine Learning myservice",
        "version": "1.0"
    },
    "schemes": [
        "https"
    ],
    "consumes": [
        "application/json"
    ],
    "produces": [
        "application/json"
    ],
    "securityDefinitions": {
        "Bearer": {
            "type": "apiKey",
            "name": "Authorization",
            "in": "header",
            "description": "For example: Bearer abc123"
        }
    },
    "paths": {
        "/": {
            "get": {
                "operationId": "ServiceHealthCheck",
                "description": "Simple health check endpoint to ensure the service is up at any given point.",
                "responses": {
                    "200": {
                        "description": "If service is up and running, this response will be returned with the content 'Healthy'",
                        "schema": {
                            "type": "string"
                        },
                        "examples": {
                            "application/json": "Healthy"
                        }
                    },
                    "default": {
                        "description": "The service failed to execute due to an error.",
                        "schema": {
                            "$ref": "#/definitions/ErrorResponse"
                        }
                    }
                }
            }
        },
        "/score": {
            "post": {
                "operationId": "RunMLService",
                "description": "Run web service's model and get the prediction output",
                "security": [
                    {
                        "Bearer": []
                    }
                ],
                "parameters": [
                    {
                        "name": "serviceInputPayload",
                        "in": "body",
                        "description": "The input payload for executing the real-time machine learning service.",
                        "schema": {
                            "$ref": "#/definitions/ServiceInput"
                        }
                    }
                ],
                "responses": {
                    "200": {
                        "description": "The service processed the input correctly and provided a result prediction, if applicable.",
                        "schema": {
                            "$ref": "#/definitions/ServiceOutput"
                        }
                    },
                    "default": {
                        "description": "The service failed to execute due to an error.",
                        "schema": {
                            "$ref": "#/definitions/ErrorResponse"
                        }
                    }
                }
            }
        }
    },
    "definitions": {
        "ServiceInput": {
            "type": "object",
            "properties": {
                "data": {
                    "type": "array",
                    "items": {
                        "type": "array",
                        "items": {
                            "type": "integer",
                            "format": "int64"
                        }
                    }
                }
            },
            "example": {
                "data": [
                    [ 10, 9, 8, 7, 6, 5, 4, 3, 2, 1 ]
                ]
            }
        },
        "ServiceOutput": {
            "type": "array",
            "items": {
                "type": "number",
                "format": "double"
            },
            "example": [
                3726.995
            ]
        },
        "ErrorResponse": {
            "type": "object",
            "properties": {
                "status_code": {
                    "type": "integer",
                    "format": "int32"
                },
                "message": {
                    "type": "string"
                }
            }
        }
    }
}
```

Para más información, consulte la [especificación de OpenAPI](https://swagger.io/specification/).

Para obtener una utilidad que puede crear bibliotecas de cliente a partir de la especificación, consulte [swagger-codegen](https://github.com/swagger-api/swagger-codegen).


> [!TIP]
> Puede recuperar el documento JSON del esquema después de implementar el servicio. Use la [propiedad swagger_uri](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.local.localwebservice?view=azure-ml-py#swagger-uri) del servicio web implementado (por ejemplo, `service.swagger_uri`) para obtener el URI del archivo Swagger del servicio web local.

## <a name="consume-the-service-from-power-bi"></a>Consumo del servicio desde Power BI

Power BI admite el consumo de servicios web de Azure Machine Learning para enriquecer los datos de Power BI con predicciones. 

Para generar un servicio web cuyo consumo se admita en Power BI, el esquema debe admitir el formato que requiere Power BI. [Aprenda a crear un esquema admitido por Power BI](https://docs.microsoft.com/azure/machine-learning/how-to-deploy-and-where#example-entry-script).

Una vez implementado el servicio web, los flujos de datos de Power BI lo podrán consumir. [Aprenda a consumir un servicio web de Azure Machine Learning desde Power BI](https://docs.microsoft.com/power-bi/service-machine-learning-integration).

## <a name="next-steps"></a>Pasos siguientes

Para ver una arquitectura de referencia para la puntuación en tiempo real de Python y los modelos de aprendizaje profundo, vaya al [centro de arquitectura de Azure](/azure/architecture/reference-architectures/ai/realtime-scoring-python).
