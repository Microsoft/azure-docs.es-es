---
title: Descripción de la herramienta de migración para alertas de Azure Monitor
description: Comprenda el funcionamiento de la herramienta de migración de alertas y solucione problemas.
ms.topic: conceptual
ms.date: 07/10/2019
ms.author: yalavi
author: yalavi
ms.subservice: alerts
ms.openlocfilehash: 533d114e08464ff95c654a6f071ea28a04caf510
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87564102"
---
# <a name="understand-how-the-migration-tool-works"></a>Descripción del funcionamiento de la herramienta de migración

Tal como se [anunció anteriormente](monitoring-classic-retirement.md), las alertas clásicas en Azure Monitor quedarán en desuso el 31 de agosto de 2019 (la fecha original era el 30 de junio de 2019). Hay disponible una herramienta de migración en Azure Portal para los clientes que usan reglas de alertas clásicas y que desean desencadenar la migración ellos mismos.

En este artículo se explica cómo funciona la herramienta de migración voluntaria. También se describen soluciones para algunos problemas comunes.

> [!NOTE]
> Debido a un retraso en la implementación de la herramienta de migración, la fecha de retirada para la migración de alertas clásicas se [extendió hasta el 31 de agosto de 2019](https://azure.microsoft.com/updates/azure-monitor-classic-alerts-retirement-date-extended-to-august-31st-2019/), desde la fecha anunciada originalmente del 30 de junio de 2019.

## <a name="classic-alert-rules-that-will-not-be-migrated"></a>Reglas de alertas clásicas que no se migrarán

> [!IMPORTANT]
> La migración no afecta a las alertas de registro de actividad (incluidas alertas de estado de servicio) ni las alertas de registros. La migración solo se aplica a las reglas de alertas clásicas que se describen [aquí](monitoring-classic-retirement.md#retirement-of-classic-monitoring-and-alerting-platform).

Aunque la herramienta puede migrar casi todas las [reglas de alertas clásicas](monitoring-classic-retirement.md#retirement-of-classic-monitoring-and-alerting-platform), existen algunas excepciones. Las siguientes reglas de alertas no se migrarán mediante la herramienta (ni durante la migración automática que comenzará en septiembre de 2019):

- Reglas de alertas clásica en las métricas de invitado de máquina virtual (Windows y Linux). Consulte la [guía para volver a crear estas reglas de alertas en nuevas alertas de métricas](#guest-metrics-on-virtual-machines) más adelante en este artículo.
- Reglas de alertas clásicas en las métricas de almacenamiento clásico. Consulte la [guía para supervisar las cuentas de almacenamiento clásico](https://azure.microsoft.com/blog/modernize-alerting-using-arm-storage-accounts/).
- Reglas de alertas clásicas en algunas métricas de cuenta de almacenamiento. Consulte los [detalles](#storage-account-metrics) más adelante en este artículo.
- Reglas de alertas clásicas en algunas métricas de Cosmos DB. Consulte los [detalles](#cosmos-db-metrics) más adelante en este artículo.
- Reglas de alertas clásicas en todas las métricas clásicas de máquinas virtuales y Cloud Services (Microsoft.ClassicCompute/virtualMachines y Microsoft.ClassicCompute/domainNames/slots/roles). Consulte los [detalles](#classic-compute-metrics) más adelante en este artículo.

Si su suscripción tiene este tipo de regla clásica, deberá migrarlas manualmente. Dado que no podemos proporcionar una migración automática, las alertas de métricas clásicas existentes de estos tipos seguirán funcionando hasta junio de 2020. Esta extensión le dará tiempo para migrar a nuevas alertas. También puede continuar creando nuevas alertas clásicas en las excepciones indicadas anteriormente hasta junio de 2020. Sin embargo, en el resto de casos, no se crearán nuevas alertas clásicas después de agosto de 2019.

> [!NOTE]
> Además de las excepciones que se indican anteriormente, si las reglas de alertas clásicas no son válidas, es decir, se encuentran en [métricas en desuso](#classic-alert-rules-on-deprecated-metrics) o recursos eliminados, no se migrarán ni estarán disponibles tras retirarse el servicio.

### <a name="guest-metrics-on-virtual-machines"></a>Métricas de invitado en máquinas virtuales

Para poder crear nuevas alertas de métricas en las métricas de invitado, estas últimas se deben enviar al almacén de métricas personalizadas de Azure Monitor. Siga estas instrucciones para habilitar el receptor de Azure Monitor en la configuración de diagnóstico:

- [Habilitar métricas de invitado para VM de Windows](collect-custom-metrics-guestos-resource-manager-vm.md)
- [Habilitar métricas de invitado para VM de Linux](collect-custom-metrics-linux-telegraf.md)

Una vez realizados estos pasos, puede crear nuevas alertas de métricas en las métricas de invitado. Además, después de crear nuevas alertas de métricas, puede eliminar las alertas clásicas.

### <a name="storage-account-metrics"></a>Métricas de la cuenta de almacenamiento

Todas las alertas clásicas en las cuentas de almacenamiento se pueden migrar, excepto las alertas de estas métricas:

- PercentAuthorizationError
- PercentClientOtherError
- PercentNetworkError
- PercentServerOtherError
- PercentSuccess
- PercentThrottlingError
- PercentTimeoutError
- AnonymousThrottlingError
- SASThrottlingError
- ThrottlingError

La reglas de alertas clásicas en las métricas de porcentaje se deben migrar en función de la [asignación entre las métricas de almacenamiento antiguas y nuevas](../../storage/common/storage-metrics-migration.md#metrics-mapping-between-old-metrics-and-new-metrics). Los umbrales deberán modificarse según corresponda porque la nueva métrica disponible es absoluta.

Las reglas de alertas clásicas en AnonymousThrottlingError, SASThrottlingError y ThrottlingError deben dividirse en dos nuevas alertas porque no hay ninguna métrica combinada que proporciona la misma funcionalidad. Los umbrales deberán adaptarse según corresponda.

### <a name="cosmos-db-metrics"></a>Métricas de Cosmos DB

Todas las alertas clásicas de las métricas de Cosmos DB se pueden migrar, excepto las alertas de estas métricas:

- Solicitudes medias por segundo
- Nivel de coherencia
- Http 2xx
- Http 3xx
- Http 400
- Http 401
- Internal Server Error
- Número máximo de RUPM consumidas por minuto
- Número máximo de RU por segundo
- Solicitudes con error de recuento de Mongo
- Solicitudes con error de eliminación de Mongo
- Solicitudes con error de inserción de Mongo
- Otras solicitudes con error de Mongo
- Cargo de otras solicitudes de Mongo
- Velocidad de otras solicitudes de Mongo
- Solicitudes con error de consultas de Mongo
- Solicitudes con error de actualización de Mongo
- Latencia de lectura observada
- Latencia de escritura observada
- Disponibilidad del servicio
- Capacidad de almacenamiento
- Solicitudes limitadas
- Total de solicitudes

Actualmente no están disponibles en el [nuevo sistema](metrics-supported.md#microsoftdocumentdbdatabaseaccounts) las solicitudes medias por segundo, el nivel de coherencia, el número máximo de RUPM consumidas por minuto, el número máximo de RU por segundo, la latencia de lectura observada, la latencia de escritura observada y la capacidad de almacenamiento.

Las alertas de métricas de solicitud como Http 2xx, Http 3xx, Http 400, Http 401, error interno del servidor, disponibilidad del servicio, solicitudes limitadas y solicitudes totales no se migran porque la manera de contar las solicitudes en las métricas clásicas y las métricas nuevas es diferente. Las alertas de este tipo deberán volver a crearse manualmente con umbrales ajustados.

Las alertas de las métricas de solicitudes con error de Mongo deben dividirse en varias alertas, dado que no hay ninguna métrica combinada que proporcione la misma funcionalidad. Los umbrales deberán adaptarse según corresponda.

### <a name="classic-compute-metrics"></a>Métricas de proceso clásico

Las alertas de métricas de proceso clásico no se migrarán con la herramienta de migración, ya que los recursos de proceso clásico todavía no se admiten con las nuevas alertas. La compatibilidad con nuevas alertas en estos tipos de recursos se encuentra actualmente en versión preliminar pública y los clientes pueden volver a crear nuevas reglas de alerta equivalentes en función de las reglas de alertas clásicas.

### <a name="classic-alert-rules-on-deprecated-metrics"></a>Reglas de alertas clásicas en métricas en desuso

Estas son las reglas de alertas clásicas en las métricas que se admitían anteriormente, pero finalmente han quedado en desuso. Un pequeño porcentaje de clientes podría tener reglas de alertas clásicas no válidas en esas métricas. Puesto que estas reglas de alertas no son válidas, no se migrarán.

| Tipo de recurso| Métricas en desuso |
|-------------|----------------- |
| Microsoft.DBforMySQL/servers | compute_consumption_percent, compute_limit |
| Microsoft.DBforPostgreSQL/servers | compute_consumption_percent, compute_limit |
| Microsoft.Network/publicIPAddresses | defaultddostriggerrate |
| Microsoft.SQL/servers/databases | service_level_objective, storage_limit, storage_used, throttling, dtu_consumption_percent, storage_used |
| Microsoft.Web/hostingEnvironments/multirolepools | averagememoryworkingset |
| Microsoft.Web/hostingEnvironments/workerpools | bytesreceived, httpqueuelength |

## <a name="how-equivalent-new-alert-rules-and-action-groups-are-created"></a>Cómo se crean nuevas reglas de alertas equivalente y grupos de acciones

La herramienta de migración convierte las reglas de alertas clásicas en nuevas reglas de alertas equivalentes y grupos de acciones. Para la mayoría las reglas de alertas clásicas, las nuevas reglas de alertas equivalentes están en la misma métrica con las mismas propiedades, como `windowSize` y `aggregationType`. Sin embargo, hay algunas reglas de alertas clásicas que se encuentran en las métricas que tienen una métrica equivalente diferente en el nuevo sistema. Los siguientes principios se aplican a la migración de las alertas clásicas, a menos que especifique en la sección siguiente:

- **Frecuencia**: define la frecuencia con la que se comprueba la condición de una regla de alertas clásicas o nueva. El elemento `frequency` en las reglas de alertas clásicas era configurable por el usuario y era siempre 5 minutos para todos los tipos de recursos, excepto los componentes de Application Insights, para el que era 1 minuto. Por lo tanto, la frecuencia de las reglas equivalentes también se establece en 5 minutos y 1 minuto, respectivamente.
- **Tipo de agregación**: define cómo se agrega la métrica a través de la ventana de interés. El elemento `aggregationType` también es el mismo entre las alertas clásicas y las alertas nuevas para la mayoría de las métricas. En algunos casos, dado que la métrica es diferente entre las alertas clásicas y las alertas nuevas, se usa el elemento `aggregationType` equivalente o el elemento `primary Aggregation Type` definido para la métrica.
- **Unidades**: propiedad de la métrica en la que se crea la alerta. Algunas métricas equivalentes tienen unidades diferentes. El umbral se ajusta según corresponda y en función de la necesidad. Por ejemplo, si la métrica original tiene segundos como unidades pero la nueva métrica equivalente tiene milisegundos como unidades, el umbral original se multiplica por 1000 para garantizar el mismo comportamiento.
- **Tamaño de la ventana**: define la ventana durante la que los datos de la métrica se agregan para compararlos con el umbral. Para valores `windowSize` estándares, como 5 minutos, 15 minutos, 30 minutos, 1 hora, 3 horas, 6 horas, 12 horas o 1 día, no hay ningún cambio realizado para la nueva regla de alertas equivalente. Para otros valores, se usará el elemento `windowSize` más cercano. Para la mayoría de los clientes, este cambio no tiene ningún impacto. Para un pequeño porcentaje de clientes, es posible que sea necesario ajustar el umbral para obtener exactamente el mismo comportamiento.

En las secciones siguientes, detallamos las métricas que tienen una métrica diferente equivalente en el nuevo sistema. Las métricas que permanecen iguales para las mismas reglas de alertas clásicas y nuevas no se indican. Encontrará una lista de las métricas que se admiten en el nuevo sistema [aquí](metrics-supported.md).

### <a name="microsoftstorageaccountsservices"></a>Microsoft.StorageAccounts/services

Para servicios de cuenta de almacenamiento, como blob, tabla, archivo y cola, se asignan las siguientes métricas a las métricas equivalentes, tal como se muestra a continuación:

| Métrica en las alertas clásicas | Métrica equivalente en las alertas nuevas | Comentarios|
|--------------------------|---------------------------------|---------|
| AnonymousAuthorizationError| Métrica Transacciones con las dimensiones "ResponseType"="AuthorizationError" y "Authentication" = "Anonymous"| |
| AnonymousClientOtherError | Métrica Transacciones con las dimensiones "ResponseType"="ClientOtherError" y "Authentication" = "Anonymous" | |
| AnonymousClientTimeOutError| Métrica Transacciones con las dimensiones "ResponseType"="ClientTimeOutError" y "Authentication" = "Anonymous" | |
| AnonymousNetworkError | Métrica Transacciones con las dimensiones "ResponseType"="NetworkError" y "Authentication" = "Anonymous" | |
| AnonymousServerOtherError | Métrica Transacciones con las dimensiones "ResponseType"="ServerOtherError" y "Authentication" = "Anonymous" | |
| AnonymousServerTimeOutError | Métrica Transacciones con las dimensiones "ResponseType"="ServerTimeOutError" y "Authentication" = "Anonymous" | |
| AnonymousSuccess | Métrica Transacciones con las dimensiones "ResponseType"="Success" y "Authentication" = "Anonymous" | |
| AuthorizationError | Métrica Transacciones con las dimensiones "ResponseType"="AuthorizationError" | |
| AverageE2ELatency | SuccessE2ELatency | |
| AverageServerLatency | SuccessServerLatency | |
| Capacity | BlobCapacity | Use `aggregationType` "average" en lugar de "last". La métrica solo se aplica a Blob Service |
| ClientOtherError | Métrica Transacciones con las dimensiones "ResponseType"="ClientOtherError"  | |
| ClientTimeoutError | Métrica Transacciones con las dimensiones "ResponseType"="ClientTimeOutError" | |
| ContainerCount | ContainerCount | Use `aggregationType` "average" en lugar de "last". La métrica solo se aplica a Blob Service |
| NetworkError | Métrica Transacciones con las dimensiones "ResponseType"="NetworkError" | |
| ObjectCount | BlobCount| Use `aggregationType` "average" en lugar de "last". La métrica solo se aplica a Blob Service |
| SASAuthorizationError | Métrica Transacciones con las dimensiones "ResponseType"="AuthorizationError" y "Authentication" = "SAS" | |
| SASClientOtherError | Métrica Transacciones con las dimensiones "ResponseType"="ClientOtherError" y "Authentication" = "SAS" | |
| SASClientTimeOutError | Métrica Transacciones con las dimensiones "ResponseType"="ClientTimeOutError" y "Authentication" = "SAS" | |
| SASNetworkError | Métrica Transacciones con las dimensiones "ResponseType"="NetworkError" y "Authentication" = "SAS" | |
| SASServerOtherError | Métrica Transacciones con las dimensiones "ResponseType"="ServerOtherError" y "Authentication" = "SAS" | |
| SASServerTimeOutError | Métrica Transacciones con las dimensiones "ResponseType"="ServerTimeOutError" and "Authentication" = "SAS" | |
| SASSuccess | Métrica Transacciones con las dimensiones "ResponseType"="NetworkError" y "Authentication" = "SAS" | |
| ServerOtherError | Métrica Transacciones con las dimensiones "ResponseType"="ClientOtherError" | |
| ServerTimeOutError | Métrica Transacciones con las dimensiones "ResponseType"="ServerTimeOutError"  | |
| Correcto | Métrica Transacciones con las dimensiones "ResponseType"="Success" | |
| TotalBillableRequests| Transacciones | |
| TotalEgress | Salida | |
| TotalIngress | Entrada | |
| TotalRequests | Transacciones | |

### <a name="microsoftinsightscomponents"></a>Microsoft.insights/components

Para Application Insights, las métricas equivalentes se muestran a continuación:

| Métrica en las alertas clásicas | Métrica equivalente en las alertas nuevas | Comentarios|
|--------------------------|---------------------------------|---------|
| availability.availabilityMetric.value | availabilityResults/availabilityPercentage|   |
| availability.durationMetric.value | availabilityResults/duration| Multiplique umbral original por 1000, porque las unidades para la métrica clásica se expresan en segundos y para la nueva métrica se expresan en milisegundos.  |
| basicExceptionBrowser.count | exceptions/browser|  Use `aggregationType` "count" en lugar de "sum". |
| basicExceptionServer.count | exceptions/server| Use `aggregationType` "count" en lugar de "sum".  |
| clientPerformance.clientProcess.value | browserTimings/processingDuration| Multiplique umbral original por 1000, porque las unidades para la métrica clásica se expresan en segundos y para la nueva métrica se expresan en milisegundos.  |
| clientPerformance.networkConnection.value | browserTimings/networkDuration|  Multiplique umbral original por 1000, porque las unidades para la métrica clásica se expresan en segundos y para la nueva métrica se expresan en milisegundos. |
| clientPerformance.receiveRequest.value | browserTimings/receiveDuration| Multiplique umbral original por 1000, porque las unidades para la métrica clásica se expresan en segundos y para la nueva métrica se expresan en milisegundos.  |
| clientPerformance.sendRequest.value | browserTimings/sendDuration| Multiplique umbral original por 1000, porque las unidades para la métrica clásica se expresan en segundos y para la nueva métrica se expresan en milisegundos.  |
| clientPerformance.total.value | browserTimings/totalDuration| Multiplique umbral original por 1000, porque las unidades para la métrica clásica se expresan en segundos y para la nueva métrica se expresan en milisegundos.  |
| performanceCounter.available_bytes.value | performanceCounters/memoryAvailableBytes|   |
| performanceCounter.io_data_bytes_per_sec.value | performanceCounters/processIOBytesPerSecond|   |
| performanceCounter.number_of_exceps_thrown_per_sec.value | performanceCounters/exceptionsPerSecond|   |
| performanceCounter.percentage_processor_time_normalized.value | performanceCounters/processCpuPercentage|   |
| performanceCounter.percentage_processor_time.value | performanceCounters/processCpuPercentage| El umbral se tendrá que modificar según corresponda, ya que la métrica original se aplicaba a todos los núcleos mientras que la métrica nueva se normaliza en un núcleo. La herramienta de migración no cambia los umbrales.  |
| performanceCounter.percentage_processor_total.value | performanceCounters/processorCpuPercentage|   |
| performanceCounter.process_private_bytes.value | performanceCounters/processPrivateBytes|   |
| performanceCounter.request_execution_time.value | performanceCounters/requestExecutionTime|   |
| performanceCounter.requests_in_application_queue.value | performanceCounters/requestsInQueue|   |
| performanceCounter.requests_per_sec.value | performanceCounters/requestsPerSecond|   |
| request.duration | requests/duration| Multiplique umbral original por 1000, porque las unidades para la métrica clásica se expresan en segundos y para la nueva métrica se expresan en milisegundos.  |
| request.rate | requests/rate|   |
| requestFailed.count | requests/failed| Use `aggregationType` "count" en lugar de "sum".   |
| view.count | pageViews/count| Use `aggregationType` "count" en lugar de "sum".   |

### <a name="microsoftdocumentdbdatabaseaccounts"></a>Microsoft.DocumentDB/databaseAccounts

Para Cosmos DB, las métricas equivalentes se muestran a continuación:

| Métrica en las alertas clásicas | Métrica equivalente en las alertas nuevas | Comentarios|
|--------------------------|---------------------------------|---------|
| AvailableStorage     |AvailableStorage|   |
| Tamaño de datos | DataUsage| |
| Recuento de documentos | DocumentCount||
| Tamaño de índice | IndexUsage||
| Cargo de la solicitud de recuento de Mongo| MongoRequestCharge con dimensión "CommandName" = "count"||
| Velocidad de la solicitud de recuento de Mongo | MongoRequestsCount con dimensión "CommandName" = "count"||
| Carga de la solicitud de eliminación de Mongo | MongoRequestCharge con dimensión "CommandName" = "delete"||
| Velocidad de la solicitud de eliminación de Mongo | MongoRequestsCount con dimensión "CommandName" = "delete"||
| Cargo de la solicitud de inserción de Mongo | MongoRequestCharge con dimensión "CommandName" = "insert"||
| Velocidad de la solicitud de inserción de Mongo | MongoRequestsCount con dimensión "CommandName" = "insert"||
| Cargo de la solicitud de consulta de Mongo | MongoRequestCharge con dimensión "CommandName" = "find"||
| Velocidad de la solicitud de consulta de Mongo | MongoRequestsCount con dimensión "CommandName" = "find"||
| Carga de la solicitud de actualización de Mongo | MongoRequestCharge con dimensión "CommandName" = "update"||
| Servicio no disponible| ServiceAvailability||
| TotalRequestUnits | TotalRequestUnits||

### <a name="how-equivalent-action-groups-are-created"></a>Creación de grupos de acción equivalentes

Las reglas de alertas clásicas tenían vinculadas acciones de correo electrónico, webhook, runbook y aplicación lógica. Las nuevas reglas de alertas usan grupos de acciones que se pueden volver a usar en varias reglas de alertas. La herramienta de migración crea el grupo de acciones único para las mismas acciones, independientemente de cuántas reglas de alertas usan la acción. Los grupos de acciones que crea la herramienta de migración usan el formato de nombres "Migrated_AG*".

> [!NOTE]
> Las alertas clásicas envían correos electrónicos localizados en función de la configuración regional del administrador clásico cuando se usan para notificar roles de administrador clásico. Los nuevos mensajes de alerta se envían a través de grupos de acciones y solo están en inglés.

## <a name="rollout-phases"></a>Fases de lanzamiento

La herramienta de migración se está lanzando en fases para los clientes que usan reglas de alertas clásicas. Los propietarios de suscripciones recibirán un correo electrónico cuando la suscripción esté lista para la migración mediante el uso de la herramienta.

> [!NOTE]
> Dado que la herramienta se está lanzando en fases, es posible que vea que algunas de sus suscripciones no están preparadas para la migración durante las fases iniciales.

Actualmente, la mayoría de las suscripciones están marcadas como listas para la migración. Solo las suscripciones que tienen alertas clásicas en los siguientes tipos de recursos todavía no están listas para la migración.

- Microsoft.classicCompute/domainNames/slots/roles
- Microsoft.insights/components

## <a name="who-can-trigger-the-migration"></a>¿Quién puede desencadenar la migración?

Cualquier usuario que tenga el rol integrado de colaborador de supervisión en el nivel de la suscripción puede desencadenar la migración. Los usuarios que tienen un rol personalizado con los siguientes permisos también pueden desencadenar la migración:

- */read
- Microsoft.Insights/actiongroups/*
- Microsoft.Insights/AlertRules/*
- Microsoft.Insights/metricAlerts/*
- Microsoft.AlertsManagement/smartDetectorAlertRules/*

> [!NOTE]
> Además de tener los permisos mencionados anteriormente, su suscripción debería haberse registrado en el proveedor de recursos Microsoft.AlertsManagement. Esto es necesario para migrar correctamente las alertas de anomalías de error de Application Insights. 

## <a name="common-problems-and-remedies"></a>Problemas comunes y soluciones

Después de [desencadenar la migración](alerts-using-migration-tool.md), recibirá un correo electrónico en las direcciones que proporcionó para avisarle de que la migración se ha completado o si se requiere alguna acción de su parte. En esta sección se describen algunos problemas comunes y cómo resolverlos.

### <a name="validation-failed"></a>Error de validación

Debido a algunos cambios recientes en las reglas de alertas clásicas de la suscripción, no se puede migrar la suscripción. Este problema es temporal. Puede reiniciar la migración una vez que el estado de migración vuelva a **Ready for migration** (Listo para la migración) en unos días.

### <a name="scope-lock-preventing-us-from-migrating-your-rules"></a>Ámbito que nos impide migrar las reglas

Como parte de la migración, se crearán nuevas alertas de métricas y nuevos grupos de acciones, y luego se eliminarán las reglas de alerta clásicas. Sin embargo, un bloqueo de ámbito puede impedir la creación o eliminación de recursos. Según el bloqueo de ámbito, no se pudieron migrar algunas reglas o ninguna de ellas. Puede resolver este problema si quita el bloqueo de ámbito de la suscripción, el grupo de recursos o el recurso que aparece en la [herramienta de migración](https://portal.azure.com/#blade/Microsoft_Azure_Monitoring/MigrationBladeViewModel) y vuelve a desencadenar la migración. No se puede deshabilitar el bloqueo del ámbito y se debe quitar mientras dure el proceso de migración. [Más información sobre cómo administrar los bloqueos de ámbito](../../azure-resource-manager/management/lock-resources.md#portal).

### <a name="policy-with-deny-effect-preventing-us-from-migrating-your-rules"></a>Directiva con efecto de denegación que nos impide migrar las reglas

Como parte de la migración, se crearán nuevas alertas de métricas y nuevos grupos de acciones, y luego se eliminarán las reglas de alerta clásicas. Sin embargo, una directiva puede evitar la creación de recursos. Según la directiva, no se pudieron migrar algunas reglas o ninguna de ellas. Las directivas que bloquean el proceso se enumeran en la [herramienta de migración](https://portal.azure.com/#blade/Microsoft_Azure_Monitoring/MigrationBladeViewModel). Para resolver este problema:

- Excluya las suscripciones o los grupos de recursos mientras dure el proceso de migración de la asignación de directiva. [Más información sobre la administración del ámbito de exclusión de las directivas](../../governance/policy/tutorials/create-and-manage.md#exempt-a-non-compliant-or-denied-resource-using-exclusion).
- Quite o cambie el efecto a auditoría o anexión (lo que, por ejemplo, puede solucionar problemas relativos a la ausencia de etiquetas). [Obtenga más información sobre la administración del efecto de las directivas](../../governance/policy/concepts/definition-structure.md#policy-rule).

## <a name="next-steps"></a>Pasos siguientes

- [Uso de la herramienta de migración](alerts-using-migration-tool.md)
- [Preparación para la migración](alerts-prepare-migration.md)
