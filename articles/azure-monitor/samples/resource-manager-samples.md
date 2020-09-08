---
title: Ejemplos de plantillas de Resource Manager para Azure Monitor
description: Implementación y configuración de características de Azure Monitor mediante plantillas de Resource Manager
author: bwren
ms.author: bwren
services: azure-monitor
ms.topic: sample
ms.date: 05/18/2020
ms.subservice: ''
ms.openlocfilehash: ab869fc8577d4d1934be96404ded5a2051237bbf
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87922735"
---
# <a name="resource-manager-template-samples-for-azure-monitor"></a>Ejemplos de plantillas de Resource Manager para Azure Monitor

Azure Monitor se puede implementar y configurar a gran escala mediante una [plantilla de Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md). Los artículos siguientes ofrecen plantillas de ejemplo para diferentes características de Azure Monitor. Estos ejemplos se pueden modificar con arreglo a sus requisitos y se pueden aplicar utilizando cualquier método estándar para implementar plantillas de Resource Manager. 

## <a name="deploying-the-sample-templates"></a>Implementación de plantillas de ejemplo
A continuación, se incluyen los pasos básicos para utilizar las plantillas de ejemplo:

1. Copie la plantilla y guárdela como un archivo JSON.
2. Modifique los parámetros de su entorno y guárdelos como un archivo JSON.
4. Implemente la plantilla mediante [cualquier método de implementación de plantillas de Resource Manager](../../azure-resource-manager/templates/deploy-powershell.md). 

Por ejemplo, use los siguientes comandos para implementar el archivo de plantilla y parámetros en un grupo de recursos mediante PowerShell o la CLI de Azure.


```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName my-subscription
New-AzResourceGroupDeployment -Name AzureMonitorDeployment -ResourceGroupName my-resource-group -TemplateFile azure-monitor-deploy.json -TemplateParameterFile azure-monitor-deploy.parameters.json
```

```azurecli
az login
az deployment group create \
    --name AzureMonitorDeployment \
    --resource-group ResourceGroupofTargetResource \
    --template-file azure-monitor-deploy.json \
    --parameters azure-monitor-deploy.parameters.json
```

## <a name="list-of-sample-templates"></a>Lista de plantillas de ejemplo

- [Agentes](resource-manager-agent.md): implemente y configure la extensión de diagnóstico y el agente de Log Analytics.
- Alertas
  - [Reglas de alertas de registro](resource-manager-alerts-log.md): alertas de las consultas de registros y del registro de actividad de Azure.
  - [Reglas de alertas de métricas](resource-manager-alerts-metric.md): alertas de métricas que usan diferentes tipos de lógica.
- [Application Insights](resource-manager-app-resource.md)
- [Configuración de diagnóstico](resource-manager-diagnostic-settings.md): cree una configuración de diagnóstico para reenviar los registros y las métricas desde diferentes tipos de recursos.
- [Consultas de registros](resource-manager-log-queries.md): cree consultas de registros guardadas, en un área de trabajo de Log Analytics.
- [Área de trabajo de Log Analytics](resource-manager-workspace.md): cree el área de trabajo Log Analytics y configure la recopilación de diferentes orígenes de datos a partir del agente de Log Analytics.
- [Libros](resource-manager-workbooks.md): cree libros.
- [Azure Monitor para contenedores](resource-manager-container-insights.md): incorpore clústeres a Azure Monitor para contenedores.
- [Azure Monitor para VM](resource-manager-vminsights.md): incorpore máquinas virtuales a Azure Monitor para VM.



## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre las [plantillas de Resource Manager](../../azure-resource-manager/templates/overview.md).