---
title: Auditoría de operaciones de plano de control de Azure Cosmos DB
description: Aprenda a auditar las operaciones del plano de control, como agregar una región, actualizar el rendimiento, conmutar por error regiones o agregar una red virtual, entre otras, en Azure Cosmos DB
author: SnehaGunda
ms.service: cosmos-db
ms.topic: how-to
ms.date: 06/25/2020
ms.author: sngun
ms.openlocfilehash: ae1d2743934c5ae8df9f2a1514bdda9b34262b9d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87023694"
---
# <a name="how-to-audit-azure-cosmos-db-control-plane-operations"></a>Auditoría de operaciones de plano de control de Azure Cosmos DB

El plano de control en Azure Cosmos DB es un servicio RESTful que le permite realizar un conjunto diverso de operaciones en la cuenta de Azure Cosmos. Expone un modelo de recursos público (por ejemplo: base de datos, cuenta) y varias operaciones a los usuarios finales para realizar acciones en el modelo de recursos. Las operaciones del plano de control incluyen cambios en la cuenta o el contenedor de Azure Cosmos. Por ejemplo, las operaciones como crear una cuenta de Azure Cosmos, agregar una región, actualizar el rendimiento, conmutar por error regiones o agregar una red virtual son algunas de las operaciones del plano de control. En este artículo se explica cómo auditar las operaciones de plano de control en Azure Cosmos DB. Puede ejecutar las operaciones del plano de control en las cuentas de Azure Cosmos mediante la CLI de Azure, PowerShell o Azure Portal, mientras que, para los contenedores, use la CLI de Azure o PowerShell.

A continuación se muestran algunos escenarios de ejemplo en los que resulta útil auditar operaciones del plano de control:

* Quiere obtener una alerta cuando se modifican las reglas de firewall para su cuenta de Azure Cosmos. La alerta es necesaria para averiguar modificaciones no autorizadas en las reglas que rigen la seguridad de red de la cuenta de Azure Cosmos y realizar acciones rápidas.

* Quiere obtener una alerta si se agrega o quita una nueva región de la cuenta de Azure Cosmos. La adición o eliminación de regiones tiene consecuencias en la facturación y en los requisitos de soberanía de datos. Esta alerta le ayudará a detectar una adición o eliminación accidentales de la región en su cuenta.

* Quiere obtener más detalles sobre lo que ha cambiado en los registros de diagnóstico. Por ejemplo, una red virtual ha cambiado.

## <a name="disable-key-based-metadata-write-access"></a>Deshabilitar el acceso de escritura de metadatos basado en claves

Antes de auditar las operaciones de plano de control en Azure Cosmos DB, deshabilite el acceso de escritura de los metadatos basado en claves en su cuenta. Al deshabilitar el acceso de escritura de metadatos basado en claves, los clientes que se conecten a la cuenta de Azure Cosmos a través de claves de cuenta no podrán acceder a la cuenta. Para deshabilitar el acceso de escritura, defina la propiedad `disableKeyBasedMetadataWriteAccess` como true. Una vez definida esta propiedad, pueden aplicar cambios en los recursos los usuarios con las credenciales y el rol de control de acceso basado en rol (RBAC) adecuados. Para obtener más información sobre cómo establecer esta propiedad, consulte el artículo [Evitar cambios de SDK](role-based-access-control.md#prevent-sdk-changes). 

Una vez activado `disableKeyBasedMetadataWriteAccess`, si los clientes basados en SDK ejecutan operaciones de creación o actualización, se devuelve un error de tipo *no se permite la operación "POST" en el recurso "ContainerNameorDatabaseName" mediante el punto de conexión de Azure Cosmos DB*. Tendrá que activar el acceso a dichas operaciones para su cuenta o realizar las operaciones de creación y actualización mediante Azure Resource Manager, la CLI de Azure o Azure PowerShell. Para volver a cambiar, establezca el valor disableKeyBasedMetadataWriteAccess en **falso** mediante la CLI de Azure, tal como se describe en el artículo [Evitar cambios en el SDK de Cosmos](role-based-access-control.md#prevent-sdk-changes). Asegúrese de cambiar el valor de `disableKeyBasedMetadataWriteAccess` a falso en lugar de a true.

Tenga en cuenta los siguientes puntos al desactivar el acceso de escritura de metadatos:

* Evalúe y asegúrese de que las aplicaciones no realizan llamadas de metadatos que cambian los recursos anteriores (por ejemplo, crear una colección, actualizar el rendimiento, etc.) mediante el SDK o las claves de cuenta.

* Actualmente, Azure Portal usa claves de cuenta para las operaciones de metadatos y, por lo tanto, estas operaciones se bloquearán. También puede usar las implementaciones de la CLI de Azure, SDK o plantillas de Resource Manager para realizar estas operaciones.

## <a name="enable-diagnostic-logs-for-control-plane-operations"></a>Habilitar registros de diagnóstico para las operaciones del plano de control

Puede habilitar los registros de diagnóstico para las operaciones del plano de control mediante Azure Portal. Después de habilitar, los registros de diagnóstico registrarán la operación como un par de eventos de inicio y finalización con los detalles pertinentes. Por ejemplo, *RegionFailoverStart* y *RegionFailoverComplete* completarán el evento de conmutación por error de la región.

Siga estos pasos para habilitar el registro en las operaciones del plano de control:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y desplácese hasta la cuenta de Azure Cosmos.

1. Abra el panel de **configuración de diagnóstico** y proporcione un **Nombre** para crear los registros.

1. Seleccione **ControlPlaneRequests** para el tipo de registro y seleccione la opción **Enviar a Log Analytics**.

También puede almacenar los registros en una cuenta de almacenamiento o transmitir a un centro eventos. En este artículo se muestra cómo enviar registros a Log Analytics y, a continuación, consultarlos. Después de habilitarlos, los registros de diagnóstico tardan unos minutos en estar disponibles. Se puede realizar un seguimiento de todas las operaciones del plano de control realizadas después de ese punto. La captura de pantalla siguiente muestra cómo habilitar los registros del plano de control:

:::image type="content" source="./media/audit-control-plane-logs/enable-control-plane-requests-logs.png" alt-text="Habilitación del registro de solicitudes del plano de control":::

## <a name="view-the-control-plane-operations"></a>Ver las operaciones del plano de control

Después de activar el registro, siga estos pasos para realizar un seguimiento de las operaciones de una cuenta específica:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Abra la pestaña **Monitor** desde la navegación izquierda y seleccione el panel **Registros**. Se abre una interfaz de usuario donde se pueden ejecutar fácilmente consultas con esa cuenta específica en el ámbito. Ejecute la siguiente consulta para ver los registros del plano de control:

   ```kusto
   AzureDiagnostics
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="ControlPlaneRequests"
   | where TimeGenerated >= ago(1h)
   ```

Las capturas de pantalla siguientes capturan registros cuando se cambia un nivel de coherencia para una cuenta de Azure Cosmos:

:::image type="content" source="./media/audit-control-plane-logs/add-ip-filter-logs.png" alt-text="Registros del plano de control cuando se agrega una red virtual":::

Las capturas de pantalla siguientes capturan los registros cuando se crea el espacio de claves o una tabla de una cuenta de Cassandra, y cuando se actualiza el rendimiento. Los registros del plano de control para las operaciones de creación y actualización en la base de datos y en el contenedor se registran por separado, tal como se muestra en la siguiente captura de pantalla:

:::image type="content" source="./media/audit-control-plane-logs/throughput-update-logs.png" alt-text="Registros de plano de control cuando se actualiza el rendimiento":::

## <a name="identify-the-identity-associated-to-a-specific-operation"></a>Identificar la identidad asociada con una operación específica

Si quiere depurar más, puedes identificar una operación específica en el **Registro de actividad** mediante el identificador de actividad o la marca de tiempo de la operación. La marca de tiempo se usa para algunos clientes de Resource Manager cuando el identificador de actividad no se pasa explícitamente. El registro de actividad proporciona detalles sobre la identidad con la que se inició la operación. En la captura de pantalla siguiente se muestra cómo usar el identificador de la actividad y buscar las operaciones asociadas con él en el registro de actividad:

:::image type="content" source="./media/audit-control-plane-logs/find-operations-with-activity-id.png" alt-text="Uso del identificador de actividad y búsqueda de las operaciones":::

## <a name="control-plane-operations-for-azure-cosmos-account"></a>Operaciones del plano de control para la cuenta de Azure Cosmos

A continuación se indican las operaciones del plano de control disponibles en el nivel de cuenta. En el nivel de cuenta se realiza un seguimiento de la mayoría de las operaciones. Estas operaciones están disponibles como métricas en Azure Monitor:

* Región agregada
* Región quitada
* Cuenta eliminada
* Región conmutada por error
* Cuenta creada
* Red virtual eliminada
* Configuración actualizada de la red de la cuenta
* Configuración actualizada de replicación de la cuenta
* Claves de cuenta actualizadas
* Configuración actualizada de copia de seguridad de la cuenta
* Configuración actualizada de diagnóstico de la cuenta

## <a name="control-plane-operations-for-database-or-containers"></a>Operaciones del plano de control para bases de datos o contenedores

A continuación se indican las operaciones del plano de control disponibles en el nivel de base de datos y contenedor. Estas operaciones están disponibles como métricas en Azure Monitor:

* Base de datos SQL creada
* SQL Database actualizada
* Rendimiento de SQL Database actualizado
* SQL Database eliminada
* Contenedor SQL creado
* Contenedor SQL actualizado
* Rendimiento de contenedor de SQL actualizado
* Contenedor de SQL eliminado
* Espacio de claves de Cassandra creado
* Espacio de claves de Cassandra actualizado
* Rendimiento de un espacio de claves de Cassandra actualizado
* Espacio de claves de Cassandra eliminado
* Tabla de Cassandra creada
* Tabla de Cassandra actualizada
* Rendimiento de una tabla de Cassandra actualizado
* Tabla de Cassandra eliminada
* Base de datos de Gremlin creada
* Base de datos de Gremlin actualizada
* Rendimiento de una base de datos de Gremlin actualizado
* Base de datos de Gremlin eliminada
* Grafo de Gremlin creado
* Grafo de Gremlin actualizado
* Rendimiento de un grafo de Gremlin actualizado
* Gráfico de Gremlin eliminado
* Base de datos de Mongo creada
* Base de datos de Mongo actualizada
* Rendimiento de la base de datos de Mongo actualizado
* Base de datos de Mongo eliminada
* Colección de Mongo creada
* Colección de Mongo actualizada
* Rendimiento de la colección de Mongo actualizado
* Colección de Mongo eliminada
* Tabla de AzureTable creada
* Tabla AzureTable actualizada
* Rendimiento de la tabla AzureTable actualizado
* Tabla AzureTable eliminada

## <a name="diagnostic-log-operations"></a>Operaciones del registro de diagnóstico

A continuación se muestran los nombres de operación en los registros de diagnóstico para distintas operaciones:

* RegionAddStart, RegionAddComplete
* RegionRemoveStart, RegionRemoveComplete
* AccountDeleteStart, AccountDeleteComplete
* RegionFailoverStart, RegionFailoverComplete
* AccountCreateStart, AccountCreateComplete
* AccountUpdateStart, AccountUpdateComplete
* VirtualNetworkDeleteStart, VirtualNetworkDeleteComplete
* DiagnosticLogUpdateStart, DiagnosticLogUpdateComplete

Para las operaciones específicas de la API, la operación se denomina con el siguiente formato:

* ApiKind + ApiKindResourceType + OperationType
* ApiKind + ApiKindResourceType + "Rendimiento" + operationType

**Ejemplo** 

* CassandraKeyspacesCreate
* CassandraKeyspacesUpdate
* CassandraKeyspacesThroughputUpdate
* SqlContainersUpdate

La propiedad *ResourceDetails* contiene el cuerpo del recurso completo como una carga de solicitud y contiene todas las propiedades solicitadas para actualizar.

## <a name="diagnostic-log-queries-for-control-plane-operations"></a>Consultas de registros de diagnóstico para las operaciones del plano de control

A continuación se muestran algunos ejemplos para obtener registros de diagnóstico de las operaciones del plano de control:

```kusto
AzureDiagnostics 
| where Category startswith "ControlPlane"
| where OperationName contains "Update"
| project httpstatusCode_s, statusCode_s, OperationName, resourceDetails_s, activityId_g
```

```kusto
AzureDiagnostics 
| where Category =="ControlPlaneRequests"
| where TimeGenerated >= todatetime('2020-05-14T17:37:09.563Z')
| project TimeGenerated, OperationName, apiKind_s, apiKindResourceType_s, operationType_s, resourceDetails_s
```

```kusto
AzureDiagnostics 
| where Category =="ControlPlaneRequests"
| where  OperationName startswith "SqlContainersUpdate"
```

```kusto
AzureDiagnostics 
| where Category =="ControlPlaneRequests"
| where  OperationName startswith "SqlContainersThroughputUpdate"
```

## <a name="next-steps"></a>Pasos siguientes

* [Exploración de Azure Monitor para Azure Cosmos DB](../azure-monitor/insights/cosmosdb-insights-overview.md?toc=/azure/cosmos-db/toc.json&bc=/azure/cosmos-db/breadcrumb/toc.json)
* [Supervisión y depuración con métricas de Azure Cosmos DB](use-metrics.md)
