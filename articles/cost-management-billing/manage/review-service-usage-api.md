---
title: Revisión del uso de recursos de servicios de Azure con la API REST
description: Aprenda a utilizar las API REST de Azure para consultar el uso de recursos por parte de los servicios de Azure. Cree un informe de administración de costos y fíltrelo según los tipos de recursos específicos.
author: lleonard-msft
ms.service: cost-management-billing
ms.subservice: enterprise
ms.topic: reference
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: a7b06ff41e537513558e7f4dc8e1732966299b01
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88684735"
---
# <a name="review-azure-resource-usage-using-the-rest-api"></a>Consulta del uso de recursos de Azure con la API REST

Azure Cost Management API le ayuda a consultar y administrar el consumo de los recursos de Azure.

En este artículo, aprenderá a crear un informe diario que generará un documento de valores separados por comas con los datos de uso por hora. También aprenderá a utilizar filtros para personalizar el informe de forma que pueda consultar el uso de las máquinas virtuales, las bases de datos y los recursos etiquetados de un grupo de recursos de Azure.

>[!NOTE]
> Actualmente, Cost Management API está disponible como versión preliminar privada.

## <a name="create-a-basic-cost-management-report"></a>Creación de un informe básico sobre administración de costos

Utilice la operación `reports` de Cost Management API para especificar cómo se van a generar los informes de costos y dónde se van a publicar.

```http
https://management.azure.com/subscriptions/{subscriptionGuid}/providers/Microsoft.CostManagement/reports/{reportName}?api-version=2018-09-01-preview
Content-Type: application/json   
Authorization: Bearer
```

El parámetro `{subscriptionGuid}` es obligatorio y debe contener un identificador de suscripción que pueda leerse con las credenciales proporcionadas en el token de la API. 

El parámetro `{reportName}` especifica el nombre del informe. Para obtener una lista de nombres de informe, puede usar la operación Reports_List: `/subscriptions/{subscriptionId}/providers/Microsoft.CostManagement/reports`. Vea el resultado de ejemplo en [GitHub](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/cost-management/resource-manager/Microsoft.CostManagement/preview/2018-08-01-preview/examples/ReportList.json).

Los siguientes encabezados son obligatorios:

|Encabezado de solicitud|Descripción|  
|--------------------|-----------------|  
|*Content-Type:*| Necesario. Establézcalo en `application/json`. |  
|*Authorization:*| Necesario. Debe establecerse en un token `Bearer` válido. |

Configure los parámetros del informe en el cuerpo de la solicitud HTTP. En el ejemplo siguiente, el informe se ha configurado para que se genere cada día cuando esté activo, es un archivo CSV que se escribe en un contenedor de blobs de Azure Storage y contiene información sobre los costos por hora de todos los recursos del grupo de recursos `westus`.

```json
{
    "properties": {
        "schedule": {
            "status": "Inactive",
            "recurrence": "Daily",
            "recurrencePeriod": {
                "from": "2018-08-21",
                "to": "2019-10-31"
            }
        },
        "deliveryInfo": {
            "destination": {
                "resourceId": "/subscriptions/{subscriptionGuid}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}",
                "container": "MyReportContainer",
                "rootFolderPath": "MyScheduleTest"
            }
        },
        "format": "Csv",
        "definition": {
            "type": "Usage",
            "timeframe": "MonthToDate",
            "dataSet": {
                "granularity": "Hourly",
                "filter": {
                    "dimensions": {
                        "name": "ResourceLocation",
                        "operator": "In",
                        "values": [
                            "westus"
                        ]
                    }
                }
            }
        }
    }
}
```

Tenga en cuenta que

## <a name="filtering-reports"></a>Filtrado de informes

Cuando se crea un informe, la sección `filter` y `dimensions` del cuerpo de la solicitud permiten concentrarse en los costos de ciertos tipos de recursos. En el cuerpo de la solicitud anterior, se muestra cómo se puede filtrar por todos los recursos de una región.

### <a name="get-all-compute-usage"></a>Consulta de toda la información sobre el uso de procesos

Utilice la dimensión `ResourceType` para crear informes sobre los costos de las máquinas virtuales de Azure generados en su suscripción en todas las regiones.

```json
"filter": {
    "dimensions": {
        "name": "ResourceType",
        "operator": "In",
        "values": [
                "Microsoft.ClassicCompute/virtualMachines",
                "Microsoft.Compute/virtualMachines"
        ]
    }
}
```

### <a name="get-all-database-usage"></a>Consulta de toda la información sobre el uso de bases de datos

Utilice la dimensión `ResourceType` para crear informes sobre los costos de Azure SQL Database generados en su suscripción en todas las regiones.

```json
"filter": {
    "dimensions": {
        "name": "ResourceType",
        "operator": "In",
        "values": [
                "Microsoft.Sql/servers"
        ]
    }
}
```

### <a name="report-on-specific-instances"></a>Informes sobre instancias específicas

La dimensión `Resource` le permite crear informes sobre los costos de determinados recursos.

```json
"filter": {
    "dimensions": {
        "name": "Resource",
        "operator": "In",
        "values": [
            "subscriptions/{subscriptionGuid}/resourceGroups/{resourceGroup}/providers/Microsoft.ClassicCompute/virtualMachines/{ResourceName}"
        ]
    }
}
```

### <a name="changing-timeframes"></a>Modificación de los períodos de tiempo

Establezca la definición de `timeframe` en `Custom` para definir un período de tiempo diferente al de las opciones integradas "Semana hasta la fecha" y "Mes hasta la fecha".

```json
"timeframe": "Custom",
"timePeriod": {
    "from": "2017-12-31T00:00:00.000Z",
    "to": "2018-12-30T00:00:00.000Z"
}
```

## <a name="next-steps"></a>Pasos siguientes
- [Get started with Azure REST API](https://docs.microsoft.com/rest/api/azure/) (Introducción a la API REST de Azure)   
