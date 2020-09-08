---
title: 'Administración de una identidad administrada asignada por el usuario mediante la CLI de Azure: Azure AD'
description: Instrucciones paso a paso sobre cómo crear, enumerar y eliminar una identidad administrada asignada por el usuario mediante la CLI de Azure.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/17/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurecli
ms.openlocfilehash: 5c3af6c46dd8ad69915e8f870d739f33375dba5e
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89266415"
---
# <a name="create-list-or-delete-a-user-assigned-managed-identity-using-the-azure-cli"></a>Creación, enumeración o eliminación de una identidad administrada asignada por el usuario mediante la CLI de Azure


Las identidades administradas para los recursos de Azure proporcionan a los servicios de Azure una identidad administrada en Azure Active Directory. Puede usar esta identidad para autenticar a cualquier servicio que admita la autenticación de Azure AD, sin necesidad de tener credenciales en el código. 

En este artículo, aprenderá a crear, enumerar y eliminar una identidad administrada asignada por el usuario mediante la CLI de Azure.

## <a name="prerequisites"></a>Requisitos previos

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md). **No olvide revisar la [diferencia entre una identidad administrada asignada por el sistema y una identidad administrada asignada por el usuario](overview.md#managed-identity-types)** .
- Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.
- Para ejecutar los ejemplos de script de la CLI, tiene tres opciones:
    - Usar [Azure Cloud Shell](../../cloud-shell/overview.md) desde Azure Portal (consulte la sección siguiente).
    - Utilice Azure Cloud Shell integrado mediante el botón "Pruébelo", situado en la esquina superior derecha de cada bloque de código.
    - [Instale la versión más reciente de la CLI de Azure](/cli/azure/install-azure-cli) (2.0.13 o posterior) si prefiere usar una consola de CLI local. Inicie sesión en Azure con `az login`, usando una cuenta asociada a la suscripción de Azure en la que desearía implementar la identidad administrada asignada por el usuario.


> [!NOTE]
> Para modificar los permisos de usuario al usar una entidad de servicio de aplicación mediante la CLI, debe proporcionar los permisos adicionales de la entidad de servicio en Graph API de Azure AD como partes de la CLI, realice solicitudes GET en Graph API. De lo contrario, puede acabar recibiendo el mensaje "No tiene privilegios suficientes para completar la operación". Para ello, tendrá que entrar en el Registro de aplicación en Azure Active Directory, seleccionar la aplicación, hacer clic en Permisos de API, desplazarse hacia abajo y seleccionar Azure Active Directory Graph. Desde allí, seleccione Permisos de la aplicación y, a continuación, agregue los permisos adecuados. 



[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="create-a-user-assigned-managed-identity"></a>Crear una identidad administrada asignada por el usuario 

Para crear una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

Use el comando [az identity create](/cli/azure/identity#az-identity-create) para crear una identidad administrada asignada por el usuario. El parámetro `-g` especifica el grupo de recursos donde se creará la identidad administrada asignada por el usuario, mientras que el parámetro `-n` especifica su nombre. Reemplace los valores de los parámetros `<RESOURCE GROUP>` y `<USER ASSIGNED IDENTITY NAME>` con sus propios valores:

[!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

 ```azurecli-interactive
az identity create -g <RESOURCE GROUP> -n <USER ASSIGNED IDENTITY NAME>
```
## <a name="list-user-assigned-managed-identities"></a>Enumerar identidades administradas asignadas por el usuario

Para crear o leer una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Operador de identidades administradas ](../../role-based-access-control/built-in-roles.md#managed-identity-operator) o [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

Para enumerar las identidades administradas asignadas por el usuario, use el comando [az identity list](/cli/azure/identity#az-identity-list). Reemplace `<RESOURCE GROUP>` por su propio valor:

```azurecli-interactive
az identity list -g <RESOURCE GROUP>
```
En la respuesta JSON, a las identidades administradas asignadas por el usuario se devuelve el valor `"Microsoft.ManagedIdentity/userAssignedIdentities"` para la clave `type`.

`"type": "Microsoft.ManagedIdentity/userAssignedIdentities"`

## <a name="delete-a-user-assigned-managed-identity"></a>Eliminar una identidad administrada asignada por el usuario

Para eliminar una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

Para eliminar una identidad administrada asignada por el usuario, use el comando [az identity delete](/cli/azure/identity#az-identity-delete).  El parámetro -n especifica el nombre, y el parámetro -g especifica el grupo de recursos donde se creó la identidad administrada asignada por el usuario. Reemplace los valores de los parámetros `<USER ASSIGNED IDENTITY NAME>` y `<RESOURCE GROUP>` con sus propios valores:

 ```azurecli-interactive
az identity delete -n <USER ASSIGNED IDENTITY NAME> -g <RESOURCE GROUP>
```
> [!NOTE]
> El proceso de eliminación de una identidad administrada asignada por el usuario no quitará la referencia de ningún recurso al que se haya asignado. Quite esas máquinas virtuales y VMSS con el comando `az vm/vmss identity remove`.

## <a name="next-steps"></a>Pasos siguientes

Para obtener una lista completa de comandos de identidad de la CLI de Azure, consulte [az identity](/cli/azure/identity).

Para obtener información sobre cómo asignar una identidad administrada asignada por el usuario a una VM de Azure, consulte [Configure managed identities for Azure resources on an Azure VM using Azure CLI](qs-configure-cli-windows-vm.md#user-assigned-managed-identity) (Configuración de identidades administradas de recursos de Azure en una VM de Azure con la CLI de Azure).


