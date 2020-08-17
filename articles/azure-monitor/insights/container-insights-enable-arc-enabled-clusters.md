---
title: Configuración de un clúster de Kubernetes habilitado para Azure Arc con Azure Monitor para contenedores | Microsoft Docs
description: En este artículo se describe cómo configurar la supervisión con Azure Monitor para contenedores en clústeres de Kubernetes habilitados para Azure Arc.
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: f8002b20f37ca5149c58ca3e29402916ebbc1333
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87092888"
---
# <a name="enable-monitoring-of-azure-arc-enabled-kubernetes-cluster"></a>Habilitación de la supervisión en el clúster de Kubernetes habilitado para Azure Arc

Azure Monitor para contenedores proporciona una experiencia de supervisión enriquecida para los clústeres de Azure Kubernetes Service (AKS) y Motor de AKS. En este artículo se describe cómo habilitar la supervisión de clústeres de Kubernetes hospedados fuera de Azure que están habilitados con Azure Arc para lograr una experiencia de supervisión similar.

Azure Monitor para contenedores puede habilitarse para una o varias implementaciones de Kubernetes existentes mediante PowerShell o un script de Bash.

## <a name="supported-configurations"></a>Configuraciones admitidas

Azure Monitor para contenedores admite la supervisión de Kubernetes habilitado para Azure Arc (versión preliminar) tal y como se describe en el artículo [Introducción](container-insights-overview.md), excepto para las siguientes características:

- Datos en directo (versión preliminar)

- [Recopilación de métricas](container-insights-update-metrics.md) de nodos y pods de clúster, y almacenamiento de estos en la base de datos de métricas de Azure Monitor

Lo siguiente se admite oficialmente con Azure Monitor para contenedores:

- Las versiones de Kubernetes y de la directiva de soporte son las mismas que las versiones de [AKS compatibles](../../aks/supported-kubernetes-versions.md).

- Se admiten los siguientes entornos de ejecución de contenedores: Docker, Moby y entornos de ejecución compatibles con CRI, como CRI-O y ContainerD.

- La versión del sistema operativo Linux admitida para los nodos maestro y de trabajo es: Ubuntu (18.04 LTS y 16.04 LTS).

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar, asegúrese de que dispone de lo siguiente:

- Un área de trabajo de Log Analytics.

    Azure Monitor para contenedores admite un área de trabajo de Log Analytics en las regiones enumeradas en los [productos por región](https://azure.microsoft.com/global-infrastructure/services/?regions=all&products=monitor) de Azure. Para crear el área de trabajo, puede configurarla mediante [Azure Resource Manager](../platform/template-workspace-configuration.md), [PowerShell](../scripts/powershell-sample-create-workspace.md?toc=%2fpowershell%2fmodule%2ftoc.json) o en el [Azure Portal](../learn/quick-create-workspace.md).

- Para habilitar y acceder a las características de Azure Monitor para contenedores debe ser miembro, como mínimo, del rol *Colaborador* de la suscripción de Azure y miembro del rol [*Colaborador de Log Analytics*](../platform/manage-access.md#manage-access-using-azure-permissions) del área de trabajo de Log Analytics configurada con Azure Monitor para contenedores.

- Es miembro del rol de [colaborador](../../role-based-access-control/built-in-roles.md#contributor) en el recurso de clúster de Azure Arc.

- Para ver los datos de supervisión, debe ser miembro del permiso de rol [*Lector de Log Analytics*](../platform/manage-access.md#manage-access-using-azure-permissions) en el área de trabajo de Log Analytics configurada con Azure Monitor para contenedores.

- [Cliente de HELM](https://helm.sh/docs/using_helm/) para incorporar gráfico de Azure Monitor para contenedores para el clúster de Kubernetes especificado.

- La información de la tabla siguiente muestra la configuración de proxy y firewall requerida para que la versión en contenedor del agente de Log Analytics para Linux se comunique con Azure Monitor:

    |Recurso del agente|Puertos |
    |------|---------|
    |`*.ods.opinsights.azure.com` |Puerto 443 |
    |`*.oms.opinsights.azure.com` |Puerto 443 |
    |`*.dc.services.visualstudio.com` |Puerto 443 |

- El agente en contenedor requiere `cAdvisor secure port: 10250` o `unsecure port :10255` de Kubelet para que se abra en todos los nodos del clúster para recopilar las métricas de rendimiento. Se recomienda configurar `secure port: 10250` en cAdvisor de Kubelet si aún no está configurado.

- El agente en contenedor requiere que se especifiquen las siguientes variables de entorno en el contenedor para comunicarse con el servicio de API de Kubernetes en el clúster para recopilar datos de inventario -`KUBERNETES_SERVICE_HOST` y `KUBERNETES_PORT_443_TCP_PORT`.

    >[!IMPORTANT]
    >La versión de agente mínima admitida para la supervisión de clústeres de Kubernetes habilitados para Arc es ciprod04162020 o posterior.

- [PowerShell Core](/powershell/scripting/install/installing-powershell?view=powershell-6) es necesario si habilita la supervisión con el método de script de PowerShell.

- [Bash versión 4](https://www.gnu.org/software/bash/) es necesario si habilita la supervisión con el método de script de Bash.

## <a name="identify-workspace-resource-id"></a>Identificación del id. del recurso de área de trabajo

Para habilitar la supervisión del clúster mediante PowerShell o el script de Bash que descargó anteriormente e integrarla con un área de trabajo de Log Analytics existente, realice los pasos siguientes para detectar primero el id. completo del recurso del área de trabajo de Log Analytics. Esto es necesario para el parámetro `workspaceResourceId` al ejecutar el comando para habilitar el complemento de supervisión en el área de trabajo especificada. Si no dispone de un área de trabajo para especificar, puede no incluir el parámetro `workspaceResourceId` y dejar que el script cree una nueva área de trabajo.

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

    ```
    az resource list --resource-type Microsoft.OperationalInsights/workspaces -o json
    ```

    En la salida, busque el nombre del área de trabajo y copie el identificador de recurso completo de esa área de trabajo de Log Analytics en el campo **identificador**.

## <a name="enable-monitoring-using-powershell"></a>Habilitación de la supervisión mediante PowerShell

1. Descargue y guarde el script que configura el clúster en una carpeta local con el complemento de supervisión mediante el uso de los siguientes comandos:

    ```powershell
    wget https://aka.ms/enable-monitoring-powershell-script -outfile enable-monitoring.ps1
    ```

2. Configure la variable `$azureArcClusterResourceId`; para ello, establezca los valores correspondientes para `subscriptionId`, `resourceGroupName` y `clusterName`, que representan el identificador del recurso de clúster de Kubernetes habilitado para Azure Arc.

    ```powershell
    $azureArcClusterResourceId = "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Kubernetes/connectedClusters/<clusterName>"
    ```

3. Configure la variable `$kubeContext` con el valor de **kube-context** del clúster mediante la ejecución del comando `kubectl config get-contexts`. Si quiere usar el contexto actual, establezca el valor en `""`.

    ```powershell
    $kubeContext = "<kubeContext name of your k8s cluster>"
    ```

4. Si quiere usar el área de trabajo de Log Analytics de Azure Monitor existente, configure la variable `$logAnalyticsWorkspaceResourceId` con el valor correspondiente que representa el id. de recurso del área de trabajo. De lo contrario, configure la variable en `""` y el script creará un área de trabajo predeterminada en el grupo de recursos predeterminado de la suscripción al clúster, si aún no existe en la región. El área de trabajo predeterminada creada es similar al formato de *DefaultWorkspace-\<SubscriptionID>-\<Region>* .

    ```powershell
    $logAnalyticsWorkspaceResourceId = “/subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/microsoft.operationalinsights/workspaces/<workspaceName>”
    ```

5. Si el clúster de Kubernetes habilitado para Arc se comunica mediante un servidor proxy, configure la variable `$proxyEndpoint` con la dirección URL del servidor proxy. Si el clúster no se comunica mediante un servidor proxy, no podrá configurar el valor en `""`.  Para más información, consulte [Configuración del punto de conexión proxy](#configure-proxy-endpoint) más adelante en este artículo.

6. Ejecute el siguiente comando para habilitar la supervisión.

    ```powershell
    .\enable-monitoring.ps1 -clusterResourceId $azureArcClusterResourceId -kubeContext $kubeContext -workspaceResourceId $logAnalyticsWorkspaceResourceId -proxyEndpoint $proxyEndpoint
    ```

Después de habilitar la supervisión, pueden pasar unos 15 minutos hasta que pueda ver la métrica de estado del clúster.

## <a name="enable-using-bash-script"></a>Habilitación mediante un script de Bash

Siga estos pasos para habilitar la supervisión mediante el script de Bash proporcionado.

1. Descargue y guarde el script que configura el clúster en una carpeta local con el complemento de supervisión mediante el uso de los siguientes comandos:

    ```bash
    curl -o enable-monitoring.sh -L https://aka.ms/enable-monitoring-bash-script
    ```

2. Configure la variable `azureArcClusterResourceId`; para ello, establezca los valores correspondientes para `subscriptionId`, `resourceGroupName` y `clusterName`, que representan el identificador de recurso del recurso de clúster de Kubernetes habilitado para Azure Arc.

    ```bash
    export azureArcClusterResourceId="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Kubernetes/connectedClusters/<clusterName>"
    ```

3. Configure la variable `kubeContext` con el valor de **kube-context** del clúster mediante la ejecución del comando `kubectl config get-contexts`. Si quiere usar el contexto actual, establezca el valor en `""`.

    ```bash
    export kubeContext="<kubeContext name of your k8s cluster>"
    ```

4. Si quiere usar el área de trabajo de Log Analytics de Azure Monitor existente, configure la variable `logAnalyticsWorkspaceResourceId` con el valor correspondiente que representa el id. de recurso del área de trabajo. De lo contrario, configure la variable en `""` y el script creará un área de trabajo predeterminada en el grupo de recursos predeterminado de la suscripción al clúster, si aún no existe en la región. El área de trabajo predeterminada creada es similar al formato de *DefaultWorkspace-\<SubscriptionID>-\<Region>* .

    ```bash
    export logAnalyticsWorkspaceResourceId=“/subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/microsoft.operationalinsights/workspaces/<workspaceName>”
    ```

5. Si el clúster de Kubernetes habilitado para Arc se comunica mediante un servidor proxy, configure la variable `proxyEndpoint` con la dirección URL del servidor proxy. Si el clúster no se comunica mediante un servidor proxy, no podrá configurar el valor en `""`. Para más información, consulte [Configuración del punto de conexión proxy](#configure-proxy-endpoint) más adelante en este artículo.

6. Para habilitar la supervisión del clúster, se proporcionan comandos diferentes según el escenario de implementación.

    Ejecute el siguiente comando para habilitar la supervisión con opciones predeterminadas, como el uso del kube-context actual, crear un área de trabajo de Log Analytics predeterminada y no especificar un servidor proxy:

    ```bash
    bash enable-monitoring.sh --resource-id $azureArcClusterResourceId
    ```

    Ejecute el siguiente comando para crear un área de trabajo de Log Analytics predeterminada y no especificar un servidor proxy:

    ```bash
   bash enable-monitoring.sh --resource-id $azureArcClusterResourceId --kube-context $kubeContext
    ```

    Ejecute el siguiente comando para usar un área de trabajo de Log Analytics existente y no especificar un servidor proxy:

    ```bash
    bash enable-monitoring.sh --resource-id $azureArcClusterResourceId --kube-context $kubeContext  --workspace-id $logAnalyticsWorkspaceResourceId
    ```

    Ejecute el siguiente comando para usar un área de trabajo de Log Analytics existente y especificar un servidor proxy:

    ```bash
    bash enable-monitoring.sh --resource-id $azureArcClusterResourceId --kube-context $kubeContext  --workspace-id $logAnalyticsWorkspaceResourceId --proxy $proxyEndpoint
    ```

Después de habilitar la supervisión, pueden pasar unos 15 minutos hasta que pueda ver la métrica de estado del clúster.

## <a name="configure-proxy-endpoint"></a>Configuración del punto de conexión proxy

Con el agente en contenedor para Azure Monitor para contenedores, puede configurar un punto de conexión proxy para permitir que se comunique a través del servidor proxy. La comunicación entre el agente en contenedor y Azure Monitor puede ser un servidor proxy HTTP o HTTPS y se admiten la autenticación anónima y básica (nombre de usuario/contraseña).

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

Por ejemplo: `http://user01:password@proxy01.contoso.com:3128`

Si especifica el protocolo como **http**, las solicitudes HTTP se crean con una conexión segura SSL/TLS. El servidor proxy debe admitir los protocolos SSL y TLS.

### <a name="configure-using-powershell"></a>Configuración mediante PowerShell

Especifique el nombre de usuario y la contraseña, la dirección IP o FQDN y el número de puerto del servidor proxy. Por ejemplo:

```powershell
$proxyEndpoint = https://<user>:<password>@<proxyhost>:<port>
```

### <a name="configure-using-bash"></a>Configuración mediante Bash

Especifique el nombre de usuario y la contraseña, la dirección IP o FQDN y el número de puerto del servidor proxy. Por ejemplo:

```bash
export proxyEndpoint=https://<user>:<password>@<proxyhost>:<port>
```

## <a name="next-steps"></a>Pasos siguientes

- Con la supervisión habilitada para recopilar el estado y la utilización de recursos de su clúster de Kubernetes habilitado para Arc y las cargas de trabajo que se ejecutan en ellos, aprenda [cómo usar](container-insights-analyze.md) Azure Monitor para contenedores.

- De forma predeterminada, el agente en contenedores recopila los registros de contenedor stdout y stderr de todos los contenedores que se ejecutan en todos los espacios de nombres excepto kube-system. Para configurar la recopilación de registros de contenedor específica de uno o varios espacios de nombres determinados, consulte [Configuración del agente de Container Insights](container-insights-agent-config.md) para configurar las opciones de recopilación de datos que desee en el archivo de configuraciones ConfigMap.

- Para extraer y analizar las métricas de Prometheus desde el clúster, consulte [Configuración de la extracción de métricas de Prometheus](container-insights-prometheus-integration.md).

- Para aprender a detener la supervisión del clúster de Kubernetes habilitado para Arc con Azure Monitor para contenedores, consulte [Cómo detener la supervisión de su clúster híbrido](container-insights-optout-hybrid.md#how-to-stop-monitoring-on-arc-enabled-kubernetes).
