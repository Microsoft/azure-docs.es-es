---
title: Cómo habilitar Azure Monitor para contenedores | Microsoft Docs
description: Este artículo describe cómo habilitar y configurar Azure Monitor para los contenedores para que pueda comprender cómo funciona el contenedor y se han identificado los problemas relacionados con el rendimiento.
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
ms.date: 04/25/2019
ms.author: magoedte
ms.openlocfilehash: 5e149fa96e0a62656804906b52adf10273321d17
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65521897"
---
# <a name="how-to-enable-azure-monitor-for-containers"></a>Cómo habilitar a Azure Monitor para contenedores  

Este artículo proporciona información general de las opciones disponibles para configurar Azure Monitor para los contenedores para supervisar el rendimiento de las cargas de trabajo que se implementan en entornos de Kubernetes y hospedadas en [Azure Kubernetes Service](https://docs.microsoft.com/azure/aks/).

Azure Monitor para contenedores puede habilitarse para implementaciones de AKS nuevas, o en una o más existentes, mediante los siguientes métodos compatibles:

* Desde el portal de Azure, Azure PowerShell, o con la CLI de Azure
* Usando [Terraform y AKS](../../terraform/terraform-create-k8s-cluster-with-tf-and-aks.md)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos 
Antes de empezar, asegúrese de que dispone de lo siguiente:

- **Un área de trabajo de Log Analytics.** Puede crearla al habilitar la supervisión de su nuevo clúster de AKS o dejar que la experiencia de incorporación cree un área de trabajo predeterminada en el grupo de recursos predeterminado de la suscripción del clúster de AKS. Si opta por crear el área de trabajo usted mismo, puede usar [Azure Resource Manager](../platform/template-workspace-configuration.md), mediante [PowerShell](../scripts/powershell-sample-create-workspace.md?toc=%2fpowershell%2fmodule%2ftoc.json) o [Azure Portal](../learn/quick-create-workspace.md).
- Es un miembro de la **rol de colaborador de Log Analytics** para habilitar la supervisión del contenedor. Para más información acerca de cómo controlar el acceso a un área de trabajo de Log Analytics, consulte [Administración de áreas de trabajo](../platform/manage-access.md).
- Es un miembro de la **[propietario](../../role-based-access-control/built-in-roles.md#owner)** rol en el recurso de clúster AKS. 

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)]

## <a name="components"></a>Componentes 

La capacidad de supervisar el rendimiento se basa en un agente de Log Analytics en contenedores para Linux, desarrollado específicamente para Azure Monitor para contenedores. Este agente especializado recopila datos de rendimiento y de eventos de todos los nodos del clúster, y el agente se implementa y registra automáticamente en el área de trabajo de Log Analytics especificada durante la implementación. La versión del agente es microsoft/oms:ciprod04202018 o posterior y se representa mediante una fecha en el siguiente formato: *mmddaaaa*. 

>[!NOTE]
>Con la versión preliminar de soporte técnico de Windows Server para AKS, un clúster de AKS con nodos de Windows Server no tiene un agente instalado para recopilar datos y reenviarlos a Azure Monitor. En su lugar, se implementan automáticamente en el clúster como parte de la implementación estándar de un nodo de Linux recopila y reenvía los datos a Azure Monitor en nombre de todos los nodos de Windows en el clúster.  
>

Cuando se publica una nueva versión del agente, se actualiza automáticamente en los clústeres de Kubernetes administrado que se hospedan en Azure Kubernetes Service (AKS). Para seguir las versiones publicadas, consulte los [anuncios de versiones del agente](https://github.com/microsoft/docker-provider/tree/ci_feature_prod). 

>[!NOTE] 
>Si ya ha implementado un clúster de AKS, puede habilitar la supervisión mediante la CLI de Azure o una plantilla de Azure Resource Manager que se proporciona, como se muestra más adelante en este artículo. No puede usar `kubectl` para actualizar, eliminar, volver a implementar o implementar el agente. La plantilla debe implementarse en el mismo grupo de recursos que el clúster.

Habilitar a Monitor de Azure para los contenedores mediante uno de los métodos siguientes que se describe en la tabla siguiente.

| Estado de implementación | Método | DESCRIPCIÓN | 
|------------------|--------|-------------| 
| Nuevo clúster AKS | [Crear el clúster mediante la CLI de Azure](../../aks/kubernetes-walkthrough.md#create-aks-cluster)| Puede habilitar la supervisión de un nuevo clúster AKS que se crea con la CLI de Azure. | 
| | [Creación de clúster mediante Terraform](container-insights-enable-new-cluster.md#enable-using-terraform)| Puede habilitar la supervisión de un nuevo clúster AKS que se crean con la herramienta de código abierto Terraform. | 
| Clúster de AKS existente | [Habilitar el uso de la CLI de Azure](container-insights-enable-existing-clusters.md#enable-using-azure-cli) | Puede habilitar la supervisión de un clúster de AKS ya implementado mediante la CLI de Azure. | 
| |[Habilitar el uso de Terraform](container-insights-enable-existing-clusters.md#enable-using-terraform) | Puede habilitar la supervisión de un clúster de AKS ya implementado mediante la herramienta de código abierto Terraform. | 
| | [Habilitar desde Azure Monitor](container-insights-enable-existing-clusters.md#enable-from-azure-monitor-in-the-portal)| Puede habilitar la supervisión de uno o varios clústeres AKS ya implementados desde la página de varios clúster AKS en Azure Monitor. | 
| | [Habilitar el clúster de AKS](container-insights-enable-existing-clusters.md#enable-directly-from-aks-cluster-in-the-portal)| Puede habilitar la supervisión directamente desde un clúster de AKS en Azure portal. | 
| | [Habilitar el uso de una plantilla de Azure Resource Manager](container-insights-enable-existing-clusters.md#enable-using-an-azure-resource-manager-template)| Puede habilitar la supervisión de un clúster de AKS con una plantilla preconfigurada de Azure Resource Manager. | 

## <a name="next-steps"></a>Pasos siguientes

* Con la supervisión habilitada para capturar métricas de estado para los pods y nodos del clúster de AKS, estas métricas de estado están disponibles en Azure Portal. Para obtener información sobre cómo usar Azure Monitor para contenedores, consulte el artículo sobre cómo [ver el estado de Azure Kubernetes Service](container-insights-analyze.md).
