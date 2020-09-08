---
title: Diagnóstico y solución de problemas al usar el SDK de .NET de Azure Cosmos DB
description: Use características como registro del lado cliente y otras herramientas de terceros para identificar, diagnosticar y solucionar problemas de Azure Cosmos DB cuando use el SDK de .NET.
author: anfeldma-ms
ms.service: cosmos-db
ms.date: 06/16/2020
ms.author: anfeldma
ms.subservice: cosmosdb-sql
ms.topic: troubleshooting
ms.reviewer: sngun
ms.custom: devx-track-dotnet
ms.openlocfilehash: bc5af781b86ef559abaf33b0cb027ef14adb4262
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89021908"
---
# <a name="diagnose-and-troubleshoot-issues-when-using-azure-cosmos-db-net-sdk"></a>Diagnóstico y solución de problemas al usar el SDK de .NET de Azure Cosmos DB

> [!div class="op_single_selector"]
> * [SDK de Java v4](troubleshoot-java-sdk-v4-sql.md)
> * [SDK de Java v2 asincrónico](troubleshoot-java-async-sdk.md)
> * [.NET](troubleshoot-dot-net-sdk.md)
> 

En este artículo se tratan problemas comunes, soluciones alternativas, pasos de diagnóstico y herramientas al usar el [SDK de .NET](sql-api-sdk-dotnet.md) con las cuentas de la API de SQL de Azure Cosmos DB.
El SDK de .NET proporciona la representación lógica del lado cliente para acceder a la API de SQL de Azure Cosmos DB. En este artículo se describen herramientas y enfoques para ayudarle si surge algún problema.

## <a name="checklist-for-troubleshooting-issues"></a>Lista de comprobación para la solución de problemas
Tenga en cuenta la siguiente lista de comprobación antes de que la aplicación pase a la fase de producción. El uso de la lista de comprobación impedirá que se produzcan distintos problemas comunes que podrían surgir. Puede diagnosticar rápidamente cuándo se produce un problema:

*    Utilice el [SDK](sql-api-sdk-dotnet-standard.md) más reciente. No se debe utilizar la previsualización de SDK en la producción. Esto evitará problemas conocidos que ya están solucionados.
*    Revise los [consejos de rendimiento](performance-tips.md) y siga las prácticas sugeridas. Esto ayudará a evitar el escalado, la latencia y otros problemas de rendimiento.
*    Habilite el registro de SDK para ayudarle a solucionar un problema. La habilitación del registro puede afectar al rendimiento, por lo que es mejor hacerlo solo para solucionar problemas. Puede habilitar los registros siguientes:
*    Registre [métricas](monitor-accounts.md) mediante Azure Portal. Las métricas del portal muestran la telemetría de Azure Cosmos DB, que resulta útil para determinar si el problema corresponde a Azure Cosmos DB o al cliente.
*    Registre la [cadena de diagnósticos](https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.client.resourceresponsebase.requestdiagnosticsstring) en el SDK de V2 o el [diagnóstico](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.responsemessage.diagnostics) en el SDK de V3 a partir de las respuestas de operación de punto.
*    Registre las [métricas de consultas SQL](sql-api-query-metrics.md) desde todas las respuestas de consultas. 
*    Siga la configuración del [registro del SDK]( https://github.com/Azure/azure-cosmos-dotnet-v2/blob/master/docs/documentdb-sdk_capture_etl.md).

Eche un vistazo a la sección [Problemas y soluciones](#common-issues-workarounds) de este artículo.

Compruebe que la [sección de problemas de GitHub](https://github.com/Azure/azure-cosmos-dotnet-v2/issues) se esté supervisando activamente. Compruebe si encuentra algún problema similar con una solución alternativa ya registrada. Si no logró solucionarlo, registre un problema de GitHub. Puede abrir una incidencia de soporte técnico para problemas urgentes.


## <a name="common-issues-and-workarounds"></a><a name="common-issues-workarounds"></a>Problemas comunes y soluciones alternativas

### <a name="general-suggestions"></a>Sugerencias generales
* Ejecute la aplicación en la misma región de Azure que su cuenta de Azure Cosmos DB, si es posible. 
* Es posible que experimente problemas de conectividad o disponibilidad debido a falta de recursos en el equipo cliente. Le recomendamos que supervise el uso de la CPU en los nodos que ejecutan el cliente de Azure Cosmos DB y la escalación vertical u horizontal si están ejecutando una carga alta.

### <a name="check-the-portal-metrics"></a>Comprobación de las métricas del portal
La comprobación de las [métricas del portal](monitor-accounts.md) le ayudarán a determinar si hay un problema por parte del cliente o si hay un problema con el servicio. Por ejemplo, si las métricas contienen una alta tasa de solicitudes de velocidad limitada (código de estado HTTP 429), lo que significa que la solicitud se ha limitado, consulte la sección [Tasa de solicitudes demasiado grande](troubleshoot-request-rate-too-large.md). 

## <a name="common-error-status-codes"></a>Códigos de estado de error habituales <a id="error-codes"></a>

| Código de estado | Descripción | 
|----------|-------------|
| 400 | Solicitud incorrecta (depende del mensaje de error)| 
| 401 | [No autorizado](troubleshoot-unauthorized.md) | 
| 404 | [No se encuentra el recurso](troubleshoot-not-found.md) |
| 408 | [Se ha agotado el tiempo de espera para la solicitud](troubleshoot-dot-net-sdk-request-timeout.md) |
| 409 | Un error de conflicto se produce cuando un recurso existente ha tomado el identificador proporcionado para un recurso en una operación de escritura. Use otro identificador para que el recurso resuelva este problema, ya que el identificador debe ser único en todos los documentos con el mismo valor de clave de partición. |
| 410 | Excepciones no superadas (error transitorio que no debería infringir el acuerdo de nivel de servicio). |
| 412 | Un error en la condición previa se produce cuando la operación especificó un valor eTag que es diferente de la versión disponible en el servidor. Es un error de simultaneidad optimista. Vuelva a intentar la solicitud después de leer la versión más reciente del recurso y de actualizar el valor eTag en la solicitud.
| 413 | [Entidad de solicitud demasiado grande](concepts-limits.md#per-item-limits) |
| 429 | [Demasiadas solicitudes](troubleshoot-request-rate-too-large.md) |
| 449 | Error transitorio que solo se produce en las operaciones de escritura y es seguro de reintentar. |
| 500 | No se pudo realizar la operación debido a un error de servicio inesperado. Póngase en contacto con el servicio de soporte técnico. Consulte Cómo presentar un [problema de soporte técnico de Azure](https://aka.ms/azure-support). |
| 503 | [Servicio no disponible](troubleshoot-service-unavailable.md) | 

### <a name="azure-snat-pat-port-exhaustion"></a><a name="snat"></a>Agotamiento de puertos SNAT (PAT) de Azure

Si la aplicación está implementada en [Azure Virtual Machines sin una dirección IP pública](../load-balancer/load-balancer-outbound-connections.md), los [puertos SNAT de Azure](../load-balancer/load-balancer-outbound-connections.md#preallocatedports) se usan de manera predeterminada para establecer conexiones con cualquier punto de conexión fuera de la VM. El número de conexiones permitidas desde la máquina virtual hasta el punto de conexión de Azure Cosmos DB está limitado por la [configuración de Azure SNAT](../load-balancer/load-balancer-outbound-connections.md#preallocatedports). Esta situación puede conducir a la limitación de la conexión, al cierre de la conexión o a los [tiempos de espera de solicitudes](troubleshoot-dot-net-sdk-request-timeout.md) mencionados anteriormente.

 Los puertos SNAT de Azure se usan solo cuando la VM tiene una dirección IP privada y se conecta a una dirección IP pública. Hay dos soluciones alternativas para evitar la limitación de SNAT de Azure (siempre que esté usando una única instancia de cliente en toda la aplicación):

* Agregue el punto de conexión de servicio de Azure Cosmos DB a la subred de la red virtual de Azure Virtual Machines. Para obtener más información, consulte [puntos de conexión de servicio de red virtual de Azure](../virtual-network/virtual-network-service-endpoints-overview.md). 

    Cuando se habilita el punto de conexión de servicio, las solicitudes ya no se envían desde una dirección IP pública a Azure Cosmos DB. En su lugar, se envían la red virtual y la identidad de la subred. Este cambio puede producir caídas de firewall si solo se permiten direcciones IP públicas. Si usa un firewall, cuando se habilite el punto de conexión de servicio, agregue una subred al firewall mediante las [ACL de Virtual Network](../virtual-network/virtual-networks-acl.md).
* Asigne una [dirección IP pública a la VM de Azure](../load-balancer/troubleshoot-outbound-connection.md#assignilpip).

### <a name="high-network-latency"></a><a name="high-network-latency"></a>Latencia de red alta
La latencia de red alta puede identificarse mediante la [cadena de diagnósticos ](https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.client.resourceresponsebase.requestdiagnosticsstring?view=azure-dotnet) en el SDK de V2 o el [diagnóstico](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.responsemessage.diagnostics?view=azure-dotnet#Microsoft_Azure_Cosmos_ResponseMessage_Diagnostics) en el SDK de V3.

Si no hay ningún [tiempo de espera ](troubleshoot-dot-net-sdk-request-timeout.md) presente y el diagnóstico muestra solicitudes únicas en las que la latencia alta es evidente en la diferencia entre `ResponseTime` y `RequestStartTime`, como se muestra a continuación (> 300 milisegundos en este ejemplo):

```bash
RequestStartTime: 2020-03-09T22:44:49.5373624Z, RequestEndTime: 2020-03-09T22:44:49.9279906Z,  Number of regions attempted:1
ResponseTime: 2020-03-09T22:44:49.9279906Z, StoreResult: StorePhysicalAddress: rntbd://..., ...
```

Esta latencia puede tener varias causas:

* La aplicación no se está ejecutando en la misma región que la cuenta de Azure Cosmos DB.
* La configuración de [PreferredLocations](https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.client.connectionpolicy.preferredlocations) o [ApplicationRegion](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationregion) es incorrecta y está intentando conectarse a una región diferente de aquella en la que se está ejecutando actualmente la aplicación.
* Puede haber un cuello de botella en la interfaz de red debido al elevado tráfico. Si la aplicación se ejecuta en Azure Virtual Machines, existen alternativas posibles:
    * Considere la posibilidad de usar una [máquina virtual con la opción Redes aceleradas habilitada](../virtual-network/create-vm-accelerated-networking-powershell.md).
    * Habilite la opción [Redes aceleradas en una máquina virtual existente](../virtual-network/create-vm-accelerated-networking-powershell.md#enable-accelerated-networking-on-existing-vms).
    * Considere la posibilidad de usar una [máquina virtual superior](../virtual-machines/windows/sizes.md).

### <a name="slow-query-performance"></a>Rendimiento lento de las consultas
Las [métricas de consulta](sql-api-query-metrics.md) le ayudarán a determinar dónde está dedicando más tiempo la consulta. En las métricas de consulta, puede ver la cantidad que se dedica al back-end en comparación con el cliente.
* Si la consulta de back-end se devuelve rápidamente y se dedica una gran cantidad de tiempo al cliente, compruebe la carga en la máquina. Es probable que no haya suficientes recursos y el SDK esté esperando que haya recursos disponibles para gestionar la respuesta.
* Si la consulta de back-end es lenta, intente [optimizar la consulta](optimize-cost-queries.md) y examine la [directiva de indexación](index-overview.md). 

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre las guías de rendimiento para [.NET V3](performance-tips-dotnet-sdk-v3-sql.md) y [.NET V2](performance-tips.md)
* Más información sobre los [SDK de Java basados en reactores](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/reactor-pattern-guide.md)

 <!--Anchors-->
[Common issues and workarounds]: #common-issues-workarounds
[Enable client SDK logging]: #logging
[Azure SNAT (PAT) port exhaustion]: #snat
[Production check list]: #production-check-list
