---
title: Cómo solucionar problemas de Azure Monitor para contenedores | Microsoft Docs
description: En este artículo se describe cómo se pueden detectar y solucionar problemas con Azure Monitor para contenedores.
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/27/2018
ms.author: magoedte
ms.openlocfilehash: 2e3e39ef24d82393d981c0ce276b3338419e0b2d
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65521769"
---
# <a name="troubleshooting-azure-monitor-for-containers"></a>Solución de problemas de Azure Monitor para contenedores

Al configurar la supervisión del clúster de Azure Kubernetes Service (AKS) con Azure Monitor para contenedores, puede encontrar un problema que impida la recopilación de datos o el informe del estado. En este artículo se detallan algunos problemas comunes y los pasos para solucionarlos.

## <a name="authorization-error-during-onboarding-or-update-operation"></a>Error de autorización durante la operación de incorporación o actualización
Durante la habilitación de Azure Monitor para contenedores o actualizar un clúster para que admiten recopilar métricas, puede recibir un error similar al siguiente: *el cliente < identidad del usuario >' con el objeto no tiene Id. '< objectId del usuario >' autorización para realizar la acción 'Microsoft.Authorization/roleAssignments/write' en el ámbito*

Durante el proceso de incorporación o actualización, conceder la **Publisher de métricas de supervisión** se intenta la asignación de roles en el recurso de clúster. El usuario que inicia el proceso para habilitar Azure Monitor para contenedores o la actualización admitir la colección de métricas debe tener acceso a la **Microsoft.Authorization/roleAssignments/write** permiso en el clúster de AKS ámbito de recurso. Solo los miembros de la **propietario** y **Administrador de acceso de usuario** roles integrados tienen acceso a este permiso. Si las directivas de seguridad requieren asignar permisos a nivel granulares, le recomendamos que visualice [roles personalizados](../../role-based-access-control/custom-roles.md) y asignarla a los usuarios que lo necesiten. 

Puede conceder este rol también manualmente desde el portal de Azure realizando los pasos siguientes:

1. Inicie sesión en el [Azure Portal](https://portal.azure.com). 
2. En Azure Portal, haga clic en **Todos los servicios**, en la esquina superior izquierda. En la lista de recursos, escriba **Kubernetes**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Seleccione **Azure Kubernetes**.
3. En la lista de clústeres de Kubernetes, seleccione uno de la lista.
2. En el menú izquierdo, haga clic en **control de acceso (IAM)**.
3. Seleccione **+ agregar** para agregar una asignación de roles y seleccione el **Publisher de métricas de supervisión** rol y, en el **seleccione** cuadro, escriba **AKS** a filtro de entidades definidas en la suscripción de servicio de los resultados en solo los clústeres. Seleccione aquella en la lista que es específica de ese clúster.
4. Haga clic en **Guardar** para finalizar la asignación del rol. 

## <a name="azure-monitor-for-containers-is-enabled-but-not-reporting-any-information"></a>Azure Monitor para contenedores está habilitado, pero no proporciona ninguna información
Si está habilitado y configurado, correctamente Azure Monitor para los contenedores, pero no se puede ver información de estado o se devuelve ningún resultado de una consulta de registro, diagnosticar el problema siguiendo estos pasos: 

1. Ejecute este comando para comprobar el estado del agente: 

    `kubectl get ds omsagent --namespace=kube-system`

    Si la salida se parece a la siguiente, la implementación se ha realizado correctamente:

    ```
    User@aksuser:~$ kubectl get ds omsagent --namespace=kube-system 
    NAME       DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
    omsagent   2         2         2         2            2           beta.kubernetes.io/os=linux   1d
    ```  
2. Compruebe el estado de implementación con la versión del agente *06072018* o posterior con el comando:

    `kubectl get deployment omsagent-rs -n=kube-system`

    Si la salida se parece al ejemplo siguiente, la implementación se ha realizado correctamente:

    ```
    User@aksuser:~$ kubectl get deployment omsagent-rs -n=kube-system 
    NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE    AGE
    omsagent   1         1         1            1            3h
    ```

3. Con el comando siguiente, consulte el estado del pod para comprobar que se está ejecutando: `kubectl get pods --namespace=kube-system`

    La salida debe parecerse al ejemplo siguiente con un estado de *Running* (En ejecución) en omsagent:

    ```
    User@aksuser:~$ kubectl get pods --namespace=kube-system 
    NAME                                READY     STATUS    RESTARTS   AGE 
    aks-ssh-139866255-5n7k5             1/1       Running   0          8d 
    azure-vote-back-4149398501-7skz0    1/1       Running   0          22d 
    azure-vote-front-3826909965-30n62   1/1       Running   0          22d 
    omsagent-484hw                      1/1       Running   0          1d 
    omsagent-fkq7g                      1/1       Running   0          1d 
    ```

4. Compruebe los registros del agente. Cuando se implementa el agente en contenedor, se ejecuta una comprobación rápida mediante la ejecución de comandos de OMI y se muestra la versión del agente y el proveedor. 

5. Para comprobar que el agente se ha implementado correctamente, ejecute el comando: `kubectl logs omsagent-484hw --namespace=kube-system`

    El estado debe ser similar al ejemplo siguiente:

    ```
    User@aksuser:~$ kubectl logs omsagent-484hw --namespace=kube-system
    :
    :
    instance of Container_HostInventory
    {
        [Key] InstanceID=3a4407a5-d840-4c59-b2f0-8d42e07298c2
        Computer=aks-nodepool1-39773055-0
        DockerVersion=1.13.1
        OperatingSystem=Ubuntu 16.04.3 LTS
        Volume=local
        Network=bridge host macvlan null overlay
        NodeRole=Not Orchestrated
        OrchestratorType=Kubernetes
    }
    Primary Workspace: b438b4f6-912a-46d5-9cb1-b44069212abc    Status: Onboarded(OMSAgent Running)
    omi 1.4.2.2
    omsagent 1.6.0.23
    docker-cimprov 1.0.0.31
    ```

## <a name="error-messages"></a>mensajes de error

En la tabla siguiente se resumen los errores conocidos que pueden producirse al usar Azure Monitor para contenedores.

| mensajes de error  | . |  
| ---- | --- |  
| Mensaje de error `No data for selected filters`  | Puede tardar algún tiempo en establecer el flujo de datos de supervisión de clústeres recién creados. Permitir al menos 10 a 15 minutos para los datos que aparecerá para el clúster. |   
| Mensaje de error `Error retrieving data` | Mientras el clúster de Azure Kubernetes Service se configura para la supervisión de rendimiento y de mantenimiento, se establece una conexión entre el clúster y el área de trabajo de Azure Log Analytics. Se utiliza un área de trabajo de Log Analytics para almacenar todos los datos de supervisión del clúster. Este error puede producirse cuando el área de trabajo de Log Analytics se ha eliminado o perdido. Revise [administrar acceso](../platform/manage-access.md#view-workspace-details) para comprobar si el área de trabajo está disponible. Si falta el área de trabajo, deberá volver a habilitar la supervisión del clúster con Azure Monitor para contenedores. Para volver a habilitar, deberá [deshabilitar](container-insights-optout.md) supervisión para el clúster y [habilitar](container-insights-enable-new-cluster.md) Azure Monitor para los contenedores de nuevo. |  
| `Error retrieving data` después de agregar Azure Monitor para contenedores mediante az aks cli | Al habilitar la supervisión mediante `az aks cli`, Azure Monitor para los contenedores no se pueden implementar correctamente. Compruebe si se ha implementado la solución. Para ello, vaya al área de trabajo de Log Analytics y vea si la solución está disponible seleccionado **Soluciones** en el panel de la izquierda. Para resolver este problema, tendrá que volver a implementar la solución siguiendo las instrucciones de [Cómo incorporar Azure Monitor para contenedores](container-insights-onboard.md). |  

Para ayudar a diagnosticar el problema, hemos incluido un script de solución de problemas que está disponible [aquí](https://github.com/Microsoft/OMS-docker/tree/ci_feature_prod/Troubleshoot#troubleshooting-script).  

## <a name="next-steps"></a>Pasos siguientes
Con la supervisión habilitada para capturar métricas de estado para los pods y nodos del clúster de AKS, estas métricas de estado están disponibles en Azure Portal. Para obtener información sobre cómo usar Azure Monitor para contenedores, consulte el artículo sobre cómo [ver el estado de Azure Kubernetes Service](container-insights-analyze.md).