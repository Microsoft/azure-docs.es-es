---
title: Configuración del área de trabajo de Log Analytics para Azure Monitor para VM
description: Describe cómo crear y configurar el área de trabajo Log Analytics que usa Azure Monitor para VM.
ms.subservice: ''
ms.topic: conceptual
ms.custom: references_regions
author: bwren
ms.author: bwren
ms.date: 07/27/2020
ms.openlocfilehash: 40ff7bfe713f401a23dea0b3660be1b7728a1380
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87828564"
---
# <a name="configure-log-analytics-workspace-for-azure-monitor-for-vms"></a>Configuración del área de trabajo de Log Analytics para Azure Monitor para VM
Azure Monitor para VM recopila sus datos de una o varias áreas de trabajo de Log Analytics en Azure Monitor. Antes de incorporar agentes, debe crear y configurar un área de trabajo. En este artículo se describen los requisitos del área de trabajo y cómo configurarla para Azure Monitor para VM.

## <a name="overview"></a>Información general
Una suscripción única puede usar cualquier número de áreas de trabajo en función de sus requisitos. El único requisito del área de trabajo es que se encuentre en una ubicación admitida y esté configurada con la solución *VMInsights*.

Una vez configurada el área de trabajo, puede usar cualquiera de las opciones disponibles para instalar los agentes necesarios en la VM y VMMS, y especificar un área de trabajo para que envíen sus datos. Azure Monitor para VM recopilará datos de cualquier área de trabajo configurada en su suscripción.

> [!NOTE]
> Al habilitar Azure Monitor para VM en una única VM o en VMMS con Azure Portal, se le ofrece la opción de seleccionar un área de trabajo existente o crear una nueva. La solución *VMInsights* se instalará en este área de trabajo si aún no lo está. Después, puede usar este área de trabajo para otros agentes.


## <a name="create-log-analytics-workspace"></a>Creación de un área de trabajo de Log Analytics

>[!NOTE]
>La información que se describe en esta sección también se aplica a la [solución Service Map](service-map.md). 

Acceda a las áreas de trabajo de Log Analytics en Azure Portal desde el menú **Áreas de trabajo de Log Analytics**.

[![Áreas de trabajo de Log Analytics](media/vminsights-configure-workspace/log-analytics-workspaces.png)](media/vminsights-configure-workspace/log-analytics-workspaces.png#lightbox)

Puede crear una nueva área de trabajo de Log Analytics mediante cualquiera de los métodos siguientes. Consulte [Diseño de la implementación de registros de Azure Monitor](../platform/design-logs-deployment.md) para obtener instrucciones sobre cómo determinar el número de áreas de trabajo que debe usar en su entorno y cómo diseñar su estrategia de acceso.


* [Azure Portal](../../azure-monitor/learn/quick-create-workspace.md)
* [CLI de Azure](../../azure-monitor/learn/quick-create-workspace-cli.md)
* [PowerShell](../platform/powershell-workspace-configuration.md)
* [Azure Resource Manager](../../azure-monitor/platform/template-workspace-configuration.md)

## <a name="supported-regions"></a>Regiones admitidas

Azure Monitor para VM admite áreas de trabajo de Log Analytics en las siguientes regiones, aunque puede supervisar las máquinas virtuales en cualquier región. Las máquinas virtuales en sí no se limitan a las regiones admitidas por el área de trabajo de Log Analytics.

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

## <a name="role-based-access-control"></a>Control de acceso basado en rol
Para habilitar y obtener acceso a las características de Azure Monitor para VM, debe tener el rol de [colaborador de Log Analytics](../platform/manage-access.md#manage-access-using-azure-permissions) en el área de trabajo. Para ver los datos de rendimiento, mantenimiento y el mapa, debe tener el [rol de lector de supervisión](../platform/roles-permissions-security.md#built-in-monitoring-roles) en la máquina virtual de Azure. Para más información acerca de cómo controlar el acceso a un área de trabajo de Log Analytics, consulte [Administración de áreas de trabajo](../platform/manage-access.md).

## <a name="add-vminsights-solution-to-workspace"></a>Incorporación de la solución VMInsights al área de trabajo
Antes de que se pueda utilizar un área de trabajo de Log Analytics con Azure Monitor para VM, debe tener instalada la solución *VMInsights*. Los métodos para configurar el área de trabajo se describen en las secciones siguientes.

> [!NOTE]
> Al agregar la solución *VMInsights* al área de trabajo, todas las máquinas virtuales existentes conectadas al área de trabajo comenzarán a enviar datos a InsightsMetrics. Los datos del resto de tipos de datos no se recopilarán hasta que agregue Dependency Agent a las máquinas virtuales existentes conectadas al área de trabajo.

### <a name="azure-portal"></a>Azure portal
Existen tres opciones para configurar un área de trabajo existente desde Azure Portal.

Para configurar una única área de trabajo, seleccione **Otras opciones de incorporación** y, a continuación, **Configurar un área de trabajo**. Seleccione una suscripción y un área de trabajo y, a continuación, haga clic en **Configurar**.

[![Configuración del área de trabajo](media/vminsights-enable-at-scale-policy/configure-workspace.png)](media/vminsights-enable-at-scale-policy/configure-workspace.png#lightbox)

Para configurar varias áreas de trabajo, seleccione la pestaña **Configuración del área de trabajo** en el menú **Máquinas virtuales** del menú **Supervisión** de Azure Portal. Establezca los valores de filtro para mostrar una lista de áreas de trabajo existentes. Marque la casilla situada junto a cada área de trabajo que quiera habilitar y, a continuación, haga clic en **Configurar selección**.

[![Configuración del área de trabajo](media/vminsights-enable-at-scale-policy/workspace-configuration.png)](media/vminsights-enable-at-scale-policy/workspace-configuration.png#lightbox)


Al habilitar Azure Monitor para VM en una única VM o en VMMS con Azure Portal, se le ofrece la opción de seleccionar un área de trabajo existente o crear una nueva. La solución *VMInsights* se instalará en este área de trabajo si aún no lo está. Después, puede usar este área de trabajo para otros agentes.

[![Habilitación de una única VM en el portal](media/vminsights-enable-single-vm/enable-vminsights-vm-portal.png)](media/vminsights-enable-single-vm/enable-vminsights-vm-portal.png#lightbox)


### <a name="resource-manager-template"></a>Plantilla de Resource Manager
Las plantillas de Azure Resource Manager para Azure Monitor para VM se suministran en un archivo de almacenamiento (.zip) que puede [descargar desde nuestro repositorio de GitHub](https://aka.ms/VmInsightsARMTemplates). Incluye una plantilla llamada **ConfigureWorkspace** que configura un área de trabajo de Log Analytics para Azure Monitor para VM. Esta plantilla se implementa mediante cualquiera de los métodos estándar, incluidos los comandos de PowerShell y de la CLI de ejemplo siguientes: 

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az deployment group create --name ConfigureWorkspace --resource-group my-resource-group --template-file CreateWorkspace.json  --parameters workspaceResourceId='/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/my-resource-group/providers/microsoft.operationalinsights/workspaces/my-workspace' workspaceLocation='eastus'

```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
New-AzResourceGroupDeployment -Name ConfigureWorkspace -ResourceGroupName my-resource-group -TemplateFile ConfigureWorkspace.json -workspaceResourceId /subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/my-resource-group/providers/microsoft.operationalinsights/workspaces/my-workspace -location eastus
```

---



## <a name="next-steps"></a>Pasos siguientes
- Consulte [Incorporación de agentes a Azure Monitor para VM](vminsights-enable-overview.md) para conectar agentes a Azure Monitor para VM.
- Consulte [Soluciones de supervisión como destino en Azure Monitor (versión preliminar)](solution-targeting.md) para limitar la cantidad de datos que se envían desde una solución al área de trabajo.