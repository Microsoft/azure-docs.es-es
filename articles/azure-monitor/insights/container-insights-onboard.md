---
title: Habilitar Azure Monitor para contenedores | Microsoft Docs
description: En este artículo se describe cómo habilitar y configurar Azure Monitor para contenedores, de forma que pueda conocer el rendimiento de su contenedor y qué problemas relacionados con su rendimiento se han identificado.
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: 1cb393faded714593ce635669f585d5979ee69b8
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87320313"
---
# <a name="enable-azure-monitor-for-containers"></a>Habilita Azure Monitor para contenedores.

En este artículo se proporciona información general sobre las opciones disponibles para configurar Azure Monitor para contenedores para supervisar el rendimiento de las cargas de trabajo que se implementan en entornos de Kubernetes y se hospedan en:

- [Azure Kubernetes Service (AKS)](../../aks/index.yml)  
- [Red Hat OpenShift en Azure](../../openshift/intro-openshift.md), versiones 3.x y 4.x  
- [Red Hat OpenShift](https://docs.openshift.com/container-platform/4.3/welcome/index.html) versión 4.x  
- Un [clúster de Kubernetes habilitado para Arc](../../azure-arc/kubernetes/overview.md)

También puede supervisar el rendimiento de las cargas de trabajo que se implementan en clústeres de Kubernetes autoadministrados hospedados en:
- Azure, mediante el [motor de AKS](https://github.com/Azure/aks-engine)
- [Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview?view=azs-1910) o local, mediante el motor de AKS

Puede habilitar Azure Monitor para contenedores para una implementación nueva o para una o varias implementaciones existentes de Kubernetes mediante los siguientes métodos compatibles:

- El Portal de Azure
- Azure PowerShell
- La CLI de Azure
- [Terraform y AKS](/azure/developer/terraform/create-k8s-cluster-with-tf-and-aks)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, asegúrese de que cumple los requisitos siguientes:

- Tiene un área de trabajo de Log Analytics.

   Azure Monitor para contenedores admite un área de trabajo de Log Analytics en las regiones enumeradas en [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?regions=all&products=monitor).

   Puede crear un área de trabajo al habilitar la supervisión de su nuevo clúster de AKS o dejar que la experiencia de incorporación cree un área de trabajo predeterminada en el grupo de recursos predeterminado de la suscripción del clúster de AKS. 
   
   Si decide crear el área de trabajo, puede hacerlo mediante: 
   - [Azure Resource Manager](../platform/template-workspace-configuration.md)
   - [PowerShell](../scripts/powershell-sample-create-workspace.md?toc=%2fpowershell%2fmodule%2ftoc.json)
   - [Portal de Azure](../learn/quick-create-workspace.md) 
   
   Para obtener una lista de los pares de asignación compatibles utilizados para el área de trabajo predeterminada, vea [Asignaciones de región compatibles con Azure Monitor para contenedores](container-insights-region-mapping.md).

- Es miembro del grupo *Colaborador de Log Analytics* para habilitar la supervisión de contenedores. Para más información acerca de cómo controlar el acceso a un área de trabajo de Log Analytics, consulte [Administración de áreas de trabajo](../platform/manage-access.md).

- Es miembro del [grupo *Propietario*](../../role-based-access-control/built-in-roles.md#owner) en el recurso de clúster de AKS.

   [!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)]

- Para ver los datos de supervisión, debe tener el rol [*Lector de Log Analytics*](../platform/manage-access.md#manage-access-using-azure-permissions) en el área de trabajo de Log Analytics configurada con Azure Monitor para contenedores.

- Las métricas de Prometheus no se recopilan de forma predeterminada. Antes de [configurar el agente](container-insights-prometheus-integration.md) para recopilar las métricas, es importante revisar la [documentación de Prometheus](https://prometheus.io/) para saber qué datos se pueden recopilar y qué métodos se admiten.

## <a name="supported-configurations"></a>Configuraciones admitidas

Azure Monitor para contenedores admite oficialmente las siguientes configuraciones:

- Entornos: Red Hat OpenShift en Azure, Kubernetes local y el motor de AKS en Azure y Azure Stack. Para más información, vea el [motor de AKS en Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview?view=azs-1908).
- Las versiones de Kubernetes y de la directiva de compatibilidad son las mismas que las [compatibles con Azure Kubernetes Service (AKS)](../../aks/supported-kubernetes-versions.md). 

## <a name="network-firewall-requirements"></a>Requisitos de firewall de red

En la tabla siguiente se muestra la información sobre la configuración de proxy y firewall requerida para que el agente en contenedor se comunique con Azure Monitor para contenedores. Todo el tráfico de red del agente se enlaza a Azure Monitor.

|Recurso del agente|Port |
|--------------|------|
| `*.ods.opinsights.azure.com` | 443 |
| `*.oms.opinsights.azure.com` | 443 |
| `dc.services.visualstudio.com` | 443 |
| `*.monitoring.azure.com` | 443 |
| `login.microsoftonline.com` | 443 |

En la tabla siguiente se muestra la información sobre la configuración de proxy y firewall para Azure China 21Vianet:

|Recurso del agente|Port |Descripción | 
|--------------|------|-------------|
| `*.ods.opinsights.azure.cn` | 443 | Ingesta de datos |
| `*.oms.opinsights.azure.cn` | 443 | Incorporación de OMS |
| `dc.services.visualstudio.com` | 443 | Para la telemetría del agente que usa Application Insights en la nube pública de Azure. |

En la tabla siguiente se muestra la información sobre la configuración de proxy y firewall para Azure US Government:

|Recurso del agente|Port |Descripción | 
|--------------|------|-------------|
| `*.ods.opinsights.azure.us` | 443 | Ingesta de datos |
| `*.oms.opinsights.azure.us` | 443 | Incorporación de OMS |
| `dc.services.visualstudio.com` | 443 | Para la telemetría del agente que usa Application Insights en la nube pública de Azure. |

## <a name="components"></a>Componentes

La capacidad de supervisar el rendimiento se basa en un agente de Log Analytics en contenedores para Linux, desarrollado específicamente para Azure Monitor para contenedores. Este agente especializado recopila datos de rendimiento y de eventos de todos los nodos del clúster, y el agente se implementa y registra automáticamente en el área de trabajo de Log Analytics especificada durante la implementación. 

La versión del agente es microsoft/oms:ciprod04202018 o posterior y se representa mediante una fecha en el siguiente formato: *mmddaaaa*.

>[!NOTE]
>Con la disponibilidad general de la compatibilidad con Windows Server para AKS, un clúster de AKS con nodos de Windows Server tiene un agente de versión preliminar instalado como pod de daemonset en cada nodo individual de Windows Server para recopilar registros y reenviarlos a Log Analytics. Para las métricas de rendimiento, un nodo de Linux implementado automáticamente en el clúster como parte de la implementación estándar recopila y reenvía los datos a Azure Monitor en nombre de todos los nodos de Windows del clúster.

Cuando se publica una nueva versión del agente, se actualiza automáticamente en los clústeres de Kubernetes administrados que se hospedan en Azure Kubernetes Service (AKS). Para realizar un seguimiento de las versiones publicadas, vea los [anuncios de versiones del agente](https://github.com/microsoft/docker-provider/tree/ci_feature_prod).

> [!NOTE]
> Si ya ha implementado un clúster de AKS, puede habilitar la supervisión mediante la CLI de Azure o una plantilla de Azure Resource Manager proporcionada, como se muestra más adelante en este artículo. No puede usar `kubectl` para actualizar, eliminar, reimplementar o implementar el agente.
>
> La plantilla debe implementarse en el mismo grupo de recursos que el clúster.

Para habilitar Azure Monitor para contenedores, puede usar uno de los métodos descritos en la tabla siguiente:

| Estado de la implementación | Método | Descripción |
|------------------|--------|-------------|
| Nuevo clúster de Kubernetes | [Crear un clúster de AKS mediante la CLI de Azure](../../aks/kubernetes-walkthrough.md#create-aks-cluster)| Puede habilitar la supervisión de un nuevo clúster de AKS que cree con la CLI de Azure. |
| | [Crear de un clúster de AKS con Terraform](container-insights-enable-new-cluster.md#enable-using-terraform)| Puede habilitar la supervisión de un nuevo clúster de AKS que cree mediante la herramienta de código abierto Terraform. |
| | [Crear un clúster de OpenShift mediante una plantilla de Azure Resource Manager](container-insights-azure-redhat-setup.md#enable-for-a-new-cluster-using-an-azure-resource-manager-template) | Puede habilitar la supervisión de un nuevo clúster de OpenShift que cree con una plantilla preconfigurada de Azure Resource Manager. |
| | [Crear un clúster de OpenShift mediante la CLI de Azure](/cli/azure/openshift?view=azure-cli-latest#az-openshift-create) | Puede habilitar la supervisión durante la implementación de un nuevo clúster de OpenShift con la CLI de Azure. |
| Clúster de Kubernetes existente | [Habilitar la supervisión de un clúster de AKS mediante la CLI de Azure](container-insights-enable-existing-clusters.md#enable-using-azure-cli) | Puede habilitar la supervisión de un clúster de AKS ya implementado mediante la CLI de Azure. |
| |[Habilitar un clúster de AKS con Terraform](container-insights-enable-existing-clusters.md#enable-using-terraform) | Puede habilitar la supervisión de un clúster de AKS ya implementado mediante la herramienta de código abierto Terraform. |
| | [Habilitar un clúster de AKS desde Azure Monitor](container-insights-enable-existing-clusters.md#enable-from-azure-monitor-in-the-portal)| Puede habilitar la supervisión de uno o varios clústeres de AKS ya implementados desde la página de varios clústeres en Azure Monitor. |
| | [Habilitación desde el clúster de AKS](container-insights-enable-existing-clusters.md#enable-directly-from-aks-cluster-in-the-portal)| Puede habilitar la supervisión directamente desde un clúster de AKS en Azure Portal. |
| | [Habilitar para un clúster de AKS mediante una plantilla de Azure Resource Manager](container-insights-enable-existing-clusters.md#enable-using-an-azure-resource-manager-template)| Puede habilitar la supervisión de un clúster de AKS con una plantilla preconfigurada de Azure Resource Manager. |
| | [Habilitación para un clúster de Kubernetes híbrido](container-insights-hybrid-setup.md) | Puede habilitar la supervisión del motor de AKS que se hospeda en Azure Stack o para un clúster de Kubernetes hospedado de forma local. |
| | [Habilitar un clúster de Kubernetes habilitado para Arc](container-insights-enable-arc-enabled-clusters.md) | Puede habilitar la supervisión de los clústeres de Kubernetes que se hospedan fuera de Azure y que están habilitados para Azure Arc. |
| | [Habilitar para un clúster de OpenShift mediante una plantilla de Azure Resource Manager](container-insights-azure-redhat-setup.md#enable-using-an-azure-resource-manager-template) | Puede habilitar la supervisión de un clúster de OpenShift existente con una plantilla preconfigurada de Azure Resource Manager. |
| | [Habilitar para un clúster de OpenShift desde Azure Monitor](container-insights-azure-redhat-setup.md#from-the-azure-portal) | Puede habilitar la supervisión de uno o varios clústeres de OpenShift ya implementados desde la página de varios clústeres en Azure Monitor. |

## <a name="next-steps"></a>Pasos siguientes

Ahora que ya ha habilitado la supervisión, puede empezar a analizar el rendimiento de los clústeres de Kubernetes hospedados en Azure Kubernetes Service (AKS), Azure Stack u otro entorno. Para aprender a usar Azure Monitor para contenedores, consulte [Visualización del rendimiento del clúster de Kubernetes](container-insights-analyze.md).

