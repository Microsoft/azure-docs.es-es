---
title: Solución de problemas relacionados con Azure Application Insights Profiler
description: Este artículo muestra información y los pasos a seguir para la solución de problema para ayudar a los desarrolladores que tienen dificultades para habilitar o usar Application Insights Profiler.
ms.topic: conceptual
author: cweining
ms.author: cweining
ms.date: 08/06/2018
ms.reviewer: mbullwin
ms.openlocfilehash: aa9b186e74ed3b8fe5496afd5b21c54f50537d5f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87049782"
---
# <a name="troubleshoot-problems-enabling-or-viewing-application-insights-profiler"></a>Solución de problemas de activación o visualización de Application Insights Profiler

> [!CAUTION]
> Hay un error al ejecutar el generador de perfiles con aplicaciones de ASP.NET Core en Azure App Service. Tenemos una solución, pero tardará unas semanas en implementarse en todo el mundo. Puede solucionar el error agregando el SDK de Application Insights a la aplicación con las instrucciones que se indican [aquí](./asp-net-core.md#enable-application-insights-server-side-telemetry-visual-studio).

## <a name="general-troubleshooting"></a><a id="troubleshooting"></a>Solución general de problemas

### <a name="profiles-are-uploaded-only-if-there-are-requests-to-your-application-while-profiler-is-running"></a>Los perfiles solo se cargan si hay solicitudes en la aplicación mientras se ejecuta Profiler

Azure Application Insights Profiler recopila los datos de generación de perfiles durante dos minutos cada hora. También recopila datos cuando se selecciona el botón **Generar perfiles ahora**  en el panel **Configurar Application Insights Profiler**. Pero los datos de generación de perfiles solo se cargan cuando se pueden adjuntar a una solicitud que se ja realizado mientras se ejecuta Profiler. 

Profiler escribe mensajes de seguimiento y eventos personalizados en el recurso de Application Insights. Estos eventos se pueden usar para ver cómo se ejecuta Profiler. Si cree que Profiler debe ejecutar y capturar seguimientos, pero estos no se muestran en el panel **Rendimiento**, puede comprobar el funcionamiento de Profiler:

1. Busque mensajes de seguimiento y eventos personalizados que Profiler ha enviado a su recurso de Application Insights. Puede usar esta cadena de búsqueda para encontrar los datos pertinentes:

    ```
    stopprofiler OR startprofiler OR upload OR ServiceProfilerSample
    ```
    La siguiente imagen muestra dos ejemplos de búsquedas de dos recursos de inteligencia artificial: 
    
   * A la izquierda, la aplicación no recibe solicitudes mientras se ejecuta Profiler. El mensaje explica que la carga se ha cancelado debido a la falta de actividad. 

   * A la derecha, Profiler se ha iniciado y ha enviado eventos personalizados cuando ha detectado las solicitudes que se han producido mientras se estaba ejecutando Profiler. Si se muestra el evento personalizado ServiceProfilerSample, significa que Profiler ha adjuntado un seguimiento a una solicitud y que el seguimiento se puede ver en el panel **Rendimiento de Application Insights**.

     Si no se muestran datos de telemetría, Profiler no está en ejecución. Para solucionar problemas, consulte las secciones de solución de problemas del tipo de aplicación concreto más adelante en este artículo.  

     ![Buscar telemetría de Profiler][profiler-search-telemetry]

1. Si hubiera solicitudes durante la ejecución de Profiler, asegúrese de que la parte de la aplicación que tiene habilitado Profiler es la que controla las solicitudes. Aunque las aplicaciones a veces constan de varios componentes, Profiler está habilitado solo para algunos de los componentes. El panel **Configurar Application Insights Profiler** muestra los componentes que han cargado seguimientos.

### <a name="other-things-to-check"></a>Otros aspectos que hay que comprobar
* Asegúrese de que la aplicación se ejecuta en .NET Framework 4.6.
* Si la aplicación web es una aplicación de ASP.NET Core, debe ejecutar al menos ASP.NET Core 2.0.
* Si la antigüedad de los datos que intenta ver supera el par de semanas, pruebe a limitar el filtro de tiempo y inténtelo de nuevo. Los seguimientos se eliminan pasados siete días.
* Asegúrese de que no haya servidores proxy ni un firewall que bloqueen el acceso a https://gateway.azureserviceprofiler.net.
* No se admite Profiler en planes de App Service gratuitos o compartidos. Si usa uno de estos planes, intente escalar verticalmente a uno de los planes básicos y Profiler debería empezar a funcionar.

### <a name="double-counting-in-parallel-threads"></a><a id="double-counting"></a>Doble recuento de subprocesos paralelos

En algunos casos, la métrica de tiempo total del visor de la pila es mayor que la duración de la solicitud.

Esta situación puede darse cuando hay dos o más subprocesos asociados a una solicitud y funcionan en paralelo. En ese caso, el tiempo de subproceso total es superior al tiempo transcurrido. Es posible que uno de los subprocesos esté a la espera de que el otro finalice. El visor intenta detectar esta situación y omite la espera sin interés. Al hacerlo, prefiere mostrar demasiada información antes que omitir lo que podría ser información crítica.

Cuando vea subprocesos en paralelo en sus seguimientos, determine cuáles de ellos están en espera, con el fin de que pueda determinar cuál es la ruta crítica de la solicitud. Normalmente, el subproceso que pasa rápidamente a un estado de espera tan solo espera a los restantes subprocesos. Concéntrese en los demás subprocesos e ignore el tiempo de los subprocesos en espera.

### <a name="error-report-in-the-profile-viewer"></a>Informe de errores del visor de generación de perfiles
Envíe una incidencia de soporte técnico desde el portal. Asegúrese de incluir el identificador de correlación del mensaje de error.

## <a name="troubleshoot-profiler-on-azure-app-service"></a>Solución de los problemas de Profiler en Azure App Service
Para que Profiler funcione correctamente:
* El plan de servicio de aplicación web tiene que ser de nivel Básico o superior.
* La aplicación web debe tener Application Insights habilitado.
* La aplicación web debe tener las siguientes opciones:

    |Configuración de aplicación    | Value    |
    |---------------|----------|
    |APPINSIGHTS_INSTRUMENTATIONKEY         | iKey para el recurso de Application Insights    |
    |APPINSIGHTS_PROFILERFEATURE_VERSION | 1.0.0 |
    |DiagnosticServices_EXTENSION_VERSION | ~3 |


* El WebJob **ApplicationInsightsProfiler3** debe estar en ejecución. Para comprobarlo:
   1. Vaya a [Kudu](/archive/blogs/cdndevs/the-kudu-debug-console-azure-websites-best-kept-secret).
   1. En el menú **Tools** (Herramientas), seleccione **WebJobs Dashboard** (Panel de WebJobs).  
      Se abre el panel **WebJobs**. 
   
      ![profiler-webjob]   
   
   1. Para ver los detalles del webjob, incluido el registro, seleccione el vínculo **ApplicationInsightsProfiler3**.  
     Se abre el panel **Continuous WebJob Details** (Detalles de WebJobs continuos).

      ![profiler-webjob-log]

Si no puede averiguar el motivo por el que Profiler no funciona, puede descargar el registro y enviarlo a nuestro equipo para que este pueda ayudarle, serviceprofilerhelp@microsoft.com. 
    
### <a name="manual-installation"></a>Instalación manual

Cuando se configura Profiler, se realizan las siguientes actualizaciones en la configuración de la aplicación web. Si su entorno lo requiere, las actualizaciones se pueden aplicar manualmente. Un ejemplo podría ser la ejecución de su aplicación en un entorno de Web Apps para PowerApps. Para aplicar las actualizaciones manualmente:

1. En el panel de **control de la aplicación web**, abra **Configuración**.

1. Establezca la **versión de .NET Framework** en **v4.6**.

1. Establezca **Siempre activado** en **Activado**.
1. Cree esta configuración de aplicación:

    |Configuración de aplicación    | Value    |
    |---------------|----------|
    |APPINSIGHTS_INSTRUMENTATIONKEY         | iKey para el recurso de Application Insights    |
    |APPINSIGHTS_PROFILERFEATURE_VERSION | 1.0.0 |
    |DiagnosticServices_EXTENSION_VERSION | ~3 |

### <a name="too-many-active-profiling-sessions"></a>Hay demasiadas sesiones de generación de perfiles activas

Actualmente, puede habilitar Profiler en hasta cuatro aplicaciones web de Azure y ranuras de implementación que se ejecuten en el mismo plan de servicio. Si tiene más aplicaciones en ejecución en un plan de servicio de una aplicación, Profiler podría iniciar la excepción *Microsoft.ServiceProfiler.Exceptions.TooManyETWSessionException*. Profiler se ejecuta por separado en cada aplicación web e intenta iniciar una sesión de ETW (Seguimiento de eventos para Windows ) en cada una de ellas. Sin embargo, hay un número máximo de sesiones de ETW que pueden estar activas al mismo tiempo. Si el WebJob de Profiler notifica que hay demasiadas sesiones de generación de perfiles activas, mueva algunas aplicaciones web a otro plan de servicio.

### <a name="deployment-error-directory-not-empty-dhomesitewwwrootapp_datajobs"></a>Error de implementación: El directorio no está vacío "D:\\home\\site\\wwwroot\\App_Data\\jobs"

Si va a volver a implementar la aplicación web en un recurso de Web Apps con Profiler habilitado, puede que aparezca un mensaje similar al siguiente:

*El directorio no está vacío "D:\\home\\site\\wwwroot\\App_Data\\jobs"*

Este error se produce si Web Deploy se ejecuta desde scripts o desde la canalización de implementación de Azure DevOps. La solución consiste en agregar los siguientes parámetros de implementación adicionales a la tarea de Web Deploy:

```
-skip:Directory='.*\\App_Data\\jobs\\continuous\\ApplicationInsightsProfiler.*' -skip:skipAction=Delete,objectname='dirPath',absolutepath='.*\\App_Data\\jobs\\continuous$' -skip:skipAction=Delete,objectname='dirPath',absolutepath='.*\\App_Data\\jobs$'  -skip:skipAction=Delete,objectname='dirPath',absolutepath='.*\\App_Data$'
```

Estos parámetros eliminan la carpeta que usa Application Insights Profiler y desbloquea el proceso de reeimplementación. No afectan a la instancia de Profiler actualmente en ejecución.

### <a name="how-do-i-determine-whether-application-insights-profiler-is-running"></a>¿Cómo se puede determinar si Application Insights Profiler se está ejecutando?

Profiler se ejecuta como un WebJob continuo en la aplicación web. Puede abrir el recurso de aplicación web en [Azure Portal](https://portal.azure.com). En el panel **WebJobs**, compruebe el estado de **ApplicationInsightsProfiler**. Si no se está ejecutando, abra **Registros** para obtener más información.

## <a name="troubleshoot-vms-and-cloud-services"></a>Solución de problemas de servicios en la nube y máquinas virtuales

>**Se ha corregido el error en el generador de perfiles que se incluye con WAD para Cloud Services.** La versión más reciente de WAD (1.12.2.0) para Cloud Services funciona con todas las versiones recientes del SDK de App Insights. Los hosts de Cloud Services actualizarán WAD automáticamente, pero no es algo inmediato. Para forzar una actualización, puede volver a implementar su servicio o reiniciar el nodo.

Para ver si Azure Diagnostics ha configurado Profiler correctamente, realice las tres acciones siguientes: 
1. En primer lugar, tiene que comprobar si el contenido de la configuración de Azure Diagnostics que se ha implementado es el que esperaba. 

1. En segundo lugar, asegúrese de que Azure Diagnostics pasa el valor de iKey adecuado en la línea de comandos de Profiler. 

1. En tercer lugar, puede compruebe en el archivo de registro de Profiler si este se ha ejecutado, pero ha devuelto error. 

Para comprobar la configuración que se usó para configurar Azure Diagnostics:

1. Inicie sesión en la máquina virtual (VM) y abra el archivo de registro que se encuentra en esta ubicación. La versión del complemento puede ser más reciente en la máquina.
    
    Para máquinas virtuales:
    ```
    c:\WindowsAzure\logs\Plugins\Microsoft.Azure.Diagnostics.PaaSDiagnostics\1.11.3.12\DiagnosticsPlugin.log
    ```
    
    Para Cloud Services:
    ```
    c:\logs\Plugins\Microsoft.Azure.Diagnostics.PaaSDiagnostics\1.11.3.12\DiagnosticsPlugin.log  
    ```

1. En dicho archivo puede buscar la cadena **WadCfg** para saber qué valores se han pasado a la máquina virtual para configurar Azure Diagnostics. Puede comprobar si el valor de iKey que ha usado el receptor de Profiler es correcto.

1. Compruebe la línea de comandos que se usa para iniciar Profiler. Los argumentos que se utilizan para iniciar Profiler están en el siguiente archivo. (La unidad podría ser la c: o la d: y el directorio puede estar oculto).

    Para máquinas virtuales:
    ```
    C:\ProgramData\ApplicationInsightsProfiler\config.json
    ```
    
    Para Cloud Services:
    ```
    D:\ProgramData\ApplicationInsightsProfiler\config.json
    ```

1. Asegúrese de que el valor de iKey en la línea de comandos de Profiler es correcto. 

1. Use la ruta de acceso que se encuentra en el archivo *config.json* anterior para consultar el archivo de registro de Profiler, denominado **BootstrapN.log**. Muestra la información de depuración que indica la configuración que usa Profiler. También muestra el estado y los mensajes de error de Profiler.  

    Para máquinas virtuales, el archivo suele estar aquí:
    ```
    C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\1.17.0.6\ApplicationInsightsProfiler
    ```

    Para Cloud Services:
    ```
    C:\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\1.17.0.6\ApplicationInsightsProfiler
    ```

    Si Profiler se está ejecutando mientras la aplicación recibe solicitudes, se mostrará el siguiente mensaje: *Activity detected from iKey* (Se ha detectado actividad en la clave de instrumentación). 

    Cuando se cargue el seguimiento, se muestra el mensaje siguiente: *Start to upload trace* (Inicio de carga del seguimiento). 


## <a name="edit-network-proxy-or-firewall-rules"></a>Edición de reglas de firewall o proxy de red

Si la aplicación se conecta a Internet a través de un proxy o un firewall, es posible que tenga que editar las reglas para permitir que la aplicación se comunique con el servicio Application Insights Profiler. Las direcciones IP que se usan en Application Insights Profiler se incluyen en la etiqueta de servicio de Azure Monitor.


[profiler-search-telemetry]:./media/profiler-troubleshooting/Profiler-Search-Telemetry.png
[profiler-webjob]:./media/profiler-troubleshooting/Profiler-webjob.png
[profiler-webjob-log]:./media/profiler-troubleshooting/Profiler-webjob-log.png
