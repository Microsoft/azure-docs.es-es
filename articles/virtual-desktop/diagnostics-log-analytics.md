---
title: 'Análisis de registros de diagnóstico de Windows Virtual Desktop: Azure'
description: Cómo usar el análisis de registros con la característica de diagnóstico de Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 05/27/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: f4b1207f85f87755c8c0f2b8e9935f7e88118df3
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88005117"
---
# <a name="use-log-analytics-for-the-diagnostics-feature"></a>Uso de Log Analytics para la característica de diagnóstico

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop con objetos de Windows Virtual Desktop de Azure Resource Manager. Si usa Windows Virtual Desktop (clásico) sin objetos de Azure Resource Manager, consulte [este artículo](./virtual-desktop-fall-2019/diagnostics-log-analytics-2019.md).

Windows Virtual Desktop usa [Azure Monitor](../azure-monitor/overview.md) con fines de supervisión y envío de alertas como muchos otros servicios de Azure. Esto permite a los administradores identificar problemas con una única interfaz. El servicio crea registros de actividad para las acciones de usuario y administrativas. Cada registro de actividad entra dentro de una de las siguientes categorías:

- Actividades de administración:
    - Haga un seguimiento para determinar si los intentos de cambiar objetos de Windows Virtual Desktop mediante API o PowerShell se realizan correctamente. Por ejemplo, ¿es posible crear correctamente un grupo de hosts mediante PowerShell?
- Fuente:
    - ¿Los usuarios pueden suscribirse correctamente a las áreas de trabajo?
    - ¿Los usuarios ven todos los recursos publicados en el cliente de Escritorio remoto?
- Conexiones:
    - Cuando los usuarios inician y finalizan las conexiones al servicio.
- Registro de host:
    - ¿Se registró correctamente el host de sesión con el servicio al establecer conexión?
- Errores:
    - ¿Los usuarios detectan algún problema con actividades específicas? Esta característica puede generar una tabla que haga un seguimiento de los datos de actividad, siempre y cuando la información esté asociada a las actividades.
- Puntos de control:
    - Pasos específicos que se han alcanzado como parte de la duración de una actividad. Por ejemplo, durante una sesión, se ha equilibrado la carga de un usuario en un host determinado y, posteriormente, se ha iniciado la sesión del usuario durante una conexión, etc.

Las conexiones que no lleguen a Windows Virtual Desktop no aparecerán en los resultados de diagnóstico, ya que el propio servicio de rol de diagnóstico forma parte de Windows Virtual Desktop. Se pueden producir problemas de conexión a Windows Virtual Desktop si el usuario está experimentando problemas de conectividad de red.

Azure Monitor le permite analizar los datos de Windows Virtual Desktop y revisar los contadores de rendimiento de la máquina virtual. Todo ello en la misma herramienta. En este artículo se explica cómo habilitar los diagnósticos en su entorno de Windows Virtual Desktop.

>[!NOTE]
>Para más información sobre cómo supervisar las máquinas virtuales en Azure, consulte [Supervisión de máquinas virtuales de Azure con Azure Monitor](../azure-monitor/insights/monitor-vm-azure.md). Asegúrese también de [comprobar los umbrales de los contadores de rendimiento](../virtual-desktop/virtual-desktop-fall-2019/deploy-diagnostics.md#windows-performance-counter-thresholds) para entender mejor su experiencia de usuario en el host de sesión.

## <a name="before-you-get-started"></a>Antes de comenzar

Antes de usar Log Analytics, deberá crear un área de trabajo. Para ello, siga las instrucciones de uno de los dos artículos siguientes:

- Si prefiere usar Azure Portal, consulte [Creación de un área de trabajo de Log Analytics en Azure Portal](../azure-monitor/learn/quick-create-workspace.md).
- Si prefiere usar PowerShell, consulte [Creación de un área de trabajo de Log Analytics con PowerShell](../azure-monitor/learn/quick-create-workspace-posh.md).

Después de crear el área de trabajo, siga las instrucciones del artículo [Conexión de equipos Windows a Azure Monitor](../azure-monitor/platform/agent-windows.md#obtain-workspace-id-and-key) para obtener la siguiente información:

- Identificador del área de trabajo
- Clave principal del área de trabajo

Necesitará esta información posteriormente en el proceso de instalación.

Asegúrese de comprobar la administración de permisos de Azure Monitor a fin de habilitar el acceso a los datos para los usuarios que supervisan y mantienen su entorno de Windows Virtual Desktop. Para más información, consulte [Introducción a roles, permisos y seguridad con Azure Monitor](../azure-monitor/platform/roles-permissions-security.md).

## <a name="push-diagnostics-data-to-your-workspace"></a>Inserción de datos de diagnóstico en el área de trabajo

Puede insertar datos de diagnóstico desde los objetos de Windows Virtual Desktop en Log Analytics para su área de trabajo. Puede configurar esta característica de inmediato al crear los objetos por primera vez.

Para configurar Log Analytics para un objeto nuevo:

1. Inicie sesión en Azure Portal y vaya a **Windows Virtual Desktop**.

2. Vaya hasta el objeto (por ejemplo, un grupo de hosts, un grupo de aplicaciones o un área de trabajo) para el que desee capturar registros y eventos.

3. Seleccione **Configuración de diagnóstico** en el menú del lado izquierdo de la pantalla.

4. Seleccione **Agregar configuración de diagnóstico** en el menú que aparece en el lado derecho de la pantalla.

    Las opciones que se muestran en la página Configuración de diagnóstico variarán en función del tipo de objeto que esté editando.

    Por ejemplo, al habilitar los diagnósticos para un grupo de aplicaciones, verá opciones para configurar los puntos de control, los errores y la administración. En el caso de las áreas de trabajo, estas categorías configuran una fuente para hacer un seguimiento cuando los usuarios se suscriben a la lista de aplicaciones. Para más información sobre la configuración de diagnóstico, consulte [Creación de una configuración de diagnóstico para recopilar registros de plataforma y métricas en Azure](../azure-monitor/platform/diagnostic-settings.md).

     >[!IMPORTANT]
     >Recuerde habilitar los diagnósticos para cada objeto de Azure Resource Manager que desee supervisar. Una vez que se haya habilitado el diagnóstico, habrá datos disponibles para las actividades. Esto puede tardar algunas horas después de la primera configuración.

5. Escriba un nombre para la configuración de opciones y, a continuación, seleccione **Enviar a Log Analytics**. El nombre que se utilice no debe tener espacios y deberá ajustarse a las [convenciones de nomenclatura de Azure](../azure-resource-manager/management/resource-name-rules.md). Como parte de los registros, puede seleccionar todas las opciones que desee agregar a Log Analytics, como punto de control, errores, administración, etc.

6. Seleccione **Guardar**.

>[!NOTE]
>Log Analytics permite transmitir datos a [Event Hubs](../event-hubs/event-hubs-about.md) o archivarlos en una cuenta de almacenamiento. Para más información sobre esta característica, consulte [Transmisión de datos de supervisión de Azure a un centro de eventos](../azure-monitor/platform/stream-monitoring-data-event-hubs.md) y [Archivado de registros de recurso de Azure en la cuenta de almacenamiento](../azure-monitor/platform/resource-logs-collect-storage.md).

## <a name="how-to-access-log-analytics"></a>Cómo acceder a Log Analytics

Puede acceder a las áreas de trabajo de Log Analytics en Azure Portal o Azure Monitor.

### <a name="access-log-analytics-on-a-log-analytics-workspace"></a>Acceso a Log Analytics en un área de trabajo de Log Analytics

1. Inicie sesión en Azure Portal.

2. Busque **Área de trabajo de Log Analytics**.

3. En Servicios, seleccione **Áreas de trabajo de Log Analytics**.

4. En la lista, seleccione el área de trabajo que configuró para el objeto de Windows Virtual Desktop.

5. En el área de trabajo, seleccione **Registros**. Puede filtrar la lista de menús mediante la función de **búsqueda**.

### <a name="access-log-analytics-on-azure-monitor"></a>Acceso a Log Analytics en Azure Monitor

1. Inicie sesión en Azure Portal.

2. Busque y seleccione **Monitor**.

3. Seleccione **Registros**.

4. Siga las instrucciones de la página Registro para establecer el ámbito de la consulta.

5. Ya está listo para consultar los diagnósticos. Todas las tablas de diagnóstico tienen un prefijo "WVD".

>[!NOTE]
>Para obtener información más detallada acerca de las tablas almacenadas en los registros de Azure Monitor, consulte la [referencia de datos de Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/reference/). Todas las tablas relacionadas con Windows Virtual Desktop tienen la etiqueta "WVD".

## <a name="cadence-for-sending-diagnostic-events"></a>Cadencia para enviar eventos de diagnóstico

Los eventos de diagnóstico se envían a Log Analytics cuando se completan.

Log Analytics solo informa en estos estados intermedios de las actividades de conexión:

- Iniciado: cuando un usuario selecciona y se conecta a una aplicación o un escritorio en el cliente de Escritorio remoto.
- Conectado: cuando el usuario se conecta correctamente a la VM donde se hospeda la aplicación o el escritorio.
- Completado: cuando el usuario o el servidor desconecta la sesión en la que tuvo lugar la actividad.

## <a name="example-queries"></a>Consultas de ejemplo

Acceda a consultas de ejemplo desde la interfaz de usuario de Azure Monitor Log Analytics:
1. Vaya al área de trabajo de Log Analytics y seleccione **Registros**. La interfaz de usuario de consulta de ejemplo se muestra de forma automática.
1. Cambie el filtro a **Categoría**.
1. Seleccione **Windows Virtual Desktop** para examinar las consultas disponibles.
1. Seleccione **Ejecutar** para ejecutar la consulta seleccionada.

Obtenga más información sobre la interfaz de consulta de ejemplo en [Consultas guardadas en Log Analytics de Azure Monitor](../azure-monitor/log-query/saved-queries.md).

La siguiente lista de consultas le permite examinar la información de conexión o los problemas de un solo usuario. Puede ejecutar estas consultas en el [editor de consultas de Log Analytics](../azure-monitor/log-query/get-started-portal.md#write-and-run-basic-queries). En todas las consultas, reemplace `userupn` por el UPN del usuario que desee buscar.


Para buscar todas las conexiones de un solo usuario:

```kusto
WVDConnections
|where UserName == "userupn"
|take 100
|sort by TimeGenerated asc, CorrelationId
```


Para buscar el número de veces que un usuario se conectó por día:

```kusto
WVDConnections
|where UserName == "userupn"
|take 100
|sort by TimeGenerated asc, CorrelationId
|summarize dcount(CorrelationId) by bin(TimeGenerated, 1d)
```

Para buscar la duración de la sesión por usuario:

```kusto
let Events = WVDConnections | where UserName == "userupn" ;
Events
| where State == "Connected"
| project CorrelationId , UserName, ResourceAlias , StartTime=TimeGenerated
| join (Events
| where State == "Completed"
| project EndTime=TimeGenerated, CorrelationId)
on CorrelationId
| project Duration = EndTime - StartTime, ResourceAlias
| sort by Duration asc
```

Para buscar errores para un usuario específico:

```kusto
WVDErrors
| where UserName == "userupn"
|take 100
```

Para determinar si otros usuarios han tenido un error específico:

```kusto
WVDErrors
| where CodeSymbolic =="ErrorSymbolicCode"
| summarize count(UserName) by CodeSymbolic
```


>[!NOTE]
>- Cuando un usuario abre el escritorio completo, no se realiza un seguimiento de su uso de las aplicaciones en la sesión con puntos de control en la tabla WVDCheckpoints.
>- La columna ResourcesAlias de la tabla WVDConnections muestra si algún usuario se ha conectado a un escritorio completo o a una aplicación publicada. La columna solo muestra la primera aplicación que se abre durante la conexión. Se realiza un seguimiento de todas las aplicaciones publicadas que el usuario abra en WVDCheckpoints.
>- En la tabla WVDErrors se muestran los errores de administración, los problemas de registro del host y otros problemas que se producen mientras el usuario se suscribe a una lista de aplicaciones o equipos de escritorio.
>- WVDErrors le ayuda a identificar los problemas que pueden resolver las tareas de administrador. El valor de ServiceError siempre indica "false" para esos tipos de problemas. Si ServiceError = "true", deberá notificar el problema a Microsoft. Asegúrese de especificar el valor de CorrelationID en los errores que escale.

## <a name="next-steps"></a>Pasos siguientes

Para revisar los escenarios de error comunes que la característica de diagnóstico puede identificar automáticamente, consulte [Identificación y diagnóstico de problemas](diagnostics-role-service.md#common-error-scenarios).
