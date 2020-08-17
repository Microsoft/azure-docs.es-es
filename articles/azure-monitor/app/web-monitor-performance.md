---
title: Supervisar el estado y el uso de su aplicación con Application Insights
description: Introducción a Application Insights. Analice el uso, la disponibilidad y el rendimiento de sus aplicaciones web de Microsoft Azure o local.
ms.topic: conceptual
ms.date: 05/10/2018
ms.reviewer: sdash
ms.openlocfilehash: d624286d214a86364fe85192bf5ede885d4b6a78
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323458"
---
# <a name="monitor-performance-in-web-applications"></a>Supervisar el rendimiento de aplicaciones web


Asegúrese de que la aplicación tendrá un rendimiento correcto y descubra rápidamente los posibles errores. [Application Insights][start] le indicará las excepciones y los problemas de rendimiento, y lo ayudará a buscar y diagnosticar las causas principales.

Application Insights puede supervisar aplicaciones y servicios web Java y ASP.NET y servicios WCF. Pueden estar hospedados localmente, en máquinas virtuales o como sitios web de Microsoft Azure. 

En el lado del cliente, Application Insights puede obtener datos de telemetría de páginas web y una amplia variedad de dispositivos, entre los que se incluyen iOS, Android y aplicaciones de la Tienda Windows.

## <a name="set-up-performance-monitoring"></a><a name="setup"></a>Configuración de la supervisión de rendimiento
Si todavía no ha agregado Application Insights a un proyecto (es decir, si no dispone de ApplicationInsights.config), puede comenzar con uno de estos procedimientos:

* [Aplicaciones web ASP.NET](./asp-net.md)
  * [Agregar supervisión de excepciones](./asp-net-exceptions.md)
  * [Agregar supervisión de dependencias](./monitor-performance-live-website-now.md)
* [Aplicaciones web de Java EE](./java-in-process-agent.md)

## <a name="exploring-performance-metrics"></a><a name="view"></a>Exploración de las métricas de rendimiento
En el [Portal de Azure](https://portal.azure.com), busque el recurso de Application Insights que configuró para la aplicación. La hoja de información general muestra los datos de rendimiento básicos:

Haga clic en cualquier gráfico para ver su contenido con mayor detalle y los resultados durante más tiempo. Por ejemplo, haga clic en el mosaico Solicitudes y seleccione un intervalo de tiempo:

![Haga clic en las distintas opciones para obtener más datos y seleccionar un intervalo de tiempo](./media/web-monitor-performance/appinsights-48metrics.png)

Haga clic en un gráfico para elegir las métricas mostradas o para agregar un nuevo gráfico y seleccionar sus métricas:

![Haga clic en un gráfico para elegir las métricas](./media/web-monitor-performance/appinsights-61perfchoices.png)

> [!NOTE]
> **Desactive todas las métricas** para ver todas las métricas disponibles. Las métricas se organizan en grupos. Cuando se selecciona un miembro de un grupo, solo aparecen los demás miembros de dicho grupo.

## <a name="what-does-it-all-mean-performance-tiles-and-reports"></a><a name="metrics"></a>¿Qué significa todo esto? Informes y mosaicos de rendimiento
Puede obtener una gran variedad de métricas de rendimiento. Comencemos por las que aparecen de forma predeterminada en la hoja de la aplicación.

### <a name="requests"></a>Requests
El número de solicitudes HTTP que se reciben en un período específico. Compare este dato con los resultados de otros informes para ver cómo se comporta la aplicación cuando la carga varía.

Las solicitudes HTTP incluyen todas las solicitudes GET o POST de páginas, datos e imágenes.

Haga clic en el mosaico para obtener las cuentas de URL específicas.

### <a name="average-response-time"></a>Tiempo medio de respuesta
Mide el tiempo entre la entrada de una solicitud web y la devolución de la respuesta correspondiente.

Los puntos muestran una media móvil. Si hay muchas solicitudes, es posible que algunas de ellas se desvíen de la media sin que en el gráfico se muestre ningún pico o descenso evidente.

Busque picos poco habituales. Por lo general, el tiempo de respuesta aumenta cuando aumentan las solicitudes. Si este aumento es desproporcionado, la aplicación puede alcanzar el límite de recursos (por ejemplo, de CPU o de un servicio que esté usando).

Haga clic en el icono para obtener los tiempos de direcciones URL concretas.

![Captura de pantalla del panel estado de la aplicación, que muestra los gráficos de líneas a lo largo del tiempo del promedio de transiciones de solicitudes y del tiempo de respuesta.](./media/web-monitor-performance/appinsights-42reqs.png)

### <a name="slowest-requests"></a>Solicitudes más lentas
![Captura de pantalla de una lista de las solicitudes más lentas y los tiempos de respuesta.](./media/web-monitor-performance/appinsights-44slowest.png)

Muestra las solicitudes que pueden precisar un ajuste de rendimiento.

### <a name="failed-requests"></a>Error en las solicitudes
![Captura de pantalla de un gráfico de líneas a lo largo del tiempo del número de solicitudes con error; es decir, las solicitudes que han producido excepciones no detectadas.](./media/web-monitor-performance/appinsights-46failed.png)

Un recuento de las solicitudes que inician excepciones no detectadas.

Haga clic en el mosaico para ver los detalles de errores específicos y seleccione una solicitud individual para revisarla de forma más exhaustiva. 

Solo se conserva una muestra representativa de los errores para su inspección individual.

### <a name="other-metrics"></a>Otras métricas
Si desea ver las demás métricas que puede mostrar, haga clic en un gráfico y desactive todas las métricas para ver el conjunto completo de opciones disponibles. Haga clic en (i) para ver la definición de cada métrica.

![Cancele todas las métricas para ver el conjunto completo](./media/web-monitor-performance/appinsights-62allchoices.png)

Si selecciona una métrica, se deshabilitan las demás, por lo que no pueden aparecer en el mismo gráfico.

## <a name="set-alerts"></a>Establecer alertas
Para recibir notificaciones por correo electrónico de los valores no habituales de cualquier métrica, agregue una alerta. Puede decidir si se debe enviar un mensaje de correo electrónico a los administradores de cuentas o a direcciones de correo electrónico específicas.

![Captura de pantalla del cuadro de diálogo Agregar una regla de alerta, junto con capturas de pantalla conectadas mediante flechas que muestran cómo acceder a dicho cuadro de diálogo desde el Explorador de métricas.](./media/web-monitor-performance/appinsights-413setMetricAlert.png)

Establezca el recurso antes de las demás propiedades. No elija los recursos de la prueba web si desea establecer alertas en las métricas de rendimiento o de uso.

Asegúrese de tener en cuenta las unidades en las que se le pide que escriba el valor de umbral.

*No puedo ver el botón Agregar alerta* : ¿es una cuenta de grupo a la que tiene acceso de solo lectura? Consulte con el administrador de cuenta.

## <a name="diagnosing-issues"></a><a name="diagnosis"></a>Diagnóstico de problemas
Para buscar y diagnosticar problemas de rendimiento, lea estas sugerencias:

* Configure las [pruebas web][availability] para recibir una alerta si el sitio web deja de funcionar o si no responde correctamente, o lo hace más lentamente de lo habitual. 
* Compare el recuento de Solicitudes con otras métricas para ver si la lentitud de respuesta o los errores se encuentran relacionados con la carga.
* [Inserte y busque instrucciones de seguimiento][diagnostic] en el código para facilitar la identificación de los problemas.
* Supervise el funcionamiento de la aplicación web con [Live Metrics Stream][livestream].
* Capture el estado de una aplicación .NET con [Snapshot Debugger][snapshot].

## <a name="find-and-fix-performance-bottlenecks-with-performance-investigation-experience"></a>Búsqueda y corrección de cuellos de botella en el rendimiento con la experiencia de investigación interactiva del rendimiento

Puede usar la experiencia de investigación del rendimiento para revisar las operaciones de ejecución lenta en su aplicación web. Puede seleccionar una operación específica lenta rápidamente y usar [Profiler](./profiler.md) para encontrar la causa del funcionamiento lento en el código. Use la distribución de la nueva duración que se muestra para la operación seleccionada para evaluar rápidamente cómo perciben la experiencia los clientes. Puede ver cuántas de sus interacciones de usuario se vieron afectadas por cada operación lenta. En el siguiente ejemplo, hemos decidido echar un vistazo más de cerca a la experiencia de la operación GET Customers/Details. En la distribución de duración podemos ver que hay tres picos. El pico del extremo izquierdo es de, aproximadamente, 400 ms y representa una experiencia con una gran capacidad de respuesta. El pico intermedio es de, aproximadamente, 1,2 s y representa una experiencia mediocre. Finalmente, en 3,6 s tenemos un pequeño aumento que representa la experiencia de percentil 99, lo que podría hacer que nuestros clientes se fueran insatisfechos. Esta experiencia es diez veces más lenta que la experiencia excelente de la misma operación. 

![Tres picos de duración de GET Customers/Details](./media/web-monitor-performance/PerformanceTriageViewZoomedDistribution.png)

Para obtener una mejor idea de las experiencias de usuario para esta operación, podemos seleccionar un intervalo de tiempo mayor. A continuación, podemos reducir el tiempo a un intervalo concreto en que la operación fue lenta. En el siguiente ejemplo hemos cambiado el valor predeterminado de 24 horas por el de 7 días y hemos ampliado el periodo entre 9:47 y 12:47 del martes 12 y el miércoles 13. La distribución de la duración y el número de seguimientos de Profiler y de ejemplo se han actualizado a la derecha.

![Tres picos de duración para GET Customers/Details en un intervalo de 7 días en un periodo de tiempo](./media/web-monitor-performance/PerformanceTriageView7DaysZoomedTrend.png)

Para limitar las experiencias lentas, a continuación, ampliamos las duraciones situadas en el percentil entre 95 y 99. Estas representan el 4 % de las interacciones del usuario que eran lentas.

![Tres picos de duración para GET Customers/Details en un intervalo de 7 días en un periodo de tiempo](./media/web-monitor-performance/PerformanceTriageView7DaysZoomedTrendZoomed95th99th.png)

Ahora podemos observar los ejemplos representativos haciendo clic en el botón Ejemplos o los seguimientos de Profiler haciendo clic en el botón Seguimientos de Profiler. En este ejemplo hay cuatro seguimientos que se han recopilado para obtener clientes y detalles en el periodo de tiempo y el intervalo de duración de interés.

A veces, el problema no estará en el código, sino en una dependencia que invoca el código. Puede cambiar a la pestaña Dependencias de la vista de evaluación de prioridades para investigar tales dependencias lentas. De forma predeterminada, la vista de rendimiento muestra medias de tendencia, pero el dato relevante es el del percentil 95 (o 99, en caso de que esté supervisando un servicio consolidado). En el siguiente ejemplo, nos hemos centrado en la dependencia de blob de Azure lenta, donde se invocó PUT fabrikamaccount. Las buenas experiencias se agrupan alrededor de los 40 ms, mientras que las llamadas lentas a la misma dependencia son tres veces más lentas, aproximadamente 120 ms. No se necesitan muchas llamadas para que la operación respectiva se ralentice considerablemente. Puede profundizar en los ejemplos representativos y seguimientos de Profiler, exactamente igual que con la pestaña operaciones.

![Tres picos de duración para GET Customers/Details en un intervalo de 7 días en un periodo de tiempo](./media/web-monitor-performance/SlowDependencies95thTrend.png)

La experiencia de investigación de rendimiento muestra conclusiones significativas a la largo del conjunto de muestra en el que ha decido centrarse. La mejor manera de buscar en todas las informaciones disponibles es cambiar a un intervalo de tiempo de 30 días y, a continuación, seleccionar Información general para ver información de todas las operaciones del mes pasado.

![Tres picos de duración para GET Customers/Details en un intervalo de 7 días en un periodo de tiempo](./media/web-monitor-performance/Performance30DayOveralllnsights.png)


## <a name="next-steps"></a><a name="next"></a>Pasos siguientes
[Pruebas web][availability]: reciba solicitudes web en su aplicación de forma periódica desde distintos lugares del mundo.

[Capture y busque seguimientos de diagnóstico][diagnostic]: inserte llamadas de seguimiento y cribe los resultados para identificar los problemas.

[Seguimiento del uso][usage]: averigüe cuántas personas usan la aplicación.

[Solución de problemas][qna]: con preguntas y respuestas



<!--Link references-->

[availability]: ./monitor-web-app-availability.md
[diagnostic]: ./diagnostic-search.md
[greenbrown]: ./asp-net.md
[qna]: ../faq.md
[redfield]: ./monitor-performance-live-website-now.md
[start]: ./app-insights-overview.md
[usage]: usage-overview.md
[livestream]: ./live-stream.md
[snapshot]: ./snapshot-debugger.md

