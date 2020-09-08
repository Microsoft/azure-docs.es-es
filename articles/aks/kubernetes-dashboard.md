---
title: Administración de un clúster de Azure Kubernetes Service con el panel web
description: Obtenga información sobre cómo usar el panel integrado de la interfaz de usuario web de Kubernetes para administrar un clúster de Azure Kubernetes Service (AKS)
services: container-service
author: mlearned
ms.topic: article
ms.date: 06/03/2020
ms.author: mlearned
ms.openlocfilehash: 35424c0a9e566a9dfa780c524e23945348335040
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88225995"
---
# <a name="access-the-kubernetes-web-dashboard-in-azure-kubernetes-service-aks"></a>Acceso al panel web de Kubernetes en Azure Kubernetes Service (AKS)

Kubernetes incluye un panel web que se puede usar para operaciones básicas de administración. Este panel le permite ver el estado de mantenimiento básico y las métricas para sus aplicaciones, crear e implementar servicios, y modificar las aplicaciones existentes. Este artículo muestra cómo acceder al panel de Kubernetes mediante la CLI de Azure y luego lo guía por algunas operaciones básicas del panel.

Para más información sobre el panel de Kubernetes, consulte la información sobre el [panel de la interfaz de usuario web de Kubernetes][kubernetes-dashboard]. AKS usa la versión 2.0 y posterior del panel de código abierto.

> [!WARNING]
> **El complemento del panel de AKS está programado para quedar en desuso. Use la [vista de recursos de Kubernetes en Azure Portal (versión preliminar)][kubernetes-portal] en su lugar.** 
> * El panel de Kubernetes está habilitado de forma predeterminada para los clústeres que ejecutan una versión de Kubernetes inferior a la 1.18.
> * El complemento de panel se deshabilitará de forma predeterminada para todos los nuevos clústeres creados en Kubernetes 1.18 o posterior. 
 > * A partir de Kubernetes 1.19 en versión preliminar, AKS ya no admitirá la instalación del complemento administrado kube-dashboard. 
 > * Los clústeres actuales con el complemento habilitado no se verán afectados. Los usuarios seguirán pudiendo instalar manualmente el panel de código abierto como software instalado por el usuario.

## <a name="before-you-begin"></a>Antes de empezar

En los pasos que se detallan en este documento se da por hecho que ha creado un clúster de AKS y que ha establecido una conexión `kubectl` con dicho clúster. Si necesita crear un clúster de AKS, consulte [Inicio rápido: Implementación de un clúster de Azure Kubernetes Service mediante la CLI de Azure][aks-quickstart].

También es preciso que esté instalada y configurada la versión 2.6.0 de la CLI de Azure u otra versión posterior. Ejecute  `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte  [Install Azure CLI][install-azure-cli] (Instalación de la CLI de Azure).

## <a name="disable-the-kubernetes-dashboard"></a>Deshabilitación del panel de Kubernetes

El complemento kube-dashboard está **habilitado de forma predeterminada en los clústeres anteriores a K8s 1.18**. El complemento se puede deshabilitar ejecutando el siguiente comando.

``` azure-cli
az aks disable-addons -g myRG -n myAKScluster -a kube-dashboard
```

## <a name="start-the-kubernetes-dashboard"></a>Inicio del panel de Kubernetes

Para iniciar el panel de Kubernetes en un clúster, use el comando [az aks browse][az-aks-browse]. Este comando requiere la instalación del complemento kube-dashboard en el clúster, que se incluye de forma predeterminada en los clústeres que ejecutan cualquier versión anterior a Kubernetes 1.18.

El ejemplo siguiente abre el panel para el clúster denominado *myAKSCluster* en el grupo de recursos denominado *myResourceGroup*:

```azurecli
az aks browse --resource-group myResourceGroup --name myAKSCluster
```

Este comando crea a un proxy entre el sistema de desarrollo y la API de Kubernetes y abre un explorador web en el panel de Kubernetes. Si un explorador web no se abre en el panel de Kubernetes, copie y pegue la dirección URL que anotó en la CLI de Azure, normalmente `http://127.0.0.1:8001`.

> [!NOTE]
> Si no ve el panel en `http://127.0.0.1:8001`, puede enrutar manualmente a las siguientes direcciones. Los clústeres que ejecutan la versión 1.16 o posterior usan HTTPS y requieren un punto de conexión independiente.
> * K8s 1.16 o posterior: `http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy`
> * K8s 1.15 y anterior: `http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/kubernetes-dashboard:/proxy`

<!--
![The login page of the Kubernetes web dashboard](./media/kubernetes-dashboard/dashboard-login.png)

You have the following options to sign in to your cluster's dashboard:

* A [kubeconfig file][kubeconfig-file]. You can generate a kubeconfig file using [az aks get-credentials][az-aks-get-credentials].
* A token, such as a [service account token][aks-service-accounts] or user token. On [AAD-enabled clusters][aad-cluster], this token would be an AAD token. You can use `kubectl config view` to list the tokens in your kubeconfig file. For more details on creating an AAD token for use with an AKS cluster see [Integrate Azure Active Directory with Azure Kubernetes Service using the Azure CLI][aad-cluster].
* The default dashboard service account, which is used if you click *Skip*.

> [!WARNING]
> Never expose the Kubernetes dashboard publicly, regardless of the authentication method used.
> 
> When setting up authentication for the Kubernetes dashboard, it is recommended that you use a token over the default dashboard service account. A token allows each user to use their own permissions. Using the default dashboard service account may allow a user to bypass their own permissions and use the service account instead.
> 
> If you do choose to use the default dashboard service account and your AKS cluster uses RBAC, a *ClusterRoleBinding* must be created before you can correctly access the dashboard. By default, the Kubernetes dashboard is deployed with minimal read access and displays RBAC access errors. A cluster administrator can choose to grant additional access to the *kubernetes-dashboard* service account, however this can be a vector for privilege escalation. You can also integrate Azure Active Directory authentication to provide a more granular level of access.
>
> To create a binding, use the [kubectl create clusterrolebinding][kubectl-create-clusterrolebinding] command as shown in the following example. **This sample binding does not apply any additional authentication components and may lead to insecure use.**
>
> ```console
> kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard
> ```
> 
> You can now access the Kubernetes dashboard in your RBAC-enabled cluster. To start the Kubernetes dashboard, use the [az aks browse][az-aks-browse] command as detailed in the previous step.
>
> If your cluster does not use RBAC, it is not recommended to create a *ClusterRoleBinding*.
> 
> For more information on using the different authentication methods, see the Kubernetes dashboard wiki on [access controls][dashboard-authentication].

After you choose a method to sign in, the Kubernetes dashboard is displayed. If you chose to use *token* or *skip*, the Kubernetes dashboard will use the permissions of the currently logged in user to access the cluster.

> [!IMPORTANT]
> If your AKS cluster uses RBAC, a *ClusterRoleBinding* must be created before you can correctly access the dashboard. By default, the Kubernetes dashboard is deployed with minimal read access and displays RBAC access errors. The Kubernetes dashboard does not currently support user-provided credentials to determine the level of access, rather it uses the roles granted to the service account. A cluster administrator can choose to grant additional access to the *kubernetes-dashboard* service account, however this can be a vector for privilege escalation. You can also integrate Azure Active Directory authentication to provide a more granular level of access.
> 
> To create a binding, use the [kubectl create clusterrolebinding][kubectl-create-clusterrolebinding] command. The following example shows how to create a sample binding, however, this sample binding does not apply any additional authentication components and may lead to insecure use. The Kubernetes dashboard is open to anyone with access to the URL. Do not expose the Kubernetes dashboard publicly.
>
> ```console
> kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard
> ```
> 
> For more information on using the different authentication methods, see the Kubernetes dashboard wiki on [access controls][dashboard-authentication].
-->

## <a name="sign-in-to-the-dashboard-kubernetes-116"></a>Inicio de sesión en el panel (kubernetes 1.16 o posterior)

> [!IMPORTANT]
> A partir de la [versión 1.10.1 del panel de Kubernetes](https://github.com/kubernetes/dashboard/releases/tag/v1.10.1) o la versión 1.16 o posterior de Kubernetes, la cuenta de servicio "kubernetes-dashboard" ya no se puede usar para recuperar recursos debido a una [corrección de seguridad en esa versión](https://github.com/kubernetes/dashboard/pull/3400). Como resultado, las solicitudes sin información de autenticación devuelven un error 401 no autorizado. Un token de portador recuperado de una cuenta de servicio puede seguir utilizándose como en este [ejemplo del panel de Kubernetes](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/#accessing-the-dashboard-ui), pero esto afecta al flujo de inicio de sesión del complemento del panel en comparación con versiones anteriores.
>
>Si sigue ejecutando una versión anterior a la 1.16, todavía puede conceder permisos a la cuenta de servicio "kubernetes-dashboard", pero no se **recomienda**:
> ```console
> kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard
> ```

La pantalla inicial presentada requiere un archivo kubeconfig o un token. Ambas opciones requieren permisos de recursos para mostrar esos recursos en el panel.

![pantalla de inicio de sesión](./media/kubernetes-dashboard/login.png)

**Uso de un archivo kubeconfig**

En el caso de los clústeres tanto habilitados como no habilitados para Azure AD, se puede pasar un archivo kubeconfig. Asegúrese de que los tokens de acceso sean válidos. Si los tokens han expirado, puede actualizarlos a través de kubectl.

1. Establezca el archivo kubeconfig de administración con `az aks get-credentials -a --resource-group <RG_NAME> --name <CLUSTER_NAME>`.
1. Seleccione `Kubeconfig` y haga clic en `Choose kubeconfig file` para abrir el selector de archivos.
1. Seleccione el archivo kubeconfig (de forma predeterminada, se encuentra en $HOME/.kube/config).
1. Haga clic en `Sign In`

**Uso de un token**

1. En el caso de los **clústeres no habilitados para Azure AD**, ejecute `kubectl config view` y copie el token asociado a la cuenta de usuario del clúster.
1. Péguelo en la opción de token en el inicio de sesión.    
1. Haga clic en `Sign In`

En el caso de los clústeres habilitados para Azure AD, recupere el token de AAD con el siguiente comando. Compruebe que ha reemplazado el grupo de recursos y el nombre del clúster en el comando.

```
## Update <RESOURCE_GROUP and <AKS_NAME> with your input.

kubectl config view -o jsonpath='{.users[?(@.name == "clusterUser_<RESOURCE GROUP>_<AKS_NAME>")].user.auth-provider.config.access-token}'
```

Una vez que se realiza correctamente, se mostrará una página similar a la siguiente.

![Página de información general del panel web de Kubernetes](./media/kubernetes-dashboard/dashboard-overview.png)

## <a name="create-an-application"></a>Crear una aplicación

Los pasos siguientes requieren que el usuario tenga permisos para los recursos respectivos. 

Para ver cómo el panel de Kubernetes puede reducir la complejidad de las tareas de administración, vamos a crear una aplicación. Puede crear una aplicación desde el panel de Kubernetes, ya sea proporcionando la entrada de texto, un archivo YAML o mediante un asistente gráfico.

Para crear una aplicación, realice los pasos siguientes:

1. Seleccione el botón **Crear** de la ventana superior derecha.
1. Para usar el asistente para gráficos, elija **Crear una aplicación**.
1. Proporcione un nombre para la implementación, por ejemplo *nginx*
1. Escriba el nombre de la imagen de contenedor que se utilizará, como *nginx:1.15.5*
1. Para exponer el puerto 80 para el tráfico web, cree un servicio de Kubernetes. En **Service** (Servicio), seleccione **External** (Externo) y escriba **80** tanto para el puerto como para el puerto de destino.
1. Cuando esté preparado, seleccione **Implementar** para crear la aplicación.

![Implementación de una aplicación en el panel web de Kubernetes](./media/kubernetes-dashboard/create-app.png)

Se tarda un minuto o dos en que una dirección IP externa pública se asigne al servicio de Kubernetes. A la izquierda, bajo **Discovery and Load Balancing** (Detección y equilibrio de carga), seleccione **Servicios**. Se muestra el servicio de la aplicación, incluidos los *puntos de conexión externos*, como se muestra en el ejemplo siguiente:

![Visualización de la lista de servicios y puntos de conexión](./media/kubernetes-dashboard/view-services.png)

Seleccione la dirección de punto de conexión para abrir una ventana del explorador web con la página predeterminada de NGINX:

![Visualización de la página predeterminada de NGINX de la aplicación implementada](./media/kubernetes-dashboard/default-nginx.png)

## <a name="view-pod-information"></a>Visualización de la información de pod

El panel de Kubernetes puede proporcionar métricas de supervisión básicas e información para solución de problemas como los registros.

Para más información acerca de los pods de aplicación, seleccione **Pods** en el menú izquierdo. Se muestra una lista de los pods disponibles. Elija su pod *nginx* para ver información, por ejemplo, del consumo de recursos:

![Visualización de la información de pod](./media/kubernetes-dashboard/view-pod-info.png)

## <a name="edit-the-application"></a>Edición de aplicación

Además de crear y visualizar las aplicaciones, el panel de Kubernetes puede utilizarse para editar y actualizar las implementaciones de aplicaciones. Para proporcionar redundancia adicional para la aplicación, vamos a aumentar el número de réplicas de NGINX.

Para editar una implementación:

1. Seleccione **Deployments** (Implementaciones) en el menú izquierdo y, a continuación, elija su implementación de *nginx*.
1. Seleccione **Edit** (Editar) en la barra de navegación superior derecha.
1. Busque el valor de `spec.replica`, en torno a la línea 20. Para aumentar el número de réplicas de la aplicación, cambie este valor de *1* a *3*.
1. Seleccione **Update** (Actualizar) cuando esté listo.

![Edición de la implementación para actualizar el número de réplicas](./media/kubernetes-dashboard/edit-deployment.png)

Se tarda unos minutos en que los nuevos pods se creen dentro de un conjunto de réplicas. En el menú izquierdo, elija **Replica Sets** (Conjuntos de réplicas) y, a continuación, elija su conjunto de réplicas de *nginx*. La lista de los pods ahora refleja el número de réplicas actualizado, como se muestra en la salida del ejemplo siguiente:

![Visualización de información sobre el conjunto de réplicas](./media/kubernetes-dashboard/view-replica-set.png)

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el panel de Kubernetes, consulte el [panel de la interfaz de usuario web de Kubernetes][kubernetes-dashboard].

<!-- LINKS - external -->
[dashboard-authentication]: https://github.com/kubernetes/dashboard/wiki/Access-control
[kubeconfig-file]: https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
[kubectl-create-clusterrolebinding]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-clusterrolebinding-em-
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubernetes-dashboard]: https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

<!-- LINKS - internal -->
[aad-cluster]: ./azure-ad-integration-cli.md
[aks-quickstart]: ./kubernetes-walkthrough.md
[aks-service-accounts]: ./concepts-identity.md#kubernetes-service-accounts
[az-account-get-access-token]: /cli/azure/account?view=azure-cli-latest#az-account-get-access-token
[az-aks-browse]: /cli/azure/aks#az-aks-browse
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[install-azure-cli]: /cli/azure/install-azure-cli
[kubernetes-portal]: ./kubernetes-portal.md
