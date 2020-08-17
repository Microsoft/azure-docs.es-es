---
title: 'Inicio rápido: Implementación de un clúster de AKS mediante PowerShell'
description: Aprenda a crear rápidamente un clúster de Kubernetes, implementar una aplicación y supervisar el rendimiento en Azure Kubernetes Service (AKS) mediante PowerShell.
services: container-service
ms.topic: quickstart
ms.date: 05/26/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 219772fd81617dc3e69b452057e0122eedef69f9
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87499864"
---
# <a name="quickstart-deploy-an-azure-kubernetes-service-cluster-using-powershell"></a>Inicio rápido: Implementación de un clúster de Azure Kubernetes Service mediante PowerShell

En este inicio rápido, se va a implementar un clúster de Azure Kubernetes Service (AKS) mediante PowerShell. AKS es un servicio de Kubernetes administrado que permite implementar y administrar clústeres rápidamente. En el clúster se ejecuta una aplicación de varios contenedores que incluye un front-end web y una instancia de Redis. A continuación, verá cómo supervisar el mantenimiento del clúster y los pods que ejecutan la aplicación.

Para más información sobre cómo crear un grupo de nodos de Windows Server, consulte [Creación de un clúster de AKS que admita contenedores de Windows Server][windows-container-powershell].

![Aplicación de votación implementada en Azure Kubernetes Service](./media/kubernetes-walkthrough-powershell/voting-app-deployed-in-azure-kubernetes-service.png)

En esta guía rápida se presupone un conocimiento básico de los conceptos de Kubernetes. Para más información, consulte [Conceptos básicos de Kubernetes de Azure Kubernetes Service (AKS)][kubernetes-concepts].

## <a name="prerequisites"></a>Prerrequisitos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

Si decide usar PowerShell de forma local, para este artículo es preciso que instale el módulo Az PowerShell y que se conecte a su cuenta de Azure con el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/Connect-AzAccount). Para más información sobre cómo instalar el módulo Az PowerShell, consulte [Instalación de Azure PowerShell][install-azure-powershell].

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

Si tiene varias suscripciones a Azure, elija la suscripción adecuada en la que se debe facturar el recurso. Seleccione un identificador de suscripción específico con el cmdlet [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) es un grupo lógico en el que se implementan y administran recursos de Azure. Cuando se crea un grupo de recursos, se le pide que especifique una ubicación. Dicha ubicación es donde se almacenan los metadatos del grupo de recursos, así como el lugar en el que los recursos se ejecutan en Azure si no se especifica otra región al crear los recursos. Cree un grupo de recursos con el cmdlet [New-AzResourceGroup][new-azresourcegroup].

En el ejemplo siguiente, se crea un grupo de recursos denominado **myResourceGroup** en la región **eastus**.

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location eastus
```

En la siguiente salida de ejemplo se muestra que los recursos se crearon correctamente:

```plaintext
ResourceGroupName : myResourceGroup
Location          : eastus
ProvisioningState : Succeeded
Tags              :
ResourceId        : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup
```

## <a name="create-aks-cluster"></a>Creación de un clúster de AKS

Use la utilidad de línea de comandos `ssh-keygen` para generar un par de claves SSH. Para más información, consulte [Pasos rápidos: Creación y uso de un par de claves pública-privada SSH para máquinas virtuales Linux en Azure](../virtual-machines/linux/mac-create-ssh-keys.md).

Use el cmdlet [New-AzAks][new-azaks] para crear un clúster de AKS. En el siguiente ejemplo se crea un clúster denominado **myAKSCluster** con un nodo. Azure Monitor para contenedores también está habilitado de forma predeterminada. La operación tarda varios minutos en completarse.

> [!NOTE]
> Al crear un clúster de AKS, se crea automáticamente un segundo grupo de recursos para almacenar los recursos de AKS. Para más información, consulte [¿Por qué se crean dos grupos de recursos con AKS?](./faq.md#why-are-two-resource-groups-created-with-aks)

```azurepowershell-interactive
New-AzAks -ResourceGroupName myResourceGroup -Name myAKSCluster -NodeCount 1
```

Transcurridos unos minutos, el comando se completa y devuelve información en formato JSON sobre el clúster.

## <a name="connect-to-the-cluster"></a>Conectarse al clúster

Para administrar un clúster de Kubernetes, usará [kubectl][kubectl], el cliente de línea de comandos de Kubernetes. Si usa Azure Cloud Shell, `kubectl` ya está instalado. Para instalar `kubectl` localmente, use el cmdlet `Install-AzAksKubectl`:

```azurepowershell
Install-AzAksKubectl
```

Para configurar `kubectl` para conectarse al clúster de Kubernetes, use el cmdlet [Import-AzAksCredential][import-azakscredential]. En el ejemplo siguiente se descargan las credenciales y se configura la CLI de Kubernetes para usarlas.

```azurepowershell-interactive
Import-AzAksCredential -ResourceGroupName myResourceGroup -Name myAKSCluster
```

Para comprobar la conexión al clúster, use el comando [kubectl get][kubectl-get] para devolver una lista de los nodos del clúster.

```azurepowershell-interactive
.\kubectl get nodes
```

La salida del ejemplo siguiente muestra el nodo único creado en los pasos anteriores. Asegúrese de que el estado del nodo es **Listo**:

```plaintext
NAME                       STATUS   ROLES   AGE     VERSION
aks-nodepool1-31718369-0   Ready    agent   6m44s   v1.15.10
```

## <a name="run-the-application"></a>Ejecución de la aplicación

Un archivo de manifiesto de Kubernetes define un estado deseado del clúster, por ejemplo, qué imágenes de contenedor se van a ejecutar. En esta guía de inicio rápido, se usa un manifiesto para crear todos los objetos necesarios para ejecutar la aplicación Azure Vote. Este manifiesto incluye dos [implementaciones de Kubernetes][kubernetes-deployment], una para las aplicaciones Python de Azure Vote de ejemplo y otra para una instancia de Redis. También se crean dos servicios de Kubernetes, uno interno para la instancia de Redis y otro externo para acceder a la aplicación Azure Vote desde Internet.

> [!TIP]
> En esta guía de inicio rápido, se crean e implementan manualmente los manifiestos de aplicación en el clúster de AKS.
> En escenarios más reales, puede usar [Azure Dev Spaces][azure-dev-spaces] para iterar rápidamente y depurar el código directamente en el clúster de AKS. Puede usar Dev Spaces entre plataformas de sistemas operativos y entornos de desarrollo, y trabajar junto con otras personas de su equipo.

Cree un archivo denominado `azure-vote.yaml` y cópielo en la siguiente definición de código YAML. Si usa Azure Cloud Shell, este archivo se puede crear mediante `vi` o `nano` como si funcionara en un sistema físico o virtual:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: azure-vote-back
spec:
  replicas: 1
  selector:
    matchLabels:
      app: azure-vote-back
  template:
    metadata:
      labels:
        app: azure-vote-back
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": linux
      containers:
      - name: azure-vote-back
        image: redis
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 6379
          name: redis
---
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-back
spec:
  ports:
  - port: 6379
  selector:
    app: azure-vote-back
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: azure-vote-front
spec:
  replicas: 1
  selector:
    matchLabels:
      app: azure-vote-front
  template:
    metadata:
      labels:
        app: azure-vote-front
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": linux
      containers:
      - name: azure-vote-front
        image: microsoft/azure-vote-front:v1
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 80
        env:
        - name: REDIS
          value: "azure-vote-back"
---
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-front
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: azure-vote-front
```

Implemente la aplicación mediante el comando [kubectl apply][kubectl-apply] y especifique el nombre del manifiesto de YAML:

```azurepowershell-interactive
.\kubectl apply -f azure-vote.yaml
```

En la salida de ejemplo siguiente se muestran las implementaciones y los servicios creados correctamente:

```plaintext
deployment.apps/azure-vote-back created
service/azure-vote-back created
deployment.apps/azure-vote-front created
service/azure-vote-front created
```

## <a name="test-the-application"></a>Prueba de la aplicación

Cuando se ejecuta la aplicación, un servicio de Kubernetes expone el front-end de la aplicación a Internet.
Este proceso puede tardar unos minutos en completarse.

Para supervisar el progreso, utilice el comando [kubectl get service][kubectl-get] con el argumento `--watch`.

```azurepowershell-interactive
.\kubectl get service azure-vote-front --watch
```

En un primer momento, el parámetro **EXTERNAL-IP** del servicio **azure-vote-front** se muestra como **pendiente**.

```plaintext
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
azure-vote-front   LoadBalancer   10.0.37.27   <pending>     80:30572/TCP   6s
```

Cuando la dirección **EXTERNAL-IP** cambie de **pendiente** a una dirección IP pública real, use `CTRL-C` para detener el proceso de inspección de `kubectl`. En la salida del ejemplo siguiente se muestra una dirección IP pública válida asignada al servicio:

```plaintext
azure-vote-front   LoadBalancer   10.0.37.27   52.179.23.131   80:30572/TCP   2m
```

Para ver la aplicación Azure Vote en acción, abra un explorador web en la dirección IP externa del servicio.

![Aplicación de votación implementada en Azure Kubernetes Service](./media/kubernetes-walkthrough-powershell/voting-app-deployed-in-azure-kubernetes-service.png)

Cuando se creó el clúster de AKS, se habilitó [Azure Monitor para contenedores](../azure-monitor/insights/container-insights-overview.md) para capturar métricas de mantenimiento tanto para los nodos de clúster como para los pods. Estas métricas de mantenimiento están disponibles en Azure Portal.

## <a name="delete-the-cluster"></a>Eliminación del clúster

Para evitar los cargos de Azure, se recomienda limpiar los recursos que no sean necesarios. Cuando el clúster ya no se necesite, use el comando [Remove-AzResourceGroup][remove-azresourcegroup] para eliminar el grupo de recursos, el servicio de contenedor y todos los recursos relacionados.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
```

> [!NOTE]
> Cuando elimina el clúster, la entidad de servicio Azure Active Directory que utiliza el clúster de AKS no se quita. Para conocer los pasos que hay que realizar para quitar la entidad de servicio, consulte [Consideraciones principales y eliminación de AKS][sp-delete]. Si usó una identidad administrada, esta está administrada por la plataforma y no requiere que la quite.

## <a name="get-the-code"></a>Obtención del código

En este inicio rápido, se han usado imágenes de un contenedor creado previamente para crear una implementación de Kubernetes. El código de la aplicación relacionada, Dockerfile, y el archivo de manifiesto de Kubernetes están disponibles en GitHub.

[https://github.com/Azure-Samples/azure-voting-app-redis][azure-vote-app]

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha implementado un clúster de Kubernetes y una aplicación de varios contenedores en él. También puede [acceder al panel web de Kubernetes][kubernetes-dashboard] del clúster de AKS.

Para obtener más información sobre AKS y un ejemplo completo desde el código hasta la implementación, continúe con el tutorial del clúster de Kubernetes.

> [!div class="nextstepaction"]
> [Tutorial de AKS][aks-tutorial]

<!-- LINKS - external -->
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[azure-dev-spaces]: ../dev-spaces/index.yml
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[azure-vote-app]: https://github.com/Azure-Samples/azure-voting-app-redis.git

<!-- LINKS - internal -->
[windows-container-powershell]: windows-container-powershell.md
[kubernetes-concepts]: concepts-clusters-workloads.md
[install-azure-powershell]: /powershell/azure/install-az-ps
[new-azresourcegroup]: /powershell/module/az.resources/new-azresourcegroup
[new-azaks]: /powershell/module/az.aks/new-azaks
[import-azakscredential]: /powershell/module/az.aks/import-azakscredential
[kubernetes-deployment]: concepts-clusters-workloads.md#deployments-and-yaml-manifests
[kubernetes-service]: concepts-network.md#services
[remove-azresourcegroup]: /powershell/module/az.resources/remove-azresourcegroup
[sp-delete]: kubernetes-service-principal.md#additional-considerations
[kubernetes-dashboard]: kubernetes-dashboard.md
[aks-tutorial]: ./tutorial-kubernetes-prepare-app.md
