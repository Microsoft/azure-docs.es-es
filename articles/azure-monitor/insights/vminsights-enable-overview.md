---
title: Información general sobre la habilitación de Azure Monitor para VM
description: Aprenda a implementar y configurar Azure Monitor para VM. Más información sobre los requisitos del sistema.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/27/2020
ms.openlocfilehash: 449979443577d22f8cc2ec35ec770dd1e107bb76
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998414"
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

Azure Monitor para VM admite cualquier sistema operativo que sea compatible con el agente de Log Analytics y el agente de dependencia. Consulte [Información general sobre los agentes de Azure Monitor](../platform/agents-overview.md#supported-operating-systems).

Consulte la siguiente lista de consideraciones sobre la compatibilidad de Linux del agente de dependencia que admite Azure Monitor para VM:

- Se admiten solo versiones de kernel SMP Linux y predeterminados.
- Las versiones de kernel no estándar, como Physical Address Extension (PAE) y Xen, no son compatibles con ninguna distribución de Linux. Por ejemplo, un sistema con la cadena de versión *2.6.16.21-0.8-xen* no es compatible.
- No se admiten los kernel personalizados, incluidas las recompilaciones de kernels estándar.
- Para distribuciones de Debian que no sean la versión 9.4, no se admite la característica de asignación y la característica de rendimiento solo está disponible en el menú de Azure Monitor. No está disponible directamente desde el panel izquierdo de la máquina virtual de Azure.
- Se admite el kernel de CentOSPlus.
- El kernel de Linux debe revisarse para la vulnerabilidad de Spectre. Consulte al proveedor de distribución de Linux para más información.



## <a name="supported-azure-arc-machines"></a>Máquinas de Azure Arc admitidas
Azure Monitor para VM está disponible para los servidores habilitados para Azure Arc en las regiones donde el servicio de extensión de Arc está disponible. Debe ejecutar la versión 0 9 o superior del agente de Arc.

| Origen conectado | Compatible | Descripción |
|:--|:--|:--|
| Agentes de Windows | Sí | Junto con el [agente de Log Analytics para Windows](../platform/log-analytics-agent.md), los agentes de Windows requieren Dependency Agent. Para más información, consulte el artículo sobre los [sistemas operativos compatibles](../platform/agents-overview.md#supported-operating-systems). |
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