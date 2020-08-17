---
title: Interacción con contenedores de Windows
services: azure-dev-spaces
ms.date: 01/16/2020
ms.topic: conceptual
description: Obtenga información sobre cómo ejecutar Azure Dev Spaces en un clúster existente con contenedores de Windows.
keywords: Azure Dev Spaces, Dev Spaces, Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, contenedores, contenedores de Windows
ms.openlocfilehash: deee02bf13778950b934d87d02eb21068d89f323
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88077057"
---
# <a name="interact-with-windows-containers-using-azure-dev-spaces"></a>Uso de Azure Dev Spaces para interactuar con contenedores de Windows

Puede habilitar Azure Dev Spaces en los espacios de nombres de Kubernetes nuevos y existentes. Azure Dev Spaces ejecutará e instrumentará servicios que se ejecutan en contenedores de Linux. Estos servicios también pueden interactuar con aplicaciones que se ejecutan en contenedores de Windows en el mismo espacio de nombres. En este artículo se muestra cómo usar Dev Spaces para ejecutar servicios en un espacio de nombres con contenedores de Windows existentes. En este momento, no se pueden depurar o adjuntar a contenedores de Windows con Azure Dev Spaces.

## <a name="set-up-your-cluster"></a>Configuración del clúster

En este artículo se da por supuesto que ya tiene un clúster con grupos de nodos de Windows y Linux. Si necesita crear un clúster con grupos de nodos de Windows y Linux, puede seguir las instrucciones que se indican [aquí][windows-container-cli].

Conecte el clúster con [kubectl][kubectl], el cliente de la línea de comandos de Kubernetes. Para configurar `kubectl` para conectarse a su clúster de Kubernetes, use el comando [az aks get-credentials][az-aks-get-credentials]. Con este comando se descargan las credenciales y se configura la CLI de Kubernetes para usarlas.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Para comprobar la conexión al clúster, use el comando [kubectl get][kubectl-get] para devolver una lista de los nodos del clúster.

```azurecli-interactive
kubectl get nodes
```

En la siguiente salida de ejemplo se muestra un clúster tanto con un nodo de Windows como con uno de Linux. Antes de continuar, asegúrese de que el estado de cada nodo es *Listo*.

```console
NAME                                STATUS   ROLES   AGE    VERSION
aks-nodepool1-12345678-vmss000000   Ready    agent   13m    v1.14.8
aks-nodepool1-12345678-vmss000001   Ready    agent   13m    v1.14.8
aksnpwin000000                      Ready    agent   108s   v1.14.8
```

Aplique un valor [taint][using-taints] a los nodos de Windows. El valor taint en los nodos de Windows evita que Dev Spaces programe los contenedores de Linux para que se ejecuten en los nodos de Windows. El comando de ejemplo siguiente aplica un valor de taint al nodo de Windows *aksnpwin987654* del ejemplo anterior.

```azurecli-interactive
kubectl taint node aksnpwin987654 sku=win-node:NoSchedule
```

> [!IMPORTANT]
> Cuando aplica un valor taint a un nodo, debe configurar un valor de toleration coincidente en la plantilla de implementación del servicio a fin de ejecutar el servicio en ese nodo. La aplicación de ejemplo ya está configurada con un valor de [toleration coincidente][sample-application-toleration-example] con el valor de taint que configuró en el comando anterior.

## <a name="run-your-windows-service"></a>Ejecución del servicio de Windows

Ejecute el servicio de Windows en el clúster de AKS y compruebe que se encuentra en estado *en ejecución*. En este artículo se usa una [aplicación de ejemplo][sample-application] para mostrar un servicio de Windows y Linux que se ejecuta en el clúster.

Clone la aplicación de ejemplo desde GitHub y vaya al directorio `dev-spaces/samples/existingWindowsBackend/mywebapi-windows`:

```console
git clone https://github.com/Azure/dev-spaces
cd dev-spaces/samples/existingWindowsBackend/mywebapi-windows
```

La aplicación de ejemplo usa [Helm 3][helm-installed] para ejecutar el servicio de Windows en el clúster. Vaya al directorio `charts` y use la ejecución de Helm para servicio de Windows:

```console
cd charts/
kubectl create ns dev
helm install windows-service . --namespace dev
```

El comando anterior usa Helm para ejecutar el servicio de Windows en el espacio de nombres *dev*. Si no tiene un espacio de nombres denominado *dev*, se creará.

Use el comando `kubectl get pods` para comprobar que el servicio de Windows se está ejecutando en el clúster. 

```console
$ kubectl get pods --namespace dev --watch
NAME                     READY   STATUS              RESTARTS   AGE
myapi-4b9667d123-1a2b3   0/1     ContainerCreating   0          47s
...
myapi-4b9667d123-1a2b3   1/1     Running             0          98s
```

## <a name="enable-azure-dev-spaces"></a>Habilitación de Azure Dev Spaces

Habilite Dev Spaces en el mismo espacio de nombres que usó para ejecutar el servicio de Windows. El siguiente comando habilita Dev Spaces en el espacio de nombres *dev*:

```console
az aks use-dev-spaces -g myResourceGroup -n myAKSCluster --space dev --yes
```

## <a name="update-your-windows-service-for-dev-spaces"></a>Actualización del servicio de Windows para Dev Spaces

Al habilitar Dev Spaces en un espacio de nombres existente con contenedores que ya se están ejecutando, de manera predeterminada, Dev Spaces probará e instrumentará los contenedores nuevos que se ejecuten en ese espacio de nombres. Dev Spaces también probará e instrumentará los nuevos contenedores creados para el servicio que ya se está ejecutando en el espacio de nombres. Para evitar que Dev Spaces instrumente un contenedor que se ejecuta en el espacio de nombres, agregue el encabezado *sin proxy* a `deployment.yaml`.

Agregue`azds.io/no-proxy: "true"` al archivo `existingWindowsBackend/mywebapi-windows/charts/templates/deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  ...
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    ...
  template:
    metadata:
      labels:
        app.kubernetes.io/name: {{ include "mywebapi.name" . }}
        app.kubernetes.io/instance: {{ .Release.Name }}
        azds.io/no-proxy: "true"
```

Use `helm list` para mostrar la implementación del servicio de Windows:

```cmd
$ helm list --namespace dev
NAME              REVISION  UPDATED                     STATUS      CHART           APP VERSION NAMESPACE
windows-service 1           Wed Jul 24 15:45:59 2019    DEPLOYED    mywebapi-0.1.0  1.0         dev  
```

En el ejemplo anterior, el nombre de la implementación es *windows-service*. Actualice el servicio de Windows con la nueva configuración mediante `helm upgrade`:

```cmd
helm upgrade windows-service . --namespace dev
```

Como ha actualizado `deployment.yaml`, Dev Spaces no probará ni instrumentará el servicio.

## <a name="run-your-linux-application-with-azure-dev-spaces"></a>Ejecución de la aplicación de Linux con Azure Dev Spaces

Vaya al directorio `webfrontend` y use los comandos `azds prep` y `azds up` para ejecutar la aplicación de Linux en el clúster.

```console
cd ../../webfrontend-linux/
azds prep --enable-ingress
azds up
```

El comando `azds prep --enable-ingress` genera el gráfico de Helm y Dockerfile para la aplicación.

> [!TIP]
> Azure Dev Spaces usa [Dockerfile y el gráfico de Helm](../how-dev-spaces-works-prep.md#prepare-your-code) del proyecto para compilar y ejecutar el código, pero el usuario puede modificar estos archivos si desea cambiar la forma en que se compila y se ejecuta el proyecto.

El comando `azds up` ejecuta el servicio en el espacio de nombres.

```console
$ azds up
Using dev space 'dev' with target 'myAKSCluster'
Synchronizing files...4s
Installing Helm chart...11s
Waiting for container image build...6s
Building container image...
Step 1/12 : FROM mcr.microsoft.com/dotnet/core/sdk:2.2
...
Step 12/12 : ENTRYPOINT ["/bin/bash", "/entrypoint.sh"]
Built container image in 36s
Waiting for container...2s
Service 'webfrontend' port 'http' is available at http://dev.webfrontend.abcdef0123.eus.azds.io/
Service 'webfrontend' port 80 (http) is available via port forwarding at http://localhost:57648
```

Para ver el servicio que se está ejecutando, abra la dirección URL pública que aparece en la salida del comando azds up. En este ejemplo, la dirección URL pública es `http://dev.webfrontend.abcdef0123.eus.azds.io/`. Vaya al servicio en un explorador y haga clic en *Acerca de* en la parte superior. Compruebe que ve un mensaje del servicio *mywebapi* que contiene la versión de Windows que usa el contenedor.

![Aplicación de ejemplo que muestra la versión de Windows de mywebapi](../media/run-dev-spaces-windows-containers/sample-app.png)

## <a name="next-steps"></a>Pasos siguientes

Obtenga información acerca de la forma en que Azure Dev Spaces le ayuda a desarrollar aplicaciones más complejas en varios contenedores y cómo puede simplificar el desarrollo en colaboración mediante el uso de distintas versiones o bifurcaciones del código en distintos espacios.

> [!div class="nextstepaction"]
> [Desarrollo en equipo en Azure Dev Spaces][team-development-qs]

[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[helm-installed]: https://helm.sh/docs/intro/install/
[sample-application]: https://github.com/Azure/dev-spaces/tree/master/samples/existingWindowsBackend
[sample-application-toleration-example]: https://github.com/Azure/dev-spaces/blob/master/samples/existingWindowsBackend/mywebapi-windows/charts/templates/deployment.yaml#L24-L27
[team-development-qs]: ../quickstart-team-development.md
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[using-taints]: ../../aks/use-multiple-node-pools.md#schedule-pods-using-taints-and-tolerations
[windows-container-cli]: ../../aks/windows-container-cli.md
