---
title: Cómo detener la supervisión del clúster híbrido de Kubernetes | Microsoft Docs
description: En este artículo se describe cómo puede detener la supervisión del clúster híbrido de Kubernetes con Azure Monitor para contenedores.
ms.topic: conceptual
ms.date: 06/16/2020
ms.openlocfilehash: 8369c82b83cfbaa7128383c6203aaf584916cae9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87091205"
---
# <a name="how-to-stop-monitoring-your-hybrid-cluster"></a>Cómo detener la supervisión del clúster híbrido

Después de habilitar la supervisión del clúster de Kubernetes, puede detener la supervisión del clúster con Azure Monitor para contenedores si decide que ya no desea continuar haciéndolo. En este artículo se muestra cómo hacerlo para los siguientes entornos:

- Motor de AKS en Azure y Azure Stack
- OpenShift versión 4 y versiones posteriores
- Kubernetes habilitado para Azure Arc (versión preliminar)

## <a name="how-to-stop-monitoring-using-helm"></a>Cómo detener la supervisión con Helm

Los pasos siguientes se aplican a los siguientes entornos:

- Motor de AKS en Azure y Azure Stack
- OpenShift versión 4 y versiones posteriores

1. Para identificar primero la versión del gráfico de Helm de Azure Monitor para contenedores instalada en el clúster, ejecute el siguiente comando de Helm.

    ```
    helm list
    ```

    La salida será similar a la siguiente:

    ```
    NAME                            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
    azmon-containers-release-1      default         3               2020-04-21 15:27:24.1201959 -0700 PDT   deployed        azuremonitor-containers-2.7.0   7.0.0-1
    ```

    *azmon-containers-release-1* representa la versión del gráfico de Helm para Azure Monitor para contenedores.

2. Para eliminar la versión del gráfico, ejecute el siguiente comando de Helm.

    `helm delete <releaseName>`

    Ejemplo:

    `helm delete azmon-containers-release-1`

    Esto eliminará la versión del clúster. Puede comprobarlo mediante la ejecución del comando `helm list`:

    ```
    NAME                            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
    ```

El cambio de configuración puede tardar unos minutos en completarse. Dado que Helm realiza un seguimiento de las versiones incluso después de eliminarlas, es posible auditar el historial de un clúster e incluso recuperar una versión con `helm rollback`.

## <a name="how-to-stop-monitoring-on-arc-enabled-kubernetes"></a>Cómo detener la supervisión en Kubernetes habilitado para Arc

### <a name="using-powershell"></a>Usar PowerShell

1. Descargue y guarde el script que configura el clúster en una carpeta local con el complemento de supervisión mediante el uso de los siguientes comandos:

    ```powershell
    wget https://aka.ms/disable-monitoring-powershell-script -OutFile disable-monitoring.ps1
    ```

2. Configure la variable `$azureArcClusterResourceId`; para ello, establezca los valores correspondientes para `subscriptionId`, `resourceGroupName` y `clusterName`, que representan el identificador de recurso del recurso de clúster de Kubernetes habilitado para Azure Arc.

    ```powershell
    $azureArcClusterResourceId = "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Kubernetes/connectedClusters/<clusterName>"
    ```

3. Configure la variable `$kubeContext` con el valor de **kube-context** del clúster mediante la ejecución del comando `kubectl config get-contexts`. Si desea utilizar el contexto actual, establezca el valor en `""`.

    ```powershell
    $kubeContext = "<kubeContext name of your k8s cluster>"
    ```

4. Ejecute el siguiente comando para detener la supervisión del clúster.

    ```powershell
    .\disable-monitoring.ps1 -clusterResourceId $azureArcClusterResourceId -kubeContext $kubeContext
    ```

### <a name="using-bash"></a>Uso de Bash

1. Descargue y guarde el script que configura el clúster en una carpeta local con el complemento de supervisión mediante el uso de los siguientes comandos:

    ```bash
    curl -o disable-monitoring.sh -L https://aka.ms/disable-monitoring-bash-script
    ```

2. Configure la variable `azureArcClusterResourceId`; para ello, establezca los valores correspondientes para `subscriptionId`, `resourceGroupName` y `clusterName`, que representan el identificador de recurso del recurso de clúster de Kubernetes habilitado para Azure Arc.

    ```bash
    export azureArcClusterResourceId="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Kubernetes/connectedClusters/<clusterName>"
    ```

3. Configure la variable `kubeContext` con el valor de **kube-context** del clúster mediante la ejecución del comando `kubectl config get-contexts`.

    ```bash
    export kubeContext="<kubeContext name of your k8s cluster>"
    ```

4. Para detener la supervisión del clúster, se proporcionan comandos diferentes según el escenario de implementación.

    Ejecute el siguiente comando para detener la supervisión del clúster con el contexto actual.

    ```bash
    bash disable-monitoring.sh --resource-id $azureArcClusterResourceId
    ```

    Ejecute el siguiente comando para detener la supervisión del clúster mediante la especificación de un contexto.

    ```bash
    bash disable-monitoring.sh --resource-id $azureArcClusterResourceId --kube-context $kubeContext
    ```

## <a name="next-steps"></a>Pasos siguientes

Si el área de trabajo de Log Analytics se creó solamente para admitir la supervisión del clúster y ya no es necesaria, tiene que eliminarla manualmente. Si no está familiarizado con la eliminación de un área de trabajo, consulte [Eliminación de un área de trabajo de Azure Log Analytics](../platform/delete-workspace.md).
