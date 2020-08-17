---
title: Cómo solucionar problemas de Azure Monitor para contenedores | Microsoft Docs
description: En este artículo se describe cómo se pueden detectar y solucionar problemas con Azure Monitor para contenedores.
ms.topic: conceptual
ms.date: 07/21/2020
ms.openlocfilehash: fcd799c63e4afb68d96f67d1c03016a4d3b10f34
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87092837"
---
# <a name="troubleshooting-azure-monitor-for-containers"></a>Solución de problemas de Azure Monitor para contenedores

Al configurar la supervisión del clúster de Azure Kubernetes Service (AKS) con Azure Monitor para contenedores, puede encontrar un problema que impida la recopilación de datos o el informe del estado. En este artículo se detallan algunos problemas comunes y los pasos para solucionarlos.

## <a name="authorization-error-during-onboarding-or-update-operation"></a>Error de autorización durante la operación de incorporación o actualización

Durante la habilitación de Azure Monitor para los contenedores o la actualización de un clúster para que admita la recopilación de métricas, puede recibir un error similar al siguiente: *El cliente <identidad del usuario >" con Id. de objeto "< objectId del usuario >"no tiene autorización para realizar la acción "Microsoft.Authorization/roleAssignments/write" en el ámbito*

Durante el proceso de incorporación o de actualización, se intenta la asignación del rol **Publicador de métricas de supervisión** en el recurso de clúster. El usuario que inicia el proceso para habilitar Azure Monitor para contenedores o la actualización para admitir la recopilación de métricas debe tener acceso al permiso **Microsoft.Authorization/roleAssignments/write** en el ámbito de recurso de clúster de AKS. Solo a los miembros de los roles integrados **Propietario** y **Administrador de acceso de usuario** se les ha concedido acceso a este permiso. Si las directivas de seguridad requieren la asignación de permisos a un nivel específico, se recomienda consultar los [roles personalizados](../../role-based-access-control/custom-roles.md) y asignarlos a los usuarios que lo necesiten.

También puede conceder este rol de forma manual desde Azure Portal siguiendo estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En Azure Portal, haga clic en **Todos los servicios**, en la esquina superior izquierda. En la lista de recursos, escriba **Kubernetes**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Seleccione **Azure Kubernetes**.
3. En la lista de clústeres de Kubernetes, seleccione uno.
2. En el menú de la izquierda, haga clic en **Control de acceso (IAM)** .
3. Seleccione **+ Agregar** para agregar una asignación de roles, seleccione el cuadro **Publicador de métricas de supervisión** y, en el cuadro **Seleccionar**, escriba **AKS** para filtrar los resultados en las entidades de servicio de clústeres definidas en la suscripción. Seleccione aquella en la lista que sea específica de ese clúster.
4. Haga clic en **Guardar** para finalizar la asignación del rol.

## <a name="azure-monitor-for-containers-is-enabled-but-not-reporting-any-information"></a>Azure Monitor para contenedores está habilitado, pero no proporciona ninguna información

Si Azure Monitor para contenedores está habilitado y configurado correctamente, pero no puede ver la información de estado o no se devuelve ningún resultado de una consulta de registros, siga estos pasos para diagnosticar el problema:

1. Ejecute este comando para comprobar el estado del agente:

    `kubectl get ds omsagent --namespace=kube-system`

    Si la salida se parece al ejemplo siguiente, la implementación se ha realizado correctamente:

    ```
    User@aksuser:~$ kubectl get ds omsagent --namespace=kube-system
    NAME       DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
    omsagent   2         2         2         2            2           beta.kubernetes.io/os=linux   1d
    ```
2. Si tiene nodos de Windows Server, compruebe el estado del agente mediante la ejecución del comando:

    `kubectl get ds omsagent-win --namespace=kube-system`

    Si la salida se parece al ejemplo siguiente, la implementación se ha realizado correctamente:

    ```
    User@aksuser:~$ kubectl get ds omsagent-win --namespace=kube-system
    NAME                   DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                   AGE
    omsagent-win           2         2         2         2            2           beta.kubernetes.io/os=windows   1d
    ```
3. Compruebe el estado de implementación con la versión del agente *06072018* o posterior con el comando:

    `kubectl get deployment omsagent-rs -n=kube-system`

    Si la salida se parece al ejemplo siguiente, la implementación se ha realizado correctamente:

    ```
    User@aksuser:~$ kubectl get deployment omsagent-rs -n=kube-system
    NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE    AGE
    omsagent   1         1         1            1            3h
    ```

4. Con el comando siguiente, consulte el estado del pod para comprobar que se está ejecutando: `kubectl get pods --namespace=kube-system`

    La salida debe parecerse al ejemplo siguiente con un estado de *Running* (En ejecución) en omsagent:

    ```
    User@aksuser:~$ kubectl get pods --namespace=kube-system
    NAME                                READY     STATUS    RESTARTS   AGE
    aks-ssh-139866255-5n7k5             1/1       Running   0          8d
    azure-vote-back-4149398501-7skz0    1/1       Running   0          22d
    azure-vote-front-3826909965-30n62   1/1       Running   0          22d
    omsagent-484hw                      1/1       Running   0          1d
    omsagent-fkq7g                      1/1       Running   0          1d
    omsagent-win-6drwq                  1/1       Running   0          1d
    ```

## <a name="error-messages"></a>Mensajes de error

En la tabla siguiente se resumen los errores conocidos que pueden producirse al usar Azure Monitor para contenedores.

| Mensajes de error  | Acción |
| ---- | --- |
| Mensaje de error `No data for selected filters`  | Puede tardar algún tiempo en establecer el flujo de datos de supervisión de clústeres recién creados. Espere de 10 a 15 minutos como mínimo para que aparezcan los datos del clúster. |
| Mensaje de error `Error retrieving data` | Mientras el clúster de Azure Kubernetes Service se configura para la supervisión de rendimiento y de mantenimiento, se establece una conexión entre el clúster y el área de trabajo de Azure Log Analytics. Se utiliza un área de trabajo de Log Analytics para almacenar todos los datos de supervisión del clúster. Este error puede producirse cuando el área de trabajo de Log Analytics se ha eliminado. Compruebe si se eliminó el área de trabajo y si fue así, deberá volver a habilitar la supervisión del clúster con Azure Monitor para los contenedores y especificar un área de trabajo existente o crear una nueva. Para volver a habilitarlo, tiene que [deshabilitar](container-insights-optout.md) la supervisión del clúster y [habilitar](container-insights-enable-new-cluster.md) de nuevo Azure Monitor para contenedores. |
| `Error retrieving data` después de agregar Azure Monitor para contenedores mediante az aks cli | Al habilitar la supervisión mediante `az aks cli`, Azure Monitor para contenedores puede que no se implemente correctamente. Compruebe que la solución se ha implementado. Para ello, vaya al área de trabajo de Log Analytics y vea si la solución está disponible seleccionado **Soluciones** en el panel de la izquierda. Para resolver este problema, tendrá que volver a implementar la solución siguiendo las instrucciones de [Cómo incorporar Azure Monitor para contenedores](container-insights-onboard.md). |

Para ayudar a diagnosticar el problema, hemos incluido un script de solución de problemas que está disponible [aquí](https://raw.githubusercontent.com/microsoft/Docker-Provider/ci_dev/scripts/troubleshoot/TroubleshootError_nonAzureK8s.ps1).

## <a name="azure-monitor-for-containers-agent-replicaset-pods-are-not-scheduled-on-non-azure-kubernetes-cluster"></a>Los pods de ReplicaSet del agente de Azure Monitor para contenedores no están programados en un clúster de Kubernetes que no es de Azure.

Los pods de ReplicaSet del agente de Azure Monitor para contenedores tienen una dependencia de los siguientes selectores de nodo en los nodos de trabajo (o agente) para la programación:

```
nodeSelector:
  beta.kubernetes.io/os: Linux
  kubernetes.io/role: agent
```

Si los nodos de trabajo no tienen etiquetas de nodo adjuntas, los pods de ReplicaSet del agente no se programarán. Consulte [Selectores de etiqueta de asignación de Kubernetes](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/) para obtener instrucciones sobre cómo adjuntar la etiqueta.

## <a name="performance-charts-dont-show-cpu-or-memory-of-nodes-and-containers-on-a-non-azure-cluster"></a>Los gráficos de rendimiento no muestran la CPU o la memoria de los nodos y contenedores de un clúster que no es de Azure.

Los pods del agente de Azure Monitor para contenedores usan el punto de conexión cAdvisor en el agente de nodo para recopilar las métricas de rendimiento. Compruebe que el agente en contenedor del nodo está configurado para permitir que se abra `cAdvisor port: 10255` en todos los nodos del clúster para recopilar las métricas de rendimiento.

## <a name="non-azure-kubernetes-cluster-are-not-showing-in-azure-monitor-for-containers"></a>No se muestran los clústeres de Kubernetes que no son de Azure en Azure Monitor para contenedores.

Para ver el clúster de Kubernetes que no es de Azure en Azure Monitor para contenedores, se necesita acceso de lectura en el área de trabajo de Log Analytics que admite esta información y en el recurso de la solución de Container Insights **ContainerInsights (*área de trabajo*)** .

## <a name="next-steps"></a>Pasos siguientes

Con la supervisión habilitada para capturar métricas de estado para los pods y nodos del clúster de AKS, estas métricas de estado están disponibles en Azure Portal. Para obtener información sobre cómo usar Azure Monitor para contenedores, consulte el artículo sobre cómo [ver el estado de Azure Kubernetes Service](container-insights-analyze.md).
