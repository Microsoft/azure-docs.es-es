---
title: Actividad web en Azure Data Factory
description: Aprenda a usar la actividad web, una de las actividades de flujo de control admitidas por Data Factory, para invocar un punto de conexión REST desde una canalización.
services: data-factory
documentationcenter: ''
author: djpmsft
ms.author: daperlov
manager: jroth
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 12/19/2018
ms.openlocfilehash: 95cbb509beba82a14b9f8f8a11c603a6d7b8689d
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87280807"
---
# <a name="web-activity-in-azure-data-factory"></a>Actividad web en Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]


La actividad Web puede usarse para llamar a un punto de conexión REST personalizado desde una canalización de Data Factory. Puede pasar conjuntos de datos y servicios vinculados que la actividad consumirá y a los que tendrá acceso.

> [!NOTE]
> Se admite la actividad web para invocar direcciones URL que se hospedan en una red virtual privada y mediante el uso de un entorno de ejecución de integración autohospedado. El entorno de ejecución de integración debe tener una línea de visión al punto de conexión de la dirección URL. 

## <a name="syntax"></a>Sintaxis

```json
{
   "name":"MyWebActivity",
   "type":"WebActivity",
   "typeProperties":{
      "method":"Post",
      "url":"<URLEndpoint>",
      "connectVia": {
          "referenceName": "<integrationRuntimeName>",
          "type": "IntegrationRuntimeReference"
      }
      "headers":{
         "Content-Type":"application/json"
      },
      "authentication":{
         "type":"ClientCertificate",
         "pfx":"****",
         "password":"****"
      },
      "datasets":[
         {
            "referenceName":"<ConsumedDatasetName>",
            "type":"DatasetReference",
            "parameters":{
               ...
            }
         }
      ],
      "linkedServices":[
         {
            "referenceName":"<ConsumedLinkedServiceName>",
            "type":"LinkedServiceReference"
         }
      ]
   }
}

```

## <a name="type-properties"></a>Propiedades de tipo

Propiedad | Descripción | Valores permitidos | Obligatorio
-------- | ----------- | -------------- | --------
name | Nombre de la actividad web | String | Sí
type | Se debe establecer en **WebActivity**. | String | Sí
method | Método de API de REST para el punto de conexión de destino. | String. <br/><br/>Tipos admitidos: "GET", "POST", "PUT" | Sí
url | Punto de conexión y ruta de acceso de destino | Cadena (o expresión con un valor resultType de cadena). La actividad dará un error por tiempo de espera después de 1 minuto si no recibe una respuesta desde el punto de conexión. | Sí
headers | Encabezados que se envían a la solicitud. Por ejemplo, para establecer el idioma y el tipo en una solicitud: `"headers" : { "Accept-Language": "en-us", "Content-Type": "application/json" }`. | Cadena (o expresión con un valor resultType de cadena) | Sí, el encabezado Content-type es necesario. `"headers":{ "Content-Type":"application/json"}`
body | Representa la carga útil que se envía al punto de conexión.  | Cadena (o expresión con un valor resultType de cadena). <br/><br/>Vea el esquema de la carga de solicitud en la sección [Solicitar un esquema de carga](#request-payload-schema). | Necesario para los métodos POST o PUT.
autenticación | Método de autenticación usado para llamar al punto de conexión. Los tipos admitidos son "Basic" y "ClientCertificate". Para más información, vea la sección [Autenticación](#authentication). Si la autenticación no es necesaria, excluya esta propiedad. | Cadena (o expresión con un valor resultType de cadena) | No
conjuntos de datos | Lista de conjuntos de datos que se pasan al punto de conexión. | Matriz de referencias de conjunto de datos. Puede ser una matriz vacía. | Sí
linkedServices | Lista de servicios vinculados que se pasan al punto de conexión. | Matriz de referencias de servicios vinculados. Puede ser una matriz vacía. | Sí
connectVia | El [entorno de ejecución de integración](https://docs.microsoft.com/azure/data-factory/concepts-integration-runtime) que se usará para conectarse al almacén de datos. Se puede usar Azure Integration Runtime o un entorno de ejecución de integración autohospedado (si el almacén de datos está en una red privada). Si no se especifica esta propiedad, el servicio usa el valor predeterminado de Azure Integration Runtime. | La referencia al entorno de ejecución de integración. | No 

> [!NOTE]
> Los puntos de conexión REST que invoca la actividad web deben devolver una respuesta de tipo JSON. La actividad dará un error por tiempo de espera después de 1 minuto si no recibe una respuesta desde el punto de conexión.

En la tabla siguiente se enumeran los requisitos del contenido JSON:

| Tipo de valor | Cuerpo de la solicitud | Response body |
|---|---|---|
|Objeto JSON | Compatible. | Compatible. |
|Matriz JSON | Compatible <br/>(En la actualidad, las matrices JSON no funcionan como resultado un error. La corrección está en curso). | No compatible |
| Valor JSON | Compatible | No compatible |
| Tipo distinto de JSON | No compatible | No compatible |
||||

## <a name="authentication"></a>Authentication

A continuación, se muestran los tipos de autenticación admitidos en la actividad web.

### <a name="none"></a>None

Si la autenticación no es necesaria, no incluya la propiedad "authentication".

### <a name="basic"></a>Básico

Especifique el nombre de usuario y la contraseña que se usarán con la autenticación básica.

```json
"authentication":{
   "type":"Basic",
   "username":"****",
   "password":"****"
}
```

### <a name="client-certificate"></a>Certificado de cliente

Especifique un contenido codificado en base64 de un archivo PFX y la contraseña.

```json
"authentication":{
   "type":"ClientCertificate",
   "pfx":"****",
   "password":"****"
}
```

### <a name="managed-identity"></a>Identidad administrada

Especifique el URI de recurso para el que el token de acceso se solicitará utilizando la identidad administrada para la factoría de datos. Para llamar a la API de Azure Resource Management, use `https://management.azure.com/`. Para más información sobre cómo funcionan las identidades administradas, consulte la página de información general [Identidades administradas de recursos de Azure](/azure/active-directory/managed-identities-azure-resources/overview).

```json
"authentication": {
    "type": "MSI",
    "resource": "https://management.azure.com/"
}
```

> [!NOTE]
> Si la factoría de datos está configurada con un repositorio de Git, debe almacenar las credenciales en Azure Key Vault para usar la autenticación de certificado de cliente o básica. Azure Data Factory no almacena contraseñas en Git.

## <a name="request-payload-schema"></a>Solicitar un esquema de carga
Al usar el método POST o PUT, la propiedad body representa la carga que se envía al punto de conexión. Puede pasar servicios vinculados y conjuntos de datos como parte de la carga. Este es el esquema de la carga:

```json
{
    "body": {
        "myMessage": "Sample",
        "datasets": [{
            "name": "MyDataset1",
            "properties": {
                ...
            }
        }],
        "linkedServices": [{
            "name": "MyStorageLinkedService1",
            "properties": {
                ...
            }
        }]
    }
}
```

## <a name="example"></a>Ejemplo
En este ejemplo, la actividad web de la canalización llama a un punto de conexión REST. Pasa un servicio vinculado de Azure SQL y un conjunto de datos de Azure SQL al punto de conexión. El punto de conexión REST usa la cadena de conexión de Azure SQL para conectarse al servidor SQL lógico y devuelve el nombre de la instancia de SQL Server.

### <a name="pipeline-definition"></a>Definición de la canalización

```json
{
    "name": "<MyWebActivityPipeline>",
    "properties": {
        "activities": [
            {
                "name": "<MyWebActivity>",
                "type": "WebActivity",
                "typeProperties": {
                    "method": "Post",
                    "url": "@pipeline().parameters.url",
                    "headers": {
                        "Content-Type": "application/json"
                    },
                    "authentication": {
                        "type": "ClientCertificate",
                        "pfx": "*****",
                        "password": "*****"
                    },
                    "datasets": [
                        {
                            "referenceName": "MySQLDataset",
                            "type": "DatasetReference",
                            "parameters": {
                                "SqlTableName": "@pipeline().parameters.sqlTableName"
                            }
                        }
                    ],
                    "linkedServices": [
                        {
                            "referenceName": "SqlLinkedService",
                            "type": "LinkedServiceReference"
                        }
                    ]
                }
            }
        ],
        "parameters": {
            "sqlTableName": {
                "type": "String"
            },
            "url": {
                "type": "String"
            }
        }
    }
}

```

### <a name="pipeline-parameter-values"></a>Valores de parámetro de canalización

```json
{
    "sqlTableName": "department",
    "url": "https://adftes.azurewebsites.net/api/execute/running"
}

```

### <a name="web-service-endpoint-code"></a>Código de punto de conexión del servicio web

```csharp

[HttpPost]
public HttpResponseMessage Execute(JObject payload)
{
    Trace.TraceInformation("Start Execute");

    JObject result = new JObject();
    result.Add("status", "complete");

    JArray datasets = payload.GetValue("datasets") as JArray;
    result.Add("sinktable", datasets[0]["properties"]["typeProperties"]["tableName"].ToString());

    JArray linkedServices = payload.GetValue("linkedServices") as JArray;
    string connString = linkedServices[0]["properties"]["typeProperties"]["connectionString"].ToString();

    System.Data.SqlClient.SqlConnection sqlConn = new System.Data.SqlClient.SqlConnection(connString);

    result.Add("sinkServer", sqlConn.DataSource);

    Trace.TraceInformation("Stop Execute");

    return this.Request.CreateResponse(HttpStatusCode.OK, result);
}

```

## <a name="next-steps"></a>Pasos siguientes
Consulte otras actividades de flujo de control compatibles con Data Factory:

- [Actividad de ejecución de canalización](control-flow-execute-pipeline-activity.md)
- [Para cada actividad](control-flow-for-each-activity.md)
- [Actividad GetMetadata](control-flow-get-metadata-activity.md)
- [Actividad Lookup](control-flow-lookup-activity.md)
