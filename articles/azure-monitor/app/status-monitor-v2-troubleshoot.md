---
title: Soluciones de problemas conocidos de Azure Application Insights Agent | Microsoft Docs
description: Problemas conocidos de Application Insights Agent y ejemplos de soluciones. Supervise el rendimiento de los sitios web sin volver a implementarlos. Funciona con las aplicaciones web de ASP.NET hospedadas en local, en las máquinas virtuales o en Azure.
ms.topic: conceptual
author: TimothyMothra
ms.author: tilee
ms.date: 04/23/2019
ms.openlocfilehash: 07b5ae99b98c6c6bbf4dc903cd1b583eb655a825
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87041406"
---
# <a name="troubleshooting-application-insights-agent-formerly-named-status-monitor-v2"></a>Solución de problemas de Application Insights Agent (antes Monitor de estado v2)

Cuando se habilita la supervisión, podría experimentar problemas que impidan la recopilación de datos.
En este artículo se enumeran todos los problemas conocidos y se proporcionan ejemplos de solución de problemas.
Si encuentra un problema que no aparece aquí, puede ponerse en contacto con nosotros en [GitHub](https://github.com/Microsoft/ApplicationInsights-Home/issues).

## <a name="known-issues"></a>Problemas conocidos

### <a name="conflicting-dlls-in-an-apps-bin-directory"></a>Archivos DLL en conflicto en el directorio bin de una aplicación

Si cualquiera de estas DLL están presente en el directorio bin, la supervisión podría dar error:

- Microsoft.ApplicationInsights.dll
- Microsoft.AspNet.TelemetryCorrelation.dll
- System.Diagnostics.DiagnosticSource.dll

Algunos de estos archivos DLL se incluyen en las plantillas de aplicación predeterminadas de Visual Studio, incluso si la aplicación no las usa.
Puede usar las herramientas de solución de problemas para ver el comportamiento de los síntomas:

- PerfView:
    ```
    ThreadID="7,500" 
    ProcessorNumber="0" 
    msg="Found 'System.Diagnostics.DiagnosticSource, Version=4.0.2.1, Culture=neutral, PublicKeyToken=cc7b13ffcd2ddd51' assembly, skipping attaching redfield binaries" 
    ExtVer="2.8.13.5972" 
    SubscriptionId="" 
    AppName="" 
    FormattedMessage="Found 'System.Diagnostics.DiagnosticSource, Version=4.0.2.1, Culture=neutral, PublicKeyToken=cc7b13ffcd2ddd51' assembly, skipping attaching redfield binaries" 
    ```

- IISReset y carga de aplicación (sin telemetría). Investigue con Sysinternals (Handle.exe y ListDLLs.exe):
    ```
    .\handle64.exe -p w3wp | findstr /I "InstrumentationEngine AI. ApplicationInsights"
    E54: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll

    .\Listdlls64.exe w3wp | findstr /I "InstrumentationEngine AI ApplicationInsights"
    0x0000000009be0000  0x127000  C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\MicrosoftInstrumentationEngine_x64.dll
    0x0000000009b90000  0x4f000   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\Microsoft.ApplicationInsights.ExtensionsHost_x64.dll
    0x0000000004d20000  0xb2000   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\Microsoft.ApplicationInsights.Extensions.Base_x64.dll
    ```

### <a name="conflict-with-iis-shared-configuration"></a>Conflicto con la configuración compartida de IIS

Si tiene un clúster de servidores web, puede que esté usando una [configuración compartida](/iis/web-hosting/configuring-servers-in-the-windows-web-platform/shared-configuration_211).
HttpModule no pueden insertarse en esta configuración compartida.
Ejecute el comando Enable en cada servidor web para instalar el archivo DLL en la GAC de cada servidor.

Después de ejecutar el comando Enable, siga estos pasos:
1. Vaya al directorio de configuración compartida y busque el archivo applicationHost.config.
2. Agregue esta línea a la sección de módulos de la configuración:
    ```
    <modules>
        <!-- Registered global managed http module handler. The 'Microsoft.AppInsights.IIS.ManagedHttpModuleHelper.dll' must be installed in the GAC before this config is applied. -->
        <add name="ManagedHttpModuleHelper" type="Microsoft.AppInsights.IIS.ManagedHttpModuleHelper.ManagedHttpModuleHelper, Microsoft.AppInsights.IIS.ManagedHttpModuleHelper, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" preCondition="managedHandler,runtimeVersionv4.0" />
    </modules>
    ```

### <a name="iis-nested-applications"></a>Aplicaciones anidadas en IIS

En la versión 1.0 so se instrumentan aplicaciones anidadas en IIS.
[Aquí](https://github.com/microsoft/ApplicationInsights-Home/issues/369) puede ver el seguimiento de este problema.

### <a name="advanced-sdk-configuration-isnt-available"></a>La configuración avanzada del SDK no está disponible.

La configuración del SDK no se muestra al usuario final en la versión 1.0.
[Aquí](https://github.com/microsoft/ApplicationInsights-Home/issues/375) puede ver el seguimiento de este problema.

    
    
## <a name="troubleshooting"></a>Solución de problemas
    
### <a name="troubleshooting-powershell"></a>Solución de problemas de PowerShell

#### <a name="determine-which-modules-are-available"></a>Determine qué módulos están disponibles
Puede usar el comando `Get-Module -ListAvailable` para determinar qué módulos se instalan.

#### <a name="import-a-module-into-the-current-session"></a>Importe un módulo en la sesión actual
Si un módulo no se ha cargado en una sesión de PowerShell, puede cargarlo manualmente mediante el comando `Import-Module <path to psd1>`.


### <a name="troubleshooting-the-application-insights-agent-module"></a>Solución de problemas del módulo Application Insights Agent

#### <a name="list-the-commands-available-in-the-application-insights-agent-module"></a>Enumeración de los comandos disponibles en el módulo Application Insights Agent
Ejecute el comando `Get-Command -Module Az.ApplicationMonitor` para obtener los comandos disponibles:

```
CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Disable-ApplicationInsightsMonitoring              0.4.0      Az.ApplicationMonitor
Cmdlet          Disable-InstrumentationEngine                      0.4.0      Az.ApplicationMonitor
Cmdlet          Enable-ApplicationInsightsMonitoring               0.4.0      Az.ApplicationMonitor
Cmdlet          Enable-InstrumentationEngine                       0.4.0      Az.ApplicationMonitor
Cmdlet          Get-ApplicationInsightsMonitoringConfig            0.4.0      Az.ApplicationMonitor
Cmdlet          Get-ApplicationInsightsMonitoringStatus            0.4.0      Az.ApplicationMonitor
Cmdlet          Set-ApplicationInsightsMonitoringConfig            0.4.0      Az.ApplicationMonitor
Cmdlet          Start-ApplicationInsightsMonitoringTrace           0.4.0      Az.ApplicationMonitor
```

#### <a name="determine-the-current-version-of-the-application-insights-agent-module"></a>Determinación de la versión actual del módulo Application Insights Agent
Ejecute el comando `Get-ApplicationInsightsMonitoringStatus -PowerShellModule` para mostrar la siguiente información sobre el módulo:
   - Versión del módulo de PowerShell
   - Versión del SDK de Application Insights
   - Rutas de acceso de los archivos del módulo de PowerShell
    
Revise la [referencia de la API](status-monitor-v2-api-reference.md) para obtener una descripción detallada de cómo usar este cmdlet.


### <a name="troubleshooting-running-processes"></a>Solución de problemas de los procesos en ejecución

Puede inspeccionar los procesos en el equipo instrumentado para determinar si todos los archivos DLL se cargan.
Si funciona la supervisión, deben cargarse al menos 12 DLL.

Use el comando `Get-ApplicationInsightsMonitoringStatus -InspectProcess` para comprobar las DLL.

Revise la [referencia de la API](status-monitor-v2-api-reference.md) para obtener una descripción detallada de cómo usar este cmdlet.


### <a name="collect-etw-logs-by-using-perfview"></a>Recopilación de los registros ETW con PerfView

#### <a name="setup"></a>Configurar

1. Descargue PerfView.exe y PerfView64.exe desde [GitHub](https://github.com/Microsoft/perfview/releases).
2. Inicie PerfView64.exe.
3. Expanda **Opciones avanzadas**.
4. Desactive estas casillas de verificación:
    - **Zip**
    - **Combinar**
    - **.NET Symbol Collection** (Colección de símbolos de .NET)
5. Establezca estos **proveedores adicionales**: `61f6ca3b-4b5f-5602-fa60-759a2a2d1fbd,323adc25-e39b-5c87-8658-2c1af1a92dc5,925fa42b-9ef6-5fa7-10b8-56449d7a2040,f7d60e07-e910-5aca-bdd2-9de45b46c560,7c739bb9-7861-412e-ba50-bf30d95eae36,61f6ca3b-4b5f-5602-fa60-759a2a2d1fbd,323adc25-e39b-5c87-8658-2c1af1a92dc5,252e28f4-43f9-5771-197a-e8c7e750a984`


#### <a name="collecting-logs"></a>Recopilación de registros

1. En una consola de comandos con privilegios de administrador, ejecute el comando `iisreset /stop` para desactivar IIS y todas las aplicaciones web.
2. En PerfView, seleccione **Iniciar colección**.
3. En una consola de comandos con privilegios de administrador, ejecute el comando `iisreset /start` para iniciar IIS.
4. Intente ir a la aplicación.
5. Cuando la aplicación se cargue, vuelva a PerfView y seleccione **Detener colección**.



## <a name="next-steps"></a>Pasos siguientes

- Revise la [referencia de API](status-monitor-v2-overview.md#powershell-api-reference) para obtener información acerca de los parámetros que podría haber pasado por alto.
- Si encuentra un problema que no aparece aquí, puede ponerse en contacto con nosotros en [GitHub](https://github.com/Microsoft/ApplicationInsights-Home/issues).
