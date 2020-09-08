---
title: Escalabilidad horizontal de Azure Analysis Services | Microsoft Docs
description: Replique servidores de Azure Analysis Services con la escalabilidad horizontal. Las consultas del cliente se pueden distribuir entre varias réplicas de consultas en un grupo de consultas con escalabilidad horizontal.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: ceed2a287fb210a421972e9c9f9e6c77c6cb1879
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88716935"
---
# <a name="azure-analysis-services-scale-out"></a>Escalabilidad horizontal de Azure Analysis Services

Con la escalabilidad horizontal, las consultas de cliente pueden distribuirse entre varias *réplicas de consulta* de un *grupo de consultas*, reduciendo así los tiempos de respuesta durante las cargas de trabajo con un número elevado de consultas. También puede separar el procesamiento del grupo de consultas, lo que garantiza que las operaciones de procesamiento no afecten de forma negativa a las consultas de cliente. La escalabilidad horizontal puede configurarse en Azure Portal o con el uso de la API de REST de Analysis Services.

La escalabilidad horizontal está disponible para los servidores en el plan de tarifa Estándar. Cada réplica de consulta se factura a la misma tarifa que el servidor. Todas las réplicas de la consulta se crean en la misma región que el servidor. El número de réplicas de consultas que puede configurar está limitado por la región en la que se encuentra el servidor. Para más información, consulte la [disponibilidad por región](analysis-services-overview.md#availability-by-region). La escalabilidad horizontal no aumenta la cantidad de memoria disponible para el servidor. Para aumentar la memoria, debe actualizar el plan. 

## <a name="why-scale-out"></a>Motivos para escalar horizontalmente

En una implementación típica de servidor, un servidor actúa como servidor de procesamiento y servidor de consulta. Si el número de consultas de cliente en los modelos en el servidor supera la unidades de procesamiento de consultas (QPU) del plan de su servidor o el procesamiento del modelo se produce al mismo tiempo que las cargas de trabajo con un número elevado de consultas, el rendimiento podría bajar. 

Con la escalabilidad horizontal, puede crear un grupo de consultas con hasta siete recursos de réplica de consulta adicionales (ocho en total, incluido el servidor *principal*). Puede escalar el número de réplicas del grupo de consultas para satisfacer las demandas de QPU en los momentos críticos y, además, puede separar un servidor de procesamiento del grupo de consultas en cualquier momento. 

Independientemente del número de réplicas de consultas que tenga en un grupo de consultas, las cargas de trabajo de procesamiento no se distribuyen entre las réplicas de consulta. El servidor principal actúa como servidor de procesamiento. Las réplicas de consulta solo realizan consultas en las bases de datos de modelo sincronizadas entre el servidor principal y cada réplica del grupo de consultas. 

Durante el escalado horizontal, las nuevas réplicas de consulta pueden tardar hasta cinco minutos en agregarse de forma incremental al grupo de consulta. Cuando todas las nuevas réplicas de consulta están en funcionamiento, se equilibra la carga de las nuevas conexiones de cliente en todos los recursos del grupo de consultas. Las conexiones de cliente existentes no cambian del recurso al que están conectadas actualmente. En la reducción horizontal, se terminan las conexiones de cliente existentes a un recurso de grupo de consultas que se va a quitar del grupo de consultas. Los clientes pueden volver a conectarse a un recurso del grupo de consultas restante.

## <a name="how-it-works"></a>Funcionamiento

Al configurar el escalado horizontal por primera vez, las bases de datos de modelo en el servidor principal se sincronizan *automáticamente* con las nuevas réplicas de un nuevo grupo de consultas. La sincronización automática se produce solo una vez. Durante la sincronización automática, los archivos de datos del servidor principal (cifrados en reposo en Blob Storage) se copian en una segunda ubicación, también cifrados en reposo en Blob Storage. Las réplicas del grupo de consultas se *hidratan* con datos del segundo conjunto de archivos. 

Aunque una sincronización automática solo se realiza cuando se escala horizontalmente un servidor por primera vez, también puede realizar una sincronización manual. La sincronización garantiza que los datos en las réplicas del grupo de consultas coinciden con los del servidor principal. Al procesar (actualizar) modelos en el servidor principal, se debe realizar una sincronización *después* de la finalización de las operaciones de procesamiento. Esta sincronización copia los datos actualizados de los archivos del servidor principal de Blob Storage en el segundo conjunto de archivos. Las réplicas del grupo de consultas se hidratan con los datos actualizados del segundo conjunto de archivos de Blob Storage. 

Al realizar una operación de escalado horizontal posterior, por ejemplo, aumentar el número de réplicas del grupo de consultas de dos a cinco, las nuevas réplicas se hidratan con los datos del segundo conjunto de archivos de Blob Storage. No hay ninguna sincronización. Si efectuara una sincronización después del escalado horizontal, las nuevas réplicas del grupo de consultas se hidratarían dos veces, lo que generaría una hidratación redundante. Al realizar una operación de escalado horizontal posterior, es importante tener en cuenta:

* Realice una sincronización *antes de la operación de escalado horizontal* para evitar la hidratación redundante de las réplicas agregadas. No se permiten operaciones de escalado horizontal y sincronización simultáneas que se ejecuten al mismo tiempo.

* Al automatizar las operaciones de procesamiento *y* escalado horizontal, es importante procesar primero los datos en el servidor principal, luego realizar una sincronización y, por último, realizar la operación de escalado horizontal. Esta secuencia garantiza un impacto mínimo en los recursos de memoria y QPU.

* Se permite la sincronización aun cuando no haya réplicas en el grupo de consultas. Si va a escalar horizontalmente de cero a una o más réplicas con nuevos datos de una operación de procesamiento en el servidor principal, primero realice la sincronización sin réplicas en el grupo de consultas y luego escale horizontalmente. La sincronización antes del escalado horizontal evita la hidratación redundante de las réplicas recién agregadas.

* Al eliminar una base de datos de modelo del servidor principal, no se elimina automáticamente de las réplicas del grupo de consultas. Debe realizar una operación de sincronización mediante el comando de PowerShell [Sync-AzAnalysisServicesInstance](https://docs.microsoft.com/powershell/module/az.analysisservices/sync-AzAnalysisServicesinstance) que quita los archivos de esa base de datos de la ubicación de Blob Storage compartida de la réplica y luego elimina la base de datos de modelo en las réplicas del grupo de consultas. Para determinar si existe una base de datos de modelo en las réplicas del grupo de consultas, pero no en el servidor principal, asegúrese de que la opción **Separar el servidor de procesamiento del grupo de consultas** esté establecida en **Sí**. Luego use SSMS para conectarse al servidor principal mediante el calificador `:rw` para ver si existe la base de datos. Después conéctese a las réplicas del grupo de consultas sin el calificador `:rw` para ver si también existe la misma base de datos. Si la base de datos existe en las réplicas del grupo de consultas pero no en el servidor principal, ejecute una operación de sincronización.   

* Al cambiar el nombre de una base de datos en el servidor principal, hay un paso adicional necesario para garantizar que la base de datos se sincronice correctamente con las réplicas. Después de cambiar el nombre, realice una sincronización con el comando [Sync-AzAnalysisServicesInstance](https://docs.microsoft.com/powershell/module/az.analysisservices/sync-AzAnalysisServicesinstance) especificando el parámetro `-Database` con el nombre antiguo de la base de datos. Esta sincronización quita la base de datos y los archivos con el nombre antiguo de las réplicas. Luego realice otra sincronización especificando el parámetro `-Database` con el nuevo nombre de la base de datos. La segunda sincronización copia la base de datos con el nombre nuevo en el segundo conjunto de archivos e hidrata las réplicas. Estas sincronizaciones no se pueden realizar mediante el comando de modelo Sincronizar del portal.

### <a name="synchronization-mode"></a>Modo de sincronización

De forma predeterminada, las réplicas de consultas se rehidratan en su totalidad, no de forma incremental. La rehidratación tiene lugar en fases. Las réplicas se desasocian y se asocian de dos en dos (suponiendo que haya al menos tres réplicas) para garantizar que, en un momento dado, hay al menos una réplica que se mantiene en línea para las consultas. En algunos casos, es posible que los clientes necesiten volver a conectarse a una de las réplicas en línea mientras se está llevando a cabo este proceso. Con la opción **ReplicaSyncMode**, ahora puede especificar que la sincronización de réplicas de consulta se realice en paralelo. La sincronización paralela brinda las siguientes ventajas: 

- Una reducción significativa del tiempo de sincronización. 
- Es más fácil que los datos de las diferentes réplicas mantengan la coherencia durante el proceso de sincronización. 
- Como las bases de datos se mantienen en línea en todas las réplicas del proceso de sincronización, los clientes no necesitan volver a establecer la conexión. 
- La caché en memoria se actualiza de forma incremental exclusivamente con los datos modificados, lo que puede ser más rápido que rehidratar por completo el modelo. 

#### <a name="setting-replicasyncmode"></a>Opción ReplicaSyncMode

Utilice SSMS para configurar ReplicaSyncMode en Propiedades avanzadas. Los valores posibles son: 

- `1` (predeterminado): se realiza una rehidratación completa de la base de datos de réplicas en fases (incremental). 
- `2`: sincronización optimizada en paralelo. 

![Opción RelicaSyncMode](media/analysis-services-scale-out/aas-scale-out-sync-mode.png)

Cuando se establece **ReplicaSyncMode=2**, en función del tiempo que la memoria caché necesite para actualizarse, las réplicas de consulta pueden consumir más memoria. Si desea mantener la base de datos en línea y disponible para las consultas, dependiendo de cuánto hayan cambiado los datos, la operación puede necesitar hasta el *doble de memoria* en la réplica, ya que se mantienen los dos segmentos (el nuevo y el antiguo) simultáneamente. Los nodos de la réplica tienen la misma asignación de memoria que el nodo principal y, por lo general, suele haber memoria extra en el nodo principal para las operaciones de actualización, por lo que es poco probable que las réplicas se queden sin memoria. Además, lo más habitual es que la base de datos se actualice incrementalmente en el nodo principal, por lo que pocas veces resulta necesario duplicar la memoria. Si la operación de sincronización registra un error por falta de memoria, esta operación se intentará de nuevo utilizando la técnica predeterminada (asociar y desasociar las réplicas de dos en dos). 

### <a name="separate-processing-from-query-pool"></a>Separar el procesamiento del grupo de consultas

Para obtener un rendimiento máximo para las operaciones de procesamiento y consulta, puede elegir separar el servidor de procesamiento del grupo de consultas. Cuando están separadas, las conexiones de cliente nuevas se asignan a réplicas de consulta del grupo de consultas únicamente. Si las operaciones de procesamiento solo ocupan un breve período de tiempo, puede elegir separar el servidor de procesamiento del grupo de consultas solo durante el tiempo que se tarda en realizar las operaciones de procesamiento y sincronización y, a continuación, volver a incluirlo en el grupo de consultas. Al separar el servidor de procesamiento del grupo de consultas o al agregarlo de nuevo a este, la operación puede tardar hasta cinco minutos en realizarse.

## <a name="monitor-qpu-usage"></a>Supervisión del uso de QPU

Para determinar si es necesario realizar un escalado horizontal del servidor, [supervise el servidor](analysis-services-monitor.md) en Azure Portal con métricas. Si se supera el nivel máximo de QPU con regularidad, significa que el número de consultas en los modelos excede el límite de QPU del plan. La métrica de longitud de la cola de trabajos del grupo de consultas también aumenta si el número de consultas de la cola del grupo de subprocesos de consulta excede el número de QPU disponibles. 

Otra buena métrica para inspeccionar es QPU media por ServerResourceType. Esta métrica compara el promedio de la unidad de procesamiento de consultas (QPU) del servidor principal con el grupo de consultas. 

![Métricas de escalado horizontal de consultas](media/analysis-services-scale-out/aas-scale-out-monitor.png)

**Para configurar la QPU mediante ServerResourceType**

1. En un gráfico de líneas Métricas, haga clic en **Agregar métrica**. 
2. En **RECURSO**, seleccione el servidor y, en **ESPACIO DE NOMBRES DE MÉTRICA**, seleccione **Métricas estándar de Analysis Services**; luego, en **MÉTRICA**, seleccione **QPU** y, en **AGREGACIÓN**, seleccione **Media**. 
3. Haga clic en **Aplicar separación**. 
4. En **VALORES**, seleccione **ServerResourceType**.  

### <a name="detailed-diagnostic-logging"></a>Registros de diagnóstico detallados

Utilice Azure Monitor Logs para realizar diagnósticos más detallados de los recursos del servidor con escala horizontal. Con los registros, puede usar consultas de Log Analytics para desglosar la QPU y la memoria en función de cada servidor y cada réplica. Para más información, vea estas consultas de ejemplo de los [registros de diagnóstico de Analysis Services](analysis-services-logging.md#example-queries).


## <a name="configure-scale-out"></a>Configuración de la escalabilidad horizontal

### <a name="in-azure-portal"></a>En Azure Portal

1. En Azure Portal, haga clic en **Escalabilidad horizontal**. Utilice el control deslizante para seleccionar el número de servidores de réplica de consulta. El número de réplicas elegido se suma al servidor existente.  

2. En **Separar el servidor de procesamiento del grupo de consultas**, seleccione Sí para excluir el servidor de procesamiento de los servidores de consulta. Las [conexiones](#connections) de cliente mediante la cadena de conexión predeterminada (sin `:rw`) se redirigen a las réplicas del grupo de consultas. 

   ![Control deslizante de escalabilidad horizontal](media/analysis-services-scale-out/aas-scale-out-slider.png)

3. Haga clic en **Guardar** para aprovisionar los nuevos servidores de réplica de consulta. 

Al configurar el escalado horizontal para un servidor por primera vez, los modelos en el servidor principal se sincronizan automáticamente con las réplicas del grupo de consultas. La sincronización automática solo se produce una vez, cuando se configura el escalado horizontal en una o más réplicas por primera vez. Los cambios posteriores en el número de réplicas en el mismo servidor *no desencadenan otra sincronización automática*. La sincronización automática no se vuelve a producir aun cuando se establezca el servidor en cero réplicas y luego se vuelva a escalar horizontalmente en cualquier número de réplicas. 

## <a name="synchronize"></a>Sincronizar 

Las operaciones de sincronización deben realizarse manualmente o mediante la API de REST.

### <a name="in-azure-portal"></a>En Azure Portal

En **Información general** > modelo > **Sincronizar el modelo**.

![Control deslizante de escalabilidad horizontal](media/analysis-services-scale-out/aas-scale-out-sync.png)

### <a name="rest-api"></a>API DE REST

Use la operación **sync**.

#### <a name="synchronize-a-model"></a>Sincronización de un modelo   

`POST https://<region>.asazure.windows.net/servers/<servername>:rw/models/<modelname>/sync`

#### <a name="get-sync-status"></a>Obtención del estado de sincronización  

`GET https://<region>.asazure.windows.net/servers/<servername>/models/<modelname>/sync`

Códigos de estado devueltos:


|Código  |Descripción  |
|---------|---------|
|-1     |  No válida       |
|0     | Replicando        |
|1     |  Rehidratando       |
|2     |   Completed       |
|3     |   Con error      |
|4     |    Finalizando     |
|||


### <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Antes de usar PowerShell, [instale o actualice la última versión del módulo de Azure PowerShell](/powershell/azure/install-az-ps). 

Para ejecutar la sincronización, use [Sync-AzAnalysisServicesInstance](https://docs.microsoft.com/powershell/module/az.analysisservices/sync-AzAnalysisServicesinstance).

Para establecer el número de réplicas de consulta, use [Set-AzAnalysisServicesServer](https://docs.microsoft.com/powershell/module/az.analysisservices/set-azanalysisservicesserver). Especifique el parámetro opcional `-ReadonlyReplicaCount`.

Para separar el servidor de procesamiento del grupo de consultas, use [Set-AzAnalysisServicesServer](https://docs.microsoft.com/powershell/module/az.analysisservices/set-azanalysisservicesserver). Especifique el parámetro opcional `-DefaultConnectionMode` para usar `Readonly`.

Para obtener más información, vea [Uso de una entidad de servicio con el módulo Az.AnalysisServices](analysis-services-service-principal.md#azmodule).

## <a name="connections"></a>Conexiones

En la página de información general del servidor, hay dos nombres de servidor. Si aún no ha configurado la escalabilidad horizontal de un servidor, los dos nombres de servidor funcionan igual. Una vez configurada la escalabilidad horizontal de un servidor, debe especificar el nombre de servidor adecuado según el tipo de conexión. 

Para las conexiones de cliente para el usuario final como Power BI Desktop, Excel y aplicaciones personalizadas, utilice **Nombre del servidor**. 

Para SSMS, Visual Studio y cadenas de conexión en PowerShell, las aplicaciones de Azure Functions y AMO usan el **nombre del servidor de administración**. El nombre del servidor de administración incluye un calificador especial `:rw` (lectura y escritura). Todas las operaciones de procesamiento se producen en el servidor de administración (principal).

![Nombres de servidor](media/analysis-services-scale-out/aas-scale-out-name.png)

## <a name="scale-up-scale-down-vs-scale-out"></a>Escalado y reducción verticales frente a Escalado horizontal

Puede cambiar el plan de tarifa en un servidor con varias réplicas. El mismo plan de tarifa se aplica a todas las réplicas. Una operación de escalado desactiva todas las réplicas a la vez y, luego, muestra todas las réplicas en el nuevo plan de tarifa.

## <a name="troubleshoot"></a>Solución de problemas

**Problema:** los usuarios ven la instancia de error **No se puede encontrar el servidor "\<Name of the server>" en el modo de conexión "ReadOnly".**

**Solución:** al seleccionar la opción **Separar el servidor de procesamiento del grupo de consultas**, las conexiones del cliente que usan la cadena de conexión predeterminada (sin `:rw`) se redirigen a las réplicas del grupo de consultas. Si las réplicas en el grupo de consultas no están en línea porque la sincronización aún no se ha completado, es posible que se produzca un error en las conexiones de cliente redirigidas. Para evitar errores de conexión, debe haber al menos dos servidores en el grupo de consultas al realizar una sincronización. Cada servidor se sincroniza individualmente mientras que otros siguen en línea. Si decide no incluir el servidor de procesamiento en el grupo de consultas durante el procesamiento, puede quitarlo del grupo para el procesamiento y agregarlo de nuevo una vez completado el procesamiento, pero antes de la sincronización. Use las métricas de memoria y QPU para supervisar el estado de sincronización.



## <a name="related-information"></a>Información relacionada

[Supervisión de las métricas del servidor](analysis-services-monitor.md)   
[Administración de Azure Analysis Services](analysis-services-manage.md) 
