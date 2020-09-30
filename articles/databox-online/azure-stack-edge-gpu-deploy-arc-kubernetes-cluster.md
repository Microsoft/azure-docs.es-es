---
title: Habilitación de Azure Arc en Kubernetes en un dispositivo Azure Stack Edge Pro con GPU | Microsoft Docs
description: Se describe cómo habilitar Azure Arc en un clúster de Kubernetes existente en el dispositivo Azure Stack Edge Pro con GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 09/01/2020
ms.author: alkohli
ms.openlocfilehash: 423345739ca5c078fbff4f267e1e8a118abf107c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90903189"
---
# <a name="enable-azure-arc-on-kubernetes-cluster-on-your-azure-stack-edge-pro-gpu-device"></a>Habilitación de Azure Arc en un clúster de Kubernetes en el dispositivo Azure Stack Edge Pro con GPU

En este artículo se muestra cómo habilitar Azure Arc en un clúster de Kubernetes existente en el dispositivo Azure Stack Edge Pro. 

Este procedimiento está dirigido a los usuarios que han examinado las [cargas de trabajo de Kubernetes en un dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-kubernetes-workload-management.md) y están familiarizados con los conceptos de [¿Qué es Kubernetes habilitado para Azure Arc (versión preliminar)?](https://docs.microsoft.com/azure/azure-arc/kubernetes/overview).


## <a name="prerequisites"></a>Requisitos previos

Antes de habilitar Azure Arc en un clúster de Kubernetes, asegúrese de que ha completado los siguientes requisitos previos en el dispositivo Azure Stack Edge Pro y en el cliente que vaya a usar para acceder al dispositivo:

### <a name="for-device"></a>Para el dispositivo

1. Tiene credenciales de inicio de sesión en un dispositivo Azure Stack Edge Pro de un nodo.
    1. El dispositivo está activado. Consulte [Activación del dispositivo](azure-stack-edge-gpu-deploy-activate.md).
    1. El dispositivo tiene el rol de proceso configurado desde Azure Portal y un clúster de Kubernetes. Consulte [Configuración del proceso](azure-stack-edge-gpu-deploy-configure-compute.md).

1. Tiene acceso de propietario a la suscripción. Necesita este acceso durante el paso de asignación de roles para la entidad de servicio.
 

### <a name="for-client-accessing-the-device"></a>Para el cliente que va a acceder al dispositivo

1. Tiene un sistema cliente de Windows que se usará para acceder al dispositivo Azure Stack Edge Pro.
  
    - El cliente ejecuta Windows PowerShell 5.0 o una versión posterior. Para descargar la última versión de Windows PowerShell, vaya a [Instalación de Windows PowerShell](https://docs.microsoft.com/powershell/scripting/install/installing-windows-powershell?view=powershell-7).
    
    - También puede utilizar cualquier otro cliente con un [sistema operativo compatible](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device). En este artículo, el procedimiento que se describe emplea un cliente Windows. 
    
1. Ha completado el procedimiento descrito en [Acceso al clúster de Kubernetes en el dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md). Ha:
    
    - Instalado `kubectl` en el cliente.  <!--and saved the `kubeconfig` file with the user configuration to C:\\Users\\&lt;username&gt;\\.kube. -->
    
    - Asegúrese de que la versión del cliente de `kubectl` es como máximo una versión superior o inferior a la versión maestra de Kubernetes que se ejecuta en el dispositivo Azure Stack Edge Pro. 
      - Use `kubectl version` para comprobar la versión de kubectl que se ejecuta en el cliente. Anote la versión completa.
      - En la interfaz de usuario local del dispositivo Azure Stack Edge Pro, vaya a **Actualización de software** y anote el número de versión del servidor de Kubernetes. 
    
        ![Comprobación del número de versión del servidor de Kubernetes](media/azure-stack-edge-gpu-connect-powershell-interface/verify-kubernetes-version-1.png)      
      
      - Compruebe que estas dos versiones son compatibles. 

<!-- az cli version requirements-->

## <a name="register-kubernetes-resource-providers"></a>Registro de proveedores de recursos de Kubernetes

Antes de habilitar Azure Arc en el clúster de Kubernetes, deberá habilitar y registrar `Microsoft.Kubernetes` y `Microsoft.KubernetesConfiguration` en su suscripción. 

1. Para habilitar un proveedor de recursos, en Azure Portal, vaya a la suscripción que piensa usar para la implementación. Vaya a **Proveedores de recursos**. 
1. En el panel de la derecha, busque los proveedores que desee agregar. En este ejemplo, `Microsoft.Kubernetes` y `Microsoft.KubernetesConfiguration`.

    ![Registro de proveedores de recursos de Kubernetes](media/azure-stack-edge-gpu-connect-powershell-interface/register-k8-resource-providers-1.png)

1. Seleccione un proveedor de recursos y, en la parte superior de la barra de comandos, seleccione **Registrar**. El proceso de registro tarda varios minutos. 

    ![Registro de proveedores de recursos de Kubernetes](media/azure-stack-edge-gpu-connect-powershell-interface/register-k8-resource-providers-2.png)

1. Actualice la interfaz de usuario hasta que vea que el proveedor de recursos está registrado. Repita el proceso con ambos proveedores de recursos.
    
    ![Registro de proveedores de recursos de Kubernetes](media/azure-stack-edge-gpu-connect-powershell-interface/register-k8-resource-providers-4.png)

También puede registrar proveedores de recursos con `az cli`. Para más información, consulte [Registre los dos proveedores para Kubernetes habilitado para Azure Arc](../azure-arc/kubernetes/connect-cluster.md#register-the-two-providers-for-azure-arc-enabled-kubernetes).

## <a name="create-service-principal-assign-role"></a>Creación de una entidad de servicio y asignación del rol

1. Asegúrese de que tiene el valor de `Subscription ID` y el nombre del grupo de recursos que usó para la implementación de recursos en el servicio de Azure Stack Edge. Para obtener el identificador de suscripción, vaya al recurso de Azure Stack Edge en Azure Portal. Vaya a **Información general > Información esencial**.

    ![Obtener id. de suscripción](media/azure-stack-edge-gpu-connect-powershell-interface/get-subscription-id-1.png)

    Para obtener el nombre del grupo de recursos, vaya a **Propiedades**.

    ![Obtención del nombre del grupo de recursos](media/azure-stack-edge-gpu-connect-powershell-interface/get-resource-group-name-1.png)

1. Para crear una entidad de servicio, use el siguiente comando en `az cli`.

    `az ad sp create-for-rbac --skip assignment --name "<Informative name for service principal>"`  

    Para obtener información sobre cómo iniciar sesión en `az cli`, consulte [Inicio de Cloud Shell en Azure Portal](../cloud-shell/quickstart-powershell.md?view=azure-cli-latest#start-cloud-shell).

    A continuación se muestra un ejemplo: 
    
    ```azurecli
    PS /home/user> az ad sp create-for-rbac --skip-assignment --name "https://azure-arc-for-ase-k8s"
    {
      "appId": "aa8a082e-0fa1-4a82-b51c-e8b2a9fdaa8b",
      "displayName": "azure-arc-for-ase-k8s",
      "name": "https://azure-arc-for-ase-k8s",
      "password": "<password>",
      "tenant": "72f988bf-86f1-41af-91ab-2d7cd011db47"
    }
    PS /home/user>
    ```

1. Tome nota de los valores de `appID`, `name`, `password` y `tenantID`, ya que los utilizará como entrada en el comando siguiente.

1. Después de crear la nueva entidad de servicio, asígnele el rol `Kubernetes Cluster - Azure Arc Onboarding`. Se trata de un rol integrado de Azure (use el identificador de rol en el comando) con permisos limitados. Use el comando siguiente:

    `az role assignment create --role 34e09817-6cbe-4d01-b1a2-e0eac5743d41 --assignee <appId-from-service-principal> --scope /subscriptions/<SubscriptionID>/resourceGroups/<Resource-group-name>`

    A continuación se muestra un ejemplo:
    
    ```azurecli
    PS /home/user> az role assignment create --role 34e09817-6cbe-4d01-b1a2-e0eac5743d41 --assignee aa8a082e-0fa1-4a82-b51c-e8b2a9fdaa8b --scope /subscriptions/062c67a6-019b-40af-a775-c4dc1abe56ed/resourceGroups/myaserg1
    {
      "canDelegate": null,
      "id": "/subscriptions/062c67a6-019b-40af-a775-c4dc1abe56ed/resourceGroups/myaserg1/providers/Microsoft.Authorization/roleAssignments/59272f92-e5ce-4aeb-9c0c-62532d8caf25",
      "name": "59272f92-e5ce-4aeb-9c0c-62532d8caf25",
      "principalId": "b045b3fe-8745-4097-9674-91cb0afaad91",
      "principalType": "ServicePrincipal",
      "resourceGroup": "myaserg1",
      "roleDefinitionId": "/subscriptions/062c67a6-019b-40af-a775-c4dc1abe56ed/providers/Microsoft.Authorization/roleDefinitions/34e09817-6cbe-4d01-b1a2-e0eac5743d41",
      "scope": "/subscriptions/062c67a6-019b-40af-a775-c4dc1abe56ed/resourceGroups/myaserg1",
      "type": "Microsoft.Authorization/roleAssignments"
    }
    PS /home/user>
    ```
    Para obtener más información sobre cómo crear una entidad de servicio y realizar la asignación de roles, consulte los pasos de [Creación de una entidad de servicio de incorporación habilitada para Azure Arc](https://docs.microsoft.com/azure/azure-arc/kubernetes/create-onboarding-service-principal).


## <a name="enable-arc-on-kubernetes-cluster"></a>Habilitación de Arc en el clúster de Kubernetes

Siga estos pasos para configurar el clúster de Kubernetes para la administración de Azure Arc:

1. [Conéctese a la interfaz de PowerShell](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface) del dispositivo.

1. Escriba:

    `Set-HcsKubernetesAzureArcAgent -SubscriptionId "<Your Azure Subscription Id>" -ResourceGroupName "<Resource Group Name>" -ResourceName "<Azure Arc resource name (shouldn't exist already)>" -Location "<Region associated with resource group>" -TenantId "<Tenant Id of service principal>" -ClientId "<App id of service principal>" -ClientSecret "<Password of service principal>"`

    Para implementar Azure Arc en un dispositivo Azure Stack Edge Pro, asegúrese de que está usando una [región admitida para Azure Arc](../azure-arc/kubernetes/overview.md#supported-regions). Azure Arc se encuentra actualmente en versión preliminar. Si utiliza el comando `az account list-locations`, también puede obtener el nombre exacto de la región que se va a pasar en el cmdlet.
    
    Este es un ejemplo:
   
    ```powershell
    [10.128.44.240]: PS>Set-HcsKubernetesAzureArcAgent -SubscriptionId "062c67a6-019b-40af-a775-c4dc1abe56ed" -ResourceGroupName "myaserg1" -ResourceName "myasetestresarc" -Location "westeurope" -TenantId "72f988bf-86f1-41af-91ab-2d7cd011db47" -ClientId "aa8a082e-0fa1-4a82-b51c-e8b2a9fdaa8b" -ClientSecret "<password>"
        [10.128.44.240]: PS>
    ```
    
    En Azure Portal, se debe crear un recurso con el nombre que ha proporcionado en el comando anterior.

    ![Ir al recurso de Azure Arc](media/azure-stack-edge-gpu-connect-powershell-interface/verify-azure-arc-enabled-1.png)

1. Para comprobar que Azure Arc se ha habilitado correctamente, ejecute el siguiente comando desde la interfaz de PowerShell:

    `kubectl get deployments -n azure-arc`

    Este comando busca todas las aplicaciones que se implementan en el espacio de nombres `azure-arc` correspondiente a Azure Arc.

    Este es un ejemplo de salida que muestra los agentes de Azure Arc que se implementaron en el clúster de Kubernetes en el espacio de nombres `azure-arc`. 


    ```powershell
    [10.128.44.240]: PS>kubectl get deployments -n azure-arc
    NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
    cluster-metadata-operator   1/1     1            1           45m
    clusteridentityoperator     1/1     1            1           45m
    config-agent                1/1     1            1           45m
    connect-agent               1/1     1            1           45m
    controller-manager          1/1     1            1           45m
    flux-logs-agent             1/1     1            1           45m
    metrics-agent               1/1     1            1           45m
    resource-sync-agent         1/1     1            1           45m
    [10.128.44.240]: PS>
    ```

    También puede obtener una lista de los pods que se ejecutan en el clúster de Kubernetes en el espacio de nombres `azure-arc`. Un pod es un contenedor o un proceso de la aplicación que se ejecuta en el clúster de Kubernetes. 

    Use el comando siguiente:
    
    `kubectl get pods -n azure-arc`
    
    Esta es una salida de ejemplo:
    
    ```powershell
    [10.128.44.240]: PS>kubectl get pods -n azure-arc
    NAME                                         READY   STATUS    RESTARTS   AGE
    cluster-metadata-operator-64cbdf95b4-s2q52   2/2     Running   0          16m
    clusteridentityoperator-6f6dbccf7-nwnxg      3/3     Running   0          16m
    config-agent-7df5bf497b-mjm8k                3/3     Running   0          16m
    connect-agent-5d4c766764-m7h46               1/1     Running   0          16m
    controller-manager-777555fb57-t7tdp          3/3     Running   0          16m
    flux-logs-agent-845476c899-zcmtj             2/2     Running   0          16m
    metrics-agent-84d6fc8f4d-g9jkm               2/2     Running   0          16m
    resource-sync-agent-8f88dbf96-zgxjj          3/3     Running   0          16m
    [10.128.44.240]: PS>
    ```


Como muestra la salida anterior, Kubernetes habilitado para Azure Arc consta de algunos agentes (operadores) que se ejecutan en el clúster implementado en el espacio de nombres `azure-arc`.

- `config-agent`: busca en el clúster conectado los recursos de configuración de control de código fuente aplicados en el clúster y actualiza el estado de cumplimiento.
- `controller-manager`: es un operador de operadores y organiza las interacciones entre los componentes de Azure Arc.
- `metrics-agent`: recopila métricas de otros agentes de Arc para asegurarse de que presentan un rendimiento óptimo.
- `cluster-metadata-operator`: Recopila metadatos del clúster (la versión del clúster, el número de nodos y la versión del agente de Azure Arc).
- `resource-sync-agent`: sincroniza con Azure los metadatos de clúster mencionados anteriormente.
- `clusteridentityoperator`: Kubernetes habilitado para Azure Arc admite actualmente la identidad asignada por el sistema. clusteridentityoperator mantiene el certificado de Managed Service Identity (MSI) que usan otros agentes para la comunicación con Azure.
- `flux-logs-agent`: recopila registros de los operadores de Flux implementados como parte de la configuración de control de código fuente.
- `connect-agent`: se comunica con el recurso de Azure Arc.

### <a name="remove-arc-from-the-kubernetes-cluster"></a>Eliminación de Arc del clúster de Kubernetes

Para eliminar la administración de Azure Arc, siga estos pasos:

1. 1. [Conéctese a la interfaz de PowerShell](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface) del dispositivo.
2. Escriba:

    `Remove-HcsKubernetesAzureArcAgent` 


## <a name="next-steps"></a>Pasos siguientes

Para saber cómo ejecutar una implementación de Azure Arc, vea [Implementación de una aplicación PHP sin estado de libro de visitas con Redis mediante GitOps en un dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-stateless-application-git-ops-guestbook.md).
