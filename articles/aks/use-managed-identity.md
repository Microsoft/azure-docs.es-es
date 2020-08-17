---
title: Uso de identidades administradas en Azure Kubernetes Service
description: Aprenda a utilizar identidades administradas en Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 07/17/2020
ms.author: thomasge
ms.openlocfilehash: 8c5c4a6e5d8b2997d80c7263ba17a705d3846ed8
ms.sourcegitcommit: 25bb515efe62bfb8a8377293b56c3163f46122bf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87987399"
---
# <a name="use-managed-identities-in-azure-kubernetes-service"></a>Uso de identidades administradas en Azure Kubernetes Service

Actualmente, un clúster de Azure Kubernetes Service (AKS) (específicamente, el proveedor de nube Kubernetes) requiere una identidad para crear recursos adicionales, como equilibradores de carga y discos administrados en Azure. Esta identidad puede ser una *identidad administrada* o una *entidad de servicio*. Si usa una [entidad de servicio](kubernetes-service-principal.md), debe indicarla, o bien AKS la creará automáticamente. Si usa la identidad administrada, AKS la creará automáticamente. Llega un momento en que los clústeres que usan entidades de servicio alcanzan un estado en el que se debe renovar la entidad de servicio para mantener el clúster en funcionamiento. La administración de entidades de servicio agrega complejidad, por lo que es más fácil usar identidades administradas. Se aplican los mismos requisitos de permisos tanto en las entidades de servicio como en las identidades administradas.

Las *identidades administradas* son básicamente un contenedor relacionado con las entidades de servicio y facilitan su administración. La rotación de credenciales para MI se produce automáticamente cada 46 días según el valor predeterminado de Azure Active Directory. AKS usa tipos de identidad administrados asignados por el sistema y asignados por el usuario. Estas identidades son inmutables actualmente. Para más información, consulte el tema sobre [identidades administradas para recursos de Azure](../active-directory/managed-identities-azure-resources/overview.md).

## <a name="before-you-begin"></a>Antes de empezar

Debe tener instalado el siguiente recurso:

- La CLI de Azure, versión 2.8.0 o posterior

## <a name="limitations"></a>Limitaciones

* Los clústeres de AKS con identidades administradas solo se pueden habilitar durante la creación del clúster.
* Los clústeres de AKS existentes no se pueden migrar a identidades administradas.
* Durante las operaciones de **actualización** del clúster, la identidad administrada no está disponible temporalmente.
* No se admite que los inquilinos trasladen o migren los clústeres habilitados para identidades administradas.

## <a name="summary-of-managed-identities"></a>Resumen de identidades administradas

AKS usa varias identidades administradas para servicios integrados y complementos.

| Identidad                       | Nombre    | Caso de uso | Permisos predeterminados | Traiga su propia identidad
|----------------------------|-----------|----------|
| Plano de control | no visible | Usada por AKS para los recursos de red administrados, incluidos los equilibradores de carga de entrada y las direcciones IP públicas administradas por AKS. | Rol de colaborador para un grupo de recursos de nodo | Versión preliminar
| Kubelet | Nombre de clúster de AKS-agentpool | Autenticación con Azure Container Registry (ACR) | Rol de lector para el grupo de recursos de nodo | No se admite actualmente.
| Complemento | AzureNPM | No se requiere ninguna identidad | N/D | No
| Complemento | Supervisión de red AzureCNI | No se requiere ninguna identidad | N/D | No
| Complemento | azurepolicy (operador de control) | No se requiere ninguna identidad | N/D | No
| Complemento | azurepolicy | No se requiere ninguna identidad | N/D | No
| Complemento | Calico | No se requiere ninguna identidad | N/D | No
| Complemento | Panel | No se requiere ninguna identidad | N/D | No
| Complemento | HTTPApplicationRouting | Administra los recursos de red necesarios | Rol de lector para grupo de recursos de nodo, rol colaborador para zona DNS | No
| Complemento | Ingresar a la puerta de enlace de aplicaciones | Administra los recursos de red necesarios| Rol de colaborador para grupo de recursos de nodo | No
| Complemento | omsagent | Se usa para enviar métricas de AKS a Azure Monitor | Supervisión del rol del publicador de métricas | No
| Complemento | Nodo virtual (ACIConnector) | Administra los recursos de red necesarios para Azure Container Instances (ACI) | Rol de colaborador para grupo de recursos de nodo | No


## <a name="create-an-aks-cluster-with-managed-identities"></a>Creación de un clúster de AKS con identidades administradas

Ahora puede crear un clúster de AKS con identidades administradas utilizando los siguientes comandos de la CLI.

En primer lugar, cree un grupo de recursos de Azure:

```azurecli-interactive
# Create an Azure resource group
az group create --name myResourceGroup --location westus2
```

Después, cree un clúster de AKS:

```azurecli-interactive
az aks create -g myResourceGroup -n myManagedCluster --enable-managed-identity
```

La creación correcta de un clúster con identidades administradas contiene esta información de perfil de la entidad de servicio:

```output
"servicePrincipalProfile": {
    "clientId": "msi"
  }
```

Use el siguiente comando para consultar el objectId de la identidad administrada del plano de control:

```azurecli-interactive
az aks show -g myResourceGroup -n myManagedCluster --query "identity"
```

El resultado debería tener este aspecto:

```output
{
  "principalId": "<object_id>",   
  "tenantId": "<tenant_id>",      
  "type": "SystemAssigned"                                 
}
```

Una vez creado el clúster, puede implementar las cargas de trabajo de la aplicación en el nuevo clúster e interactuar con él del mismo modo que con los clústeres de AKS basados en una entidad de servicio.

> [!NOTE]
> Para crear y usar su propia red virtual, una dirección IP estática o un disco de Azure conectado en el que los recursos estén fuera del grupo de recursos del nodo de trabajo, use el PrincipalID del sistema de clúster asignado a la identidad administrada para realizar una asignación de roles. Para obtener más información sobre la asignación, consulte [Delegación del acceso a otros recursos de Azure](kubernetes-service-principal.md#delegate-access-to-other-azure-resources).
>
> Las concesiones de permisos a la identidad administrada de clúster que el proveedor en la nube de Azure utiliza pueden tardar hasta 60 minutos en rellenarse.

Por último, obtenga credenciales para acceder al clúster:

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myManagedCluster
```

## <a name="bring-your-own-control-plane-mi-preview"></a>Traer su propia instancia administrada de plano de control (versión preliminar)
Una identidad de plano de control personalizado permite que se conceda acceso a la identidad existente antes de la creación del clúster. Esto permite escenarios como el uso de una red virtual personalizada o outboundType de UDR con una identidad administrada.

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

Debe tener instalados los siguientes recursos:
- La CLI de Azure, versión 2.9.0 o posterior
- La extensión aks-preview 0.4.57

Limitaciones para traer su propia instancia administrada de plano de control (versión preliminar):
* Azure Government no se admite actualmente.
* Azure China 21Vianet no se admite actualmente.

```azurecli-interactive
az extension add --name aks-preview
az extension list
```

```azurecli-interactive
az extension update --name aks-preview
az extension list
```

```azurecli-interactive
az feature register --name UserAssignedIdentityPreview --namespace Microsoft.ContainerService
```

Pueden pasar unos minutos hasta que el estado aparezca como **Registrado**. Puede comprobar el estado del registro con el comando [az feature list](/cli/azure/feature?view=azure-cli-latest#az-feature-list):

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/UserAssignedIdentityPreview')].{Name:name,State:properties.state}"
```

Cuando el estado se muestre como Registrado, actualice el registro del proveedor de recursos `Microsoft.ContainerService` mediante el comando [az provider register](/cli/azure/provider?view=azure-cli-latest#az-provider-register):

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

Si aún no tiene una identidad administrada, debería continuar y crear una, por ejemplo, mediante [az identity CLI][az-identity-create].

```azurecli-interactive
az identity create --name myIdentity --resource-group myResourceGroup
```
El resultado debería tener este aspecto:

```output
{                                                                                                                                                                                 
  "clientId": "<client-id>",
  "clientSecretUrl": "<clientSecretUrl>",
  "id": "/subscriptions/<subscriptionid>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myIdentity", 
  "location": "westus2",
  "name": "myIdentity",
  "principalId": "<principalId>",
  "resourceGroup": "myResourceGroup",                       
  "tags": {},
  "tenantId": "<tenant-id>>",
  "type": "Microsoft.ManagedIdentity/userAssignedIdentities"
}
```

Si la identidad administrada forma parte de su suscripción, puede usar el [comando az identity CLI][az-identity-list] para consultarla.  

```azurecli-interactive
az identity list --query "[].{Name:name, Id:id, Location:location}" -o table
```

Ahora puede usar el siguiente comando para crear el clúster con la identidad existente:

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myManagedCluster \
    --network-plugin azure \
    --vnet-subnet-id <subnet-id> \
    --docker-bridge-address 172.17.0.1/16 \
    --dns-service-ip 10.2.0.10 \
    --service-cidr 10.2.0.0/24 \
    --enable-managed-identity \
    --assign-identity <identity-id> \
```

La creación correcta de un clúster con sus propias identidades administradas contiene esta información de perfil de userAssignedIdentities:

```output
 "identity": {
   "principalId": null,
   "tenantId": null,
   "type": "UserAssigned",
   "userAssignedIdentities": {
     "/subscriptions/<subscriptionid>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myIdentity": {
       "clientId": "<client-id>",
       "principalId": "<principal-id>"
     }
   }
 },
```

## <a name="next-steps"></a>Pasos siguientes
* Use las [plantillas de Azure Resource Manager (ARM) ][aks-arm-template] para crear clústeres habilitados para identidades administradas.

<!-- LINKS - external -->
[aks-arm-template]: /azure/templates/microsoft.containerservice/managedclusters
[az-identity-create]: /cli/azure/identity?view=azure-cli-latest#az-identity-create
[az-identity-list]: /cli/azure/identity?view=azure-cli-latest#az-identity-list
