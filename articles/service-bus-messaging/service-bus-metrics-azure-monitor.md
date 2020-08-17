---
title: Métricas de Azure Service Bus en Azure Monitor | Microsoft Docs
description: En este artículo se explica cómo usar Azure Monitor para supervisar las entidades de Service Bus (colas, temas y suscripciones).
ms.topic: article
ms.date: 07/15/2020
ms.openlocfilehash: 3081b46bebdba8e83e5584178b37aab2dffee599
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88065019"
---
# <a name="azure-service-bus-metrics-in-azure-monitor"></a>Métricas de Azure Service Bus en Azure Monitor

Las métricas de Service Bus le permiten conocer el estado de los recursos de la suscripción de Azure. Con un amplio conjunto de datos de métricas, puede evaluar el estado general de los recursos de Service Bus, no solo en el nivel de espacio de nombres, sino también en el nivel de entidad. Estas estadísticas pueden ser importantes, ya que le ayudan a supervisar el estado de Service Bus. Las métricas también pueden ayudarle a solucionar problemas de causa principal sin necesidad de ponerse en contacto con el soporte técnico de Azure.

Azure Monitor proporciona interfaces de usuario unificadas para la supervisión de distintos servicios de Azure. Para obtener más información, vea [Supervisión de Microsoft Azure](../azure-monitor/overview.md) y el ejemplo [Retrieve Azure Monitor metrics with .NET](https://github.com/Azure-Samples/monitor-dotnet-metrics-api) (Recuperación de métricas de Azure Monitor con .NET) en GitHub.

> [!IMPORTANT]
> Cuando no se haya realizado ninguna interacción con una entidad durante 2 horas, las métricas empezarán a mostrar "0" como valor hasta que la entidad deje de estar inactiva.

## <a name="access-metrics"></a>Acceso a la métrica

Azure Monitor proporciona varias maneras de tener acceso a las métricas. Puede acceder a las métricas desde [Azure Portal](https://portal.azure.com) o usar las API de Azure Monitor (REST y .NET) y soluciones de análisis como los registros de Azure Monitor y Event Hubs. Para más información, consulte [Métricas en Azure Monitor](../azure-monitor/platform/data-platform-metrics.md).

De forma predeterminada, las métricas están habilitadas y puede acceder a datos de los últimos 30 días. Si es necesario conservar los datos durante un periodo mayor, se pueden archivar en una cuenta de Azure Storage. Este valor se configura en la [configuración de diagnóstico](../azure-monitor/platform/diagnostic-settings.md) de Azure Monitor.

## <a name="access-metrics-in-the-portal"></a>Acceso a métricas del portal

Una vez transcurrido un tiempo, las métricas se pueden supervisar en [Azure Portal](https://portal.azure.com). En el ejemplo siguiente se muestra cómo ver las solicitudes correctas y las solicitudes entrantes en el nivel de cuenta:

![Captura de pantalla de la página Monitor: Métricas (versión preliminar) en Azure Portal.][1]

También puede tener acceso a las métricas directamente a través del espacio de nombres. Para ello, seleccione el espacio de nombres y, después, haga clic en **Métricas**. Para mostrar las métricas filtradas según el ámbito de la entidad, seleccione la entidad y, luego, haga clic en **Métricas**.

![Captura de pantalla de la página Monitor: Métricas (versión preliminar) filtrada para el ámbito de la entidad.][2]

Para ver métricas que admitan las dimensiones, debe filtrar por el valor de la dimensión que desee.

## <a name="billing"></a>Facturación

Las métricas y las alertas en Azure Monitor se cobran por alerta. Estos cargos deben estar disponibles en el portal cuando se configura la alerta y antes de guardarla. 

Otras soluciones que ingieren los datos de las métricas se facturan directamente según esas soluciones. Por ejemplo, se le facturará por Azure Storage si archiva datos de métricas en una cuenta de Azure Storage. También se le facturará por Log Analytics si transmite datos de métricas a Log Analytics para realizar análisis avanzados.

Las siguientes métricas ofrecen una visión general del estado de su servicio. 

> [!NOTE]
> Se están dejando de usar algunas métricas a medida que se mueven a otro nombre. Esto podría requerir que actualice las referencias. Las métricas que se marquen con la palabra clave "en desuso" no se admitirán en el futuro.

Los valores de las métricas se envían a Azure Monitor cada minuto. La granularidad de tiempo define el intervalo de tiempo para el que se presentan los valores de las métricas. El intervalo de tiempo compatible para todas las métricas de Service Bus es 1 minuto.

## <a name="request-metrics"></a>Solicitud de métricas

Cuenta el número de solicitudes de operaciones de datos y administración.

| Nombre de la métrica | Descripción |
| ------------------- | ----------------- |
| Solicitudes entrantes| Número de solicitudes realizadas al servicio de Service Bus durante un período determinado. <br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensión: Nombre de entidad|
|Solicitudes correctas|Número de solicitudes correctas realizadas al servicio de Service Bus durante un período determinado.<br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensión: Nombre de entidad|
|Errores del servidor|Número de solicitudes no procesadas debido a un error en el servicio de Service Bus durante un período determinado.<br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensión: Nombre de entidad|
|Errores de usuario (consulte la subsección siguiente)|Número de solicitudes no procesadas debido a errores de usuario durante un período determinado.<br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensión: Nombre de entidad|
|Solicitudes limitadas|Número de solicitudes que se han limitado porque se superó el uso.<br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensión: Nombre de entidad|

### <a name="user-errors"></a>Errores de usuario

Los dos tipos de errores siguientes se clasifican como errores de usuario:

1. Errores en el cliente (en HTTP, serían errores 400).
2. Errores que se producen al procesar los mensajes, como [MessageLockLostException](/dotnet/api/microsoft.azure.servicebus.messagelocklostexception).


## <a name="message-metrics"></a>Métricas de mensaje

| Nombre de la métrica | Descripción |
| ------------------- | ----------------- |
|Mensajes entrantes|Número de eventos o mensajes enviados a Service Bus durante un período determinado.<br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensión: Nombre de entidad|
|Mensajes salientes|Número de eventos o mensajes recibidos de Service Bus durante un período determinado.<br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensión: Nombre de entidad|
| error de Hadoop| Recuento de mensajes de una cola/tema. <br/><br/> Unidad: Count <br/> Tipo de agregación: Average <br/> Dimensión: Nombre de entidad |
| Mensajes activos| Recuento de mensajes activos de una cola/tema. <br/><br/> Unidad: Count <br/> Tipo de agregación: Average <br/> Dimensión: Nombre de entidad |
| Mensajes fallidos| Recuento de mensajes fallidos de una cola/tema. <br/><br/> Unidad: Count <br/> Tipo de agregación: Average <br/>Dimensión: Nombre de entidad |
| Mensajes programados| Recuento de mensajes programados de una cola/tema. <br/><br/> Unidad: Count <br/> Tipo de agregación: Average  <br/> Dimensión: Nombre de entidad |
| Size | Tamaño de una entidad (cola o tema) en bytes. <br/><br/>Unidad: Count <br/>Tipo de agregación: Average <br/>Dimensión: Nombre de entidad | 

> [!NOTE]
> Los valores de las métricas siguientes son valores de un momento dado. Es posible que los mensajes entrantes que se consumieron inmediatamente después de ese momento dado no se reflejen en estas métricas. 
> - error de Hadoop
> - Mensajes activos 
> - Mensajes fallidos 
> - Mensajes programados 

## <a name="connection-metrics"></a>Métricas de conexión

| Nombre de la métrica | Descripción |
| ------------------- | ----------------- |
|Conexiones activas|Número de conexiones activas en un espacio de nombres, así como en una entidad.<br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensión: Nombre de entidad|
|Conexiones abiertas |Número de conexiones abiertas.<br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensión: Nombre de entidad|
|Conexiones cerradas |Número de conexiones cerradas.<br/><br/> Unidad: Count <br/> Tipo de agregación: Total <br/> Dimensión: Nombre de entidad|

## <a name="resource-usage-metrics"></a>Métricas de uso de recursos

> [!NOTE] 
> Estas métricas están disponibles solo con el nivel **Premium**. 
> 
> Estas son las métricas importantes que supervisar en busca de posibles interrupciones de un espacio de nombres de nivel Premium: **Uso de CPU por espacio de nombres** y **Uso del tamaño de memoria por espacio de nombres**. [Configure alertas](../azure-monitor/platform/alerts-metric.md) para estas métricas usando Azure Monitor.
> 
> La otra métrica que se podría supervisar es: **Solicitudes limitadas**. Con todo, esto no debería ser un problema siempre y cuando el espacio de nombres permanezca dentro de sus límites de memoria, CPU y conexiones desacopladas correspondientes. Para más información, vea [Limitación en el nivel Prémium de Azure Service Bus](service-bus-throttling.md#throttling-in-azure-service-bus-premium-tier).

| Nombre de la métrica | Descripción |
| ------------------- | ----------------- |
|Uso de CPU por espacio de nombres|Porcentaje de uso de CPU del espacio de nombres.<br/><br/> Unidad: Percent <br/> Tipo de agregación: Máxima <br/> Dimensión: Nombre de entidad|
|Uso de tamaño de memoria por espacio de nombres|Porcentaje de uso de memoria del espacio de nombres.<br/><br/> Unidad: Percent <br/> Tipo de agregación: Máxima <br/> Dimensión: Nombre de entidad|

## <a name="metrics-dimensions"></a>Dimensiones de métricas

Azure Service Bus admite las siguientes dimensiones para las métricas de Azure Monitor. La adición de dimensiones a las métricas es opcional. Si no agrega dimensiones, las métricas se especifican en el nivel de espacio de nombres. 

|Nombre de dimensión|Descripción|
| ------------------- | ----------------- |
|Nombre de entidad| Service Bus admite las entidades de mensajería en el espacio de nombres.|

## <a name="set-up-alerts-on-metrics"></a>Configuración de alertas en métricas

1. En la pestaña **Métricas** de la página **Espacio de nombres de Service Bus**, seleccione **Configurar alertas**. 

    ![Página Métricas; menú Configurar alertas](./media/service-bus-metrics-azure-monitor/metrics-page-configure-alerts-menu.png)
2. Seleccione la opción **Seleccionar destino**, y realice las acciones siguientes en la página **Seleccionar un recurso**: 
    1. Seleccione **Espacios de nombres de Service Bus** en el campo **Filtrar por tipo de recurso**. 
    2. Seleccione su suscripción en el campo **Filtrar por suscripción**.
    3. Seleccione el **espacio de nombres de Service Bus** en la lista. 
    4. Seleccione **Listo**. 
    
        ![Seleccionar espacio de nombres](./media/service-bus-metrics-azure-monitor/select-namespace.png)
1. Seleccione **Agregar criterios** y realice las acciones siguientes en la página **Configurar lógica de señal**:
    1. Seleccione **Métricas** para **Tipo de señal**. 
    2. Seleccione una señal. Por ejemplo: **Errores de servicio**. 

        ![Seleccionar errores del servidor](./media/service-bus-metrics-azure-monitor/select-server-errors.png)
    1. Seleccione **Mayor que** en **Condición**.
    2. Seleccione **Total** para **Agregación de tiempo**. 
    3. Escriba **5** en **Umbral**. 
    4. Seleccione **Listo**.    

        ![Especificar condición](./media/service-bus-metrics-azure-monitor/specify-condition.png)    
1. En la página **Crear regla**, expanda **Definir detalles de alerta** y realice las acciones siguientes:
    1. Escriba un **Nombre** para la alerta. 
    2. Escriba una **descripción** para la alerta.
    3. Seleccione la **gravedad** de la alerta. 

        ![Detalles de alertas](./media/service-bus-metrics-azure-monitor/alert-details.png)
1. En la página **Crear regla**, expanda **Definir grupo de acciones**, seleccione **Nuevo grupo de acciones**, y realice las acciones siguientes en la página **Agregar grupo de acciones**. 
    1. Escriba un nombre para el grupo de acciones.
    2. Escriba un nombre corto para el grupo de acciones. 
    3. Seleccione su suscripción. 
    4. Seleccione un grupo de recursos. 
    5. En este tutorial, escriba **Enviar correo electrónico** para **NOMBRE DE ACCIÓN**.
    6. Seleccione **Correo electrónico/SMS/Push/Voz** para **TIPO DE ACCIÓN**. 
    7. Seleccione **Editar detalles**. 
    8. En la página **Correo electrónico/SMS/Push/Voz**, realice las siguientes acciones:
        1. Seleccione **Correo electrónico**. 
        2. Escriba la **dirección de correo electrónico**. 
        3. Seleccione **Aceptar**.

            ![Detalles de alertas](./media/service-bus-metrics-azure-monitor/add-action-group.png)
        4. En la página **Agregar grupo de acciones**, seleccione **Aceptar**. 
1. En la página **Crear regla**, seleccione **Crear regla de alertas**. 

    ![Botón Crear regla de alertas](./media/service-bus-metrics-azure-monitor/create-alert-rule.png)

## <a name="next-steps"></a>Pasos siguientes

Consulte la [información general sobre Azure Monitor](../azure-monitor/overview.md).

[1]: ./media/service-bus-metrics-azure-monitor/service-bus-monitor1.png
[2]: ./media/service-bus-metrics-azure-monitor/service-bus-monitor2.png