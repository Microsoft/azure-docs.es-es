---
title: Supervisión de Azure Blockchain Service (ABS)
description: Supervisión de Azure Blockchain Service a través de Azure Monitor
ms.date: 01/08/2020
ms.topic: how-to
ms.reviewer: v-umha
ms.openlocfilehash: 7300a5dcfb0150e6182636dcb71bacfa68c787db
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87076915"
---
# <a name="monitor-azure-blockchain-service-through-azure-monitor"></a>Supervisión de Azure Blockchain Service a través de Azure Monitor  

A medida que los clientes ejecutan escenarios de cadena de bloques de producción en Azure Blockchain Service (ABS), resulta fundamental supervisar los recursos en cuanto a disponibilidad, rendimiento y operaciones. En este artículo se describen los datos de supervisión generados por Azure Blockchain Service y cómo se pueden usar las distintas características e integraciones de Azure Monitor para analizar y alertar sobre ello a fin de administrar entornos de nivel de producción.  

## <a name="what-is-azure-monitor"></a>¿Qué es Azure Monitor?

Azure Blockchain Service crea datos de supervisión mediante Azure Monitor, que es un servicio de supervisión de pila completo de Azure que proporciona un conjunto completo de características para supervisar los recursos de Azure. Para más información acerca de Azure Monitor, consulte [Supervisión de recursos de Azure con Azure Monitor](../../azure-monitor/insights/monitor-azure-resource.md).
 

Las secciones siguientes se basan en este artículo, donde se describen los datos específicos recopilados de Azure Blockchain Service y se proporcionan ejemplos para configurar la recopilación de datos y el análisis de estos datos con herramientas de Azure.

## <a name="monitor-data-collected-from-azure-blockchain-service"></a>Supervisión de los datos recopilados de Azure Blockchain Service  

Azure Blockchain Service recopila los mismos tipos de datos de supervisión que otros recursos de Azure, que se describen en [Supervisión de datos](../../azure-monitor/insights/monitor-azure-resource.md#monitoring-data) de recursos de Azure. Consulte la [referencia de datos de supervisión de Azure Blockchain Service](#monitor-azure-blockchain-service-data-reference) para obtener más detalles de los registros y las métricas creados por Azure Blockchain Service.

La página de información general de Azure Portal para cada recurso de miembro de Azure Blockchain Service incluye una breve vista de las transacciones, incluidas las solicitudes administradas y los bloques procesados. Algunos de estos datos se recopilan automáticamente y están disponibles para su análisis en cuanto crea el recurso del miembro de Azure Blockchain Service, mientras que se puede habilitar la recopilación de datos adicional con alguna configuración.

## <a name="diagnostic-settings"></a>Configuración de diagnóstico  

Las métricas de la plataforma y el registro de actividad se recopilan automáticamente, pero debe crear una configuración de diagnóstico para recopilar registros de recursos o reenviarlos fuera de Azure Monitor. Consulte [Creación de una configuración de diagnóstico para recopilar registros de plataforma y métricas en Azure](../../azure-monitor/platform/diagnostic-settings.md) para ver el proceso detallado para crear una configuración de diagnóstico mediante el Azure Portal, la CLI o PowerShell.

Cuando se crea una configuración de diagnóstico, se especifican las categorías de registros que se van a recopilar. A continuación se enumeran las categorías de Azure Blockchain Service.

**Registros de proxy de cadena de bloques**: seleccione la categoría si desea supervisar los registros de proxy de NGNIX. Todos los detalles de la transacción del cliente están disponibles para fines de auditoría y depuración.  

**Registros de aplicación de cadena de bloques**: seleccione la categoría para obtener los registros de la aplicación de cadena de bloques hospedada por el servicio administrado. Por ejemplo, para un miembro de ABS-Quorum, estos registros serían los registros del propio Quorum.  

**Solicitudes de métricas**: seleccione esta opción para recopilar datos de métricas de Azure Cosmos DB a los destinos en la configuración de diagnóstico, que se recopilan automáticamente en las métricas de Azure. Recopile datos de métricas con registros de recursos para analizar ambos tipos de datos juntos y para enviar datos de métricas fuera de Azure Monitor.

## <a name="analyze-metric-data"></a>Análisis de datos de métricas  

Puede analizar las métricas de Azure Blockchain Service con el explorador de métricas, vaya a la pestaña Métricas en la sección Supervisión de la hoja de recursos de ABS. Consulte [Introducción al explorador de métricas de Azure](../../azure-monitor/platform/metrics-getting-started.md) para obtener más información sobre esta herramienta. Las métricas completas de Azure Blockchain Service se encuentran en las métricas estándar de Azure Blockchain Service del espacio de nombres.

Puede usar la dimensión de **nodo** al agregar un filtro o dividir las métricas, que básicamente proporciona valores de métrica por nodos de transacción y nodos de validador del miembro de ABS.

## <a name="analyze-log-data"></a>Análisis de datos de registro

Estas son algunas consultas que puede escribir en la barra de búsqueda para ayudarle a supervisar los miembros de Azure Blockchain Service. Estas consultas funcionan con el [nuevo lenguaje](../../azure-monitor/log-query/log-query-overview.md).

Para consultar las condiciones de error en los registros de aplicación de la cadena de bloques, use la siguiente consulta:

```
BlockchainApplicationLog | where BlockchainMessage contains "ERROR" or BlockchainMessage contains "fatal"

```

Para consultar las condiciones de error en los registros de aplicación de la cadena de bloques, use la siguiente consulta.  


```
BlockchainProxyLog
| filter Code != 200
| limit 500

```
Puede usar los filtros de tiempo disponibles en los registros de Azure para filtrar la consulta por un intervalo de tiempo específico.

## <a name="monitor-azure-blockchain-service-data-reference"></a>Supervisión de la referencia de datos de Azure Blockchain Service  

En este artículo se proporciona una referencia de datos de registro y métricas recopilados para analizar el rendimiento y la disponibilidad de Azure Blockchain Service.  

### <a name="resource-logs"></a>Registros del recurso

Todos los registros de recursos comparten un esquema común de nivel superior con pocas propiedades únicas específicas del servicio de cadena de bloques. Puede consultar el artículo [Esquema de registros de recursos de nivel superior](../../azure-monitor/platform/resource-logs-schema.md#top-level-common-schema); a continuación se describen los detalles de las propiedades específicas de Azure Blockchain Service.  

En la tabla siguiente se enumeran las propiedades de los registros del proxy de Azure Blockchain cuando se recopilan en registros de Azure Monitor o Azure Storage.  


| Nombre de propiedad  | Descripción |
|:---|:---|
| time | La fecha y hora (UTC) cuando se produjo la operación. |
| resourceID  | El recurso de Azure Blockchain Service para el que se habilitan los registros.  |
| category  |En el caso de Azure Blockchain Service, los valores posibles son **Proxylogs** y **Applicationlogs**. |
| operationName  | Nombre de la operación representada por este evento.   |
| Nivel de registro  | De forma predeterminada, Azure Blockchain Service habilita el nivel de registro **informativo**.   |
| NodeLocation  | Región de Azure donde se implementa el miembro de la cadena de bloques.  |
| BlockchainNodeName  | Nombre del nodo del miembro de Azure Blockchain Service en el que se realiza la operación.   |
| EthMethod  | El método, al que llama el protocolo de cadena de bloques subyacente, en Quorum, podría ser eth_sendTransactions, eth_getBlockByNumber, etc.  |
| Agente  | Agente de usuario que actúa en nombre de un usuario, como el explorador web Mozilla, Microsoft Edge, etc. Algunos ejemplos de estos valores son: "Mozilla/5.0 (Linux x64) node.js/8.16.0 v8/6.2.414.77"  |
| Código   | Códigos de error HTTP. Normalmente, 4XX y 5XX son condiciones de error.  |
| NodeHost  | Nombre DNS del nodo.   |
| RequestMethodName | Se llama al método HTTP, los valores posibles aquí son PUT para crear un miembro, GET para obtener detalles de un miembro existente, DELETE para eliminar un miembro, PATCH actualizar un miembro.   |
| BlockchainMemberName  | Nombre del miembro de Azure Blockchain Service proporcionado por el usuario.  |
| Consortium | Nombre del consorcio proporcionado por el usuario.   |
| Remote  | Dirección IP del cliente de donde procede la solicitud.  |
| RequestSize  | Tamaño de la solicitud realizada en bytes.  |
| RequestTime  | Duración de la solicitud en milisegundos.|




En la tabla siguiente se enumeran las propiedades de los registros de aplicaciones de Azure Blockchain.


| Nombre de propiedad  | Descripción |
|:---|:---|
| time | La fecha y hora (UTC) cuando se produjo la operación. |
| resourceID  | El recurso de Azure Blockchain Service para el que se habilitan los registros.|
| category  |En el caso de Azure Blockchain Service, los valores posibles son **Proxylogs** y **Applicationlogs**.  |
| operationName  | Nombre de la operación representada por este evento.   |
| Nivel de registro  | De forma predeterminada, Azure Blockchain Service habilita el nivel de registro **informativo**.   |
| NodeLocation  | Región de Azure donde se implementa el miembro de la cadena de bloques.  |
| BlockchainNodeName  | Nombre del nodo del miembro de Azure Blockchain Service en el que se realiza la operación.   |
| BlockchainMessage    | Este campo contendrá el registro de la aplicación Blockchain que contiene los registros de datos sin formato. En el caso de ABS-Quorum, esto tendría registros de Quorum. Contiene información sobre el tipo de entrada de registro que es informativa, de error, de advertencia y de una cadena que proporciona más información sobre la acción ejecutada.   |
| TenantID    | El inquilino específico de la región de Azure Blockchain Service. El formato de este campo es https://westlake-rp-prod.<region>.cloudapp.azure.com, donde la región especifica la región de Azure del miembro implementado.       |
| SourceSystem   | El sistema rellena los registros, en este caso es **Azure**.    |



### <a name="metrics"></a>Métricas

En las tablas siguientes se enumeran las métricas de plataforma recopiladas para Azure Blockchain Service. Todas las métricas se almacenan en las métricas estándar de **Azure Blockchain Service** del espacio de nombres.

Para obtener una lista de todas las métricas compatibles de Azure Monitor (incluido Azure Blockchain Service), consulte [Métricas compatibles con Azure Monitor](../../azure-monitor/platform/metrics-supported.md).

### <a name="blockchain-metrics"></a>Métricas de Blockchain

En la tabla siguiente se especifica la lista de métricas de Blockchain que se recopilan para el recurso de miembro de Azure Blockchain Service.


| Nombre de métrica | Unidad  |  Tipo de agregación| Descripción   |
|---|---|---|---|
| Pending Transactions (Transacciones pendientes)   | Count  |  Average | Número de transacciones que esperan extracción.   |
| Processed Blocks (Bloques procesados)   | Count  | Sum  |  Número de bloques procesados en cada intervalo de tiempo. Actualmente, el tamaño de bloque es de 5 segundos, por lo que cada nodo procesará 12 bloques y 60 bloques en 5 minutos.   |
|Processed Transactions (Transacciones procesadas)    | Count  | Sum  | Número de transacciones procesadas en un bloque.    |
|Queued Transactions (Transacciones en cola)    |  Count | Average  | Número de transacciones que no pueden extraerse de inmediato. Puede deberse a que han llegado fuera de orden y la futura está esperando a que llegue la anterior. O bien, puede ser que dos transacciones tengan el mismo número que se usa solo una vez (nonce) y el mismo valor de gas; por lo tanto, no se puede extraer e segunda.   |

### <a name="connection-metrics"></a>Métricas de conexión  

En la tabla siguiente se enumeran las diferentes métricas de conexión que se recopilan para el recurso de miembro de Azure Blockchain Service. Estas son las métricas de proxy de NGINX.


| Nombre de métrica | Unidad  |  Tipo de agregación| Descripción |
|---|---|---|---|
| Accepted Connections (Conexiones aceptadas)   | Count  |  Sum | Número total de conexiones de cliente aceptadas.   |
| Conexiones activas  | Count  | Average  |  Número actual de conexiones de cliente activas, incluidas las conexiones en espera.    |
|Handled Connections (Conexiones controladas)    | Count  | Sum  | Número total de conexiones controladas. Por lo general, el valor del parámetro es el mismo que el de las conexiones aceptadas, a menos que se hayan alcanzado algunos límites de recursos.     |
|Handled Requests (Solicitudes controladas)     |  Count | Sum  | Número total de solicitudes de cliente.  |


### <a name="performance-metrics"></a>Métricas de rendimiento.

En la tabla siguiente se enumeran las métricas de conexión que se recopilan para cada uno de los nodos del recurso del miembro de Azure Blockchain.  


| Nombre de métrica | Unidad  |  Tipo de agregación| Descripción   |
|---|---|---|---|
| Porcentaje de uso de CPU   | Porcentaje  |  Max | Porcentaje de uso de CPU.     |
| IO Read Bytes (Bytes de lectura de E/S)   | Kilobytes   | Sum  |  Suma de bytes de lectura de E/S en todos los nodos del recurso de miembro de la cadena de bloques.      |
|IO Write Bytes (Bytes de escritura de E/S)     | Kilobytes   | Sum  | Suma de bytes de escritura de E/S en todos los nodos del recurso de miembro de la cadena de bloques.     |
|Memory Limit (Límite de memoria)       |  Gigabytes   | Average    | Memoria máxima disponible para el proceso de la cadena de bloques por nodo. |
|Uso de la memoria     | Gigabytes  |  Average | Cantidad de memoria utilizada en promedio en todos los nodos.  |
| Memory Usage Percentage (Porcentaje de uso de memoria)     | Porcentaje   | Average  |  Porcentaje de memoria utilizada en promedio en todos los nodos.       |
|Storage Usage (Uso del almacenamiento)      | Gigabytes   | Average  | GB de almacenamiento utilizado en promedio en todos los nodos.       |


## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre [Blockchain Data Manager](./data-manager.md) para capturar y transformar los datos de cadena de bloques en Azure Event Grid.
