---
title: 'Preguntas más frecuentes sobre el rendimiento de aplicaciones: Azure App Service | Microsoft Docs'
description: Conozca las respuestas a las preguntas más frecuentes sobre la disponibilidad, rendimiento y problemas de la aplicación en la característica Web Apps de Azure App Service.
services: app-service\web
documentationcenter: ''
author: genlin
manager: cshepard
editor: ''
tags: top-support-issue
ms.assetid: 2fa5ee6b-51a6-4237-805f-518e6c57d11b
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: article
ms.date: 10/31/2018
ms.author: genli
ms.custom: seodec18
ms.openlocfilehash: 3dd61ae9298a03ce0cbb9e9a5b143229c15ea9bb
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65786375"
---
# <a name="application-performance-faqs-for-web-apps-in-azure"></a>Preguntas más frecuentes sobre el rendimiento de aplicaciones para Web Apps de Azure

> [!NOTE]
> Algunas de las siguientes directrices solo funcionen en Windows o Linux App Services. Por ejemplo, App Services de Linux se ejecute en modo de 64 bits de forma predeterminada.
>

En este artículo se proporcionan las respuestas a las preguntas más frecuentes sobre los problemas relacionados con el rendimiento de las aplicaciones en la [característica Web Apps de Azure App Service](https://azure.microsoft.com/services/app-service/web/).

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="why-is-my-app-slow"></a>¿Por qué mi aplicación es lenta?

Varios factores podrían contribuir a la ralentización del rendimiento de la aplicación. Para obtener los pasos detallados de solución de problemas, consulte [Solucionar los problemas de rendimiento reducido de aplicaciones web en Azure Web Apps](troubleshoot-performance-degradation.md).

## <a name="how-do-i-troubleshoot-a-high-cpu-consumption-scenario"></a>¿Cómo puedo solucionar problemas en un escenario de consumo elevado de CPU?

En algunos escenarios de consumo elevado de CPU, es probable que la aplicación realmente requiera más recursos informáticos. En ese caso, considere la posibilidad de escalar a un mayor nivel de servicio para que la aplicación obtenga todos los recursos necesarios. En otras ocasiones, el consumo elevado de CPU podría deberse a un bucle incorrecto o una práctica de codificación. Obtener una visión general de lo que está desencadenando ese mayor consumo de CPU es un proceso de dos partes. En primer lugar, cree un volcado de proceso y, después, analice el volcado de memoria del proceso. Para más información, consulte la entrada de blog [Capture and analyze a dump file for high CPU consumption for Web Apps](https://blogs.msdn.microsoft.com/asiatech/2016/01/20/how-to-capture-dump-when-intermittent-high-cpu-happens-on-azure-web-app/) (Captura y análisis de un archivo de volcado de memoria para un consumo elevado de CPU para Web Apps).

## <a name="how-do-i-troubleshoot-a-high-memory-consumption-scenario"></a>¿Cómo puedo solucionar problemas en un escenario de consumo elevado de memoria?

En algunos escenarios de consumo elevado de memoria, es probable que la aplicación realmente requiera más recursos informáticos. En ese caso, considere la posibilidad de escalar a un mayor nivel de servicio para que la aplicación obtenga todos los recursos necesarios. En otras ocasiones, un error del código podría producir una pérdida de memoria. Una práctica de codificación también podría aumentar el consumo de memoria. Obtener una visión general de lo que está desencadenando ese mayor consumo de memoria es un proceso de dos partes. En primer lugar, cree un volcado de proceso y, después, analice el volcado de memoria del proceso. El diagnóstico de bloqueos de la galería de extensión de sitios de Azure puede realizar de manera eficaz estos dos pasos. Para más información, consulte la entrada de blog [How to capture and analyze dump for intermittent High Memory on Azure Web App](https://blogs.msdn.microsoft.com/asiatech/2016/02/02/how-to-capture-and-analyze-dump-for-intermittent-high-memory-on-azure-web-app/) (Captura y análisis de un archivo de volcado de memoria para un consumo elevado intermitente de memoria para Web Apps).

## <a name="how-do-i-automate-app-service-web-apps-by-using-powershell"></a>¿Cómo se puede automatizar App Service Web Apps mediante PowerShell?

Puede usar los cmdlets de PowerShell para administrar y mantener App Service Web Apps. En nuestra entrada de blog [Automate web apps hosted in Azure App Service by using PowerShell](https://blogs.msdn.microsoft.com/puneetgupta/2016/03/21/automating-webapps-hosted-in-azure-app-service-through-powershell-arm-way/) (Automatización de aplicaciones web hospedadas en Azure App Service mediante PowerShell), se describe cómo usar los cmdlets de PowerShell basada en Azure Resource Manager para automatizar tareas comunes. La entrada de blog también incluye código de ejemplo para diversas tareas de administración de aplicaciones web. Para obtener descripciones y sintaxis de todos los cmdlets de App Service Web Apps, consulte [Az.Websites](/powershell/module/az.websites).

## <a name="how-do-i-view-my-web-apps-event-logs"></a>¿Cómo se pueden ver los registros de eventos de mi aplicación web?

Para ver los registros de eventos de la aplicación web:

1. Inicie sesión en su [sitio web de Kudu](https://*yourwebsitename*.scm.azurewebsites.net).
2. En el menú, seleccione **Consola de depuración** > **CMD**.
3. Abra la carpeta **LogFiles**.
4. Para ver los registros de eventos, seleccione el icono de lápiz junto a **eventlog.xml**.
5. Para descargar los registros, ejecute el cmdlet de PowerShell `Save-AzureWebSiteLog -Name webappname`.

## <a name="how-do-i-capture-a-user-mode-memory-dump-of-my-web-app"></a>¿Cómo se puede capturar un volcado de memoria de modo de usuario de la aplicación web?

Para capturar un volcado de memoria de modo de usuario de la aplicación web:

1. Inicie sesión en su [sitio web de Kudu](https://*yourwebsitename*.scm.azurewebsites.net).
2. Seleccione el menú **Explorador de procesos**.
3. Haga clic con el botón derecho en el proceso **w3wp.exe** o en el proceso de trabajo web.
4. Seleccione **Download Memory Dump** > **Full Dump** (Descargar volcado de memoria > Volcado de memoria completo).

## <a name="how-do-i-view-process-level-info-for-my-web-app"></a>¿Cómo se puede ver la información de nivel de proceso para mi aplicación web?

Tiene dos opciones para ver la información de nivel de proceso para la aplicación web:

*   En Azure Portal:
    1. Abra el **explorador de procesos** para la aplicación web.
    2. Para ver los detalles, seleccione el proceso **w3wp.exe**.
*   En la consola de Kudu:
    1. Inicie sesión en su [sitio web de Kudu](https://*yourwebsitename*.scm.azurewebsites.net).
    2. Seleccione el menú **Explorador de procesos**.
    3. En el proceso **w3wp.exe**, seleccione **Propiedades**.

## <a name="when-i-browse-to-my-app-i-see-error-403---this-web-app-is-stopped-how-do-i-resolve-this"></a>Cuando navego a mi aplicación, veo el error 403 que indica que la aplicación web se ha detenido. ¿Cómo se resuelve este problema?

Este error pueden deberse a tres condiciones:

* La aplicación web ha alcanzado un límite de facturación y se ha deshabilitado el sitio.
* La aplicación web se detuvo en el portal.
* La aplicación web ha alcanzado un límite de cuota de recursos que se puede aplicar a un plan de servicio de escala Gratis o Compartido.

Para ver cuál es la causa del error y resolver el problema, siga los pasos de [Web Apps: "Error 403 – This web app is stopped"](https://blogs.msdn.microsoft.com/waws/2016/01/05/azure-web-apps-error-403-this-web-app-is-stopped/) (Web Apps: "Error 403: esta aplicación web se ha detenido").

## <a name="where-can-i-learn-more-about-quotas-and-limits-for-various-app-service-plans"></a>¿Dónde puedo obtener más información sobre cuotas y límites de los diversos planes de App Service?

Para más información sobre cuotas y límites, consulte [Límites de App Service](../azure-subscription-service-limits.md#app-service-limits). 

## <a name="how-do-i-decrease-the-response-time-for-the-first-request-after-idle-time"></a>¿Cómo se puede reducir el tiempo de respuesta de la primera solicitud después de tiempo de inactividad?

De forma predeterminada, las aplicaciones web se descargan si están inactivas durante algún tiempo. Esto permite que el sistema conserve recursos. El inconveniente es que la respuesta a la primera solicitud después de descargar la aplicación web es más larga, para que la aplicación web pueda cargar y comenzar a atender a las respuestas. En los planes de servicio Básico y Estándar, puede activar el valor **Siempre activado** para que la aplicación web se mantenga cargada en todo momento. Esto elimina mayores tiempos de carga después de la inactividad de la aplicación. Para cambiar la configuración de **Siempre activado**:

1. En Azure Portal, vaya a la aplicación web.
2. Seleccione **Configuración de la aplicación**.
3. Para **Siempre activado**, seleccione **Activado**.

## <a name="how-do-i-turn-on-failed-request-tracing"></a>¿Cómo se activa el seguimiento de solicitudes con error?

Para activar el seguimiento de solicitudes con error:

1. En Azure Portal, vaya a la aplicación web.
3. Seleccione **Toda la configuración** > **Registros de diagnóstico**.
4. En **Seguimiento de solicitudes con error**, seleccione **Activado**.
5. Seleccione **Guardar**.
6. En la hoja de aplicación web, seleccione **Herramientas**.
7. Seleccione **Visual Studio Online**.
8. Si esta opción no está **activada**, seleccione **Activado**.
9. Seleccione **Ir**.
10. Seleccione **Web.config**.
11. En system.webServer, agregue esta configuración (para capturar una dirección URL específica):

    ```xml
    <system.webServer>
    <tracing> <traceFailedRequests>
    <remove path="*api*" />
    <add path="*api*">
    <traceAreas>
    <add provider="ASP" verbosity="Verbose" />
    <add provider="ASPNET" areas="Infrastructure,Module,Page,AppServices" verbosity="Verbose" />
    <add provider="ISAPI Extension" verbosity="Verbose" />
    <add provider="WWW Server" areas="Authentication,Security,Filter,StaticFile,CGI,Compression, Cache,RequestNotifications,Module,FastCGI" verbosity="Verbose" />
    </traceAreas>
    <failureDefinitions statusCodes="200-999" />
    </add> </traceFailedRequests>
    </tracing>
    ```
12. Para solucionar problemas de rendimiento lento, agregue esta configuración (si la solicitud de captura está tardando más de 30 segundos):
    ```xml
    <system.webServer>
    <tracing> <traceFailedRequests>
    <remove path="*" />
    <add path="*">
    <traceAreas> <add provider="ASP" verbosity="Verbose" />
    <add provider="ASPNET" areas="Infrastructure,Module,Page,AppServices" verbosity="Verbose" />
    <add provider="ISAPI Extension" verbosity="Verbose" />
    <add provider="WWW Server" areas="Authentication,Security,Filter,StaticFile,CGI,Compression, Cache,RequestNotifications,Module,FastCGI" verbosity="Verbose" />
    </traceAreas>
    <failureDefinitions timeTaken="00:00:30" statusCodes="200-999" />
    </add> </traceFailedRequests>
    </tracing>
    ```
13. Para descargar los seguimientos de solicitudes con error, en el [portal](https://portal.azure.com), vaya al sitio web.
15. Seleccione **Herramientas** > **Kudu** > **Ir**.
18. En el menú, seleccione **Consola de depuración** > **CMD**.
19. Seleccione la carpeta **LogFiles** y, después, seleccione la carpeta con un nombre que comienza por **W3SVC**.
20. Para ver el archivo XML, seleccione el icono de lápiz.

## <a name="i-see-the-message-worker-process-requested-recycle-due-to-percent-memory-limit-how-do-i-address-this-issue"></a>Aparece un mensaje que indica que el proceso de trabajo ha solicitado el reciclado debido a un límite de 'porcentaje de memoria'. ¿Cómo se soluciona este problema?

La cantidad máxima de memoria disponible para un proceso de 32 bits (incluso en un sistema operativo de 64 bits) es de 2 GB. De forma predeterminada, el proceso de trabajo está establecido en 32 bits en App Service (por compatibilidad con aplicaciones web heredadas).

Considere la posibilidad de cambiar a procesos de 64 bits, por lo que podrá aprovechar la memoria adicional disponible en el rol de trabajo web. Esto desencadena un reinicio de la aplicación web, por lo que programe en consecuencia.

Tenga en cuenta que un entorno de 64 bits requiere un plan de servicio Básico o Estándar. Los planes Gratis y Compartido siempre se ejecutan en un entorno de 32 bits.

Para más información, consulte [Configuración de aplicaciones web en Azure App Service](web-sites-configure.md).

## <a name="why-does-my-request-time-out-after-230-seconds"></a>¿Por qué se agota el tiempo de espera de la solicitud después de 230 segundos?

Azure Load Balancer tiene un valor de tiempo de expiración de inactividad predeterminado de cuatro minutos. Por lo general, suele ser un límite de tiempo de respuesta razonable para una solicitud web. Si la aplicación web requiere un procesamiento en segundo plano, se recomienda el uso de Azure WebJobs. La aplicación web de Azure puede llamar a WebJobs y se recibirá una notificación cuando haya finalizado el procesamiento en segundo plano. Puede elegir entre varios métodos para usar WebJobs, incluidos las colas y los desencadenadores.

WebJobs está pensado para el procesamiento en segundo plano. Puede realizar tanto procesamiento en segundo plano como desee en un WebJob. Para más información, consulte [Ejecutar tareas en segundo plano con WebJobs](webjobs-create.md).

## <a name="aspnet-core-applications-that-are-hosted-in-app-service-sometimes-stop-responding-how-do-i-fix-this-issue"></a>Las aplicaciones de ASP.NET Core hospedadas en App Service a veces dejan de responder. ¿Cómo se corrige este problema?

Un problema conocido con una [versión Kestrel](https://github.com/aspnet/KestrelHttpServer/issues/1182) anterior podía provocar que una aplicación de ASP.NET Core 1.0 hospedada en App Service de vez en cuando dejase de responder. También podría ver el mensaje: "La aplicación CGI especificada encontró un error y el servidor terminó el proceso".

Este problema está corregido Kestrel versión 1.0.2. Esta versión se incluye en la actualización de ASP.NET Core 1.0.3. Para resolver este problema, asegúrese de que actualiza las dependencias de la aplicación para usar Kestrel versión 1.0.2. Como alternativa, puede usar una de las dos soluciones alternativas que se describen en la entrada de blog [ASP.NET Core 1.0 slow perf issues in App Service web apps](https://blogs.msdn.microsoft.com/waws/2016/12/11/asp-net-core-slow-perf-issues-on-azure-websites) (Problemas de rendimiento lento de ASP.NET Core 1.0 en aplicaciones web de App Service).


## <a name="i-cant-find-my-log-files-in-the-file-structure-of-my-web-app-how-can-i-find-them"></a>No se encuentran mis archivos de registro en la estructura de archivos de la aplicación web. ¿Cómo se pueden encontrar?

Si utiliza la característica de caché local de App Service, la estructura de carpetas de las carpetas de archivos de registro y datos para la instancia de App Service se verá afectada. Cuando se utiliza la memoria caché local, se crean las subcarpetas en las carpetas de datos y de archivos de registro del almacenamiento. Las subcarpetas usan el patrón de nomenclatura "identificador único" + marca de tiempo. Cada subcarpeta se corresponde con una instancia de máquina virtual en donde se ejecuta la aplicación web o se ha ejecutado.

Para determinar si está utilizando la memoria caché local, compruebe la pestaña **Configuración de la aplicación** de App Service. Si se utiliza la memoria caché local, la configuración de la aplicación `WEBSITE_LOCAL_CACHE_OPTION` se establece en `Always`.

Si no se está usando la memoria caché local y está experimentando este problema, envíe una solicitud de soporte técnico.

## <a name="i-see-the-message-an-attempt-was-made-to-access-a-socket-in-a-way-forbidden-by-its-access-permissions-how-do-i-resolve-this"></a>Aparece un mensaje que indica que se realizó un intento de acceder a un socket de una manera prohibida por los permisos de acceso. ¿Cómo se resuelve este problema?

Este error suele producirse si se agotan las conexiones TCP salientes en la instancia de máquina virtual. En App Service, los límites se aplican para el número máximo de conexiones salientes que puede realizar cada instancia de máquina virtual. Para más información, consulte la sección [Cross-VM numerical limits](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox#cross-vm-numerical-limits) (Límites numéricos entre máquinas virtuales).

Este error también puede producirse si intenta tener acceso a una dirección local desde la aplicación. Para más información, consulte la sección [Local address requests](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox#local-address-requests) (Peticiones de direcciones locales).

Para más información acerca de las conexiones salientes en la aplicación web, consulte la entrada de blog sobre [conexiones salientes a sitios web de Azure](https://www.freekpaans.nl/2015/08/starving-outgoing-connections-on-windows-azure-web-sites/).

## <a name="how-do-i-use-visual-studio-to-remote-debug-my-app-service-web-app"></a>¿Cómo usar Visual Studio para depurar de forma remota la aplicación web de App Service?

Para ver un tutorial detallado que muestra cómo depurar la aplicación web con Visual Studio, consulte la entrada de blog [Remote debug your App Service web app](https://blogs.msdn.microsoft.com/benjaminperkins/2016/09/22/remote-debug-your-azure-app-service-web-app/) (Depuración de forma remota de la aplicación web de App Service).
