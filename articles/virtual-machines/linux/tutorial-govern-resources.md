---
title: 'Tutorial: Administración de máquinas virtuales con la CLI'
description: En este tutorial aprenderá a usar la CLI de Azure para administrar máquinas virtuales de Azure aplicando RBAC, directivas, bloqueos y etiquetas.
services: virtual-machines-linux
documentationcenter: virtual-machines
author: tfitzmac
manager: gwallace
ms.service: virtual-machines-linux
ms.workload: infrastructure
ms.tgt_pltfrm: vm-linux
ms.topic: tutorial
ms.date: 09/30/2019
ms.author: tomfitz
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 890afadc82acc90ab0324058e07aa5c4d34d04e0
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87926168"
---
# <a name="tutorial-learn-about-linux-virtual-machine-management-with-azure-cli"></a>Tutorial: Información acerca de la administración de máquinas virtuales Linux con la CLI de Azure

[!INCLUDE [Resource Manager governance introduction](../../../includes/resource-manager-governance-intro.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI de Azure en el entorno local, en este tutorial es preciso que ejecute la versión 2.0.30 u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="understand-scope"></a>Descripción del ámbito

[!INCLUDE [Resource Manager governance scope](../../../includes/resource-manager-governance-scope.md)]

En este tutorial, se aplica toda la configuración de administración a un grupo de recursos para poder quitar fácilmente esas opciones al finalizar.

Vamos a crear el grupo de recursos.

```azurecli-interactive
az group create --name myResourceGroup --location "East US"
```

Actualmente, el grupo de recursos está vacío.

## <a name="role-based-access-control"></a>Control de acceso basado en rol

Desea asegurarse de que los usuarios de una organización tengan el nivel adecuado de acceso a estos recursos. No desea conceder acceso ilimitado a los usuarios, pero también es necesario garantizar que puedan realizar su trabajo. El [control de acceso basado en rol de Azure (Azure RBAC)](../../role-based-access-control/overview.md) le permite administrar los usuarios con permiso para completar acciones específicas en un ámbito.

Para crear y quitar las asignaciones de rol, los usuarios deben tener acceso a `Microsoft.Authorization/roleAssignments/*`. Esta acción se concede mediante los roles Propietario o Administrador de acceso de usuario.

Para administrar las soluciones de máquina virtual, hay tres roles específicos a los recursos, que normalmente proporcionan el acceso necesario:

* [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor)
* [Colaborador de la red](../../role-based-access-control/built-in-roles.md#network-contributor)
* [Colaborador de la cuenta de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-account-contributor)

En lugar de asignar roles a usuarios individuales, a menudo resulta más fácil usar un grupo de Azure Active Directory con usuarios que tienen que realizar acciones similares. A continuación, asigne a ese grupo el rol apropiado. Para este artículo puede usar un grupo existente para administrar la máquina virtual o el portal para [crear un grupo de Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

Después de crear un grupo o de buscar uno existente, use el comando [az role assignment create](/cli/azure/policy/assignment?view=azure-cli-latest#az-policy-assignment-create) para asignar el nuevo grupo de Azure Active Directory al rol Colaborador de la máquina virtual para el grupo de recursos.

```azurecli-interactive
adgroupId=$(az ad group show --group <your-group-name> --query objectId --output tsv)

az role assignment create --assignee-object-id $adgroupId --role "Virtual Machine Contributor" --resource-group myResourceGroup
```

Si recibe un error que indica **Principal \<guid> does not exist in the directory** (La entidad de seguridad no existe en el directorio), el nuevo grupo no se ha propagado en Azure Active Directory. Intente ejecutar el comando de nuevo.

Por lo general, repetirá el proceso para *Colaborador de la red* y *Colaborador de la cuenta de almacenamiento* con el objetivo de asegurarse de que los usuarios se asignan para administrar los recursos implementados. En este artículo, puede omitir esos pasos.

## <a name="azure-policy"></a>Azure Policy

[Azure Policy](../../governance/policy/overview.md) ayuda a asegurarse de que todos los recursos de la suscripción satisfacen los estándares corporativos. La suscripción ya tiene varias definiciones de directiva. Para ver las definiciones de directivas disponibles, use el comando [az policy definition list](/cli/azure/policy/definition?view=azure-cli-latest#az-policy-definition-list):

```azurecli-interactive
az policy definition list --query "[].[displayName, policyType, name]" --output table
```

Ve las definiciones de directiva existentes. El tipo de directiva es **BuiltIn** o **Custom**. Examine las definiciones para buscar las que describan una condición que quiera asignar. En este artículo, puede asignar directivas que:

* Limite las ubicaciones para todos los recursos.
* Limite las SKU para las máquinas virtuales.
* Realice auditorías de las máquinas virtuales que no usen discos administrados.

En el siguiente ejemplo, se recuperan tres definiciones de directivas en función del nombre para mostrar. Puede usar el comando [az policy assignment create](/cli/azure/policy/assignment?view=azure-cli-latest#az-policy-assignment-create) para asignar esas definiciones al grupo de recursos. En algunas directivas, puede proporcionar los valores de los parámetros para especificar los valores permitidos.

```azurecli-interactive
# Get policy definitions for allowed locations, allowed SKUs, and auditing VMs that don't use managed disks
locationDefinition=$(az policy definition list --query "[?displayName=='Allowed locations'].name | [0]" --output tsv)
skuDefinition=$(az policy definition list --query "[?displayName=='Allowed virtual machine SKUs'].name | [0]" --output tsv)
auditDefinition=$(az policy definition list --query "[?displayName=='Audit VMs that do not use managed disks'].name | [0]" --output tsv)

# Assign policy for allowed locations
az policy assignment create --name "Set permitted locations" \
  --resource-group myResourceGroup \
  --policy $locationDefinition \
  --params '{ 
      "listOfAllowedLocations": {
        "value": [
          "eastus", 
          "eastus2"
        ]
      }
    }'

# Assign policy for allowed SKUs
az policy assignment create --name "Set permitted VM SKUs" \
  --resource-group myResourceGroup \
  --policy $skuDefinition \
  --params '{ 
      "listOfAllowedSKUs": {
        "value": [
          "Standard_DS1_v2", 
          "Standard_E2s_v2"
        ]
      }
    }'

# Assign policy for auditing unmanaged disks
az policy assignment create --name "Audit unmanaged disks" \
  --resource-group myResourceGroup \
  --policy $auditDefinition
```

En el ejemplo anterior se da por hecho que ya conoce los parámetros de una directiva. Si necesita ver los parámetros, use:

```azurecli-interactive
az policy definition show --name $locationDefinition --query parameters
```

## <a name="deploy-the-virtual-machine"></a>Implementación de la máquina virtual

Ha asignado roles y directivas, por lo que está listo para implementar la solución. El tamaño predeterminado es Standard_DS1_v2, que es una de las SKU permitidas. El comando crea claves SSH si aún no existen en una ubicación predeterminada.

```azurecli-interactive
az vm create --resource-group myResourceGroup --name myVM --image UbuntuLTS --generate-ssh-keys
```

Una vez finalizada la implementación, puede aplicar más opciones de configuración de administración a la solución.

## <a name="lock-resources"></a>Bloqueo de recursos

Los [bloqueos de recursos](../../azure-resource-manager/management/lock-resources.md) impiden que los usuarios de una organización eliminen o modifiquen de forma accidental recursos críticos. A diferencia del control de acceso basado en rol, los bloqueos de recursos aplican una restricción a todos los usuarios y roles. Puede establecer el bloqueo de nivel en *CanNotDelete* o *ReadOnly*.

Para crear o eliminar bloqueos de administración, debe tener acceso a las acciones `Microsoft.Authorization/locks/*`. Entre los roles integrados, solamente se conceden esas acciones al **propietario** y al **administrador de acceso de usuarios**.

Para bloquear la máquina virtual y el grupo de seguridad de red, use el comando [az lock create](/cli/azure/resource/lock?view=azure-cli-latest#az-resource-lock-create):

```azurecli-interactive
# Add CanNotDelete lock to the VM
az lock create --name LockVM \
  --lock-type CanNotDelete \
  --resource-group myResourceGroup \
  --resource-name myVM \
  --resource-type Microsoft.Compute/virtualMachines

# Add CanNotDelete lock to the network security group
az lock create --name LockNSG \
  --lock-type CanNotDelete \
  --resource-group myResourceGroup \
  --resource-name myVMNSG \
  --resource-type Microsoft.Network/networkSecurityGroups
```

Para probar los bloqueos, intente ejecutar el siguiente comando:

```azurecli-interactive 
az group delete --name myResourceGroup
```

Verá un error en el que se indica que la operación de eliminación no se puede realizar debido a un bloqueo. Solo se puede eliminar el grupo de recursos si quita los bloqueos específicamente. Ese paso se muestra en [Limpieza de recursos](#clean-up-resources).

## <a name="tag-resources"></a>Etiquetado de recursos

Se aplican [etiquetas](../../azure-resource-manager/management/tag-resources.md) a los recursos de Azure para organizarlos de forma lógica por categorías. Cada etiqueta consta de un nombre y un valor. Por ejemplo, puede aplicar el nombre "Environment" y el valor "Production" a todos los recursos en producción.

[!INCLUDE [Resource Manager governance tags CLI](../../../includes/resource-manager-governance-tags-cli.md)]

Para aplicar etiquetas a una máquina virtual, use el comando [az resource tag](/cli/azure/resource?view=azure-cli-latest#az-resource-list). No se retiene ninguna etiqueta existente del recurso.

```azurecli-interactive
az resource tag -n myVM \
  -g myResourceGroup \
  --tags Dept=IT Environment=Test Project=Documentation \
  --resource-type "Microsoft.Compute/virtualMachines"
```

### <a name="find-resources-by-tag"></a>Búsqueda de recursos por etiqueta

Para buscar recursos con un nombre y valor de etiqueta, use el comando [az resource list](/cli/azure/resource?view=azure-cli-latest#az-resource-list):

```azurecli-interactive
az resource list --tag Environment=Test --query [].name
```

Puede utilizar los valores devueltos para las tareas de administración, como detener todas las máquinas virtuales con un valor de etiqueta.

```azurecli-interactive
az vm stop --ids $(az resource list --tag Environment=Test --query "[?type=='Microsoft.Compute/virtualMachines'].id" --output tsv)
```

### <a name="view-costs-by-tag-values"></a>Visualización de los costos por valores de etiqueta

[!INCLUDE [Resource Manager governance tags billing](../../../includes/resource-manager-governance-tags-billing.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

El grupo de seguridad de red bloqueado no se puede eliminar hasta que se quite el bloqueo. Para quitar el bloqueo, recupere los ID de los bloqueos y proporcióneselos al comando [az lock delete](/cli/azure/resource/lock?view=azure-cli-latest#az-resource-lock-delete):

```azurecli-interactive
vmlock=$(az lock show --name LockVM \
  --resource-group myResourceGroup \
  --resource-type Microsoft.Compute/virtualMachines \
  --resource-name myVM --output tsv --query id)
nsglock=$(az lock show --name LockNSG \
  --resource-group myResourceGroup \
  --resource-type Microsoft.Network/networkSecurityGroups \
  --resource-name myVMNSG --output tsv --query id)
az lock delete --ids $vmlock $nsglock
```

Cuando ya no se necesiten, puede usar el comando [az group delete](/cli/azure/group?view=azure-cli-latest#az-group-delete) para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados. Salga de la sesión SSH a la máquina virtual y, luego, elimine los recursos como se indica a continuación:

```azurecli-interactive 
az group delete --name myResourceGroup
```


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado una imagen de máquina virtual personalizada. Ha aprendido a:

> [!div class="checklist"]
> * Asignar un rol a los usuarios
> * Aplicar directivas que hagan cumplir los estándares
> * Proteger los recursos críticos con bloqueos
> * Etiquetar recursos para la facturación y la administración

Avance al siguiente tutorial para aprender a identificar los cambios y administrar las actualizaciones de paquetes en una máquina virtual.

> [!div class="nextstepaction"]
> [Administración de máquinas virtuales](tutorial-config-management.md)
