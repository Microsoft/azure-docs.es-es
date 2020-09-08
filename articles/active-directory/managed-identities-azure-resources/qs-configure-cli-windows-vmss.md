---
title: 'Configuración de identidades administradas en un conjunto de escalado de máquinas virtuales: CLI de Azure: Azure AD'
description: Instrucciones paso a paso para configurar identidades administradas asignadas por el sistema y por el usuario en un conjunto de escalado de máquinas virtuales de Azure, mediante la CLI de Azure.
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
ms.openlocfilehash: 3915108b9bd182053b62ee427fb95b5b984233db
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89255423"
---
# <a name="configure-managed-identities-for-azure-resources-on-a-virtual-machine-scale-set-using-azure-cli"></a>Configuración de identidades administradas de recursos de Azure en un conjunto de escalado de máquinas virtuales mediante la CLI de Azure

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Las identidades administradas para los recursos de Azure proporcionan a los servicios de Azure una identidad administrada automáticamente en Azure Active Directory. Puede usar esta identidad para autenticar a cualquier servicio que admita la autenticación de Azure AD, sin necesidad de tener credenciales en el código. 

En este artículo obtendrá más información sobre cómo realizar las siguientes operaciones de identidades administradas de recursos de Azure en un conjunto de escalado de máquinas virtuales de Azure mediante la CLI de Azure:
- Habilitación y deshabilitación de la identidad administrada asignada por el sistema en un conjunto de escalado de máquinas virtuales de Azure
- Adición y eliminación de una identidad asignada por el usuario un conjunto de escalado de máquinas virtuales de Azure


## <a name="prerequisites"></a>Requisitos previos

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md). **No olvide revisar la [diferencia entre una identidad administrada asignada por el sistema y una identidad administrada asignada por el usuario](overview.md#managed-identity-types)** .
- Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.
- Para realizar las operaciones de administración de este artículo, su cuenta debe tener las siguientes asignaciones de control de acceso basado en rol:

    > [!NOTE]
    > No se requieren asignaciones de roles del directorio de Azure AD.

    - [Colaborador de máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) para crear un conjunto de escalado de máquinas virtuales y habilitar y quitar la identidad administrada asignada por el usuario o por el sistema desde un conjunto de escalado de máquinas virtuales.
    - Rol [Colaborador de identidad administrada](../../role-based-access-control/built-in-roles.md#managed-identity-contributor) para crear una identidad administrada asignada por el usuario.
    - Rol [Operador de identidad administrada](../../role-based-access-control/built-in-roles.md#managed-identity-operator) para asignar y quitar una identidad administrada asignada por el usuario en un conjunto de escalado de máquinas virtuales.
- Para ejecutar los ejemplos de script de la CLI, tiene tres opciones:
    - Usar [Azure Cloud Shell](../../cloud-shell/overview.md) desde Azure Portal (consulte la sección siguiente).
    - Usar Azure Cloud Shell integrado a través del botón "Pruébelo", situado en la esquina superior derecha de cada bloque de código.
    - [Instale la versión más reciente de la CLI de Azure](/cli/azure/install-azure-cli) (2.0.13 o posterior) si prefiere usar una consola de CLI local. 
      
      > [!NOTE]
      > Los comandos se han actualizado para reflejar la versión más reciente de la [CLI de Azure](/cli/azure/install-azure-cli).

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="system-assigned-managed-identity"></a>Identidad administrada asignada por el sistema

En esta sección, obtendrá información sobre cómo habilitar y deshabilitar la identidad administrada asignada por el sistema para un conjunto de escalado de máquinas virtuales de Azure mediante la CLI de Azure.

### <a name="enable-system-assigned-managed-identity-during-creation-of-an-azure-virtual-machine-scale-set"></a>Habilitación de la identidad administrada asignada por el sistema durante la creación de un conjunto de escalado de máquinas virtuales de Azure

Para crear un conjunto de escalado de máquinas virtuales con la identidad administrada asignada por el sistema habilitada:

1. Si usa la CLI de Azure en una consola local, lo primero que debe hacer es iniciar sesión en Azure mediante el [inicio de sesión de az](/cli/azure/reference-index#az-login). Use una cuenta asociada a la suscripción de Azure en la que desearía implementar el conjunto de escalado de máquinas virtuales:

   ```azurecli-interactive
   az login
   ```

2. Cree un [grupo de recursos](../../azure-resource-manager/management/overview.md#terminology) para contener e implementar el conjunto de escalado de máquinas virtuales y sus recursos relacionados, con [az group create](/cli/azure/group/#az-group-create). Puede omitir este paso si ya tiene un grupo de recursos que le gustaría usar en su lugar:

   ```azurecli-interactive 
   az group create --name myResourceGroup --location westus
   ```

3. [Cree](/cli/azure/vmss/#az-vmss-create) un conjunto de escalado de máquinas virtuales. En el ejemplo siguiente se crea un conjunto de escalado de máquinas virtuales denominado *myVMSS* con una identidad administrada asignada por el sistema, como solicitó el parámetro `--assign-identity`. Los parámetros `--admin-username` y `--admin-password` especifican el nombre de usuario administrativo y la contraseña de la cuenta para el inicio de sesión en la máquina virtual. Actualice estos valores según convenga para su entorno: 

   ```azurecli-interactive 
   az vmss create --resource-group myResourceGroup --name myVMSS --image win2016datacenter --upgrade-policy-mode automatic --custom-data cloud-init.txt --admin-username azureuser --admin-password myPassword12 --assign-identity --generate-ssh-keys
   ```

### <a name="enable-system-assigned-managed-identity-on-an-existing-azure-virtual-machine-scale-set"></a>Habilitación de la identidad administrada asignada por el sistema en un conjunto de escalado de máquinas virtuales de Azure existente

Si necesita habilitar la identidad administrada asignada por el sistema en un conjunto de escalado de máquinas virtuales de Azure existente:

1. Si usa la CLI de Azure en una consola local, lo primero que debe hacer es iniciar sesión en Azure mediante el [inicio de sesión de az](/cli/azure/reference-index#az-login). Use una cuenta asociada a la suscripción de Azure que contenga el conjunto de escalado de máquinas virtuales.

   ```azurecli-interactive
   az login
   ```

2. [Habilite](/cli/azure/vmss/identity/#az-vmss-identity-assign) una identidad administrada asignada por el sistema en una máquina virtual existente:

   ```azurecli-interactive
   az vmss identity assign -g myResourceGroup -n myVMSS
   ```

### <a name="disable-system-assigned-managed-identity-from-an-azure-virtual-machine-scale-set"></a>Deshabilitación de la identidad administrada asignada por el sistema de un conjunto de escalado de máquinas virtuales de Azure

Si tiene un conjunto de escalado de máquinas virtuales que ya no necesita la identidad administrada asignada por el sistema, pero aún necesita identidades administradas asignadas por el usuario, use el siguiente comando:

```azurecli-interactive
az vmss update -n myVM -g myResourceGroup --set identity.type='UserAssigned' 
```

Si tiene una máquina virtual que ya no necesita la identidad administrada asignada por el sistema y no tiene identidades administradas asignadas por el usuario, use el siguiente comando:

> [!NOTE]
> El valor `none` no distingue mayúsculas de minúsculas. Debe estar en minúscula. 

```azurecli-interactive
az vmss update -n myVM -g myResourceGroup --set identity.type="none"
```



## <a name="user-assigned-managed-identity"></a>Identidad administrada asignada por el usuario

En esta sección obtendrá más información sobre cómo habilitar y quitar una identidad administrada asignada por el usuario mediante la CLI de Azure.

### <a name="assign-a-user-assigned-managed-identity-during-the-creation-of-a-virtual-machine-scale-set"></a>Asignación de una identidad administrada asignada por el usuario durante la creación de un conjunto de escalado de máquinas virtuales

En esta sección se explica el proceso de creación de un conjunto de escalado de máquinas virtuales y de asignación de una identidad administrada asignada por el usuario al conjunto de escalado de máquinas virtuales. Si ya tiene un conjunto de escalado de máquinas virtuales que quiera usar, omita esta sección y vaya a la siguiente.

1. Puede omitir este paso si ya tiene un grupo de recursos que le gustaría usar. Cree un [grupo de recursos](~/articles/azure-resource-manager/management/overview.md#terminology) para contener e implementar la identidad administrada asignada por el usuario mediante [az group create](/cli/azure/group/#az-group-create). Asegúrese de reemplazar los valores de los parámetros `<RESOURCE GROUP>` y `<LOCATION>` con sus propios valores. :

   ```azurecli-interactive 
   az group create --name <RESOURCE GROUP> --location <LOCATION>
   ```

2. Cree una identidad administrada asignada por el usuario mediante [az identity create](/cli/azure/identity#az-identity-create).  El parámetro `-g` especifica el grupo de recursos en el que se crea la identidad administrada asignada por el usuario, mientras que el parámetro `-n` especifica su nombre. Asegúrese de reemplazar los valores de los parámetros `<RESOURCE GROUP>` y `<USER ASSIGNED IDENTITY NAME>` por sus propios valores:

   [!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

   ```azurecli-interactive
   az identity create -g <RESOURCE GROUP> -n <USER ASSIGNED IDENTITY NAME>
   ```
   La respuesta contiene detalles de la identidad administrada asignada por el usuario que se ha creado, de forma similar a lo siguiente. El valor del recurso `id` asignado a la identidad administrada asignada por el usuario se usa en el paso siguiente.

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

3. [Cree](/cli/azure/vmss/#az-vmss-create) un conjunto de escalado de máquinas virtuales. En el ejemplo siguiente se crea un conjunto de escalado de máquinas virtuales asociado a la nueva identidad administrada asignada por el usuario, según lo especificado por el parámetro `--assign-identity`. Asegúrese de reemplazar los valores de los parámetros `<RESOURCE GROUP>`, `<VMSS NAME>`, `<USER NAME>`, `<PASSWORD>` y `<USER ASSIGNED IDENTITY>` por sus propios valores. 

   ```azurecli-interactive 
   az vmss create --resource-group <RESOURCE GROUP> --name <VMSS NAME> --image UbuntuLTS --admin-username <USER NAME> --admin-password <PASSWORD> --assign-identity <USER ASSIGNED IDENTITY>
   ```

### <a name="assign-a-user-assigned-managed-identity-to-an-existing-virtual-machine-scale-set"></a>Asignación de una identidad administrada asignada por el usuario a un conjunto de escalado de máquinas virtuales existente

1. Cree una identidad administrada asignada por el usuario mediante [az identity create](/cli/azure/identity#az-identity-create).  El parámetro `-g` especifica el grupo de recursos en el que se crea la identidad administrada asignada por el usuario, mientras que el parámetro `-n` especifica su nombre. Asegúrese de reemplazar los valores de los parámetros `<RESOURCE GROUP>` y `<USER ASSIGNED IDENTITY NAME>` por sus propios valores:

    ```azurecli-interactive
    az identity create -g <RESOURCE GROUP> -n <USER ASSIGNED IDENTITY NAME>
    ```
   La respuesta contiene detalles de la identidad administrada asignada por el usuario que se ha creado, de forma similar a lo siguiente.

   ```json
   {
        "clientId": "73444643-8088-4d70-9532-c3a0fdc190fz",
        "clientSecretUrl": "https://control-westcentralus.identity.azure.net/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY >/credentials?tid=5678&oid=9012&aid=73444643-8088-4d70-9532-c3a0fdc190fz",
        "id": "/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY>",
        "location": "westcentralus",
        "name": "<USER ASSIGNED IDENTITY>",
        "principalId": "e5fdfdc1-ed84-4d48-8551-fe9fb9dedfll",
        "resourceGroup": "<RESOURCE GROUP>",
        "tags": {},
        "tenantId": "733a8f0e-ec41-4e69-8ad8-971fc4b533bl",
        "type": "Microsoft.ManagedIdentity/userAssignedIdentities"    
   }
   ```

2. [Asigne](/cli/azure/vmss/identity) la identidad administrada asignada por el usuario al conjunto de escalado de máquinas virtuales. Asegúrese de reemplazar los valores de los parámetros `<RESOURCE GROUP>` y `<VIRTUAL MACHINE SCALE SET NAME>` con sus propios valores. El parámetro `<USER ASSIGNED IDENTITY>` es la propiedad `name` del recurso de la identidad asignada por el usuario, tal como se creó en el paso anterior:

    ```azurecli-interactive
    az vmss identity assign -g <RESOURCE GROUP> -n <VIRTUAL MACHINE SCALE SET NAME> --identities <USER ASSIGNED IDENTITY>
    ```

### <a name="remove-a-user-assigned-managed-identity-from-an-azure-virtual-machine-scale-set"></a>Eliminación de una identidad administrada asignada por el usuario de un conjunto de escalado de máquinas virtuales de Azure

Para [eliminar](/cli/azure/vmss/identity#az-vmss-identity-remove) una identidad administrada asignada por el usuario de un conjunto de escalado de máquinas virtuales, use `az vmss identity remove`. Si se trata de la única identidad administrada asignada por el usuario y que se ha asignado a un conjunto de escalado de máquinas virtuales, `UserAssigned` se quitará del valor de tipo de identidad.  Asegúrese de reemplazar los valores de los parámetros `<RESOURCE GROUP>` y `<VIRTUAL MACHINE SCALE SET NAME>` con sus propios valores. El parámetro `<USER ASSIGNED IDENTITY>` será la propiedad `name` de la identidad administrada asignada por el usuario, que se puede encontrar en la sección de identidades del conjunto de escalado de máquinas virtuales mediante `az vmss identity show`:

```azurecli-interactive
az vmss identity remove -g <RESOURCE GROUP> -n <VIRTUAL MACHINE SCALE SET NAME> --identities <USER ASSIGNED IDENTITY>
```

Si el conjunto de escalado de máquinas virtuales no tiene una identidad administrada asignada por el sistema y desea quitar de ella todas las identidades administradas asignadas por el usuario, utilice el siguiente comando:

> [!NOTE]
> El valor `none` no distingue mayúsculas de minúsculas. Debe estar en minúscula.

```azurecli-interactive
az vmss update -n myVMSS -g myResourceGroup --set identity.type="none" identity.userAssignedIdentities=null
```

Si el conjunto de escalado de máquinas virtuales tiene identidades administradas asignadas tanto por el sistema como por el usuario, puede quitar todas las identidades asignadas por el usuario si cambia para usar solo las identidades administradas asignadas por el sistema. Use el comando siguiente:

```azurecli-interactive
az vmss update -n myVMSS -g myResourceGroup --set identity.type='SystemAssigned' identity.userAssignedIdentities=null 
```

## <a name="next-steps"></a>Pasos siguientes

- [Información general sobre las identidades administradas de recursos de Azure](overview.md)
- Si desea consultar la guía completa de inicio rápido sobre creación de un conjunto de escalado de máquinas virtuales de Azure, consulte: 

  - [Creación de un conjunto de escalado de máquinas virtuales con la CLI](../../virtual-machines/linux/tutorial-create-vmss.md#create-a-scale-set)