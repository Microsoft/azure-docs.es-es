---
title: Habilitación de Azure Monitor para VM mediante plantillas de Resource Manager
description: En este artículo se describe cómo habilitar Azure Monitor para VM para una o más máquinas virtuales o conjuntos de escalado de máquinas virtuales de Azure mediante Azure PowerShell o plantillas de Azure Resource Manager.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/27/2020
ms.openlocfilehash: 89a9a1b762e02237a8ee08dca5d6eedefabaafbb
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87328028"
---
# <a name="enable-azure-monitor-for-vms-using-resource-manager-templates"></a>Habilitación de Azure Monitor para VM mediante plantillas de Resource Manager
En este artículo se describe cómo habilitar Azure Monitor para VM para una máquina virtual o un conjunto de escalado de máquinas virtuales mediante plantillas de Resource Manager. Este procedimiento se puede usar para:

- Máquina virtual de Azure
- Conjunto de escalado de máquinas virtuales de Azure
- Máquina virtual híbrida conectada con Azure Arc

## <a name="prerequisites"></a>Requisitos previos

- [Cree y configure un área de trabajo de Log Analytics](vminsights-configure-workspace.md). 
- Vea [Sistemas operativos admitidos](vminsights-enable-overview.md#supported-operating-systems) para asegurarse de que el sistema operativo de la máquina virtual o el conjunto de escalado de máquinas virtuales que va a habilitar son compatibles. 

## <a name="resource-manager-templates"></a>Plantillas de Resource Manager

Hemos creado plantillas de Azure Resource Manager de ejemplo para incorporar sus máquinas virtuales y conjuntos de escalado de máquinas virtuales. Estas plantillas incluyen escenarios que puede usar para habilitar la supervisión de un recurso existente y para crear un nuevo recurso que tenga la supervisión habilitada.

>[!NOTE]
>La plantilla debe implementarse en el mismo grupo de recursos que la máquina virtual o el conjunto de escalado de máquinas virtuales que se van a habilitar.


Las plantillas de Azure Resource Manager se suministran en un archivo de almacenamiento (.zip) que puede [descargar](https://aka.ms/VmInsightsARMTemplates) desde nuestro repositorio de GitHub. El contenido del archivo incluye carpetas que representan cada escenario de implementación con un archivo de plantilla y un archivo de parámetros. Antes de ejecutarlos, modifique el archivo de parámetros y especifique los valores necesarios. 

El archivo de descarga contiene las siguientes plantillas para distintos escenarios:

- La plantilla **ExistingVmOnboarding** habilita Azure Monitor para VM si la máquina virtual ya existe.
- La plantilla **NewVmOnboarding** crea una máquina virtual y habilita Azure Monitor para VM para que la supervise.
- La plantilla **ExistingVmssOnboarding** habilita Azure Monitor para VM si el conjunto de escalado de máquinas virtuales ya existe.
- La plantilla **NewVmssOnboarding** crea conjuntos de escalado de máquinas virtuales y habilita Azure Monitor para VM para que lo supervise.
- La plantilla **ConfigureWorkspace** configura el área de trabajo de Log Analytics para que admita Azure Monitor para VM habilitando las soluciones y la colección de contadores de rendimiento de los sistemas operativos Linux y Windows.

>[!NOTE]
>Si ya había conjuntos de escalado de máquinas virtuales y la directiva de actualización está establecida en **Manual**, Azure Monitor para VM no se habilitará para las instancias de manera predeterminada tras ejecutar la plantilla de Azure Resource Manager **ExistingVmssOnboarding**. Tendrá que actualizar manualmente las instancias.

## <a name="deploy-templates"></a>Implementación de plantillas
Las plantillas se pueden implementar mediante [cualquier método de implementación de plantillas de Resource Manager](../../azure-resource-manager/templates/deploy-powershell.md), incluidos los ejemplos siguientes con PowerShell y la CLI.

```powershell
New-AzResourceGroupDeployment -Name OnboardCluster -ResourceGroupName <ResourceGroupName> -TemplateFile <Template.json> -TemplateParameterFile <Parameters.json>
```


```azurecli
az group deployment create --resource-group <ResourceGroupName> --template-file <Template.json> --parameters <Parameters.json>
```



## <a name="next-steps"></a>Pasos siguientes

Ahora que la supervisión está habilitada para las máquinas virtuales, esta información está disponible para analizarse con Azure Monitor para VM.

- Para ver las dependencias de las aplicaciones detectadas, consulte [Uso de la asignación de Azure Monitor para VM (versión preliminar) para conocer los componentes de una aplicación](vminsights-maps.md).

- Para identificar los cuellos de botella y el uso general con el rendimiento de la máquina virtual, vea [Cómo representar el rendimiento en gráficos con Azure Monitor para VM (versión preliminar)](vminsights-performance.md).
