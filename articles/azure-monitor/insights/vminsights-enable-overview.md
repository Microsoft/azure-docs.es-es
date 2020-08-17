---
title: Información general sobre la habilitación de Azure Monitor para VM
description: Aprenda a implementar y configurar Azure Monitor para VM. Más información sobre los requisitos del sistema.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/27/2020
ms.openlocfilehash: e3c5f6d7e04620cf36f6cd952467d47afd775b19
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87824773"
---
# <a name="enable-azure-monitor-for-vms-overview"></a>Información general sobre la habilitación de Azure Monitor para VM

En este artículo se proporciona información general sobre las opciones disponibles para habilitar Azure Monitor para VM con el fin de supervisar el estado y el rendimiento de lo siguiente:

- Azure Virtual Machines 
- Conjuntos de escalado de máquinas virtuales de Azure
- Máquinas virtuales híbridas conectadas con Azure Arc
- Máquinas virtuales locales
- Máquinas virtuales hospedadas en otro entorno de nube  

Para configurar Azure Monitor para VM:

* Habilite una única VM de Azure, VMSS de Azure o máquina de Azure Arc; para ello, seleccione **Insights** directamente del menú de Azure Portal.
* Habilite varias VM de Azure, VMSS de Azure o máquinas de Azure Arc mediante Azure Policy. Este método asegura que se instalan y configuran correctamente las dependencias necesarias en las máquinas virtuales y conjuntos de escalado nuevos y existentes. Se informa de las máquinas virtuales y conjuntos de escalado no compatibles para que pueda decidir si habilitarlas y cómo corregir las incompatibilidades.
* Habilite varias VM de Azure, VM de Azure Arc, VMSS de Azure o máquinas de Azure Arc a través de una suscripción o un grupo de recursos especificado mediante PowerShell.
* Habilite Azure Monitor para VM para supervisar las máquinas virtuales o equipos físicos hospedados en la red corporativa o en otro entorno de nube.

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar, asegúrese de conocer la información de los apartados siguientes. 

>[!NOTE]
>La siguiente información que se describe en esta sección también se aplica a la [solución Service Map](service-map.md).  

### <a name="log-analytics"></a>Log Analytics

Azure Monitor para VM admite un área de trabajo de Log Analytics en las siguientes regiones:

- Centro-Oeste de EE. UU.
- Oeste de EE. UU.
- Oeste de EE. UU. 2
- Centro-sur de EE. UU.
- Este de EE. UU.
- Este de EE. UU. 2
- Centro de EE. UU.
- Centro-Norte de EE. UU
- US Gov Az
- US Gov Va
- Centro de Canadá
- Sur de Reino Unido
- Norte de Europa
- Oeste de Europa
- Este de Asia
- Sudeste de Asia
- Centro de la India
- Japón Oriental
- Este de Australia
- Sudeste de Australia

>[!NOTE]
>Puede supervisar máquinas virtuales de Azure en cualquier región. Las máquinas virtuales no se limitan a las regiones admitidas por el área de trabajo de Log Analytics.
>

Si no tiene un área de trabajo de Log Analytics, puede crear una usando alguno de estos recursos:
* [CLI de Azure](../learn/quick-create-workspace-cli.md)
* [PowerShell](../platform/powershell-workspace-configuration.md)
* [Azure Portal](../learn/quick-create-workspace.md)
* [Azure Resource Manager](../platform/template-workspace-configuration.md)

- Máquina virtual de Azure
- Conjunto de escalado de máquinas virtuales de Azure
- Máquina virtual híbrida conectada con Azure Arc

## <a name="supported-operating-systems"></a>Sistemas operativos admitidos

La siguiente es una lista de los sistemas operativos Windows y Linux que son compatibles con Azure Monitor para VM. Más adelante en esta sección, encontrará una lista completa que detalla las versiones de kernel admitidas y las versiones de sistema operativo Linux principales y secundarias.

|Versión del SO |Rendimiento |Mapas |
|-----------|------------|-----|
|Windows Server 2019 | X | X |
|Windows Server 2016 1803 | X | X |
|Windows Server 2016 | X | X |
|Windows Server 2012 R2 | X | X |
|Windows Server 2012 | X | X |
|Windows Server 2008 R2 | X | X|
|Windows 10 1803 | X | X |
|Windows 8.1 | X | X |
|Windows 8 | X | X |
|Windows 7 SP1 | X | X |
|Red Hat Enterprise Linux (RHEL) 6, 7| X | X| 
|Ubuntu 18.04, 16.04 | X | X |
|CentOS Linux 7, 6 | X | X |
|SUSE Linux Enterprise Server (SLES) 12 | X | X |
|Debian 9.4, 8 | X<sup>1</sup> | |

<sup>1</sup> La característica Rendimiento de Azure Monitor para VM solo está disponible desde Azure Monitor. No está disponible directamente desde el panel izquierdo de la máquina virtual de Azure.

>[!NOTE]
>En el sistema operativo Linux:
> - Se admiten solo versiones de kernel SMP Linux y predeterminados.
> - Las versiones de kernel no estándar, como Physical Address Extension (PAE) y Xen, no son compatibles con ninguna distribución de Linux. Por ejemplo, un sistema con la cadena de versión *2.6.16.21-0.8-xen* no es compatible.
> - No se admiten los kernel personalizados, incluidas las recompilaciones de kernels estándar.
> - Se admite el kernel de CentOSPlus.
> - El kernel de Linux debe revisarse para la vulnerabilidad de Spectre. Consulte al proveedor de distribución de Linux para más información.

#### <a name="red-hat-linux-7"></a>Red Hat Linux 7

| Versión del SO | Versión del kernel |
|:--|:--|
| 7.6 | 3.10.0-957 |
| 7.5 | 3.10.0-862 |
| 7.4 | 3.10.0-693 |

#### <a name="red-hat-linux-6"></a>Red Hat Linux 6

| Versión del SO | Versión del kernel |
|:--|:--|
| 6.10 | 2.6.32-754 |
| 6.9 | 2.6.32-696 |

#### <a name="centosplus"></a>CentOSPlus

| Versión del SO | Versión del kernel |
|:--|:--|
| 6.10 | 2.6.32-754.3.5<br>2.6.32-696.30.1 |
| 6.9 | 2.6.32-696.30.1<br>2.6.32-696.18.7 |

#### <a name="ubuntu-server"></a>Ubuntu Server

| Versión del SO | Versión del kernel |
|:--|:--|
| 18,04 | 5.3.0-1020<br>5.0 (incluye kernel optimizado para Azure)<br>4.18 *<br>4.15* |
| 16.04.3 | 4.15.* |
| 16.04 | 4.13.\*<br>4.11.\*<br>4.10.\*<br>4.8.\*<br>4.4.\* |

#### <a name="suse-linux-12-enterprise-server"></a>SUSE Linux 12 Enterprise Server

| Versión del SO | Versión del kernel |
|:--|:--|
|12 SP4 | 4.12.* (incluye kernel optimizado para Azure) |
|12 SP3 | 4.4.* |
|12 SP2 | 4.4.* |

#### <a name="debian"></a>Debian 

| Versión del SO | Versión del kernel |
|:--|:--|
| 9 | 4,9 | 

## <a name="supported-azure-arc-machines"></a>Máquinas de Azure Arc admitidas
Azure Monitor para VM está disponible para los servidores habilitados para Azure Arc en las regiones donde el servicio de extensión de Arc está disponible. Debe ejecutar la versión 0 9 o superior del agente de Arc.

| Origen conectado | Compatible | Descripción |
|:--|:--|:--|
| Agentes de Windows | Sí | Junto con el [agente de Log Analytics para Windows](../platform/log-analytics-agent.md), los agentes de Windows requieren Dependency Agent. Para más información, consulte el artículo sobre los [sistemas operativos compatibles](#supported-operating-systems). |
| Agentes de Linux | Sí | Junto con el [agente de Log Analytics para Linux](../platform/log-analytics-agent.md), los agentes de Linux requieren Dependency Agent. Para más información, consulte el artículo sobre los [sistemas operativos compatibles](#supported-operating-systems). |
| Grupo de administración de System Center Operations Manager | No | |

## <a name="agents"></a>Agentes
Azure Monitor para VM requiere la instalación de los dos agentes siguientes en cada máquina virtual o conjunto de escalado de máquinas virtuales que se supervisarán. La instalación de estos agentes y su conexión al área de trabajo es el único requisito para incorporar el recurso.

- [Agente de Log Analytics](../platform/log-analytics-agent.md). Recopila eventos y datos de rendimiento de la máquina virtual o del conjunto de escalado de máquinas virtuales y los entrega al área de trabajo de Log Analytics. Los métodos de implementación para el agente de Log Analytics en los recursos de Azure usan la extensión de VM para [Windows](../../virtual-machines/extensions/oms-windows.md) y [Linux](../../virtual-machines/extensions/oms-linux.md).
- Dependency Agent. Recopila datos detectados acerca de los procesos que se ejecutan en las dependencias de las máquinas virtuales y los procesos externos, que usa la [característica de asignación de Azure Monitor para VM](vminsights-maps.md). Dependency Agent depende del agente de Log Analytics para entregar los datos a Azure Monitor. Los métodos de implementación de Dependency Agent en los recursos de Azure usan la extensión VM para [Windows](../../virtual-machines/extensions/agent-dependency-windows.md) y [Linux](../../virtual-machines/extensions/agent-dependency-linux.md).

> [!NOTE]
> El agente de Log Analytics es el mismo agente que se usa para System Center Operations Manager. Azure Monitor para VM puede supervisar agentes que también están supervisados por Operations Manager si están conectados directamente, e instala Dependency Agent en ellos. Azure Monitor para VM no puede supervisar los agentes conectados a Azure Monitor a través de una [conexión de grupo de administración](../tform/../platform/om-agents.md).

A continuación se muestran varios métodos para implementar estos agentes. 

| Método | Descripción |
|:---|:---|
| [Azure Portal](./vminsights-enable-portal.md) | Instale ambos agentes en una sola máquina virtual, en un conjunto de escalado de máquinas virtuales o en máquinas virtuales híbridas conectadas a Azure Arc. |
| [Plantillas de Resource Manager](vminsights-enable-powershell.md) | Instale ambos agentes con cualquiera de los métodos admitidos para implementar una plantilla de Resource Manager, como la CLI y PowerShell. |
| [Azure Policy](./vminsights-enable-policy.md) | Asigne la iniciativa de Azure Policy para instalar automáticamente los agentes cuando se crea una máquina virtual o un conjunto de escalado de máquinas virtuales. |
| [Instalación manual](./vminsights-enable-hybrid.md) | Instale los agentes en el sistema operativo invitado en equipos hospedados fuera de Azure, incluidos los de su centro de datos u otros entornos en la nube. |




## <a name="management-packs"></a>Módulos de administración
Cuando un área de trabajo de Log Analytics está configurada para Azure Monitor para VM, se reenvían dos módulo de administración a todos los equipos Windows conectados a esa área de trabajo. Los módulos de administración se denominan *Microsoft.IntelligencePacks.ApplicationDependencyMonitor* y *Microsoft.IntelligencePacks.VMInsights* y se escriben en *%Programfiles%\Microsoft Monitoring Agent\Agent\Health Service State\Management Packs\*. 

El origen de datos que utiliza el módulo de administración *ApplicationDependencyMonitor* es * *%Program files%\Microsoft Monitoring Agent\Agent\Health Service State\Resources\<AutoGeneratedID>\Microsoft.EnterpriseManagement.Advisor.ApplicationDependencyMonitorDataSource.dll*. El origen de datos que utiliza el módulo de administración *VMInsights* es *%Program files%\Microsoft Monitoring Agent\Agent\Health Service State\Resources\<AutoGeneratedID>\ Microsoft.VirtualMachineMonitoringModule.dll*.

## <a name="diagnostic-and-usage-data"></a>Datos de diagnóstico y uso

Microsoft recopila automáticamente datos de uso y rendimiento a través del servicio Azure Monitor. Microsoft usa estos datos para mejorar la calidad, la seguridad y la integridad del servicio. 

Para proporcionar funcionalidades de solución de problemas precisas y eficientes, la característica de asignación incluye datos sobre la configuración del software. Los datos proporcionan información como el sistema operativo y versión, la dirección IP, el nombre DNS y el nombre de la estación de trabajo. Microsoft no recopila nombres, direcciones ni otra información de contacto.

Para más información sobre el uso y la recopilación de datos, vea la [Declaración de privacidad de Microsoft Online Services](https://go.microsoft.com/fwlink/?LinkId=512132).

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="next-steps"></a>Pasos siguientes

Para aprender a usar la característica de supervisión Rendimiento, consulte [Visualización del rendimiento de Azure Monitor para VM](vminsights-performance.md). Para ver las dependencias de las aplicaciones detectadas, consulte [Uso de la asignación de Azure Monitor para VM (versión preliminar) para conocer los componentes de una aplicación](vminsights-maps.md).