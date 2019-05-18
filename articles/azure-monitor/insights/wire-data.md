---
title: Solución de datos en Azure Monitor de conexión | Microsoft Docs
description: Los datos de conexión son datos consolidados de rendimiento y de red procedentes de equipos con agentes de Log Analytics. Los datos de red se combinan con los datos de registro para ayudar a correlacionar datos.
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: fc3d7127-0baa-4772-858a-5ba995d1519b
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 10/03/2018
ms.author: magoedte
ms.openlocfilehash: b7968de8ad4357241eb8d86d51e991f4bbab959d
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65601580"
---
# <a name="wire-data-20-preview-solution-in-azure-monitor"></a>Solución de Wire Data 2.0 (versión preliminar) en Azure Monitor

![Símbolo de Wire Data](media/wire-data/wire-data2-symbol.png)

Los datos de conexión son datos consolidados de rendimiento y de red recopilados de equipos conectados a Windows y Linux con el agente de Log Analytics, incluidos los que supervisa Operations Manager en el entorno. Los datos de red se combinan con otros datos de registro para ayudar a correlacionar datos.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

Además del agente de Log Analytics, la solución Wire Data usa las instancias de Microsoft Dependency Agent que instale en los equipos de su infraestructura de TI. Los agentes de dependencia supervisan los datos de red enviados a los equipos y desde estos para los niveles 2 y 3 de la red en el [modelo OSI](https://en.wikipedia.org/wiki/OSI_model), incluidos los distintos protocolos y puertos que se usan. Datos, a continuación, se envían a Azure Monitor mediante agentes.  

>[!NOTE]
>Si ya ha implementado Service Map, o está pensando en mapa de servicio o [Azure Monitor para las máquinas virtuales](../../azure-monitor/insights/vminsights-overview.md), hay un métrica conexión nuevo conjunto de datos que recopile y almacenan en Azure Monitor que proporciona información equivalente a los datos de conexión.

De forma predeterminada, Azure Monitor registra los datos de la CPU, memoria, disco y datos de rendimiento de red de contadores integrados en Windows y Linux, así como otros contadores de rendimiento que se pueden especificar. La recopilación de datos de red y de otros datos se realiza en tiempo real para cada agente, incluidas las subredes y los protocolos de nivel de aplicación utilizados por el equipo.  Wire Data analiza los datos de red en el nivel de aplicación, no hacia abajo en la capa de transporte TCP.  La solución no busca ACK y SYN individuales.  Una vez que se completa el protocolo de enlace, se considera una conexión dinámica y se marca como conectada. La conexión permanece activa siempre y cuando ambos lados acuerden que el socket está abierto y los datos puedan pasar bidireccionalmente.  Una vez que uno de los lados cierra la conexión, se marca como desconectada.  Por lo tanto, solo se cuenta el ancho de banda de los paquetes completados correctamente; no tiene en cuenta los que vuelven a enviar ni los que no se completan.

Si ha usado [sFlow](http://www.sflow.org/) u otro software con el [protocolo NetFlow de Cisco](https://www.cisco.com/c/en/us/products/collateral/ios-nx-os-software/ios-netflow/prod_white_paper0900aecd80406232.html), las estadísticas y los datos que ve de los datos de conexión le resultarán familiares.

Algunos de los tipos de consultas de búsqueda de registros integradas son:

- Agentes que proporcionan datos de conexión
- Dirección IP de los agentes que proporcionan datos de conexión
- Comunicaciones salientes por direcciones IP
- Número de bytes enviados por protocolos de aplicación
- Número de bytes enviados por un servicio de aplicación
- Bytes recibidos por distintos protocolos
- Número total de bytes enviados y recibidos por versión IP
- Promedio de latencia de las conexiones que se midieron de forma fiable
- Procesos de equipo que iniciaron o recibieron tráfico de red
- Cantidad de tráfico de red de un proceso

Cuando realice una búsqueda con datos de conexión, puede filtrar y agrupar datos para ver información sobre los principales agentes y protocolos. O puede ver cuándo determinados equipos (direcciones IP y MAC) se comunicaron entre sí, durante cuánto tiempo y cuántos datos se enviaron (básicamente, verá metadatos sobre el tráfico de red, que se basan en la búsqueda).

Pero como está viendo los metadatos, no son necesariamente útiles para la solución de problemas detallada. Los datos de conexión en Azure Monitor no estén una captura completa de los datos de la red.  Por tanto, no están diseñados para la solución de problemas profunda en el nivel de paquete. La ventaja de usar al agente, en comparación con otros métodos de recopilación, es que no tiene que instalar dispositivos, volver a configurar los conmutadores de red o realizar configuraciones complicadas. Los datos de conexión simplemente se basan en agente: instala el agente en un equipo y supervisará su propio tráfico de red. Otra ventaja es cuando quiere supervisar cargas de trabajo que se ejecutan en proveedores de nube o que hospedan el proveedor de servicio o Microsoft Azure, donde el usuario no tiene la capa de tejido.

## <a name="connected-sources"></a>Orígenes conectados

Wire Data obtiene sus datos de Microsoft Dependency Agent. El agente de dependencia depende del agente de Log Analytics para sus conexiones a Azure Monitor. Es decir, un servidor debe tener instalado y configurado el agente de Log Analytics con Dependency Agent. En la tabla siguiente se describen los orígenes conectados que son compatibles con la solución Wire Data.

| **Origen conectado** | **Compatible** | **Descripción** |
| --- | --- | --- |
| Agentes de Windows | Sí | Wire Data analiza y recopila datos de equipos del agente de Windows. <br><br> Además del [agente de Log Analytics para Windows](../../azure-monitor/platform/agent-windows.md), los agentes de Windows requieren Microsoft Dependency Agent. Consulte los [sistemas operativos compatibles](../../azure-monitor/insights/service-map-configure.md#supported-windows-operating-systems) para obtener una lista completa de las versiones de sistema operativo. |
| Agentes de Linux | Sí | Wire Data analiza y recopila datos de equipos del agente de Linux.<br><br> Además del [agente de Log Analytics para Linux](../../azure-monitor/learn/quick-collect-linux-computer.md), los agentes de Linux requieren Microsoft Dependency Agent. Consulte los [sistemas operativos compatibles](../../azure-monitor/insights/service-map-configure.md#supported-linux-operating-systems) para obtener una lista completa de las versiones de sistema operativo. |
| Grupo de administración de System Center Operations Manager | Sí | Wire Data analiza y recopila datos de los agentes de Windows y Linux en un [grupo de administración de System Center Operations Manager](../../azure-monitor/platform/om-agents.md) conectado. <br><br> Se requiere una conexión directa desde el equipo de agente de System Center Operations Manager a Azure Monitor. |
| Cuenta de almacenamiento de Azure | Sin  | Wire Data recopila datos de equipos de agente, por lo que no hay ningún dato en él que recopilar desde Azure Storage. |

En Windows, se usa Microsoft Monitoring Agent (MMA), System Center Operations Manager y Azure Monitor para recopilar y enviar datos. En función del contexto, el agente se denomina agente de System Center Operations Manager, agente de Log Analytics, MMA o agente directo. System Center Operations Manager y Azure Monitor proporcionan versiones ligeramente diferentes de MMA. Entre estas versiones puede informar a System Center Operations Manager, a Azure Monitor o a ambos.

En Linux, el agente de Log Analytics para Linux recopila y envía datos a Azure Monitor. Puede usar Wire Data en servidores con agentes conectados directamente a Azure Monitor o en servidores que se conectan a Azure Monitor a través de grupos de administración de System Center Operations Manager.

Dependency Agent no transmite los datos por sí mismo y no requiere ningún cambio en los firewalls o puertos. Los datos de Wire Data siempre se transmiten mediante el agente de Log Analytics a Azure Monitor, ya sea directamente o a través de la puerta de enlace de Log Analytics.

![diagrama de agente](./media/wire-data/agents.png)

Si es un usuario de System Center Operations Manager con un grupo de administración conectado a Azure Monitor:

- Cuando los agentes de System Center Operations Manager pueden acceder a internet para conectarse a Azure Monitor, se requiere ninguna configuración adicional.
- Deberá configurar la puerta de enlace de Log Analytics para que funcione con System Center Operations Manager cuando los agentes de System Center Operations Manager no pueden acceder a Azure Monitor a través de internet.

Si los equipos de Windows o Linux no pueden conectarse directamente al servicio, deberá configurar el agente de Log Analytics para conectarse a Azure Monitor con la puerta de enlace de Log Analytics. Puede descargar la nueva versión de la puerta de enlace de Log Analytics en el [Centro de descarga de Microsoft](https://www.microsoft.com/download/details.aspx?id=52666).

## <a name="prerequisites"></a>Requisitos previos

- Se requiere la oferta de la solución [Insight and Analytics](https://www.microsoft.com/cloud-platform/operations-management-suite-pricing).
- Si usa la versión anterior de la solución Wire Data, primero debe quitarla. Pero todos los datos capturados a través de la solución Wire Data original siguen estando disponibles en Wire Data 2.0 y la búsqueda de registros.
- Se requieren privilegios de administrador para instalar o desinstalar Dependency Agent.
- Se debe instalar Dependency Agent en un equipo con un sistema operativo de 64 bits.

### <a name="operating-systems"></a>Sistemas operativos

Las secciones siguientes enumeran los sistemas operativos compatibles para el agente de dependencia. Wire Data no admite arquitecturas de 32 bits para ningún sistema operativo.

#### <a name="windows-server"></a>Windows Server

- Windows Server 2019
- Windows Server 2016 1803
- Windows Server 2016
- Windows Server 2012 R2
- Windows Server 2012
- Windows Server 2008 R2 SP1

#### <a name="windows-desktop"></a>Escritorio de Windows

- Windows 10 1803
- Windows 10
- Windows 8.1
- Windows 8
- Windows 7

#### <a name="supported-linux-operating-systems"></a>Sistemas operativos Linux compatibles
Las secciones siguientes enumeran los sistemas operativos compatibles para el agente de dependencia en Linux.  

- Se admiten solo versiones de kernel SMP Linux y predeterminados.
- Las versiones de kernel no estándar, como PAE y Xen, no son compatibles con ninguna distribución de Linux. Por ejemplo, un sistema con la cadena de versión de "2.6.16.21-0.8-xen" no es compatible.
- No se admiten los kernel personalizados, incluidas las recompilaciones de kernels estándar.

##### <a name="red-hat-linux-7"></a>Red Hat Linux 7

| Versión del SO | Versión del kernel |
|:--|:--|
| 7.4 | 3.10.0-693 |
| 7.5 | 3.10.0-862 |
| 7.6 | 3.10.0-957 |

##### <a name="red-hat-linux-6"></a>Red Hat Linux 6

| Versión del SO | Versión del kernel |
|:--|:--|
| 6.9 | 2.6.32-696 |
| 6.10 | 2.6.32-754 |

##### <a name="centosplus"></a>CentOSPlus
| Versión del SO | Versión del kernel |
|:--|:--|
| 6.9 | 2.6.32-696.18.7<br>2.6.32-696.30.1 |
| 6.10 | 2.6.32-696.30.1<br>2.6.32-754.3.5 |

##### <a name="ubuntu-server"></a>Ubuntu Server

| Versión del SO | Versión del kernel |
|:--|:--|
| Ubuntu 18.04 | kernel 4.15.\*<br>4.18* |
| Ubuntu 16.04.3 | kernel 4.15.* |
| 16.04 | 4.4.\*<br>4.8.\*<br>4.10.\*<br>4.11.\*<br>4.13.\* |
| 14.04 | 3.13.\*<br>4.4.\* |

##### <a name="suse-linux-11-enterprise-server"></a>SUSE Linux 11 Enterprise Server

| Versión del SO | Versión del kernel
|:--|:--|
| 11 SP4 | 3.0.* |

##### <a name="suse-linux-12-enterprise-server"></a>SUSE Linux 12 Enterprise Server

| Versión del SO | Versión del kernel
|:--|:--|
| 12 SP2 | 4.4.* |
| 12 SP3 | 4.4.* |

### <a name="dependency-agent-downloads"></a>Descargas de Dependency Agent

| Archivo | SO | Versión | SHA-256 |
|:--|:--|:--|:--|
| [InstallDependencyAgent-Windows.exe](https://aka.ms/dependencyagentwindows) |  Windows | 9.7.4 | A111B92AB6CF28EB68B696C60FE51F980BFDFF78C36A900575E17083972989E0 |
| [InstallDependencyAgent-Linux64.bin](https://aka.ms/dependencyagentlinux) | Linux | 9.7.4 | AB58F3DB8B1C3DEE7512690E5A65F1DFC41B43831543B5C040FCCE8390F2282C |



## <a name="configuration"></a>Configuración

Siga los pasos siguientes para configurar la solución Wire Data para las áreas de trabajo.

1. Habilite la solución Activity Log Analytics desde la [Azure marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.WireData2OMS?tab=Overview) o mediante el proceso descrito en [agregar soluciones desde la Galería de soluciones de supervisión](../../azure-monitor/insights/solutions.md).
2. Instale Dependency Agent en cada equipo donde quiere obtener datos. Dependency Agent puede supervisar las conexiones a los vecinos inmediatos, por lo que es posible que no necesite un agente en cada equipo.

> [!NOTE]
> No se puede agregar la versión anterior de la solución Wire Data a nuevas áreas de trabajo. Si tiene la solución Wire Data original habilitada pueden continuar usándola. Pero para usar Wire Data 2.0, primero debe quitar la versión original.
> 
 
### <a name="install-the-dependency-agent-on-windows"></a>Instalar Dependency Agent en Windows

Se requieren privilegios de administrador para instalar o desinstalar al agente.

Dependency Agent se instala en equipos que ejecutan Windows mediante InstallDependencyAgent-Windows.exe. Si ejecuta este archivo ejecutable sin opciones, se inicia un asistente que puede seguir para realizar la instalación de forma interactiva.

Use los pasos siguientes para instalar Dependency Agent en cada equipo que ejecute Windows:

1. Instale el agente de Log Analytics siguiendo los pasos descritos en [Collect data from Windows computers hosted in your environment](../../azure-monitor/platform/agent-windows.md) (Recopilar datos de equipos Windows hospedados en el entorno).
2. Descargue Dependency Agent de Windows mediante el vínculo de la sección anterior y, después, ejecútelo mediante el comando siguiente: `InstallDependencyAgent-Windows.exe`.
3. Siga el asistente para instalar el agente.
4. Si el agente de dependencia no se inicia, compruebe los registros para obtener información detallada del error. En los agentes de Windows, el directorio de registro es %Programfiles%\Microsoft Dependency Agent\logs.

#### <a name="windows-command-line"></a>Línea de comandos de Windows

Use las opciones de la tabla siguiente para realizar la instalación desde una línea de comandos. Para ver una lista de marcas de instalación, ejecute el programa de instalación con la marca /? de la manera siguiente.

InstallDependencyAgent-Windows.exe /?

| **Marca** | **Descripción** |
| --- | --- |
| <code>/?</code> | Obtenga una lista de las opciones de la línea de comandos. |
| <code>/S</code> | Realice una instalación silenciosa sin preguntas. |

Los archivos para el agente de dependencia de Windows se ubican en C:\Archivos de programa\Microsoft Dependency Agent de manera predeterminada.

### <a name="install-the-dependency-agent-on-linux"></a>Instalar Dependency Agent en Linux

Es necesario el acceso raíz para instalar o configurar el agente.

Dependency Agent se instala en equipos Linux con InstallDependencyAgent-Linux64.bin, un script de shell con un archivo binario autoextraíble. Puede ejecutar el archivo con _sh_ o agregar permisos de ejecución al propio archivo.

Utilice los pasos siguientes para instalar el agente de dependencia en cada equipo de Linux:

1. Instale el agente de Log Analytics siguiendo los pasos descritos en [Collect data from Linux computers hosted in your environment](../../azure-monitor/learn/quick-collect-linux-computer.md#obtain-workspace-id-and-key) (Recopilar datos de equipos Linux hospedados en el entorno).
2. Descargue Dependency Agent de Linux mediante el vínculo de la sección anterior y, después, instálelo como raíz mediante el comando siguiente: sh InstallDependencyAgent-Linux64.bin
3. Si el agente de dependencia no se inicia, compruebe los registros para obtener información detallada del error. En los agentes de Linux, el directorio de registro es: /var/opt/microsoft/dependency-agent/log.

Para ver una lista de marcas de instalación, ejecute el programa de instalación con la marca `-help` de la siguiente forma.

```
InstallDependencyAgent-Linux64.bin -help
```

| **Marca** | **Descripción** |
| --- | --- |
| <code>-help</code> | Obtenga una lista de las opciones de la línea de comandos. |
| <code>-s</code> | Realice una instalación silenciosa sin preguntas. |
| <code>--check</code> | Compruebe los permisos y el sistema operativo, pero no instale el agente. |

Los archivos del agente de dependencia se colocan en los directorios siguientes:

| **Archivos** | **Ubicación** |
| --- | --- |
| Archivos principales | /opt/microsoft/dependency-agent |
| Archivos de registro | /var/opt/microsoft/dependency-agent/log |
| Archivos de configuración | /etc/opt/microsoft/dependency-agent/config |
| Archivos ejecutables de servicio | /opt/microsoft/dependency-agent/bin/microsoft-dependency-agent<br><br>/opt/microsoft/dependency-agent/bin/microsoft-dependency-agent-manager |
| Archivos binarios de almacenamiento | /var/opt/microsoft/dependency-agent/storage |

### <a name="installation-script-examples"></a>Ejemplos de script de instalación

Para implementar fácilmente Dependency Agent en muchos servidores a la vez, resulta útil usar un script. Puede usar los siguientes ejemplos de script para descargar e instalar Dependency Agent en Windows o Linux.

#### <a name="powershell-script-for-windows"></a>Script de PowerShell para Windows

```powershell

Invoke-WebRequest "https://aka.ms/dependencyagentwindows" -OutFile InstallDependencyAgent-Windows.exe

.\InstallDependencyAgent-Windows.exe /S

```

#### <a name="shell-script-for-linux"></a>Script de shell para Linux

```
wget --content-disposition https://aka.ms/dependencyagentlinux -O InstallDependencyAgent-Linux64.bin
```

```
sh InstallDependencyAgent-Linux64.bin -s
```

### <a name="desired-state-configuration"></a>Configuración de estado deseada

Para implementar Dependency Agent a través de Desired State Configuration, puede usar el módulo xPSDesiredStateConfiguration y un fragmento de código similar al siguiente:

```powershell
Import-DscResource -ModuleName xPSDesiredStateConfiguration

$DAPackageLocalPath = "C:\InstallDependencyAgent-Windows.exe"



Node $NodeName

{

    # Download and install the Dependency Agent

    xRemoteFile DAPackage

    {

        Uri = "https://aka.ms/dependencyagentwindows"

        DestinationPath = $DAPackageLocalPath

        DependsOn = "[Package]OI"

    }

    xPackage DA

    {

        Ensure = "Present"

        Name = "Dependency Agent"

        Path = $DAPackageLocalPath

        Arguments = '/S'

        ProductId = ""

        InstalledCheckRegKey = "HKEY\_LOCAL\_MACHINE\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\DependencyAgent"

        InstalledCheckRegValueName = "DisplayName"

        InstalledCheckRegValueData = "Dependency Agent"

    }

}

```

### <a name="uninstall-the-dependency-agent"></a>Desinstalar Dependency Agent

Use las secciones siguientes para quitar Dependency Agent.

#### <a name="uninstall-the-dependency-agent-on-windows"></a>Desinstalar Dependency Agent en Windows

Un administrador puede desinstalar Dependency Agent de Windows a través del Panel de control.

Un administrador también puede ejecutar %Programfiles%\Microsoft Dependency Agent\Uninstall.exe para desinstalar Dependency Agent.

#### <a name="uninstall-the-dependency-agent-on-linux"></a>Desinstalar Dependency Agent en Linux

Para desinstalar completamente Dependency Agent de Linux, debe quitar el agente y el conector que se instala automáticamente con el agente. Puede desinstalar ambos mediante el comando siguiente:

```
rpm -e dependency-agent dependency-agent-connector
```

## <a name="management-packs"></a>Módulos de administración

Cuando se activa Wire Data en un área de trabajo de Log Analytics, se envía un módulo de administración de 300 KB a todos los servidores Windows en esa área de trabajo. Si usa agentes de System Center Operations Manager en un [grupo de administración conectado](../platform/om-agents.md), el módulo de administración de Dependency Monitor se implementa desde System Center Operations Manager. Si los agentes se conectan directamente, Azure Monitor ofrece el módulo de administración.

El módulo de administración se denomina Microsoft.IntelligencePacks.ApplicationDependencyMonitor. Se escribe en: %Programfiles%\Microsoft Monitoring Agent\Agent\Health Service State\Management Packs. El origen de datos que el módulo de administración usa es: %Program files%\Microsoft Monitoring Agent\Agent\Health Service State\Resources&lt;AutoGeneratedID&gt;\Microsoft.EnterpriseManagement.Advisor.ApplicationDependencyMonitorDataSource.dll.

## <a name="using-the-solution"></a>Uso de la solución

Utilice la siguiente información para instalar y configurar la solución.

- La solución Datos de conexión adquiere datos desde equipos que ejecutan Windows Server 2012 R2, Windows 8.1 y sistemas operativos posteriores.
- Se requiere Microsoft .NET Framework 4.0 o posterior en equipos de los que desea adquirir datos de conexión.
- Agregar la solución Wire Data a su área de trabajo de Log Analytics mediante el proceso descrito en [agregar soluciones desde la Galería de soluciones de supervisión](solutions.md). No es necesario realizar ninguna configuración más.
- Si quiere ver los datos de conexión de una solución específica, debe tener la solución ya agregada a su área de trabajo.

Después de instalar los agentes y la solución, el icono de Wire Data 2.0 aparece en el área de trabajo.

![Icono de Wire Data](./media/wire-data/wire-data-tile.png)

## <a name="using-the-wire-data-20-solution"></a>Uso de la solución Wire Data 2.0

En la página **Introducción** del área de trabajo de Log Analytics en Azure Portal, haga clic en el icono **Wire Data 2.0** para abrir el panel de Wire Data. El panel incluye las hojas de la tabla siguiente. Cada hoja muestra hasta diez elementos que coinciden con los criterios de esa hoja para el ámbito e intervalo de tiempo especificados. Puede ejecutar una búsqueda de registros que devuelva todos los registros si hace clic en **Ver todo** en la parte inferior de la hoja o si hace clic en el encabezado de esta.

| **Hoja** | **Descripción** |
| --- | --- |
| Agentes que capturan el tráfico de red | Muestra el número de agentes que capturan el tráfico de red y enumera los 10 equipos principales que capturan el tráfico. Haga clic en el número para ejecutar una búsqueda de registros de <code>WireData \| summarize sum(TotalBytes) by Computer \| take 500000</code>. Haga clic en un equipo de la lista para ejecutar una búsqueda de registros que devuelve el número total de bytes capturados. |
| Subredes locales | Muestra el número de subredes locales que los agentes han detectado.  Haga clic en el número para ejecutar una búsqueda de registros para <code>WireData \| summarize sum(TotalBytes) by LocalSubnet</code> que enumera todas las subredes con el número de bytes enviados a través de cada una de ellas. Haga clic en una subred de la lista para ejecutar una búsqueda de registros que devuelve el número total de bytes enviados sobre la subred. |
| Protocolos de nivel de aplicación | Muestra el número de protocolos de nivel de aplicación en uso detectados por los agentes. Haga clic en el número para ejecutar una búsqueda de registros de <code>WireData \| summarize sum(TotalBytes) by ApplicationProtocol</code>. Haga clic en un protocolo para ejecutar una búsqueda de registros que devuelve el número total de bytes enviados mediante el protocolo. |

![Panel de Wire Data](./media/wire-data/wire-data-dash.png)

Puede usar la hoja **Agentes que capturan el tráfico de red** para determinar la cantidad de ancho de banda de red que usan los equipos. Esta hoja puede ayudar a encontrar fácilmente el equipo _que emite más información_ en su entorno. Estos equipos se pueden sobrecargar, actuar de forma anómala o usar más recursos de red de lo normal.

![ejemplo de búsqueda de registros](./media/wire-data/log-search-example01.png)

De forma similar, puede usar la hoja **Subredes locales** para determinar cuánto tráfico de red se está moviendo a través de las subredes. A menudo, los usuarios definen subredes en torno a áreas cruciales de sus aplicaciones. En esta hoja se ofrece una vista de esas áreas.

![ejemplo de búsqueda de registros](./media/wire-data/log-search-example02.png)

La hoja **Protocolos en el nivel de la aplicación** es útil porque ayuda a saber qué protocolos están en uso. Por ejemplo, es posible que espere que SSH no esté en uso en su entorno de red. Ver la información disponible en la hoja permite confirmar o desmentir rápidamente las expectativas.

![ejemplo de búsqueda de registros](./media/wire-data/log-search-example03.png)

También es útil saber si el tráfico de protocolo aumenta o disminuye con el tiempo. Por ejemplo, si está aumentando la cantidad de datos que transmite una aplicación, es posible que se trate de algo que deba tener en cuenta o que le resulte de interés.

## <a name="input-data"></a>Datos de entrada

La solución Datos de conexión recopila metadatos sobre el tráfico de red con los agentes habilitados. Cada agente envía datos cada 15 segundos aproximadamente.

## <a name="output-data"></a>Datos de salida

Se crea un registro con un tipo de _WireData_ para cada tipo de datos de entrada. Los registros de WireData tienen las propiedades mostradas en la tabla siguiente:

| Propiedad | Descripción |
|---|---|
| Computer | Nombre de equipo del que se recopilan los datos |
| TimeGenerated | Hora del registro |
| LocalIP | Dirección IP del equipo local |
| SessionState | Conectado o desconectado |
| ReceivedBytes | Cantidad de bytes recibidos |
| ProtocolName | Nombre del protocolo de red que se usa |
| IPVersion | Versión de la dirección IP |
| Dirección | De entrada o de salida |
| MaliciousIP | Dirección IP de un origen malintencionado conocido |
| Severity | Gravedad del supuesto malware |
| RemoteIPCountry | País o región de la dirección IP remota |
| ManagementGroupName | Nombre del grupo de administración de Operations Manager |
| SourceSystem | Origen del que se recopilan los datos |
| SessionStartTime | Hora de inicio de la sesión |
| SessionEndTime | Hora de finalización de la sesión |
| LocalSubnet | Subred de la que se recopilan los datos |
| LocalPortNumber | Número de puerto local |
| RemoteIP | Dirección IP remota que usa el equipo remoto |
| RemotePortNumber | Número de puerto usado por la dirección IP remota |
| SessionID | Un valor único que identifica la sesión de comunicaciones entre dos direcciones IP |
| SentBytes | Número de bytes enviados |
| TotalBytes | Número total de bytes enviados durante la sesión |
| ApplicationProtocol | Tipo del protocolo de red que se usa   |
| ProcessID | Id. de proceso de Windows |
| ProcessName | Nombre de archivo y ruta de acceso del proceso |
| RemoteIPLongitude | Valor de longitud IP |
| RemoteIPLatitude | Valor de latitud IP |


## <a name="next-steps"></a>Pasos siguientes

- [Búsquedas de registros en Log Analytics](../../azure-monitor/log-query/log-query-overview.md) para más información sobre cómo ver registros de búsqueda de datos de conexión detallados.