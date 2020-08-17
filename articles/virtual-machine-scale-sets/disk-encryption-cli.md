---
title: Cifrado de discos para conjuntos de escalado de Azure con la CLI de Azure
description: Obtenga información sobre cómo usar Azure PowerShell para cifrar las instancias de máquina virtual y los discos conectados en un conjunto de escalado de máquinas virtuales Windows
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: disks
ms.date: 10/15/2019
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: dfb8a0dc6c27b042319f43bc5970dd8d1bc2b094
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496957"
---
# <a name="encrypt-os-and-attached-data-disks-in-a-virtual-machine-scale-set-with-the-azure-cli"></a>Cifrado de discos de datos conectados y de sistema operativo en un conjunto de escalado de máquinas virtuales con la CLI de Azure

La CLI de Azure se usa para crear y administrar recursos de Azure desde la línea de comandos o en scripts. En esta guía de inicio rápido se muestra cómo usar la CLI de Azure para crear y cifrar un conjunto de escalado de máquinas virtuales. Para más información sobre cómo aplicar Azure Disk Encryption a un conjunto de escalado de máquinas virtuales, consulte [Azure Disk Encryption para conjuntos de escalado de máquinas virtuales](disk-encryption-overview.md).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si elige instalar y usar la CLI localmente, para este tutorial es preciso que ejecute la CLI de Azure de la versión 2.0.31 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="create-a-scale-set"></a>Creación de un conjunto de escalado

Antes de poder crear un conjunto de escalado, cree un grupo de recursos con [az group create](/cli/azure/group). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Ahora, cree un conjunto de escalado de máquinas virtuales con [az vmss create](/cli/azure/vmss). En el ejemplo siguiente se crea un conjunto de escalado llamado *myScaleSet* que se establece para actualizarse automáticamente a medida que se aplican los cambios, y se generan claves SSH si no existen en *~/.ssh/id_rsa*. Se conecta un disco de datos de 32 Gb a cada instancia de máquina virtual y se usa la [extensión de script personalizado](../virtual-machines/extensions/custom-script-linux.md) de Azure para preparar los discos de datos con [az vmss extension set](/cli/azure/vmss/extension):

```azurecli-interactive
# Create a scale set with attached data disk
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --admin-username azureuser \
  --generate-ssh-keys \
  --data-disk-sizes-gb 32

# Prepare the data disk for use with the Custom Script Extension
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroup \
  --vmss-name myScaleSet \
  --settings '{"fileUris":["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/prepare_vm_disks.sh"],"commandToExecute":"./prepare_vm_disks.sh"}'
```

Se tardan unos minutos en crear y configurar todos los recursos de conjunto de escalado y máquinas virtuales.

## <a name="create-an-azure-key-vault-enabled-for-disk-encryption"></a>Creación de un almacén de claves de Azure habilitado para el cifrado de discos

Azure Key Vault puede almacenar claves, secretos o contraseñas que permiten su implementación segura en las aplicaciones y los servicios. Las claves criptográficas se almacenan en Azure Key Vault con protección de software, o puede importar o generar las claves en módulos de seguridad de hardware (HSM) certificados conforme a las normas FIPS 140-2 de nivel 2. Las claves criptográficas se usan para cifrar y descifrar los discos virtuales conectados a la máquina virtual. Estas claves criptográficas se pueden controlar y se puede auditar su uso.

Defina su propio elemento *keyvault_name* exclusivo. Luego cree una nueva instancia de Key Vault con [az keyvault create](/cli/azure/ext/keyvault-preview/keyvault#ext-keyvault-preview-az-keyvault-create) en la misma suscripción y región que el conjunto de escalado y establezca la directiva de acceso *--enabled-for-disk-encryption*.

```azurecli-interactive
# Provide your own unique Key Vault name
keyvault_name=myuniquekeyvaultname

# Create Key Vault
az keyvault create --resource-group myResourceGroup --name $keyvault_name --enabled-for-disk-encryption
```

### <a name="use-an-existing-key-vault"></a>Uso de una instancia existente de Key Vault

Este paso solo es necesario si tiene una instancia de Key Vault que desea usar con el cifrado de discos. Omita este paso si creó una instancia de Key Vault en la sección anterior.

Defina su propio elemento *keyvault_name* exclusivo. Luego actualice la instancia de Key Vault con [az keyvault update](/cli/azure/ext/keyvault-preview/keyvault#ext-keyvault-preview-az-keyvault-update) y establezca la directiva de acceso *--enabled-for-disk-encryption*.

```azurecli-interactive
# Provide your own unique Key Vault name
keyvault_name=myuniquekeyvaultname

# Create Key Vault
az keyvault update --name $keyvault_name --enabled-for-disk-encryption
```

## <a name="enable-encryption"></a>Habilitación del cifrado

Para cifrar instancias de máquina virtual de un conjunto de escalado, primero obtenga información sobre el identificador de recurso de Key Vault con [az keyvault show](/cli/azure/ext/keyvault-preview/keyvault#ext-keyvault-preview-az-keyvault-show). Estas variables se usan luego para iniciar el proceso de cifrado con [az vmss encryption enable](/cli/azure/vmss/encryption#az-vmss-encryption-enable):

```azurecli-interactive
# Get the resource ID of the Key Vault
vaultResourceId=$(az keyvault show --resource-group myResourceGroup --name $keyvault_name --query id -o tsv)

# Enable encryption of the data disks in a scale set
az vmss encryption enable \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --disk-encryption-keyvault $vaultResourceId \
    --volume-type DATA
```

El proceso de cifrado puede tardar uno o dos minutos en iniciarse.

Como la directiva de actualización del conjunto de escalado creado en un paso anterior se establece en *automática*, las instancias de máquina virtual inician automáticamente el proceso de cifrado. En conjuntos de escalado donde la directiva de actualización es manual, inicie la directiva de cifrado en las instancias de máquina virtual con [az vmss update-instances](/cli/azure/vmss#az-vmss-update-instances).

### <a name="enable-encryption-using-kek-to-wrap-the-key"></a>Habilitación del cifrado con KEK para ajustar la clave

También puede usar una clave de cifrado de claves para mayor seguridad al cifrar el conjunto de escalado de máquinas virtuales.

```azurecli-interactive
# Get the resource ID of the Key Vault
vaultResourceId=$(az keyvault show --resource-group myResourceGroup --name $keyvault_name --query id -o tsv)

# Enable encryption of the data disks in a scale set
az vmss encryption enable \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --disk-encryption-keyvault $vaultResourceId \
    --key-encryption-key myKEK \
    --key-encryption-keyvault $vaultResourceId \
    --volume-type DATA
```

> [!NOTE]
>  La sintaxis para el valor del parámetro disk-encryption-keyvault es la cadena de identificador completa:</br>
/subscriptions/[guid-id-suscripción]/resourceGroups/[nombre-grupo-recursos]/providers/Microsoft.KeyVault/vaults/[nombre-almacén-claves]</br></br>
> La sintaxis para el valor del parámetro key-encryption-key es el identificador URI completo de la clave de cifrado de claves, como se muestra a continuación:</br>
https://[nombre-almacén-claves].vault.azure.net/keys/[nombre-clave-cifrado]/[id-exclusivo-clave-cifrado]

## <a name="check-encryption-progress"></a>Comprobación del progreso del cifrado

Para comprobar el estado del cifrado de disco, use [az vmss encryption show](/cli/azure/vmss/encryption#az-vmss-encryption-show):

```azurecli-interactive
az vmss encryption show --resource-group myResourceGroup --name myScaleSet
```

Cuando las instancias de máquina virtual están cifradas, el código de estado indica *EncryptionState/encrypted*, como se muestra en la salida de ejemplo siguiente:

```bash
[
  {
    "disks": [
      {
        "encryptionSettings": null,
        "name": "myScaleSet_myScaleSet_0_disk2_3f39c2019b174218b98b3dfae3424e69",
        "statuses": [
          {
            "additionalProperties": {},
            "code": "EncryptionState/encrypted",
            "displayStatus": "Encryption is enabled on disk",
            "level": "Info",
            "message": null,
            "time": null
          }
        ]
      }
    ],
    "id": "/subscriptions/guid/resourceGroups/MYRESOURCEGROUP/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/virtualMachines/0",
    "resourceGroup": "MYRESOURCEGROUP"
  }
]
```

## <a name="disable-encryption"></a>Deshabilitación del cifrado

Si ya no quiere usar los discos de instancias de máquina virtual cifrados, puede deshabilitar el cifrado con [az vmss encryption disable](/cli/azure/vmss/encryption?view=azure-cli-latest#az-vmss-encryption-disable), como se indica a continuación:

```azurecli-interactive
az vmss encryption disable --resource-group myResourceGroup --name myScaleSet
```

## <a name="next-steps"></a>Pasos siguientes

- En este artículo se ha usado la CLI de Azure para cifrar un conjunto de escalado de máquinas virtuales. También puede usar [Azure PowerShell](disk-encryption-powershell.md) o [plantillas de Azure Resource Manager](disk-encryption-azure-resource-manager.md).
- Si quiere aplicar Azure Disk Encryption después de aprovisionar otra extensión, puede usar la [secuenciación de extensiones](virtual-machine-scale-sets-extension-sequencing.md). 
- [Aquí](https://gist.githubusercontent.com/ejarvi/7766dad1475d5f7078544ffbb449f29b/raw/03e5d990b798f62cf188706221ba6c0c7c2efb3f/enable-linux-vmss.bat) se puede encontrar un ejemplo de archivo por lotes de un extremo a otro para el cifrado de discos de datos de un conjunto de escalado de Linux. En este ejemplo se crea un grupo de recursos y un conjunto de escalado de Linux, se monta un disco de datos de 5 GB y se cifra el conjunto de escalado de máquinas virtuales.
