---
title: Supervisión de una aplicación web de ASP.NET con Azure Application Insights | Microsoft Docs
description: Supervise el rendimiento de un sitio web sin volver a implementarlo. Funciona con las aplicaciones web de ASP.NET hospedadas en local o en máquinas virtuales.
ms.topic: conceptual
ms.date: 08/26/2019
ms.custom: devx-track-dotnet
ms.openlocfilehash: 603cbde20ae6e8d19e4ad3ae290bbfc925483595
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88923830"
---
# <a name="instrument-web-apps-at-runtime-with-application-insights-codeless-attach"></a>Instrumentación de aplicaciones web en tiempo de ejecución con Adjuntar sin código de Application Insights

> [!IMPORTANT]
> Ya no se recomienda el uso del Monitor de estado y, **a partir del 1 de junio de 2021**, no se admitirá esta versión del Monitor de estado. Se ha reemplazado por Azure Application Insights Agent (anteriormente denominado Monitor de estado V2). Consulte nuestra documentación sobre las [implementaciones de servidores locales](./status-monitor-v2-overview.md) o las [implementaciones de máquinas virtuales y conjuntos de escalado de máquinas virtuales de Azure](./azure-vm-vmss-apps.md).

Puede instrumentar una aplicación web activa con Azure Application Insights sin tener que modificar ni volver a implementar el código. Necesita una suscripción a [Microsoft Azure](https://azure.com) .

Monitor de estado se usa para instrumentar una aplicación .NET hospedada en IIS, ya sea en el entorno local o en una máquina virtual.

- Si la aplicación se implementa en una máquina virtual de Azure o en un conjunto de escalado de máquinas virtuales de Azure, siga [estas instrucciones](azure-vm-vmss-apps.md).
- Si la aplicación se implementa en Azure App Services, siga [estas instrucciones](azure-web-apps.md).
- Si la aplicación se implementa en una máquina virtual de Azure, puede activar la supervisión de Application Insights en el panel de control de Azure.
- (También hay artículos independientes sobre cómo instrumentar [Azure Cloud Services](./cloudservices.md)).


![Captura de pantalla de gráficos de información general de App Insights con información sobre las solicitudes con error, el tiempo de respuesta del servidor y las solicitudes del servidor](./media/monitor-performance-live-website-now/overview-graphs.png)

Puede optar entre dos vías de aplicar Application Insights a sus aplicaciones web .NET:

* **Tiempo de compilación:** [Agregue el SDK de Application Insights][greenbrown] al código de la aplicación web.
* **Tiempo de ejecución:** instrumenta la aplicación web en el servidor, como se describe a continuación, sin volver a generar e implementar el código.

> [!NOTE]
> Si usa la instrumentación en tiempo de compilación, la instrumentación en tiempo de ejecución no funcionará ni aunque esté activada.

A continuación hay un resumen de lo que se obtiene por cada vía:

|  | Tiempo de compilación | Tiempo de ejecución |
| --- | --- | --- |
| **Solicitudes y excepciones** |Sí |Sí |
| **[Excepciones más detalladas](./asp-net-exceptions.md)** | |Sí |
| **[Diagnósticos de dependencia](./asp-net-dependencies.md)** |En .NET 4.6 +, pero con menos detalle |Sí, detalles completos: códigos de resultado, texto de comandos SQL, verbo HTTP|
| **[Contadores de rendimiento del sistema](./performance-counters.md)** |Sí |Sí |
| **[API para la telemetría personalizada][api]** |Sí |No |
| **[Integración del registro de seguimiento](./asp-net-trace-logs.md)** |Sí |No |
| **[Vista de página y datos de usuario](./javascript.md)** |Sí |No |
| **Es necesario volver a compilar el código** |Sí | No |



## <a name="monitor-a-live-iis-web-app"></a>Supervisión de una aplicación web de IIS activa

Si la aplicación se hospeda en un servidor IIS, habilite Application Insights con el Monitor de estado.

1. En el servidor web IIS, inicie sesión con las credenciales de administrador.
2. Si el Monitor de estado de Application Insights todavía no está instalado, [descargue y ejecute el instalador](#download).
3. En el Monitor de estado, seleccione la aplicación web instalada o el sitio web que desea supervisar. Inicie sesión con sus credenciales de Azure.

    Configure el recurso donde desee ver los resultados en el portal de Application Insights. (Normalmente, es mejor crear un nuevo recurso. Seleccione un recurso existente si ya tiene [pruebas web][availability] o la [supervisión de cliente][client] para esta aplicación). 

    ![Elija una aplicación y un recurso.](./media/monitor-performance-live-website-now/appinsights-036-configAIC.png)

4. Reinicie IIS.

    ![Seleccione Reiniciar en la parte superior del cuadro de diálogo.](./media/monitor-performance-live-website-now/appinsights-036-restart.png)

    El servicio web se interrumpe durante un breve período.

## <a name="customize-monitoring-options"></a>Personalización de las opciones de supervisión

Si se habilita Application Insights, se agregan archivos DLL y ApplicationInsights.config a la aplicación web. También puede [editar el archivo .config](./configuration-with-applicationinsights-config.md) para cambiar algunas de las opciones.

## <a name="when-you-re-publish-your-app-re-enable-application-insights"></a>Al volver a publicar la aplicación, vuelva a habilitar Application Insights

Antes de volver a publicar la aplicación, considere la posibilidad de [agregar Application Insights al código de Visual Studio][greenbrown]. Obtendrá una telemetría más detallada y la capacidad para escribir telemetría personalizada.

Si desea volver a publicar sin agregar Application Insights al código, tenga en cuenta que el proceso de implementación puede eliminar los archivos DLL y ApplicationInsights.config desde el sitio web publicado. Por lo tanto:

1. Si edita el archivo ApplicationInsights.config, realice una copia del mismo antes de volver a publicar la aplicación.
2. Vuelva a publicar la aplicación.
3. Vuelva a habilitar la supervisión de Application Insights. (Use el método adecuado: ya sea el panel de control de la aplicación web de Azure o el Monitor de estado en un host IIS).
4. Restablezca las modificaciones realizadas en el archivo .config.


## <a name="troubleshooting"></a><a name="troubleshoot"></a>Solución de problemas

### <a name="confirm-a-valid-installation"></a>Confirmación de una instalación válida 

Estos son algunos de los pasos que puede seguir para confirmar que la instalación se completó correctamente.

- Confirme que el archivo applicationInsights.config esté en el directorio de la aplicación de destino y que incluya la clave de instrumentación.

- Si sospecha que faltan datos, puede ejecutar una consulta en [Analytics](../log-query/get-started-portal.md) para mostrar todos los roles de la nube que actualmente envían datos de telemetría.
  ```Kusto
  union * | summarize count() by cloud_RoleName, cloud_RoleInstance
  ```

- Si tiene que confirmar que Application Insights se ha adjuntado correctamente, puede ejecutar [Sysinternals Handle](/sysinternals/downloads/handle) en una ventana de comando para confirmar que IIS cargó applicationinsights.dll.

  ```console
  handle.exe /p w3wp.exe
  ```


### <a name="cant-connect-no-telemetry"></a>¿No se puede conectar? ¿No hay telemetría?

* Abra los [puertos de salida necesarios](./ip-addresses.md#outgoing-ports) en el firewall del servidor para que el Monitor de estado funcione.

### <a name="unable-to-login"></a>Error de inicio de sesión

Si el Monitor de estado no puede iniciar sesión, haga una instalación de línea de comandos. El Monitor de estado intenta iniciar sesión para recopilar la clave de instrumentación, pero la puede proporcionar manualmente con el comando:

```powershell
Import-Module 'C:\Program Files\Microsoft Application Insights\Status Monitor\PowerShell\Microsoft.Diagnostics.Agent.StatusMonitor.PowerShell.dll'
Start-ApplicationInsightsMonitoring -Name appName -InstrumentationKey 00000000-000-000-000-0000000
```

### <a name="could-not-load-file-or-assembly-systemdiagnosticsdiagnosticsource"></a>No se pudo cargar el archivo ni el ensamblado "System.Diagnostics.DiagnosticSource"

Puede recibir este error después de habilitar Application Insights. Esto se debe a que el instalador reemplaza este archivo dll en el directorio bin.
Para corregir y actualizar web.config:

```xml
<dependentAssembly>
    <assemblyIdentity name="System.Diagnostics.DiagnosticSource" publicKeyToken="cc7b13ffcd2ddd51"/>
    <bindingRedirect oldVersion="0.0.0.0-4.*.*.*" newVersion="4.0.2.1"/>
</dependentAssembly>
```

Hacemos un seguimiento de este problema [aquí](https://github.com/Microsoft/ApplicationInsights-Home/issues/301).


### <a name="application-diagnostic-messages"></a>Mensajes de diagnósticos de la aplicación

* Abrir el Monitor de estado y seleccione la aplicación en el panel izquierdo. Compruebe si hay algún mensaje de diagnóstico para esta aplicación en la sección "Notificaciones de configuración":

  ![Abra la hoja Rendimiento para ver la solicitud, el tiempo de respuesta, la dependencia y otros datos.](./media/monitor-performance-live-website-now/appinsights-status-monitor-diagnostics-message.png)
  
### <a name="detailed-logs"></a>Registros detallados

* De manera predeterminada, el Monitor de estado generará los registros de diagnóstico en: `C:\Program Files\Microsoft Application Insights\Status Monitor\diagnostics.log`

* Para generar registros detallados, modifique el archivo de configuración: `C:\Program Files\Microsoft Application Insights\Status Monitor\Microsoft.Diagnostics.Agent.StatusMonitor.exe.config` y agregue `<add key="TraceLevel" value="All" />` en `appsettings`.
Luego, reinicie el monitor de estado.

* Puesto que Monitor de estado es una aplicación .NET, también es posible habilitar [seguimiento de .net mediante la adición de los diagnósticos adecuados al archivo de configuración](/dotnet/framework/configure-apps/file-schema/trace-debug/system-diagnostics-element). Por ejemplo, en algunos escenarios puede resultar útil ver lo que sucede en el nivel de red mediante la [configuración del seguimiento de red](/dotnet/framework/network-programming/how-to-configure-network-tracing).

### <a name="insufficient-permissions"></a>Permisos insuficientes
  
* En el servidor, si ve en un mensaje acerca de "permisos insuficientes", intente lo siguiente:
  * En el Administrador de IIS, seleccione el grupo de aplicaciones, abra **Configuración avanzada** y en **Modelo de proceso**, anote la identidad.
  * En el panel de control de administración del equipo, agregue esta identidad al grupo Usuarios del monitor de sistema.

### <a name="conflict-with-systems-center-operations-manager"></a>Conflictos con Systems Center Operations Manager

* Si tiene instalado MMA/SCOM (Systems Center Operation Manager) en el servidor, algunas versiones pueden entrar en conflicto. Desinstale SCOM y el Monitor de estado y vuelva a instalar las versiones más recientes.

### <a name="failed-or-incomplete-installation"></a>Instalación errónea o incompleta

Si se produce un error del Monitor de estado durante una instalación, se podría generar una instalación incompleta de la que el Monitor de estado no se pueda recuperar. Para esto se necesitará un restablecimiento manual.

Elimine cualquiera de estos archivos encontrados en el directorio de la aplicación:
- Cualquier archivo DLL en el directorio bin que comience con "Microsoft.AI." o "Microsoft.ApplicationInsights.".
- Este archivo DLL en el directorio bin: "Microsoft.Web.Infrastructure.dll"
- Este archivo DLL en el directorio bin: "System.Diagnostics.DiagnosticSource.dll"
- En el directorio de la aplicación, quite "App_Data\packages"
- En el directorio de la aplicación, quite "applicationinsights.config"


### <a name="additional-troubleshooting"></a>Más soluciones de problemas

* Consulte Más [soluciones de problemas][qna].

## <a name="system-requirements"></a>Requisitos del sistema
Compatibilidad de sistema operativo para el Monitor de estado de Application Insights en servidor:

* Windows Server 2008
* Windows Server 2008 R2
* Windows Server 2012
* Windows Server 2012 R2
* Windows Server 2016

con .NET Framework 4.5 y la versión más reciente de SP (Monitor de estado se compila en esta versión de framework).

En el lado cliente: Windows 7, 8, 8.1 y 10, nuevamente con .NET Framework 4.5

La compatibilidad con IIS es: IIS 7, 7.5, 8, 8.5 (se requiere IIS)

## <a name="automation-with-powershell"></a>Automatización con PowerShell
Puede iniciar y detener la supervisión mediante PowerShell en el servidor IIS.

En primer lugar, importe el módulo de Application Insights:

```powershell
Import-Module 'C:\Program Files\Microsoft Application Insights\Status Monitor\PowerShell\Microsoft.Diagnostics.Agent.StatusMonitor.PowerShell.dll'
```

Encuentre las aplicaciones en supervisión:

`Get-ApplicationInsightsMonitoringStatus [-Name appName]`

* `-Name` (Opcional) Nombre de una aplicación web.
* Muestra el estado de supervisión de Application Insights para cada aplicación web (u otra aplicación con nombre) en este servidor IIS.
* Devuelve `ApplicationInsightsApplication` para cada aplicación:

  * `SdkState==EnabledAfterDeployment`: la aplicación se está supervisando y se ha instrumentado en tiempo de ejecución, ya sea con la herramienta Monitor de estado o con `Start-ApplicationInsightsMonitoring`.
  * `SdkState==Disabled`: la aplicación no se ha instrumentado para Application Insights. Nunca se ha instrumentado, o bien se deshabilitó la supervisión en tiempo de ejecución con la herramienta Monitor de estado o con `Stop-ApplicationInsightsMonitoring`.
  * `SdkState==EnabledByCodeInstrumentation`: se ha instrumentado la aplicación agregando el SDK al código fuente. El SDK no se puede actualizar ni detener.
  * `SdkVersion` muestra la versión en uso para la supervisión de esta aplicación.
  * `LatestAvailableSdkVersion`muestra la versión disponible en la galería de NuGet. Para actualizar la aplicación a esta versión, use `Update-ApplicationInsightsMonitoring`.

`Start-ApplicationInsightsMonitoring -Name appName -InstrumentationKey 00000000-000-000-000-0000000`

* `-Name` El nombre de la aplicación en IIS
* `-InstrumentationKey` El valor ikey del recurso de Application Insights donde quiere que se muestren los resultados.
* Este cmdlet solo afecta a las aplicaciones que no se han instrumentado, es decir, aquellas cuyo SdkState == NotInstrumented.

    El cmdlet no afecta a una aplicación que ya se ha instrumentado. No importa que se instrumentara en tiempo de compilación mediante la adición del SDK al código, o en tiempo de ejecución mediante un uso anterior de este cmdlet.

    La versión del SDK utilizada para instrumentar la aplicación es la versión que se ha descargado más recientemente en este servidor.

    Para descargar la versión más reciente, use Update-ApplicationInsightsVersion.
* Si se descarga correctamente, devuelve `ApplicationInsightsApplication` . Si se produce un error, inicia un seguimiento a stderr.

   ```output
   Name                      : Default Web Site/WebApp1
   InstrumentationKey        : 00000000-0000-0000-0000-000000000000
   ProfilerState             : ApplicationInsights
   SdkState                  : EnabledAfterDeployment
   SdkVersion                : 1.2.1
   LatestAvailableSdkVersion : 1.2.3
   ```

`Stop-ApplicationInsightsMonitoring [-Name appName | -All]`

* `-Name` El nombre de una aplicación en IIS
* `-All` Detiene la supervisión de todas las aplicaciones en este servidor IIS con `SdkState==EnabledAfterDeployment`
* Detiene la supervisión de las aplicaciones especificadas y quita la instrumentación. Solo funciona para las aplicaciones que se han instrumentado en tiempo de ejecución con la herramienta Monitor de estado o Start-ApplicationInsightsApplication. (`SdkState==EnabledAfterDeployment`)
* Devuelve ApplicationInsightsApplication.

`Update-ApplicationInsightsMonitoring -Name appName [-InstrumentationKey "0000000-0000-000-000-0000"`]

* `-Name`: el nombre de la aplicación web en IIS.
* `-InstrumentationKey` (Opcional). Utilice esto para cambiar el recurso al que se envía la telemetría de la aplicación.
* Este cmdlet:
  * Permite actualizar la aplicación con nombre a la versión del SDK descargada más recientemente en este equipo. (Solo funciona si `SdkState==EnabledAfterDeployment`)
  * Si proporciona una clave de instrumentación, se vuelve a configurar la aplicación con nombre para enviar los datos de telemetría al recurso con esa clave. (Funciona si `SdkState != Disabled`)

`Update-ApplicationInsightsVersion`

* Permite descargar el SDK más reciente de Application Insights en el servidor.

## <a name="questions-about-status-monitor"></a><a name="questions"></a>Preguntas acerca del Monitor de estado

### <a name="what-is-status-monitor"></a>¿Qué es el Monitor de estado?

Una aplicación de escritorio que se instala en el servidor web de IIS. Le ayuda a instrumentar y configurar aplicaciones web. 

### <a name="when-do-i-use-status-monitor"></a>¿Cuándo puedo usar el Monitor de estado?

* Para instrumentar cualquier aplicación web que se ejecute en el servidor IIS, incluso si ya se está en ejecución.
* Para habilitar la telemetría adicional para las aplicaciones web que se han [compilado con el SDK de Application Insights](./asp-net.md) en el momento de la compilación. 

### <a name="can-i-close-it-after-it-runs"></a>¿Puedo cerrarlo después de ejecutarlo?

Sí. Una vez que se hayan instrumentado los sitios web seleccionados, puede cerrarlo.

No recopila telemetría por sí mismo. Simplemente configura las aplicaciones web y establece algunos permisos.

### <a name="what-does-status-monitor-do"></a>¿Qué hace el Monitor de estado?

Cuando selecciona una aplicación web para instrumentarla con el Monitor de estado:

* Descarga y coloca los ensamblados de Application Insights y el archivo ApplicationInsights.config en la carpeta de archivos binarios de la aplicación web.
* Habilita la generación de perfiles de CLR para recopilar llamadas de dependencia.

### <a name="what-version-of-application-insights-sdk-does-status-monitor-install"></a>¿Qué versión del SDK de Application Insights realiza la instalación de Monitor de estado?

A partir de ahora, Monitor de estado solo puede instalar las versiones 2.3 o 2.4 del SDK de Application Insights. 

La versión 2.4 del SDK de Application Insights es la [última compatible con .NET 4.0](https://github.com/microsoft/ApplicationInsights-dotnet/releases/tag/v2.5.0-beta1), cuyo [final de vida útil fue en enero de 2016](https://devblogs.microsoft.com/dotnet/support-ending-for-the-net-framework-4-4-5-and-4-5-1/). Por lo tanto, a partir de ahora se puede usar Monitor de estado para instrumentar una aplicación .NET 4.0. 

### <a name="do-i-need-to-run-status-monitor-whenever-i-update-the-app"></a>¿Tengo que actualizar el Monitor de estado siempre que actualice la aplicación?

No si reimplementa de forma incremental. 

Si selecciona la opción "eliminar archivos existentes" en el proceso de publicación, necesitará volver a ejecutar el Monitor de estado para configurar Application Insights.

### <a name="what-telemetry-is-collected"></a>¿Qué telemetría se recopila?

En el caso de las aplicaciones que instrumenta solo en tiempo de ejecución mediante el Monitor de estado:

* Solicitudes HTTP
* Llamadas a dependencias
* Excepciones
* Contadores de rendimiento

En el caso de las aplicaciones ya instrumentadas en el momento de la compilación:

 * Contadores de proceso.
 * Llamadas de dependencia (.NET 4.5); valores devueltos en las llamadas de dependencia (.NET 4.6).
 * Valores de seguimiento de la pila de excepción.

[Más información](https://apmtips.com/posts/2016-11-18-how-application-insights-status-monitor-not-monitors-dependencies/)

## <a name="video"></a>Vídeo

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/100/player]

## <a name="download-status-monitor"></a><a name="download"></a>Descarga del Monitor de estado

- Use el nuevo [Módulo de PowerShell](./status-monitor-v2-overview.md)
- Descargue y ejecute el [instalador del Monitor de estado](https://go.microsoft.com/fwlink/?LinkId=506648)
- O ejecute el [Instalador de plataforma web](https://www.microsoft.com/web/downloads/platform.aspx) y busque ahí el Monitor de estado de Application Insights.

## <a name="next-steps"></a><a name="next"></a>Pasos siguientes

Vea la telemetría:

* [Explore las métricas](../platform/metrics-charts.md) para supervisar el rendimiento y uso.
* [Busque en los eventos y los registros][diagnostic] para diagnosticar problemas.
* [Análisis](../log-query/log-query-overview.md) para más consultas avanzadas

Agregue más telemetría:

* [Cree pruebas web][availability] para asegurarse de que el sitio permanece activo.
* [Agregue telemetría de cliente web][usage] para ver las excepciones de código de la página web y para que le permitan insertar llamadas de seguimiento.
* [Agregue el SDK de Application Insights al código][greenbrown] para que pueda insertar llamadas de seguimiento y registro.

<!--Link references-->

[api]: ./api-custom-events-metrics.md
[availability]: monitor-web-app-availability.md
[client]: ./javascript.md
[diagnostic]: ./diagnostic-search.md
[greenbrown]: ./asp-net.md
[qna]: ../faq.md
[roles]: ./resources-roles-access-control.md
[usage]: ./javascript.md

