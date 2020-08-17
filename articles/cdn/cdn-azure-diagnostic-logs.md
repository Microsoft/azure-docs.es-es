---
title: Registros de diagnóstico
titleSuffix: Azure Content Delivery Network
description: Los clientes pueden habilitar el análisis de registros para Azure CDN.
services: cdn
author: asudbring
manager: KumudD
ms.assetid: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 07/15/2020
ms.author: allensu
ms.openlocfilehash: dbaba67a163bb0f948de5ba2ebbdba5497ad5ff9
ms.sourcegitcommit: 0820c743038459a218c40ecfb6f60d12cbf538b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87116978"
---
# <a name="diagnostic-logs---azure-content-delivery-network"></a>Registros de diagnóstico: Azure Content Delivery Network

Con los registros de Azure Diagnostics, puede ver análisis básicos y guardarlos en uno o varios destino, por ejemplo:

* Cuenta de Azure Storage
* Área de trabajo de Log Analytics
* Azure Event Hubs

Esta característica está disponible en puntos de conexión de CDN de todos los planes de tarifa. 

Los registros de diagnóstico le permiten exportar métricas básicas de uso desde su punto de conexión de CDN a distintos tipos de orígenes, de modo que pueda consumirlas de forma personalizada. Puede realizar los siguientes tipos de exportación de datos:

* Exportar datos a Blob Storage, exportar a CSV y generar gráficos en Excel.
* Exportar datos a Event Hubs y correlacionarlos con los datos de otros servicios de Azure.
* Exportar datos a los registros de Azure Monitor y verlos en su propia área de trabajo de Log Analytics

Se requiere un perfil de Azure CDN para los pasos siguientes. Consulte la [creación de un perfil y un punto de conexión de Azure CDN](cdn-create-new-endpoint.md) antes de continuar.

## <a name="enable-logging-with-the-azure-portal"></a>Habilitar el registro con Azure Portal

Siga estos pasos para habilitar el registro para el punto de conexión de Azure CDN:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 

2. En Azure Portal, vaya a **Todos los recursos** -> **su-perfil-de-cdn**.

2. Seleccione el punto de conexión de CDN para el que quiera habilitar los registros de diagnóstico:

    :::image type="content" source="./media/cdn-diagnostics-log/02_browse-to-diagnostics-logs.png" alt-text="Selección del punto de conexión de CDN." border="true":::

3. Seleccione **Registros de diagnóstico** en la sección **Supervisión**:

    :::image type="content" source="./media/cdn-diagnostics-log/03_diagnostics-logs-options.png" alt-text="Selección de registros de diagnósticos." border="true":::

### <a name="enable-logging-with-azure-storage"></a>Habilitación del registro con Azure Storage

Para usar una cuenta de almacenamiento y almacenar los registros, siga estos pasos:

 >[!NOTE] 
 >Se necesita una cuenta de almacenamiento para completar estos pasos. Consulte **[Creación de una cuenta de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-account-create?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json&tabs=azure-portal)** para obtener más información.
    
1. En **Diagnostic setting name** (Nombre de la configuración de diagnóstico), escriba un nombre para la configuración del registro de diagnóstico.
 
2. Seleccione **Archivar en una cuenta de almacenamiento** y luego seleccione **CoreAnalytics**. 

3. En **Retención (días)** , elija el número de días de retención. Con una retención de cero días, los registros se almacenan indefinidamente. 

4. Seleccione la suscripción y la cuenta de almacenamiento para los registros.

    :::image type="content" source="./media/cdn-diagnostics-log/04_diagnostics-logs-storage.png" alt-text="Registros de diagnóstico: almacenamiento." border="true":::

3. Seleccione **Guardar**.

### <a name="send-to-log-analytics"></a>Enviar a Log Analytics

Para usar Log Analytics para los registros, siga estos pasos:

>[!NOTE] 
>Se requiere un área de trabajo de Log Analytics para completar estos pasos. Consulte **[Creación de un área de trabajo de Log Analytics en Azure Portal](https://docs.microsoft.com/azure/azure-monitor/learn/quick-create-workspace)** para obtener más información.
    
1. En **Diagnostic setting name** (Nombre de la configuración de diagnóstico), escriba un nombre para la configuración del registro de diagnóstico.

2. Seleccione **Enviar a Log Analytics** y luego seleccione **CoreAnalytics**. 

3. Seleccione la suscripción y el área de trabajo de Log Analytics para los registros.

   :::image type="content" source="./media/cdn-diagnostics-log/05-la-workspace.png" alt-text="Registros de diagnóstico: Log Analytics." border="true":::

4. Seleccione **Guardar**.

### <a name="stream-to-an-event-hub"></a>Transmitir a un centro de eventos

Para usar un centro de eventos para los registros, siga estos pasos:

>[!NOTE] 
>Se necesita un centro de eventos para completar estos pasos. Consulte **[Guía de inicio rápido: Creación de un centro de eventos mediante Azure Portal](https://docs.microsoft.com/azure/event-hubs/event-hubs-create)** para obtener más información.
    
1. En **Diagnostic setting name** (Nombre de la configuración de diagnóstico), escriba un nombre para la configuración del registro de diagnóstico.

2. Seleccione **Transmitir a un centro de eventos** y luego **CoreAnalytics**. 

3. Seleccione la suscripción y el espacio de nombres del centro de eventos para los registros.

   :::image type="content" source="./media/cdn-diagnostics-log/06-eventhub-namespace.png" alt-text="Registros de diagnóstico: centro de eventos" border="true":::

4. Seleccione **Guardar**.


## <a name="enable-logging-with-powershell"></a>Habilitación del registro con PowerShell

En los ejemplos siguientes se muestra cómo habilitar registros de diagnóstico mediante los cmdlets de Azure PowerShell.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="enable-diagnostic-logs-in-a-storage-account"></a>Habilitación de registros de diagnóstico en una cuenta de almacenamiento

1. Inicie sesión en Azure PowerShell:

    ```azurepowershell-interactive
    Connect-AzAccount 
    ```

2. Para habilitar los registros de diagnóstico en una cuenta de almacenamiento, escriba estos comandos. Reemplace las variables por sus valores:

    ```azurepowershell-interactive
    $rsg = <your-resource-group-name>
    $cdnprofile = <your-cdn-profile-name>
    $cdnendpoint = <your-cdn-endpoint-name>
    $storageacct = <your-storage-account-name>
    $diagname = <your-diagnostic-setting-name>

    $cdn = Get-AzCdnEndpoint -ResourceGroupName $rsg -ProfileName $cdnprofile -EndpointName $cdnendpoint

    $storage = Get-AzStorageAccount -ResourceGroupName $rsg -Name $storageacct

    Set-AzDiagnosticSetting -Name $diagname -ResourceId $cdn.id -StorageAccountId $storage.id -Enabled $true -Categories CoreAnalytics
    ```

### <a name="enable-diagnostics-logs-for-log-analytics-workspace"></a>Habilitación de registros de diagnóstico en el área de trabajo de Log Analytics

1. Inicie sesión en Azure PowerShell:

    ```azurepowershell-interactive
    Connect-AzAccount 
    ```
2. Para habilitar los registros de diagnóstico en un área de trabajo de Log Analytics, escriba estos comandos. Reemplace las variables por sus valores:

    ```azurepowershell-interactive
    $rsg = <your-resource-group-name>
    $cdnprofile = <your-cdn-profile-name>
    $cdnendpoint = <your-cdn-endpoint-name>
    $workspacename = <your-log-analytics-workspace-name>
    $diagname = <your-diagnostic-setting-name>

    $cdn = Get-AzCdnEndpoint -ResourceGroupName $rsg -ProfileName $cdnprofile -EndpointName $cdnendpoint

    $workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName $rsg -Name $workspacename

    Set-AzDiagnosticSetting -Name $diagname -ResourceId $cdn.id -WorkspaceId $workspace.ResourceId -Enabled $true -Categories CoreAnalytics
    ```
### <a name="enable-diagnostics-logs-for-event-hub-namespace"></a>Habilitación de registros de diagnóstico en un espacio de nombres del centro de eventos

1. Inicie sesión en Azure PowerShell:

    ```azurepowershell-interactive
    Connect-AzAccount 
    ```
2. Para habilitar los registros de diagnóstico en un área de trabajo de Log Analytics, escriba estos comandos. Reemplace las variables por sus valores:

    ```azurepowershell-interactive
    $rsg = <your-resource-group-name>
    $cdnprofile = <your-cdn-profile-name>
    $cdnendpoint = <your-cdn-endpoint-name>
    $eventhubname = <your-event-hub-namespace-name>
    $diagname = <your-diagnostic-setting-name>

    $cdn = Get-AzCdnEndpoint -ResourceGroupName $rsg -ProfileName $cdnprofile -EndpointName $cdnendpoint

    Set-AzDiagnosticSetting -Name $diagname -ResourceId $cdn.id -EventHubName $eventhubname -Enabled $true -Categories CoreAnalytics
    ```

## <a name="consuming-diagnostics-logs-from-azure-storage"></a>Consumo de registros de diagnósticos desde Azure Storage
En esta sección se describe el esquema del análisis básico de CDN, la organización en una cuenta de Azure Storage, y se proporciona código de ejemplo para descargar los registros en un archivo CSV.

### <a name="using-microsoft-azure-storage-explorer"></a>Uso del Explorador de Microsoft Azure Storage
Para descargar la herramienta, consulte [Explorador de Azure Storage](https://storageexplorer.com/). Después de descargar e instalar el software, configúrelo para usar la misma cuenta de Azure Storage que configuró como destino para los registros de diagnóstico de la red CDN.

1.  Abra el **Explorador de Microsoft Azure Storage**
2.  Busque la cuenta de almacenamiento.
3.  Expanda el nodo **Contenedores de blob** en esta cuenta de almacenamiento.
4.  Seleccione el contenedor llamado *insights-logs-coreanalytics*.
5.  Los resultados se muestran en el panel derecho, comenzando por el primer nivel, como *resourceId=* . Siga seleccionando cada nivel hasta encontrar el archivo *PT1H.json*. Para obtener una explicación de la ruta de acceso, consulte [Formato de las rutas de acceso de blob](cdn-azure-diagnostic-logs.md#blob-path-format).
6.  Cada archivo blob *PT1H.json* representa los registros de análisis durante una hora de un punto de conexión de red CDN concreto o de su dominio personalizado.
7.  El esquema del contenido de este archivo JSON se describe en la sección de esquema de los registros de análisis básico.


#### <a name="blob-path-format"></a>Formato de las rutas de acceso de blob

Los registros de análisis básico se generan cada hora y los datos se recopilan y almacenan en un único blob de Azure como una carga JSON. La herramienta Explorador de almacenamiento interpreta "/" como un separador de directorio y muestra la jerarquía. La ruta de acceso a este blob de Azure aparece como si hubiera una estructura jerárquica y representa el nombre del blob. Este nombre del blob sigue la convención de nomenclatura siguiente:    

```resourceId=/SUBSCRIPTIONS/{Subscription Id}/RESOURCEGROUPS/{Resource Group Name}/PROVIDERS/MICROSOFT.CDN/PROFILES/{Profile Name}/ENDPOINTS/{Endpoint Name}/ y={Year}/m={Month}/d={Day}/h={Hour}/m={Minutes}/PT1H.json```

**Descripción de los campos:**

|Value|Descripción|
|-------|---------|
|Id. de suscripción    |Identificador de la suscripción de Azure en formato GUID.|
|Nombre del grupo de recursos |Nombre del grupo de recursos al que pertenecen los recursos de CDN.|
|Nombre del perfil |Nombre del perfil de CDN|
|Nombre del punto de conexión |Nombre del punto de conexión de CDN|
|Year|  Representación del año en formato de cuatro dígitos, por ejemplo, 2017|
|Month| Representación del mes en formato de dos dígitos. 01=enero ... 12=diciembre|
|Día|   Representación del día del mes en formato de dos dígitos|
|PT1H.json| Archivo JSON real donde se almacenan los datos de análisis|

### <a name="exporting-the-core-analytics-data-to-a-csv-file"></a>Exportación de los datos de análisis básico a un archivo CSV

Para acceder al análisis básico, se proporciona código de ejemplo para una herramienta. Esta herramienta permite descargar los archivos JSON en un formato de archivo plano separado por comas que se puede usar para crear gráficos y otras agregaciones.

A continuación, se muestra cómo puede usar la herramienta:

1.  Visite el vínculo de GitHub: [https://github.com/Azure-Samples/azure-cdn-samples/tree/master/CoreAnalytics-ExportToCsv](https://github.com/Azure-Samples/azure-cdn-samples/tree/master/CoreAnalytics-ExportToCsv)
2.  Descargue el código.
3.  Siga las instrucciones para compilarlo y configurarlo.
4.  Ejecute la herramienta.
5.  El archivo CSV resultante muestra los datos de análisis en una jerarquía sencilla plana.

## <a name="log-data-delays"></a>Retrasos en el registro de datos

En la tabla siguiente se muestran los retrasos de los datos de registro de **Azure CDN Estándar de Microsoft**, **Azure CDN Estándar de Akamai** y **Azure CDN Estándar/Premium de Verizon**.

Retrasos de datos de registro de Microsoft | Retrasos en el registro de datos de Verizon | Retrasos en el registro de datos de Akamai
--- | --- | ---
Retraso de 1 hora. | Se retrasan 1 hora y pueden tardar hasta 2 horas en comenzar a aparecer tras la finalización de la propagación del punto de conexión. | Se retrasan 24 horas, y tardan 2 horas en comenzar a aparecer si se crearon hace más de 24 horas. Si se acaban de crear, los registros tardan en comenzar a aparecer hasta 25 horas.

## <a name="diagnostic-log-types-for-cdn-core-analytics"></a>Tipos de registro de diagnóstico para análisis básico de la red CDN

Actualmente, Microsoft solo ofrece los registros de análisis básico que contienen métricas que muestran estadísticas de respuesta HTTP y estadísticas de salida, como se ven desde los servidores POP y perimetrales de la red CDN.

### <a name="core-analytics-metrics-details"></a>Detalles de las métricas de análisis básico
En la tabla siguiente se muestra una lista de métricas disponibles en los registros de análisis básico para:

* **Azure CDN Estándar de Microsoft**
* **Azure CDN Estándar de Akamai**
* **Azure CDN Estándar/Premium de Verizon**

No todas las métricas están disponibles en todos los proveedores, si bien tales diferencias son mínimas. En la tabla también se muestra si un proveedor dispone de una métrica determinada. Las métricas solo están disponibles para esos puntos de conexión de red CDN que contienen tráfico.


|Métrica                     | Descripción | Microsoft | Verizon | Akamai |
|---------------------------|-------------|-----------|---------|--------|
| RequestCountTotal         | Número total de aciertos de solicitud durante este periodo. | Sí | Sí |Sí |
| RequestCountHttpStatus2xx | Recuento de todas las solicitudes que dieron lugar a un código HTTP 2xx (por ejemplo, 200, 202). | Sí | Sí |Sí |
| RequestCountHttpStatus3xx | Recuento de todas las solicitudes que dieron lugar a un código HTTP 3xx (por ejemplo, 300, 302). | Sí | Sí |Sí |
| RequestCountHttpStatus4xx | Recuento de todas las solicitudes que dieron lugar a un código HTTP 4xx (por ejemplo, 400, 404). | Sí | Sí |Sí |
| RequestCountHttpStatus5xx | Recuento de todas las solicitudes que dieron lugar a un código HTTP 5xx (por ejemplo, 500, 504). | Sí | Sí |Sí |
| RequestCountHttpStatusOthers | Recuento de todos los demás códigos HTTP (fuera del intervalo 2xx-5xx). | Sí | Sí |Sí |
| RequestCountHttpStatus200 | Recuento de todas las solicitudes que dieron lugar a una respuesta de código HTTP 200. | Sí | No  |Sí |
| RequestCountHttpStatus206 | Recuento de todas las solicitudes que dieron lugar a una respuesta de código HTTP 206. | Sí | No  |Sí |
| RequestCountHttpStatus302 | Recuento de todas las solicitudes que dieron lugar a una respuesta de código HTTP 302. | Sí | No  |Sí |
| RequestCountHttpStatus304 | Recuento de todas las solicitudes que dieron lugar a una respuesta de código HTTP 304. | Sí | No  |Sí |
| RequestCountHttpStatus404 | Recuento de todas las solicitudes que dieron lugar a una respuesta de código HTTP 404. | Sí | No  |Sí |
| RequestCountCacheHit | Recuento de todas las solicitudes que dieron lugar a un acierto de caché. El recurso se atendió directamente desde el servidor POP al cliente. | Sí | Sí | No  |
| RequestCountCacheMiss | Recuento de todas las solicitudes que dieron lugar a un error de caché. Un error de caché significa que el recurso no se encontró en el servidor POP más cercano al cliente y se recuperó del origen. | Sí | Sí | No |
| RequestCountCacheNoCache | Recuento de todas las solicitudes a un recurso a las que se les impide almacenarse en caché debido a una configuración de usuario en el servidor perimetral. | Sí | Sí | No |
| RequestCountCacheUncacheable | Recuento de todas las solicitudes a recursos cuyos encabezados Cache-Control y Expires impiden que se almacenen en caché. Este recuento indica que no se debería almacenar en caché en un servidor POP ni por el cliente HTTP. | Sí | Sí | No |
| RequestCountCacheOthers | Recuento de todas las solicitudes con un estado de caché no cubierto por lo anterior. | No | Sí | No  |
| EgressTotal | Transferencia de datos salientes en GB | Sí |Sí |Sí |
| EgressHttpStatus2xx | Transferencia de datos salientes* para respuestas con códigos de estado HTTP 2xx en GB. | Sí | Sí | No  |
| EgressHttpStatus3xx | Transferencia de datos salientes para respuestas con códigos de estado HTTP 3xx en GB. | Sí | Sí | No  |
| EgressHttpStatus4xx | Transferencia de datos de salida para respuestas con códigos de estado HTTP 4xx en GB. | Sí | Sí | No  |
| EgressHttpStatus5xx | Transferencia de datos de salida para respuestas con códigos de estado HTTP 5xx en GB. | Sí | Sí | No |
| EgressHttpStatusOthers | Transferencia de datos de salida para respuestas con otros códigos de estado HTTP en GB. | Sí | Sí | No  |
| EgressCacheHit | Transferencia de datos de salida para respuestas que se entregaron directamente desde la caché de CDN en los servidores POP/perimetrales de CDN. | Sí | Sí | No |
| EgressCacheMiss. | Transferencia de datos de salida para respuestas que no se encontraron en el servidor POP más cercano y que se recuperaron del servidor de origen. | Sí | Sí | No |
| EgressCacheNoCache | Transferencia de datos de salida para recursos a los que se les impide almacenarse en caché debido a una configuración de usuario en el servidor perimetral. | Sí | Sí | No |
| EgressCacheUncacheable | Transferencia de datos de salida para recursos cuyos encabezados Cache-Control o Expires impiden que se almacenen en caché. Indica que no se debería almacenar en caché en un servidor POP ni por el cliente HTTP. | Sí | Sí | No |
| EgressCacheOthers | Transferencias de datos de salida para otros escenarios de caché. | No | Sí | No |

\*Con transferencia de datos de salida nos referimos al tráfico entregado al cliente desde los servidores POP de la red CDN.


### <a name="schema-of-the-core-analytics-logs"></a>Esquema de los registros de análisis básico 

Todos los registros se almacenan en formato JSON y cada entrada tiene campos de cadena que siguen el siguiente esquema:

```json
    "records": [
        {
            "time": "2017-04-27T01:00:00",
            "resourceId": "<ARM Resource Id of the CDN Endpoint>",
            "operationName": "Microsoft.Cdn/profiles/endpoints/contentDelivery",
            "category": "CoreAnalytics",
            "properties": {
                "DomainName": "<Name of the domain for which the statistics is reported>",
                "RequestCountTotal": integer value,
                "RequestCountHttpStatus2xx": integer value,
                "RequestCountHttpStatus3xx": integer value,
                "RequestCountHttpStatus4xx": integer value,
                "RequestCountHttpStatus5xx": integer value,
                "RequestCountHttpStatusOthers": integer value,
                "RequestCountHttpStatus200": integer value,
                "RequestCountHttpStatus206": integer value,
                "RequestCountHttpStatus302": integer value,
                "RequestCountHttpStatus304": integer value,
                "RequestCountHttpStatus404": integer value,
                "RequestCountCacheHit": integer value,
                "RequestCountCacheMiss": integer value,
                "RequestCountCacheNoCache": integer value,
                "RequestCountCacheUncacheable": integer value,
                "RequestCountCacheOthers": integer value,
                "EgressTotal": double value,
                "EgressHttpStatus2xx": double value,
                "EgressHttpStatus3xx": double value,
                "EgressHttpStatus4xx": double value,
                "EgressHttpStatus5xx": double value,
                "EgressHttpStatusOthers": double value,
                "EgressCacheHit": double value,
                "EgressCacheMiss": double value,
                "EgressCacheNoCache": double value,
                "EgressCacheUncacheable": double value,
                "EgressCacheOthers": double value,
            }
        }

    ]
}
```

Donde *time* representa la hora de inicio del límite horario cuyas estadísticas se notifican. Una métrica no admitida por un proveedor de CDN, en lugar de un valor doble o entero, genera un valor nulo. Este valor nulo indica la ausencia de una métrica, y esto es diferente de un valor de 0. Un conjunto de estas métricas por dominio se configura en el punto de conexión.

Propiedades de ejemplo:

```json
{
     "DomainName": "manlingakamaitest2.azureedge.net",
     "RequestCountTotal": 480,
     "RequestCountHttpStatus2xx": 480,
     "RequestCountHttpStatus3xx": 0,
     "RequestCountHttpStatus4xx": 0,
     "RequestCountHttpStatus5xx": 0,
     "RequestCountHttpStatusOthers": 0,
     "RequestCountHttpStatus200": 480,
     "RequestCountHttpStatus206": 0,
     "RequestCountHttpStatus302": 0,
     "RequestCountHttpStatus304": 0,
     "RequestCountHttpStatus404": 0,
     "RequestCountCacheHit": null,
     "RequestCountCacheMiss": null,
     "RequestCountCacheNoCache": null,
     "RequestCountCacheUncacheable": null,
     "RequestCountCacheOthers": null,
     "EgressTotal": 0.09,
     "EgressHttpStatus2xx": null,
     "EgressHttpStatus3xx": null,
     "EgressHttpStatus4xx": null,
     "EgressHttpStatus5xx": null,
     "EgressHttpStatusOthers": null,
     "EgressCacheHit": null,
     "EgressCacheMiss": null,
     "EgressCacheNoCache": null,
     "EgressCacheUncacheable": null,
     "EgressCacheOthers": null
}

```

## <a name="additional-resources"></a>Recursos adicionales

* [Registros de diagnóstico de Azure](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs)
* [Análisis básico mediante el portal complementario de Azure CDN](https://docs.microsoft.com/azure/cdn/cdn-analyze-usage-patterns)
* [Registros de Azure Monitor](https://docs.microsoft.com/azure/log-analytics/log-analytics-overview)
* [API de REST de Azure Log Analytics](https://docs.microsoft.com/rest/api/loganalytics)







