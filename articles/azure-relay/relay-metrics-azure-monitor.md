---
title: Métricas de Azure Relay en Azure Monitor | Microsoft Docs
description: En este artículo se proporciona información sobre cómo puede usar Azure Monitor para supervisar el estado de Azure Relay.
services: service-bus-relay
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 5fcc0f820983da388d808cadf8ce64a555e8ef0a
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87532890"
---
# <a name="azure-relay-metrics-in-azure-monitor"></a>Métricas de Azure Relay en Azure Monitor 
Las métricas de Azure Relay le permiten conocer el estado de los recursos de su suscripción de Azure. Con un amplio conjunto de datos de métricas, puede evaluar el estado general de los recursos de Relay, no solo en el nivel de espacio de nombres, sino también en el nivel de entidad. Estas estadísticas pueden ser importantes, ya que le ayudan a supervisar el estado de Azure Relay. Las métricas también pueden ayudarle a solucionar problemas de causa principal sin necesidad de ponerse en contacto con el soporte técnico de Azure.

Azure Monitor proporciona interfaces de usuario unificadas para la supervisión de distintos servicios de Azure. Para obtener más información, vea [Supervisión de Microsoft Azure](../azure-monitor/overview.md) y el ejemplo [Retrieve Azure Monitor metrics with .NET](https://github.com/Azure-Samples/monitor-dotnet-metrics-api) (Recuperación de métricas de Azure Monitor con .NET) en GitHub.

> [!IMPORTANT]
> Este artículo solo se aplica a la característica Conexiones híbridas de Azure Relay, no a WCF Relay. 

## <a name="access-metrics"></a>Acceso a la métrica

Azure Monitor proporciona varias maneras de tener acceso a las métricas. Puede acceder a las métricas desde [Azure Portal](https://portal.azure.com), o usar las API de Azure Monitor (REST y .NET) y soluciones de análisis como Operation Management Suite y Event Hubs. Para más información, vea [Datos de supervisión recopilados por Azure Monitor](../azure-monitor/platform/data-platform.md).

De forma predeterminada, las métricas están habilitadas y puede acceder a datos de los últimos 30 días. Si es necesario conservar los datos durante un periodo mayor, se pueden archivar en una cuenta de Azure Storage. Esto se configura en la [configuración de diagnóstico](../azure-monitor/platform/diagnostic-settings.md) de Azure Monitor.

## <a name="access-metrics-in-the-portal"></a>Acceso a métricas del portal

Una vez transcurrido un tiempo, las métricas se pueden supervisar en [Azure Portal](https://portal.azure.com). En el ejemplo siguiente se muestra cómo ver las solicitudes correctas y las solicitudes entrantes en el nivel de cuenta:

![Una página titulada "Monitor: Métricas (versión preliminar)" muestra un gráfico de líneas de uso de memoria para los últimos 30 días.][1]

También puede tener acceso a las métricas directamente a través del espacio de nombres. Para ello, seleccione el espacio de nombres y, después, haga clic en **Métricas**. 

Para ver métricas que admitan las dimensiones, debe filtrar por el valor de la dimensión que desee.

## <a name="billing"></a>Facturación

Actualmente, el uso de métricas en Azure Monitor es gratuito, ya que se trata de una versión preliminar. Sin embargo, si usa otras soluciones que ingieren datos de métricas, puede que se le facturen dichas soluciones. Por ejemplo, se le facturará por Azure Storage si archiva datos de métricas en una cuenta de Azure Storage. También se le facturará por los registros de Azure Monitor si transmite datos de métricas a registros de Azure Monitor para realizar análisis avanzados.

Las siguientes métricas ofrecen una visión general del estado de su servicio. 

> [!NOTE]
> Se están dejando de usar algunas métricas a medida que se mueven a otro nombre. Esto podría requerir que actualice las referencias. Las métricas que se marquen con la palabra clave "en desuso" no se admitirán en el futuro.

Los valores de las métricas se envían a Azure Monitor cada minuto. La granularidad de tiempo define el intervalo de tiempo para el que se presentan los valores de las métricas. El intervalo de tiempo compatible para todas las métricas de Azure Relay es 1 minuto.

## <a name="connection-metrics"></a>Métricas de conexión

| Nombre de la métrica | Descripción |
| ------------------- | ----------------- |
| ListenerConnections-Success  | Número de conexiones correctas del agente de escucha a Azure Relay durante un determinado periodo. <br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensión: EntityName|
|ListenerConnections-ClientError |Número de errores de cliente en las conexiones del agente de escucha durante un determinado periodo.<br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensión: EntityName|
|ListenerConnections-ServerError |Número de errores del servidor en las conexiones del agente de escucha durante un determinado periodo.<br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensión: EntityName|
|SenderConnections-Success |Número de conexiones correctas del remitente durante un determinado periodo.<br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensión: EntityName|
|SenderConnections-ClientError |Número de errores de cliente en las conexiones del remitente durante un determinado periodo.<br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensión: EntityName|
|SenderConnections-ServerError |Número de errores del servidor en las conexiones del remitente durante un determinado periodo.<br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensión: EntityName|
|ListenerConnections-TotalRequests |Número total de errores en las conexiones del agente de escucha durante un determinado periodo.<br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensión: EntityName|
|SenderConnections-TotalRequests |Solicitudes de conexión realizadas por los remitentes durante un determinado periodo.<br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensión: EntityName|
|ActiveConnections |Número de conexiones activas. Este valor es un valor de un momento dado.<br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensión: EntityName|
|ActiveListeners |El número de licencias activas. Este valor es un valor de un momento dado.<br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensión: EntityName|
|ListenerDisconnects |Número de agentes de escucha desconectados durante un determinado periodo.<br/><br/> Unidad: Bytes <br/> Tipo de agregación: Total <br/> Dimensión: EntityName|
|SenderDisconnects |Número de remitentes desconectados durante un determinado periodo.<br/><br/> Unidad: Bytes <br/> Tipo de agregación: Total <br/> Dimensión: EntityName|

## <a name="memory-usage-metrics"></a>Métricas de uso de memoria

| Nombre de la métrica | Descripción |
| ------------------- | ----------------- |
|BytesTransferred |Número de bytes transferidos durante un determinado periodo.<br/><br/> Unidad: Bytes <br/> Tipo de agregación: Total <br/> Dimensión: EntityName|

## <a name="metrics-dimensions"></a>Dimensiones de métricas

Azure Relay admite las siguientes dimensiones para las métricas de Azure Monitor. La adición de dimensiones a las métricas es opcional. Si no agrega dimensiones, las métricas se especifican en el nivel de espacio de nombres. 

|Nombre de dimensión|Descripción|
| ------------------- | ----------------- |
|EntityName| Azure Relay admite entidades de mensajería en el espacio de nombres.|

## <a name="next-steps"></a>Pasos siguientes

Vea [Información general sobre la supervisión en Microsoft Azure](../azure-monitor/overview.md).

[1]: ./media/relay-metrics-azure-monitor/relay-monitor1.png
