---
title: Cuentas de almacenamiento de propiedad del cliente para la ingesta de registros
description: Use su propia cuenta de almacenamiento para la ingesta de datos de registro en un área de trabajo de Log Analytics en Azure Monitor.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/20/2020
ms.openlocfilehash: a14f7ca3e5a7b291e430db6ea536edc5396b5448
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318902"
---
# <a name="customer-owned-storage-accounts-for-log-ingestion-in-azure-monitor"></a>Cuentas de almacenamiento de propiedad del cliente para la ingesta de registros en Azure Monitor

Azure Monitor usa cuentas de almacenamiento en el proceso de ingesta de algunos tipos de datos tales como [registros personalizados](data-sources-custom-logs.md) y algunos [registros de Azure](./diagnostics-extension-logs.md). Durante el proceso de ingesta, los registros se envían primero a una cuenta de almacenamiento y, posteriormente, se ingieren en Log Analytics o Application Insights. Si quiere controlar los datos durante la ingesta, puede usar sus propias cuentas de almacenamiento en lugar del almacenamiento administrado por el servicio. El uso de su propia cuenta de almacenamiento le proporciona control sobre el acceso, el contenido, el cifrado y la retención de los registros durante la ingesta. Llamamos a esto Traiga su propio almacenamiento (BYOS). 

Un escenario que requiere BYOS es el aislamiento de red a través de vínculos privados. Cuando se usa una red virtual, el aislamiento de red suele ser un requisito y el acceso a Internet público es limitado. En tales casos, el acceso al almacenamiento de servicio de Azure Monitor para la ingesta de registros está completamente bloqueado o se considera como procedimiento incorrecto. En su lugar, los registros se deben ingerir a través de una cuenta de almacenamiento propiedad del cliente dentro de la red virtual o fácilmente accesible desde ella.

Otro escenario es el cifrado de los registros con claves administradas por el cliente (CMK). Los clientes pueden cifrar los datos registrados mediante CMK en los clústeres que almacenan los registros. También se puede usar la misma clave para cifrar los registros durante el proceso de ingesta.

## <a name="data-types-supported"></a>Tipos de datos compatibles

Entre los tipos de datos que se ingieren desde una cuenta de almacenamiento se incluyen los siguientes. Consulte [Recopilación de datos de Azure Diagnostics Extension en los registros de Azure Monitor](./diagnostics-extension-logs.md) para más información sobre la ingesta de estos tipos.

| Tipo | Información de tabla |
|:-----|:------------------|
| Registros IIS | Blob: wad-iis-logfiles|
|Registros de eventos de Windows | Tabla: WADWindowsEventLogsTable |
| syslog | Tabla: LinuxsyslogVer2v0 |
| Registros de ETW de Windows | Tabla: WADETWEventTable|
| Service Fabric | Tabla: WADServiceFabricSystemEventTable <br/> WADServiceFabricReliableActorEventTable<br/> WADServiceFabricReliableServicEventTable |
| Registros personalizados | N/D |
| Archivos de volcado de memoria de Watson de Azure Security Center | N/D|  

## <a name="storage-account-requirements"></a>Requisitos de la cuenta de almacenamiento 
La cuenta de almacenamiento debe cumplir los requisitos siguientes:

- Accesible para los recursos de la red virtual que escriben registros en el almacenamiento.
- Debe estar en la misma región que el área de trabajo a la que está vinculada.
- Permitir acceso de Azure Monitor: si eligió limitar el acceso de la cuenta de almacenamiento a redes seleccionadas, asegúrese de permitir esta excepción: *permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento*.

## <a name="process-to-configure-customer-owned-storage"></a>Proceso para configurar el almacenamiento propiedad del cliente
El proceso básico de usar su propia cuenta de almacenamiento para la ingesta es el siguiente:

1. Cree una cuenta de almacenamiento o seleccione una cuenta existente.
2. Vincule la cuenta de almacenamiento a un área de trabajo de Log Analytics.
3. Administre el almacenamiento revisando su carga y retención para asegurarse de que funciona según lo previsto.

El único método disponible para crear y quitar vínculos es a través de la API REST. En las secciones siguientes se proporcionan detalles sobre la solicitud de API específica necesaria para cada proceso.

## <a name="command-line-and-rest-api"></a>Línea de comandos y API de REST

### <a name="command-line"></a>Línea de comandos
Para crear y administrar cuentas de almacenamiento vinculadas, use [az monitor log-analytics workspace linked-storage](/cli/azure/monitor/log-analytics/workspace/linked-storage). Este comando puede vincular y desvincular cuentas de almacenamiento de un área de trabajo y enumerar las cuentas de almacenamiento vinculadas.

### <a name="request-and-cli-values"></a>Valores de la solicitud y la CLI

#### <a name="datasourcetype"></a>dataSourceType 

- AzureWatson: use este valor para los archivos de volcado de memoria de Watson de Azure Security Center.
- CustomLogs: use este valor para estos tipos de datos:
  - Registros personalizados
  - Registros IIS
  - Eventos (Windows)
  - Syslog (Linux)
  - Registros de ETW
  - Eventos de Service Fabric
  - Datos de evaluación  

#### <a name="storage_account_resource_id"></a>storage_account_resource_id
Este valor usa la estructura siguiente:

```
subscriptions/{subscriptionId}/resourcesGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName1}
```


### <a name="get-linked-storage-accounts-for-all-data-source-types"></a>Obtención de cuentas de almacenamiento vinculadas para todos los tipos de origen de datos

#### <a name="api-request"></a>Solicitud a la API

```
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{workspaceName}/linkedStorageAccounts?api-version=2019-08-01-preview  
```

#### <a name="response"></a>Response

```json
{
    [
        {
            "properties":
            {
                "dataSourceType": "CustomLogs",
                "storageAccountIds  ": 
                [  
                    "<storage_account_resource_id_1>",
                    "<storage_account_resource_id_2>"
                ],
            },
            "id":"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/microsoft. operationalinsights/workspaces/{resourceName}/linkedStorageAccounts/CustomLogs",
            "name": "CustomLogs",
            "type": "Microsoft.OperationalInsights/workspaces/linkedStorageAccounts"
        },
        {
            "properties":
            {
                "dataSourceType": " AzureWatson "
                "storageAccountIds  ": 
                [  
                    "<storage_account_resource_id_3>",
                    "<storage_account_resource_id_4>"
                ],
            },
            "id":"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/microsoft. operationalinsights/workspaces/{resourceName}/linkedStorageAccounts/AzureWatson",
            "name": "AzureWatson",
            "type": "Microsoft.OperationalInsights/workspaces/linkedStorageAccounts"
        }
    ]
}
```


### <a name="get-linked-storage-accounts-for-a-specific-data-source-type"></a>Obtención de cuentas de almacenamiento vinculadas para un tipo de origen de datos específico

#### <a name="api-request"></a>Solicitud a la API

```
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{workspaceName}/linkedStorageAccounts/{dataSourceType}?api-version=2019-08-01-preview  
```

#### <a name="response"></a>Response 

```json
{
    "properties":
    {
        "dataSourceType": "CustomLogs",
        "storageAccountIds  ": 
        [  
            "<storage_account_resource_id_1>",
            "<storage_account_resource_id_2>"
        ],
    },
    "id":"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/microsoft. operationalinsights/workspaces/{resourceName}/linkedStorageAccounts/CustomLogs",
    "name": "CustomLogs",
    "type": "Microsoft.OperationalInsights/workspaces/linkedStorageAccounts"
}
```

## <a name="create-or-modify-a-link"></a>Creación o modificación de un vínculo

Una vez que vincule una cuenta de almacenamiento a un área de trabajo, Log Analytics comenzará a usarla en lugar de la cuenta de almacenamiento propiedad del servicio. Puede registrar una lista de cuentas de almacenamiento al mismo tiempo y puede usar la misma cuenta de almacenamiento para varias áreas de trabajo.

Si el área de trabajo controla tanto los recursos de red virtual como los recursos fuera de una red virtual, debe asegurarse de que no rechaza el tráfico procedente de Internet. El almacenamiento debe tener la misma configuración que el área de trabajo y estar disponible para los recursos fuera de la red virtual. 

### <a name="api-request"></a>Solicitud a la API

```
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{workspaceName}/linkedStorageAccounts/{dataSourceType}?api-version=2019-08-01-preview  
```

### <a name="payload"></a>Carga

```json
{
    "properties":
    {
        "storageAccountIds  " : 
        [  
            "<storage_account_resource_id_1>",
            "<storage_account_resource_id_2>"
        ],
    }
}
```

### <a name="response"></a>Response

```json
{
    "properties":
    {
        "dataSourceType": "CustomLogs"
        "storageAccountIds  ": 
        [  
            "<storage_account_resource_id_1>",
            "<storage_account_resource_id_2>"
        ],
    },
"id":"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/microsoft. operationalinsights/workspaces/{resourceName}/linkedStorageAccounts/CustomLogs",
"name": "CustomLogs",
"type": "Microsoft.OperationalInsights/workspaces/linkedStorageAccounts"
}
```


## <a name="unlink-a-storage-account"></a>Desvinculación de una cuenta de almacenamiento
Si decide dejar de usar una cuenta de almacenamiento para la ingesta o reemplazar el área de trabajo que utiliza, debe desvincular el almacenamiento del área de trabajo.

Al desvincular todas las cuentas de almacenamiento de un área de trabajo, la ingesta intentará basarse en las cuentas de almacenamiento administradas por el servicio. Si los agentes se ejecutan en una red virtual con acceso limitado a Internet, se espera que se produzca un error en la ingesta. El área de trabajo debe tener una cuenta de almacenamiento vinculada que sea accesible desde los recursos supervisados.

Antes de eliminar una cuenta de almacenamiento, debe asegurarse de que todos los datos que contiene se han ingerido en el área de trabajo. Como medida de precaución, mantenga su cuenta de almacenamiento disponible después de vincular un almacenamiento alternativo. Elimínela solo una vez que se haya ingerido todo su contenido y podrá ver que los datos nuevos se escriben en la cuenta de almacenamiento recién conectada.


### <a name="api-request"></a>Solicitud a la API
```
DELETE https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{workspaceName}/linkedStorageAccounts/{dataSourceType}?api-version=2019-08-01-preview  
```

## <a name="replace-a-storage-account"></a>Reemplazo de una cuenta de almacenamiento

Para reemplazar una cuenta de almacenamiento usada para la ingesta, primero debe crear un vínculo para una cuenta de almacenamiento nueva. Los agentes de registro obtendrán la configuración actualizada y comenzarán a enviar datos también al almacenamiento nuevo.

Después, desvincule la cuenta de almacenamiento anterior para que los agentes dejen de escribir en la cuenta quitada. El proceso de ingesta sigue leyendo los datos de esta cuenta hasta que se realice la ingesta de todos. No elimine la cuenta de almacenamiento hasta que vea que se ingirieron todos los registros.

La configuración del agente se actualizará después de unos minutos y cambiarán al almacenamiento nuevo.

## <a name="manage-storage-account"></a>Administración de una cuenta de almacenamiento

### <a name="load"></a>Cargar

Las cuentas de almacenamiento pueden controlar cierta carga de solicitudes de lectura y escritura antes de iniciar las solicitudes de limitación. La limitación afecta al tiempo que se tarda en ingerir registros y puede dar lugar a la pérdida de datos. Si el almacenamiento está sobrecargado, registre cuentas de almacenamiento adicionales y reparta la carga entre ellos. 

### <a name="related-charges"></a>Cargos relacionados

Las cuentas de almacenamiento se cobran según el volumen de datos almacenados, los tipos de almacenamiento y el tipo de redundancia. Para detalles, consulte [Precios de los blobs en bloques](https://azure.microsoft.com/pricing/details/storage/blobs/) y [Precios de Azure Table Storage](https://azure.microsoft.com/pricing/details/storage/tables/).

Si la cuenta de almacenamiento registrada del área de trabajo está en otra región, se le cobrará por la salida según estos [Detalles de precios de ancho de banda](https://azure.microsoft.com/pricing/details/bandwidth/).



## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre cómo configurar un vínculo privado, consulte [Uso de Azure Private Link para conectar redes a Azure Monitor de forma segura](private-link-security.md).

