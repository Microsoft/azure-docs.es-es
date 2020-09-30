---
title: Application Insights para Azure Cloud Services | Microsoft Docs
description: Supervisión de los roles web y de trabajo de manera eficaz con Application Insights
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 09/05/2018
ms.openlocfilehash: 1662b45d8243217357d1e69124832c499d587812
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89437333"
---
# <a name="application-insights-for-azure-cloud-services"></a>Application Insights para Azure Cloud Services
[Application Insights][start] puede supervisar las [aplicaciones de Azure Cloud Service](https://azure.microsoft.com/services/cloud-services/) para comprobar la disponibilidad, el rendimiento, los errores y el uso al combinar datos de los SDK de Application Insights con datos de [Azure Diagnostics](../platform/diagnostics-extension-overview.md) de los servicios en la nube. Con los comentarios que obtendrá sobre el rendimiento y la eficacia de la aplicación en su entorno natural, pueda tomar decisiones meditadas sobre la dirección del diseño en cada ciclo de vida de desarrollo.

![Panel de información general](./media/cloudservices/overview-graphs.png)

## <a name="prerequisites"></a>Requisitos previos
Antes de comenzar, necesitará lo siguiente:

* Una suscripción a [Azure](https://azure.com). Inicie sesión con una cuenta Microsoft para Windows, Xbox Live u otros servicios en la nube de Microsoft. 
* Herramientas de Microsoft Azure 2.9 o posterior.
* Developer Analytics Tools 7.10 o posterior.

## <a name="get-started-quickly"></a>Introducción rápida
La manera más rápida y sencilla de supervisar su servicio en la nube con Application Insights es elegir esa opción al publicar su servicio en Azure.

![Ejemplo de página Configuración de diagnóstico](./media/cloudservices/azure-cloud-application-insights.png)

Esta opción instrumenta una aplicación en el entorno de ejecución, lo que le proporciona la telemetría que necesita para supervisar las solicitudes, excepciones y dependencias en el rol web. También supervisa los contadores de rendimiento de los roles de trabajo. Los seguimientos de diagnóstico que la aplicación genera también se envían a Application Insights.

Si esta opción es todo lo que necesita, ha terminado. 

Los siguientes pasos son [ver las métricas de la aplicación](../platform/metrics-charts.md) y [consultar los datos con Analytics](../log-query/log-query-overview.md). 

Para supervisar el rendimiento en el explorador, puede que quiera configurar [pruebas de disponibilidad](./monitor-web-app-availability.md) y [agregar código a las páginas web](./javascript.md).

Las secciones siguientes tratan las siguientes opciones adicionales:

* Enviar datos de diferentes componentes y crear configuraciones para separar recursos.
* Agregar telemetría personalizada desde su aplicación.

## <a name="sample-app-instrumented-with-application-insights"></a>Aplicación de ejemplo instrumentada con Application Insights
En esta [aplicación de ejemplo](https://github.com/Microsoft/ApplicationInsights-Home/tree/master/Samples/AzureEmailService), Application Insights se agrega a un servicio en la nube con dos roles de trabajo hospedados en Azure. 

En la sección a continuación, se explica cómo adaptar su propio proyecto de servicio en la nube de la misma manera.

## <a name="plan-resources-and-resource-groups"></a>Planeamiento de recursos y grupos de recursos
La telemetría de la aplicación se almacena, analiza y muestra en un recurso de Azure de tipo Application Insights. 

Cada recurso pertenece a un grupo de recursos. Los grupos de recursos se usan para administrar los costos, para conceder acceso a los miembros del equipo y para implementar las actualizaciones en una transacción coordinada única. Por ejemplo, podría [escribir un script para implementar](../../azure-resource-manager/templates/deploy-powershell.md) un servicio en la nube de Azure y sus recursos de supervisión de Application Insights en una operación todo en uno.

### <a name="resources-for-components"></a>Recursos para componentes
Le recomendamos que cree un recurso independiente para cada componente de la aplicación. Es decir, crear un recurso para cada rol web y rol de trabajo. Puede analizar cada componente por separado, pero crear un [panel](./overview-dashboard.md) que reúna los gráficos de clave de todos los componentes, de modo que pueda compararlos y supervisarlos conjuntamente de un solo vistazo. 

Un enfoque alternativo consiste en enviar la telemetría de más de un rol al mismo recurso, pero [agregar una propiedad de dimensión para cada elemento de telemetría](./api-filtering-sampling.md#addmodify-properties-itelemetryinitializer) que identifique su rol de origen. En este enfoque, los gráficos de métricas, como las excepciones, normalmente muestran una agregación de las cuentas de diferentes roles, pero puede segmentar el gráfico por el identificador de rol cuando sea necesario. También se pueden filtrar las búsquedas por la misma dimensión. Esta alternativa hace un poco más fácil el poder ver todo al mismo tiempo, pero también puede causar confusión entre los roles.

La telemetría del explorador suele incluirse en el mismo recurso que su rol web de lado servidor.

Coloque los recursos de Application Insights para los diferentes componentes en un grupo de recursos. Este enfoque facilita administrarlos juntos. 

### <a name="separate-development-test-and-production"></a>Separación de desarrollo, prueba y producción
Si va a desarrollar eventos personalizados para la característica siguiente mientras la versión anterior está activa, querrá enviar la telemetría de desarrollo a un recurso de Application Insights independiente. De lo contrario, puede resultar difícil encontrar su telemetría de prueba entre todo el tráfico desde el sitio activo.

Para evitar esta situación, cree recursos independientes para cada configuración de compilación o "sello" (desarrollo, prueba, producción, etc.) del sistema. Coloque los recursos para cada configuración de compilación en un grupo de recursos independiente. 

Para enviar la telemetría a los recursos adecuados, puede configurar el SDK de Application Insights para que seleccione una clave de instrumentación diferente dependiendo de la configuración de compilación. 

Obtenga información acerca de cómo [establecer dinámicamente la clave de instrumentación](https://docs.microsoft.com/azure/azure-monitor/app/separate-resources#dynamic-ikey) para diferentes fases. 

## <a name="create-an-application-insights-resource-for-each-role"></a>Creación de un recurso de Application Insights para cada rol

Si ha decidido crear un recurso independiente para cada rol, y quizás otro conjunto distinto para cada configuración de compilación, es más fácil crearlos en el portal de Application Insights. Si crea muchos recursos, puede [automatizar el proceso](./powershell.md).

1. En [Azure Portal][portal], seleccione **Nuevo** > **Servicios de desarrollo** > **Application Insights**.  

    ![Panel de Application Insights](./media/cloudservices/01-new.png)

1. En la lista desplegable **Tipo de aplicación**, seleccione **Aplicación web ASP.NET**.

Cada recurso se identifica con una clave de instrumentación. Podría necesitar esta clave más adelante si quisiera configurar o comprobar manualmente la configuración del SDK.


## <a name="set-up-azure-diagnostics-for-each-role"></a>Configuración de Diagnósticos de Azure para cada rol
Establezca esta opción para supervisar la aplicación con Application Insights. Para los roles web, esta opción permite la supervisión del rendimiento, alertas y diagnósticos, así como el análisis del uso. Para otros roles, puede buscar y supervisar Azure Diagnostics, como el reinicio, los contadores de rendimiento y las llamadas a System.Diagnostics.Trace. 

1. En el Explorador de soluciones de Visual Studio, en **\<YourCloudService>**  > **Roles**, abra las propiedades de cada rol.

1. En **Configuración**, seleccione la casilla **Enviar datos de diagnóstico a Application Insights** y seleccione el recurso de Application Insights adecuado que creó anteriormente.

Si ha decidido usar un recurso de Application Insights distinto para cada configuración de compilación, seleccione primero la configuración.

![Configuración de Application Insights](./media/cloudservices/configure-azure-diagnostics.png)

Esto tiene el efecto de insertar las claves de instrumentación de Application Insights en los archivos denominados *ServiceConfiguration.\*.cscfg*. Este es el [ejemplo de código](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/AzureEmailService/ServiceConfiguration.Cloud.cscfg).

Si quiere modificar el nivel de la información de diagnóstico enviada a Application Insights, puede hacerlo [editando los archivos *.cscfg* directamente](../platform/diagnostics-extension-to-application-insights.md).

## <a name="install-the-sdk-in-each-project"></a><a name="sdk"></a>Instalación del SDK en cada proyecto
Con esta opción, puede agregar telemetría de negocio personalizada a cualquier rol. La opción proporciona un análisis más detallado de cómo la aplicación se usa y se comporta.

En Visual Studio, configure el SDK de Application Insights para cada proyecto de aplicación en la nube.

1. Para configurar los **roles web**, haga clic con el botón derecho en el proyecto y seleccione **Configurar Application Insights** o **Agregar > Telemetría de Application Insights**.

1. Para configurar **roles de trabajo**: 

    a. Haga clic con el botón derecho en el proyecto y seleccione **Administrar paquetes NuGet**.

    b. Agregue [Application Insights para servidores de Windows](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsServer/).

    ![Busque "Application Insights"](./media/cloudservices/04-ai-nuget.png)

1. Para configurar el SDK para enviar datos al recurso de Application Insights.

    a. En una función de inicio adecuada, establezca la clave de instrumentación a partir de la opción de configuración del archivo *.cscfg*:
 
    ```csharp
   
     TelemetryConfiguration.Active.InstrumentationKey = RoleEnvironment.GetConfigurationSettingValue("APPINSIGHTS_INSTRUMENTATIONKEY");
    ```
   
    b. Repita el "paso a" para cada rol de la aplicación. Consulte los ejemplos:
   
    * [Rol web](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/Global.asax.cs#L27)
    * [Rol de trabajo](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/WorkerRoleA.cs#L232)
    * [Para las páginas web](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/Views/Shared/_Layout.cshtml#L13) 

1. Establezca el archivo *ApplicationInsights.config* para que se copie siempre en el directorio de salida.

   Aparece un mensaje en el archivo *.config* en el que se le pide que coloque allí la clave de instrumentación. Sin embargo, para las aplicaciones de nube es mejor establecerla desde el archivo *.cscfg*. Este enfoque garantiza que el rol se identifica correctamente en el portal.

## <a name="set-up-status-monitor-to-collect-full-sql-queries-optional"></a>Configuración del monitor de estado para recopilar consultas completas de SQL (opcional)

Este paso solo es necesario si quiere capturar consultas completas de SQL en .NET Framework. 

1. En el archivo `\*.csdef`, agregue una [tarea de inicio](../../cloud-services/cloud-services-startup-tasks.md) para todos los roles similares a 

    ```xml
    <Startup>
      <Task commandLine="AppInsightsAgent\InstallAgent.bat" executionContext="elevated" taskType="simple">
        <Environment>
          <Variable name="ApplicationInsightsAgent.DownloadLink" value="http://go.microsoft.com/fwlink/?LinkID=522371" />
          <Variable name="RoleEnvironment.IsEmulated">
            <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
          </Variable>
        </Environment>
      </Task>
    </Startup>
    ```
    
2. Descargue [InstallAgent.bat](https://github.com/microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/AppInsightsAgent/InstallAgent.bat) e [InstallAgent.ps1](https://github.com/microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/AppInsightsAgent/InstallAgent.ps1) y colóquelos en la carpeta `AppInsightsAgent` de cada proyecto de rol. Asegúrese de copiarlos en el directorio de salida a través de las propiedades de archivo de Visual Studio o mediante scripts de compilación.

3. En todos los roles de trabajo, agregue las siguientes variables de entorno: 

    ```xml
      <Environment>
        <Variable name="COR_ENABLE_PROFILING" value="1" />
        <Variable name="COR_PROFILER" value="{324F817A-7420-4E6D-B3C1-143FBED6D855}" />
        <Variable name="MicrosoftInstrumentationEngine_Host" value="{CA487940-57D2-10BF-11B2-A3AD5A13CBC0}" />
      </Environment>
    ```
    
## <a name="run-and-publish-the-app"></a>Ejecución y publicación de la aplicación

1. Ejecute la aplicación e inicie sesión en Azure. 

1. Abra los recursos de Application Insights que creó.

   Los puntos de datos individuales se muestran en [Buscar][diagnostic] y los datos agregados se muestran en el [Explorador de métricas](../platform/metrics-charts.md).

1. Agregue más telemetría (consulte las secciones siguientes) y luego publique la aplicación para obtener diagnósticos en vivo y comentarios sobre el uso. 

Si no hay ningún dato, haga lo siguiente:

1. Para ver los eventos individuales, abra el icono [Búsqueda][diagnostic].
1. En la aplicación, abra distintas páginas para que genere telemetría.
1. Espere unos segundos y haga clic en **Actualizar**.  

Para más información, consulte [Solución de problemas][qna].

## <a name="view-azure-diagnostics-events"></a>Visualización de eventos de Microsoft Azure Diagnostics
En las siguientes ubicaciones, puede encontrar la información de [Azure Diagnostics](../platform/diagnostics-extension-overview.md) en Application Insights:

* Los contadores de rendimiento se muestran como métricas personalizadas. 
* Los registros de eventos de Windows se muestran como seguimientos y eventos personalizados.
* Los registros de aplicación, los registros ETW y los registros de infraestructura de diagnóstico aparecen como seguimientos.

Para ver los recuentos de eventos y los contadores de rendimiento, abra [Explorador de métricas](../platform/metrics-charts.md) y agregue un nuevo gráfico:

![Datos de Azure Diagnostics](./media/cloudservices/23-wad.png)

Para buscar en los distintos registros de seguimiento enviados por Azure Diagnostics, use [Búsqueda](./diagnostic-search.md) o una [consulta de Analytics](../log-query/get-started-portal.md). Por ejemplo, suponga que tiene una excepción no controlada que provocó el bloqueo y reciclaje de un rol. Esa información aparecería en el canal Aplicación del registro de eventos de Windows. Puede utilizar Buscar para ver el error del registro de eventos de Windows y obtener el seguimiento de la pila completo para la excepción. Así podrá encontrar la causa principal del problema.

![Búsqueda de Azure Diagnostics](./media/cloudservices/25-wad.png)

## <a name="more-telemetry"></a>Más telemetría
En las secciones siguientes se describe cómo obtener telemetría adicional de diversos aspectos de la aplicación.

## <a name="track-requests-from-worker-roles"></a>Seguimiento de las solicitudes de roles de trabajo
En los roles web, el módulo de solicitudes recopila automáticamente datos acerca de las solicitudes HTTP. Puede obtener ejemplos de cómo invalidar el comportamiento de la colección predeterminada si consulta el [MVCWebRole de ejemplo](https://github.com/Microsoft/ApplicationInsights-Home/tree/master/Samples/AzureEmailService/MvcWebRole). 

Puede capturar el rendimiento de las llamadas a los roles de trabajo realizando su seguimiento de la misma manera que con las solicitudes HTTP. En Application Insights, el tipo de telemetría Solicitud mide una unidad de trabajo del lado servidor con nombre que se puede programar y puede realizarse correctamente o producir un error de forma independiente. Si bien el SDK captura automáticamente las solicitudes HTTP, puede insertar su propio código para realizar el seguimiento de las solicitudes a los roles de trabajo.

Consulte los dos roles de trabajo de ejemplo instrumentados en las solicitudes de informes: 
* [WorkerRoleA](https://github.com/Microsoft/ApplicationInsights-Home/tree/master/Samples/AzureEmailService/WorkerRoleA)
* [WorkerRoleB](https://github.com/Microsoft/ApplicationInsights-Home/tree/master/Samples/AzureEmailService/WorkerRoleB)

## <a name="exceptions"></a>Excepciones
Para obtener información acerca de cómo puede recopilar excepciones no controladas de tipos de aplicaciones web diferentes, consulte [Supervisión de excepciones en Application Insights](./asp-net-exceptions.md).

El rol de web ejemplo tiene los controladores MVC5 y Web API 2. Las excepciones no controladas de los dos se capturan con los siguientes controladores:

* La configuración de [AiHandleErrorAttribute](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/Telemetry/AiHandleErrorAttribute.cs) para los controladores MVC5 [se muestran en este ejemplo](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/App_Start/FilterConfig.cs#L12) 
* La configuración de [AiWebApiExceptionLogger](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/Telemetry/AiWebApiExceptionLogger.cs) para los controladores Web API 2 [se muestran en este ejemplo](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/App_Start/WebApiConfig.cs#L25) 

Para los roles de trabajo, el seguimiento de las excepciones se puede realizar de dos maneras:

* Use TrackException(ex).
* Si ha agregado el paquete NuGet del agente de escucha de seguimiento de Application Insights, puede usar System.Diagnostics.Trace para registrar las excepciones [como se muestra en este ejemplo](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/WorkerRoleA.cs#L107).

## <a name="performance-counters"></a>Contadores de rendimiento
Los siguientes contadores se recopilan de forma predeterminada:

* \Process(??APP_WIN32_PROC??)\% Tiempo de procesador
* \Memoria\Bytes disponibles
* \.NET CLR Exceptions(??APP_CLR_PROC??)\# of Exceps Thrown / sec
* \Process(??APP_WIN32_PROC??)\Private Bytes
* \Process(??APP_WIN32_PROC??)\IO Data Bytes/sec
* Procesador(_Total)\% Hora del procesador

Además, también se recopilan los siguientes contadores para los roles web:

* \ASP.NET Applications(??APP_W3SVC_PROC??)\Requests/Sec
* \ASP.NET Applications(??APP_W3SVC_PROC??)\Request Execution Time
* \ASP.NET Applications(??APP_W3SVC_PROC??)\Requests In Application Queue

Puede especificar otros contadores de rendimiento de Windows o personalizados si modifica *ApplicationInsights.config* [como se muestra en este ejemplo](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/ApplicationInsights.config#L14).

  ![Contadores de rendimiento](./media/cloudservices/002-servers.png)

## <a name="correlated-telemetry-for-worker-roles"></a>Telemetría correlacionada para roles de trabajo
Para obtener una sofisticada experiencia de diagnóstico, puede ver qué provocó una solicitud de latencia alta o con errores. Con los roles web, el SDK establece automáticamente una correlación entre la telemetría relacionada. 

Si desea lograr esta vista para los roles de trabajo, puede utilizar un inicializador de telemetría personalizado para establecer un atributo de contexto Operation.Id común para toda la telemetría. Ello le permite comprobar de un vistazo si el problema de latencia o de los errores se produjo debido a una dependencia o al código. 

A continuación, se indica cómo puede hacerlo.

* Establezca el identificador de la correlación en una CallContext [como se muestra en este ejemplo](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/WorkerRoleA.cs#L36). En este caso, usamos el identificador de solicitud como identificador de correlación.
* Agregue una implementación personalizada de TelemetryInitializer para establecer el Operation.Id en el correlationId que se estableció antes. Para obtener un ejemplo, consulte [ItemCorrelationTelemetryInitializer](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/Telemetry/ItemCorrelationTelemetryInitializer.cs#L13).
* Agregue el inicializador de telemetría personalizado. Puede hacerlo en el archivo *ApplicationInsights.config* o en el código [según se muestra en este ejemplo](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/WorkerRoleA.cs#L233).

## <a name="client-telemetry"></a>Telemetría de cliente
Para obtener telemetría basada en el navegador (como recuentos de vista de página, tiempos de carga de página y excepciones de script) y escribir telemetría personalizada en los scripts de página, consulte [Adición del SDK de JavaScript a las páginas web][client].

## <a name="availability-tests"></a>Pruebas de disponibilidad
Para comprobar que la aplicación efectivamente está activa y responde adecuadamente, [configure las pruebas web][availability].

## <a name="display-everything-together"></a>Mostrar todos los elementos juntos
Para obtener una imagen general del sistema, puede mostrar los gráficos de supervisión clave en un [panel](./overview-dashboard.md). Por ejemplo, puede anclar la solicitud y el número de errores de cada rol. 

Si el sistema usa otros servicios de Azure, como Stream Analytics, incluya también sus gráficos de supervisión. 

Si tiene una aplicación móvil cliente, utilice [App Center](../learn/mobile-center-quickstart.md). Cree consultas en [Analytics](../log-query/log-query-overview.md) para mostrar los recuentos de eventos y anclarlos al panel.

## <a name="example"></a>Ejemplo
[El ejemplo](https://github.com/Microsoft/ApplicationInsights-Home/tree/master/Samples/AzureEmailService) supervisa un servicio que tiene un rol web y dos roles de trabajo.

## <a name="exception-method-not-found-on-running-in-azure-cloud-services"></a>Excepción "Método no encontrado" en la ejecución en Azure Cloud Services
¿Ha realizado la compilación para .NET 4.6? .NET 4.6 no se admite automáticamente en los roles de Azure Cloud Service. [Instale .NET 4.6 en cada rol](../../cloud-services/cloud-services-dotnet-install-dotnet.md) antes de ejecutar la aplicación.

## <a name="video"></a>Vídeo

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/100/player]

## <a name="next-steps"></a>Pasos siguientes
* [Configuración del envío de diagnósticos de Azure a Application Insights](../platform/diagnostics-extension-to-application-insights.md)
* [Creación automática de recursos de Application Insights](./powershell.md)
* [Automatización de Azure Diagnostics](./powershell-azure-diagnostics.md)
* [Funciones de Azure](https://github.com/christopheranderson/azure-functions-app-insights-sample)

[api]: ./api-custom-events-metrics.md
[availability]: ./monitor-web-app-availability.md
[azure]: ./app-insights-overview.md
[client]: ./javascript.md
[diagnostic]: ./diagnostic-search.md
[netlogs]: ./asp-net-trace-logs.md
[portal]: https://portal.azure.com/
[qna]: ../faq.md
[redfield]: ./monitor-performance-live-website-now.md
[start]: ./app-insights-overview.md

