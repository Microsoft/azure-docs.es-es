---
title: 'Supervisión del rendimiento de aplicaciones web: Azure Application Insights'
description: Cómo encaja Application Insights en el ciclo de devOps
ms.topic: conceptual
ms.date: 12/21/2018
ms.openlocfilehash: d2c3326c3f655d4bdfeaa42ac272658b251b7f82
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87324495"
---
# <a name="deep-diagnostics-for-web-apps-and-services-with-application-insights"></a>Diagnósticos detallados para servicios y aplicaciones web con Application Insights
## <a name="why-do-i-need-application-insights"></a>¿Por qué necesito Application Insights?
Application Insights supervisa la ejecución de la aplicación web. Indica los errores y problemas de rendimiento y le ayuda a analizar el modo en que los clientes usan las aplicaciones. Sirve para aplicaciones que se ejecutan en muchas plataformas (ASP.NET, Java EE, Node.js, etc.) y se hospeda en la nube o en el entorno local. 

![Aspectos de la complejidad del suministro de aplicaciones web](./media/devops/010.png)

Resulta fundamental para supervisar una aplicación moderna mientras se ejecuta. Y lo más importante, puede detectar los errores antes que la mayoría de sus clientes. También puede detectar y resolver problemas de rendimiento que, si bien no son catastróficos, quizás ralenticen la velocidad del sistema u ocasionen algún inconveniente a los usuarios. Y si el sistema rinde como espera, querrá saber lo que hacen con él los usuarios: ¿usan las características más recientes? ¿Les funcionan bien?

Las aplicaciones web modernas se desarrollan en un ciclo de entrega continua: lanzan una nueva característica o mejora, observan cómo funciona para los usuarios y, en función de esa información, planean el siguiente incremento de desarrollo. Una parte fundamental de este ciclo es la fase de observación. Application Insights proporciona las herramientas para supervisar el rendimiento y uso de una aplicación web.

El aspecto más importante de este proceso es el diagnóstico. Si una aplicación no funciona correctamente, el negocio se pierde. Por lo tanto, el papel fundamental de un marco de supervisión es detectar los errores de manera confiable, notificarlos inmediatamente y presentar la información necesaria para diagnosticar el problema. Esto es exactamente lo que hace Application Insights.

### <a name="where-do-bugs-come-from"></a>¿De dónde proceden los errores?
Los errores de los sistemas web proceden normalmente de problemas de configuración o de interacciones incorrectas entre sus muchos componentes. La primera tarea al abordar un incidente en un sitio activo es, por lo tanto, identificar la raíz del problema: ¿qué componente o relación es la causa?

Algunos de nosotros, los más veteranos, podemos recordar una época más sencilla en la que un programa informático se ejecutaba en un equipo. Los desarrolladores lo probaban a fondo antes de enviarlo; y, una vez enviado, era muy raro volver verlo o acordarse de él. Los usuarios tenían que soportar los errores residuales durante muchos años. 

Ahora, las cosas son muy diferentes. La aplicación se ejecuta en una enorme cantidad de dispositivos diferentes y puede ser difícil garantizar el mismo comportamiento exacto en cada uno. El hospedaje de aplicaciones en el nube significa que los errores se puede resolver más rápido, pero también una continua rivalidad y la expectativa de nuevas características a intervalos frecuentes. 

En estas condiciones, la única manera de mantener un control estricto sobre el número de errores es la prueba unitaria automatizada. Sería imposible volver a probarlo todo manualmente en cada entrega. La prueba unitaria es ahora una parte común del proceso de compilación. Herramientas como la ayuda de Xamarin Test Cloud ayudan al proporcionar pruebas automatizadas de la interfaz de usuario en varias versiones de explorador. Estos regímenes de pruebas nos permiten tener la esperanza de que la tasa de errores encontrados en una aplicación se pueda mantener al mínimo.

Las aplicaciones web típicas tienen muchos componentes activos. Además del cliente (en una aplicación de explorador o dispositivo) y el servidor web, es probable que haya bastante procesamiento back-end. Quizás el back-end es una canalización de componentes o una colección de piezas de colaboración más flexible. Y muchos de ellos no estarán bajo su control, y son servicios externos de los que depende.

En este tipo de configuraciones, puede ser difícil y costoso probar, o predecir, cada posible modo de error, que no sea el propio sistema activo. 

### <a name="questions-"></a>Preguntas...
Algunas preguntas que hacemos cuando desarrollamos un sistema web:

* ¿Se bloqueará mi aplicación? 
* ¿Qué ha sucedido exactamente? Si una solicitud ha dado error, quiero saber cómo hemos llegado hasta aquí. Necesitamos un seguimiento de eventos...
* ¿Es mi aplicación lo suficientemente rápida? ¿Cuánto tarda en responder a solicitudes normales?
* ¿Puede administrar el servidor la carga? Cuando aumenta la tasa de solicitudes, ¿se mantiene estable el tiempo de respuesta?
* ¿Cuál es la capacidad de respuesta de mis dependencias? (las API de REST, las bases de datos y otros componentes a los que llama mi aplicación). En concreto, si el sistema es lento, ¿se debe a alguno de mis componentes o a otra cosa?
* ¿Está mi aplicación activa o inactiva? ¿se puede ver desde cualquier parte del mundo? Quiero saber si se detiene....
* ¿Cuál es la causa? ¿Se debe el error a un componente o una dependencia? ¿Es un problema de comunicación?
* ¿Cuántos usuarios están afectados? Si tiene varios problemas a los que hacer frente, ¿cuál es el más importante?

## <a name="what-is-application-insights"></a>¿Qué es Application Insights?
![Flujo de trabajo básico de Application Insights](./media/devops/020.png)

1. Application Insights instrumenta la aplicación y envía datos de telemetría sobre ella mientras esta se ejecuta. Puede general el SDK de Application Insights en la aplicación o aplicar la instrumentación en tiempo de ejecución. El primer método es más flexible, ya que puede agregar sus propios datos de telemetría a los módulos normales.
2. Estos datos se envían al portal de Application Insights, donde se almacenan y procesan. (Aunque Application Insights se hospeda en Microsoft Azure, puede supervisar cualquier aplicación web, no solo las de Azure).
3. La información de telemetría se presenta en forma de gráficos y tablas de eventos.

Hay dos tipos principales de telemetría: instancias agregadas y sin formato. 

* Por ejemplo, los datos de instancia incluyen un informe de una solicitud que ha recibido la aplicación web. Se pueden buscar e inspeccionar los detalles de una solicitud mediante la herramienta de búsqueda del portal de Application Insights. La instancia incluiría datos, como el tiempo que la aplicación ha tardado en responder a la solicitud, la dirección URL solicitada, la ubicación aproximada del cliente y otros datos.
* Los datos agregados incluyen recuentos de eventos por unidad de tiempo, de modo que puede comparar la tasa de solicitudes con los tiempos de respuesta. También incluyen los promedios de las métricas, como los tiempos de respuesta de solicitud.

Las principales categorías de datos son:

* Solicitudes a la aplicación (normalmente solicitudes HTTP), con datos sobre dirección URL, tiempo de respuesta y éxito o error.
* Dependencias: llamadas de REST y SQL realizadas por la aplicación, también con identificador URI, tiempos de respuesta y éxito.
* Excepciones, que incluyen seguimientos de pila.
* Datos de vista de página, que proceden de los exploradores de los usuarios.
* Métricas, como contadores de rendimiento, y métricas escritas por usted mismo. 
* Eventos personalizados que puede utilizar para realizar el seguimiento de eventos empresariales.
* Seguimientos de registros usados para la depuración.

## <a name="case-study-real-madrid-fc"></a>Caso práctico: Real Madrid F.C.
El servicio web del [Club de fútbol del Real Madrid](https://www.realmadrid.com/) presta servicio a aproximadamente 450 millones de aficionados de todo el mundo. Los aficionados acceden a él tanto mediante exploradores web como las aplicaciones móviles del Club. No solo reservan entradas, sino que también acceden a información y a clips de vídeo sobre resultados, jugadores y próximos partidos. Pueden realizar búsquedas con filtros, como el número de goles marcados. También hay vínculos a redes sociales. La experiencia del usuario está enormemente personalizada y está diseñada como una comunicación bidireccional para favorecer la participación de los aficionados.

La solución [es un sistema de servicios y aplicaciones en Microsoft Azure](https://www.microsoft.com/inculture/sports/real-madrid/). La escalabilidad es un requisito clave: el tráfico es variable y puede alcanzar volúmenes elevados antes y durante los partidos.

Para el Real Madrid, es fundamental supervisar el rendimiento del sistema. Azure Application Insights ofrece una vista completa del sistema, lo que garantiza un nivel de servicio alto y confiable. 

El Club también obtiene un profundo conocimiento de sus aficionados: dónde están (solo el 3 % está en España), qué interés tienen en los jugadores, los resultados históricos y los próximos partidos y cómo responden a los resultados de los partidos.

La mayoría de estos datos de telemetría se recopilan automáticamente sin ningún código adicional, lo que simplifica la solución y reduce la complejidad operacional.  Para el Real Madrid, Application Insights trata con 3800 millones de puntos de datos de telemetría al mes.

El Real Madrid utiliza el módulo Power BI para ver estos datos.

![Vistas de telemetría de Application Insights en Power BI](./media/devops/080.png)

## <a name="smart-detection"></a>Detección inteligente
[Proactive diagnostics](./proactive-diagnostics.md) es una característica reciente. Sin ninguna configuración especial por parte del usuario, Application Insights detecta y le avisa automáticamente de cualquier aumento en la tasa de errores en la aplicación. Es lo suficientemente inteligente como para omitir un trasfondo de errores ocasionales y también de aumentos que son simplemente proporcionales a un aumento en las solicitudes. Por ejemplo, si se produce un error en uno de los servicios de los que depende, o si la nueva compilación que acaba de implementar no funciona tan bien, lo sabrá en cuanto mire su correo electrónico. (Y hay webhooks para que pueda desencadenar otras aplicaciones).

Otro aspecto de esta característica es que realiza un análisis exhaustivo a diario de los datos de telemetría, buscando patrones de rendimiento poco habituales difíciles de detectar. Por ejemplo, puede encontrar un rendimiento lento asociado a una zona geográfica determinada, o a una versión de explorador determinada.

En ambos casos, la alerta no solo indica los síntomas detectados, sino que también ofrece datos que necesita para diagnosticar el problema, como los informes de excepciones pertinentes.

![Correo electrónico procedente de diagnóstico proactivo](./media/devops/030.png)

El cliente Samtec afirmó: "Durante una migración reciente de características, nos encontramos con una base de datos no escalada lo suficiente que estaba llegando al límite de sus recursos y ocasionando tiempos de espera. Las alertas de detección proactiva nos salvaron literalmente mientras clasificábamos el problema, casi en tiempo real como anunciábamos. Estas alertas junto con las alertas de la plataforma Azure nos ayudaron a solucionar el problema de forma casi instantánea. Tiempo de inactividad total < 10 minutos".

## <a name="live-metrics-stream"></a>Secuencia de métricas en directo
La implementación de la compilación más reciente puede ser una experiencia Implementar la compilación más reciente puede ser una experiencia inquietante. Si hay problemas, querrá saberlo enseguida para poder dar marcha atrás en caso necesario. La secuencia de métricas activas le proporciona métricas claves con una latencia de aproximadamente un segundo.

![Métricas activas](./media/devops/0040.png)

Y le permite inspeccionar inmediatamente una muestra de los errores o excepciones.

![Eventos de error en vivo](./media/devops/002-live-stream-failures.png)

## <a name="application-map"></a>Mapa de aplicación
El mapa de aplicación detecta automáticamente la topología de la aplicación y sitúa la información del rendimiento en primer lugar, para que pueda identificar fácilmente los cuellos de botella y los flujos problemáticos en su entorno distribuido. Además, permite detectar dependencias de aplicaciones en servicios de Azure. Al comprender si el problema está relacionado con el código o con las dependencias puede clasificarlo y, en un solo lugar, profundizar en la experiencia relacionada con el diagnóstico. Por ejemplo, puede que los errores de la aplicación se deban a la degradación del rendimiento en el nivel de SQL. Con el mapa de aplicación, puede verlo inmediatamente y profundizar en la experiencia de SQL Index Advisor o de Query Insights.

![Mapa de aplicación](./media/devops/0050.png)

## <a name="application-insights-analytics"></a>Application Insights Analytics
Con [Analytics](../log-query/log-query-overview.md), puede escribir consultas arbitrarias en un lenguaje avanzado, tipo SQL.  El diagnóstico en la pila entera de aplicaciones se convierte en una tarea sencilla gracias a la conexión de diversas perspectivas, y puede formular las preguntas adecuadas para correlacionar el rendimiento del servicio con las métricas empresariales y la experiencia del cliente. 

Puede consultar todas instancias de telemetría y los datos sin procesar de métricas en el portal. El lenguaje incluye el filtro, unión, agregación y otras operaciones. Puede calcular campos y realizar análisis estadísticos. Dispone de visualizaciones tanto gráficas como tabulares.

![Gráfico de consultas y resultados de Analytics](./media/devops/0025.png)

Por ejemplo, es fácil:

* Segmentar los datos de rendimiento de solicitud de la aplicación por niveles de clientes para comprender su experiencia.
* Buscar códigos de error específicos o nombres de eventos personalizado durante las investigaciones del sitio activo.
* Profundizar en el uso de aplicaciones de clientes específicos para entender cómo se adquieren y adoptar las características.
* Realizar un seguimiento de las sesiones y los tiempos de respuesta para usuarios específicos para que los equipos de operaciones y soporte técnico puedan proporcionar soporte al cliente al instante.
* Determinar las características de aplicaciones utilizadas con frecuencia para responder a preguntas de priorización de características.

El cliente DNN afirmó: "Application Insights nos ha proporcionado la parte que faltaba de la ecuación para poder combinar, ordenar, consultar y filtrar los datos de acuerdo con nuestras necesidades. Al permitir que nuestro equipo use su propio ingenio y experiencia para buscar datos con un lenguaje de consulta avanzado, hemos podido descubrir cosas y resolver problemas que ni siquiera sabíamos que teníamos. Un gran número de respuestas interesantes proceden de las preguntas que comienzan con *'Me pregunto si...'* ".

## <a name="development-tools-integration"></a>Integración de herramientas de desarrollo
### <a name="configuring-application-insights"></a>Configuración de Application Insights
Visual Studio y Eclipse tienen herramientas para configurar los paquetes SDK correctos para el proyecto que está desarrollando. Hay un comando de menú para agregar Application Insights.

Si usa una plataforma de registro de seguimiento, como Log4N, NLog o System.Diagnostics.Trace, tiene la opción de enviar los registros a Application Insights junto con los demás datos de telemetría, de modo que pueda correlacionar fácilmente los seguimientos con solicitudes, llamadas de dependencia y excepciones.

### <a name="search-telemetry-in-visual-studio"></a>Telemetría de búsqueda en Visual Studio
Al desarrollar y depurar una característica, puede ver y buscar la telemetría directamente en Visual Studio, con las mismas utilidades de búsqueda que en el portal web.

Y cuando Application Insights registra una excepción, puede ver el punto de datos en Visual Studio y saltar directamente al código pertinente.

![Búsqueda de Visual Studio](./media/devops/060.png)

Durante la depuración, tiene la opción de conservar los datos de telemetría en el equipo de desarrollo y verlos en Visual Studio sin enviarlos al portal. Esta opción local evita mezclar la depuración con la telemetría de producción.

### <a name="work-items"></a>Elementos de trabajo
Cuando se genera una alerta, Application Insights puede crear automáticamente un elemento de trabajo en su sistema de seguimiento de trabajos.

## <a name="but-what-about"></a>Pero, ¿y qué sucede con...?
* [Privacidad y almacenamiento](./data-retention-privacy.md) : la telemetría se mantiene en servidores seguros de Azure.
* Rendimiento: el impacto es muy bajo. la telemetría se procesa por lotes.
* [Precios](./pricing.md) : puede comenzar de forma gratuita y continuar así mientras el volumen sea bajo.


## <a name="video"></a>Vídeo

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/112/player]

## <a name="next-steps"></a>Pasos siguientes
Empezar a usar Application Insights es fácil. Tiene opciones principalmente para:

* [Servidores IIS](./monitor-performance-live-website-now.md) y también para [Azure App Service](./app-insights-overview.md).
* Instrumentar el proyecto durante el desarrollo. Puede hacerlo para aplicaciones de [ASP.NET](./asp-net.md) o [Java](./java-get-started.md), así como para [Node.js](./nodejs.md) y [otros tipos](./platforms.md). 
* Instrumentar [cualquier página web](./javascript.md) agregando un fragmento de código corto.

