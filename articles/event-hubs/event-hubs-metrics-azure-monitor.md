---
title: Métricas de Azure Event Hubs en Azure Monitor | Microsoft Docs
description: En este artículo se proporciona información sobre cómo usar Azure Monitoring para supervisar Azure Event Hubs.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: c7841007fa1736c88c30108099f86b99d3dad0fb
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074866"
---
# <a name="azure-event-hubs-metrics-in-azure-monitor"></a>Métricas de Azure Event Hubs en Azure Monitor

Las métricas de Event Hubs le permiten conocer el estado de los recursos de Event Hubs en su suscripción de Azure. Con un amplio conjunto de datos de métricas, puede evaluar el estado general de los centros de eventos, no solo en el nivel de espacio de nombres, sino también en el nivel de entidad. Estas estadísticas pueden ser importantes, ya que le ayudan a supervisar el estado de los centros de eventos. Las métricas también pueden ayudarle a solucionar problemas de causa principal sin necesidad de ponerse en contacto con el soporte técnico de Azure.

Azure Monitor proporciona interfaces de usuario unificadas para la supervisión de distintos servicios de Azure. Para obtener más información, vea [Supervisión de Microsoft Azure](../azure-monitor/overview.md) y el ejemplo [Retrieve Azure Monitor metrics with .NET](https://github.com/Azure-Samples/monitor-dotnet-metrics-api) (Recuperación de métricas de Azure Monitor con .NET) en GitHub.

## <a name="access-metrics"></a>Acceso a la métrica

Azure Monitor proporciona varias maneras de tener acceso a las métricas. Puede acceder a las métricas desde [Azure Portal](https://portal.azure.com), o usar las API de Azure Monitor (REST y .NET) y soluciones de análisis como Log Analytics y Event Hubs. Para más información, vea [Datos de supervisión recopilados por Azure Monitor](../azure-monitor/platform/data-platform.md).

De forma predeterminada, las métricas están habilitadas y puede acceder a datos de los últimos 30 días. Si es necesario conservar los datos durante un periodo mayor, se pueden archivar en una cuenta de Azure Storage. Esto se configura en la [configuración de diagnóstico](../azure-monitor/platform/diagnostic-settings.md) de Azure Monitor.


## <a name="access-metrics-in-the-portal"></a>Acceso a métricas del portal

Una vez transcurrido un tiempo, las métricas se pueden supervisar en [Azure Portal](https://portal.azure.com). En el ejemplo siguiente se muestra cómo ver las solicitudes correctas y las solicitudes entrantes en el nivel de cuenta:

![Ver las métricas de éxito][1]

También puede tener acceso a las métricas directamente a través del espacio de nombres. Para ello, seleccione el espacio de nombres y, después, haga clic en **Métricas**. Para mostrar las métricas filtradas al ámbito del centro de eventos, seleccione el centro de eventos y, después, haga clic en **Métricas**.

Para ver métricas que admitan las dimensiones, debe filtrar por el valor de la dimensión que desee como se muestra en el siguiente ejemplo:

![Filtrar por valor de dimensión][2]

## <a name="billing"></a>Facturación

Actualmente, el uso de métricas en Azure Monitor es gratuito. Sin embargo, si usa otras soluciones que ingieren datos de métricas, puede que se le facturen dichas soluciones. Por ejemplo, se le facturará por Azure Storage si archiva datos de métricas en una cuenta de Azure Storage. También se le facturará por Azure si transmite datos de métricas a registros de Azure Monitor para realizar análisis avanzados.

Las siguientes métricas ofrecen una visión general del estado de su servicio. 

> [!NOTE]
> Se están dejando de usar algunas métricas a medida que se mueven a otro nombre. Esto podría requerir que actualice las referencias. Las métricas que se marquen con la palabra clave "en desuso" no se admitirán en el futuro.

Los valores de las métricas se envían a Azure Monitor cada minuto. La granularidad de tiempo define el intervalo de tiempo para el que se presentan los valores de las métricas. El intervalo de tiempo compatible para todas las métricas de Event Hubs es 1 minuto.

## <a name="request-metrics"></a>Solicitud de métricas

Cuenta el número de solicitudes de operaciones de datos y administración.

| Nombre de la métrica | Descripción |
| ------------------- | ----------------- |
| Solicitudes entrantes  | Número de solicitudes realizadas al servicio de Azure Event Hubs durante un período determinado. <br/><br/> Unidad: recuento <br/> Tipo de agregación: total <br/> Dimensión: EntityName |
| Solicitudes correctas    | Número de solicitudes correctas realizadas al servicio de Azure Event Hubs durante un período determinado. <br/><br/> Unidad: recuento <br/> Tipo de agregación: total <br/> Dimensión: EntityName |
| Errores del servidor  | Número de solicitudes no procesadas debido a un error en el servicio de Azure Event Hubs durante un período determinado. <br/><br/>Unidad: recuento <br/> Tipo de agregación: total <br/> Dimensión: EntityName |
|Errores de usuario |Número de solicitudes no procesadas debido a errores de usuario durante un período determinado.<br/><br/> Unidad: recuento <br/> Tipo de agregación: total <br/> Dimensión: EntityName|
|Cuota de errores superada |El número de solicitudes que superó la cuota disponible. Para más información sobre las cuotas de Event Hubs, vea [este artículo](event-hubs-quotas.md).<br/><br/> Unidad: recuento <br/> Tipo de agregación: total <br/> Dimensión: EntityName|

## <a name="throughput-metrics"></a>Métricas de procesamiento

| Nombre de la métrica | Descripción |
| ------------------- | ----------------- |
|Solicitudes limitadas |Número de solicitudes que se han limitado porque se superó el uso de la unidad de procesamiento.<br/><br/> Unidad: recuento <br/> Tipo de agregación: total <br/> Dimensión: EntityName|

## <a name="message-metrics"></a>Métricas de mensaje

| Nombre de la métrica | Descripción |
| ------------------- | ----------------- |
|Mensajes entrantes |Número de eventos o mensajes enviados a Event Hubs durante un período determinado.<br/><br/> Unidad: recuento <br/> Tipo de agregación: total <br/> Dimensión: EntityName|
|Mensajes salientes |Número de eventos o mensajes recuperados de Event Hubs durante un período determinado.<br/><br/> Unidad: recuento <br/> Tipo de agregación: total <br/> Dimensión: EntityName|
|Bytes entrantes |El número de bytes enviado al servicio de Azure Event Hubs durante un período determinado.<br/><br/> Unidad: bytes <br/> Tipo de agregación: total <br/> Dimensión: EntityName|
|Bytes salientes |El número de bytes recuperado del servicio de Azure Event Hubs durante un período determinado.<br/><br/> Unidad: bytes <br/> Tipo de agregación: total <br/> Dimensión: EntityName|

## <a name="connection-metrics"></a>Métricas de conexión

| Nombre de la métrica | Descripción |
| ------------------- | ----------------- |
|ActiveConnections |Número de conexiones activas en un espacio de nombres, así como en una entidad.<br/><br/> Unidad: recuento <br/> Tipo de agregación: total <br/> Dimensión: EntityName|
|Conexiones abiertas |Número de conexiones abiertas.<br/><br/> Unidad: recuento <br/> Tipo de agregación: total <br/> Dimensión: EntityName|
|Conexiones cerradas |Número de conexiones cerradas.<br/><br/> Unidad: recuento <br/> Tipo de agregación: total <br/> Dimensión: EntityName|

## <a name="event-hubs-capture-metrics"></a>Métricas de captura de Event Hubs

Puede supervisar las métricas de captura de Event Hubs cuando se habilita la característica de captura de los centros de eventos. Las métricas siguientes describen lo que puede supervisar con la característica de captura habilitada.

| Nombre de la métrica | Descripción |
| ------------------- | ----------------- |
|Capture el trabajo pendiente |El número de bytes que se aún se tiene que capturar en el destino elegido.<br/><br/> Unidad: bytes <br/> Tipo de agregación: total <br/> Dimensión: EntityName|
|Mensajes capturados |El número de mensajes o eventos que se capturan en el destino elegido durante un período determinado.<br/><br/> Unidad: recuento <br/> Tipo de agregación: total <br/> Dimensión: EntityName|
|Bytes capturados |El número de bytes que se capturan en el destino elegido durante un período determinado.<br/><br/> Unidad: bytes <br/> Tipo de agregación: total <br/> Dimensión: EntityName|

## <a name="metrics-dimensions"></a>Dimensiones de métricas

Azure Event Hubs admite las siguientes dimensiones para las métricas de Azure Monitor. La adición de dimensiones a las métricas es opcional. Si no agrega dimensiones, las métricas se especifican en el nivel de espacio de nombres. 

| Nombre de la métrica | Descripción |
| ------------------- | ----------------- |
|EntityName| Event Hubs admite las entidades de concentrador de eventos en el espacio de nombres.|

## <a name="azure-monitor-integration-with-siem-tools"></a>Integración de Azure Monitor con las herramientas SIEM
El enrutamiento de los datos de supervisión (registros de actividad, registros de diagnóstico, etc.) a un centro de eventos con Azure Monitor permite conseguir una integración sencilla con las herramientas de Administración de eventos e información de seguridad (SIEM). Para más información, consulte los siguientes artículos o entradas de blog:

- [Flujo de datos de supervisión de Azure a un centro de eventos para que lo consuma una herramienta externa](../azure-monitor/platform/stream-monitoring-data-event-hubs.md)
- [Introducción a Azure Log Integration](/previous-versions/azure/security/fundamentals/azure-log-integration-overview)
- [Uso de Azure Monitor para su integración con herramientas SIEM](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

En el escenario en el que una herramienta SIEM consume datos de registro de un centro de eventos, si no ve ningún mensaje entrante o si ve mensajes entrantes pero no hay mensajes salientes en el gráfico de métricas, siga estos pasos:

- Si **no hay mensajes entrantes**, significa que el servicio Azure Monitor no está moviendo los registros de auditoría o diagnóstico al centro de eventos. En este caso, abra una incidencia de soporte técnico con el equipo de Azure Monitor. 
- Si hay mensajes entrantes, pero **no hay mensajes salientes**, significa que la aplicación SIEM no está leyendo los mensajes. Póngase en contacto con el proveedor de SIEM para determinar si la configuración del centro de eventos en esas aplicaciones es correcta.


## <a name="next-steps"></a>Pasos siguientes

* Vea [Información general sobre la supervisión en Microsoft Azure](../azure-monitor/overview.md).
* Ejemplo [Retrieve Azure Monitor metrics with .NET](https://github.com/Azure-Samples/monitor-dotnet-metrics-api) (Recuperación de métricas de Azure Monitor con .NET) en GitHub. 

Para obtener más información acerca de Event Hubs, visite los vínculos siguientes:

- Empiece a trabajar con un tutorial de Event Hubs
    - [.NET Core](get-started-dotnet-standard-send-v2.md)
    - [Java](get-started-java-send-v2.md)
    - [Python](get-started-python-send-v2.md)
    - [JavaScript](get-started-java-send-v2.md)
* [Preguntas más frecuentes sobre Event Hubs](event-hubs-faq.md)
* [Aplicaciones de ejemplo que usan Event Hubs](https://github.com/Azure/azure-event-hubs/tree/master/samples)

[1]: ./media/event-hubs-metrics-azure-monitor/event-hubs-monitor1.png
[2]: ./media/event-hubs-metrics-azure-monitor/event-hubs-monitor2.png
