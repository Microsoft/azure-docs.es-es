---
title: Descripción de las métricas de Azure Spring Cloud
description: Aprenda a revisar las métricas en Azure Spring Cloud
author: bmitchell287
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 12/06/2019
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 1fcec8b591192bba862bf91040e5d6ca1ef9bcd8
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87037736"
---
# <a name="understand-metrics-for-azure-spring-cloud"></a>Descripción de las métricas en Azure Spring Cloud

El explorador de métricas de Azure es un componente de Microsoft Azure Portal que permite trazar gráficos, correlacionar tendencias visualmente e investigar máximos y mínimos en las métricas. Utilice el Explorador de métricas para investigar el estado y la utilización de recursos. 

En Azure Spring Cloud, hay dos puntos de vista para las métricas.
* Gráficos en todas las páginas de información general de aplicaciones
* Página de métricas comunes

 ![Gráficos de métricas](media/metrics/metrics-1.png)

Los gráficos de la **Información general** de la aplicación proporcionan comprobaciones de estado rápidas para cada aplicación. La página de **Métricas** comunes contiene todas las métricas disponibles como referencia. Puede crear sus propios gráficos en la página de métricas comunes y anclarlos en el Panel.

## <a name="application-overview-page"></a>Página de información general de la aplicación
Seleccione una aplicación en **Aplicaciones** para buscar gráficos en la página de información general.  

 ![Administración de métricas de aplicación](media/metrics/metrics-2.png)

La página **Información general de aplicación** de cada aplicación muestra un gráfico de métricas que le permite realizar una comprobación rápida del estado de la aplicación.  

 ![Información general de métricas de aplicación](media/metrics/metrics-3.png)

Azure Spring Cloud proporciona estos cinco gráficos con métricas que se actualizan cada minuto:

* **Errores de servidor HTTP**: recuento de errores en las solicitudes HTTP a su aplicación
* **Entrada de datos**: bytes que ha recibido su aplicación
* **Salida de datos**: bytes que ha enviado su aplicación
* **Solicitud**: solicitudes que ha recibido su aplicación
* **Tiempo promedio de respuesta**: tiempo promedio de respuesta de su aplicación

En el gráfico, puede seleccionar un intervalo de tiempo de entre una hora y siete días.

## <a name="common-metrics-page"></a>Página de métricas comunes

El vínculo **Métricas** en el panel de navegación izquierdo abre la página de métricas comunes.

Primero, seleccione las métricas que quiere visualizar:

![Seleccionar la vista de métricas](media/metrics/metrics-4.png)

En la [sección](#user-metrics-options) siguiente puede encontrar información sobre las opciones de todas las métricas.

A continuación, seleccione el tipo de agregación para cada métrica:

![Agregación de métricas](media/metrics/metrics-5.png)

El tipo de agregación indica cómo agregar puntos de métrica en el gráfico por tiempo. Hay un punto de métrica sin procesar cada minuto y el tipo de agregación previa dentro de un minuto lo predefine el tipo de métrica.
* Suma: suma todos los valores como salida del destino.
* Promedio: use el valor promedio del período como salida del destino.
* Máx./mín.: Use el valor Máx./mín. del período como salida del destino.

El intervalo de tiempo también se puede ajustar entre los últimos 30 minutos y los últimos 30 días, o un intervalo de tiempo personalizado.

![Modificación de métricas](media/metrics/metrics-6.png)

La vista predeterminada reúne todas las métricas de las aplicaciones de un servicio de Azure Spring Cloud. Se pueden filtrar las métricas de una aplicación o instancia en la pantalla.  Haga clic en **Agregar filtro**, establezca la propiedad en **Aplicación** y seleccione la aplicación de destino que quiere supervisar en el cuadro de texto **Valores**. 

Puede usar dos tipos de filtros (propiedades):
* Aplicación: filtrar por nombre de la aplicación
* Instancia: filtrar por instancia de la aplicación

![Filtros de métricas](media/metrics/metrics-7.png)

También puede usar la opción **Aplicar división**, que dibujará varias líneas para una aplicación:

![División de métricas](media/metrics/metrics-8.png)

>[!TIP]
> Puede crear sus propios gráficos en la página de métricas y anclarlos a su **panel**. Empiece por asignar un nombre al gráfico.  Después, seleccione **Anclar al panel en la esquina superior derecha**. Ya puede comprobar la aplicación en el **panel** del portal.

## <a name="user-metrics-options"></a>Opciones de métricas de usuario

En las tablas siguientes se muestran las métricas y los detalles disponibles.

### <a name="error"></a>Error
>[!div class="mx-tdCol2BreakAll"]
>| Nombre | Nombre de la métrica de Spring Actuator | Unidad | Detalles |
>|----|----|----|------------|
>| tomcat.global.error | tomcat.global.error | Count | Número de errores que se produjeron en las solicitudes procesadas |

### <a name="performance"></a>Rendimiento
>[!div class="mx-tdCol2BreakAll"]
>| Nombre | Nombre de la métrica de Spring Actuator | Unidad | Detalles |
>|----|----|----|------------|
>| system.cpu.usage | system.cpu.usage | Percent | Uso de CPU reciente de todo el sistema. Este es un valor Double en el intervalo [0,0, 1,0]. Un valor de 0,0 indica que todas las CPU estuvieron inactivas durante el período de tiempo reciente observado, mientras que un valor de 1,0 indica que todas las CPU se estaban ejecutando activamente el 100 % del tiempo durante el último período observado.|
>| process.cpu.usage | App CPU Usage Percentage (Porcentaje de uso de CPU de la aplicación) | Percent | Uso reciente de la CPU para el proceso de Máquina virtual Java. Este es un valor Double en el intervalo [0,0, 1,0]. Un valor de 0,0 indica que ninguna de las CPU estaba ejecutando subprocesos del proceso de Máquina virtual Java durante el período de tiempo reciente observado, mientras que un valor de 1,0 indica que todas las CPU estaban ejecutando activamente subprocesos de la Máquina virtual Java el 100 % del tiempo durante el último período observado. Los subprocesos de la Máquina virtual Java incluyen los subprocesos de la aplicación, así como los subprocesos internos de Máquina virtual Java.|
>| jvm.memory.committed | jvm.memory.committed | Bytes | Representa la cantidad de memoria que se garantiza que va a estar disponible para que la use la Máquina virtual Java. Es posible que la Máquina virtual Java libere memoria para el sistema y que la memoria confirmada sea menor que la inicial. La memoria asignada siempre será mayor o igual que la utilizada. |
>| jvm.memory.used | jvm.memory.used | Bytes | Representa la cantidad de memoria que se usa actualmente en bytes. |
>| jvm.memory.max | jvm.memory.max | Bytes | Representa la cantidad máxima de memoria que se puede usar para la administración de memoria. La cantidad de memoria usada y asignada siempre será menor o igual al valor máximo, si se definió el máximo. Se puede producir un error de asignación de memoria si intenta aumentar la memoria usada de modo que la memoria usada sea mayor que la memoria asignada, incluso si la memoria usada es menor o igual que el valor máximo (por ejemplo, cuando el sistema tiene poca memoria virtual). |
>| jvm.gc.max.data.size | jvm.gc.max.data.size | Bytes | Uso de memoria máximo del bloque de memoria de generación anterior desde que se inició la máquina virtual Java. |
>| jvm.gc.live.data.size | jvm.gc.live.data.size | Bytes | Tamaño del grupo de memoria de generación anterior después de una recolección de elementos no utilizados completa. |
>| jvm.gc.memory.promoted | jvm.gc.memory.promoted | Bytes | Recuento de aumentos positivos en el tamaño del grupo de memoria de generación anterior entre antes y después de una recolección de elementos no utilizados. |
>| jvm.gc.memory.allocated | jvm.gc.memory.allocated | Bytes | Se incrementa por un aumento del tamaño del grupo de memoria de nueva generación después de una recolección de elementos no utilizados y antes de la siguiente. |
>| jvm.gc.pause.total.count | jvm.gc.pause (recuento total) | Count | Recuento total de recolecciones de elementos no utilizados después del inicio de esta Máquina virtual Java, incluidas las recolecciones de elementos no utilizados anteriores y nuevas. |
>| jvm.gc.pause.total.time | jvm.gc.pause (tiempo total) | Milisegundos | Recuento total de tiempo consumido en recolecciones de elementos no utilizados después del inicio de esta Máquina virtual Java, incluidas las recolecciones de elementos no utilizados anteriores y nuevas. |

### <a name="request"></a>Solicitud
>[!div class="mx-tdCol2BreakAll"]
>| Nombre | Nombre de la métrica de Spring Actuator | Unidad | Detalles |
>|----|----|----|------------|
>| tomcat.global.sent | tomcat.global.sent | Bytes | Cantidad de datos enviados por el servidor web de Tomcat |
>| tomcat.global.received | tomcat.global.received | Bytes | Cantidad de datos recibidos por el servidor web de Tomcat |
>| tomcat.global.request.total.count | tomcat.global.request (recuento total) | Count | Recuento total de solicitudes procesadas por el servidor web de Tomcat |
>| tomcat.global.request.max | tomcat.global.request.max | Milisegundos | Tiempo máximo del servidor web de Tomcat para procesar una solicitud |

### <a name="session"></a>Sesión
>[!div class="mx-tdCol2BreakAll"]
>| Nombre | Nombre de la métrica de Spring Actuator | Unidad | Detalles |
>|----|----|----|------------|
>| tomcat.sessions.active.max | tomcat.sessions.active.max | Count | Número máximo de sesiones que han estado activas al mismo tiempo |
>| tomcat.sessions.alive.max | tomcat.sessions.alive.max | Milisegundos | Período más largo (en segundos) que una sesión expirada ha estado activa |
>| tomcat.sessions.created | tomcat.sessions.created | Count | Número de sesiones que se han creado |
>| tomcat.sessions.expired | tomcat.sessions.expired | Count | Número de sesiones que han expirado |
>| tomcat.sessions.rejected | tomcat.sessions.rejected | Count | Número de sesiones que no se crearon porque se alcanzó el número máximo de sesiones activas. |
>| tomcat.sessions.active.current | tomcat.sessions.active.current | Count | Recuento de sesiones activas de Tomcat |

## <a name="see-also"></a>Consulte también
* [Introducción al Explorador de métricas de Azure](https://docs.microsoft.com/azure/azure-monitor/platform/metrics-getting-started)

* [Análisis de registros y métricas con la configuración de diagnóstico](https://docs.microsoft.com/azure/spring-cloud/diagnostic-services)

## <a name="next-steps"></a>Pasos siguientes
* [Tutorial: Supervisión de recursos de Spring Cloud mediante alertas y grupos de acciones](https://docs.microsoft.com/azure/spring-cloud/spring-cloud-tutorial-alerts-action-groups)

* [Cuotas y planes de servicio de Azure Spring Cloud](https://docs.microsoft.com/azure/spring-cloud/spring-cloud-quotas)

