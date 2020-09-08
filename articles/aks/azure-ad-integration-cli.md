---
title: Integración de Azure Active Directory con Azure Kubernetes Service (heredado)
description: Obtenga información sobre cómo usar la CLI de Azure para crear un clúster de Azure Kubernetes Service (AKS) habilitado para Azure Active Directory (heredado)
services: container-service
author: TomGeske
ms.topic: article
ms.date: 07/20/2020
ms.author: thomasge
ms.openlocfilehash: ab25ec5406c75316aaa1ee8efd0192dc0207ad79
ms.sourcegitcommit: cd0a1ae644b95dbd3aac4be295eb4ef811be9aaa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88612425"
---
# <a name="integrate-azure-active-directory-with-azure-kubernetes-service-using-the-azure-cli-legacy"></a>Integración de Azure Active Directory con Azure Kubernetes Service mediante la CLI de Azure (heredado)

Es posible configurar Azure Kubernetes Service (AKS) para que utilice Azure Active Directory (AD) para la autenticación de usuarios. En esta configuración, puede iniciar sesión en un clúster de AKS mediante un token de autenticación de Azure AD. Los operadores del clúster también pueden configurar el control de acceso basado en roles (RBAC) de Kubernetes en función de la identidad de los usuarios o su pertenencia a un grupo del directorio.

En este artículo se muestra cómo crear los requisitos necesarios para Azure AD y, luego, implementar un clúster habilitado para Azure AD y crear un rol de RBAC simple en el clúster de AKS.

Para obtener el script de ejemplo completo que se usa en este artículo, consulte [Ejemplos de la CLI de Azure: integración de AKS con Azure AD][complete-script].

> [!Important]
> AKS tiene una nueva experiencia mejorada de [Azure AD administrado por AKS][managed-aad] que no requiere que administre la aplicación cliente o de servidor. Si desea migrar, siga las instrucciones [aquí][managed-aad-migrate].

## <a name="the-following-limitations-apply"></a>Se aplican las siguientes limitaciones:

- Azure AD solo puede habilitarse en el clúster habilitado para RBAC.
- El servicio de integración heredado de Azure AD solo se puede habilitar durante la creación del clúster.

## <a name="before-you-begin"></a>Antes de empezar

Es preciso que esté instalada y configurada la versión 2.0.61 de la CLI de Azure u otra versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][install-azure-cli].

Vaya a [https://shell.azure.com](https://shell.azure.com) para abrir Cloud Shell en el explorador.

Para mantener la coherencia y ayudar a ejecutar los comandos en este artículo, cree una variable para el nombre que quiera del clúster de AKS. En el ejemplo siguiente se usa el nombre *myakscluster*:

```console
aksname="myakscluster"
```

## <a name="azure-ad-authentication-overview"></a>Introducción a la autenticación de Azure AD

La autenticación de Azure AD se proporciona a los clústeres de AKS con OpenID Connect. OpenID Connect es una capa de identidad creada basándose en el protocolo OAuth 2.0. Puede encontrar más información sobre OpenID Connect en la [documentación de OpenID Connect][open-id-connect].

Dentro del clúster de Kubernetes, se usa la autenticación de token de webhook para verificar los tokens de autenticación. La autenticación de token de webhook se configura y administra como parte del clúster de AKS. Puede encontrar más información sobre la autenticación de token de Webhook en la [documentación de autenticación de webhook][kubernetes-webhook].

> [!NOTE]
> Al configurar Azure AD para la autenticación de AKS, se configuran dos aplicaciones de Azure AD. Un administrador del inquilino de Azure debe realizar esta operación.

## <a name="create-azure-ad-server-component"></a>Creación del componente de servidor de Azure AD

Para integrar con AKS, cree y use una aplicación de Azure AD que sirva de punto de conexión para las solicitudes de identidad. La primera aplicación de Azure AD que se necesita obtiene la pertenencia a un grupo de Azure AD de un usuario.

Cree el componente de aplicación de servidor mediante el comando [az ad app create][az-ad-app-create] y, a continuación, actualice las notificaciones de pertenencia a grupo mediante el comando [az ad app update][az-ad-app-update]. En el ejemplo siguiente se usa la variable *aksname* definida en la sección [Antes de comenzar](#before-you-begin) y se crea una variable.

```azurecli-interactive
# Create the Azure AD application
serverApplicationId=$(az ad app create \
    --display-name "${aksname}Server" \
    --identifier-uris "https://${aksname}Server" \
    --query appId -o tsv)

# Update the application group membership claims
az ad app update --id $serverApplicationId --set groupMembershipClaims=All
```

Ahora, cree una entidad de servicio para la aplicación de servidor mediante el comando [az ad sp create][az-ad-sp-create]. Esta entidad de servicio se usa para autenticarse en la plataforma Azure. A continuación, obtenga el secreto de la entidad de servicio con el comando [az ad sp credential reset][az-ad-sp-credential-reset] y asígnelo a la variable llamada *serverApplicationSecret* para su uso en uno de los pasos siguientes:

```azurecli-interactive
# Create a service principal for the Azure AD application
az ad sp create --id $serverApplicationId

# Get the service principal secret
serverApplicationSecret=$(az ad sp credential reset \
    --name $serverApplicationId \
    --credential-description "AKSPassword" \
    --query password -o tsv)
```

La entidad de servicio de Azure AD necesita permisos para realizar las siguientes acciones:

* Leer datos de directorio
* Iniciar sesión y leer el perfil de usuario

Asigne estos permisos con el comando [az ad app permission add][az-ad-app-permission-add]:

```azurecli-interactive
az ad app permission add \
    --id $serverApplicationId \
    --api 00000003-0000-0000-c000-000000000000 \
    --api-permissions e1fe6dd8-ba31-4d61-89e7-88639da4683d=Scope 06da0dbc-49e2-44d2-8312-53f166ab848a=Scope 7ab1d382-f21e-4acd-a863-ba3e13f7da61=Role
```

Por último, conceda los permisos asignados en el paso anterior para la aplicación de servidor con el comando [az ad app permission grant][az-ad-app-permission-grant]. Se producirá un error en este paso si la cuenta actual no es un administrador del inquilino. También tendrá que agregar permisos para que la aplicación de Azure AD solicite información que, de lo contrario, podría requerir consentimiento administrativo mediante el comando [az ad app permission admin-consent][az-ad-app-permission-admin-consent]:

```azurecli-interactive
az ad app permission grant --id $serverApplicationId --api 00000003-0000-0000-c000-000000000000
az ad app permission admin-consent --id  $serverApplicationId
```

## <a name="create-azure-ad-client-component"></a>Creación del componente de cliente de Azure AD

La segunda aplicación de Azure AD se usa cuando un usuario inicia sesión en el clúster de AKS con la CLI de Kubernetes (`kubectl`). Esta aplicación cliente toma la solicitud de autenticación del usuario y comprueba sus credenciales y permisos. Cree la aplicación de Azure AD para el componente de cliente mediante el comando [az ad app create][az-ad-app-create]:

```azurecli-interactive
clientApplicationId=$(az ad app create \
    --display-name "${aksname}Client" \
    --native-app \
    --reply-urls "https://${aksname}Client" \
    --query appId -o tsv)
```

Cree una entidad de servicio para la aplicación cliente mediante el comando [az ad sp create][az-ad-sp-create]:

```azurecli-interactive
az ad sp create --id $clientApplicationId
```

Obtenga el id. de oAuth2 para la aplicación de servidor con el fin de permitir el flujo de autenticación entre los dos componentes de las aplicaciones mediante el comando [az ad app show][az-ad-app-show]. Este id. de oAuth2 se usa en el paso siguiente.

```azurecli-interactive
oAuthPermissionId=$(az ad app show --id $serverApplicationId --query "oauth2Permissions[0].id" -o tsv)
```

Agregue los permisos para los componentes de la aplicación cliente y la aplicación de servidor con el fin de usar el flujo de comunicación de oAuth2 mediante el comando [az ad app permission add][az-ad-app-permission-add]. A continuación, conceda permisos para que la aplicación cliente se comunique con la aplicación de servidor mediante el comando [az ad app permission grant][az-ad-app-permission-grant]:

```azurecli-interactive
az ad app permission add --id $clientApplicationId --api $serverApplicationId --api-permissions ${oAuthPermissionId}=Scope
az ad app permission grant --id $clientApplicationId --api $serverApplicationId
```

## <a name="deploy-the-cluster"></a>Implementación del clúster

Con las dos aplicaciones de Azure AD creadas, cree ahora el propio clúster de AKS. En primer lugar, cree un grupo de recursos con el comando [az group create][az-group-create]. En el ejemplo siguiente se crea un grupo de recursos en la región *EastUS*:

Cree un grupo de recursos para el clúster:

```azurecli-interactive
az group create --name myResourceGroup --location EastUS
```

Obtenga el identificador de inquilino de la suscripción de Azure mediante el comando [az account show][az-account-show]. Después, cree el clúster de AKS con el comando [az aks create][az-aks-create]. El comando para crear el clúster de AKS proporciona los identificadores de servidor y aplicación cliente, el secreto de entidad de servicio de la aplicación de servidor y el identificador de inquilino:

```azurecli-interactive
tenantId=$(az account show --query tenantId -o tsv)

az aks create \
    --resource-group myResourceGroup \
    --name $aksname \
    --node-count 1 \
    --generate-ssh-keys \
    --aad-server-app-id $serverApplicationId \
    --aad-server-app-secret $serverApplicationSecret \
    --aad-client-app-id $clientApplicationId \
    --aad-tenant-id $tenantId
```

Por último, obtenga las credenciales de administrador del clúster mediante el comando [az aks get-credentials][az-aks-get-credentials]. En uno de los pasos siguientes se obtendrán las credenciales del clúster del *usuario* normal para ver el flujo de autenticación de Azure AD en acción.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name $aksname --admin
```

## <a name="create-rbac-binding"></a>Creación del enlace de RBAC

Para poder usar una cuenta de Azure Active Directory con el clúster AKS, debe crear un enlace de rol o un enlace de rol del clúster. *Roles* define los permisos para conceder, y *enlaces* los aplica a los usuarios deseados. Estas asignaciones se pueden aplicar a un espacio de nombres especificado o a todo el clúster. Para obtener más información, consulte [Uso de la autorización de RBAC][rbac-authorization].

Obtenga el nombre principal de usuario (UPN) para el usuario con sesión iniciada actualmente mediante el comando [az ad signed-in-user show][az-ad-signed-in-user-show]. Esta cuenta de usuario está habilitada para la integración de Azure AD en el paso siguiente.

```azurecli-interactive
az ad signed-in-user show --query userPrincipalName -o tsv
```

> [!IMPORTANT]
> Si el usuario al que se le concede el enlace de RBAC se encuentra en el mismo inquilino de Azure AD, asigne permisos basados en *userPrincipalName*. Si el usuario se encuentra en un inquilino distinto de Azure AD, en su lugar, consulte y use la propiedad *objectId*.

Cree un manifiesto de YAML llamado `basic-azure-ad-binding.yaml` y pegue el siguiente contenido. En la última línea, reemplace *userPrincipalName_or_objectId* por la salida del id. de objeto o de UPN del comando anterior:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: contoso-cluster-admins
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: userPrincipalName_or_objectId
```

Cree ClusterRoleBinding mediante el comando [kubectl apply][kubectl-apply] y especifique el nombre de archivo del manifiesto de YAML:

```console
kubectl apply -f basic-azure-ad-binding.yaml
```

## <a name="access-cluster-with-azure-ad"></a>Acceso al clúster con Azure AD

Ahora vamos a probar la integración de la autenticación de Azure AD para el clúster de AKS. Establezca el contexto de configuración `kubectl` para usar las credenciales de usuario normal. Este contexto devuelve todas las solicitudes de autenticación a través de Azure AD.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name $aksname --overwrite-existing
```

Ahora use el comando [kubectl get pods][kubectl-get] para ver los pods en los espacios de nombres:

```console
kubectl get pods --all-namespaces
```

Recibirá un símbolo del sistema de inicio de sesión para autenticarse con las credenciales de Azure AD mediante un explorador web. Una vez se haya autenticado correctamente, el comando `kubectl` muestra los pods del clúster de AKS, tal como se muestra en la salida del ejemplo siguiente:

```console
kubectl get pods --all-namespaces
```

```output
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code BYMK7UXVD to authenticate.

NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
kube-system   coredns-754f947b4-2v75r                 1/1     Running   0          23h
kube-system   coredns-754f947b4-tghwh                 1/1     Running   0          23h
kube-system   coredns-autoscaler-6fcdb7d64-4wkvp      1/1     Running   0          23h
kube-system   heapster-5fb7488d97-t5wzk               2/2     Running   0          23h
kube-system   kube-proxy-2nd5m                        1/1     Running   0          23h
kube-system   kube-svc-redirect-swp9r                 2/2     Running   0          23h
kube-system   kubernetes-dashboard-847bb4ddc6-trt7m   1/1     Running   0          23h
kube-system   metrics-server-7b97f9cd9-btxzz          1/1     Running   0          23h
kube-system   tunnelfront-6ff887cffb-xkfmq            1/1     Running   0          23h
```

El token de autenticación recibido para `kubectl` se almacena en caché. Solo se le solicita de nuevo iniciar sesión cuando el token ha expirado o el archivo de configuración de Kubernetes se vuelve a crear.

Si ve un mensaje de error de autorización una vez se haya registrado correctamente en un explorador web, tal como se muestra en la salida del ejemplo siguiente, compruebe las siguientes incidencias posibles:

```output
error: You must be logged in to the server (Unauthorized)
```

* Se ha definido el identificador o UPN del objeto adecuado, en función de si la cuenta de usuario está en el mismo inquilino de Azure AD o no.
* El usuario no es miembro de más de 200 grupos.
* El secreto definido en el registro de aplicación del servidor coincide con el valor configurado mediante `--aad-server-app-secret`

## <a name="next-steps"></a>Pasos siguientes

Para obtener el script completo que contiene los comandos que se muestran en este artículo, consulte el [repositorio de ejemplos de script de integración de Azure AD en AKS][complete-script].

Para usar los usuarios y grupos de Azure AD con el fin de controlar el acceso a los recursos de clúster, consulte [Controlar el acceso a recursos de clúster mediante el control de acceso basado en roles y las identidades de Azure AD en AKS][azure-ad-rbac].

Para obtener más información sobre cómo proteger los clústeres de Kubernetes, consulte [Opciones de acceso e identificación para AKS][rbac-authorization].

Para ver procedimientos recomendados sobre el control de recursos e identidades, consulte [Procedimientos recomendados para la autenticación y autorización en AKS][operator-best-practices-identity].

<!-- LINKS - external -->
[kubernetes-webhook]:https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[complete-script]: https://github.com/Azure-Samples/azure-cli-samples/tree/master/aks/azure-ad-integration/azure-ad-integration.sh

<!-- LINKS - internal -->
[az-aks-create]: /cli/azure/aks?view=azure-cli-latest#az-aks-create
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[az-group-create]: /cli/azure/group#az-group-create
[open-id-connect]: ../active-directory/develop/v2-protocols-oidc.md
[az-ad-user-show]: /cli/azure/ad/user#az-ad-user-show
[az-ad-app-create]: /cli/azure/ad/app#az-ad-app-create
[az-ad-app-update]: /cli/azure/ad/app#az-ad-app-update
[az-ad-sp-create]: /cli/azure/ad/sp#az-ad-sp-create
[az-ad-app-permission-add]: /cli/azure/ad/app/permission#az-ad-app-permission-add
[az-ad-app-permission-grant]: /cli/azure/ad/app/permission#az-ad-app-permission-grant
[az-ad-app-permission-admin-consent]: /cli/azure/ad/app/permission#az-ad-app-permission-admin-consent
[az-ad-app-show]: /cli/azure/ad/app#az-ad-app-show
[az-group-create]: /cli/azure/group#az-group-create
[az-account-show]: /cli/azure/account#az-account-show
[az-ad-signed-in-user-show]: /cli/azure/ad/signed-in-user#az-ad-signed-in-user-show
[install-azure-cli]: /cli/azure/install-azure-cli
[az-ad-sp-credential-reset]: /cli/azure/ad/sp/credential#az-ad-sp-credential-reset
[rbac-authorization]: concepts-identity.md#kubernetes-role-based-access-control-rbac
[operator-best-practices-identity]: operator-best-practices-identity.md
[azure-ad-rbac]: azure-ad-rbac.md
[managed-aad]: managed-aad.md
[managed-aad-migrate]: managed-aad.md#upgrading-to-aks-managed-azure-ad-integration
