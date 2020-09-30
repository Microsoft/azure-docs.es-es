---
title: Información general del agente de Azure Monitor
description: Información general del agente de Azure Monitor (AMA), que recopila datos de supervisión del sistema operativo invitado de máquinas virtuales.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/10/2020
ms.openlocfilehash: bf4974fdd0c587bd537c63a454b5d5efeb54415b
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89378498"
---
# <a name="azure-monitor-agent-overview-preview"></a>Información general del agente de Azure Monitor (versión preliminar)
El agente de Azure Monitor (AMA) recopila datos de supervisión del sistema operativo invitado de máquinas virtuales y los entrega a Azure Monitor. En este artículo se proporciona información general sobre el agente de Azure Monitor, incluido cómo instalarlo y cómo configurar la recopilación de datos.

## <a name="relationship-to-other-agents"></a>Relación con otros agentes
El agente de Azure Monitor reemplaza a los siguientes agentes utilizados actualmente por Azure Monitor para recopilar datos de invitado de máquinas virtuales:

- [Agente de Log Analytics](log-analytics-agent.md): Envía datos al área de trabajo de Log Analytics y admite Azure Monitor para VM y soluciones de supervisión.
- [Extensión de Diagnostics](diagnostics-extension-overview.md): Envía datos a Azure Storage, a las métricas de Azure Monitor (solo en Windows), a Event Hubs y a Azure Storage.
- [Agente de Telegraf](collect-custom-metrics-linux-telegraf.md): Envía datos a las métricas de Azure Monitor (solo Linux).

Además de consolidar esta funcionalidad en un solo agente, el agente de Azure Monitor proporciona las siguientes ventajas frente a los agentes existentes:

- Ámbito de supervisión. Configure de forma centralizada la recopilación para diferentes conjuntos de datos de diferentes conjuntos de VM.  
- Hospedaje múltiple en Linux: Envíe datos de VM Linux a varias áreas de trabajo.
- Filtrado de eventos de Windows: Use consultas de XPATCH para filtrar los eventos de Windows que se recopilan.
- Administración mejorada de las extensiones: El agente de Azure Monitor utiliza un nuevo método de control de la extensibilidad que es más transparente y controlable que los módulos de administración y los complementos de Linux en los agentes de Log Analytics actuales.

### <a name="changes-in-data-collection"></a>Cambios en la recopilación de datos
Los métodos para definir la recopilación de datos de los agentes existentes son distintos entre sí, y cada uno de ellos tiene desafíos que se abordan con el agente de Azure Monitor.

- El agente de Log Analytics obtiene su configuración de un área de trabajo de Log Analytics. Esto es fácil de configurar de forma centralizada, pero es difícil establecer definiciones independientes para diferentes máquinas virtuales. Solo puede enviar datos a un área de trabajo de Log Analytics.

- La extensión de Diagnostics tiene una configuración para cada máquina virtual. Resulta fácil establecer definiciones independientes para diferentes máquinas virtuales, pero difícil de administrar de forma centralizada. Solo puede enviar datos a métricas de Azure Monitor, Azure Event Hubs o Azure Storage. En el caso de los agentes de Linux, se necesita el agente de Telegraf de código abierto para enviar datos a las métricas de Azure Monitor.

El agente de Azure Monitor utiliza las [reglas de recopilación de datos (DCR)](data-collection-rule-overview.md) para configurar los datos que se van a recopilar de cada agente. Las reglas de recopilación de datos permiten la capacidad de administración de la configuración de la recopilación a escala y, al mismo tiempo, habilitan configuraciones únicas con ámbito para subconjuntos de máquinas. Son independientes del área de trabajo e independientes de la máquina virtual, lo que permite definirlas una vez y reutilizarlas en distintas máquinas y entornos. Consulte [Configuración de la recopilación de datos para el agente de Azure Monitor (versión preliminar)](data-collection-rule-azure-monitor-agent.md).



## <a name="current-limitations"></a>Limitaciones actuales
Durante la versión preliminar pública del agente de Azure Monitor se aplican las siguientes limitaciones:

- El agente de Azure Monitor no admite soluciones y conclusiones, como Azure Monitor para VM y Azure Security Center. El único escenario que se admite actualmente es la recopilación de datos con las reglas de recopilación de datos que usted configure. 
- Las reglas de recopilación de datos se deben crear en la misma región que cualquier área de trabajo de Log Analytics que se use como destino.
- Actualmente solo se admiten máquinas virtuales de Azure. No se admiten actualmente máquinas virtuales locales, conjuntos de escalado de máquinas virtuales, ARC para servidores, Azure Kubernetes Service ni otros tipos de recursos de proceso.
- La máquina virtual debe tener acceso a los siguientes puntos de conexión HTTPS:
  - *.ods.opinsights.azure.com
  - *.ingest.monitor.azure.com
  - *.control.monitor.azure.com


## <a name="coexistence-with-other-agents"></a>Coexistencia con otros agentes
El agente de Azure Monitor puede coexistir con los agentes existentes para que pueda seguir usando su funcionalidad existente durante la evaluación o la migración. Esto es especialmente importante debido a las limitaciones de la versión preliminar pública para admitir las soluciones existentes. Sin embargo, debe tener cuidado al recopilar datos duplicados, ya que esto podría sesgar los resultados de las consultas y generar cargos adicionales por la ingesta y retención de datos.

Por ejemplo, Azure Monitor para VM usa el agente de Log Analytics para enviar datos de rendimiento a un área de trabajo de Log Analytics. También puede haber configurado el área de trabajo para recopilar eventos de Windows y eventos de Syslog de los agentes. Si instala el agente de Azure Monitor y crea una regla de recopilación de datos para estos mismos eventos y datos de rendimiento, se generarán datos duplicados.


## <a name="costs"></a>Costos
El agente de Azure Monitor no cuesta nada, pero puede incurrir en gastos por los datos ingeridos. Consulte [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/) para obtener más información sobre la recopilación y retención de datos de Log Analytics, y para las métricas de los clientes.

## <a name="data-sources-and-destinations"></a>Orígenes y destinos de los datos
En la tabla siguiente se enumeran los tipos de datos que se pueden recopilar actualmente con el agente de Azure Monitor mediante reglas de recopilación de datos y a dónde se pueden enviar los datos. Consulte [¿Qué supervisa Azure Monitor?](../monitor-reference.md) para ver una lista con información detallada, soluciones y otros servicios que usan el agente de Azure Monitor para recopilar otros tipos de datos.


El agente de Azure Monitor envía datos a métricas de Azure Monitor o a un área de trabajo de Log Analytics que admite registros de Azure Monitor.

| Origen de datos | Destinations | Descripción |
|:---|:---|:---|
| Rendimiento        | Métricas de Azure Monitor<br>Área de trabajo de Log Analytics | Valores numéricos que miden el rendimiento de diferentes aspectos del sistema operativo y las cargas de trabajo. |
| Registros de eventos de Windows | Área de trabajo de Log Analytics | Información enviada al sistema de registro de eventos de Windows. |
| syslog             | Área de trabajo de Log Analytics | Información enviada al sistema de registro de eventos de Windows. |


## <a name="supported-operating-systems"></a>Sistemas operativos admitidos
Los siguientes sistemas operativos son compatibles actualmente con el agente de Azure Monitor.

### <a name="windows"></a>Windows 
  - Windows Server 2019
  - Windows Server 2016
  - Windows Server 2012
  - Windows Server 2012 R2

### <a name="linux"></a>Linux
  - CentOS 6<sup>1</sup>, 7
  - Debian 9, 10
  - Oracle Linux 6<sup>1</sup>, 7
  - RHEL 6<sup>1</sup>, 7, 8
  - SLES 11, 12, 15
  - Ubuntu 14.04 LTS, 16.04 LTS, 18.04 LTS

> [!IMPORTANT]
> <sup>1</sup>Para que estas distribuciones envíen datos de Syslog, es necesario reiniciar el servicio rsyslog una vez después de haber instalado el agente.


## <a name="security"></a>Seguridad
El agente de Azure Monitor no requiere ninguna clave, pero en su lugar requiere una [identidad administrada asignada por el sistema](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md#system-assigned-managed-identity). Debe tener una identidad administrada asignada por el sistema habilitada en cada máquina virtual antes de implementar el agente.


## <a name="install-the-azure-monitor-agent"></a>Instalación del agente de Azure Monitor
El agente de Azure Monitor se implementa como [extensión de VM de Azure](../../virtual-machines/extensions/overview.md) con los detalles de la tabla siguiente. 

| Propiedad | Windows | Linux |
|:---|:---|:---|
| Publicador | Microsoft.Azure.Monitor  | Microsoft.Azure.Monitor |
| Tipo      | AzureMonitorWindowsAgent | AzureMonitorLinuxAgent  |
| TypeHandlerVersion  | 1.0 | 1.5 |

Instale el agente de Azure Monitor con cualquiera de los métodos para instalar los agentes de máquina virtual, incluidos los siguientes con PowerShell o la CLI. Como alternativa, puede instalar el agente y configurar la recopilación de datos en máquinas virtuales de su suscripción a Azure desde el portal con el procedimiento descrito en [Configuración de la recopilación de datos para el agente de Azure Monitor (versión preliminar)](data-collection-rule-azure-monitor-agent.md#create-using-the-azure-portal).

### <a name="windows"></a>Windows

# <a name="cli"></a>[CLI](#tab/CLI1)

```azurecli
az vm extension set --name AzureMonitorWindowsAgent --publisher Microsoft.Azure.Monitor --ids {resource ID of the VM}

```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell1)

```powershell
Set-AzVMExtension -Name AMAWindows -ExtensionType AzureMonitorWindowsAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName {Resource Group Name} -VMName {VM name} -Location eastus
```
---


### <a name="linux"></a>Linux

# <a name="cli"></a>[CLI](#tab/CLI2)

```azurecli
az vm extension set --name AzureMonitorLinuxAgent --publisher Microsoft.Azure.Monitor --ids {resource ID of the VM}

```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell2)

```powershell
Set-AzVMExtension -Name AMALinux -ExtensionType AzureMonitorLinuxAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName {Resource Group Name} -VMName {VM name} -Location eastus
```
---

## <a name="next-steps"></a>Pasos siguientes

- [Cree una regla de recopilación de datos](data-collection-rule-azure-monitor-agent.md) para recopilar datos del agente y enviarlos a Azure Monitor.
