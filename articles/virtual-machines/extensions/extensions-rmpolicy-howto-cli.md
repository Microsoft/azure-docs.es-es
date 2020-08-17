---
title: Uso de Azure Policy para restringir la instalación de extensiones de máquina virtual (Linux)
description: Use Azure Policy para restringir las implementaciones de extensiones de VM.
services: virtual-machines-linux
documentationcenter: ''
author: axayjo
manager: gwallace
editor: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 03/23/2018
ms.author: akjosh
ms.reviewer: cynthn
ms.openlocfilehash: 2129130dab58c9e1fb98878efc3ec668eeb45359
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87069769"
---
# <a name="use-azure-policy-to-restrict-extensions-installation-on-linux-vms"></a>Uso de Azure Policy para restringir la instalación de extensiones en VM Linux

Si quiere impedir el uso o la instalación de determinadas extensiones en VM Linux, puede crear una definición de directiva de Azure mediante la CLI para restringir las extensiones de las máquinas virtuales dentro de un grupo de recursos. 

En este tutorial se usa la CLI dentro de Azure Cloud Shell, que se actualiza constantemente a la versión más reciente. Si quiere ejecutar la CLI de Azure localmente, debe instalar la versión 2.0.26 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli). 

## <a name="create-a-rules-file"></a>Creación de un archivo de reglas

Para restringir qué extensiones se pueden instalar, debe tener una [regla](../../governance/policy/concepts/definition-structure.md#policy-rule) que proporcione la lógica para identificar la extensión.

En este ejemplo se muestra cómo denegar la instalación de extensiones publicadas por "Microsoft.OSTCExtensions" mediante la creación de un archivo de reglas en Azure Cloud Shell. Sin embargo, si está trabajando en la CLI localmente, también puede crear un archivo local y reemplazar la ruta de acceso (~/clouddrive) por la ruta de acceso al archivo local de su máquina.

En el [Bash Cloud Shell](https://shell.azure.com/bash), escriba:

```bash
vim ~/clouddrive/azurepolicy.rules.json
```

Copie y pegue el siguiente archivo .json en el archivo.

```json
{
    "if": {
        "allOf": [
            {
                "field": "type",
                "equals": "Microsoft.OSTCExtensions/virtualMachines/extensions"
            },
            {
                "field": "Microsoft.OSTCExtensions/virtualMachines/extensions/publisher",
                "equals": "Microsoft.OSTCExtensions"
            },
            {
                "field": "Microsoft.OSTCExtensions/virtualMachines/extensions/type",
                "in": "[parameters('notAllowedExtensions')]"
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
}
```

Al acabar, pulse la tecla **Esc** y escriba **:wq** para guardar y cerrar el archivo.


## <a name="create-a-parameters-file"></a>Creación de un archivo de parámetros

También necesita un archivo de [parámetros](../../governance/policy/concepts/definition-structure.md#parameters) que cree una estructura que pueda usar para pasar una lista de las extensiones que se van a bloquear. 

En este ejemplo se muestra cómo crear un archivo de parámetros para VM Linux en Cloud Shell. Sin embargo, si está trabajando en la CLI localmente, también puede crear un archivo local y reemplazar la ruta de acceso (~/clouddrive) por la ruta de acceso al archivo local de su máquina.

En el [Bash Cloud Shell](https://shell.azure.com/bash), escriba:

```bash
vim ~/clouddrive/azurepolicy.parameters.json
```

Copie y pegue el siguiente archivo .json en el archivo.

```json
{
    "notAllowedExtensions": {
        "type": "Array",
        "metadata": {
            "description": "The list of extensions that will be denied. Example: CustomScriptForLinux, VMAccessForLinux etc.",
            "displayName": "Denied extension"
        }
    }
}
```

Al acabar, pulse la tecla **Esc** y escriba **:wq** para guardar y cerrar el archivo.

## <a name="create-the-policy"></a>Creación de la directiva

Una definición de directiva es un objeto que se usa para almacenar la configuración que le gustaría utilizar. En esta definición se incluyen las reglas y los archivos de parámetros. Cree la definición de directiva mediante la [creación de la definición de directivas az](/cli/azure/role/assignment?view=azure-cli-latest).

En este ejemplo, las reglas y los parámetros son los archivos creados y almacenados como archivos .json en Cloud Shell.

```azurecli-interactive
az policy definition create \
   --name 'not-allowed-vmextension-linux' \
   --display-name 'Block VM Extensions' \
   --description 'This policy governs which VM extensions that are blocked.' \
   --rules '~/clouddrive/azurepolicy.rules.json' \
   --params '~/clouddrive/azurepolicy.parameters.json' \
   --mode All
```


## <a name="assign-the-policy"></a>Asignación de la directiva

En este ejemplo se asigna la directiva a un grupo de recursos mediante la [creación de la asignación de directivas az](/cli/azure/policy/assignment). Ninguna de las máquinas virtuales creadas en el grupo de recursos **myResourceGroup** podrá instalar el acceso a VM Linux ni las extensiones del script personalizado para Linux. El grupo de recursos debe existir para poder asignar la directiva.

Use la [lista de cuentas de az](/cli/azure/account?view=azure-cli-latest) para obtener el id. de suscripción que se debe utilizar en lugar del del ejemplo.


```azurecli-interactive
az policy assignment create \
   --name 'not-allowed-vmextension-linux' \
   --scope /subscriptions/<subscription Id>/resourceGroups/myResourceGroup \
   --policy "not-allowed-vmextension-linux" \
   --params '{
        "notAllowedExtensions": {
            "value": [
                "VMAccessForLinux",
                "CustomScriptForLinux"
            ]
        }
    }'
```

## <a name="test-the-policy"></a>Prueba de la directiva

Pruebe la directiva creando una nueva máquina virtual e intentando agregar un nuevo usuario.


```azurecli-interactive
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --generate-ssh-keys
```

Intente crear un nuevo usuario denominado **myNewUser** con la extensión de acceso a máquinas virtuales.

```azurecli-interactive
az vm user update \
  --resource-group myResourceGroup \
  --name myVM \
  --username myNewUser \
  --password 'mynewuserpwd123!'
```



## <a name="remove-the-assignment"></a>Eliminación de la asignación

```azurecli-interactive
az policy assignment delete --name 'not-allowed-vmextension-linux' --resource-group myResourceGroup
```
## <a name="remove-the-policy"></a>Eliminación de la directiva

```azurecli-interactive
az policy definition delete --name 'not-allowed-vmextension-linux'
```

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte [Azure Policy](../../governance/policy/overview.md).
