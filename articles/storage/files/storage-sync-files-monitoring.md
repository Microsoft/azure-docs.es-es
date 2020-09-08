---
title: Supervisión de Azure File Sync | Microsoft Docs
description: Examine cómo supervisar la implementación de Azure File Sync mediante Azure Monitor, el Servicio de sincronización de almacenamiento y Windows Server.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 08/05/2019
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 9db8a0397c836e8cbc45404d9c4f149255fc76fa
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88271063"
---
# <a name="monitor-azure-file-sync"></a>Supervisión de Azure File Sync

Use Azure File Sync para centralizar los recursos compartidos de archivos de su organización en Azure Files sin renunciar a la flexibilidad, el rendimiento y la compatibilidad de un servidor de archivos local. Azure File Sync transforma Windows Server en una caché rápida de los recursos compartidos de archivos de Azure. Puede usar cualquier protocolo disponible en Windows Server para acceder a sus datos localmente, como SMB, NFS y FTPS. Puede tener todas las cachés que necesite en todo el mundo.

En este artículo se describe cómo supervisar la implementación de Azure File Sync mediante Azure Monitor, el Servicio de sincronización de almacenamiento y Windows Server.

Esta guía abarca los siguientes escenarios: 
- Visualización de las métricas de Azure File Sync en Azure Monitor.
- Creación de alertas en Azure Monitor para que le notifiquen proactivamente las condiciones críticas.
- Visualización del estado de la implementación de Azure File Sync mediante Azure Portal.
- Uso de los registros de eventos y los contadores de rendimiento en los servidores de Windows Server para supervisar el estado de la implementación de Azure File Sync. 

## <a name="azure-monitor"></a>Azure Monitor

Use [Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/overview) para ver las métricas y configurar alertas para la sincronización, nube por niveles y conectividad del servidor.  

### <a name="metrics"></a>Métricas

Las métricas de Azure File Sync están habilitadas de forma predeterminada y se envían a Azure Monitor cada 15 minutos.

**Visualización de las métricas de Azure File Sync en Azure Monitor**
- Vaya a su **Servicio de sincronización de almacenamiento** en **Azure Portal** y haga clic en **Métricas**.
- Haga clic en la lista desplegable **Métrica** y seleccione la métrica que desea ver.

Las siguientes métricas para Azure File Sync están disponibles en Azure Monitor:

| Nombre de métrica | Descripción |
|-|-|
| Bytes sincronizados | Tamaño de los datos transferidos (carga y descarga).<br><br>Unidad: Bytes<br>Tipo de agregación: Sum<br>Dimensiones aplicables: Nombre de punto de conexión del servidor, dirección de sincronización, nombre del grupo de sincronización |
| Recuperación de niveles de la nube | Tamaño de los datos que se recuperan.<br><br>**Nota**: esta métrica se eliminará en el futuro. Use la métrica del tamaño de recuperación de nube por niveles para supervisar el tamaño de los datos que se recuperan.<br><br>Unidad: Bytes<br>Tipo de agregación: Sum<br>Dimensión aplicable: Nombre del servidor |
| Tamaño de recuperación de nube por niveles | Tamaño de los datos que se recuperan.<br><br>Unidad: Bytes<br>Tipo de agregación: Sum<br>Dimensión aplicable: nombre del servidor, nombre del grupo de sincronización |
| Tamaño de recuperación de nube por niveles por aplicación | Tamaño de los datos recuperados por aplicación<br><br>Unidad: Bytes<br>Tipo de agregación: Sum<br>Dimensión aplicable: nombre de la aplicación, nombre del servidor, nombre del grupo de sincronización |
| Rendimiento de recuperación de nube por niveles | Tamaño del rendimiento de recuperación de datos.<br><br>Unidad: Bytes<br>Tipo de agregación: Sum<br>Dimensión aplicable: nombre del servidor, nombre del grupo de sincronización |
| Archivos que no se están sincronizando | Recuento de archivos que no se pueden sincronizar.<br><br>Unidad: Count<br>Tipo de agregación: Sum<br>Dimensiones aplicables: Nombre de punto de conexión del servidor, dirección de sincronización, nombre del grupo de sincronización |
| Archivos sincronizados | Recuento de los ficheros transferidos (carga y descarga).<br><br>Unidad: Count<br>Tipo de agregación: Sum<br>Dimensiones aplicables: Nombre de punto de conexión del servidor, dirección de sincronización, nombre del grupo de sincronización |
| Estado en línea del servidor | Recuento de latidos recibido del servidor.<br><br>Unidad: Count<br>Tipo de agregación: Máxima<br>Dimensión aplicable: Nombre del servidor |
| Sync session result (Resultado de la sesión de sincronización) | Resultado de la sesión de sincronización (1 = sesión de sincronización correcta; 0 = sesión de sincronización con errores)<br><br>Unidad: Count<br>Tipos de agregación: Máxima<br>Dimensiones aplicables: Nombre de punto de conexión del servidor, dirección de sincronización, nombre del grupo de sincronización |

### <a name="alerts"></a>Alertas

Las alertas le informan de forma proactiva cuando se detectan condiciones importantes en los datos que supervisa. Para obtener más información sobre cómo configurar alertas en Azure Monitor, consulte [Introducción sobre las alertas en Microsoft Azure](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-overview).

**Creación de alertas para Azure File Sync**

- Vaya a su **Servicio de sincronización de almacenamiento** en **Azure Portal**. 
- En la sección Supervisión, haga clic en **Alertas** y, después, haga clic en **+ Nueva regla de alertas**.
- Haga clic en **Seleccionar condición** y especifique la siguiente información para la alerta: 
    - **Métrica**
    - **Nombre de dimensión**
    - **Lógica de alerta**
- Haga clic en **Seleccionar el grupo de acciones** y agregue un grupo de acciones (correo electrónico, SMS, etc.) a la alerta, para lo que puede seleccionar un grupo de acciones existente o crear uno nuevo.
- Rellene los **detalles de la alerta**, como el **nombre de la regla de alertas**, la **descripción** y la **gravedad**.
- Haga clic en **Crear regla de alerta** para crear la alerta.  

En la tabla siguiente se muestran algunos escenarios de ejemplo que se van a supervisar y la métrica adecuada que se va a usar para la alerta:

| Escenario | Métrica que se va a usar para la alerta |
|-|-|
| El estado del punto de conexión del servidor muestra un error en el portal | Sync session result (Resultado de la sesión de sincronización) |
| Los archivos no se pueden sincronizar con un servidor o un punto de conexión en la nube | Archivos que no se están sincronizando |
| El servidor registrado no se puede comunicar con el Servicio de sincronización de almacenamiento | Estado en línea del servidor |
| El tamaño de recuperación de nube por niveles ha superado 500 GiB en un día  | Tamaño de recuperación de nube por niveles |

Para obtener las instrucciones necesarias para crear alertas para estos escenarios, consulte la sección [Ejemplos de alertas](#alert-examples).

## <a name="storage-sync-service"></a>Servicio de sincronización de almacenamiento

Para ver el estado de la implementación de Azure File Sync en **Azure Portal**, vaya al **Servicio de sincronización de almacenamiento**, donde estará disponible la siguiente información:

- Estado del servidor registrado
- Estado del punto de conexión del servidor
    - Archivos que no se están sincronizando
    - Actividad de sincronización
    - Eficiencia de la nube por niveles
    - Archivos que no se organizan por niveles
    - Errores de recuperación
- Métricas

### <a name="registered-server-health"></a>Estado del servidor registrado

Para ver en el portal el **estado del servidor registrado**, vaya a la sección **Servidores registrados** del **Servicio de sincronización de almacenamiento**.

- Si el estado del **Servidor registrado** es **En línea**, el servidor se está comunicando correctamente con el servicio.
- Si el estado del **Servidor registrado** es **Aparece sin conexión**, el proceso del monitor de sincronización de almacenamiento (AzureStorageSyncMonitor.exe) o el servidor no puede acceder al servicio Azure File Sync. Para obtener una guía, consulte la [documentación para la solución de problemas](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cazure-portal#server-endpoint-noactivity).

### <a name="server-endpoint-health"></a>Estado del punto de conexión del servidor

Para ver el estado de un **punto de conexión del servidor** en el portal, vaya a la sección **Grupos de sincronización** del **Servicio de sincronización de almacenamiento** y seleccione un **grupo de sincronización**.

- El **estado del punto de conexión del servidor** y la **actividad de sincronización** en el portal se basan en los eventos de sincronización que se registran en el registro de eventos de telemetría en el servidor (identificadores 9102 y 9302). Si se produce un error en una sesión de sincronización debido a un error transitorio como, por ejemplo, un error cancelado, la sincronización seguirá apareciendo en el portal como correcta, siempre y cuando la sesión de sincronización actual esté en progreso (se aplican los archivos). El identificador de evento 9302 es el evento del progreso de la sincronización y el 9102 se registra una vez que se completa la sesión de sincronización.  Para obtener más información, consulte [estado de la sincronización](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=server%2Cazure-portal#broken-sync) y [progreso de la sincronización](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=server%2Cazure-portal#how-do-i-monitor-the-progress-of-a-current-sync-session). Si el portal muestra un error de debido a que la sincronización no avanza, consulte la [documentación para la solución de problemas](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cazure-portal#common-sync-errors) para obtener instrucciones.
- El recuento de **archivos que no se están sincronizando** en el portal se basa en el identificador de evento 9121, que se registra en el registro de eventos de telemetría del servidor. Este evento se registra en cada error por elemento una vez que se completa la sesión de sincronización. Para resolver los errores por elemento, consulte el apartado [¿Cómo puedo ver si hay archivos o carpetas específicos que no se están sincronizando?](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=server%2Cazure-portal#how-do-i-see-if-there-are-specific-files-or-folders-that-are-not-syncing)
- Para ver la **eficiencia de la nube por niveles** en el portal, vaya a **Propiedades del punto de conexión de servidor** y vaya a la sección **Nube por niveles**. Los datos que se proporcionan para la eficiencia de la nube por niveles se basan en el identificador de evento 9071, que se registra en el registro de eventos de telemetría del servidor. Para obtener más información, consulte la [información general de nube por niveles](https://docs.microsoft.com/azure/storage/files/storage-sync-cloud-tiering).
- Para ver **archivos que no se organizan por niveles** y **errores de recuperación** en el portal, vaya a **Propiedades del punto de conexión de servidor** y vaya a la sección **Nube por niveles**. Los **archivos que no se organizan por niveles** están basados en el identificador de evento 9003, que se registra en el registro de eventos de telemetría del servidor y los **errores de recuperación** están basados en el 9006. Para investigar los archivos que no se organizan por niveles o se recuperan, consulte [Solución de problemas de archivos que no se organizan por niveles](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cazure-portal#how-to-troubleshoot-files-that-fail-to-tier) y [Solución de problemas de archivos que no se recuperan](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cazure-portal#how-to-troubleshoot-files-that-fail-to-be-recalled).

### <a name="metric-charts"></a>Gráficos de métricas

- Los siguientes gráficos de métricas están visibles en el portal del Servicio de sincronización de almacenamiento:

  | Nombre de métrica | Descripción | Nombre de la hoja |
  |-|-|-|
  | Bytes sincronizados | Tamaño de los datos transferidos (carga y descarga) | Grupo de sincronización,punto de conexión de servidor |
  | Recuperación de niveles de la nube | El tamaño de los datos que se recuperan | Servidores registrados |
  | Archivos que no se están sincronizando | Recuento de archivos que no se pueden sincronizar | Punto de conexión de servidor |
  | Archivos sincronizados | Recuento de los ficheros transferidos (carga y descarga) | Grupo de sincronización,punto de conexión de servidor |
  | Estado en línea del servidor | Recuento de latidos recibido del servidor | Servidores registrados |

- Para obtener más información, consulte [Azure Monitor](https://docs.microsoft.com/azure/storage/files/storage-sync-files-monitoring#azure-monitor).

  > [!Note]  
  > Los gráficos del portal del servicio de sincronización de almacenamiento tienen un intervalo de tiempo de 24 horas. Para ver los diferentes intervalos de tiempo o dimensiones, use Azure Monitor.

## <a name="windows-server"></a>Windows Server

En el **servidor de Windows Server** que tiene el agente de Azure File Sync instalado, puede ver el estado de los puntos de conexión de ese servidor mediante los **registros de eventos** y los **contadores de rendimiento**.

### <a name="event-logs"></a>Registros de eventos

Usar el registro Eventos de telemetría en el servidor para supervisar el servidor registrado, la sincronización y el estado de la nube por niveles. El registro de Eventos de telemetría se encuentra en el Visor de eventos en *Applications and Services\Microsoft\FileSync\Agent*.

Estado de la sincronización

- El id. de evento 9102 se registra una vez finalizada una sesión de sincronización. Use este evento para determinar si las sesiones de sincronización son correctas (**HResult = 0**) y si hay errores de sincronización por elemento (**PerItemErrorCount**). Para obtener más información, consulte la documentación [estado de la sincronización](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=server%2Cazure-portal#broken-sync) y [errores por elemento](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=server%2Cazure-portal#how-do-i-see-if-there-are-specific-files-or-folders-that-are-not-syncing).

  > [!Note]  
  > A veces, las sesiones de sincronización producen un error general o tienen un elemento PerItemErrorCount distinto de cero. Sin embargo, aún así hacen progresos y algunos archivos se sincronizan correctamente. Se puede observar esto en campos Applied como, por ejemplo, AppliedFileCount, AppliedDirCount, AppliedTombstoneCount y AppliedSizeBytes. Estos campos indican qué parte de la sesión se ha realizado correctamente. Si ve varias sesiones de sincronización seguidas que producen un error pero que tienen un número cada vez mayor de campos Applied, debe dar tiempo para que se vuelva a intentar la sincronización antes de abrir una incidencia de soporte técnico.

- El identificador de evento 9121 se registra en cada error por elemento una vez que se completa la sesión. Utilice este evento para determinar el número de archivos que no se sincronizan con este error (**PersistentCount** y **TransientCount**). Los errores por elemento persistentes deben investigarse, consulte el apartado [¿Cómo puedo ver si hay archivos o carpetas específicos que no se están sincronizando?](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=server%2Cazure-portal#how-do-i-see-if-there-are-specific-files-or-folders-that-are-not-syncing)

- El id. de evento 9302 se registra cada 5 a 10 minutos si hay una sesión de sincronización activa. Use este evento para determinar si la sesión de sincronización actual está en progreso (**AppliedItemCount > 0**). Si la sincronización no avanza, la sesión de sincronización finalmente generará un error y se registrará el id. de evento 9102 con el error. Para obtener más información, consulte la [documentación sobre progreso de la sincronización](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=server%2Cazure-portal#how-do-i-monitor-the-progress-of-a-current-sync-session).

Estado del servidor registrado

- El id. de evento 9301 se registra cada 30 segundos, cuando un servidor consulta al servicio por los trabajos. Si GetNextJob se completa con **status = 0**, el servidor puede comunicarse con el servicio. Si GetNextJob se completa con un error, compruebe la [documentación de solución de problemas](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cazure-portal#server-endpoint-noactivity) para obtener instrucciones.

Estado de la nube por niveles

- Para supervisar la actividad por niveles en un servidor, use los id. de eventos 9003, 9016 y 9029 en el registro de Eventos de telemetría, ubicado en el Visor de eventos en *Applications and Services\Microsoft\FileSync\Agent*.

  - El identificador de evento 9003 proporciona una distribución de errores para un punto de conexión de servidor. Por ejemplo: Recuento total de errores y ErrorCode. Se registra un evento por código de error.
  - El identificador de evento 9016 proporciona resultados de conversión en fantasma para un volumen. Por ejemplo: el porcentaje de espacio libre, el Número de archivos reflejados en la sesión y el Número de archivos no reflejados.
  - El id. de evento 9029 proporciona información de la sesión de conversión en fantasma para el punto de conexión de un servidor. Por ejemplo: el Número de archivos que se han intentado en la sesión, el Número de archivos organizados en niveles de la sesión y el Número de archivos ya organizados en niveles.
  
- Para supervisar la actividad de recuperación en un servidor, use los id. de evento 9005, 9006, 9009, 9059 y 9071 en el registro de Eventos de telemetría, ubicado en el Visor de eventos en *Aplicaciones y servicios\Microsoft\FileSync\Agent*.

  - El identificador de evento 9005 proporciona confiabilidad de recuperación a un punto de conexión de servidor. Por ejemplo: el Total de archivos únicos a los que se puede acceder y el Total de archivos únicos con acceso erróneo.
  - El identificador de evento 9006 proporciona una distribución de errores de recuperación a un punto de conexión de servidor. Por ejemplo: Total de solicitudes con error y ErrorCode. Se registra un evento por código de error.
  - El id. de evento 9009 proporciona información de la sesión de recuperación para un punto de conexión de servidor. Por ejemplo: DurationSeconds, CountFilesRecallSucceeded y CountFilesRecallFailed.
  - El id. de evento 9059 proporciona una distribución de recuperación de aplicación para un punto de conexión de servidor. Por ejemplo: ShareId, Nombre de la aplicación y TotalEgressNetworkBytes.
  - El identificador de evento 9071 proporciona la eficiencia de la nube por niveles para un punto de conexión de servidor. Por ejemplo: TotalDistinctFileCountCacheHit, TotalDistinctFileCountCacheMiss, TotalCacheHitBytes y TotalCacheMissBytes.

### <a name="performance-counters"></a>Contadores de rendimiento

Use los contadores de rendimiento de Azure File Sync en el servidor para supervisar la actividad de sincronización.

Para ver los contadores de rendimiento de Azure File Sync en el servidor, abra el Monitor de rendimiento (Perfmon.exe). Puede encontrar los contadores en los objetos **AFS Bytes Transferred** y **AFS Sync Operations**.

Los siguientes contadores de rendimiento para Azure File Sync están disponibles en el Monitor de rendimiento:

| Objeto de rendimiento\Nombre del contador | Descripción |
|-|-|
| AFS Bytes Transferred\Downloaded Bytes/sec (Bytes descargados/s) | Número de bytes descargados por segundo. |
| AFS Bytes Transferred\Uploaded Bytes/sec (Bytes cargados/s) | Número de bytes cargados por segundo. |
| AFS Bytes Transferred\Total Bytes/sec (Total de bytes/s) | Número total de bytes por segundo (carga y descarga). |
| AFS Sync Operations\Downloaded Sync Files/sec (Archivos de sincronización descargados/s) | Número de archivos descargados por segundo. |
| AFS Sync Operations\Uploaded Sync Files/sec (Archivos de sincronización cargados/s) | Número de archivos cargados por segundo. |
| AFS Sync Operations\Total Sync File Operations/sec (Total de operaciones de sincronización de archivos/s) | Número total de archivos sincronizados (carga y descarga). |

## <a name="alert-examples"></a>Ejemplos de alertas
En esta sección se proporcionan varias alertas de ejemplo para Azure File Sync.

  > [!Note]  
  > Si crea una alerta y esta produce demasiados resultados irrelevantes, ajuste el valor del umbral y la lógica de alerta.
  
### <a name="how-to-create-an-alert-if-the-server-endpoint-health-shows-an-error-in-the-portal"></a>Creación de una alerta si el estado del punto de conexión del servidor muestra un error en el portal

1. En **Azure Portal**, vaya al **servicio de sincronización de almacenamiento** respectivo. 
2. Vaya a la sección **Supervisión** y haga clic en **Alertas**. 
3. Haga clic en **+ Nueva regla de alertas** para crear una regla de alertas. 
4. Configure la condición, para lo que debe hacer clic en **Seleccionar condición**.
5. En la hoja **Configurar lógica de señal**, haga clic en **Sync session result** (Resultado de la sesión de sincronización) en el nombre de la señal.  
6. Seleccione la siguiente configuración de la dimensión: 
    - Nombre de la dimensión: **Nombre del punto de conexión del servidor**  
    - Operador: **=** 
    - Valores de la dimensión: **Todos los valores actuales y futuros**  
7. Vaya a **Lógica de alerta** y complete los siguientes valores: 
    - Umbral establecido en **Estático** 
    - Operador: **Menor que** 
    - Tipo de agregación: **Máximo**  
    - Valor del umbral: **1** 
    - Se evaluó basándose en: Granularidad de agregación = **24 horas** | Frecuencia de evaluación = **Cada hora** 
    - Haga clic en **Listo**. 
8. Haga clic en **Seleccionar el grupo de acciones** para agregar un grupo de acciones (correo electrónico, SMS, etc.) a la alerta, para lo que puede seleccionar un grupo de acciones existente o crear uno nuevo.
9. Rellene los **detalles de la alerta**, como el **nombre de la regla de alertas**, la **descripción** y la **gravedad**.
10. Haga clic en **Crear regla de alertas**. 

### <a name="how-to-create-an-alert-if-files-are-failing-to-sync-to-a-server-or-cloud-endpoint"></a>Creación de una alerta si los archivos no se pueden sincronizar con un servidor o un punto de conexión en la nube

1. En **Azure Portal**, vaya al **servicio de sincronización de almacenamiento** respectivo. 
2. Vaya a la sección **Supervisión** y haga clic en **Alertas**. 
3. Haga clic en **+ Nueva regla de alertas** para crear una regla de alertas. 
4. Configure la condición, para lo que debe hacer clic en **Seleccionar condición**.
5. En la hoja **Configurar lógica de señal**, haga clic en **Archivos que no se están sincronizando** en el nombre de la señal.  
6. Seleccione la siguiente configuración de la dimensión: 
     - Nombre de la dimensión: **Nombre del punto de conexión del servidor**  
     - Operador: **=** 
     - Valores de la dimensión: **Todos los valores actuales y futuros**  
7. Vaya a **Lógica de alerta** y complete los siguientes valores: 
     - Umbral establecido en **Estático** 
     - Operador: **Mayor que** 
     - Tipo de agregación: **Total**  
     - Valor del umbral: **100** 
     - Se evaluó basándose en: Granularidad de agregación = **5 minutos** | Frecuencia de evaluación = **Cada 5 minutos** 
     - Haga clic en **Listo**. 
8. Haga clic en **Seleccionar el grupo de acciones** para agregar un grupo de acciones (correo electrónico, SMS, etc.) a la alerta, para lo que puede seleccionar un grupo de acciones existente o crear uno nuevo.
9. Rellene los **detalles de la alerta**, como el **nombre de la regla de alertas**, la **descripción** y la **gravedad**.
10. Haga clic en **Crear regla de alertas**. 

### <a name="how-to-create-an-alert-if-a-registered-server-is-failing-to-communicate-with-the-storage-sync-service"></a>Creación de una alerta si un servidor registrado no se puede comunicar con el Servicio de sincronización de almacenamiento

1. En **Azure Portal**, vaya al **servicio de sincronización de almacenamiento** respectivo. 
2. Vaya a la sección **Supervisión** y haga clic en **Alertas**. 
3. Haga clic en **+ Nueva regla de alertas** para crear una regla de alertas. 
4. Configure la condición, para lo que debe hacer clic en **Seleccionar condición**.
5. En la hoja **Configurar lógica de señal**, haga clic en **Estado de conexión del servidor** en el nombre de la señal.  
6. Seleccione la siguiente configuración de la dimensión: 
     - Nombre de dimensión: **Nombre del servidor**  
     - Operador: **=** 
     - Valores de la dimensión: **Todos los valores actuales y futuros**  
7. Vaya a **Lógica de alerta** y complete los siguientes valores: 
     - Umbral establecido en **Estático** 
     - Operador: **Menor que** 
     - Tipo de agregación: **Máximo**  
     - Valor de umbral (en bytes): **1** 
     - Se evaluó basándose en: Granularidad de agregación = **1 hora** | Frecuencia de evaluación = **Cada 30 minutos** 
     - Haga clic en **Listo**. 
8. Haga clic en **Seleccionar el grupo de acciones** para agregar un grupo de acciones (correo electrónico, SMS, etc.) a la alerta, para lo que puede seleccionar un grupo de acciones existente o crear uno nuevo.
9. Rellene los **detalles de la alerta**, como el **nombre de la regla de alertas**, la **descripción** y la **gravedad**.
10. Haga clic en **Crear regla de alertas**. 

### <a name="how-to-create-an-alert-if-the-cloud-tiering-recall-size-has-exceeded-500gib-in-a-day"></a>Creación de una alerta si el tamaño de recuperación de nube por niveles ha superado 500 GiB en un día

1. En **Azure Portal**, vaya al **servicio de sincronización de almacenamiento** respectivo. 
2. Vaya a la sección **Supervisión** y haga clic en **Alertas**. 
3. Haga clic en **+ Nueva regla de alertas** para crear una regla de alertas. 
4. Configure la condición, para lo que debe hacer clic en **Seleccionar condición**.
5. En la hoja **Configurar lógica de señal**, haga clic en **Cloud tiering recall size** (Tamaño de recuperación de nube por niveles) en el nombre de la señal.  
6. Seleccione la siguiente configuración de la dimensión: 
     - Nombre de dimensión: **Nombre del servidor**  
     - Operador: **=** 
     - Valores de la dimensión: **Todos los valores actuales y futuros**  
7. Vaya a **Lógica de alerta** y complete los siguientes valores: 
     - Umbral establecido en **Estático** 
     - Operador: **Mayor que** 
     - Tipo de agregación: **Total**  
     - Valor de umbral (en bytes): **67108864000** 
     - Se evaluó basándose en: Granularidad de agregación = **24 horas** | Frecuencia de evaluación = **Cada hora** 
    - Haga clic en **Listo**. 
8. Haga clic en **Seleccionar el grupo de acciones** para agregar un grupo de acciones (correo electrónico, SMS, etc.) a la alerta, para lo que puede seleccionar un grupo de acciones existente o crear uno nuevo.
9. Rellene los **detalles de la alerta**, como el **nombre de la regla de alertas**, la **descripción** y la **gravedad**.
10. Haga clic en **Crear regla de alertas**. 

## <a name="next-steps"></a>Pasos siguientes
- [Planeamiento de una implementación de Azure File Sync](storage-sync-files-planning.md)
- [Tenga en cuenta los valores de proxy y firewall](storage-sync-files-firewall-and-proxy.md)
- [Implementación de Azure File Sync](storage-sync-files-deployment-guide.md)
- [Solución de problemas de Azure File Sync](storage-sync-files-troubleshoot.md)
- [Preguntas más frecuentes de Azure Files](storage-files-faq.md)
