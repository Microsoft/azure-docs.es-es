---
title: Creación y cifrado de una máquina virtual Linux con la CLI de Azure
description: En esta guía de inicio rápido, aprenderá a utilizar la CLI de Azure para crear y cifrar una máquina virtual Linux
author: msmbaldwin
ms.author: mbaldwin
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: quickstart
ms.date: 05/17/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: b07ddd3ec98e2094eb3a463501f42d65e15370e6
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502365"
---
# <a name="quickstart-create-and-encrypt-a-linux-vm-with-the-azure-cli"></a>Inicio rápido: Creación y cifrado de una máquina virtual Linux con la CLI de Azure

La CLI de Azure se usa para crear y administrar recursos de Azure desde la línea de comandos o en scripts. En esta guía de inicio rápido se muestra cómo usar la CLI de Azure para crear y cifrar una máquina virtual (VM) Linux.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

Si decide instalar y usar la CLI de Azure localmente, para este inicio rápido es preciso ejecutar la CLI de Azure, versión 2.0.30 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group?view=azure-cli-latest#az-group-create). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli-interactive
az group create --name "myResourceGroup" --location "eastus"
```

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

Cree la máquina virtual con [az vm create](/cli/azure/vm?view=azure-cli-latest#az-vm-create). En el ejemplo siguiente se crea una máquina virtual denominada *myVM*.

```azurecli-interactive
az vm create \
    --resource-group "myResourceGroup" \
    --name "myVM" \
    --image "Canonical:UbuntuServer:16.04-LTS:latest" \
    --size "Standard_D2S_V3"\
    --generate-ssh-keys
```

La creación de la máquina virtual y los recursos auxiliares tarda unos minutos en realizarse. En la salida de ejemplo siguiente se muestra que la operación de creación de la máquina virtual se realizó correctamente.

```
{
  "fqdns": "",
  "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "52.174.34.95",
  "resourceGroup": "myResourceGroup"
}
```

## <a name="create-a-key-vault-configured-for-encryption-keys"></a>Creación de una instancia de Key Vault configurada para claves de cifrado

Azure Disk Encryption almacena su clave de cifrado en una instancia de Azure Key Vault. Cree una instancia de Key Vault con [az keyvault create](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-create). Para habilitar la instancia de Key Vault para almacenar claves de cifrado, use el parámetro --enabled-for-disk-encryption.

> [!Important]
> Cada almacén de claves debe tener un nombre que sea único en Azure. En los ejemplos siguientes, reemplace <su-nombre-único-del-almacén-de-claves> por el nombre que elija.

```azurecli-interactive
az keyvault create --name "<your-unique-keyvault-name>" --resource-group "myResourceGroup" --location "eastus" --enabled-for-disk-encryption
```

## <a name="encrypt-the-virtual-machine"></a>Cifrado de la máquina virtual

Cifre su máquina virtual con [az vm encryption](/cli/azure/vm/encryption?view=azure-cli-latest), que proporciona su nombre de Key Vault único al parámetro --disk-encryption-keyvault.

```azurecli-interactive
az vm encryption enable -g "MyResourceGroup" --name "myVM" --disk-encryption-keyvault "<your-unique-keyvault-name>"
```

Tras unos instantes, el proceso devolverá "The encryption request was accepted. Please use 'show' command to monitor the progress". El comando "show" es [az vm show](/cli/azure/vm/encryption#az-vm-encryption-show).

```azurecli-interactive
az vm show --name "myVM" -g "MyResourceGroup"
```

Al habilitarse el cifrado, verá lo siguiente en la salida que se devuelve:

```
"EncryptionOperation": "EnableEncryption"
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, puede usar el comando [az group delete](/cli/azure/group) para quitar el grupo de recursos, la máquina virtual y Key Vault. 

```azurecli-interactive
az group delete --name "myResourceGroup"
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, creó una máquina virtual y una instancia de Key Vault que se habilitó para claves de cifrado, y cifró la máquina virtual.  En el artículo siguiente obtendrá más información acerca Azure Disk Encryption para máquinas virtuales Linux.

> [!div class="nextstepaction"]
> [Introducción a Azure Disk Encryption](disk-encryption-overview.md)
