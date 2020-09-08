---
title: 'Configuración de identidades administradas en VM de Azure mediante la CLI de Azure: Azure AD'
description: Instrucciones paso a paso para configurar identidades administradas asignadas por el sistema y por el usuario en una VM de Azure mediante la CLI de Azure.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/26/2019
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: cdaff3dd8c1397ea2a0f70a5b84c0e42e9692412
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89255450"
---
# <a name="configure-managed-identities-for-azure-resources-on-an-azure-vm-using-azure-cli"></a>Configurar identidades administradas para recursos de Azure en una VM de Azure mediante la CLI de Azure

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Las identidades administradas para los recursos de Azure proporcionan a los servicios de Azure una identidad administrada automáticamente en Azure Active Directory. Puede usar esta identidad para autenticar a cualquier servicio que admita la autenticación de Azure AD, sin necesidad de tener credenciales en el código. 

En este artículo, con la CLI de Azure, aprenderá a usar las siguientes identidades administradas para operaciones de recursos de Azure en una VM de Azure:

- Habilitar y deshabilitar la identidad administrada asignada por el sistema en una VM de Azure
- Agregar y quitar una identidad administrada asignada por el usuario en una VM de Azure

## <a name="prerequisites"></a>Requisitos previos

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md). **No olvide revisar la [diferencia entre una identidad administrada asignada por el sistema y una identidad administrada asignada por el usuario](overview.md#managed-identity-types)** .
- Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.
- Para ejecutar los ejemplos de script de la CLI, tiene tres opciones:
    - Usar [Azure Cloud Shell](../../cloud-shell/overview.md) desde Azure Portal (consulte la sección siguiente).
    - Usar Azure Cloud Shell integrado a través del botón "Pruébelo", situado en la esquina superior derecha de cada bloque de código.
    - [Instale la versión más reciente de la CLI de Azure](/cli/azure/install-azure-cli) si prefiere usar una consola de la CLI local. 
      
      > [!NOTE]
      > Los comandos se han actualizado para reflejar la versión más reciente de la [CLI de Azure](/cli/azure/install-azure-cli).     

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="system-assigned-managed-identity"></a>Identidad administrada asignada por el sistema

En esta sección, aprenderá a habilitar y deshabilitar la identidad administrada asignada por el sistema en una VM de Azure mediante la CLI de Azure.

### <a name="enable-system-assigned-managed-identity-during-creation-of-an-azure-vm"></a>Habilitación de una identidad administrada asignada por el sistema durante la creación de una VM de Azure

Para crear una máquina virtual de Azure que tenga habilitada la identidad administrada asignada por el sistema, la cuenta debe tener la asignación de roles [Colaborador de la máquina Virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  No se requiere ninguna otra asignación de roles de directorio de Azure AD.

1. Si usa la CLI de Azure en una consola local, lo primero que debe hacer es iniciar sesión en Azure mediante el [inicio de sesión de az](/cli/azure/reference-index#az-login). Use una cuenta asociada a la suscripción de Azure en la que desearía implementar la máquina virtual:

   ```azurecli-interactive
   az login
   ```

2. Cree un [grupo de recursos](../../azure-resource-manager/management/overview.md#terminology) para contener e implementar la máquina virtual y sus recursos relacionados, con [az group create](/cli/azure/group/#az-group-create). Puede omitir este paso si ya tiene un grupo de recursos que le gustaría usar en su lugar:

   ```azurecli-interactive 
   az group create --name myResourceGroup --location westus
   ```

3. Cree una máquina virtual mediante [az vm create](/cli/azure/vm/#az-vm-create). En el ejemplo siguiente, se crea una VM denominada *myVM* con una identidad administrada asignada por el sistema, como ha solicitado el parámetro `--assign-identity`. Los parámetros `--admin-username` y `--admin-password` especifican el nombre de usuario administrativo y la contraseña de la cuenta para el inicio de sesión en la máquina virtual. Actualice estos valores según convenga para su entorno: 

   ```azurecli-interactive 
   az vm create --resource-group myResourceGroup --name myVM --image win2016datacenter --generate-ssh-keys --assign-identity --admin-username azureuser --admin-password myPassword12
   ```

### <a name="enable-system-assigned-managed-identity-on-an-existing-azure-vm"></a>Habilitación de una identidad administrada asignada por el sistema en una VM de Azure existente

Para habilitar una identidad administrada asignada por el sistema en una máquina virtual, la cuenta necesita la asignación de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  No se requiere ninguna otra asignación de roles de directorio de Azure AD.

1. Si usa la CLI de Azure en una consola local, lo primero que debe hacer es iniciar sesión en Azure mediante el [inicio de sesión de az](/cli/azure/reference-index#az-login). Use una cuenta asociada a la suscripción de Azure que contenga la máquina virtual.

   ```azurecli-interactive
   az login
   ```

2. Use [az vm identity assign](/cli/azure/vm/identity/) con el comando `identity assign` para habilitar la identidad asignada por el sistema en una VM existente:

   ```azurecli-interactive
   az vm identity assign -g myResourceGroup -n myVm
   ```

### <a name="disable-system-assigned-identity-from-an-azure-vm"></a>Deshabilitación de una identidad asignada por el sistema desde una VM de Azure

Para deshabilitar una identidad administrada asignada por el sistema en una máquina virtual, la cuenta necesita la asignación de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  No se requiere ninguna otra asignación de roles de directorio de Azure AD.

Si tiene una máquina virtual que ya no necesita la identidad asignada por el sistema, pero aún necesita identidades asignadas por el usuario, use el siguiente comando:

```azurecli-interactive
az vm update -n myVM -g myResourceGroup --set identity.type='UserAssigned' 
```

Si tiene una máquina virtual que ya no necesita la identidad asignada por el sistema y no tiene identidades asignadas por el usuario, use el siguiente comando:

> [!NOTE]
> El valor `none` no distingue mayúsculas de minúsculas. Debe estar en minúscula. 

```azurecli-interactive
az vm update -n myVM -g myResourceGroup --set identity.type="none"
```


## <a name="user-assigned-managed-identity"></a>Identidad administrada asignada por el usuario

En esta sección, aprenderá a agregar y quitar una identidad administrada asignada por el usuario de una VM de Azure con la CLI de Azure. Si crea la identidad administrada asignada por el usuario en un RG diferente al de la máquina virtual. Tendrá que usar la dirección URL de la identidad administrada para asignarla a la máquina virtual.
por ejemplo: --identities "/subscriptions/<SUBID>/resourcegroups/<RESROURCEGROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER_ASSIGNED_ID_NAME>"

### <a name="assign-a-user-assigned-managed-identity-during-the-creation-of-an-azure-vm"></a>Asignación de una identidad administrada asignada por el usuario durante la creación de una VM de Azure

Para asignar una identidad asignada por el usuario a una máquina virtual durante su creación, la cuenta debe tener las asignaciones de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) y [Operador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-operator). No se requiere ninguna otra asignación de roles de directorio de Azure AD.

1. Puede omitir este paso si ya tiene un grupo de recursos que le gustaría usar. Cree un [grupo de recursos](~/articles/azure-resource-manager/management/overview.md#terminology) para contener e implementar la identidad administrada asignada por el usuario mediante [az group create](/cli/azure/group/#az-group-create). Asegúrese de reemplazar los valores de los parámetros `<RESOURCE GROUP>` y `<LOCATION>` con sus propios valores. :

   ```azurecli-interactive 
   az group create --name <RESOURCE GROUP> --location <LOCATION>
   ```

2. Cree una identidad administrada asignada por el usuario mediante [az identity create](/cli/azure/identity#az-identity-create).  El parámetro `-g` especifica el grupo de recursos en el que se crea la identidad administrada asignada por el usuario, mientras que el parámetro `-n` especifica su nombre.    
    
   [!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

   ```azurecli-interactive
   az identity create -g myResourceGroup -n myUserAssignedIdentity
   ```
   La respuesta contiene detalles de la identidad administrada asignada por el usuario que se ha creado, de forma similar a lo siguiente. El valor del identificador del recurso asignado a la identidad administrada asignada por el usuario se usa en el paso siguiente.

   ```json
   {
       "clientId": "73444643-8088-4d70-9532-c3a0fdc190fz",
       "clientSecretUrl": "https://control-westcentralus.identity.azure.net/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<myUserAssignedIdentity>/credentials?tid=5678&oid=9012&aid=73444643-8088-4d70-9532-c3a0fdc190fz",
       "id": "/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>",
       "location": "westcentralus",
       "name": "<USER ASSIGNED IDENTITY NAME>",
       "principalId": "e5fdfdc1-ed84-4d48-8551-fe9fb9dedfll",
       "resourceGroup": "<RESOURCE GROUP>",
       "tags": {},
       "tenantId": "733a8f0e-ec41-4e69-8ad8-971fc4b533bl",
       "type": "Microsoft.ManagedIdentity/userAssignedIdentities"    
   }
   ```

3. Cree una máquina virtual mediante [az vm create](/cli/azure/vm/#az-vm-create). En el ejemplo siguiente, se crea una VM asociada a la nueva identidad asignada por el usuario, según lo especificado por el parámetro `--assign-identity`. Asegúrese de reemplazar los valores de los parámetros `<RESOURCE GROUP>`, `<VM NAME>`, `<USER NAME>`, `<PASSWORD>` y `<USER ASSIGNED IDENTITY NAME>` por sus propios valores. 

   ```azurecli-interactive 
   az vm create --resource-group <RESOURCE GROUP> --name <VM NAME> --image UbuntuLTS --admin-username <USER NAME> --admin-password <PASSWORD> --assign-identity <USER ASSIGNED IDENTITY NAME>
   ```

### <a name="assign-a-user-assigned-managed-identity-to-an-existing-azure-vm"></a>Asignación de una identidad administrada asignada por el usuario a una VM de Azure existente

Para asignar una identidad asignada por un usuario a una máquina virtual, la cuenta debe tener las asignaciones de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) y [Operador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-operator). No se requiere ninguna otra asignación de roles de directorio de Azure AD.

1. Cree una identidad asignada por el usuario mediante el cmdlet [az identity create](/cli/azure/identity#az-identity-create).  El parámetro `-g` especifica el grupo de recursos donde se crea la identidad asignada por el usuario, mientras que el parámetro `-n` especifica su nombre. Asegúrese de reemplazar los valores de los parámetros `<RESOURCE GROUP>` y `<USER ASSIGNED IDENTITY NAME>` por sus propios valores:

    > [!IMPORTANT]
    > Actualmente, no se admite la creación de identidades administradas asignadas por el usuario con caracteres especiales (por ejemplo, guion bajo) en el nombre. Use caracteres alfanuméricos. Compruebe si hay actualizaciones.  Para más información, consulte [Preguntas más frecuentes y problemas conocidos](known-issues.md).

    ```azurecli-interactive
    az identity create -g <RESOURCE GROUP> -n <USER ASSIGNED IDENTITY NAME>
    ```
   La respuesta contiene detalles de la identidad administrada asignada por el usuario que se ha creado, de forma similar a lo siguiente. 

   ```json
   {
        "clientId": "73444643-8088-4d70-9532-c3a0fdc190fz",
        "clientSecretUrl": "https://control-westcentralus.identity.azure.net/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>/credentials?tid=5678&oid=9012&aid=73444643-8088-4d70-9532-c3a0fdc190fz",
        "id": "/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>",
        "location": "westcentralus",
        "name": "<USER ASSIGNED IDENTITY NAME>",
        "principalId": "e5fdfdc1-ed84-4d48-8551-fe9fb9dedfll",
        "resourceGroup": "<RESOURCE GROUP>",
        "tags": {},
        "tenantId": "733a8f0e-ec41-4e69-8ad8-971fc4b533bl",
        "type": "Microsoft.ManagedIdentity/userAssignedIdentities"    
   }
   ```

2. Asignar a su VM la identidad asignada por el usuario mediante [az vm identity assign](/cli/azure/vm). Asegúrese de reemplazar los valores de los parámetros `<RESOURCE GROUP>` y `<VM NAME>` con sus propios valores. El parámetro `<USER ASSIGNED IDENTITY NAME>` es la propiedad `name` del recurso de la identidad administrada asignar por el usuario, tal como se creó en el paso anterior. Si ha creado la identidad administrada asignada por el usuario en un RG diferente al de la máquina virtual. Tendrá que usar la dirección URL de la identidad administrada.

    ```azurecli-interactive
    az vm identity assign -g <RESOURCE GROUP> -n <VM NAME> --identities <USER ASSIGNED IDENTITY>
    ```

### <a name="remove-a-user-assigned-managed-identity-from-an-azure-vm"></a>Eliminación de una identidad administrada asignada por el usuario de una VM de Azure

Para quitar una identidad asignada por un usuario de una máquina virtual, la cuenta necesita la asignación de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor). 

Si se trata de la única identidad administrada asignada por el usuario que se ha asignado a una máquina virtual, `UserAssigned` se quitará del valor de tipo de identidad.  Asegúrese de reemplazar los valores de los parámetros `<RESOURCE GROUP>` y `<VM NAME>` con sus propios valores. El parámetro `<USER ASSIGNED IDENTITY>` será la propiedad `name` de la identidad asignada por el usuario, que se puede encontrar en la sección de identidad de la máquina virtual mediante `az vm identity show`:

```azurecli-interactive
az vm identity remove -g <RESOURCE GROUP> -n <VM NAME> --identities <USER ASSIGNED IDENTITY>
```

Si la VM no tiene una identidad administrada asignada por el sistema y desea quitar de ella todas las identidades asignadas por el usuario, utilice el siguiente comando:

> [!NOTE]
> El valor `none` no distingue mayúsculas de minúsculas. Debe estar en minúscula.

```azurecli-interactive
az vm update -n myVM -g myResourceGroup --set identity.type="none" identity.userAssignedIdentities=null
```

Si la VM tiene identidades asignadas tanto por el sistema como por el usuario, puede quitar todas las identidades asignadas por el usuario si cambia para usar solo las asignadas por el sistema. Use el comando siguiente:

```azurecli-interactive
az vm update -n myVM -g myResourceGroup --set identity.type='SystemAssigned' identity.userAssignedIdentities=null 
```

## <a name="next-steps"></a>Pasos siguientes
- [Información general sobre las identidades administradas de recursos de Azure](overview.md)
- Para ver guías de inicio rápido completas acerca de la creación de máquinas virtuales de Azure, consulte: 
  - [Creación de una máquina virtual Windows con CLI](../../virtual-machines/windows/quick-create-cli.md)  
  - [Creación de una máquina virtual Linux con PowerShell](../../virtual-machines/linux/quick-create-cli.md)