---
title: Referencia de la API del agente de Azure Application Insights para .NET
description: Referencia de la API de Application Insights Agent. Supervise el rendimiento de los sitios web sin volver a implementarlos. Funciona con las aplicaciones web de ASP.NET hospedadas en local, en las máquinas virtuales o en Azure.
ms.topic: conceptual
author: TimothyMothra
ms.author: tilee
ms.date: 04/23/2019
ms.openlocfilehash: 2f814c54aeca8a337f786beb8da1b98accbeef7e
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87319004"
---
# <a name="azure-monitor-application-insights-agent-api-reference"></a>Referencia de la API del agente de Application Insights de Azure Monitor

Este artículo describe un cmdlet que forma parte del [módulo Az.ApplicationMonitor de PowerShell](https://www.powershellgallery.com/packages/Az.ApplicationMonitor/).

> [!NOTE] 
> - Para empezar, necesita una clave de instrumentación. Para más información, consulte [Crear un recurso](create-new-resource.md#copy-the-instrumentation-key).
> - Este cmdlet requiere que revise y acepte la licencia, y la declaración de privacidad.

> [!IMPORTANT] 
> Este cmdlet requiere una sesión de PowerShell con permisos de administrador y una directiva de ejecución elevada. Para más información, consulte [Ejecute PowerShell como administrador con una directiva de ejecución con privilegios elevados](status-monitor-v2-detailed-instructions.md#run-powershell-as-admin-with-an-elevated-execution-policy).
> - Este cmdlet requiere que revise y acepte la licencia, y la declaración de privacidad.
> - El motor de instrumentación produce más sobrecarga y está desactivado de forma predeterminada.


## <a name="enable-instrumentationengine"></a>Enable-InstrumentationEngine

Habilita el motor de instrumentación mediante la definición de algunas claves del Registro.
Reinicie IIS para aplicar los cambios.

El motor de instrumentación puede complementar los datos recopilados por los SDK de .NET.
Recopila los eventos y mensajes que describen la ejecución de un proceso administrado. Estos eventos y mensajes incluyen códigos de resultado de la dependencia, verbos HTTP y [texto de comando SQL](asp-net-dependencies.md#advanced-sql-tracking-to-get-full-sql-query).

Habilite el motor de instrumentación si:
- Ya ha habilitado la supervisión con el cmdlet Enable pero no ha habilitado el motor de instrumentación.
- Ha instrumentado la aplicación manualmente con los SDK de .NET y desea recopilar datos de telemetría adicionales.

### <a name="examples"></a>Ejemplos

```powershell
PS C:\> Enable-InstrumentationEngine
```

### <a name="parameters"></a>Parámetros

#### <a name="-acceptlicense"></a>-AcceptLicense
**Opcional.** Use este modificador para aceptar la licencia y la declaración de privacidad en las instalaciones sin periféricos.

#### <a name="-verbose"></a>-Verbose
**Parámetro común.** Use este modificador para generar registros detallados.

### <a name="output"></a>Output


##### <a name="example-output-from-successfully-enabling-the-instrumentation-engine"></a>Salida de ejemplo al habilitar correctamente el motor de instrumentación

```
Configuring IIS Environment for instrumentation engine...
Configuring registry for instrumentation engine...
```

## <a name="enable-applicationinsightsmonitoring"></a>Enable-ApplicationInsightsMonitoring.

Permite adjuntar sin código la supervisión de aplicaciones IIS en un equipo de destino.

Este cmdlet modificará el archivo IIS applicationHost.config y establecerá algunas claves del Registro.
También creará un archivo applicationinsights.ikey.config, que define la clave de instrumentación usada por cada aplicación.
IIS cargará RedfieldModule durante el inicio, lo que insertará los SDK de Application Insights en aplicaciones como las de inicio.
Reinicie IIS para que los cambios surtan efecto.

Después de habilitar la supervisión, se recomienda que use [Live Metrics](live-stream.md) para comprobar rápidamente si la aplicación enviaba telemetría.

### <a name="examples"></a>Ejemplos

#### <a name="example-with-a-single-instrumentation-key"></a>Ejemplo con una única clave de instrumentación
En este ejemplo, a todas las aplicaciones en el equipo actual se les asigna una única clave de instrumentación.

```powershell
PS C:\> Enable-ApplicationInsightsMonitoring -InstrumentationKey xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

#### <a name="example-with-an-instrumentation-key-map"></a>Ejemplo con un mapa de claves de instrumentación
En este ejemplo:
- `MachineFilter` busca la correspondencia con el equipo actual usando el comodín `'.*'`.
- `AppFilter='WebAppExclude'` proporciona una clave de instrumentación `null`. No se puede instrumentar la aplicación especificada.
- `AppFilter='WebAppOne'` asigna una clave de instrumentación única a la aplicación especificada.
- `AppFilter='WebAppTwo'` asigna una clave de instrumentación única a la aplicación especificada.
- Por último, `AppFilter` también usa el comodín `'.*'` para buscar la correspondencia de todas las aplicaciones web que no coincidan según las reglas anteriores y les asigna una clave de instrumentación predeterminada.
- Se agregan espacios para mejorar la legibilidad.

```powershell
PS C:\> Enable-ApplicationInsightsMonitoring -InstrumentationKeyMap 
    @(@{MachineFilter='.*';AppFilter='WebAppExclude'},
      @{MachineFilter='.*';AppFilter='WebAppOne';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx1'}},
      @{MachineFilter='.*';AppFilter='WebAppTwo';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx2'}},
      @{MachineFilter='.*';AppFilter='.*';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxdefault'}})

```


### <a name="parameters"></a>Parámetros

#### <a name="-instrumentationkey"></a>-InstrumentationKey
**Requerido.** Use este parámetro para proporcionar una única clave de instrumentación para que la usen todas las aplicaciones en el equipo de destino.

#### <a name="-instrumentationkeymap"></a>-InstrumentationKeyMap
**Requerido.** Use este parámetro para proporcionar varias claves de instrumentación y una asignación de las claves de instrumentación utilizadas por cada aplicación.
Puede crear un solo script de instalación para varios equipos si establece `MachineFilter`.

> [!IMPORTANT]
> Las aplicaciones se harán coincidir con las reglas en el orden en que estas se proporcionan. Por tanto, debe especificar las reglas más específicas en primer lugar y las más genéricas las últimas.

##### <a name="schema"></a>Schema
`@(@{MachineFilter='.*';AppFilter='.*';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'}})`

- **MachineFilter** es una expresión regular de C# requerida del nombre de la máquina virtual o del equipo.
    - ".*" coincidirá con todos
    - "ComputerName" coincidirá solo son aquellos equipos que tengan el nombre exacto especificado.
- **AppFilter** es una expresión regular de C# requerida del nombre del sitio de IIS. Para obtener una lista de sitios en el servidor, ejecute el comando [get-iissite](/powershell/module/iisadministration/get-iissite).
    - ".*" coincidirá con todos
    - "SiteName" coincidirá solo son el sitio de IIS que tenga el nombre exacto especificado.
- **InstrumentationKey** se requiere para habilitar la supervisión de las aplicaciones que coincidan con los dos filtros anteriores.
    - Deje este valor como null si desea definir reglas para excluir la supervisión.


#### <a name="-enableinstrumentationengine"></a>-EnableInstrumentationEngine
**Opcional.** Utilice este modificador para habilitar el motor de instrumentación y recopilar eventos y mensajes sobre lo que sucede durante la ejecución de un proceso administrado. Estos eventos y mensajes incluyen códigos de resultado de la dependencia, verbos HTTP y texto de comando SQL.

El motor de instrumentación se sobrecarga y está desactivado de forma predeterminada.

#### <a name="-acceptlicense"></a>-AcceptLicense
**Opcional.** Use este modificador para aceptar la licencia y la declaración de privacidad en las instalaciones sin periféricos.

#### <a name="-ignoresharedconfig"></a>-IgnoreSharedConfig
Si tiene un clúster de servidores web, puede que esté usando una [configuración compartida](/iis/web-hosting/configuring-servers-in-the-windows-web-platform/shared-configuration_211).
HttpModule no pueden insertarse en esta configuración compartida.
Este script generará un error con un mensaje que indica que se necesitan pasos de instalación adicionales.
Use este modificador para ignorar esta comprobación y seguir instalando requisitos previos. Para obtener más información, consulte [Conflicto con la configuración compartida de IIS](status-monitor-v2-troubleshoot.md#conflict-with-iis-shared-configuration).

#### <a name="-verbose"></a>-Verbose
**Parámetro común.** Utilice este modificador para mostrar registros detallados.

#### <a name="-whatif"></a>-WhatIf 
**Parámetro común.** Utilice este modificador para probar y validar los parámetros de entrada sin habilitar realmente la supervisión.

### <a name="output"></a>Output

#### <a name="example-output-from-a-successful-enablement"></a>Ejemplo de salida de una activación correcta

```powershell
Initiating Disable Process
Applying transformation to 'C:\Windows\System32\inetsrv\config\applicationHost.config'
'C:\Windows\System32\inetsrv\config\applicationHost.config' backed up to 'C:\Windows\System32\inetsrv\config\applicationHost.config.backup-2019-03-26_08-59-52z'
in :1,237
No element in the source document matches '/configuration/location[@path='']/system.webServer/modules/add[@name='ManagedHttpModuleHelper']'
Not executing RemoveAll (transform line 1, 546)
Transformation to 'C:\Windows\System32\inetsrv\config\applicationHost.config' was successfully applied. Operation: 'disable'
GAC Module will not be removed, since this operation might cause IIS instabilities
Configuring IIS Environment for codeless attach...
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\IISADMIN[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W3SVC[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WAS[Environment]
Configuring IIS Environment for instrumentation engine...
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\IISADMIN[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W3SVC[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WAS[Environment]
Configuring registry for instrumentation engine...
Successfully disabled Application Insights Status Monitor
Installing GAC module 'C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\0.2.0\content\Runtime\Microsoft.AppInsights.IIS.ManagedHttpModuleHelper.dll'
Applying transformation to 'C:\Windows\System32\inetsrv\config\applicationHost.config'
Found GAC module Microsoft.AppInsights.IIS.ManagedHttpModuleHelper.ManagedHttpModuleHelper, Microsoft.AppInsights.IIS.ManagedHttpModuleHelper, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35
'C:\Windows\System32\inetsrv\config\applicationHost.config' backed up to 'C:\Windows\System32\inetsrv\config\applicationHost.config.backup-2019-03-26_08-59-52z_1'
Transformation to 'C:\Windows\System32\inetsrv\config\applicationHost.config' was successfully applied. Operation: 'enable'
Configuring IIS Environment for codeless attach...
Configuring IIS Environment for instrumentation engine...
Configuring registry for instrumentation engine...
Updating app pool permissions...
Successfully enabled Application Insights Status Monitor
```

## <a name="disable-instrumentationengine"></a>Disable-InstrumentationEngine

Deshabilita el motor de instrumentación mediante la eliminación de algunas claves del Registro.
Reinicie IIS para aplicar los cambios.

### <a name="examples"></a>Ejemplos

```powershell
PS C:\> Disable-InstrumentationEngine
```

### <a name="parameters"></a>Parámetros 

#### <a name="-verbose"></a>-Verbose
**Parámetro común.** Use este modificador para generar registros detallados.

### <a name="output"></a>Output


##### <a name="example-output-from-successfully-disabling-the-instrumentation-engine"></a>Salida de ejemplo al deshabilitar correctamente el motor de instrumentación

```powershell
Configuring IIS Environment for instrumentation engine...
Registry: removing 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\IISADMIN[Environment]'
Registry: removing 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W3SVC[Environment]'
Registry: removing 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WAS[Environment]'
Configuring registry for instrumentation engine...
```

## <a name="disable-applicationinsightsmonitoring"></a>Disable-ApplicationInsightsMonitoring

Deshabilita la supervisión en el equipo de destino.
Este cmdlet quitará los cambios realizados en el archivo applicationHost.config de IIS y eliminará las claves del Registro.

### <a name="examples"></a>Ejemplos

```powershell
PS C:\> Disable-ApplicationInsightsMonitoring
```

### <a name="parameters"></a>Parámetros 

#### <a name="-verbose"></a>-Verbose
**Parámetro común.** Utilice este modificador para mostrar registros detallados.

### <a name="output"></a>Output


##### <a name="example-output-from-successfully-disabling-monitoring"></a>Salida de ejemplo al deshabilitar correctamente la supervisión

```powershell
Initiating Disable Process
Applying transformation to 'C:\Windows\System32\inetsrv\config\applicationHost.config'
'C:\Windows\System32\inetsrv\config\applicationHost.config' backed up to 'C:\Windows\System32\inetsrv\config\applicationHost.config.backup-2019-03-26_08-59-00z'
in :1,237
No element in the source document matches '/configuration/location[@path='']/system.webServer/modules/add[@name='ManagedHttpModuleHelper']'
Not executing RemoveAll (transform line 1, 546)
Transformation to 'C:\Windows\System32\inetsrv\config\applicationHost.config' was successfully applied. Operation: 'disable'
GAC Module will not be removed, since this operation might cause IIS instabilities
Configuring IIS Environment for codeless attach...
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\IISADMIN[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W3SVC[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WAS[Environment]
Configuring IIS Environment for instrumentation engine...
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\IISADMIN[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W3SVC[Environment]
Registry: skipping non-existent 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WAS[Environment]
Configuring registry for instrumentation engine...
Successfully disabled Application Insights Status Monitor
```


## <a name="get-applicationinsightsmonitoringconfig"></a>Get-ApplicationInsightsMonitoringConfig

Obtiene el archivo de configuración e imprime los valores en la consola.

### <a name="examples"></a>Ejemplos

```powershell
PS C:\> Get-ApplicationInsightsMonitoringConfig
```

### <a name="parameters"></a>Parámetros

No se requiere ningún parámetro.

### <a name="output"></a>Output


##### <a name="example-output-from-reading-the-config-file"></a>Salida de ejemplo al leer el archivo de configuración

```
RedfieldConfiguration:
Filters:
0)InstrumentationKey:  AppFilter: WebAppExclude MachineFilter: .*
1)InstrumentationKey: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx2 AppFilter: WebAppTwo MachineFilter: .*
2)InstrumentationKey: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxdefault AppFilter: .* MachineFilter: .*
```

## <a name="get-applicationinsightsmonitoringstatus"></a>Get-ApplicationInsightsMonitoringStatus

Este cmdlet proporciona información de solución de problemas sobre el Monitor de estado.
Use este cmdlet para investigar el estado de supervisión, la versión del módulo de PowerShell y para inspeccionar el proceso en ejecución.
Este cmdlet notificará la información sobre la versión e información sobre los archivos de clave necesarios para la supervisión.

### <a name="examples"></a>Ejemplos

#### <a name="example-application-status"></a>Ejemplo: Estado de la aplicación

Ejecute el comando `Get-ApplicationInsightsMonitoringStatus` para que aparezca el estado de supervisión de los sitios web.

```powershell

PS C:\Windows\system32> Get-ApplicationInsightsMonitoringStatus

IIS Websites:

SiteName               : Default Web Site
ApplicationPoolName    : DefaultAppPool
SiteId                 : 1
SiteState              : Stopped

SiteName               : DemoWebApp111
ApplicationPoolName    : DemoWebApp111
SiteId                 : 2
SiteState              : Started
ProcessId              : not found

SiteName               : DemoWebApp222
ApplicationPoolName    : DemoWebApp222
SiteId                 : 3
SiteState              : Started
ProcessId              : 2024
Instrumented           : true
InstrumentationKey     : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx123

SiteName               : DemoWebApp333
ApplicationPoolName    : DemoWebApp333
SiteId                 : 4
SiteState              : Started
ProcessId              : 5184
AppAlreadyInstrumented : true
```

En este ejemplo,
- **Machine Identifier (Identificador de la máquina)** es un identificador anónimo utilizado para identificar de forma exclusiva el servidor. Si crea una solicitud de soporte técnico, necesitaremos este identificador para buscar los registros del servidor.
- **Default Web Site (Sitio web predeterminado)** está detenido en IIS
- **DemoWebApp111** se ha iniciado en IIS, pero no ha recibido ninguna solicitud. Este informe muestra que no hay ningún proceso en ejecución (ProcessId: not found).
- **DemoWebApp222** se está ejecutando y se está supervisando (Instrumented: true). Según la configuración del usuario, la clave de instrumentación xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx123 coincidió para este sitio.
- **DemoWebApp333** se ha instrumentado manualmente mediante el SDK de Application Insights. El agente Monitor de estado ha detectado el SDK y no supervisará este sitio.


#### <a name="example-powershell-module-information"></a>Ejemplo: Información sobre el módulo de PowerShell

Ejecute el comando `Get-ApplicationInsightsMonitoringStatus -PowerShellModule` para mostrar información sobre el módulo actual:

```powershell

PS C:\> Get-ApplicationInsightsMonitoringStatus -PowerShellModule

PowerShell Module version:
0.4.0-alpha

Application Insights SDK version:
2.9.0.3872

Executing PowerShell Module Assembly:
Microsoft.ApplicationInsights.Redfield.Configurator.PowerShell, Version=2.8.14.11432, Culture=neutral, PublicKeyToken=31bf3856ad364e35

PowerShell Module Directory:
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\0.2.2\content\PowerShell

Runtime Paths:
ParentDirectory (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content

ConfigurationPath (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\applicationInsights.ikey.config

ManagedHttpModuleHelperPath (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AppInsights.IIS.ManagedHttpModuleHelper.dll

RedfieldIISModulePath (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll

InstrumentationEngine86Path (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation32\MicrosoftInstrumentationEngine_x86.dll

InstrumentationEngine64Path (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\MicrosoftInstrumentationEngine_x64.dll

InstrumentationEngineExtensionHost86Path (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation32\Microsoft.ApplicationInsights.ExtensionsHost_x86.dll

InstrumentationEngineExtensionHost64Path (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\Microsoft.ApplicationInsights.ExtensionsHost_x64.dll

InstrumentationEngineExtensionConfig86Path (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation32\Microsoft.InstrumentationEngine.Extensions.config

InstrumentationEngineExtensionConfig64Path (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\Microsoft.InstrumentationEngine.Extensions.config

ApplicationInsightsSdkPath (Exists: True)
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.dll
```

#### <a name="example-runtime-status"></a>Ejemplo: Estado en tiempo de ejecución

Puede inspeccionar el proceso en el equipo instrumentado para determinar si todos los archivos DLL se cargan. Si funciona la supervisión, deben cargarse al menos 12 DLL.

Ejecute el comando `Get-ApplicationInsightsMonitoringStatus -InspectProcess`:


```
PS C:\> Get-ApplicationInsightsMonitoringStatus -InspectProcess

iisreset.exe /status
Status for IIS Admin Service ( IISADMIN ) : Running
Status for Windows Process Activation Service ( WAS ) : Running
Status for Net.Msmq Listener Adapter ( NetMsmqActivator ) : Running
Status for Net.Pipe Listener Adapter ( NetPipeActivator ) : Running
Status for Net.Tcp Listener Adapter ( NetTcpActivator ) : Running
Status for World Wide Web Publishing Service ( W3SVC ) : Running

handle64.exe -accepteula -p w3wp
BF0: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.ServerTelemetryChannel.dll
C58: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.AzureAppServices.dll
C68: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.DependencyCollector.dll
C78: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.WindowsServer.dll
C98: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.Web.dll
CBC: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.PerfCounterCollector.dll
DB0: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.AI.Agent.Intercept.dll
B98: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll
BB4: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.Contracts.dll
BCC: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.Redfield.Lightup.dll
BE0: File  (R-D)   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.dll

listdlls64.exe -accepteula w3wp
0x0000000019ac0000  0x127000  C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\MicrosoftInstrumentationEngine_x64.dll
0x00000000198b0000  0x4f000   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\Microsoft.ApplicationInsights.ExtensionsHost_x64.dll
0x000000000c460000  0xb2000   C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Instrumentation64\Microsoft.ApplicationInsights.Extensions.Base_x64.dll
0x000000000ad60000  0x108000  C:\Windows\TEMP\2.4.0.0.Microsoft.ApplicationInsights.Extensions.Intercept_x64.dll
```

### <a name="parameters"></a>Parámetros

#### <a name="no-parameters"></a>(Sin parámetros)

De forma predeterminada, este cmdlet notificará el estado de supervisión de las aplicaciones web.
Use esta opción para revisar si la aplicación se ha instrumentado correctamente.
También puede revisar qué clave de instrumentación coincidió con su sitio.


#### <a name="-powershellmodule"></a>-PowerShellModule
**Opcional**. Utilice este modificador para informar de los números de versión y rutas de acceso de los archivos DLL necesarios para la supervisión.
Use esta opción si tiene que identificar la versión de cualquier archivo DLL, incluido el SDK de Application Insights.

#### <a name="-inspectprocess"></a>-InspectProcess

**Opcional**. Utilice este modificador para informar de si se está ejecutando IIS.
También descargará herramientas externas para determinar si los archivos DLL necesarios se han cargado en el entorno de ejecución de IIS.


Si se produce un error en este proceso por cualquier motivo, puede ejecutar estos comandos manualmente:
- iisreset.exe /status
- [handle64.exe](/sysinternals/downloads/handle) -p w3wp | findstr /I "InstrumentationEngine AI. ApplicationInsights"
- [listdlls64.exe](/sysinternals/downloads/listdlls) w3wp | findstr /I "InstrumentationEngine AI ApplicationInsights"


#### <a name="-force"></a>-Force

**Opcional**. Se utiliza solo con InspectProcess. Use este modificador para omitir el mensaje de usuario que aparece antes de que se descarguen las herramientas adicionales.


## <a name="set-applicationinsightsmonitoringconfig"></a>Set-ApplicationInsightsMonitoringConfig

Establece el archivo de configuración sin tener que realizar una reinstalación completa.
Reinicie IIS para que los cambios surtan efecto.

> [!IMPORTANT] 
> Este cmdlet requiere una sesión de PowerShell con permisos de administrador.


### <a name="examples"></a>Ejemplos

#### <a name="example-with-a-single-instrumentation-key"></a>Ejemplo con una única clave de instrumentación
En este ejemplo, a todas las aplicaciones en el equipo actual se les asignará una única clave de instrumentación.

```powershell
PS C:\> Enable-ApplicationInsightsMonitoring -InstrumentationKey xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

#### <a name="example-with-an-instrumentation-key-map"></a>Ejemplo con un mapa de claves de instrumentación
En este ejemplo:
- `MachineFilter` busca la correspondencia con el equipo actual usando el comodín `'.*'`.
- `AppFilter='WebAppExclude'` proporciona una clave de instrumentación `null`. No se puede instrumentar la aplicación especificada.
- `AppFilter='WebAppOne'` asigna una clave de instrumentación única a la aplicación especificada.
- `AppFilter='WebAppTwo'` asigna una clave de instrumentación única a la aplicación especificada.
- Por último, `AppFilter` también usa el comodín `'.*'` para buscar la correspondencia de todas las aplicaciones web que no coincidan según las reglas anteriores y les asigna una clave de instrumentación predeterminada.
- Se agregan espacios para mejorar la legibilidad.

```powershell
Enable-ApplicationInsightsMonitoring -InstrumentationKeyMap `
       @(@{MachineFilter='.*';AppFilter='WebAppExclude'},
          @{MachineFilter='.*';AppFilter='WebAppOne';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx1'}},
          @{MachineFilter='.*';AppFilter='WebAppTwo';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx2'}},
          @{MachineFilter='.*';AppFilter='.*';InstrumentationSettings=@{InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxdefault'}})
```

### <a name="parameters"></a>Parámetros

#### <a name="-instrumentationkey"></a>-InstrumentationKey
**Requerido.** Use este parámetro para proporcionar una única clave de instrumentación para que la usen todas las aplicaciones en el equipo de destino.

#### <a name="-instrumentationkeymap"></a>-InstrumentationKeyMap
**Requerido.** Use este parámetro para proporcionar varias claves de instrumentación y una asignación de las claves de instrumentación utilizadas por cada aplicación.
Puede crear un solo script de instalación para varios equipos si establece `MachineFilter`.

> [!IMPORTANT]
> Las aplicaciones se harán coincidir con las reglas en el orden en que estas se proporcionan. Por tanto, debe especificar las reglas más específicas en primer lugar y las más genéricas las últimas.

##### <a name="schema"></a>Schema
`@(@{MachineFilter='.*';AppFilter='.*';InstrumentationKey='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'})`

- **MachineFilter** es una expresión regular de C# requerida del nombre de la máquina virtual o del equipo.
    - ".*" coincidirá con todos
    - "ComputerName" coincidirá solo son aquellos equipos que tengan el nombre especificado.
- **AppFilter** es una expresión regular de C# requerida del nombre de la máquina virtual o del equipo.
    - ".*" coincidirá con todos
    - "ApplicationName" coincidirá solo son aquellas aplicaciones IIS que tengan el nombre especificado.
- **InstrumentationKey** se requiere para habilitar la supervisión de las aplicaciones que coincidan con los dos filtros anteriores.
    - Deje este valor como null si desea definir reglas para excluir la supervisión.


#### <a name="-verbose"></a>-Verbose
**Parámetro común.** Utilice este modificador para mostrar registros detallados.


### <a name="output"></a>Output

De forma predeterminada, no se produce ninguna salida.

##### <a name="example-verbose-output-from-setting-the-config-file-via--instrumentationkey"></a>Ejemplo de salida detallada de la configuración en el archivo de configuración a través de -InstrumentationKey

```
VERBOSE: Operation: InstallWithIkey
VERBOSE: InstrumentationKeyMap parsed:
Filters:
0)InstrumentationKey: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx AppFilter: .* MachineFilter: .*
VERBOSE: set config file
VERBOSE: Config File Path:
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\applicationInsights.ikey.config
```

##### <a name="example-verbose-output-from-setting-the-config-file-via--instrumentationkeymap"></a>Ejemplo de salida detallada de la configuración en el archivo de configuración a través de -InstrumentationKeyMap

```
VERBOSE: Operation: InstallWithIkeyMap
VERBOSE: InstrumentationKeyMap parsed:
Filters:
0)InstrumentationKey:  AppFilter: WebAppExclude MachineFilter: .*
1)InstrumentationKey: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx2 AppFilter: WebAppTwo MachineFilter: .*
2)InstrumentationKey: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxdefault AppFilter: .* MachineFilter: .*
VERBOSE: set config file
VERBOSE: Config File Path:
C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\applicationInsights.ikey.config
```

## <a name="start-applicationinsightsmonitoringtrace"></a>Start-ApplicationInsightsMonitoringTrace

Recopila [eventos de ETW](/windows/desktop/etw/event-tracing-portal) desde el runtime para adjuntar sin código. Este cmdlet es una alternativa a la ejecución de [PerfView](https://github.com/microsoft/perfview).

Los eventos recopilados se imprimirán en la consola en tiempo real y se guardarán en un archivo ETL. [PerfView](https://github.com/microsoft/perfview) puede abrir el archivo ETL de salida para llevar a cabo una investigación más extensa.

Este cmdlet se ejecutará hasta alcanzar que se agote el tiempo de espera (con un valor predeterminado de 5 minutos) o hasta que se lo detiene manualmente (`Ctrl + C`).

### <a name="examples"></a>Ejemplos

#### <a name="how-to-collect-events"></a>Recopilación de eventos

Por lo general, le pediríamos que recopile eventos para investigar por qué no se está instrumentando la aplicación.

El runtime para adjuntar sin código emitirá eventos de ETW cuando se inician IIA y la aplicación.

Para recopilar estos eventos:
1. En una consola cmd con privilegio de administración, ejecute `iisreset /stop` para desactivar IIS y todas las aplicaciones web.
2. Ejecute este cmdlet.
3. En una consola cmd con privilegio de administración ejecute `iisreset /start` para iniciar IIS.
4. Intente ir a la aplicación.
5. Cuando la aplicación termina de cargarse, puede detenerla de manera manual (`Ctrl + C`) o esperar a que se agote el tiempo de espera.

#### <a name="what-events-to-collect"></a>Qué eventos se recopilan

Existen tres opciones al recopilar los eventos:
1. Use el modificador `-CollectSdkEvents` para recopilar los eventos emitidos desde el SDK de Application Insights.
2. Use el modificador `-CollectRedfieldEvents` para recopilar los eventos que emite el Monitor de estado y el runtime de Redfield. Estos registros son útiles al diagnosticar el inicio de IIS y de las aplicaciones.
3. Use ambos modificadores para recopilar ambos tipos de evento.
4. De manera predeterminada, si no se especifica ningún modificador, se recopilarán ambos tipos de evento.


### <a name="parameters"></a>Parámetros

#### <a name="-maxdurationinminutes"></a>-MaxDurationInMinutes
**Opcional.** Use este parámetro para establecer durante cuánto tiempo este script debe recopilar eventos. El valor predeterminado es 5 minutos.

#### <a name="-logdirectory"></a>-LogDirectory
**Opcional.** Use este conmutador para establecer el directorio de salida del archivo ETL. De manera predeterminada, este archivo se creará en el directorio de módulos de PowerShell. Se mostrará la ruta de acceso completa durante la ejecución de un script.


#### <a name="-collectsdkevents"></a>-CollectSdkEvents
**Opcional.** Use este modificador para recopilar eventos del SDK de Application Insights.

#### <a name="-collectredfieldevents"></a>-CollectRedfieldEvents
**Opcional.** Use este modificador para recopilar eventos desde el Monitor de estado y el runtime de Redfield.

#### <a name="-verbose"></a>-Verbose
**Parámetro común.** Use este modificador para generar registros detallados.



### <a name="output"></a>Output


#### <a name="example-of-application-startup-logs"></a>Ejemplo de registros de inicio de aplicación
```powershell
PS C:\Windows\system32> Start-ApplicationInsightsMonitoringTrace -ColectRedfieldEvents
Starting...
Log File: C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\logs\20190627_144217_ApplicationInsights_ETW_Trace.etl
Tracing enabled, waiting for events.
Tracing will timeout in 5 minutes. Press CTRL+C to cancel.

2:42:31 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace Resolved variables to: MicrosoftAppInsights_ManagedHttpModulePath='C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll', MicrosoftAppInsights_ManagedHttpModuleType='Microsoft.ApplicationInsights.RedfieldIISModule.RedfieldIISModule'
2:42:31 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace Resolved variables to: MicrosoftDiagnosticServices_ManagedHttpModulePath2='', MicrosoftDiagnosticServices_ManagedHttpModuleType2=''
2:42:31 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace Environment variable 'MicrosoftDiagnosticServices_ManagedHttpModulePath2' or 'MicrosoftDiagnosticServices_ManagedHttpModuleType2' is null, skipping managed dll loading
2:42:31 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace MulticastHttpModule.constructor, success, 70 ms
2:42:31 PM EVENT: Microsoft-ApplicationInsights-RedfieldIISModule Trace Current assembly 'Microsoft.ApplicationInsights.RedfieldIISModule, Version=2.8.18.27202, Culture=neutral, PublicKeyToken=f23a46de0be5d6f3' location 'C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll'
2:42:31 PM EVENT: Microsoft-ApplicationInsights-RedfieldIISModule Trace Matched filter '.*'~'STATUSMONITORTE', '.*'~'DemoWithSql'
2:42:31 PM EVENT: Microsoft-ApplicationInsights-RedfieldIISModule Trace Lightup assembly calculated path: 'C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.Redfield.Lightup.dll'
2:42:31 PM EVENT: Microsoft-ApplicationInsights-FrameworkLightup Trace Loaded applicationInsights.config from assembly's resource Microsoft.ApplicationInsights.Redfield.Lightup, Version=2.8.18.27202, Culture=neutral, PublicKeyToken=f23a46de0be5d6f3/Microsoft.ApplicationInsights.Redfield.Lightup.ApplicationInsights-recommended.config
2:42:34 PM EVENT: Microsoft-ApplicationInsights-FrameworkLightup Trace Successfully attached ApplicationInsights SDK
2:42:34 PM EVENT: Microsoft-ApplicationInsights-RedfieldIISModule Trace RedfieldIISModule.LoadLightupAssemblyAndGetLightupHttpModuleClass, success, 2687 ms
2:42:34 PM EVENT: Microsoft-ApplicationInsights-RedfieldIISModule Trace RedfieldIISModule.CreateAndInitializeApplicationInsightsHttpModules(lightupHttpModuleClass), success
2:42:34 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace ManagedHttpModuleHelper, multicastHttpModule.Init() success, 3288 ms
2:42:35 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace Resolved variables to: MicrosoftAppInsights_ManagedHttpModulePath='C:\Program Files\WindowsPowerShell\Modules\Az.ApplicationMonitor\content\Runtime\Microsoft.ApplicationInsights.RedfieldIISModule.dll', MicrosoftAppInsights_ManagedHttpModuleType='Microsoft.ApplicationInsights.RedfieldIISModule.RedfieldIISModule'
2:42:35 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace Resolved variables to: MicrosoftDiagnosticServices_ManagedHttpModulePath2='', MicrosoftDiagnosticServices_ManagedHttpModuleType2=''
2:42:35 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace Environment variable 'MicrosoftDiagnosticServices_ManagedHttpModulePath2' or 'MicrosoftDiagnosticServices_ManagedHttpModuleType2' is null, skipping managed dll loading
2:42:35 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace MulticastHttpModule.constructor, success, 0 ms
2:42:35 PM EVENT: Microsoft-ApplicationInsights-RedfieldIISModule Trace RedfieldIISModule.CreateAndInitializeApplicationInsightsHttpModules(lightupHttpModuleClass), success
2:42:35 PM EVENT: Microsoft-ApplicationInsights-IIS-ManagedHttpModuleHelper Trace ManagedHttpModuleHelper, multicastHttpModule.Init() success, 0 ms
Timeout Reached. Stopping...
```

## <a name="next-steps"></a>Pasos siguientes

  Vea la telemetría:
 - [Explore las métricas](../platform/metrics-charts.md) para supervisar el rendimiento y el uso.
- [Busque en los eventos y los registros](./diagnostic-search.md) para diagnosticar problemas.
- Use [análisis](../log-query/log-query-overview.md) para consultas más avanzadas.
- [Cree paneles](./overview-dashboard.md).
 
 Agregue más telemetría:
 - [Cree pruebas web](monitor-web-app-availability.md) para asegurarse de que el sitio permanece activo.
- [Agregue telemetría de cliente web](./javascript.md) para ver las excepciones de código de la página web y para habilitar las llamadas de seguimiento.
- [Agregue el SDK de Application Insights al código](./asp-net.md) para que pueda insertar llamadas de seguimiento y registro.
 
 Haga mucho más con Application Insights Agent:
 - Use nuestra guía para [solucionar problemas](status-monitor-v2-troubleshoot.md) de Application Insights Agent.

