---
title: Supervisión de métricas y registro en Azure Front Door | Microsoft Docs
description: Este artículo describe las diferentes métricas y registros de acceso que admite Azure Front Door.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/18/2018
ms.author: duau
ms.openlocfilehash: 6f5051dd7dedcc49320557f17148bcdc9bf539ab
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89399759"
---
# <a name="monitoring-metrics-and-logs-in-azure-front-door"></a>Supervisión de métricas y registro en Azure Front Door

Con Azure Front Door, puede supervisar los recursos de las siguientes maneras:

- **Métricas**. Azure Front Door tiene actualmente siete métricas para ver los contadores de rendimiento.
- **Registros**. Los registros de actividad y diagnóstico permiten que un recurso guarde o consuma datos de rendimiento, acceso u otros con fines de supervisión.

### <a name="metrics"></a>Métricas

Las métricas son una característica de determinados recursos de Azure en los que puede ver contadores de rendimiento en el portal. Las métricas siguientes están disponibles en Front Door:

| Métrica | Nombre de métrica para mostrar | Unidad | Dimensions | Descripción |
| --- | --- | --- | --- | --- |
| RequestCount | Recuento de solicitudes | Count | HttpStatus</br>HttpStatusGroup</br>ClientRegion</br>ClientCountry | Número de solicitudes de cliente que atiende Front Door.  |
| RequestSize | Tamaño de la solicitud | Bytes | HttpStatus</br>HttpStatusGroup</br>ClientRegion</br>ClientCountry | Número de bytes enviados como solicitudes de clientes a Front Door. |
| ResponseSize | Tamaño de la respuesta | Bytes | HttpStatus</br>HttpStatusGroup</br>ClientRegion</br>ClientCountry | Número de bytes enviados como respuestas de Front Door a los clientes. |
| TotalLatency | Latencia total | Milisegundos | HttpStatus</br>HttpStatusGroup</br>ClientRegion</br>ClientCountry | Tiempo calculado desde que Front Door recibe la solicitud de cliente hasta que el cliente reconoce el último byte de respuesta de Front Door. |
| BackendRequestCount | Recuento de solicitudes de back-end | Count | HttpStatus</br>HttpStatusGroup</br>Back-end | Número de solicitudes enviadas de Front Door a los servidores back-end. |
| BackendRequestLatency | Latencia de las solicitudes de back-end | Milisegundos | Back-end | Tiempo calculado desde que Front Door envía la solicitud al servidor back-end hasta que Front Door recibe el último byte de respuesta del servidor back-end. |
| BackendHealthPercentage | Porcentaje de estado del back-end | Percent | Back-end</br>BackendPool | El porcentaje de sondeos de estado correctos de Front Door a los servidores back-ends. |
| WebApplicationFirewallRequestCount | Recuento de solicitudes del firewall de aplicaciones web | Count | PolicyName</br>RuleName</br>Acción | Número de solicitudes de cliente procesadas por la seguridad del nivel de aplicación de Front Door. |

## <a name="activity-logs"></a><a name="activity-log"></a>Registros de actividad

Los registros de actividad proporcionan información sobre las operaciones realizadas en Front Door. También determinan qué, quién y cuándo para cualquier operación de escritura (put, post o delete) realizada en Front Door.

>[!NOTE]
>Los registros de actividad no incluyen operaciones de lectura (get). Tampoco incluyen operaciones realizadas con Azure Portal o la Management API original.

Acceda a registros de actividad en Front Door o a los registros de todos los recursos de Azure en Azure Monitor. Para ver los registros de actividad:

1. Seleccione la instancia de Front Door.
2. Seleccione **Registro de actividad**.

    ![Registro de actividades](./media/front-door-diagnostics/activity-log.png)

3. Elija un ámbito de filtrado y, a continuación, seleccione **Aplicar**.

## <a name="diagnostic-logs"></a><a name="diagnostic-logging"></a>Registros de diagnóstico
Los registros de diagnóstico proporcionan información valiosa acerca de las operaciones y los errores que son importantes para la auditoría, así como para solucionar problemas. Los registros de diagnóstico son diferentes de los registros de actividad.

Los registros de actividad proporcionan información sobre las operaciones llevadas a cabo en los recursos de Azure. Los registros de diagnóstico proporcionan conclusiones detalladas sobre las operaciones que realiza el recurso. Para más información, vea [Información general sobre los registros de diagnóstico de Azure](../azure-monitor/platform/platform-logs-overview.md).

![Registros de diagnóstico](./media/front-door-diagnostics/diagnostic-log.png)

Para configurar los registros de diagnóstico para Front Door:

1. Seleccione su instancia del servicio Azure Front Door.

2. Seleccione **Configuración de diagnóstico**.

3. Seleccione **Activar diagnósticos**. Archive los registros de diagnóstico junto con las métricas en una cuenta de almacenamiento, transmítalos en secuencias a un centro de eventos o envíelos a los registros de Azure Monitor.

Front Door actualmente proporciona los registros de diagnóstico (agrupados por lotes cada hora). Los registros de diagnóstico proporcionan solicitudes API individuales con cada entrada con el siguiente esquema:

| Propiedad  | Descripción |
| ------------- | ------------- |
| BackendHostname | Si la solicitud se reenvió a un back-end, este campo representa el nombre de host del back-end. Este campo estará en blanco si la solicitud se redirigió o reenvió a una caché regional (si el almacenamiento en caché está habilitado para la regla de enrutamiento). |
| CacheStatus | En el caso de los escenarios de almacenamiento en caché, este campo define el número de aciertos y errores de caché en el POP |
| ClientIp | Dirección IP del cliente que realizó la solicitud. Si hubiera un encabezado X-Forwarded-For en la solicitud, la dirección IP del cliente se seleccionaría del mismo. |
| ClientPort | Puerto IP del cliente que realizó la solicitud. |
| HttpMethod | Método HTTP utilizado por la solicitud. |
| HttpStatusCode | El código de estado HTTP devuelto desde el servidor proxy. |
| HttpStatusDetails | Estado resultante en la solicitud. El significado de este valor de cadena puede encontrarse en una tabla de referencia de estado. |
| HttpVersion | Tipo de la solicitud o conexión. |
| POP | Nombre corto del perímetro en el que ha aterrizado la solicitud. |
| RequestBytes | El tamaño del mensaje de solicitud HTTP en bytes, incluidos los encabezados de solicitud y el cuerpo de solicitud. |
| RequestUri | URI de la solicitud recibida. |
| ResponseBytes | Bytes enviados por el servidor back-end como respuesta.  |
| RoutingRuleName | El nombre de la regla de enrutamiento que coincidió con la solicitud. |
| RulesEngineMatchNames | Los nombres de las reglas que coincidieron con la solicitud. |
| SecurityProtocol | La versión del protocolo TLS/SSL utilizada por la solicitud o null si no hay cifrado. |
| SentToOriginShield | Campo booleano que representa si se produjo un error de caché en el primer entorno y la solicitud se envió a la caché regional. Omita este campo si la regla de enrutamiento es un redireccionamiento o si no tiene habilitado el almacenamiento en caché. |
| TimeTaken | Período de tiempo desde el primer byte de la solicitud en Front Door hasta el último byte de la respuesta, en segundos. |
| TrackingReference | La cadena de referencia exclusiva que identifica una solicitud atendida por Front Door, que también se envía como encabezado X-Azure-Ref al cliente. Se requiere para buscar los detalles en los registros de acceso para una solicitud específica. |
| UserAgent | Tipo de explorador utilizado por el cliente. |

**Nota:** En el caso de varias configuraciones de enrutamiento y comportamientos de tráfico, algunos de los campos como backendHostname, cacheStatus, sentToOriginShield y POP, pueden responder con valores diferentes. En la tabla siguiente se explican los distintos valores que estos campos tendrán para diversos escenarios:

| Escenarios | Recuento de entradas de registro | POP | BackendHostname | SentToOriginShield | CacheStatus |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| Regla de enrutamiento sin almacenamiento en caché habilitado | 1 | Código POP perimetral | El back-end al que se reenvió la solicitud | False | CONFIG_NOCACHE |
| Regla de enrutamiento con almacenamiento en caché habilitado. Aciertos de caché en el POP perimetral | 1 | Código POP perimetral | Vacío | False | HIT |
| Regla de enrutamiento con almacenamiento en caché habilitado. Error de caché en el POP perimetral pero acierto en el POP de la caché primaria | 2 | 1. Código POP perimetral</br>2. Código POP de caché primaria | 1. Nombre de host de POP de caché primaria</br>2. Vacío | 1. True</br>2. False | 1. MISS</br>2. PARTIAL_HIT |
| Regla de enrutamiento con almacenamiento en caché habilitado. Error de caché en el POP perimetral y en el de caché primaria | 2 | 1. Código POP perimetral</br>2. Código POP de caché primaria | 1. Nombre de host de POP de caché primaria</br>2. Back-end que ayuda a rellenar la memoria caché | 1. True</br>2. False | 1. MISS</br>2. MISS |


## <a name="next-steps"></a>Pasos siguientes

- [Crear un perfil de Front Door](quickstart-create-front-door.md)
- Información acerca de cómo [funciona Front Door](front-door-routing-architecture.md)
