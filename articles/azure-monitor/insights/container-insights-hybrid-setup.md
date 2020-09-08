---
title: Configuración de los clústeres híbridos de Kubernetes con Azure Monitor para contenedores | Microsoft Docs
description: En este artículo se describe cómo puede configurar Azure Monitor para contenedores con el fin de supervisar clústeres de Kubernetes hospedados en Azure Stack u otro entorno.
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: 26846148f3212699cecd6db3318cd2da2d9aa783
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89398388"
---
# <a name="configure-hybrid-kubernetes-clusters-with-azure-monitor-for-containers"></a>Configuración de los clústeres híbridos de Kubernetes con Azure Monitor para contenedores

Azure Monitor para contenedores proporciona una experiencia de supervisión enriquecida para Azure Kubernetes Service (AKS) y el [Motor de AKS en Azure](https://github.com/Azure/aks-engine), que es un clúster de Kubernetes autoadministrado, hospedados en Azure. En este artículo se describe cómo habilitar la supervisión de clústeres de Kubernetes hospedados fuera de Azure y lograr una experiencia de supervisión similar.

## <a name="supported-configurations"></a>Configuraciones admitidas

Las siguientes configuraciones se admiten oficialmente con Azure Monitor para contenedores.

- Entornos:

    - Kubernetes en un entorno local.
    
    - Motor de AKS en Azure y Azure Stack. Para más información, consulte [Motor de AKS en Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview?view=azs-1908).
    
    - [OpenShift](https://docs.openshift.com/container-platform/4.3/welcome/index.html) versión 4 y posteriores, en el entorno local o en otros entornos en la nube.

- Las versiones de Kubernetes y de la directiva de soporte son las mismas que las versiones de [AKS compatibles](../../aks/supported-kubernetes-versions.md).

- Se admiten los siguientes entornos de ejecución de contenedores: Docker, Moby y entornos de ejecución compatibles con CRI, como CRI-O y ContainerD.

- La versión del sistema operativo Linux admitida para los nodos maestro y de trabajo es: Ubuntu (18.04 LTS y 16.04 LTS) y Red Hat Enterprise Linux CoreOS 43.81.

- Control de acceso compatible: RBAC de Kubernetes y no RBAC

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar, asegúrese de que dispone de lo siguiente:

- Un [área de trabajo de Log Analytics.](../platform/design-logs-deployment.md)

    Azure Monitor para contenedores admite un área de trabajo de Log Analytics en las regiones enumeradas en los [productos por región](https://azure.microsoft.com/global-infrastructure/services/?regions=all&products=monitor) de Azure. Para crear el área de trabajo, puede configurarla mediante [Azure Resource Manager](../platform/template-workspace-configuration.md), [PowerShell](../scripts/powershell-sample-create-workspace.md?toc=%2fpowershell%2fmodule%2ftoc.json) o en el [Azure Portal](../learn/quick-create-workspace.md).

    >[!NOTE]
    >No se admite la supervisión de varios clústeres con el mismo nombre de clúster en el mismo área de trabajo de Log Analytics. Los nombres de clúster deben ser únicos.
    >

- Es miembro del **rol de colaborador de Log Analytics** para habilitar la supervisión de contenedores. Para más información sobre cómo controlar el acceso a un área de trabajo de Log Analytics, consulte [Administración del acceso a las áreas de trabajo y los datos de registro](../platform/manage-access.md).

- Para ver los datos de supervisión, debe tener el rol [*Lector de Log Analytics*](../platform/manage-access.md#manage-access-using-azure-permissions) en el área de trabajo de Log Analytics configurada con Azure Monitor para contenedores.

- [Cliente de HELM](https://helm.sh/docs/using_helm/) para incorporar gráfico de Azure Monitor para contenedores para el clúster de Kubernetes especificado.

- La información de la tabla siguiente muestra la configuración de proxy y firewall requerida para que la versión en contenedor del agente de Log Analytics para Linux se comunique con Azure Monitor:

    |Recurso del agente|Puertos |
    |------|---------|
    |*.ods.opinsights.azure.com |Puerto 443 |
    |*.oms.opinsights.azure.com |Puerto 443 |
    |*.dc.services.visualstudio.com |Puerto 443 |

- El agente en contenedor requiere `cAdvisor secure port: 10250` o `unsecure port :10255` de Kubelet para que se abra en todos los nodos del clúster para recopilar las métricas de rendimiento. Se recomienda configurar `secure port: 10250` en cAdvisor de Kubelet si aún no está configurado.

- El agente en contenedor requiere que se especifiquen las siguientes variables de entorno en el contenedor para comunicarse con el servicio de API de Kubernetes en el clúster para recopilar datos de inventario -`KUBERNETES_SERVICE_HOST` y `KUBERNETES_PORT_443_TCP_PORT`.

>[!IMPORTANT]
>La versión de agente mínima admitida para la supervisión de clústeres híbridos de Kubernetes es ciprod10182019 o posterior.

## <a name="enable-monitoring"></a>Habilitar supervisión

La habilitación de Azure Monitor para contenedores para el clúster híbrido de Kubernetes consiste en realizar los pasos siguientes en orden.

1. Configure el área de trabajo de Log Analytics con la solución de Container Insights.   

2. Habilite el gráfico HELM de Azure Monitor para contenedores con el área de trabajo de Log Analytics.

Para obtener más información sobre las soluciones de supervisión en Azure Monitor, diríjase [aquí](../../azure-monitor/insights/solutions.md).

### <a name="how-to-add-the-azure-monitor-containers-solution"></a>Cómo agregar la solución Azure Monitor para contenedores

Con la plantilla de Azure Resource Manager proporcionada puede implementar la solución con el cmdlet `New-AzResourceGroupDeployment` de Azure PowerShell o con CLI de Azure.

Si no conoce el concepto de implementación de recursos mediante una plantilla, consulte:

- [Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell](../../azure-resource-manager/templates/deploy-powershell.md)

- [Implementación de recursos con plantillas de Resource Manager y la CLI de Azure](../../azure-resource-manager/templates/deploy-cli.md)

Si decide usar la CLI de Azure, primero debe instalar y usar la CLI localmente. Debe ejecutar la versión 2.0.59 de la CLI de Azure, o cualquier versión posterior. Para identificar la versión, ejecute `az --version`. Si necesita instalar o actualizar la CLI de Azure, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

Este método incluye dos plantillas JSON. Una plantilla especifica la configuración para habilitar la supervisión y la otra contiene los valores de los parámetros que se configuran para especificar lo siguiente:

- **workspaceResourceId**: el identificador de recurso completo del área de trabajo de Log Analytics.
- **workspaceRegion**: la región en la que se crea el área de trabajo, que también se conoce como **Ubicación** en las propiedades del área de trabajo cuando se visualiza desde el Azure Portal.

Para localizar primero el identificador de recurso completo del área de trabajo de Log Analytics necesario para el valor del parámetro `workspaceResourceId` en el archivo **containerSolutionParams.json**, realice los pasos siguientes y luego ejecute el cmdlet de PowerShell o el comando CLI de Azure para agregar la solución.

1. Enumere todas las suscripciones a las que tiene acceso con el siguiente comando:

    ```azurecli
    az account list --all -o table
    ```

    La salida será similar a la siguiente:

    ```azurecli
    Name                                  CloudName    SubscriptionId                        State    IsDefault
    ------------------------------------  -----------  ------------------------------------  -------  -----------
    Microsoft Azure                       AzureCloud   0fb60ef2-03cc-4290-b595-e71108e8f4ce  Enabled  True
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

    En la salida, busque el nombre del área de trabajo y copie el identificador de recurso completo de esa área de trabajo de Log Analytics en el campo **identificador**.

4. Copie y pegue la siguiente sintaxis JSON en el archivo:

    ```json
    {
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceResourceId": {
            "type": "string",
            "metadata": {
                "description": "Azure Monitor Log Analytics Workspace Resource ID"
            }
        },
        "workspaceRegion": {
            "type": "string",
            "metadata": {
                "description": "Azure Monitor Log Analytics Workspace region"
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.Resources/deployments",
            "name": "[Concat('ContainerInsights', '-',  uniqueString(parameters('workspaceResourceId')))]",
            "apiVersion": "2017-05-10",
            "subscriptionId": "[split(parameters('workspaceResourceId'),'/')[2]]",
            "resourceGroup": "[split(parameters('workspaceResourceId'),'/')[4]]",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {},
                    "variables": {},
                    "resources": [
                        {
                            "apiVersion": "2015-11-01-preview",
                            "type": "Microsoft.OperationsManagement/solutions",
                            "location": "[parameters('workspaceRegion')]",
                            "name": "[Concat('ContainerInsights', '(', split(parameters('workspaceResourceId'),'/')[8], ')')]",
                            "properties": {
                                "workspaceResourceId": "[parameters('workspaceResourceId')]"
                            },
                            "plan": {
                                "name": "[Concat('ContainerInsights', '(', split(parameters('workspaceResourceId'),'/')[8], ')')]",
                                "product": "[Concat('OMSGallery/', 'ContainerInsights')]",
                                "promotionCode": "",
                                "publisher": "Microsoft"
                            }
                        }
                    ]
                },
                "parameters": {}
            }
         }
      ]
   }
    ```

5. Guarde este archivo como containerSolution.json en una carpeta local.

6. Pegue la siguiente sintaxis de JSON en el archivo:

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "workspaceResourceId": {
          "value": "<workspaceResourceId>"
      },
      "workspaceRegion": {
        "value": "<workspaceRegion>"
      }
     }
    }
    ```

7. Edite los valores de **workspaceResourceId** con el valor que copió en el paso 3, y para **workspaceRegion** copie el valor de **Región** después de ejecutar el comando CLI de Azure [az monitor log-analytics workspace show](/cli/azure/monitor/log-analytics/workspace?view=azure-cli-latest#az-monitor-log-analytics-workspace-list).

8. Guarde este archivo como containerSolutionParams.json en una carpeta local.

9. Está listo para implementar esta plantilla.

   - Para implementar con Azure PowerShell, use los siguientes comandos en la carpeta que contiene la plantilla:

       ```powershell
       # configure and login to the cloud of Log Analytics workspace.Specify the corresponding cloud environment of your workspace to below command.
       Connect-AzureRmAccount -Environment <AzureCloud | AzureChinaCloud | AzureUSGovernment>
       ```

       ```powershell
       # set the context of the subscription of Log Analytics workspace
       Set-AzureRmContext -SubscriptionId <subscription Id of Log Analytics workspace>
       ```

       ```powershell
       # execute deployment command to add Container Insights solution to the specified Log Analytics workspace
       New-AzureRmResourceGroupDeployment -Name OnboardCluster -ResourceGroupName <resource group of Log Analytics workspace> -TemplateFile .\containerSolution.json -TemplateParameterFile .\containerSolutionParams.json
       ```

       El cambio de configuración puede tardar unos minutos en completarse. Cuando se completa, se muestra un mensaje que incluye el resultado y que es similar al siguiente:

       ```powershell
       provisioningState       : Succeeded
       ```

   - Para implementar con la CLI de Azure, ejecute los siguientes comandos:

       ```azurecli
       az login
       az account set --name <AzureCloud | AzureChinaCloud | AzureUSGovernment>
       az login
       az account set --subscription "Subscription Name"
       # execute deployment command to add container insights solution to the specified Log Analytics workspace
       az deployment group create --resource-group <resource group of log analytics workspace> --name <deployment name> --template-file  ./containerSolution.json --parameters @./containerSolutionParams.json
       ```

       El cambio de configuración puede tardar unos minutos en completarse. Cuando se completa, se muestra un mensaje que incluye el resultado y que es similar al siguiente:

       ```azurecli
       provisioningState       : Succeeded
       ```

       Después de habilitar la supervisión, pueden pasar unos 15 minutos hasta que pueda ver la métrica de estado del clúster.

## <a name="install-the-helm-chart"></a>Instalación del gráfico de HELM

En esta sección, instalará el agente en contenedor para Azure Monitor para contenedores. Antes de continuar, debe identificar el identificador del área de trabajo necesario para el parámetro `omsagent.secret.wsid` y la clave principal necesaria para el parámetro `omsagent.secret.key`. Puede identificar esta información con los pasos siguientes y, a continuación, ejecutar los comandos para instalar el agente mediante el gráfico de HELM.

1. Ejecute el siguiente comando para identificar el identificador del área de trabajo:

    `az monitor log-analytics workspace list --resource-group <resourceGroupName>`

    En la salida, busque el nombre del área de trabajo en el campo **name** y, a continuación, copie el identificador del área de trabajo de Log Analytics del campo **customerID**.

2. Ejecute el siguiente comando para identificar la clave principal del área de trabajo:

    `az monitor log-analytics workspace get-shared-keys --resource-group <resourceGroupName> --workspace-name <logAnalyticsWorkspaceName>`

    En la salida, busque la clave principal en el campo **primarySharedKey** y, a continuación, copie el valor.

>[!NOTE]
>Los siguientes comandos solo se aplican a la versión 2 de Helm. El uso del parámetro `--name` no es aplicable a la versión 3 de Helm. 

>[!NOTE]
>Si el clúster de Kubernetes se comunica mediante un servidor proxy, configure el parámetro `omsagent.proxy` con la dirección URL del servidor proxy. Si el clúster no se comunica mediante un servidor proxy, no es necesario especificar este parámetro. Para más información, consulte [Configuración del punto de conexión proxy](#configure-proxy-endpoint) más adelante en este artículo.

3. Agregue el repositorio de gráficos de Azure a la lista local mediante la ejecución del siguiente comando:

    ```
    helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
    ````

4. Instale el gráfico ejecutando el comando siguiente:

    ```
    $ helm install --name myrelease-1 \
    --set omsagent.secret.wsid=<logAnalyticsWorkspaceId>,omsagent.secret.key=<logAnalyticsWorkspaceKey>,omsagent.env.clusterName=<my_prod_cluster> incubator/azuremonitor-containers
    ```

    Si el área de trabajo de Log Analytics está en Azure China 21Vianet, ejecute el siguiente comando:

    ```
    $ helm install --name myrelease-1 \
     --set omsagent.domain=opinsights.azure.cn,omsagent.secret.wsid=<logAnalyticsWorkspaceId>,omsagent.secret.key=<logAnalyticsWorkspaceKey>,omsagent.env.clusterName=<your_cluster_name> incubator/azuremonitor-containers
    ```

    Si el área de trabajo de Log Analytics está en Azure Gobierno de EE. UU, ejecute el siguiente comando:

    ```
    $ helm install --name myrelease-1 \
    --set omsagent.domain=opinsights.azure.us,omsagent.secret.wsid=<logAnalyticsWorkspaceId>,omsagent.secret.key=<logAnalyticsWorkspaceKey>,omsagent.env.clusterName=<your_cluster_name> incubator/azuremonitor-containers
    ```

### <a name="enable-the-helm-chart-using-the-api-model"></a>Habilitación del gráfico de Helm mediante el modelo de API

Puede especificar un complemento en el archivo JSON de especificaciones del clúster del motor de AKS, también conocido como el modelo de API. En este complemento, proporcione la versión codificada en Base 64 de `WorkspaceGUID` y `WorkspaceKey` del área de trabajo de Log Analytics donde se almacenan los datos de supervisión recopilados. Puede encontrar el valor de `WorkspaceGUID` y `WorkspaceKey` mediante los pasos 1 y 2 de la sección anterior.

Las definiciones de API admitidas para el clúster de Azure Stack Hub se pueden encontrar en este ejemplo: [kubernetes-container-monitoring_existing_workspace_id_and_key.json](https://github.com/Azure/aks-engine/blob/master/examples/addons/container-monitoring/kubernetes-container-monitoring_existing_workspace_id_and_key.json). En concreto, busque la propiedad **addons** en **kubernetesConfig**:

```json
"orchestratorType": "Kubernetes",
       "kubernetesConfig": {
         "addons": [
           {
             "name": "container-monitoring",
             "enabled": true,
             "config": {
               "workspaceGuid": "<Azure Log Analytics Workspace Id in Base-64 encoded>",
               "workspaceKey": "<Azure Log Analytics Workspace Key in Base-64 encoded>"
             }
           }
         ]
       }
```

## <a name="configure-agent-data-collection"></a>Configuración del agente de la recopilación de datos

Al iniciarse con la versión 1.0.0 del gráfico, la configuración de la recopilación de datos del agente se controla desde ConfigMap. Consulte la documentación sobre la configuración de la recopilación de datos del agente [aquí](container-insights-agent-config.md).

Después de implementar correctamente el gráfico, puede revisar los datos de su clúster híbrido de Kubernetes en Azure Monitor para contenedores desde el Azure Portal.  

>[!NOTE]
>La latencia de ingesta de datos es de entre cinco y diez minutos desde el agente para confirmarse en el área de trabajo de Azure Log Analytics. El estado del clúster muestra el valor **Sin datos** o **Desconocido** hasta que todos los datos de supervisión necesarios estén disponibles en Azure Monitor.

## <a name="configure-proxy-endpoint"></a>Configuración del punto de conexión proxy

A partir de la versión 2.7.1 del gráfico, el gráfico admitirá la especificación del punto de conexión proxy con el parámetro `omsagent.proxy` del gráfico. Esto permite que se comunique mediante el servidor proxy. La comunicación entre el agente de Azure Monitor para contenedores y Azure Monitor puede ser un servidor proxy HTTP o HTTPS y se admiten la autenticación anónima y básica (nombre de usuario/contraseña).

El valor de configuración del servidor proxy tiene la siguiente sintaxis: `[protocol://][user:password@]proxyhost[:port]`.

> [!NOTE]
>Si el servidor proxy no requiere autenticación, tendrá que especificar un nombre de usuario y una contraseña ficticios. Puede ser cualquier nombre de usuario o contraseña.

|Propiedad| Descripción |
|--------|-------------|
|Protocolo | http o https |
|usuario | Nombre de usuario opcional para la autenticación de proxy |
|password | Contraseña opcional para la autenticación de proxy |
|proxyhost | Dirección o FQDN del servidor proxy |
|port | Número de puerto opcional para el servidor proxy |

Por ejemplo: `omsagent.proxy=http://user01:password@proxy01.contoso.com:8080`

Si especifica el protocolo como **http**, las solicitudes HTTP se crean con una conexión segura SSL/TLS. El servidor proxy debe admitir los protocolos SSL y TLS.

## <a name="troubleshooting"></a>Solución de problemas

Si se produce un error al intentar habilitar la supervisión para el clúster híbrido de Kubernetes, copie el script de PowerShell [TroubleshootError_nonAzureK8s.ps1](https://raw.githubusercontent.com/microsoft/OMS-docker/ci_feature/Troubleshoot/TroubleshootError_nonAzureK8s.ps1) y guárdelo en una carpeta del equipo. Este script se proporciona para ayudar a detectar y corregir los problemas encontrados. Se ha diseñado para detectar e intentar corregir los siguientes problemas:

- El área de trabajo de Log Analytics especificada es válida
- El área de trabajo Log Analytics se configura con la solución Azure Monitor para contenedores. Si no es así, configure el área de trabajo.
- Los pods de replicaset de OmsAgent están en ejecución
- Los pods de daemonset de OmsAgent están en ejecución
- El servicio Health de OmsAgent está en ejecución
- El identificador y la clave del área de trabajo Log Analytics configurados en el agente en contenedor coinciden con el área de trabajo con la que se configura la información.
- Valide que todos los nodos de trabajo de Linux tienen la etiqueta `kubernetes.io/role=agent` para programar el pod de rs. Si no existe, agréguela.
- Compruebe que `cAdvisor secure port:10250` o `unsecure port: 10255` estén abiertos en todos los nodos del clúster.

Para realizar la ejecución con Azure PowerShell, use los siguientes comandos en la carpeta que contiene el script:

```powershell
.\TroubleshootError_nonAzureK8s.ps1 - azureLogAnalyticsWorkspaceResourceId </subscriptions/<subscriptionId>/resourceGroups/<resourcegroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName> -kubeConfig <kubeConfigFile> -clusterContextInKubeconfig <clusterContext>
```

## <a name="next-steps"></a>Pasos siguientes

Con la supervisión habilitada para recopilar el estado y la utilización de recursos de su clúster híbrido de Kubernetes y las cargas de trabajo que se ejecutan en ellos, aprenda [cómo usar](container-insights-analyze.md) Azure Monitor para contenedores.
