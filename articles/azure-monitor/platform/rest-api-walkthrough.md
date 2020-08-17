---
title: Tutorial sobre la API de REST de supervisión de Azure
description: Cómo autenticar las solicitudes y usar la API de REST de Azure Monitor para recuperar las definiciones de métricas y valores de métricas disponibles.
ms.subservice: metrics
ms.topic: conceptual
ms.date: 03/19/2018
ms.custom: has-adal-ref
ms.openlocfilehash: 500d5242d5185a8014283918c1f3a22c5c22cf48
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87325600"
---
# <a name="azure-monitoring-rest-api-walkthrough"></a>Tutorial sobre la API de REST de supervisión de Azure

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

En este artículo se describe cómo realizar la autenticación, para que el código pueda usar la [referencia de API de REST de Microsoft Azure Monitor](/rest/api/monitor/).

La API de Azure Monitor permite recuperar mediante programación las definiciones de métricas predeterminadas disponibles, la granularidad y los valores de métricas. Los datos pueden guardarse en un almacén de datos independiente, como Azure SQL Database, Azure Cosmos DB o Azure Data Lake. Desde allí se pueden realizar análisis adicionales según sea necesario.

Además de trabajar con varios puntos de datos de métrica, la API de Monitor también permite enumerar las reglas de alertas, ver registros de actividades y mucho más. Para obtener una lista completa de las operaciones disponibles, vea la [referencia de la API de REST de Microsoft Azure Monitor](/rest/api/monitor/).

## <a name="authenticating-azure-monitor-requests"></a>Autenticación de solicitudes de Azure Monitor

El primer paso consiste en autenticar la solicitud.

Todas las tareas ejecutadas en la API de Azure Monitor utilizan el modelo de autenticación de Azure Resource Manager. Por lo tanto, todas las solicitudes deben autenticarse con Azure Active Directory (Azure AD). Un enfoque para autenticar la aplicación cliente consiste en crear una entidad de servicio de Azure y recuperar el token de autenticación (JWT). El script de ejemplo siguiente muestra cómo crear una entidad de servicio de Azure AD a través de PowerShell. Para obtener un tutorial más detallado, vea la documentación sobre el [uso de Azure PowerShell para crear una entidad de servicio para acceder a recursos](/powershell/azure/create-azure-service-principal-azureps). También se puede [crear una entidad de servicio a través de Azure Portal](../../active-directory/develop/howto-create-service-principal-portal.md).

```powershell
$subscriptionId = "{azure-subscription-id}"
$resourceGroupName = "{resource-group-name}"

# Authenticate to a specific Azure subscription.
Connect-AzAccount -SubscriptionId $subscriptionId

# Password for the service principal
$pwd = "{service-principal-password}"
$secureStringPassword = ConvertTo-SecureString -String $pwd -AsPlainText -Force

# Create a new Azure AD application
$azureAdApplication = New-AzADApplication `
                        -DisplayName "My Azure Monitor" `
                        -HomePage "https://localhost/azure-monitor" `
                        -IdentifierUris "https://localhost/azure-monitor" `
                        -Password $secureStringPassword

# Create a new service principal associated with the designated application
New-AzADServicePrincipal -ApplicationId $azureAdApplication.ApplicationId

# Assign Reader role to the newly created service principal
New-AzRoleAssignment -RoleDefinitionName Reader `
                          -ServicePrincipalName $azureAdApplication.ApplicationId.Guid

```

Para consultar la API de Azure Monitor, la aplicación cliente debe utilizar la entidad de servicio creada anteriormente para la autenticación. El script de PowerShell de ejemplo siguiente muestra un enfoque, basado en utilizar la [Biblioteca de autenticación de Active Directory](../../active-directory/azuread-dev/active-directory-authentication-libraries.md) (ADAL) para obtener el token de autenticación de JWT. El token de JWT se pasa como parte de un parámetro de autorización HTTP en las solicitudes a la API de REST de Azure Monitor.

```powershell
$azureAdApplication = Get-AzADApplication -IdentifierUri "https://localhost/azure-monitor"

$subscription = Get-AzSubscription -SubscriptionId $subscriptionId

$clientId = $azureAdApplication.ApplicationId.Guid
$tenantId = $subscription.TenantId
$authUrl = "https://login.microsoftonline.com/${tenantId}"

$AuthContext = [Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext]$authUrl
$cred = New-Object -TypeName Microsoft.IdentityModel.Clients.ActiveDirectory.ClientCredential -ArgumentList ($clientId, $pwd)

$result = $AuthContext.AcquireTokenAsync("https://management.core.windows.net/", $cred).GetAwaiter().GetResult()

# Build an array of HTTP header values
$authHeader = @{
'Content-Type'='application/json'
'Accept'='application/json'
'Authorization'=$result.CreateAuthorizationHeader()
}
```

Después de la autenticación, las consultas pueden ejecutarse en la API de REST de Azure Monitor. Hay dos consultas útiles:

1. Lista de las definiciones de métricas de un recurso
2. Recuperación de los valores de métrica

> [!NOTE]
> Para obtener más información sobre la autenticación con la API de REST de Azure, consulte la [referencia de la API de REST de Azure](/rest/api/azure/).
>
>

## <a name="retrieve-metric-definitions-multi-dimensional-api"></a>Recuperación de las definiciones de métricas (API multidimensional)

Utilice la [API de REST de definiciones de métricas de Azure Monitor](/rest/api/monitor/metricdefinitions) para acceder a la lista de métricas disponibles para un servicio.

**Método**: GET

**URI de solicitud**: https:\/\/management.azure.com/subscriptions/ *{subscriptionId}* /resourceGroups/ *{resourceGroupName}* /providers/ *{resourceProviderNamespace}* / *{resourceType}* / *{resourceName}* /providers/microsoft.insights/metricDefinitions?api-version= *{apiVersion}*

Por ejemplo, para recuperar las definiciones de métricas para una cuenta de Azure Storage, la solicitud se parecería a la siguiente:

```powershell
$request = "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/microsoft.insights/metricDefinitions?api-version=2018-01-01"

Invoke-RestMethod -Uri $request `
                  -Headers $authHeader `
                  -Method Get `
                  -OutFile ".\contosostorage-metricdef-results.json" `
                  -Verbose

```

> [!NOTE]
> Para recuperar las definiciones de métricas usando la API de REST de Azure Monitor multidimensional, use "2018-01-01" como versión de API.
>
>

El cuerpo de respuesta JSON resultante sería similar al siguiente ejemplo: (Observe que la segunda métrica tiene dimensiones)

```json
{
    "value": [
        {
            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/microsoft.insights/metricdefinitions/UsedCapacity",
            "resourceId": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage",
            "namespace": "Microsoft.Storage/storageAccounts",
            "category": "Capacity",
            "name": {
                "value": "UsedCapacity",
                "localizedValue": "Used capacity"
            },
            "isDimensionRequired": false,
            "unit": "Bytes",
            "primaryAggregationType": "Average",
            "supportedAggregationTypes": [
                "Total",
                "Average",
                "Minimum",
                "Maximum"
            ],
            "metricAvailabilities": [
                {
                    "timeGrain": "PT1H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT6H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT12H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "P1D",
                    "retention": "P93D"
                }
            ]
        },
        {
            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/microsoft.insights/metricdefinitions/Transactions",
            "resourceId": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage",
            "namespace": "Microsoft.Storage/storageAccounts",
            "category": "Transaction",
            "name": {
                "value": "Transactions",
                "localizedValue": "Transactions"
            },
            "isDimensionRequired": false,
            "unit": "Count",
            "primaryAggregationType": "Total",
            "supportedAggregationTypes": [
                "Total"
            ],
            "metricAvailabilities": [
                {
                    "timeGrain": "PT1M",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT5M",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT15M",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT30M",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT1H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT6H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT12H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "P1D",
                    "retention": "P93D"
                }
            ],
            "dimensions": [
                {
                    "value": "ResponseType",
                    "localizedValue": "Response type"
                },
                {
                    "value": "GeoType",
                    "localizedValue": "Geo type"
                },
                {
                    "value": "ApiName",
                    "localizedValue": "API name"
                }
            ]
        },
        ...
    ]
}
```

## <a name="retrieve-dimension-values-multi-dimensional-api"></a>Recuperación de valores de dimensión (API multidimensional)

Una vez que se conocen las definiciones de métricas disponibles, puede haber algunas métricas que tengan dimensiones. Antes de consultar la métrica, puede que desee detectar cuál es el intervalo de valores que tiene una dimensión. Según estos valores de dimensión, a continuación puede elegir filtrar o segmentar las métricas en función de valores de dimensión al realizar la consulta para las métricas.  Use la [API de REST de métricas de Azure Monitor](/rest/api/monitor/metrics) para conseguirlo.

Use el nombre “value” de la métrica (no “localizedValue”) para cualquier solicitud de filtrado. Si no se especifica ningún filtro, se devuelve la métrica predeterminada. El uso de esta API solo permite que una dimensión tenga un filtro de comodín.

> [!NOTE]
> Para recuperar los valores de dimensión usando la API de REST de Azure Monitor, use "2018-01-01" como versión de API.
>
>

**Método**: GET

**URI de solicitud**: https\://management.azure.com/subscriptions/ *{subscription-id}* /resourceGroups/ *{resource-group-name}* /providers/ *{resource-provider-namespace}* / *{resource-type}* / *{resource-name}* /providers/microsoft.insights/metrics?metricnames= *{metric}* &timespan= *{starttime/endtime}* &$filter= *{filter}* &resultType=metadata&api-version= *{apiVersion}*

Por ejemplo, para recuperar la lista de valores de dimensión que se emitieron para la "dimensión de nombre de API" para la métrica "Transactions", donde la dimensión GeoType = "Primary" durante el intervalo de tiempo especificado, la solicitud sería la siguiente:

```powershell
$filter = "APIName eq '*' and GeoType eq 'Primary'"
$request = "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/microsoft.insights/metrics?metricnames=Transactions&timespan=2018-03-01T00:00:00Z/2018-03-02T00:00:00Z&resultType=metadata&`$filter=${filter}&api-version=2018-01-01"
Invoke-RestMethod -Uri $request `
    -Headers $authHeader `
    -Method Get `
    -OutFile ".\contosostorage-dimension-values.json" `
    -Verbose
```

El cuerpo de respuesta JSON resultante sería similar al siguiente ejemplo:

```json
{
  "timespan": "2018-03-01T00:00:00Z/2018-03-02T00:00:00Z",
  "value": [
    {
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/Microsoft.Insights/metrics/Transactions",
      "type": "Microsoft.Insights/metrics",
      "name": {
        "value": "Transactions",
        "localizedValue": "Transactions"
      },
      "unit": "Count",
      "timeseries": [
        {
          "metadatavalues": [
            {
              "name": {
                "value": "apiname",
                "localizedValue": "apiname"
              },
              "value": "DeleteBlob"
            }
          ]
        },
        {
          "metadatavalues": [
            {
              "name": {
                "value": "apiname",
                "localizedValue": "apiname"
              },
              "value": "SetBlobProperties"
            }
          ]
        },
        ...
      ]
    }
  ],
  "namespace": "Microsoft.Storage/storageAccounts",
  "resourceregion": "eastus"
}
```

## <a name="retrieve-metric-values-multi-dimensional-api"></a>Recuperación de valores de métrica (API multidimensional)

Una vez que se conocen las definiciones de métricas disponibles y los posibles valores de dimensión, es posible recuperar los valores de métricas relacionados.  Use la [API de REST de métricas de Azure Monitor](/rest/api/monitor/metrics) para conseguirlo.

Use el nombre “value” de la métrica (no “localizedValue”) para cualquier solicitud de filtrado. Si no se especifica ningún filtro de dimensión, se devuelve la métrica agregada acumulada. Si una consulta de métrica devuelve varias series temporales, puede usar los parámetros de consulta "Top" y "OrderBy" para devolver una lista ordenada limitada de series temporales.

> [!NOTE]
> Para recuperar los valores de métrica multidimensionales usando la API de REST de Azure Monitor, use "2018-01-01" como versión de API.
>
>

**Método**: GET

**Identificador URI de la solicitud**: https:\//management.azure.com/subscriptions/ *{subscription-id}* /resourceGroups/ *{resource-group-name}* /providers/ *{resource-provider-namespace}* / *{resource-type}* / *{resource-name}* /providers/microsoft.insights/metrics?metricnames= *{metric}* &timespan= *{starttime/endtime}* &$filter= *{filter}* &interval= *{timeGrain}* &aggregation= *{aggreation}* &api-version= *{apiVersion}*

Por ejemplo, para recuperar las tres API superiores, en orden descendente de valor, por el número de "Transacciones" durante un intervalo de 5 minutos, donde el valor de GeotType era "Primary", la solicitud sería la siguiente:

```powershell
$filter = "APIName eq '*' and GeoType eq 'Primary'"
$request = "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/microsoft.insights/metrics?metricnames=Transactions&timespan=2018-03-01T02:00:00Z/2018-03-01T02:05:00Z&`$filter=${filter}&interval=PT1M&aggregation=Total&top=3&orderby=Total desc&api-version=2018-01-01"
Invoke-RestMethod -Uri $request `
    -Headers $authHeader `
    -Method Get `
    -OutFile ".\contosostorage-metric-values.json" `
    -Verbose
```

El cuerpo de respuesta JSON resultante sería similar al siguiente ejemplo:

```json
{
  "cost": 0,
  "timespan": "2018-03-01T02:00:00Z/2018-03-01T02:05:00Z",
  "interval": "PT1M",
  "value": [
    {
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/Microsoft.Insights/metrics/Transactions",
      "type": "Microsoft.Insights/metrics",
      "name": {
        "value": "Transactions",
        "localizedValue": "Transactions"
      },
      "unit": "Count",
      "timeseries": [
        {
          "metadatavalues": [
            {
              "name": {
                "value": "apiname",
                "localizedValue": "apiname"
              },
              "value": "GetBlobProperties"
            }
          ],
          "data": [
            {
              "timeStamp": "2017-09-19T02:00:00Z",
              "total": 2
            },
            {
              "timeStamp": "2017-09-19T02:01:00Z",
              "total": 1
            },
            {
              "timeStamp": "2017-09-19T02:02:00Z",
              "total": 3
            },
            {
              "timeStamp": "2017-09-19T02:03:00Z",
              "total": 7
            },
            {
              "timeStamp": "2017-09-19T02:04:00Z",
              "total": 2
            }
          ]
        },
        ...
      ]
    }
  ],
  "namespace": "Microsoft.Storage/storageAccounts",
  "resourceregion": "eastus"
}
```

## <a name="retrieve-metric-definitions"></a>Recuperación de las definiciones de métricas

Utilice la [API de REST de definiciones de métricas de Azure Monitor](/rest/api/monitor/metricdefinitions) para acceder a la lista de métricas disponibles para un servicio.

**Método**: GET

**URI de solicitud**: https:\/\/management.azure.com/subscriptions/ *{subscriptionId}* /resourceGroups/ *{resourceGroupName}* /providers/ *{resourceProviderNamespace}* / *{resourceType}* / *{resourceName}* /providers/microsoft.insights/metricDefinitions?api-version= *{apiVersion}*

Por ejemplo, para recuperar las definiciones de métricas para una aplicación lógica de Azure, la solicitud se parecería a la siguiente:

```powershell
$request = "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/microsoft.insights/metricDefinitions?api-version=2016-03-01"

Invoke-RestMethod -Uri $request `
                  -Headers $authHeader `
                  -Method Get `
                  -OutFile ".\contosotweets-metricdef-results.json" `
                  -Verbose
```

> [!NOTE]
> Para recuperar las definiciones de métricas usando la API de REST de Azure Monitor, use "2016-03-01" como la versión de API.
>
>

El cuerpo de respuesta JSON resultante sería similar al siguiente ejemplo:

```json
{
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/microsoft.insights/metricdefinitions",
  "value": [
    {
      "name": {
        "value": "RunsStarted",
        "localizedValue": "Runs Started"
      },
      "category": "AllMetrics",
      "startTime": "0001-01-01T00:00:00Z",
      "endTime": "0001-01-01T00:00:00Z",
      "unit": "Count",
      "primaryAggregationType": "Total",
      "resourceUri": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets",
      "resourceId": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets",
      "metricAvailabilities": [
        {
          "timeGrain": "PT1M",
          "retention": "P30D",
          "location": null,
          "blobLocation": null
        },
        {
          "timeGrain": "PT1H",
          "retention": "P30D",
          "location": null,
          "blobLocation": null
        }
      ],
      "properties": null,
      "dimensions": null,
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/microsoft.insights/metricdefinitions/RunsStarted",
      "supportedAggregationTypes": [ "None", "Average", "Minimum", "Maximum", "Total", "Count" ]
    }
  ]
}
```

Para obtener más información, vea la documentación sobre la [lista de las definiciones de métricas de un recurso en la API de REST de Azure Monitor](/rest/api/monitor/metricdefinitions) .

## <a name="retrieve-metric-values"></a>Recuperación de los valores de métrica

Una vez que se conocen las definiciones de métricas disponibles, es posible recuperar los valores de métricas relacionados. Utilice el “valor” del nombre de métrica (no “localizedValue”) para todas las solicitudes de filtro (por ejemplo, recuperación de puntos de datos de las métricas de “CpuTime” y “Requests”). Si no se especifica ningún filtro, se devuelve la métrica predeterminada.

> [!NOTE]
> Para recuperar los valores de métricas usando la API de REST de Azure Monitor, use "2016-09-01" como la versión de API.
>
>

**Método**: GET

**URI de solicitud**: `https:\//management.azure.com/subscriptions/\*{subscription-id}*/resourceGroups/*{resource-group-name}*/providers/*{resource-provider-namespace}*/*{resource-type}*/*{resource-name}*/providers/microsoft.insights/metrics?$filter=*{filter}*&api-version=*{apiVersion}*`

Por ejemplo, para recuperar los puntos de datos de métricas RunsSucceeded para el intervalo de tiempo determinado y para un intervalo de agregación de 1 hora, la solicitud sería como sigue:

```powershell
$filter = "(name.value eq 'RunsSucceeded') and aggregationType eq 'Total' and startTime eq 2017-08-18T19:00:00 and endTime eq 2017-08-18T23:00:00 and timeGrain eq duration'PT1H'"
$request = "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/microsoft.insights/metrics?`$filter=${filter}&api-version=2016-09-01"
Invoke-RestMethod -Uri $request `
    -Headers $authHeader `
    -Method Get `
    -OutFile ".\contosotweets-metrics-results.json" `
    -Verbose
```

El cuerpo de respuesta JSON resultante sería similar al siguiente ejemplo:

```json
{
  "value": [
    {
      "data": [
        {
          "timeStamp": "2017-08-18T19:00:00Z",
          "total": 0.0
        },
        {
          "timeStamp": "2017-08-18T20:00:00Z",
          "total": 159.0
        },
        {
          "timeStamp": "2017-08-18T21:00:00Z",
          "total": 174.0
        },
        {
          "timeStamp": "2017-08-18T22:00:00Z",
          "total": 97.0
        }
      ],
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/Microsoft.Insights/metrics/RunsSucceeded",
      "name": {
        "value": "RunsSucceeded",
        "localizedValue": "Runs Succeeded"
      },
      "type": "Microsoft.Insights/metrics",
      "unit": "Count"
    }
  ]
}
```

Para recuperar varios puntos de datos o agregación, agregue los nombres de definición de la métrica y los tipos de agregación para el filtro, tal como se muestra en el ejemplo siguiente:

```powershell
$filter = "(name.value eq 'ActionsCompleted' or name.value eq 'RunsSucceeded') and (aggregationType eq 'Total' or aggregationType eq 'Average') and startTime eq 2017-08-18T21:00:00 and endTime eq 2017-08-18T21:30:00 and timeGrain eq duration'PT1M'"
$request = "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/microsoft.insights/metrics?`$filter=${filter}&api-version=2016-09-01"
Invoke-RestMethod -Uri $request `
    -Headers $authHeader `
    -Method Get `
    -OutFile ".\contosotweets-metrics-multiple-results.json" `
    -Verbose
```

El cuerpo de respuesta JSON resultante sería similar al siguiente ejemplo:

```json
{
  "value": [
    {
      "data": [
        {
          "timeStamp": "2017-08-18T21:03:00Z",
          "total": 5.0,
          "average": 1.0
        },
        {
          "timeStamp": "2017-08-18T21:04:00Z",
          "total": 7.0,
          "average": 1.0
        }
      ],
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/Microsoft.Insights/metrics/ActionsCompleted",
      "name": {
        "value": "ActionsCompleted",
        "localizedValue": "Actions Completed "
      },
      "type": "Microsoft.Insights/metrics",
      "unit": "Count"
    },
    {
      "data": [
        {
          "timeStamp": "2017-08-18T21:03:00Z",
          "total": 5.0,
          "average": 1.0
        },
        {
          "timeStamp": "2017-08-18T21:04:00Z",
          "total": 7.0,
          "average": 1.0
        }
      ],
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/Microsoft.Insights/metrics/RunsSucceeded",
      "name": {
        "value": "RunsSucceeded",
        "localizedValue": "Runs Succeeded"
      },
      "type": "Microsoft.Insights/metrics",
      "unit": "Count"
    }
  ]
}
```

### <a name="use-armclient"></a>Uso de ARMClient

Un enfoque más consiste en utilizar [ARMClient](https://github.com/projectkudu/armclient) en la máquina Windows. ARMClient controla automáticamente la autenticación de Azure AD (y el token JWT resultante). Los siguientes pasos describen el uso de ARMClient para recuperar datos de métricas:

1. Instale [Chocolatey](https://chocolatey.org/) y [ARMClient](https://github.com/projectkudu/armclient).
2. En una ventana de terminal, escriba *inicio de sesión de armclient.exe*. Al hacerlo, le pide que inicie sesión en Azure.
3. Escriba *armclient GET [your_resource_id]/providers/microsoft.insights/metricdefinitions?api-version=2016-03-01*
4. Escriba *armclient GET [your_resource_id]/providers/microsoft.insights/metrics?api-version=2016-09-01*

Por ejemplo, para recuperar las definiciones de métricas para una aplicación lógica específica, ejecute el comando siguiente:

```console
armclient GET /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/microsoft.insights/metricDefinitions?api-version=2016-03-01
```

## <a name="retrieve-the-resource-id"></a>Recuperación del identificador de recursos

Mediante la API de REST, puede ayudarlo a comprender las definiciones de métricas disponibles, la granularidad y los valores relacionados. Dicha información es útil cuando se usa la [biblioteca de administración de Azure](/previous-versions/azure/reference/mt417623(v=azure.100)).

Para el código anterior, el identificador de recurso que se usa es la ruta de acceso completa al recurso de Azure deseado. Por ejemplo, para consultar en una aplicación web de Azure, el identificador de recurso sería:

*/subscriptions/{identificador-suscripción}/resourceGroups/{nombre-grupo-recursos}/providers/Microsoft.Web/sites/{nombre-sitio}/*

La lista siguiente contiene algunos ejemplos de formatos de identificador de recursos para los distintos recursos de Azure:

* **IoT Hub** - /subscriptions/ *{subscription-id}* /resourceGroups/ *{resource-group-name}* /providers/Microsoft.Devices/IotHubs/ *{iot-hub-name}*
* **Grupo elástico SQL** - /subscriptions/ *{subscription-id}* /resourceGroups/ *{resource-group-name}* /providers/Microsoft.Sql/servers/ *{pool-db}* /elasticpools/ *{sql-pool-name}*
* **SQL Database (v12)** - /subscriptions/ *{subscription-id}* /resourceGroups/ *{resource-group-name}* /providers/Microsoft.Sql/servers/ *{server-name}* /databases/ *{database-name}*
* **Service Bus** - /subscriptions/ *{subscription-id}* /resourceGroups/ *{resource-group-name}* /providers/Microsoft.ServiceBus/ *{namespace}* / *{servicebus-name}*
* **Conjunto de escalado de máquinas virtuales**: /subscriptions/ *{subscription-id}* /resourceGroups/ *{resource-group-name}* /providers/Microsoft.Compute/virtualMachineScaleSets/ *{vm-name}*
* **VM** - /subscriptions/ *{subscription-id}* /resourceGroups/ *{resource-group-name}* /providers/Microsoft.Compute/virtualMachines/ *{vm-name}*
* **Event Hubs** - /subscriptions/ *{subscription-id}* /resourceGroups/ *{resource-group-name}* /providers/Microsoft.EventHub/namespaces/ *{eventhub-namespace}*

Hay enfoques alternativos para recuperar el identificador de recurso, incluido el uso del explorador de recursos de Azure, la visualización del recurso deseado en Azure Portal y a través de PowerShell o la CLI de Azure.

### <a name="azure-resource-explorer"></a>Azure Resource Explorer

Para buscar el identificador de recurso para un recurso deseado, un enfoque útil consiste en utilizar el [Explorador de recursos de Azure](https://resources.azure.com) . Vaya al recurso deseado y observe el identificador que se muestra, como en la captura de pantalla siguiente:

![Alt "Explorador de recursos de Azure"](./media/rest-api-walkthrough/azure_resource_explorer.png)

### <a name="azure-portal"></a>Azure portal

El identificador de recurso también puede obtenerse desde Azure Portal. Para ello, desplácese hasta el recurso deseado y luego seleccione Propiedades. El identificador de recurso se muestra en la sección Propiedades, como se muestra en la siguiente captura de pantalla:

![Alt "Identificador de recurso mostrado en la hoja Propiedades de Azure Portal"](./media/rest-api-walkthrough/resourceid_azure_portal.png)

### <a name="azure-powershell"></a>Azure PowerShell

El identificador de recurso también se puede recuperar mediante cmdlets de Azure PowerShell. Por ejemplo, para obtener el identificador de recurso de una aplicación lógica de Azure, ejecute el cmdlet Get-AzureLogicApp, como se muestra en el ejemplo siguiente:

```powershell
Get-AzLogicApp -ResourceGroupName azmon-rest-api-walkthrough -Name contosotweets
```

El resultado debería asemejarse al ejemplo siguiente:

```output
Id             : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets
Name           : ContosoTweets
Type           : Microsoft.Logic/workflows
Location       : centralus
ChangedTime    : 8/21/2017 6:58:57 PM
CreatedTime    : 8/18/2017 7:54:21 PM
AccessEndpoint : https://prod-08.centralus.logic.azure.com:443/workflows/f3a91b352fcc47e6bff989b85446c5db
State          : Enabled
Definition     : {$schema, contentVersion, parameters, triggers...}
Parameters     : {[$connections, Microsoft.Azure.Management.Logic.Models.WorkflowParameter]}
SkuName        :
AppServicePlan :
PlanType       :
PlanId         :
Version        : 08586982649483762729
```

### <a name="azure-cli"></a>Azure CLI

Para recuperar el identificador de recurso para una cuenta de Azure Storage mediante la CLI de Azure, ejecute el comando `az storage account show`, tal como se muestra en el ejemplo siguiente:

```azurecli
az storage account show -g azmon-rest-api-walkthrough -n contosotweets2017
```

El resultado debería asemejarse al ejemplo siguiente:

```json
{
  "accessTier": null,
  "creationTime": "2017-08-18T19:58:41.840552+00:00",
  "customDomain": null,
  "enableHttpsTrafficOnly": false,
  "encryption": null,
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/contosotweets2017",
  "identity": null,
  "kind": "Storage",
  "lastGeoFailoverTime": null,
  "location": "centralus",
  "name": "contosotweets2017",
  "networkAcls": null,
  "primaryEndpoints": {
    "blob": "https://contosotweets2017.blob.core.windows.net/",
    "file": "https://contosotweets2017.file.core.windows.net/",
    "queue": "https://contosotweets2017.queue.core.windows.net/",
    "table": "https://contosotweets2017.table.core.windows.net/"
  },
  "primaryLocation": "centralus",
  "provisioningState": "Succeeded",
  "resourceGroup": "azmon-rest-api-walkthrough",
  "secondaryEndpoints": null,
  "secondaryLocation": "eastus2",
  "sku": {
    "name": "Standard_GRS",
    "tier": "Standard"
  },
  "statusOfPrimary": "available",
  "statusOfSecondary": "available",
  "tags": {},
  "type": "Microsoft.Storage/storageAccounts"
}
```

> [!NOTE]
> Las aplicaciones lógicas de Azure no están aún disponibles a través de la CLI de Azure, por lo tanto, en el ejemplo anterior se muestra una cuenta de Azure Storage.
>
>

## <a name="retrieve-activity-log-data"></a>Recuperación de datos del registro de actividad

Además las definiciones de métricas y los valores relacionados, también es posible usar la API de REST de Azure Monitor para recuperar información adicional interesante relacionada con recursos de Azure. Por ejemplo, es posible consultar los datos del [registro de actividades](/rest/api/monitor/activitylogs) . En las solicitudes de ejemplo siguientes se usa la API REST de Azure Monitor para consultar el registro de actividad.

Obtención de registros de actividad con filtro:

``` HTTP
GET https://management.azure.com/subscriptions/089bd33f-d4ec-47fe-8ba5-0753aa5c5b33/providers/microsoft.insights/eventtypes/management/values?api-version=2015-04-01&$filter=eventTimestamp ge '2018-01-21T20:00:00Z' and eventTimestamp le '2018-01-23T20:00:00Z' and resourceGroupName eq 'MSSupportGroup'
```

Obtención de registros de actividad con filtro y selección:

```HTTP
GET https://management.azure.com/subscriptions/089bd33f-d4ec-47fe-8ba5-0753aa5c5b33/providers/microsoft.insights/eventtypes/management/values?api-version=2015-04-01&$filter=eventTimestamp ge '2015-01-21T20:00:00Z' and eventTimestamp le '2015-01-23T20:00:00Z' and resourceGroupName eq 'MSSupportGroup'&$select=eventName,id,resourceGroupName,resourceProviderName,operationName,status,eventTimestamp,correlationId,submissionTimestamp,level
```

Obtención de registros de actividad con selección:

```HTTP
GET https://management.azure.com/subscriptions/089bd33f-d4ec-47fe-8ba5-0753aa5c5b33/providers/microsoft.insights/eventtypes/management/values?api-version=2015-04-01&$select=eventName,id,resourceGroupName,resourceProviderName,operationName,status,eventTimestamp,correlationId,submissionTimestamp,level
```

Obtención de registros de actividad sin filtro ni selección:

```HTTP
GET https://management.azure.com/subscriptions/089bd33f-d4ec-47fe-8ba5-0753aa5c5b33/providers/microsoft.insights/eventtypes/management/values?api-version=2015-04-01
```

## <a name="next-steps"></a>Pasos siguientes

* Revise la [información general de supervisión](../overview.md).
* Vea las [métricas compatibles con Azure Monitor](metrics-supported.md).
* Revise la [referencia de la API de REST de Microsoft Azure Monitor](/rest/api/monitor/).
* Revise la [biblioteca de administración de Azure](/previous-versions/azure/reference/mt417623(v=azure.100)).

