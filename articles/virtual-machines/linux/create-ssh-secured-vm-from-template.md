---
title: Creación de una máquina virtual Linux en Azure a partir de una plantilla
description: En este artículo se describe cómo usar la CLI de Azure para crear una máquina virtual Linux a partir de una plantilla de Resource Manager.
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 03/22/2019
ms.author: cynthn
ms.openlocfilehash: ef9650690435a3770e59326e757ab1c783fa7b23
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87372763"
---
# <a name="how-to-create-a-linux-virtual-machine-with-azure-resource-manager-templates"></a>Procedimiento para crear una máquina virtual Linux con plantillas de Azure Resource Manager

Aprenda a crear una máquina virtual (VM) Linux mediante una plantilla de Azure Resource Manager y la CLI de Azure desde Azure Cloud Shell. Para crear una máquina virtual Windows, consulte [Creación de una máquina virtual Windows con una plantilla de Resource Manager](../windows/ps-template.md).

## <a name="templates-overview"></a>Introducción a las plantillas

Las plantillas de Azure Resource Manager son archivos JSON que definen la infraestructura y la configuración de la solución de Azure. Mediante una plantilla, puede implementar la solución repetidamente a lo largo del ciclo de vida y tener la seguridad de que los recursos se implementan de forma coherente. Para obtener más información sobre el formato de la plantilla y cómo se crea, consulte [Inicio rápido: Creación e implementación de plantillas de Azure Resource Manager mediante Azure Portal](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md). Para ver la sintaxis JSON para los tipos de recursos, consulte [Define resources in Azure Resource Manager templates](/azure/templates/microsoft.compute/allversions) (Definición de recursos en plantillas de Azure Resource Manager).

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

La creación de una máquina virtual de Azure normalmente incluye dos pasos:

1. Cree un grupo de recursos. Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Se debe crear un grupo de recursos antes de una máquina virtual.
1. Cree una máquina virtual.

En el siguiente ejemplo se crea una máquina virtual a partir de una [plantilla de inicio rápido de Azure](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-sshkey/azuredeploy.json). Se permite solo la autenticación de SSH para esta implementación. Cuando se le solicite, proporcione el valor de su propia clave pública SSH, por ejemplo, el contenido de *~/.ssh/id_rsa.pub*. Si necesita crear un par de claves SSH, consulte el artículo sobre cómo [crear y usar un par de claves SSH pública y privada para máquinas virtuales Linux en Azure](mac-create-ssh-keys.md). Esta es una copia de la plantilla:

[!code-json[create-linux-vm](~/quickstart-templates/101-vm-sshkey/azuredeploy.json)]

Para ejecutar el script de la CLI, seleccione **Pruébelo** para abrir Azure Cloud Shell. Para pegar el script, haga clic con el botón derecho en el shell y luego seleccione **Pegar**:

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the location (i.e. centralus):" &&
read location &&
echo "Enter the project name (used for generating resource names):" &&
read projectName &&
echo "Enter the administrator username:" &&
read username &&
echo "Enter the SSH public key:" &&
read key &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/101-vm-sshkey/azuredeploy.json --parameters projectName=$projectName adminUsername=$username adminPublicKey="$key" &&
az vm show --resource-group $resourceGroupName --name "$projectName-vm" --show-details --query publicIps --output tsv
```

El último comando de la CLI de Azure muestra la dirección IP pública de la máquina virtual recién creada. Necesita la dirección IP pública para conectarse a la máquina virtual. Consulte la siguiente sección de este artículo.

En el ejemplo anterior, especificó una plantilla almacenada en GitHub. También puede descargar o crear una plantilla y especificar la ruta de acceso local con el parámetro `--template-file`.

Estos son algunos recursos adicionales:

- Para aprender a desarrollar plantillas de Resource Manager, consulte la [documentación de Azure Resource Manager](../../azure-resource-manager/index.yml).
- Para ver los esquemas de la máquina virtual de Azure, consulte [Referencia sobre las plantillas de Azure](/azure/templates/microsoft.compute/allversions).
- Para ver más ejemplos de plantilla de máquina virtual, consulte [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Compute&pageNumber=1&sort=Popular).

## <a name="connect-to-virtual-machine"></a>Conexión a la máquina virtual

Después, puede acceder mediante SSH a la máquina virtual de la forma habitual. Proporcione su propia dirección IP pública obtenida con el comando anterior:

```bash
ssh <adminUsername>@<ipAddress>
```

## <a name="next-steps"></a>Pasos siguientes

En este ejemplo, creó una máquina virtual Linux básica. Para obtener más plantillas de Resource Manager que incluyen marcos de aplicaciones o crear entornos más complejos, examine las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Compute&pageNumber=1&sort=Popular).

Para más información sobre cómo crear plantillas, vea las propiedades y la sintaxis de JSON para los tipos de recursos que ha implementado:

- [Microsoft.Network/networkSecurityGroups](/azure/templates/microsoft.network/networksecuritygroups)
- [Microsoft.Network/publicIPAddresses](/azure/templates/microsoft.network/publicipaddresses)
- [Microsoft.Network/virtualNetworks](/azure/templates/microsoft.network/virtualnetworks)
- [Microsoft.Network/networkInterfaces](/azure/templates/microsoft.network/networkinterfaces)
- [Microsoft.Compute/virtualMachines](/azure/templates/microsoft.compute/virtualmachines)
