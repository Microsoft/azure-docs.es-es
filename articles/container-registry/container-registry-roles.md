---
title: Roles y permisos de Azure
description: Use el control de acceso basado en roles (Azure RBAC) de Azure y la administración de identidades y acceso (IAM) para proporcionar la personalización avanzada de permisos a los recursos de una instancia de Azure Container Registry.
ms.topic: article
ms.date: 08/17/2020
ms.openlocfilehash: b8562d3e33cd49082d4ba4d8567d5f0c816070b0
ms.sourcegitcommit: d18a59b2efff67934650f6ad3a2e1fe9f8269f21
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88661391"
---
# <a name="azure-container-registry-roles-and-permissions"></a>Roles y permisos de Azure Container Registry

El servicio Azure Container Registry admite un conjunto de [roles de Azure integrados](../role-based-access-control/built-in-roles.md) que proporcionan distintos niveles de permisos a una instancia de Azure Container Registry. Use el [control de acceso basado en rol de Azure](../role-based-access-control/index.yml) (RBAC de Azure) para asignar permisos específicos a usuarios, entidades de servicio u otras entidades que necesiten interactuar con un registro. También puede definir [roles personalizados](#custom-roles) con permisos específicos en un registro para distintas operaciones.

| Rol/permiso       | [Acceso a Resource Manager](#access-resource-manager) | [Crear o eliminar un registro](#create-and-delete-registry) | [Insertar imagen](#push-image) | [Extraer imagen](#pull-image) | [Eliminar los datos de imagen](#delete-image-data) | [Cambiar directivas](#change-policies) |   [Firmar imágenes](#sign-images)  |
| ---------| --------- | --------- | --------- | --------- | --------- | --------- | --------- |
| Propietario | X | X | X | X | X | X |  |  
| Colaborador | X | X | X |  X | X | X |  |  
| Lector | X |  |  | X |  |  |  |
| AcrPush |  |  | X | X | |  |  |  
| AcrPull |  |  |  | X |  |  |  |  
| AcrDelete |  |  |  |  | X |  |  |
| AcrImageSigner |  |  |  |  |  |  | X |

## <a name="differentiate-users-and-services"></a>Diferenciar usuarios y servicios

Cada vez que se aplican permisos, el procedimiento recomendado es proporcionar el conjunto de permisos más limitados para que una persona o servicio puedan completar una tarea. Los siguientes conjuntos de permisos representan un conjunto de funciones para usuarios humanos y servicios desatendidos.

### <a name="cicd-solutions"></a>Soluciones de CI/CD

Al automatizar comandos de `docker build` de soluciones de CI/CD, necesita funciones de `docker push`. En estos escenarios de servicios desatendidos, le sugerimos que asigne el rol **AcrPush**. Este rol, al contrario que el rol **Colaborador** (que es más amplio), impide que la cuenta pueda realizar otras operaciones de registro o que acceda a Azure Resource Manager.

### <a name="container-host-nodes"></a>Nodos de host de contenedor

Del mismo modo, los nodos que ejecuten los contenedores han de tener asignado el rol **AcrPull**, pero no necesitan de forma obligatoria funciones de **Lector**.

### <a name="visual-studio-code-docker-extension"></a>Extensión de Docker de Visual Studio Code

Para herramientas como la [extensión de Docker](https://code.visualstudio.com/docs/azure/docker) de Visual Studio Code, se necesita acceso adicional de proveedor de recursos para mostrar la lista de registros de contenedor de Azure. En este caso, proporcione a los usuarios acceso al rol **Lector** o **Colaborador**. Estos roles permiten `docker pull`, `docker push`, `az acr list`, `az acr build` y otras funciones. 

## <a name="access-resource-manager"></a>Acceso a Resource Manager

Se necesita obtener acceso a Azure Resource Manager para Azure Portal y a la administración de registros con la [CLI de Azure](/cli/azure/). Por ejemplo, para obtener una lista de registros mediante el comando `az acr list`, necesita este conjunto de permisos. 

## <a name="create-and-delete-registry"></a>Crear y eliminar registros

Capacidad para crear y eliminar registros de contenedor de Azure.

## <a name="push-image"></a>Imagen de inserción

La capacidad para `docker push` una imagen o insertar otro [artefacto compatible](container-registry-image-formats.md), como un gráfico de Helm, en un registro. Se necesita [autenticación](container-registry-authentication.md) con el registro mediante la identidad autorizada. 

## <a name="pull-image"></a>Extraer imagen

La capacidad para `docker pull` una imagen que no esté en cuarentena o extraer otro [artefacto compatible](container-registry-image-formats.md), como un gráfico de Helm, de un registro. Se necesita [autenticación](container-registry-authentication.md) con el registro mediante la identidad autorizada.

## <a name="delete-image-data"></a>Eliminación de los datos de la imagen

La capacidad de [eliminar imágenes de contenedor](container-registry-delete.md) u otros [artefactos admitidos](container-registry-image-formats.md), como gráficos de Helm, desde un registro.

## <a name="change-policies"></a>Cambiar directivas

Capacidad para configurar directivas en un registro. En las directivas, se incluye la purga de imágenes, la activación de la cuarentena y la firma de imágenes.

## <a name="sign-images"></a>Firmar imágenes

Capacidad para firmar imágenes, que suelen estar asignadas a un proceso automatizado, lo que usaría una entidad de servicio. Este permiso suele combinarse con la inserción de imágenes para permitir [insertar una imagen](#push-image) de confianza en un registro. Para obtener más información, vea [Confianza de contenido en Azure Container Registry](container-registry-content-trust.md).

## <a name="custom-roles"></a>Roles personalizados

Como con otros recursos de Azure, puede crear [roles personalizados](../role-based-access-control/custom-roles.md) con permisos específicos para Azure Container Registry. A continuación, asigne los roles personalizados a los usuarios, entidades de servicio u otras identidades que necesiten interactuar con un registro. 

Para determinar los permisos que se van a aplicar a un rol personalizado, consulte la lista de [acciones](../role-based-access-control/resource-provider-operations.md#microsoftcontainerregistry) de Microsoft.ContainerRegistry, revise las acciones permitidas de los [roles de ACR integrados](../role-based-access-control/built-in-roles.md) o ejecute el siguiente comando:

```azurecli
az provider operation show --namespace Microsoft.ContainerRegistry
```

Para definir un rol personalizado, consulte [Pasos para crear un rol personalizado](../role-based-access-control/custom-roles.md#steps-to-create-a-custom-role).

> [!IMPORTANT]
> En un rol personalizado, Azure Container Registry no admite actualmente caracteres comodín como `Microsoft.ContainerRegistry/*` o `Microsoft.ContainerRegistry/registries/*` que conceden acceso a todas las acciones coincidentes. Especifique cualquier acción necesaria individualmente en el rol.

### <a name="example-custom-role-to-import-images"></a>Ejemplo: Rol personalizado para importar imágenes

Por ejemplo, el siguiente JSON define las acciones mínimas para un rol personalizado que permite [importar imágenes](container-registry-import-images.md) a un registro.

```json
{
   "assignableScopes": [
     "/subscriptions/<optional, but you can limit the visibility to one or more subscriptions>"
   ],
   "description": "Can import images to registry",
   "Name": "AcrImport",
   "permissions": [
     {
       "actions": [
         "Microsoft.ContainerRegistry/registries/push/write",
         "Microsoft.ContainerRegistry/registries/pull/read",
         "Microsoft.ContainerRegistry/registries/read",
         "Microsoft.ContainerRegistry/registries/importImage/action"
       ],
       "dataActions": [],
       "notActions": [],
       "notDataActions": []
     }
   ],
   "roleType": "CustomRole"
 }
```

Para crear o actualizar un rol personalizado mediante la descripción de JSON, use la [CLI de Azure](../role-based-access-control/custom-roles-cli.md), la [plantilla de Azure Resource Manager](../role-based-access-control/custom-roles-template.md), [Azure PowerShell](../role-based-access-control/custom-roles-powershell.md) u otras herramientas de Azure. Agregue o quite las asignaciones de roles de un rol personalizado de la misma manera en que administra las asignaciones de roles para los roles integrados de Azure.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre cómo asignar roles de Azure a una identidad de Azure mediante [Azure Portal](../role-based-access-control/role-assignments-portal.md), la [CLI de Azure](../role-based-access-control/role-assignments-cli.md) u otras herramientas de Azure.

* Obtenga información sobre las [opciones de autenticación](container-registry-authentication.md) de Azure Container Registry.

* Obtenga información sobre cómo habilitar [permisos de ámbito de repositorio](container-registry-repository-scoped-permissions.md) (versión preliminar) en un registro de contenedor.