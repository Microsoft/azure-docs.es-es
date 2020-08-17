---
title: ¿Qué es Azure Application Insights? | Microsoft Docs
description: Application Performance Management y seguimiento del uso de la aplicación web en directo.  Detecte, evalúe y diagnostique problemas, y comprenda cómo utilizan las personas la aplicación.
ms.topic: overview
ms.date: 06/03/2019
ms.custom: mvc
ms.openlocfilehash: f80322e8cde27ea48ca343ca8eee5985cba23ceb
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87321452"
---
# <a name="what-is-application-insights"></a>¿Qué es Application Insights?
Application Insights es una característica de [Azure Monitor](../overview.md) que es un servicio de Application Performance Management (APM) extensible para desarrolladores y profesionales de DevOps. Úselo para supervisar las aplicaciones en directo. Detectará automáticamente anomalías en el rendimiento e incluye eficaces herramientas de análisis que le ayudan a diagnosticar problemas y a saber lo que hacen realmente los usuarios con la aplicación.  Está diseñado para ayudarle a mejorar continuamente el rendimiento y la facilidad de uso. Funciona con diversas aplicaciones y en una amplia variedad de plataformas, como .NET, Node.js, Java y Python, hospedadas en el entorno local, de forma híbrida o en cualquier nube pública. Se integra con el proceso de DevOps y tiene puntos de conexión a numerosas herramientas de desarrollo. Puede supervisar y analizar la telemetría de aplicaciones móviles mediante la integración con Visual Studio App Center.

## <a name="how-does-application-insights-work"></a>¿Cómo funciona Application Insights?
Instale un paquete de instrumentación pequeño (SDK) en la aplicación o habilite Application Insights mediante el agente de Application Insights cuando se [admita](./platforms.md). La instrumentación supervisa la aplicación y dirige los datos de telemetría a un recurso de Azure Application Insights mediante un GUID único al que se hace referencia como una clave de instrumentación.

No solo puede instrumentar la aplicación de servicio web, sino también todos los componentes en segundo plano y JavaScript en las propias páginas web. La aplicación y sus componentes se pueden ejecutar en cualquier lugar; no tienen que estar hospedados en Azure.

![La instrumentación de Application Insights de la aplicación envía datos de telemetría al recurso de Application Insights.](./media/app-insights-overview/diagram.png)

Además, puede obtener la telemetría de los entornos del host, como pueden ser contadores de rendimiento, diagnósticos de Azure o registros de Docker. También puede configurar pruebas web que envíen periódicamente solicitudes sintéticas al servicio web.

Todos estos flujos de telemetría están integrados en Azure Monitor. En Azure Portal, puede aplicar versátiles herramientas de análisis y búsqueda a los datos sin procesar.

### <a name="whats-the-overhead"></a>¿Cuál es la sobrecarga?
El impacto sobre el rendimiento de la aplicación es pequeño. Las llamadas de seguimiento no suponen ningún tipo de bloqueo y se agrupan por lotes y se envían en un subproceso aparte.

## <a name="what-does-application-insights-monitor"></a>¿Qué supervisa Application Insights?

Application Insights está dirigido al equipo de desarrollo y sirve ayudarle a conocer el rendimiento de una aplicación y cómo se utiliza. Supervisa:

* **Tasas de solicitud, tiempos de respuesta y tasas de error** - Averigüe qué páginas que son las más conocidas, en qué momento del día y dónde están los usuarios. Vea qué páginas presentan mejor rendimiento. Si los tiempos de respuesta y las tasas de error aumentan cuando hay más solicitudes, quizás tiene un problema de recursos. 
* **Tasas de dependencia, tiempos de respuesta y tasa de error** - Averigüe si los servicios externos le ralentizan.
* **Excepciones**: - Analice las estadísticas agregadas o seleccione instancias concretas y profundice en el seguimiento de la pila y las solicitudes relacionadas. Se notifican tanto las excepciones de servidor como las de explorador.
* **Vistas de página y rendimiento de carga** - Notificados por los exploradores de los usuarios.
* **Llamadas AJAX** desde páginas web - Tasas, tiempos de respuesta y tasas de error.
* **Número de usuarios y sesiones**.
* **Contadores de rendimiento** de las máquinas de servidor de Windows o Linux, como CPU, memoria y uso de la red. 
* **Diagnóstico de host** de Docker o Azure. 
* **Registros de seguimiento de diagnóstico** de la aplicación - De esta forma puede correlacionar eventos de seguimiento con las solicitudes.
* **Métricas y eventos personalizados** que usted mismo escribe en el código de cliente o servidor para realizar un seguimiento de eventos empresariales, como artículos vendidos o partidas ganadas.

## <a name="where-do-i-see-my-telemetry"></a>¿Dónde veo la telemetría?

Hay muchas formas de explorar los datos. Consulte estos artículos:

|  |  |
| --- | --- |
| [**Detección inteligente y alertas manuales**](./proactive-diagnostics.md)<br/>Las alertas automáticas configuradas se adaptan a los patrones normales de telemetría de la aplicación y se desencadenan cuando algo no responde al patrón habitual. También puede [establecer alertas](../platform/alerts-log.md) sobre niveles concretos de métricas estándares o personalizadas. |![Ejemplo de alerta](./media/app-insights-overview/alerts-tn.png) |
| [**Mapa de aplicación**](./app-map.md)<br/>Explore los componentes de la aplicación, con alertas y métricas clave. |![Mapa de aplicación](./media/app-insights-overview/appmap-tn.png)  |
| [**Generador de perfiles**](./profiler.md)<br/>Inspeccione los perfiles de ejecución de solicitudes muestreadas. |![Generador de perfiles](./media/app-insights-overview/profiler.png) |
| [**Análisis de uso**](./usage-overview.md)<br/>Analice la segmentación y la retención de usuarios.|![Herramienta Retención](./media/app-insights-overview/retention.png) |
| [**Búsqueda de diagnóstico para datos de instancia**](./diagnostic-search.md)<br/>Busque y filtre eventos como solicitudes, excepciones, llamadas de dependencia, seguimientos de registro y vistas de páginas.  |![Buscar telemetría](./media/app-insights-overview/search-tn.png) |
| [**Explorador de métricas para datos agregados**](../platform/metrics-charts.md)<br/>Explore, filtre y segmente datos agregados, como los índices de solicitudes, errores y excepciones; los tiempos de respuesta y los tiempos de carga de página. |![Métricas](./media/app-insights-overview/metrics-tn.png) |
| [**Paneles**](./overview-dashboard.md)<br/>Combine datos de varios recursos y compártalos con otros. Ideal para aplicaciones de varios componentes y para la presentación continua en la sala de reuniones. |![Ejemplo de paneles](./media/app-insights-overview/dashboard-tn.png) |
| [**Secuencia de métricas en directo**](./live-stream.md)<br/>Al implementar una nueva compilación, fíjese en estos indicadores de rendimiento casi en tiempo real para asegurarse de que todo funciona según lo esperado. |![Ejemplo de métricas en directo](./media/app-insights-overview/live-metrics-tn.png) |
| [**Análisis**](../log-query/log-query-overview.md)<br/>Responda preguntas complejas acerca del uso y el rendimiento de su aplicación mediante este eficaz lenguaje de consulta. |![Ejemplo de análisis](./media/app-insights-overview/analytics-tn.png) |
| [**Visual Studio**](./visual-studio.md)<br/>Vea los datos de rendimiento en el código. Vaya al código desde los seguimientos de la pila.|![Visual Studio](./media/app-insights-overview/visual-studio-tn.png) |
| [**Depurador de instantáneas**](./snapshot-debugger.md)<br/>Depure instantáneas muestreadas desde operaciones en directo, con valores de parámetro.|![Visual Studio](./media/app-insights-overview/snapshot.png) |
| [**Power BI**](./export-power-bi.md)<br/>Integre métricas de uso con otra inteligencia empresarial.| ![Power BI](./media/app-insights-overview/power-bi.png)|
| [**API DE REST**](https://dev.applicationinsights.io/)<br/>Escriba código para ejecutar consultas sobre las métricas y los datos sin procesar.| ![API DE REST](./media/app-insights-overview/rest-tn.png) |
| [**Exportación continua**](./export-telemetry.md)<br/>Exportación masiva de datos sin procesar al almacenamiento tan pronto como llegan. |![Exportación](./media/app-insights-overview/export-tn.png) |

## <a name="how-do-i-use-application-insights"></a>¿Cómo uso Application Insights?

### <a name="monitor"></a>Supervisión
Instale Application Insights en la aplicación web, configure las [pruebas web de disponibilidad](./monitor-web-app-availability.md) y:

* Revise el [panel de aplicación](./overview-dashboard.md) predeterminado para que su equipo no pierda de vista la carga, la capacidad de respuesta y el rendimiento de las dependencias, las cargas de páginas y las llamadas AJAX.
* Detecte cuáles son las solicitudes más lentas y con mayor número de errores.
* Vea [Live Stream](./live-stream.md) cuando implemente una versión nueva, con el fin de conocer inmediatamente la existencia de cualquier degradación.

### <a name="detect-diagnose"></a>Detección y diagnóstico
Cuando reciba una alerta o detecte un problema:

* Evalúe cuántos usuarios se ven afectados.
* Correlacione los errores con las excepciones, las llamadas de dependencia y los seguimientos.
* Examine el generador de perfiles, las instantáneas, los volcados de pila y los registros de seguimiento.

### <a name="build-measure-learn"></a>Compilación, medición y aprendizaje
[Mida la eficacia](./usage-overview.md) de cada característica nueva que implemente.

* Planee la medición de la forma en que los clientes utilizan las nuevas características empresariales y de experiencia de usuario.
* Escriba datos de telemetría personalizados en el código.
* Base el siguiente ciclo de desarrollo en pruebas contundentes de la telemetría.

## <a name="get-started"></a>Introducción
Application Insights es uno de los muchos servicios hospedados en Microsoft Azure y los datos de telemetría se envían ahí para analizarlos y mostrarlos. Por tanto, antes de nada, se necesita una suscripción a [Microsoft Azure](https://azure.com). El registro es gratuito y, si elige el [plan de precios](https://azure.microsoft.com/pricing/details/application-insights/) básico de Application Insights, no habrá cargo alguno hasta que la aplicación tenga un uso considerable. Si la organización ya tiene una suscripción, puede agregarle su cuenta de Microsoft.

Hay varias formas de empezar. Comience con la que más se ajuste a sus necesidades. Puede agregar los demás posteriormente.

* **En tiempo de ejecución: instrumente su aplicación web en el servidor.** Ideal para las aplicaciones ya implementadas. Evita toda actualización del código.
  * [**Aplicaciones ASP.NET o ASP.NET Core hospedadas en Azure Web Apps**](./azure-web-apps.md)
  * [**Aplicaciones ASP.NET hospedadas en IIS en una máquina virtual de Azure o un conjunto de escalado de máquinas virtuales de Azure**](./azure-vm-vmss-apps.md)
  * [**Aplicaciones ASP.NET hospedadas en una máquina virtual en el entorno local de IIS**](./monitor-performance-live-website-now.md)
* **En tiempo de desarrollo: agregue Application Insights al código.** Le permite personalizar la recopilación de telemetría personalizada y enviar telemetría adicional.
  * [Aplicaciones ASP.NET](./asp-net.md)
  * [Aplicación ASP.NET Core](./asp-net-core.md)
  * [Aplicaciones de consola .NET](./console.md)
  * [Java](./java-get-started.md)
  * [Node.js](./nodejs.md)
  * [Python](./opencensus-python.md)
  * [Otras plataformas](./platforms.md)
* **[Instrumente sus páginas web](./javascript.md)** para la vista de la página, AJAX y otros datos de telemetría del lado cliente.
* **[Analice el uso de aplicaciones móviles](../learn/mobile-center-quickstart.md)** mediante la integración con Visual Studio App Center.
* **[Pruebas de disponibilidad](./monitor-web-app-availability.md)** : haga ping a su sitio web de manera regular desde nuestros servidores.

## <a name="next-steps"></a>Pasos siguientes
Comience en el tiempo de ejecución con:

* [Aplicaciones hospedadas en IIS en máquina virtual de Azure y conjunto de escalado de máquinas virtuales de Azure](./azure-vm-vmss-apps.md)
* [Servidor IIS](./monitor-performance-live-website-now.md)
* [Azure Web Apps](./azure-web-apps.md)

Comience en el tiempo de desarrollo con:

* [ASP.NET](./asp-net.md)
* [ASP.NET Core](./asp-net-core.md)
* [Java](./java-get-started.md)
* [Node.js](./nodejs.md)
* [Python](./opencensus-python.md)


## <a name="support-and-feedback"></a>Soporte y comentarios
* Preguntas y problemas:
  * [Solución de problemas][qna]
  * [Página de preguntas y respuestas de Microsoft](/answers/topics/azure-monitor.html)
  * [Stackoverflow](https://stackoverflow.com/questions/tagged/ms-application-insights)
* Sus sugerencias:
  * [UserVoice](https://feedback.azure.com/forums/357324-application-insights/filters/top)
* Blog:
  * [Blog de Application Insights](https://azure.microsoft.com/blog/tag/application-insights)

<!--Link references-->

[android]: ../learn/mobile-center-quickstart.md
[azure]: ../../insights-perf-analytics.md
[client]: ./javascript.md
[desktop]: ./windows-desktop.md
[greenbrown]: ./asp-net.md
[ios]: ../learn/mobile-center-quickstart.md
[java]: ./java-get-started.md
[knowUsers]: app-insights-web-track-usage.md
[platforms]: ./platforms.md
[portal]: https://portal.azure.com/
[qna]: ../faq.md
[redfield]: ./monitor-performance-live-website-now.md

