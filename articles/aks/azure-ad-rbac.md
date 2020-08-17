---
title: Uso de Azure AD y RBAC para clústeres
titleSuffix: Azure Kubernetes Service
description: Obtenga información sobre cómo usar la pertenencia a grupos de Azure Active Directory para restringir el acceso a recursos de clúster mediante el control de acceso basado en roles (RBAC) en Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 07/21/2020
ms.openlocfilehash: 2845a091c8a89f22e8892141dd2dad26d6049447
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88006849"
---
# <a name="control-access-to-cluster-resources-using-role-based-access-control-and-azure-active-directory-identities-in-azure-kubernetes-service"></a>Administración del acceso a recursos de clúster mediante el control de acceso basado en roles y las identidades de Azure Active Directory en Azure Kubernetes Service

Es posible configurar Azure Kubernetes Service (AKS) para que utilice Azure Active Directory (AD) para la autenticación de usuarios. En esta configuración, inicie sesión en un clúster de AKS mediante un token de autenticación de Azure AD. También puede configurar el control de acceso basado en roles (RBAC) de Kubernetes para limitar el acceso a los recursos de clúster en función de la identidad de los usuarios o su pertenencia a un grupo.

Este artículo muestra cómo usar la pertenencia a grupos de Azure AD para controlar el acceso a los espacios de nombres y los recursos de clúster con Kubernetes RBAC en un clúster de AKS. Los usuarios y grupos de ejemplo se crean en Azure AD y, a continuación, se crean roles y enlaces de roles en el clúster de AKS para conceder los permisos adecuados para crear y ver los recursos.

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se supone que ya tiene un clúster de AKS habilitado con integración de Azure AD. Si necesita un clúster de AKS, consulte [Integración de Azure Active Directory con AKS][azure-ad-aks-cli].

Es preciso que esté instalada y configurada la versión 2.0.61 de la CLI de Azure u otra versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][install-azure-cli].

## <a name="create-demo-groups-in-azure-ad"></a>Creación de grupos de demostración en Azure AD

En este artículo, vamos a crear dos roles de usuario que se pueden usar para mostrar cómo Kubernetes RBAC y Azure AD controlan el acceso a recursos de clúster. Se usan los siguientes dos roles de ejemplo:

* **Desarrollador de aplicaciones**
    * Un usuario denominado *aksdev* que forma parte del grupo *appdev*.
* **Ingeniero encargado de garantizar la fiabilidad del sitio (SRE, por sus siglas en inglés)**
    * Un usuario denominado *akssre* que forma parte del grupo*opssre*.

En entornos de producción, puede usar usuarios y grupos existentes dentro de un inquilino de Azure AD.

En primer lugar, obtenga el identificador de recurso de clúster de AKS mediante el comando [az aks show][az-aks-show]. Asigne el Id. de recurso a una variable denominada *AKS_ID* de manera que pueda hacer referencia a los comandos adicionales.

```azurecli-interactive
AKS_ID=$(az aks show \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --query id -o tsv)
```

Cree el primer grupo de ejemplo en Azure AD para los desarrolladores de aplicaciones mediante el comando [az ad group create][az-ad-group-create]. En el ejemplo siguiente se crea un grupo de recursos denominado *appdev*:

```azurecli-interactive
APPDEV_ID=$(az ad group create --display-name appdev --mail-nickname appdev --query objectId -o tsv)
```

Ahora, cree una asignación de roles de Azure para el grupo *appdev* con el comando [az role assignment create][az-role-assignment-create]. Esta asignación permite que cualquier miembro del grupo pueda usar `kubectl` para interactuar con un clúster de AKS a través de la concesión del *rol de usuario de clúster de Azure Kubernetes Service*.

```azurecli-interactive
az role assignment create \
  --assignee $APPDEV_ID \
  --role "Azure Kubernetes Service Cluster User Role" \
  --scope $AKS_ID
```

> [!TIP]
> Si recibe un error como `Principal 35bfec9328bd4d8d9b54dea6dac57b82 does not exist in the directory a5443dcd-cd0e-494d-a387-3039b419f0d5.`, espere unos segundos para que el id. de objeto del grupo de Azure AD se propague través del directorio. A continuación, pruebe el comando `az role assignment create` de nuevo.

Cree un segundo grupo de ejemplo, en este caso para SRE, denominado *opssre*:

```azurecli-interactive
OPSSRE_ID=$(az ad group create --display-name opssre --mail-nickname opssre --query objectId -o tsv)
```

De nuevo, cree una asignación de rol de Azure para conceder a los miembros del grupo el *rol de usuario de clúster de Azure Kubernetes Service*:

```azurecli-interactive
az role assignment create \
  --assignee $OPSSRE_ID \
  --role "Azure Kubernetes Service Cluster User Role" \
  --scope $AKS_ID
```

## <a name="create-demo-users-in-azure-ad"></a>Creación de usuarios de demostración en Azure AD

Con dos grupos de ejemplo creados en Azure AD para nuestros desarrolladores de aplicaciones y SRE, ahora permite crear dos usuarios de ejemplo. Para probar la integración de RBAC al final del artículo, inicie sesión en el clúster de AKS con estas cuentas.

Cree la primera cuenta de usuario en Azure AD mediante el comando [az ad user create][az-ad-user-create].

En el ejemplo siguiente se crea un usuario con el nombre para mostrar *AKS Dev* y el nombre principal de usuario (UPN) de `aksdev@contoso.com`. Actualice el UPN para incluir un dominio comprobado para su inquilino de Azure AD (reemplace *contoso.com* por su propio dominio) y proporcione su propia credencial `--password` segura:

```azurecli-interactive
AKSDEV_ID=$(az ad user create \
  --display-name "AKS Dev" \
  --user-principal-name aksdev@contoso.com \
  --password P@ssw0rd1 \
  --query objectId -o tsv)
```

Ahora agregue el usuario al grupo *appdev* creado en la sección anterior mediante el comando [az ad group member add][az-ad-group-member-add]:

```azurecli-interactive
az ad group member add --group appdev --member-id $AKSDEV_ID
```

Cree una nueva cuenta de usuario. En el ejemplo siguiente se crea un usuario con el nombre para mostrar *AKS SRE* y el nombre principal de usuario (UPN) de `akssre@contoso.com`. Actualice de nuevo el UPN para incluir un dominio comprobado para su inquilino de Azure AD (reemplace *contoso.com* por su propio dominio) y proporcione su propia credencial `--password` segura:

```azurecli-interactive
# Create a user for the SRE role
AKSSRE_ID=$(az ad user create \
  --display-name "AKS SRE" \
  --user-principal-name akssre@contoso.com \
  --password P@ssw0rd1 \
  --query objectId -o tsv)

# Add the user to the opssre Azure AD group
az ad group member add --group opssre --member-id $AKSSRE_ID
```

## <a name="create-the-aks-cluster-resources-for-app-devs"></a>Creación de los recursos de clúster de AKS para desarrolladores de aplicaciones

Se han creado los usuarios y grupos de Azure AD. Además, se crearon asignaciones de roles de Azure para que los miembros del grupo se conecten a un clúster de AKS como un usuario normal. Ahora, vamos a configurar el clúster de AKS para permitir que estos grupos diferentes puedan obtener acceso a recursos específicos.

Primero, obtenga las credenciales de administrador del clúster mediante el comando [az aks get-credentials][az-aks-get-credentials]. En una de las siguientes secciones, obtendrá las credenciales del clúster del *usuario* normal para ver el flujo de autenticación de Azure AD en acción.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster --admin
```

Cree un espacio de nombres en el clúster de AKS mediante el comando [kubectl create namespace][kubectl-create]. En el siguiente ejemplo se crea un nuevo espacio de nombres, *dev*:

```console
kubectl create namespace dev
```

En Kubernetes, los *roles* definen los permisos que conceder los *enlaces de roles* los aplican a los usuarios o grupos deseados. Estas asignaciones se pueden aplicar a un espacio de nombres especificado o a todo el clúster. Para obtener más información, consulte [Uso de la autorización de RBAC][rbac-authorization].

En primer lugar, cree un rol para el espacio de nombres *dev*. Este rol concede permisos completos al espacio de nombres. En entornos de producción, puede especificar permisos más granulares para diferentes usuarios o grupos.

Cree un archivo denominado `role-dev-namespace.yaml` y pegue el siguiente manifiesto de YAML:

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-user-full-access
  namespace: dev
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["batch"]
  resources:
  - jobs
  - cronjobs
  verbs: ["*"]
```

Cree el rol mediante el comando [kubectl apply][kubectl-apply] y especifique el nombre de archivo del manifiesto de YAML:

```console
kubectl apply -f role-dev-namespace.yaml
```

A continuación, obtenga el Id. de recurso para el grupo *appdev* mediante el comando [az ad group show][az-ad-group-show]. Este grupo se establece como el sujeto de un enlace de rol en el paso siguiente.

```azurecli-interactive
az ad group show --group appdev --query objectId -o tsv
```

Ahora, cree un enlace de rol para que el grupo *appdev* use el rol creado anteriormente para el acceso de espacio de nombres. Cree un archivo denominado `rolebinding-dev-namespace.yaml` y pegue el siguiente manifiesto de YAML. En la última línea, reemplace *groupObjectId* por la salida del id. de objeto de grupo del comando anterior:

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-user-access
  namespace: dev
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: dev-user-full-access
subjects:
- kind: Group
  namespace: dev
  name: groupObjectId
```

Cree el enlace de roles mediante el comando [kubectl apply][kubectl-apply] y especifique el nombre de archivo del manifiesto de YAML:

```console
kubectl apply -f rolebinding-dev-namespace.yaml
```

## <a name="create-the-aks-cluster-resources-for-sres"></a>Creación de los recursos de clúster de AKS para SRE

Ahora, repita los pasos anteriores para crear un espacio de nombres, el rol y el enlace de rol para los SRE.

En primer lugar, cree un espacio de nombres para *sre* con el comando [kubectl create namespace][kubectl-create]:

```console
kubectl create namespace sre
```

Cree un archivo denominado `role-sre-namespace.yaml` y pegue el siguiente manifiesto de YAML:

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: sre-user-full-access
  namespace: sre
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["batch"]
  resources:
  - jobs
  - cronjobs
  verbs: ["*"]
```

Cree el rol mediante el comando [kubectl apply][kubectl-apply] y especifique el nombre de archivo del manifiesto de YAML:

```console
kubectl apply -f role-sre-namespace.yaml
```

Obtenga el identificador de recurso para el grupo *opssre* mediante el comando [az ad group show][az-ad-group-show]:

```azurecli-interactive
az ad group show --group opssre --query objectId -o tsv
```

Cree un enlace de rol para que el grupo *opssre* use el rol creado anteriormente para el acceso de espacio de nombres. Cree un archivo denominado `rolebinding-sre-namespace.yaml` y pegue el siguiente manifiesto de YAML. En la última línea, reemplace *groupObjectId* por la salida del id. de objeto de grupo del comando anterior:

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: sre-user-access
  namespace: sre
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: sre-user-full-access
subjects:
- kind: Group
  namespace: sre
  name: groupObjectId
```

Cree el enlace de roles mediante el comando [kubectl apply][kubectl-apply] y especifique el nombre de archivo del manifiesto de YAML:

```console
kubectl apply -f rolebinding-sre-namespace.yaml
```

## <a name="interact-with-cluster-resources-using-azure-ad-identities"></a>Interactuación con recursos de clúster con identidades de Azure AD

Ahora, vamos a probar el trabajo de los permisos esperados cuando se crean y administran recursos en un clúster de AKS. En estos ejemplos, programe y vea los pods en el espacio de nombres asignados del usuario. A continuación, intente programar y ver los pods fuera del espacio de nombres asignado.

En primer lugar, restablezca el contexto *kubeconfig* mediante el comando [az aks get-credentials][az-aks-get-credentials]. En una sección anterior, establezca el contexto con las credenciales de administrador de clúster. El usuario administrador omite las solicitudes de inicio de sesión de Azure AD. Sin el parámetro `--admin`, se aplica el contexto de usuario que requiere que todas las solicitudes se autentiquen mediante Azure AD.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster --overwrite-existing
```

Programación de un pod NGINX básico mediante el comando [kubectl run][kubectl-run] en el espacio de nombres *dev*:

```console
kubectl run nginx-dev --image=nginx --namespace dev
```

Cuando se le solicite iniciar sesión, escriba las credenciales de su cuenta `appdev@contoso.com` personal creada al principio del artículo. Una vez que se haya iniciado sesión correctamente, el token de la cuenta se almacena en caché para futuros comandos `kubectl`. NGINX se programa correctamente, como se muestra en la siguiente salida de ejemplo:

```console
$ kubectl run nginx-dev --image=nginx --namespace dev

To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code B24ZD6FP8 to authenticate.

pod/nginx-dev created
```

Ahora use el comando [kubectl get pods][kubectl-get] para ver los pods en el espacio de nombres *dev*.

```console
kubectl get pods --namespace dev
```

Como se muestra en la salida del ejemplo siguiente, el pod NGINX se está *ejecutando* correctamente:

```console
$ kubectl get pods --namespace dev

NAME        READY   STATUS    RESTARTS   AGE
nginx-dev   1/1     Running   0          4m
```

### <a name="create-and-view-cluster-resources-outside-of-the-assigned-namespace"></a>Creación y visualización de los recursos de clúster fuera del espacio de nombres asignado

Ahora pruebe a ver los pods fuera del espacio de nombres *dev*. Use el comando [kubectl get pods][kubectl-get] de nuevo. Esta vez debería ver `--all-namespaces` así:

```console
kubectl get pods --all-namespaces
```

La pertenencia a grupos del usuario no tiene un rol de Kubernetes que permita esta acción, como se muestra en la salida de ejemplo siguiente:

```console
$ kubectl get pods --all-namespaces

Error from server (Forbidden): pods is forbidden: User "aksdev@contoso.com" cannot list resource "pods" in API group "" at the cluster scope
```

De la misma manera, intente programar un pod en un espacio de nombres diferente, como el espacio de nombres *sre*. La pertenencia a grupos del usuario no es compatible con un rol y un enlace de rol de Kubernetes para conceder estos permisos, como se muestra en la salida de ejemplo siguiente:

```console
$ kubectl run nginx-dev --image=nginx --namespace sre

Error from server (Forbidden): pods is forbidden: User "aksdev@contoso.com" cannot create resource "pods" in API group "" in the namespace "sre"
```

### <a name="test-the-sre-access-to-the-aks-cluster-resources"></a>Prueba del acceso de SRE a los recursos de clúster de AKS

Para confirmar que la pertenencia a grupos de Azure AD y Kubernetes RBAC funcionan correctamente entre distintos usuarios y grupos, pruebe los comandos anteriores cuando inicie sesión como usuario *opssre*.

Restablezca el contexto *kubeconfig* mediante el comando [az aks get-credentials][az-aks-get-credentials] que borra el token de autenticación previamente almacenado en caché anteriormente para el usuario *aksdev* :

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster --overwrite-existing
```

Intente programar y ver los pods en el espacio de nombres *sre* asignado. Cuando se le solicite, inicie sesión con sus credenciales personales de `opssre@contoso.com` creadas al principio del artículo:

```console
kubectl run nginx-sre --image=nginx --namespace sre
kubectl get pods --namespace sre
```

Como se muestra en la siguiente salida de ejemplo, puede crear y ver los pods correctamente:

```console
$ kubectl run nginx-sre --image=nginx --namespace sre

To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code BM4RHP3FD to authenticate.

pod/nginx-sre created

$ kubectl get pods --namespace sre

NAME        READY   STATUS    RESTARTS   AGE
nginx-sre   1/1     Running   0
```

Ahora, pruebe ver o programar los pods fuera del espacio de nombres de SRE asignado:

```console
kubectl get pods --all-namespaces
kubectl run nginx-sre --image=nginx --namespace dev
```

No se pueden ejecutar estos comandos `kubectl`, como se muestra en la salida de ejemplo siguiente. La pertenencia a grupos del usuario y el rol y los enlaces de roles de Kubernetes no conceden permisos para crear o administrar recursos en otros espacios de nombres:

```console
$ kubectl get pods --all-namespaces
Error from server (Forbidden): pods is forbidden: User "akssre@contoso.com" cannot list pods at the cluster scope

$ kubectl run nginx-sre --image=nginx --namespace dev
Error from server (Forbidden): pods is forbidden: User "akssre@contoso.com" cannot create pods in the namespace "dev"
```

## <a name="clean-up-resources"></a>Limpieza de recursos

En este artículo, ha creado los recursos en el clúster de AKS y los usuarios y grupos en Azure AD. Para borrar todos estos recursos, ejecute el siguiente comando:

```azurecli-interactive
# Get the admin kubeconfig context to delete the necessary cluster resources
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster --admin

# Delete the dev and sre namespaces. This also deletes the pods, Roles, and RoleBindings
kubectl delete namespace dev
kubectl delete namespace sre

# Delete the Azure AD user accounts for aksdev and akssre
az ad user delete --upn-or-object-id $AKSDEV_ID
az ad user delete --upn-or-object-id $AKSSRE_ID

# Delete the Azure AD groups for appdev and opssre. This also deletes the Azure role assignments.
az ad group delete --group appdev
az ad group delete --group opssre
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo proteger los clústeres de Kubernetes, consulte [Opciones de acceso e identificación para AKS][rbac-authorization].

Para ver procedimientos recomendados sobre el control de recursos e identidades, consulte [Procedimientos recomendados para la autenticación y autorización en AKS][operator-best-practices-identity].

<!-- LINKS - external -->
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-run]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run

<!-- LINKS - internal -->
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[install-azure-cli]: /cli/azure/install-azure-cli
[azure-ad-aks-cli]: azure-ad-integration-cli.md
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-ad-group-create]: /cli/azure/ad/group#az-ad-group-create
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[az-ad-user-create]: /cli/azure/ad/user#az-ad-user-create
[az-ad-group-member-add]: /cli/azure/ad/group/member#az-ad-group-member-add
[az-ad-group-show]: /cli/azure/ad/group#az-ad-group-show
[rbac-authorization]: concepts-identity.md#kubernetes-role-based-access-control-rbac
[operator-best-practices-identity]: operator-best-practices-identity.md
