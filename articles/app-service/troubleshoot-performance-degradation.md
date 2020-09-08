---
title: Solución de problemas de degradación del rendimiento
description: Descubra cómo solucionar problemas de rendimiento lento de aplicaciones en Azure App Service, incluyendo la supervisión del comportamiento de la aplicación, la recopilación de datos y la mitigación del problema.
tags: top-support-issue
keywords: rendimiento de aplicaciones web, aplicación lenta, aplicaciones lentas
ms.assetid: b8783c10-3a4a-4dd6-af8c-856baafbdde5
ms.topic: article
ms.date: 08/03/2016
ms.custom: seodec18
ms.openlocfilehash: a9d3de5d4a88c782ad541ceb4916ec90a3bdd7b5
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88958259"
---
# <a name="troubleshoot-slow-app-performance-issues-in-azure-app-service"></a>Solucionar los problemas de rendimiento reducido de aplicaciones web en Azure App Service
Este artículo lo ayuda a solucionar los problemas de rendimiento reducido en las aplicaciones de [Azure App Service](https://go.microsoft.com/fwlink/?LinkId=529714).

Si necesita más ayuda en cualquier momento con este artículo, puede ponerse en contacto con los expertos de Azure en [los foros de MSDN Azure y de desbordamiento de pila](https://azure.microsoft.com/support/forums/). Como alternativa, también puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y haga clic en **Obtener soporte técnico**.

## <a name="symptom"></a>Síntoma
Al examinar la aplicación, las páginas se cargan lentamente y, en ocasiones, se agota el tiempo de espera.

## <a name="cause"></a>Causa
Con frecuencia este problema se debe a problemas en el nivel de la aplicación, como, por ejemplo:

* Solicitudes de redes que tardan mucho tiempo
* Consultas de código de aplicación o base de datos que no son eficaces
* Aplicaciones que hacen un uso elevado de memoria y CPU
* Aplicaciones que se bloquean debido a una excepción

## <a name="troubleshooting-steps"></a>Pasos para solucionar problemas
El procedimiento de solución de problemas se puede dividir en tres tareas distintas, en orden secuencial:

1. [Observación y supervisión del comportamiento de la aplicación](#observe)
2. [Recopilación de datos](#collect)
3. [Mitigación del problema](#mitigate)

[App Service](overview.md) ofrece diversas opciones en cada paso.

<a name="observe"></a>

### <a name="1-observe-and-monitor-application-behavior"></a>1. Observación y supervisión del comportamiento de la aplicación
#### <a name="track-service-health"></a>Seguimiento del estado del servicio
Cada vez que hay una interrupción del servicio o una degradación del rendimiento, Microsoft Azure lo anuncia. Puede realizar un seguimiento del estado del servicio en [Azure Portal](https://portal.azure.com/). Para más información, consulte [Track service health](../service-health/service-notifications.md) (Seguimiento del estado del servicio).

#### <a name="monitor-your-app"></a>Supervise la aplicación
Esta opción le permite averiguar si la aplicación tiene problemas. En la hoja de la aplicación, haga clic en el icono **Solicitudes y errores**. La hoja **Métrica** muestra todas las métricas que puede agregar.

Algunas de las métricas que podría querer supervisar para su aplicación son:

* Espacio de trabajo de memoria promedio
* Tiempo medio de respuesta
* Tiempo de CPU
* Espacio de trabajo de memoria
* Requests

![supervisar el rendimiento de aplicaciones](./media/app-service-web-troubleshoot-performance-degradation/1-monitor-metrics.png)

Para más información, consulte:

* [Supervisión de aplicaciones en Azure App Service](web-sites-monitor.md)
* [Recibir notificaciones de alerta](../azure-monitor/platform/alerts-overview.md)

#### <a name="monitor-web-endpoint-status"></a>estado de extremo web
Si ejecuta la aplicación en el plan de tarifa **Estándar**, App Service le permite supervisar dos puntos de conexión de tres ubicaciones geográficas.

La supervisión de extremo configura pruebas web desde ubicaciones distribuidas geográficamente que prueban el tiempo de respuesta y el tiempo activo de direcciones URL web. La prueba realiza una operación HTTP Get en la dirección URL web para determinar el tiempo de respuesta y el tiempo de actividad desde cada ubicación. Cada ubicación configurada ejecuta una prueba cada cinco minutos.

El tiempo activo se supervisa a través de códigos de respuesta de HTTP y el tiempo de respuesta se mide en milisegundos. Una prueba de supervisión da error si el código de respuesta HTTP es mayor o igual que 400 o si la respuesta demora más de 30 segundos. Un extremo se considera disponible si sus pruebas de supervisión se realizan correctamente desde todas las ubicaciones especificadas.

Para configurarlo, consulte [Supervisión de Aplicaciones en Azure App Service](web-sites-monitor.md).

Consulte también [Mantenimiento de Sitios web de Azure activos y supervisión de puntos de conexión: con Stefan Schackow](https://channel9.msdn.com/Shows/Azure-Friday/Keeping-Azure-Web-Sites-up-plus-Endpoint-Monitoring-with-Stefan-Schackow) para ver un vídeo sobre la supervisión de los puntos de conexión.

#### <a name="application-performance-monitoring-using-extensions"></a>Supervisión del rendimiento de la aplicación mediante el uso de extensiones
También puede supervisar el rendimiento de la aplicación mediante una *extensión de sitio*.

Cada aplicación de App Service proporciona un punto de conexión de administración extensible que le permite usar un eficaz conjunto de herramientas implementadas como extensiones del sitio. Entre las extensiones se incluyen: 

- Editores de código fuente como [Azure DevOps](https://www.visualstudio.com/products/what-is-visual-studio-online-vs.aspx). 
- Las herramientas de administración de los recursos conectados como, por ejemplo, una base de datos de MySQL conectada a una aplicación.

[Azure Application Insights](https://azure.microsoft.com/services/application-insights/) es una extensión de sitio de supervisión de rendimiento que está también disponible. Para usar Application Insights, recompile el código con un SDK. También puede instalar una extensión que proporcione acceso a datos adicionales. El SDK permite escribir código para supervisar el uso y el rendimiento de la aplicación con más detalle. Para más información, consulte [Supervisar el rendimiento de aplicaciones web](../azure-monitor/app/web-monitor-performance.md).

<a name="collect"></a>

### <a name="2-collect-data"></a>2. Recopilación de datos
App Service ofrece la funcionalidad de diagnóstico para registrar información del servidor web y de la aplicación web. La información se divide en diagnósticos del servidor web y diagnósticos de aplicaciones.

#### <a name="enable-web-server-diagnostics"></a>Habilitar diagnósticos del servidor web
Puede habilitar o deshabilitar los siguientes tipos de registros:

* **Registro de errores detallado** : registra información detallada de errores para códigos de estado HTTP que indican un problema (código de estado 400 o superior). Puede contener información que puede ayudar a determinar por qué el servidor ha devuelto el código de error.
* **Seguimiento de solicitudes con error** : registra información detallada acerca de solicitudes con error, incluido un seguimiento de los componentes de IIS usados para procesar la solicitud y el tiempo dedicado a cada componente. Esto puede resultar útil si está intentando mejorar el rendimiento de las aplicaciones o de aislar la causa de un error HTTP determinado.
* **Registro del servidor web** : registra todas las transacciones HTTP con el formato de archivo de registro extendido de W3C. Este informe resulta útil para determinar las métricas totales de la aplicación, como el número de solicitudes tramitadas o cuántas solicitudes proceden de una dirección IP específica.

#### <a name="enable-application-diagnostics"></a>Habilitar diagnósticos de la aplicación
Existen varias opciones para recopilar los datos de rendimiento de App Service, generar el perfil de la aplicación en directo desde Visual Studio, o modificar el código de la aplicación para registrar más información y seguimientos. Puede elegir las opciones en función de cuántos accesos tiene a la aplicación y de lo que ha observado con las herramientas de supervisión.

##### <a name="use-application-insights-profiler"></a>Uso de Application Insights Profiler
Puede habilitar Application Insights Profiler para empezar a capturar seguimientos detallados de rendimiento. Puede acceder a los seguimientos capturados de hace cinco días como máximo cuando lo necesite para investigar los problemas que se produjeron en el pasado. Puede elegir esta opción siempre que tenga acceso al recurso de Application Insights de la aplicación en Azure Portal.

Application Insights Profiler proporciona estadísticas sobre el tiempo de respuesta de cada llamada web y seguimiento que indican cuál es la línea de código que provocó las respuestas lentas. En ocasiones, la aplicación de App Service es lenta porque determinado código no está escrito pensando en el rendimiento. Por ejemplo, código secuencial que se puede ejecutar en paralelo y contenciones de bloqueo de base de datos no deseadas. La eliminación de estos cuellos de botella en el código aumenta el rendimiento de la aplicación, pero son difíciles de detectar sin configurar elaborados registros y seguimientos. Los seguimientos recopilados por Application Insights Profiler ayudan a identificar las líneas de código que ralentizan la aplicación y a superar este desafío para las aplicaciones de App Service.

 Para más información, consulte [Generación de perfiles de aplicaciones activas en Azure App Service con Application Insights](../azure-monitor/app/profiler.md).

##### <a name="use-remote-profiling"></a>Uso de la generación remota de perfiles
En instancias de Azure App Service, aplicaciones web, aplicaciones de la API, back-ends móviles y WebJobs se pueden generar perfiles de forma remota. Elija esta opción si tiene acceso al recurso de la aplicación y sabe cómo reproducir el problema o si conoce el intervalo de tiempo exacto en que ocurre el problema de rendimiento.

La generación de perfiles remota puede resultar útil si el uso de CPU del proceso es elevado y el proceso se ejecuta más lento de lo esperado o la latencia de las solicitudes HTTP es superior a la normal. En estos casos, puede generar un perfil para su proceso de forma remota y obtener las pilas de llamada de muestreo de CPU para analizar la actividad del proceso y las rutas de acceso activas del código.

Para más información, consulte [Compatibilidad con la generación remota de perfiles en Azure App Service](https://azure.microsoft.com/blog/remote-profiling-support-in-azure-app-service).

##### <a name="set-up-diagnostic-traces-manually"></a>Establecimiento manual de los seguimientos de diagnóstico
Si tiene acceso al código fuente de la aplicación web, el diagnóstico de aplicaciones le permite capturar información generada por una aplicación web. Las aplicaciones de ASP.NET pueden usar la clase `System.Diagnostics.Trace` para registrar información en el registro de diagnóstico de aplicaciones. Sin embargo, debe cambiar el código y volver a implementar la aplicación. Este método es recomendable si la aplicación se ejecuta en un entorno de prueba.

Para obtener instrucciones detalladas sobre cómo configurar su aplicación para el registro, consulte [Habilitación del registro de diagnóstico para aplicaciones en Azure App Service](troubleshoot-diagnostic-logs.md).

#### <a name="use-the-diagnostics-tool"></a>Uso de la herramienta de diagnósticos
App Service proporciona una experiencia inteligente e interactiva que le ayuda a solucionar los problemas de su aplicación sin necesidad de configurar nada. Cuando surgen problemas con la aplicación, la herramienta de diagnóstico le indicará lo que pasa y le guiará a la información correcta de forma que pueda solucionar el problema de la manera más fácil y rápida.

Para acceder al diagnóstico de App Service, vaya a la aplicación de App Service o del entorno de App Service en [Azure Portal](https://portal.azure.com). En el panel izquierdo, haga clic en **Diagnosticar y solucionar problemas**.

#### <a name="use-the-kudu-debug-console"></a>Uso de la consola de depuración Kudu
App Service incluye una consola de depuración que puede usar para depurar, explorar o cargar archivos, e incluye también puntos de conexión JSON para obtener información sobre su entorno. A esta consola se le denomina *Consola Kudu* o *Panel SCM* para la aplicación.

Puede tener acceso a este panel en el vínculo **https://&lt;Nombre de la aplicación>.scm.azurewebsites.net/** .

Algunas de las cosas que proporciona Kudu son:

* Configuración del entorno de la aplicación
* transmisión de registro
* Volcado de diagnóstico
* Depuración de la consola en la que puede ejecutar cmdlets de Powershell y comandos básicos de DOS.

Otra característica útil de Kudu es que, en caso de que la aplicación inicie excepciones de primera oportunidad, puede usar Kudu y la herramienta Procdump de SysInternals para crear volcados de memoria. Estos volcados de memoria son instantáneas del proceso y a menudo pueden ayudarle solucionar problemas más complicados de su aplicación.

Para más información sobre las características disponibles en Kudu, vea [Herramientas en línea de Azure Websites que debe conocer](https://azure.microsoft.com/blog/windows-azure-websites-online-tools-you-should-know-about/).

<a name="mitigate"></a>

### <a name="3-mitigate-the-issue"></a>3. Mitigación del problema
#### <a name="scale-the-app"></a>Escalar la aplicación
En Azure App Service, puede ajustar la escala en la que se ejecuta la aplicación para aumentar el rendimiento y la capacidad de proceso. El escalado vertical de una aplicación implica dos acciones relacionadas: cambiar el plan de App Service por un plan de tarifa más alto y configurar determinados valores después de haber cambiado a ese plan de tarifa más alto.

Para obtener más información sobre el escalado, consulte [Escalado de una aplicación en Azure App Service](manage-scale-up.md).

Además, puede elegir ejecutar la aplicación en más de una instancia. El escalado no solo le proporciona una mayor capacidad de procesamiento, sino también algo de tolerancia a errores. Si el proceso se interrumpe en una instancia, las otras instancias seguirán atendiendo las solicitudes.

Puede establecer el escalado en Manual o Automático.

#### <a name="use-autoheal"></a>Uso de AutoHeal
AutoHeal recicla el proceso de trabajo para su aplicación en función de la configuración que elija (como cambios de configuración, solicitudes, límites de memoria o el tiempo necesario para ejecutar una solicitud). Casi siempre, el proceso de reciclaje es la forma más rápida de recuperarse de un problema. Aunque siempre puede reiniciar la aplicación directamente en Azure Portal, AutoHeal lo hará automáticamente por usted. Todo lo que debe hacer es agregar algunos desencadenadores en web.config raíz de la aplicación. Esta configuración funcionaría igual aunque la aplicación no fuera de .NET.

Para obtener más información, consulte [Recuperación automática de Sitios web de Azure](https://azure.microsoft.com/blog/auto-healing-windows-azure-web-sites/).

#### <a name="restart-the-app"></a>Reiniciar la aplicación
Reiniciar suele ser la manera más sencilla de recuperarse de problemas que solo tienen lugar una vez. En el [Azure Portal](https://portal.azure.com/), en la hoja de la aplicación, tiene las opciones para detener o reiniciar la aplicación.

 ![reiniciar las aplicaciones para resolver los problemas de rendimiento](./media/app-service-web-troubleshoot-performance-degradation/2-restart.png)

También puede administrar la aplicación con Azure Powershell. Para obtener más información, vea [Uso de Azure PowerShell con el Administrador de recursos de Azure](../azure-resource-manager/management/manage-resources-powershell.md).