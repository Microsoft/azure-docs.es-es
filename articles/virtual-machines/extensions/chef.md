---
title: Extensión de Chef para máquinas virtuales de Azure
description: Implemente el cliente de Chef en una máquina virtual con la extensión de máquina virtual de Chef.
services: virtual-machines-linux
documentationcenter: ''
author: axayjo
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: article
ms.date: 09/21/2018
ms.author: akjosh
ms.openlocfilehash: 9391bb4867717b6780b50cf90b998254227d2310
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87082620"
---
# <a name="chef-vm-extension-for-linux-and-windows"></a>Extensión de máquina virtual de Chef para Linux y Windows

Chef Software proporciona una plataforma de automatización de DevOps para Linux y Windows que permite la administración de las configuraciones de servidor físicas y virtuales. La extensión de máquina virtual de Chef es una extensión que habilita Chef en máquinas virtuales.

## <a name="prerequisites"></a>Prerrequisitos

### <a name="operating-system"></a>Sistema operativo

La extensión de máquina virtual de Chef se admite en todos los [sistemas operativos admitidos por la extensión](https://support.microsoft.com/help/4078134/azure-extension-supported-operating-systems) en Azure.

### <a name="internet-connectivity"></a>Conectividad de Internet

La extensión de máquina virtual de Chef requiere que la máquina virtual de destino esté conectada a Internet para recuperar la carga del cliente de Chef de la red de entrega de contenido (CDN).  

## <a name="extension-schema"></a>Esquema de extensión

En el siguiente JSON, se muestra el esquema para la extensión de máquina virtual de Chef. La extensión requiere como mínimo la dirección URL del servidor de Chef, el nombre del cliente de validación y la clave de validación del servidor de Chef; estos valores se encuentran en el archivo `knife.rb` del archivo starter-kit.zip que se descarga cuando instala [Chef Automate](https://azuremarketplace.microsoft.com/marketplace/apps/chef-software.chef-automate) o un [servidor de Chef](https://downloads.chef.io/chef-server) independiente. Dado que la clave de validación debe tratarse como información confidencial, debe configurarse bajo el elemento **protectedSettings**, lo que significa que solo se descifrará en la máquina virtual de destino.

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "[concat(variables('vmName'),'/', parameters('chef_vm_extension_type'))]",
  "apiVersion": "2017-12-01",
  "location": "[parameters('location')]",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
  ],
  "properties": {
    "publisher": "Chef.Bootstrap.WindowsAzure",
    "type": "[parameters('chef_vm_extension_type')]",
    "typeHandlerVersion": "1210.13",
    "settings": {
      "bootstrap_options": {
        "chef_server_url": "[parameters('chef_server_url')]",
        "validation_client_name": "[parameters('chef_validation_client_name')]"
      },
      "runlist": "[parameters('chef_runlist')]"
    },
    "protectedSettings": {
      "validation_key": "[parameters('chef_validation_key')]"
    }
  }
}  
```

### <a name="core-property-values"></a>Valores de propiedad básicos

| Nombre | Valor / ejemplo | Tipo de datos
| ---- | ---- | ----
| apiVersion | `2017-12-01` | string (date) |
| publisher | `Chef.Bootstrap.WindowsAzure` | string |
| type | `LinuxChefClient`Linux, `ChefClient` (Windows) | string |
| typeHandlerVersion | `1210.13` | string (double) |

### <a name="settings"></a>Configuración

| Nombre | Valor / ejemplo | Tipo de datos | ¿Necesario?
| ---- | ---- | ---- | ----
| settings/bootstrap_options/chef_server_url | `https://api.chef.io/organizations/myorg` | string (url) | Y |
| settings/bootstrap_options/validation_client_name | `myorg-validator` | string | Y |
| settings/runlist | `recipe[mycookbook::default]` | string | Y |

### <a name="protected-settings"></a>Configuración protegida

| Nombre | Ejemplo | Tipo de datos | ¿Necesario?
| ---- | ---- | ---- | ---- |
| protectedSettings/validation_key | `-----BEGIN RSA PRIVATE KEY-----\nKEYDATA\n-----END RSA PRIVATE KEY-----` | string | Y |

<!--
### Linux-specific settings

| Name | Value / Example | Data Type |
| ---- | ---- | ---- |

### Windows-specific settings

| Name | Value / Example | Data Type |
| ---- | ---- | ---- |
-->

## <a name="template-deployment"></a>Implementación de plantilla

Las extensiones de VM de Azure pueden implementarse con plantillas de Azure Resource Manager. Las plantillas se pueden utilizar para implementar una o varias máquinas virtuales, instalar el cliente de Chef, conectarse al servidor de Chef y realizar la configuración inicial en el servidor tal y como se define en la [lista de ejecución](https://docs.chef.io/run_lists.html).

En la [Galería de inicio rápido de Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/chef-json-parameters-linux-vm), hay una plantilla de Resource Manager de ejemplo que incluye la extensión de máquina virtual de Chef.

La configuración JSON de una extensión de máquina virtual puede estar anidada en el recurso de máquina virtual o colocada en la raíz o nivel superior de una plantilla JSON de Resource Manager. La colocación de la configuración JSON afecta al valor del nombre y tipo del recurso. Para obtener más información, consulte el artículo sobre cómo [establecer el nombre y el tipo de recursos secundarios](../../azure-resource-manager/templates/child-resource-name-type.md).

## <a name="azure-cli-deployment"></a>Implementación de la CLI de Azure

La CLI de Azure puede utilizarse para implementar la extensión de máquina virtual de Chef en una máquina virtual existente. Reemplace el parámetro **validation_key** por el contenido de la clave de validación (este archivo como una extensión `.pem`).  Reemplace **validation_client_name**, **chef_server_url** y **run_list** con esos valores desde el archivo `knife.rb` en el kit de inicio.

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myExistingVM \
  --name LinuxChefClient \
  --publisher Chef.Bootstrap.WindowsAzure \
  --version 1210.13 --protected-settings '{"validation_key": "<validation_key>"}' \
  --settings '{ "bootstrap_options": { "chef_server_url": "<chef_server_url>", "validation_client_name": "<validation_client_name>" }, "runlist": "<run_list>" }'
```

## <a name="troubleshooting-and-support"></a>Solución de problemas y soporte técnico

Los datos sobre el estado de las implementaciones de extensiones pueden recuperarse desde Azure Portal y mediante la CLI de Azure. Para ver el estado de implementación de las extensiones de una máquina virtual determinada, ejecute el comando siguiente con la CLI de Azure.

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myExistingVM -o table
```

El resultado de la ejecución de las extensiones se registra en el archivo siguiente:

### <a name="linux"></a>Linux

```bash
/var/lib/waagent/Chef.Bootstrap.WindowsAzure.LinuxChefClient
```

### <a name="windows"></a>Windows

```powershell
C:\Packages\Plugins\Chef.Bootstrap.WindowsAzure.ChefClient\
```

### <a name="error-codes-and-their-meanings"></a>Códigos de error y su significado

| Código de error | Significado | Acción posible |
| :---: | --- | --- |
| 51 | Esta extensión no se admite en el sistema operativo de la máquina virtual. | |

Se puede encontrar más información adicional para solucionar problemas en el [archivo Léame de la extensión de máquina virtual de Chef](https://github.com/chef-partners/azure-chef-extension).

> [!NOTE]
> Para cualquier cosa relacionada directamente con Chef, póngase en contacto con el [soporte técnico de Chef](https://www.chef.io/support/).

## <a name="next-steps"></a>Pasos siguientes

Si necesita más ayuda con cualquier aspecto de este artículo, puede ponerse en contacto con los expertos de Azure en los [foros de MSDN Azure o Stack Overflow](https://azure.microsoft.com/support/forums/). Como alternativa, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione Obtener soporte. Para obtener información sobre el uso del soporte técnico, lea las [Preguntas más frecuentes de soporte técnico de Microsoft Azure](https://azure.microsoft.com/support/faq/).
