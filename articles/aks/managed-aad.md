---
title: Uso de Azure AD en Azure Kubernetes Service
description: Aprenda a usar Azure AD en Azure Kubernetes Service (AKS).
services: container-service
manager: gwallace
ms.topic: article
ms.date: 07/20/2020
ms.author: thomasge
ms.openlocfilehash: 06a97126df449b77bf3fcc48bd23231512c9dff2
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87056654"
---
# <a name="aks-managed-azure-active-directory-integration"></a>Integración de Azure Active Directory administrado por AKS

La integración de Azure AD administrado por AKS está diseñada para simplificar la experiencia de integración de Azure AD, donde los usuarios debían crear una aplicación cliente y otra de servidor, y donde el inquilino de Azure AD debía conceder permisos de lectura para Active Directory. En la nueva versión, el proveedor de recursos de AKS administra las aplicaciones de cliente y servidor.

## <a name="azure-ad-authentication-overview"></a>Introducción a la autenticación de Azure AD

Los administradores del clúster pueden configurar el control de acceso basado en rol (RBAC) de Kubernetes en función de la identidad de los usuarios o su pertenencia a un grupo del directorio. La autenticación de Azure AD se proporciona a los clústeres de AKS con OpenID Connect. OpenID Connect es una capa de identidad creada basándose en el protocolo OAuth 2.0. Puede encontrar más información sobre OpenID Connect en la [documentación de OpenID Connect][open-id-connect].

Obtenga más información sobre el flujo de integración de AAD en la [documentación de los conceptos de la integración de Azure Active Directory](concepts-identity.md#azure-active-directory-integration).

## <a name="region-availability"></a>Disponibilidad en regiones

La integración de Azure Active Directory administrado por AKS está disponible en las regiones públicas donde [se admite AKS](https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service).

* Azure Government no se admite actualmente.
* Azure China 21Vianet no se admite actualmente.

## <a name="limitations"></a>Limitaciones 

* No se puede deshabilitar la integración de Azure AD administrados por AKS.
* Los clústeres no habilitados para RBAC no se admiten en la integración de AAD administrado por AKS.
* No se admite el cambio del inquilino de Azure AD asociado a la integración de AAD administrado por AKS.

> [!IMPORTANT]
> Las características en versión preliminar de AKS están disponibles como opción de participación y autoservicio. Las versiones preliminares se proporcionan "tal cual" y "como están disponibles", y están excluidas de los Acuerdos de Nivel de Servicio y la garantía limitada. Las versiones preliminares de AKS reciben cobertura parcial del soporte al cliente en la medida de lo posible. Por lo tanto, estas características no están diseñadas para usarse en producción. Para más información, consulte los siguientes artículos de soporte: 
> - [Directivas de soporte técnico para AKS](support-policies.md) 
> - [Preguntas más frecuentes de soporte técnico de Azure](faq.md)

## <a name="prerequisites"></a>Requisitos previos

* La CLI de Azure, versión 2.9.0 o posterior
* Kubectl con la versión [1.18](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.18.md#v1180) como mínimo

> [!Important]
> Debe usar Kubectl con la versión 1.18, como mínimo.

Para instalar kubectl, use los comandos siguientes:

```azurecli-interactive
sudo az aks install-cli
kubectl version --client
```

Siga [estas instrucciones](https://kubernetes.io/docs/tasks/tools/install-kubectl/) para otros sistemas operativos.

```azurecli-interactive 
az feature register --name AAD-V2 --namespace Microsoft.ContainerService    
``` 

Pueden pasar unos minutos hasta que el estado aparezca como **Registrado**. Puede comprobar el estado del registro con el comando [az feature list](/cli/azure/feature?view=azure-cli-latest#az-feature-list): 

```azurecli-interactive 
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/AAD-V2')].{Name:name,State:properties.state}"    
``` 

Cuando el estado se muestre como Registrado, actualice el registro del proveedor de recursos `Microsoft.ContainerService` mediante el comando [az provider register](/cli/azure/provider?view=azure-cli-latest#az-provider-register):    

```azurecli-interactive 
az provider register --namespace Microsoft.ContainerService 
``` 


## <a name="before-you-begin"></a>Antes de empezar

Para el clúster, necesita un grupo de Azure AD. Este grupo es necesario como grupo de administradores para que el clúster conceda permisos de administrador de clústeres. Puede usar un grupo de Azure AD existente o crear uno nuevo. Anote el identificador de objeto del grupo de Azure AD.

```azurecli-interactive
# List existing groups in the directory
az ad group list --filter "displayname eq '<group-name>'" -o table
```

Para crear un grupo de Azure AD para los administradores de clúster, use el comando siguiente:

```azurecli-interactive
# Create an Azure AD group
az ad group create --display-name myAKSAdminGroup --mail-nickname myAKSAdminGroup
```

## <a name="create-an-aks-cluster-with-azure-ad-enabled"></a>Creación de un clúster de AKS con Azure AD habilitado

Cree un clúster de AKS con los comandos de la CLI siguientes.

Crear un grupo de recursos de Azure:

```azurecli-interactive
# Create an Azure resource group
az group create --name myResourceGroup --location centralus
```

Cree un clúster de AKS y habilite el acceso de administración para el grupo de Azure AD

```azurecli-interactive
# Create an AKS-managed Azure AD cluster
az aks create -g myResourceGroup -n myManagedCluster --enable-aad --aad-admin-group-object-ids <id> [--aad-tenant-id <id>]
```

Un clúster de Azure AD administrado por AKS creado correctamente contiene la sección siguiente en el cuerpo de la respuesta
```output
"AADProfile": {
    "adminGroupObjectIds": [
      "5d24****-****-****-****-****afa27aed"
    ],
    "clientAppId": null,
    "managed": true,
    "serverAppId": null,
    "serverAppSecret": null,
    "tenantId": "72f9****-****-****-****-****d011db47"
  }
```

Una vez creado el clúster, puede empezar a acceder a él.

## <a name="access-an-azure-ad-enabled-cluster"></a>Acceso a un clúster habilitado para Azure AD

Necesitará el rol integrado del [usuario del clúster de Azure Kubernetes Service](../role-based-access-control/built-in-roles.md#azure-kubernetes-service-cluster-user-role) para realizar los pasos siguientes.

Obtenga las credenciales de usuario para acceder al clúster:
 
```azurecli-interactive
 az aks get-credentials --resource-group myResourceGroup --name myManagedCluster
```
Siga las instrucciones para iniciar sesión.

Use el comando "kubectl get nodes" para ver los nodos del clúster:

```azurecli-interactive
kubectl get nodes

NAME                       STATUS   ROLES   AGE    VERSION
aks-nodepool1-15306047-0   Ready    agent   102m   v1.15.10
aks-nodepool1-15306047-1   Ready    agent   102m   v1.15.10
aks-nodepool1-15306047-2   Ready    agent   102m   v1.15.10
```
Configure el [control de acceso basado en roles (RBAC)](./azure-ad-rbac.md) para configurar grupos de seguridad adicionales para los clústeres.

## <a name="troubleshooting-access-issues-with-azure-ad"></a>Solución de problemas de acceso con Azure AD

> [!Important]
> En los pasos que se describen a continuación se omite la autenticación de grupo de Azure AD normal. Úselos únicamente en caso de emergencia.

Si está bloqueado permanentemente al no tener acceso a un grupo de Azure AD válido con acceso al clúster, de todos modos puede obtener las credenciales de administrador para acceder directamente al clúster.

Para llevar a cabo estos pasos, necesitará tener acceso al rol integrado [Administrador de clúster de Azure Kubernetes Service](../role-based-access-control/built-in-roles.md#azure-kubernetes-service-cluster-admin-role).

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myManagedCluster --admin
```

## <a name="upgrading-to-aks-managed-azure-ad-integration"></a>Actualización a la integración de Azure AD administrado por AKS

Si el clúster usa la integración de Azure AD heredada, puede actualizar a la integración de Azure AD administrado por AKS.

```azurecli-interactive
az aks update -g myResourceGroup -n myManagedCluster --enable-aad --aad-admin-group-object-ids <id> [--aad-tenant-id <id>]
```

Una migración correcta de un clúster de Azure AD administrado por AKS contiene la sección siguiente en el cuerpo de la respuesta:

```output
"AADProfile": {
    "adminGroupObjectIds": [
      "5d24****-****-****-****-****afa27aed"
    ],
    "clientAppId": null,
    "managed": true,
    "serverAppId": null,
    "serverAppSecret": null,
    "tenantId": "72f9****-****-****-****-****d011db47"
  }
```

Si quiere acceder al clúster, siga los pasos que se indican [aquí][access-cluster].

## <a name="non-interactive-sign-in-with-kubelogin"></a>Inicio de sesión no interactivo con kubelogin

Hay algunos escenarios no interactivos, como las canalizaciones de integración continua, que no están disponibles actualmente con kubectl. Puede usar [`kubelogin`](https://github.com/Azure/kubelogin) para acceder al clúster con el inicio de sesión no interactivo de la entidad de servicio.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre la [integración de Azure RBAC para la autorización de Kubernetes][azure-rbac-integration].
* Aprenda sobre la [integración de Azure AD con RBAC de Kubernetes][azure-ad-rbac].
* Use [kubelogin](https://github.com/Azure/kubelogin) para obtener acceso a las características de autenticación de Azure que no están disponibles en kubectl.
* Obtenga más información sobre los [conceptos de identidad de Kubernetes y AKS][aks-concepts-identity].
* Use las [plantillas de Azure Resource Manager (ARM)][aks-arm-template] para crear clústeres habilitados para Azure AD administrado por AKS.

<!-- LINKS - external -->
[kubernetes-webhook]:https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[aks-arm-template]: /azure/templates/microsoft.containerservice/managedclusters

<!-- LINKS - Internal -->
[azure-rbac-integration]: manage-azure-rbac.md
[aks-concepts-identity]: concepts-identity.md
[azure-ad-rbac]: azure-ad-rbac.md
[az-aks-create]: /cli/azure/aks?view=azure-cli-latest#az-aks-create
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[az-group-create]: /cli/azure/group#az-group-create
[open-id-connect]:../active-directory/develop/v2-protocols-oidc.md
[az-ad-user-show]: /cli/azure/ad/user#az-ad-user-show
[rbac-authorization]: concepts-identity.md#role-based-access-controls-rbac
[operator-best-practices-identity]: operator-best-practices-identity.md
[azure-ad-rbac]: azure-ad-rbac.md
[azure-ad-cli]: azure-ad-integration-cli.md
[access-cluster]: #access-an-azure-ad-enabled-cluster
[aad-migrate]: #upgrading-to-aks-managed-azure-ad-integration
