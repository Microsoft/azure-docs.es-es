---
title: Acceso a recursos de Kubernetes en Azure Portal (versión preliminar)
description: Descubra cómo interactuar con recursos de Kubernetes para administrar un clúster de Azure Kubernetes Service (AKS) en Azure Portal.
services: container-service
author: laurenhughes
ms.topic: article
ms.date: 08/11/2020
ms.author: lahugh
ms.openlocfilehash: 4a0acf284475f3c9119f3b9d012debad656b1faa
ms.sourcegitcommit: d18a59b2efff67934650f6ad3a2e1fe9f8269f21
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88661357"
---
# <a name="access-kubernetes-resources-from-the-azure-portal-preview"></a>Acceso a recursos de Kubernetes en Azure Portal (versión preliminar)

Azure Portal incluye un visor de recursos de Kubernetes (versión preliminar) para facilitar el acceso a los recursos de Kubernetes en el clúster de Azure Kubernetes Service (AKS). Ver los recursos de Kubernetes en Azure Portal reduce la necesidad de cambiar de contexto entre el Azure Portal y la herramienta de línea de comandos `kubectl`. De este modo, se simplifican la visualización y edición de recursos de Kubernetes. El visor de recursos actualmente incluye varios tipos de recursos, como implementaciones, pods y conjuntos de réplicas.

La vista de recursos de Kubernetes en Azure Portal reemplaza al [complemento del panel de AKS][kubernetes-dashboard], que ha quedado obsoleto.

>[!NOTE]
>Esta funcionalidad no es compatible actualmente en [los clústeres privados de Azure Kubernetes Service](https://docs.microsoft.com/azure/aks/private-clusters).

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## <a name="prerequisites"></a>Prerrequisitos

Para visualizar los recursos de Kubernetes en Azure Portal, necesita un clúster de AKS. Cualquier clúster puede valer, pero si se utiliza la integración de Azure Active Directory (Azure AD), el clúster tiene que utilizar la [integración de Azure AD administrado por AKS][aks-managed-aad]. Si su clúster utiliza la integración de Azure AD heredada, puede actualizarlo en el portal o mediante la [CLI de Azure][cli-aad-upgrade].

## <a name="view-kubernetes-resources"></a>Visualización de recursos de Kubernetes

Para ver los recursos de Kubernetes, acceda al clúster de AKS en Azure Portal. El panel de navegación de la izquierda se usa para acceder a los recursos. Los recursos incluyen:

- **Espacios de nombres** muestra los espacios de nombres del clúster. El filtro situado en la parte superior de la lista de espacios de nombres permite filtrar y visualizar de forma rápida los recursos del espacio de nombres.
- **Cargas de trabajo** muestra información sobre implementaciones, pods, conjuntos de réplicas y conjuntos de demonios que se han implementado en el clúster. La siguiente captura de pantalla muestra los pods del sistema predeterminados en un clúster de AKS de ejemplo.
- **Servicios y entradas** muestra todos los recursos de entrada y servicio del clúster.

:::image type="content" source="media/kubernetes-portal/workloads.png" alt-text="Información del pod de Kubernetes en Azure Portal." lightbox="media/kubernetes-portal/workloads.png":::

### <a name="deploy-an-application"></a>Implementar una aplicación

En este ejemplo, usaremos el clúster de AKS de ejemplo para implementar la aplicación Azure Vote desde el [inicio rápido de AKS][portal-quickstart].

1. Seleccione **Agregar** desde cualquiera de las vistas de recursos (espacio de nombres, cargas de trabajo o servicios y entradas).
1. Pegue el archivo YAML de la aplicación Azure Vote desde el [inicio rápido de AKS][portal-quickstart].
1. Seleccione **Agregar** en la parte inferior del editor YAML para implementar la aplicación. 

Una vez que se añade el archivo YAML, el visor de recursos muestra los dos servicios de Kubernetes creados: el servicio interno (azure-vote-back) y el servicio externo (azure-vote-front) para acceder a la aplicación Azure Vote. El servicio externo incluye una dirección IP externa vinculada para que pueda ver fácilmente la aplicación en el explorador.

:::image type="content" source="media/kubernetes-portal/portal-services.png" alt-text="Información de la aplicación Azure Vote mostrada en Azure Portal." lightbox="media/kubernetes-portal/portal-services.png":::

### <a name="monitor-deployment-insights"></a>Supervisión de la información de implementación

Los clústeres de AKS que tengan activado [Azure Monitor para contenedores][enable-monitor] pueden ver rápidamente la información de implementación. En la vista de recursos de Kubernetes, los usuarios pueden ver el estado activo de implementaciones individuales, incluido el uso de la CPU y de la memoria, además de cambiar a Azure Monitor para ver información más detallada. A continuación, se incluye un ejemplo de información de implementación de un clúster de AKS de ejemplo:

:::image type="content" source="media/kubernetes-portal/deployment-insights.png" alt-text="Información de implementación en Azure Portal." lightbox="media/kubernetes-portal/deployment-insights.png":::

## <a name="edit-yaml"></a>Edición de YAML

La vista de recursos de Kubernetes también incluye un editor YAML. Tener un editor YAML integrado significa que puede actualizar o crear servicios e implementaciones desde el portal y aplicar los cambios inmediatamente.

:::image type="content" source="media/kubernetes-portal/service-editor.png" alt-text="Editor YAML para un servicio de Kubernetes en Azure Portal.":::

Después de editar el archivo YAML, los cambios se aplicarán al seleccionar **revisar + guardar**, confirmarlos y, a continuación, volver a guardar.

>[!WARNING]
> No se recomienda hacer cambios de producción directamente a través de la interfaz de usuario o la CLI. Siga los [procedimientos recomendados de integración continua (CI) y de implementación continua (CD)](kubernetes-action.md). La funcionalidad de administración de Kubernetes en Azure Portal y el editor YAML se han creado para aprender sobre nuevas implementaciones y lanzarlas en una configuración de desarrollo y pruebas.

## <a name="troubleshooting"></a>Solución de problemas

En esta sección se abordan problemas comunes y los pasos para solucionarlos.

### <a name="unauthorized-access"></a>Acceso no autorizado

Para acceder a recursos de Kubernetes, debe tener acceso al clúster, la API y los objetos de Kubernetes. El usuario debe ser administrador de clústeres o tener los permisos adecuados para acceder al clúster de AKS. Para obtener más información sobre la seguridad de clúster, consulte [Opciones de acceso e identidad en Azure Kubernetes Service (AKS)][concepts-identity].

### <a name="enable-resource-view"></a>Activación de la vista de recursos

En el caso de clústeres existentes, es posible que tenga que activar la vista de recursos de Kubernetes. Para activar la vista de recursos, siga las indicaciones del portal para el clúster.

:::image type="content" source="media/kubernetes-portal/enable-resource-view.png" alt-text="Mensaje de Azure Portal para activar la vista de recursos Kubernetes." lightbox="media/kubernetes-portal/enable-resource-view.png":::

## <a name="next-steps"></a>Pasos siguientes

En este artículo se ha mostrado cómo acceder a recursos de Kubernetes para el clúster de AKS. Consulte [Implementaciones y manifiestos YAML][deployments] para familiarizarse con los recursos de clúster y los archivos YAML a los que accede mediante el visor de recursos de Kubernetes.

<!-- LINKS - internal -->
[kubernetes-dashboard]: kubernetes-dashboard.md
[concepts-identity]: concepts-identity.md
[portal-quickstart]: kubernetes-walkthrough-portal.md#run-the-application
[deployments]: concepts-clusters-workloads.md#deployments-and-yaml-manifests
[aks-managed-aad]: managed-aad.md
[cli-aad-upgrade]: managed-aad.md#upgrading-to-aks-managed-azure-ad-integration
[enable-monitor]: ../azure-monitor/insights/container-insights-enable-existing-clusters.md
