---
title: 'Inicio rápido: Creación de una instancia de Data Science Virtual Machine: Plantilla de Resource Manager'
titleSuffix: Azure Data Science Virtual Machine
description: En este inicio rápido, se usará una plantilla de Azure Resource Manager para implementar rápidamente una instancia de Data Science Virtual Machine
services: machine-learning
author: lobrien
ms.author: laobri
ms.custom: subject-armqs
ms.date: 06/10/2020
ms.service: machine-learning
ms.subservice: data-science-vm
ms.topic: quickstart
ms.openlocfilehash: 8f83cdb09b94fb7eda1f8f700390be8b2c2147e0
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90883162"
---
# <a name="quickstart-create-an-ubuntu-data-science-virtual-machine-using-an-arm-template"></a>Inicio rápido: Creación de una instancia de Data Science Virtual Machine de Ubuntu mediante una plantilla de Resource Manager

En este inicio rápido se muestra cómo crear una instancia de Data Science Virtual Machine de Ubuntu 18.04 mediante una plantilla de Azure Resource Manager. Las instancias de Data Science Virtual Machine son máquinas virtuales basadas en la nube precargadas con un conjunto de herramientas y marcos de ciencia de datos y aprendizaje automático. Cuando se implementan en recursos de proceso con tecnología GPU, todas las herramientas y bibliotecas se configuran para usar este tipo de procesador.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-vm-ubuntu-DSVM-GPU-or-CPU%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

* Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/services/machine-learning/) antes de empezar.

* Para usar los comandos de la CLI de este documento desde su **entorno local**, necesita la [CLI de Azure](/cli/azure/install-azure-cli).

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-vm-ubuntu-DSVM-GPU-or-CPU/).

:::code language="json" source="~/quickstart-templates/101-vm-ubuntu-DSVM-GPU-or-CPU/azuredeploy.json":::

Los recursos siguientes se definen en la plantilla:

* [Microsoft.Network/networkInterfaces](/azure/templates/microsoft.network/networkinterfaces)
* [Microsoft.Network/networkSecurityGroups](/azure/templates/microsoft.network/networksecuritygroups)
* [Microsoft.Network/virtualNetworks](/azure/templates/microsoft.network/virtualnetworks)
* [Microsoft.Network/publicIPAddresses](/azure/templates/microsoft.network/publicipaddresses)
* [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts)
* [Microsoft.Compute/virtualMachines](/azure/templates/microsoft.compute/virtualmachines): Cree una máquina virtual basada en la nube. En esta plantilla, la máquina virtual está configurada como una instancia de Data Science Virtual Machine que ejecuta Ubuntu 18.04.

## <a name="deploy-the-template"></a>Implementación de la plantilla

Para usar la plantilla desde la CLI de Azure, inicie sesión y elija su suscripción (consulte [Inicio de sesión con la CLI de Azure](/cli/azure/authenticate-azure-cli)). A continuación, ejecute:

```azurecli-interactive
read -p "Enter the name of the resource group to create:" resourceGroupName &&
read -p "Enter the Azure location (e.g., centralus):" location &&
read -p "Enter the authentication type (must be 'password' or 'sshPublicKey') :" authenticationType &&
read -p "Enter the login name for the administrator account (may not be 'admin'):" adminUsername &&
read -p "Enter administrator account secure string (value of password or ssh public key):" adminPasswordOrKey &&
templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-ubuntu-DSVM-GPU-or-CPU/azuredeploy.json" &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri $templateUri --parameters adminUsername=$adminUsername authenticationType=$authenticationType adminPasswordOrKey=$adminPasswordOrKey &&
echo "Press [ENTER] to continue ..." &&
read
```

Cuando ejecute el comando anterior, escriba:

1. Nombre del grupo de recursos que desea crear para que contenga la instancia de Data Science Virtual Machine y los recursos asociados.
1. La ubicación de Azure en la que desea crear la implementación.
1. El tipo de autenticación que desea usar (especifique la cadena `password` o `sshPublicKey`).
1. El nombre de inicio de sesión de la cuenta de administrador (puede que este valor no sea `admin`).
1. El valor de la contraseña o la clave pública SSH de la cuenta.

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Para ver la instancia de Data Science Virtual Machine:

1. Vaya a [Azure Portal](https://portal.azure.com).
1. Inicie sesión.
1. Elija el grupo de recursos que acaba de crear.

Verá la información del grupo de recursos:

:::image type="content" source="media/dsvm-tutorial-resource-manager/resource-group-home.png" alt-text="Captura de pantalla de un grupo de recursos básico que contiene una instancia de Data Science Virtual Machine":::

Haga clic en el recurso de máquina virtual para ir a su página de información. Aquí puede encontrar información sobre la máquina virtual, incluidos los detalles de conexión.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no desea usar esta máquina virtual, elimínela. Dado que la instancia de Data Science Virtual Machine está asociada a otros recursos, como una cuenta de almacenamiento, es probable que desee eliminar todo el grupo de recursos que ha creado. Puede eliminar el grupo de recursos mediante el portal si hace clic en el botón **Eliminar** y confirma. Además, puede eliminar el grupo de recursos desde la CLI con:

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado una instancia de Data Science Virtual Machine a partir de una plantilla de Resource Manager.

> [!div class="nextstepaction"]
> [Programas de ejemplo y tutoriales de Machine Learning](dsvm-samples-and-walkthroughs.md)
