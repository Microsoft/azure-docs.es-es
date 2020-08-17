---
title: Creación de una entidad de servicio de incorporación habilitada para Azure Arc (versión preliminar)
services: azure-arc
ms.service: azure-arc
ms.date: 05/19/2020
ms.topic: article
author: mlearned
ms.author: mlearned
description: 'Creación de una entidad de servicio de incorporación habilitada para Azure Arc '
keywords: Kubernetes, Arc, Azure, containers
ms.openlocfilehash: 02689dba32c8cc91e4a4a4de4dee98bc990b4dd6
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87050082"
---
# <a name="create-an-azure-arc-enabled-onboarding-service-principal-preview"></a>Creación de una entidad de servicio de incorporación habilitada para Azure Arc (versión preliminar)

## <a name="overview"></a>Información general

Es posible usar entidades de servicio que tengan una asignación de roles con privilegios limitados para la incorporación de clústeres de Kubernetes a Azure ARC. Esto resulta útil en las canalizaciones de integración continua e implementación continua (CI/CD), como Azure Pipelines y las Acciones de GitHub.

Los pasos siguientes proporcionan un tutorial sobre el uso de entidades de servicio para la incorporación de clústeres de Kubernetes a Azure Arc.

## <a name="create-a-new-service-principal"></a>Creación de una nueva entidad de servicio

Cree una nueva entidad de servicio con un nombre descriptivo. Tenga en cuenta que este nombre debe ser único en el inquilino de Azure Active Directory:

```console
az ad sp create-for-RBAC --skip-assignment --name "https://azure-arc-for-k8s-onboarding"
```

**Salida:**

```console
{
  "appId": "22cc2695-54b9-49c1-9a73-2269592103d8",
  "displayName": "azure-arc-for-k8s-onboarding",
  "name": "https://azure-arc-for-k8s-onboarding",
  "password": "09d3a928-b223-4dfe-80e8-fed13baa3b3d",
  "tenant": "72f988bf-86f1-41af-91ab-2d7cd011db47"
}
```

## <a name="assign-permissions"></a>Asignación de permisos

Después de crear la nueva entidad de servicio, asigne el rol "Kubernetes Cluster - Azure Arc Onboarding" (Clúster de Kubernetes - Incorporación de Azure Arc) a la entidad de seguridad recién creada. Se trata de un rol integrado de Azure con permisos limitados, que solo permite a la entidad de seguridad registrar clústeres en Azure. La entidad de seguridad no puede actualizar, eliminar ni modificar ningún otro clúster o recurso de la suscripción.

Dado que las capacidades son limitadas, los clientes pueden volver a usar fácilmente esta entidad de seguridad para incorporar varios clústeres.

Los permisos se pueden limitar aún más si se pasa el argumento `--scope` adecuado al asignar el rol. Esto permite que los clientes restrinjan el registro del clúster. Los distintos parámetros `--scope` admiten los siguientes escenarios:

| Resource  | Argumento `scope`| Efecto |
| ------------- | ------------- | ------------- |
| Subscription | `--scope /subscriptions/0b1f6471-1bf0-4dda-aec3-111122223333` | La entidad de servicio puede registrar cualquier clúster en un grupo de recursos existente en la suscripción especificada. |
| Grupo de recursos | `--scope /subscriptions/0b1f6471-1bf0-4dda-aec3-111122223333/resourceGroups/myGroup`  | La entidad de servicio __solo__ puede registrar clústeres en el grupo de recursos `myGroup`. |

```console
az role assignment create \
    --role 34e09817-6cbe-4d01-b1a2-e0eac5743d41 \      # this is the id for the built-in role
    --assignee 22cc2695-54b9-49c1-9a73-2269592103d8 \  # use the appId from the new SP
    --scope /subscriptions/<<SUBSCRIPTION_ID>>         # apply the appropriate scope
```

**Salida:**

```console
{
  "canDelegate": null,
  "id": "/subscriptions/<<SUBSCRIPTION_ID>>/providers/Microsoft.Authorization/roleAssignments/fbd819a9-01e8-486b-9eb9-f177ba400ba6",
  "name": "fbd819a9-01e8-486b-9eb9-f177ba400ba6",
  "principalId": "ddb0ddb4-ba84-4cde-b936-affc272a4b90",
  "principalType": "ServicePrincipal",
  "roleDefinitionId": "/subscriptions/<<SUBSCRIPTION_ID>>/providers/Microsoft.Authorization/roleDefinitions/34e09817-6cbe-4d01-b1a2-e0eac5743d41",
  "scope": "/subscriptions/<<SUBSCRIPTION_ID>>",
  "type": "Microsoft.Authorization/roleAssignments"
}
```

## <a name="use-service-principal-with-the-azure-cli"></a>Uso de una entidad de servicio con la CLI de Azure

Haga referencia a la entidad de servicio recién creada:

```console
az login --service-principal -u mySpnClientId -p mySpnClientSecret --tenant myTenantID
az connectedk8s connect -n myConnectedClusterName -g myResoureGroupName
```

## <a name="next-steps"></a>Pasos siguientes

* [Uso de Azure Policy para controlar la configuración del clúster](./use-azure-policy.md)
