---
title: Supervisión de un clúster de Azure Kubernetes Service (AKS) implementado | Microsoft Docs
description: Obtenga información sobre cómo habilitar la supervisión de un clúster de Azure Kubernetes Service (AKS) con Azure Monitor para contenedores ya implementados en la suscripción.
ms.topic: conceptual
ms.date: 09/12/2019
ms.custom: devx-track-terraform
ms.openlocfilehash: e3b4bd007ec2f4315ccd7d8581113e116db693ef
ms.sourcegitcommit: 1fe5127fb5c3f43761f479078251242ae5688386
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/14/2020
ms.locfileid: "90068434"
---
# <a name="enable-monitoring-of-azure-kubernetes-service-aks-cluster-already-deployed"></a>Habilitar la supervisión de un clúster de Azure Kubernetes Service (AKS) ya implementado

En este artículo se describe cómo configurar Azure Monitor para contenedores para supervisar un clúster de Kubernetes administrado, hospedado en [Azure Kubernetes Service](../../aks/index.yml), que ya se han implementado en la suscripción.

Puede habilitar la supervisión de un clúster de AKS que ya se haya implementado mediante uno de los métodos admitidos:

* Azure CLI
* Terraform
* [Desde Azure Monitor](#enable-from-azure-monitor-in-the-portal) o [directamente desde un clúster de AKS](#enable-directly-from-aks-cluster-in-the-portal) en Azure Portal.
* Con la [plantilla de Azure Resource Manager proporcionada](#enable-using-an-azure-resource-manager-template) con el cmdlet `New-AzResourceGroupDeployment` de Azure PowerShell o con la CLI de Azure.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="enable-using-azure-cli"></a>Habilitación del uso de la CLI de Azure

El paso siguiente habilita la supervisión del clúster de AKS mediante la CLI de Azure. En este ejemplo, no es necesario especificar ni crear previamente un área de trabajo existente. Este comando le simplifica el proceso al crear un área de trabajo predeterminada en el grupo de recursos predeterminado de la suscripción del clúster de AKS, si aún no existe en la región.  El área de trabajo predeterminada creada es similar al formato de *DefaultWorkspace-\<GUID>-\<Region>* .

```azurecli
az aks enable-addons -a monitoring -n MyExistingManagedCluster -g MyExistingManagedClusterRG
```

La salida será similar a la siguiente:

```output
provisioningState       : Succeeded
```

### <a name="integrate-with-an-existing-workspace"></a>Integración en un área de trabajo existente

Si prefiere realizar la integración en un área de trabajo existente, realice los pasos siguientes para detectar primero el identificador del recurso completo de su área de trabajo de Log Analytics necesario para el parámetro `--workspace-resource-id` y ejecute el comando para habilitar el complemento de supervisión en el área de trabajo especificada.

1. Enumere todas las suscripciones a las que tiene acceso con el siguiente comando:

    ```azurecli
    az account list --all -o table
    ```

    La salida será similar a la siguiente:

    ```output
    Name                                  CloudName    SubscriptionId                        State    IsDefault
    ------------------------------------  -----------  ------------------------------------  -------  -----------
    Microsoft Azure                       AzureCloud   68627f8c-91fO-4905-z48q-b032a81f8vy0  Enabled  True
    ```

    Copie el valor de **SubscriptionId**.

2. Cambie a la suscripción que hospeda el área de trabajo de Log Analytics con el siguiente comando:

    ```azurecli
    az account set -s <subscriptionId of the workspace>
    ```

3. En el ejemplo siguiente se muestra la lista de áreas de trabajo de sus suscripciones en el formato JSON predeterminado.

    ```azurecli
    az resource list --resource-type Microsoft.OperationalInsights/workspaces -o json
    ```

    En la salida, busque el nombre del área de trabajo y copie el identificador de recurso completo de esa área de trabajo de Log Analytics en el **identificador** de campo.

4. Reemplace el valor del parámetro `--workspace-resource-id` para ejecutar el comando siguiente y habilitar el complemento de supervisión. El valor de cadena debe estar entre comillas dobles:

    ```azurecli
    az aks enable-addons -a monitoring -n ExistingManagedCluster -g ExistingManagedClusterRG --workspace-resource-id "/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<WorkspaceName>"
    ```

    La salida será similar a la siguiente:

    ```output
    provisioningState       : Succeeded
    ```

## <a name="enable-using-terraform"></a>Habilitación con Terraform

1. Agregue el perfil de complemento **oms_agent** al [recurso azurerm_kubernetes_cluster](https://www.terraform.io/docs/providers/azurerm/d/kubernetes_cluster.html#addon_profile) existente.

   ```
   addon_profile {
    oms_agent {
      enabled                    = true
      log_analytics_workspace_id = "${azurerm_log_analytics_workspace.test.id}"
     }
   }
   ```

2. Agregue [azurerm_log_analytics_solution](https://www.terraform.io/docs/providers/azurerm/r/log_analytics_solution.html) siguiendo los pasos descritos en la documentación de Terraform.

## <a name="enable-from-azure-monitor-in-the-portal"></a>Habilitación desde Azure Monitor en el portal

Para habilitar la supervisión de un clúster de AKS en Azure Portal desde Azure Monitor, siga estos pasos:

1. En Azure Portal, seleccione **Monitor**.

2. Seleccione **Contenedores** en la lista.

3. En la página **Supervisión - Contenedores**, seleccione **Clústeres sin supervisar**.

4. En la lista de clústeres sin supervisar, busque el contenedor y haga clic en **Habilitar**.

5. En la página **Incorporación a Azure Monitor para contenedores**, si tiene un área de trabajo de Log Analytics ya existente en la misma suscripción que el clúster, selecciónela de la lista desplegable.
    La lista preselecciona el área de trabajo y la ubicación predeterminadas en las que se implementa el contenedor de AKS en la suscripción.

    ![Habilitación de la supervisión de conclusiones de contenedores de AKS](./media/container-insights-onboard/kubernetes-onboard-brownfield-01.png)

    >[!NOTE]
    >Si desea crear una área de trabajo de Log Analytics para almacenar los datos de supervisión del clúster, siga las instrucciones de [Creación de un área de trabajo de Log Analytics en Azure Portal](../learn/quick-create-workspace.md). Asegúrese de crear el área de trabajo en la misma suscripción en la que está implementado el contenedor de AKS.

Después de habilitar la supervisión, pueden pasar unos 15 minutos hasta que pueda ver la métrica de estado del clúster.

## <a name="enable-directly-from-aks-cluster-in-the-portal"></a>Habilitación directamente desde un clúster de AKS en el portal

Para habilitar la supervisión directamente desde un clúster de AKS en Azure Portal, siga estos pasos:

1. En Azure Portal, seleccione **Todos los servicios**.

2. En la lista de recursos, empiece a escribir **Containers**.  La lista se filtra en función de lo que escriba.

3. Seleccione **servicios de Kubernetes**.

    ![El vínculo de Servicios de Kubernetes](./media/container-insights-onboard/portal-search-containers-01.png)

4. En la lista de contenedores, seleccione uno de ellos.

5. En la página de información general del contenedor, seleccione **Contenedores de Monitor**.

6. En la página **Incorporación a Azure Monitor para contenedores**, si tiene un área de trabajo de Log Analytics ya existente en la misma suscripción que el clúster, selecciónela de la lista desplegable.
    La lista preselecciona el área de trabajo y la ubicación predeterminadas en las que se implementa el contenedor de AKS en la suscripción.

    ![Habilitación de la supervisión del mantenimiento de contenedores de AKS](./media/container-insights-onboard/kubernetes-onboard-brownfield-02.png)

    >[!NOTE]
    >Si desea crear una área de trabajo de Log Analytics para almacenar los datos de supervisión del clúster, siga las instrucciones de [Creación de un área de trabajo de Log Analytics en Azure Portal](../learn/quick-create-workspace.md). Asegúrese de crear el área de trabajo en la misma suscripción en la que está implementado el contenedor de AKS.

Después de habilitar la supervisión pueden pasar unos 15 minutos hasta que pueda ver los datos operativos del clúster.

## <a name="enable-using-an-azure-resource-manager-template"></a>Habilitación mediante una plantilla de Azure Resource Manager

Este método incluye dos plantillas JSON. Una plantilla especifica la configuración para habilitar la supervisión y la otra contiene los valores de los parámetros que se configuran para especificar lo siguiente:

* El identificador del recurso del contenedor de AKS.
* El grupo de recursos en que se implementa el clúster.

>[!NOTE]
>La plantilla debe implementarse en el mismo grupo de recursos que el clúster.
>

El área de trabajo de Log Analytics debe crearse antes de habilitar la supervisión con Azure PowerShell o la CLI. Para crear el área de trabajo, puede configurarla una mediante [Azure Resource Manager](../platform/template-workspace-configuration.md), [PowerShell](../scripts/powershell-sample-create-workspace.md?toc=%2fpowershell%2fmodule%2ftoc.json) o [Azure Portal](../learn/quick-create-workspace.md).

Si no conoce el concepto de implementación de recursos mediante una plantilla, consulte:

* [Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell](../../azure-resource-manager/templates/deploy-powershell.md)

* [Implementación de recursos con plantillas de Resource Manager y la CLI de Azure](../../azure-resource-manager/templates/deploy-cli.md)

Si decide usar la CLI de Azure, primero debe instalar y usar la CLI localmente. Debe ejecutar la versión 2.0.59 de la CLI de Azure, o cualquier versión posterior. Para identificar la versión, ejecute `az --version`. Si necesita instalar o actualizar la CLI de Azure, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

### <a name="create-and-execute-a-template"></a>Creación y ejecución de una plantilla

1. Copie y pegue la siguiente sintaxis JSON en el archivo:

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "aksResourceId": {
          "type": "string",
          "metadata": {
            "description": "AKS Cluster Resource ID"
          }
        },
        "aksResourceLocation": {
          "type": "string",
          "metadata": {
            "description": "Location of the AKS resource e.g. \"East US\""
          }
        },
        "aksResourceTagValues": {
          "type": "object",
          "metadata": {
            "description": "Existing all tags on AKS Cluster Resource"
          }
        },
        "workspaceResourceId": {
          "type": "string",
          "metadata": {
            "description": "Azure Monitor Log Analytics Resource ID"
          }
        }
      },
      "resources": [
        {
          "name": "[split(parameters('aksResourceId'),'/')[8]]",
          "type": "Microsoft.ContainerService/managedClusters",
          "location": "[parameters('aksResourceLocation')]",
          "tags": "[parameters('aksResourceTagValues')]",
          "apiVersion": "2018-03-31",
          "properties": {
            "mode": "Incremental",
            "id": "[parameters('aksResourceId')]",
            "addonProfiles": {
              "omsagent": {
                "enabled": true,
                "config": {
                  "logAnalyticsWorkspaceResourceID": "[parameters('workspaceResourceId')]"
                }
              }
            }
          }
        }
      ]
    }
    ```

2. Guarde este archivo como **existingClusterOnboarding.json** en una carpeta local.

3. Pegue la siguiente sintaxis de JSON en el archivo:

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "aksResourceId": {
          "value": "/subscriptions/<SubscriptionId>/resourcegroups/<ResourceGroup>/providers/Microsoft.ContainerService/managedClusters/<ResourceName>"
        },
        "aksResourceLocation": {
          "value": "<aksClusterLocation>"
        },
        "workspaceResourceId": {
          "value": "/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroup>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>"
        },
        "aksResourceTagValues": {
          "value": {
            "<existing-tag-name1>": "<existing-tag-value1>",
            "<existing-tag-name2>": "<existing-tag-value2>",
            "<existing-tag-nameN>": "<existing-tag-valueN>"
          }
        }
      }
    }
    ```

4. Edite los valores de **aksResourceId** y **aksResourceLocation** con los valores de la página **AKS Overview** (Introducción a AKS) del clúster de AKS. El valor de **workspaceResourceId** es el identificador de recurso completo del área de trabajo de Log Analytics e incluye el nombre del área de trabajo.

    Edite los valores de **aksResourceTagValues** para que coincidan con los valores de etiqueta existentes especificados para el clúster de AKS.

5. Guarde este archivo como **existingClusterParam.json** en una carpeta local.

6. Está listo para implementar esta plantilla.

   * Para implementar con Azure PowerShell, use los siguientes comandos en la carpeta que contiene la plantilla:

       ```powershell
       New-AzResourceGroupDeployment -Name OnboardCluster -ResourceGroupName <ResourceGroupName> -TemplateFile .\existingClusterOnboarding.json -TemplateParameterFile .\existingClusterParam.json
       ```

       El cambio de configuración puede tardar unos minutos en completarse. Cuando se completa, se muestra un mensaje que incluye el resultado y que es similar al siguiente:

       ```output
       provisioningState       : Succeeded
       ```

   * Para implementar con la CLI de Azure, ejecute los siguientes comandos:

       ```azurecli
       az login
       az account set --subscription "Subscription Name"
       az group deployment create --resource-group <ResourceGroupName> --template-file ./existingClusterOnboarding.json --parameters @./existingClusterParam.json
       ```

       El cambio de configuración puede tardar unos minutos en completarse. Cuando se completa, se muestra un mensaje que incluye el resultado y que es similar al siguiente:

       ```output
       provisioningState       : Succeeded
       ```

       Después de habilitar la supervisión, pueden pasar unos 15 minutos hasta que pueda ver la métrica de estado del clúster.

## <a name="verify-agent-and-solution-deployment"></a>Comprobar la implementación del agente y la solución

Con la versión del agente *06072018*, o cualquier versión posterior, puede comprobar que tanto el agente como la solución se han implementado correctamente. Con las versiones anteriores del agente, solo se puede comprobar la implementación del agente.

### <a name="agent-version-06072018-or-later"></a>Versión 06072018 del agente, o posterior

Ejecute el siguiente comando para comprobar que el agente se ha implementado correctamente.

```
kubectl get ds omsagent --namespace=kube-system
```

Si la salida se parece a la siguiente, la implementación se ha realizado correctamente:

```output
User@aksuser:~$ kubectl get ds omsagent --namespace=kube-system
NAME       DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
omsagent   2         2         2         2            2           beta.kubernetes.io/os=linux   1d
```

Si hay nodos de Windows Server en el clúster, puede ejecutar el siguiente comando para comprobar que el agente se ha implementado correctamente.

```
kubectl get ds omsagent-win --namespace=kube-system
```

Si la salida se parece a la siguiente, la implementación se ha realizado correctamente:

```output
User@aksuser:~$ kubectl get ds omsagent-win --namespace=kube-system
NAME                   DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                   AGE
omsagent-win           2         2         2         2            2           beta.kubernetes.io/os=windows   1d
```

Para comprobar la implementación de la solución, ejecute el siguiente comando:

```
kubectl get deployment omsagent-rs -n=kube-system
```

Si la salida se parece a la siguiente, la implementación se ha realizado correctamente:

```output
User@aksuser:~$ kubectl get deployment omsagent-rs -n=kube-system
NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE    AGE
omsagent   1         1         1            1            3h
```

### <a name="agent-version-earlier-than-06072018"></a>Versión del agente anterior a 06072018

Para comprobar que la versión del agente de Log Analytics anterior a *06072018* se ha implementado correctamente, ejecute el comando siguiente:

```
kubectl get ds omsagent --namespace=kube-system
```

Si la salida se parece a la siguiente, la implementación se ha realizado correctamente:

```output
User@aksuser:~$ kubectl get ds omsagent --namespace=kube-system
NAME       DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
omsagent   2         2         2         2            2           beta.kubernetes.io/os=linux   1d
```

## <a name="view-configuration-with-cli"></a>Visualización de la configuración con la CLI

Use el comando `aks show` para obtener detalles como si la solución está habilitada o no, cuál es el valor del identificador de recursos del área de trabajo de Log Analytics y detalles de resumen acerca del clúster.

```azurecli
az aks show -g <resourceGroupofAKSCluster> -n <nameofAksCluster>
```

Transcurridos unos minutos, el comando se completa y devuelve información en formato JSON acerca de la solución.  Los resultados del comando deben mostrar el perfil de complemento de supervisión y son similares a la salida del ejemplo siguiente:

```output
"addonProfiles": {
    "omsagent": {
      "config": {
        "logAnalyticsWorkspaceResourceID": "/subscriptions/<WorkspaceSubscription>/resourceGroups/<DefaultWorkspaceRG>/providers/Microsoft.OperationalInsights/workspaces/<defaultWorkspaceName>"
      },
      "enabled": true
    }
  }
```

## <a name="next-steps"></a>Pasos siguientes

* Si experimenta problemas al intentar incorporar la solución, consulte la [guía de solución de problemas](container-insights-troubleshoot.md).

* Con la supervisión habilitada para recopilar el estado y la utilización de recursos de su clúster AKS y las cargas de trabajo que se ejecutan en ellos, aprenda [cómo usar](container-insights-analyze.md) Azure Monitor para contenedores.

