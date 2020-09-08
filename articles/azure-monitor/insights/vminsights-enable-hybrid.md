---
title: Habilitar Azure Monitor para un entorno híbrido
description: En este artículo se describe cómo habilitar Azure Monitor para VM para un entorno de nube híbrida que contenga una o más máquinas virtuales.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/27/2020
ms.openlocfilehash: d994df4d56f4958784256ff9cd92ce1e6f3b3e50
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88642170"
---
# <a name="enable-azure-monitor-for-vms-for-a-hybrid-virtual-machine"></a>Habilitar Azure Monitor para VM para una máquina virtual híbrida
En este artículo se describe cómo habilitar Azure Monitor para VM para una máquina virtual fuera de Azure, incluidos los entornos locales y otros entornos en la nube.

> [!IMPORTANT]
> El método recomendado para habilitar máquinas virtuales híbridas primero consiste en habilitar [Azure Arc para servidores](../../azure-arc/servers/overview.md), a fin de que las máquinas virtuales se puedan habilitar para Azure Monitor para VM con procesos similares a los de las máquinas virtuales de Azure. En este artículo se describe cómo incorporar máquinas virtuales híbridas si decide no usar Azure Arc.

## <a name="prerequisites"></a>Requisitos previos

- [Cree y configure un área de trabajo de Log Analytics](vminsights-configure-workspace.md).
- Vea [Sistemas operativos admitidos](vminsights-enable-overview.md#supported-operating-systems) para asegurarse de que el sistema operativo de la máquina virtual o el conjunto de escalado de máquinas virtuales que va a habilitar son compatibles. 


## <a name="overview"></a>Información general
Las máquinas virtuales fuera de Azure requieren el mismo agente de Log Analytics y el mismo Dependency Agent que los que se usan para las máquinas virtuales de Azure. Dado que no se pueden usar las extensiones de máquina virtual para instalar los agentes, debe instalarlos manualmente en el sistema operativo invitado o mediante algún otro método. 

Vea [Conexión de equipos Windows a Azure Monitor](../platform/agent-windows.md) o [Conexión de equipos Linux a Azure Monitor](../platform/agent-linux.md) para más información sobre la implementación del agente de Log Analytics. En este artículo se proporcionan detalles sobre Dependency Agent. 

## <a name="firewall-requirements"></a>Requisitos de firewall
Los requisitos de firewall para el agente de Log Analytics se proporcionan en [Introducción al agente de Log Analytics](../platform/log-analytics-agent.md#network-requirements). La extensión Dependency Agent para Service Map de Azure Monitor para VM no transmite ningún dato y no requiere ningún cambio en firewalls o puertos. Los datos de Mapa siempre son transmitidos por el agente de Log Analytics al servicio de Azure Monitor, ya sea directamente o mediante la [Puerta de enlace de Operations Management Suite](../../azure-monitor/platform/gateway.md), si las directivas de seguridad de TI no permiten que los equipos de la red se conecten a Internet.


## <a name="dependency-agent"></a>Dependency Agent

>[!NOTE]
>La siguiente información que se describe en esta sección también se aplica a la [solución Service Map](service-map.md).  

Puede descargar Dependency Agent desde estas ubicaciones:

| Archivo | SO | Versión | SHA-256 |
|:--|:--|:--|:--|
| [InstallDependencyAgent-Windows.exe](https://aka.ms/dependencyagentwindows) | Windows | 9.10.5.10940 | C27A56D0BE9CF162DF73292DFBB2083F5FF749F2B80FCAD2545BC8B14B64A8D7  |
| [InstallDependencyAgent-Linux64.bin](https://aka.ms/dependencyagentlinux) | Linux | 9.10.5.10940 | 71B4E1DA5116E61E03317C49C6702B5069F01A0C9A7CB860F6ACFAF5C198740E |


## <a name="install-the-dependency-agent-on-windows"></a>Instalar Dependency Agent en Windows

Para instalar Dependency Agent manualmente en equipos Windows, puede ejecutar `InstallDependencyAgent-Windows.exe`. Si ejecuta este archivo ejecutable sin opciones, se inicia un asistente para instalación que puede seguir para realizar la instalación del agente de forma interactiva. Necesita privilegios de *administrador* en el sistema operativo invitado para instalar o desinstalar el agente.

En la tabla siguiente se destacan los parámetros que admite el programa de instalación para el agente desde la línea de comandos.

| Parámetro | Descripción |
|:--|:--|
| /? | Devuelve una lista de las opciones de la línea de comandos. |
| /S | Realiza una instalación silenciosa sin interacción del usuario. |

Por ejemplo, para ejecutar el programa de instalación con el parámetro `/?`, escriba **InstallDependencyAgent-Windows.exe /?** .

Los archivos de Dependency Agent para Windows se instalan en *C:\Archivos de programa\Microsoft Dependency Agent* de manera predeterminada. Si Dependency Agent no se inicia una vez completada la instalación, compruebe los registros para obtener información detallada del error. El directorio de registro es *%Programfiles%\Microsoft Dependency Agent\logs*.

### <a name="powershell-script"></a>Script de PowerShell
Use el siguiente script de PowerShell de ejemplo para descargar e instalar el agente:

```powershell
Invoke-WebRequest "https://aka.ms/dependencyagentwindows" -OutFile InstallDependencyAgent-Windows.exe

.\InstallDependencyAgent-Windows.exe /S
```


## <a name="install-the-dependency-agent-on-linux"></a>Instalación de Dependency Agent en Linux

Dependency Agent se instala en los servidores Linux desde *InstallDependencyAgent-Linux64.bin*, un script de shell con un archivo binario autoextraíble. Puede ejecutar el archivo mediante `sh` o agregar permisos de ejecución al propio archivo.

>[!NOTE]
> Es necesario el acceso raíz para instalar o configurar el agente.
>

| Parámetro | Descripción |
|:--|:--|
| -help | Obtenga una lista de las opciones de la línea de comandos. |
| -S | Realice una instalación silenciosa sin preguntas. |
| --check | Compruebe los permisos y el sistema operativo, pero no instale el agente. |

Por ejemplo, para ejecutar el programa de instalación con el parámetro `-help`, escriba **InstallDependencyAgent-Linux64.bin -help**. Para instalar Dependency en Linux como raíz, ejecute el comando `sh InstallDependencyAgent-Linux64.bin`.

Si Dependency Agent no se inicia, compruebe los registros para obtener información detallada del error. En los agentes de Linux, el directorio de registro es */var/opt/microsoft/dependency-agent/log*.

Los archivos de Dependency Agent se colocan en los directorios siguientes:

| Archivos | Location |
|:--|:--|
| Archivos principales | /opt/microsoft/dependency-agent |
| Archivos de registro | /var/opt/microsoft/dependency-agent/log |
| Archivos de configuración | /etc/opt/microsoft/dependency-agent/config |
| Archivos ejecutables de servicio | /opt/microsoft/dependency-agent/bin/microsoft-dependency-agent<br>/opt/microsoft/dependency-agent/bin/microsoft-dependency-agent-manager |
| Archivos binarios de almacenamiento | /var/opt/microsoft/dependency-agent/storage |

### <a name="shell-script"></a>Script de shell 
Use el siguiente script de shell de ejemplo para descargar e instalar el agente:

```
wget --content-disposition https://aka.ms/dependencyagentlinux -O InstallDependencyAgent-Linux64.bin
sudo sh InstallDependencyAgent-Linux64.bin -s
```

## <a name="desired-state-configuration"></a>Configuración de estado deseado

Para implementar Dependency Agent mediante Desired State Configuration (DSC), puede usar el módulo xPSDesiredStateConfiguration con el siguiente código de ejemplo:

```powershell
configuration VMInsights {

    Import-DscResource -ModuleName xPSDesiredStateConfiguration

    $DAPackageLocalPath = "C:\InstallDependencyAgent-Windows.exe"

    Node localhost
    {
        # Download and install the Dependency agent
        xRemoteFile DAPackage
        {
            Uri = "https://aka.ms/dependencyagentwindows"
            DestinationPath = $DAPackageLocalPath
        }

        xPackage DA
        {
            Ensure="Present"
            Name = "Dependency Agent"
            Path = $DAPackageLocalPath
            Arguments = '/S'
            ProductId = ""
            InstalledCheckRegKey = "HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\DependencyAgent"
            InstalledCheckRegValueName = "DisplayName"
            InstalledCheckRegValueData = "Dependency Agent"
            DependsOn = "[xRemoteFile]DAPackage"
        }
    }
}
```



## <a name="troubleshooting"></a>Solución de problemas

### <a name="vm-doesnt-appear-on-the-map"></a>La VM no aparece en la asignación

Si la instalación del agente de dependencia se completó correctamente, pero no ve el equipo en el mapa, siga estos pasos para diagnosticar el problema.

1. ¿Se ha instalado Dependency Agent correctamente? Para validarlo, compruebe si el servicio está instalado y se ejecuta.

    **Windows**: Busque el servicio llamado "Microsoft Dependency Agent".

    **Linux**: Busque el proceso en ejecución "microsoft-dependency-agent".

2. ¿Está en el [plan de tarifa gratuito de Log Analytics ](./solutions.md)? El plan gratuito permite hasta cinco equipos únicos. Ningún equipo posterior aparecerá en la asignación, incluso si los cinco anteriores ya no envían datos.

3. ¿El equipo envía datos de registro y de rendimiento a los registros de Azure Monitor? Realice la consulta siguiente para el equipo:

    ```Kusto
    Usage | where Computer == "computer-name" | summarize sum(Quantity), any(QuantityUnit) by DataType
    ```

    ¿Devolvió uno o más resultados? ¿Son los datos recientes? Si es así, el agente de Log Analytics funciona correctamente y se comunica con el servicio. Si no es así, compruebe el agente en el servidor: [Log Analytics agent for Windows troubleshooting](../platform/agent-windows-troubleshoot.md) (Cómo solucionar problemas relacionados con el agente de Windows de Log Analytics) o [Cómo solucionar problemas relacionados con el agente de Linux de Log Analytics](../platform/agent-linux-troubleshoot.md).

#### <a name="computer-appears-on-the-map-but-has-no-processes"></a>El equipo aparece en la asignación, pero no tiene ningún proceso

Si ve el servidor en el mapa, pero no tienen datos de procesos ni de conexión, eso indica que Dependency Agent está instalado y se está ejecutando, pero el controlador del kernel no se cargó.

Revise el archivo C:\Program Files\Microsoft Dependency Agent\logs\wrapper.log (Windows) o el archivo /var/opt/microsoft/dependency-agent/log/service.log (Linux). Las últimas líneas del archivo deben indicar por qué no se cargó el kernel. Por ejemplo, es posible que el kernel no sea compatible con Linux si actualizó el kernel.


## <a name="next-steps"></a>Pasos siguientes

Ahora que la supervisión está habilitada para las máquinas virtuales, esta información está disponible para analizarse con Azure Monitor para VM.

- Para ver las dependencias de las aplicaciones detectadas, consulte [Uso de la asignación de Azure Monitor para VM (versión preliminar) para conocer los componentes de una aplicación](vminsights-maps.md).

- Para identificar los cuellos de botella y el uso general con el rendimiento de la máquina virtual, vea [Cómo representar el rendimiento en gráficos con Azure Monitor para VM (versión preliminar)](vminsights-performance.md).