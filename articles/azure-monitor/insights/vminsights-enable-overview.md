---
title: Habilitar Monitor de Azure para información general de las máquinas virtuales (versión preliminar) | Microsoft Docs
description: En este artículo se describe cómo implementar y configurar a Azure Monitor para máquinas virtuales y los requisitos del sistema necesarios.
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/09/2019
ms.author: magoedte
ms.openlocfilehash: 4eb07b29f68c5f495bdbe5e23f5b226480b66661
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65524076"
---
# <a name="enable-azure-monitor-for-vms-preview-overview"></a>Habilitar a Monitor de Azure para información general de las máquinas virtuales (versión preliminar)

En este artículo proporciona información general sobre las opciones disponibles para configurar a Azure Monitor para supervisar el estado, rendimiento y detectar dependencias de aplicaciones que se ejecutan en máquinas virtuales y conjuntos de escalado de máquinas virtuales de las máquinas virtuales locales, máquinas virtuales o máquinas virtuales hospedan en otro entorno de nube.  

Para habilitar Azure Monitor para VM, puede seguir alguno de los métodos siguientes:

* Habilitar una máquina virtual de Azure o el escalado de máquinas virtuales que se establece mediante la selección **Insights (versión preliminar)** directamente desde la escala de máquina virtual o máquina virtual establecido.
* Habilitar dos o más máquinas virtuales de Azure y máquina virtual de conjuntos de escalado mediante la directiva de Azure. Mediante este método, las dependencias necesarias de las máquinas virtuales nuevas y existentes y conjuntos de escalado están instaladas y configuradas correctamente. Conjuntos de escalado y máquinas virtuales no compatible se notifican para que pueda decidir si desea habilitarlas y cómo desea corregirlas.
* Use PowerShell para habilitar dos o más máquinas virtuales o conjuntos de escalado de máquinas virtuales de Azure en una suscripción o grupo de recursos concreto.
* Habilitar para supervisar las máquinas virtuales o equipos físicos que se hospeda en la red corporativa o en otro entorno de nube.

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar, asegúrese de conocer la información de los apartados siguientes.

### <a name="log-analytics"></a>Log Analytics

Azure Monitor para las máquinas virtuales es compatible con un área de trabajo de Log Analytics en las siguientes regiones:

- Centro occidental de EE.UU.
- Este de EE. UU
- Canada Central<sup>1</sup>
- Sur de Reino Unido<sup>1</sup>
- Europa occidental
- Sudeste Asiático<sup>1</sup>

<sup>1</sup> Actualmente, esta región no admite la característica de estado de Azure Monitor para VM.

>[!NOTE]
>Se pueden implementar máquinas virtuales de Azure desde cualquier región, sin limitarse a las admitidas en el área de trabajo de Log Analytics.
>

Si no tiene un área de trabajo, puede crear una con alguno de los métodos siguientes:
* [La CLI de Azure](../../azure-monitor/learn/quick-create-workspace-cli.md)
* [PowerShell](../../azure-monitor/learn/quick-create-workspace-posh.md)
* [Portal de Azure](../../azure-monitor/learn/quick-create-workspace.md)
* [Azure Resource Manager](../../azure-monitor/platform/template-workspace-configuration.md)

Si se habilita la supervisión de una sola máquina virtual de Azure o escalado de máquinas virtuales que se establezca en el portal de Azure, puede crear un área de trabajo durante este proceso.

Para el escenario a escala mediante la directiva de Azure, Azure PowerShell o plantillas de Azure Resource Manager, el área de trabajo de Log Analytics debe en primer lugar la siguiente configuración:

* Instale las soluciones ServiceMap e InfrastructureInsights. Puede completar esta instalación mediante el uso de una plantilla de Azure Resource Manager proporcionada o mediante el **configurar área de trabajo** opción se encuentra en el **comenzar** ficha.
* Configure el área de trabajo de Log Analytics para que recopile contadores de rendimiento.

Para configurar el área de trabajo para el escenario a escala, puede configurarlo mediante uno de los métodos siguientes:

* [Azure PowerShell](vminsights-enable-at-scale-powershell.md#set-up-a-log-analytics-workspace)
* Desde el **configurar área de trabajo** opción en el Monitor de Azure para máquinas virtuales [directiva cobertura](vminsights-enable-at-scale-policy.md#manage-policy-coverage-feature-overview) página

### <a name="supported-operating-systems"></a>Sistemas operativos compatibles

La siguiente es una lista de los sistemas operativos Windows y Linux que son compatibles oficialmente con Azure Monitor para máquinas virtuales. Más adelante en esta sección, encontrará una lista completa que detalla las versiones de kernel admitidas y las versiones de sistema operativo Linux principales y secundarias.

|Versión del SO |Rendimiento |Mapas |Health |
|-----------|------------|-----|-------|
|Windows Server 2019 | X | X | |
|Windows Server 2016 1803 | X | X | X |
|Windows Server 2016 | X | X | X |
|Windows Server 2012 R2 | X | X | |
|Windows Server 2012 | X | X | |
|Windows Server 2008 R2 | X | X| |
|Red Hat Enterprise Linux (RHEL) 6, 7| X | X| X |
|Ubuntu 14.04, 16.04, 18.04 | X | X | X |
|CentOS Linux 6, 7 | X | X | X |
|SUSE Linux Enterprise Server (SLES) 11, 12 | X | X | X |
|Debian 8, 9.4 | X<sup>1</sup> | | X |

<sup>1</sup> La característica Rendimiento de Azure Monitor para VM solo está disponible desde Azure Monitor. No está disponible cuando tiene acceso a ella directamente desde el panel izquierdo de la máquina virtual de Azure.

>[!NOTE]
>La siguiente información se aplica a la compatibilidad del sistema operativo Linux:
> - Se admiten solo versiones de kernel SMP Linux y predeterminados.
> - Las versiones de kernel no estándar, como Physical Address Extension (PAE) y Xen, no son compatibles con ninguna distribución de Linux. Por ejemplo, un sistema con la cadena de versión *2.6.16.21-0.8-xen* no es compatible.
> - No se admiten los kernel personalizados, incluidas las recompilaciones de kernels estándar.
> - Se admite el kernel de CentOSPlus.

#### <a name="red-hat-linux-7"></a>Red Hat Linux 7

| Versión del SO | Versión del kernel |
|:--|:--|
| 7.4 | 3.10.0-693 |
| 7.5 | 3.10.0-862 |
| 7.6 | 3.10.0-957 |

#### <a name="red-hat-linux-6"></a>Red Hat Linux 6

| Versión del SO | Versión del kernel |
|:--|:--|
| 6.9 | 2.6.32-696 |
| 6.10 | 2.6.32-754 |

### <a name="centosplus"></a>CentOSPlus
| Versión del SO | Versión del kernel |
|:--|:--|
| 6.9 | 2.6.32-696.18.7<br>2.6.32-696.30.1 |
| 6.10 | 2.6.32-696.30.1<br>2.6.32-754.3.5 |

#### <a name="ubuntu-server"></a>Ubuntu Server

| Versión del SO | Versión del kernel |
|:--|:--|
| Ubuntu 18.04 | kernel 4.15.* |
| Ubuntu 16.04.3 | kernel 4.15.* |
| 16.04 | 4.4.\*<br>4.8.\*<br>4.10.\*<br>4.11.\*<br>4.13.\* |
| 14.04 | 3.13.\*<br>4.4.\* |

#### <a name="suse-linux-11-enterprise-server"></a>SUSE Linux 11 Enterprise Server

| Versión del SO | Versión del kernel
|:--|:--|
|11 SP4 | 3.0.* |

#### <a name="suse-linux-12-enterprise-server"></a>SUSE Linux 12 Enterprise Server

| Versión del SO | Versión del kernel
|:--|:--|
|12 SP2 | 4.4.* |
|12 SP3 | 4.4.* |

### <a name="the-microsoft-dependency-agent"></a>Microsoft Dependency Agent

La característica Service Map de Azure Monitor para VM obtiene sus datos de Microsoft Dependency Agent. Dependency Agent depende del agente de Log Analytics en lo que respecta a sus conexiones a Log Analytics. Por tanto, el sistema debe tener instalado y configurado el agente de Log Analytics con Dependency Agent.

Cuando se habilita Azure Monitor para VM para una sola máquina virtual de Azure o cuando se usa el método de implementación a escala, es necesario usar la extensión Dependency Agent de Azure VM para instalar el agente como parte de la experiencia.

En un entorno híbrido, puede descargar e instalar el agente de dependencia en cualquiera de estas dos maneras: manualmente o mediante un método de implementación automatizada para máquinas virtuales que están hospedadas fuera de Azure.

En la tabla siguiente se describen los orígenes conectados que son compatibles con la característica Asignación en un entorno híbrido.

| Origen conectado | Compatible | DESCRIPCIÓN |
|:--|:--|:--|
| Agentes de Windows | Sí | Además del [agente de Log Analytics para Windows](../../azure-monitor/platform/log-analytics-agent.md), los agentes de Windows requieren Microsoft Dependency Agent. Para obtener una lista completa de las versiones del sistema operativo, consulte los [sistemas operativos compatibles](#supported-operating-systems). |
| Agentes de Linux | Sí | Además del [agente de Log Analytics para Linux](../../azure-monitor/platform/log-analytics-agent.md), los agentes de Linux requieren Microsoft Dependency Agent. Para obtener una lista completa de las versiones del sistema operativo, consulte los [sistemas operativos compatibles](#supported-operating-systems). |
| Grupo de administración de System Center Operations Manager | No | |

Dependency Agent se puede descargar desde las ubicaciones siguientes:

| Archivo | SO | Versión | SHA-256 |
|:--|:--|:--|:--|
| [InstallDependencyAgent-Windows.exe](https://aka.ms/dependencyagentwindows) |  Windows | 9.8.1 | 622C99924385CBF539988D759BCFDC9146BB157E7D577C997CDD2674E27E08DD |
| [InstallDependencyAgent-Linux64.bin](https://aka.ms/dependencyagentlinux) | Linux | 9.8.1 | 3037934A5D3FB7911D5840A9744AE9F980F87F620A7F7B407F05E276FE7AE4A8 |

## <a name="role-based-access-control"></a>Control de acceso basado en rol

Para habilitar las características de Azure Monitor para VM y acceder a ellas, debe tener asignados los siguientes roles de acceso:

- Para habilitarla, debe tener la *colaborador de Log Analytics* rol.

- Para ver los datos de rendimiento, mantenimiento y el mapa, debe tener el rol de *lector de supervisión* en la máquina virtual de Azure. El área de trabajo de Log Analytics debe configurarse para Azure Monitor para VM.

Para más información acerca de cómo controlar el acceso a un área de trabajo de Log Analytics, consulte [Administración de áreas de trabajo](../../azure-monitor/platform/manage-access.md).

## <a name="how-to-enable-azure-monitor-for-vms-preview"></a>Cómo habilitar a Azure Monitor para las máquinas virtuales (versión preliminar)

Habilitar a Monitor de Azure para máquinas virtuales mediante uno de los métodos siguientes que se describe en la tabla siguiente.

| Estado de implementación | Método | DESCRIPCIÓN |
|------------------|--------|-------------|
| Conjunto de escalado de máquina virtual de Azure o una máquina virtual único | [Directamente desde la máquina virtual](vminsights-enable-single-vm.md) | Puede habilitar una sola máquina virtual de Azure seleccionando **Insights (versión preliminar)** directamente desde la escala de máquina virtual o máquina virtual establecido. |
| Varias máquinas virtuales de Azure o conjuntos de escalado de máquinas virtuales | [Azure Policy](vminsights-enable-at-scale-policy.md) | Puede permitir que varias máquinas virtuales de Azure con Azure Policy y definiciones de directivas disponibles. |
| Varias máquinas virtuales de Azure o conjuntos de escalado de máquinas virtuales | [Plantillas de Azure PowerShell o Azure Resource Manager](vminsights-enable-at-scale-powershell.md) | Puede habilitar varios conjuntos de escalado de máquinas virtuales de Azure o una máquina virtual a través de un grupo de recursos o suscripción especificado mediante plantillas de Azure PowerShell o Azure Resource Manager. |
| Nube híbrida | [Para habilitar el entorno híbrido](vminsights-enable-hybrid-cloud.md) | Puede implementar en máquinas virtuales o equipos físicos que se hospedan en su centro de datos u otros entornos de nube. |

## <a name="performance-counters-enabled"></a>Contadores de rendimiento habilitados

Azure Monitor para las máquinas virtuales configura un área de trabajo de Log Analytics para recopilar los contadores de rendimiento que utiliza. En la tabla siguiente se enumera los objetos y contadores recopilados cada 60 segundos.

### <a name="windows-performance-counters"></a>Contadores de rendimiento de Windows

|Nombre de objeto |Nombre del contador |
|------------|-------------|
|LogicalDisk |% de espacio libre |
|LogicalDisk |Prom. Segundos de disco/lecturas |
|LogicalDisk |Prom. Segundos de disco/transferencias |
|LogicalDisk |Prom. Segundos de disco/escrituras |
|LogicalDisk |Bytes de disco/s |
|LogicalDisk |Bytes de lectura de disco/s |
|LogicalDisk |Lecturas de disco/s  |
|LogicalDisk |Transferencias de disco/s |
|LogicalDisk |Bytes de escritura en disco/s |
|LogicalDisk | Escrituras en disco/s |
|LogicalDisk |Megabytes libres |
|Memoria |MB disponibles |
|Adaptador de red |Bytes recibidos/s |
|Adaptador de red |Bytes enviados/seg. |
|Procesador |% de tiempo de procesador |

### <a name="linux-performance-counters"></a>Contadores de rendimiento de Linux

|Nombre de objeto |Nombre del contador |
|------------|-------------|
|Disco lógico |% espacio usado |
|Disco lógico |Bytes de lectura de disco/s |
|Disco lógico |Lecturas de disco/s  |
|Disco lógico |Transferencias de disco/s |
|Disco lógico |Bytes de escritura en disco/s |
|Disco lógico | Escrituras en disco/s |
|Disco lógico |Megabytes libres |
|Disco lógico |Bytes de disco lógico/s |
|Memoria |MB de memoria disponibles |
|Red |Número total de bytes recibidos |
|Red |Número total de bytes transmitidos |
|Procesador |% de tiempo de procesador |

## <a name="diagnostic-and-usage-data"></a>Datos de uso y diagnóstico

Microsoft recopila automáticamente datos de uso y rendimiento a través del servicio Azure Monitor. Microsoft usa estos datos para proporcionar y mejorar la calidad, la seguridad y la integridad del servicio. Con el fin de proporcionar funcionalidades de solución de problemas precisas y eficientes, los datos de la característica Mapa incluyen información sobre la configuración del software, como el sistema operativo y la versión, la dirección IP, el nombre DNS y el nombre de la estación de trabajo. Microsoft no recopila nombres, direcciones ni otra información de contacto.

Para más información sobre el uso y la recopilación de datos, vea la [Declaración de privacidad de Microsoft Online Services](https://go.microsoft.com/fwlink/?LinkId=512132).

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="next-steps"></a>Pasos siguientes

Ahora que la supervisión está habilitada para la máquina virtual, esta información está disponible para analizarse con Azure Monitor para VM. Para obtener información sobre cómo usar la característica de mantenimiento, consulte [Descripción del estado de las máquinas virtuales de Azure con Azure Monitor para VM (versión preliminar)](vminsights-health.md). Para ver las dependencias de las aplicaciones detectadas, consulte [Uso de la asignación de Azure Monitor para VM (versión preliminar) para conocer los componentes de una aplicación](vminsights-maps.md).
