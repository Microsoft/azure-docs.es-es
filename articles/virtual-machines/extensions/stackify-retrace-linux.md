---
title: Extensión de agente de Linux de Azure para Stackify Retrace
description: Implementar el agente de Linux para Stackify Retrace en una máquina virtual Linux.
services: virtual-machines-linux
documentationcenter: ''
author: darinhoward
manager: gwallace
editor: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 04/12/2018
ms.author: akjosh
ms.openlocfilehash: 5914947bd994ee405f253e34c3dd919dd6561898
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87088859"
---
# <a name="stackify-retrace-linux-agent-extension"></a>Extensión de agente de Linux para Stackify Retrace

## <a name="overview"></a>Información general

Stackify proporciona productos que realizan el seguimiento de los detalles acerca de la aplicación para ayudar a encontrar y corregir problemas rápidamente. Para los equipos de desarrolladores, Retrace es un potenciador del rendimiento de aplicaciones completamente integrado y para varios entornos sumamente eficaz. Combina varias herramientas que necesita cada equipo de desarrollo.

Retrace es la ÚNICA herramienta que ofrece todas las funcionalidades siguientes en todos los entornos en una sola plataforma.

* Administración del rendimiento de las aplicaciones (APM)
* Registro de aplicaciones y servidores
* Supervisión y seguimiento de error
* Métricas personalizadas, de aplicaciones y servidores

**Acerca de la extensión de agente de Linux para Stackify**

Esta extensión proporciona una ruta de acceso de instalación para el agente de Linux para Retrace. 

## <a name="prerequisites"></a>Prerequisites

### <a name="operating-system"></a>Sistema operativo 

El agente de Retrace puede ejecutarse en estas distribuciones de Linux:

| Distribución | Versión |
|---|---|
| Ubuntu | 16.04 LTS, 14.04 LTS, 16.10 y 17.04 |
| Debian | 7.9 (o posterior) y 8.2 (o posterior), 9 |
| Red Hat | 6.7 (o posterior), 7.1 (o posterior) |
| CentOS | 6.3 (o posterior), 7.0 (o posterior) |

### <a name="internet-connectivity"></a>Conectividad de Internet

La extensión del agente de Stackify para Linux requiere que la máquina virtual de destino esté conectada a Internet. 

Puede que necesite ajustar la configuración de red para permitir las conexiones a Stackify. Consulte https://support.stackify.com/hc/en-us/articles/207891903-Adding-Exceptions-to-a-Firewall. 


## <a name="extension-schema"></a>Esquema de extensión

---

El siguiente código JSON muestra el esquema para la extensión del agente de Stackify Retrace. La extensión requiere `environment` y `activationKey`.

```json
    {
      "type": "extensions",
      "name": "StackifyExtension",
      "apiVersion": "[variables('apiVersion')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Compute/virtualMachines',variables('vmName'))]"
      ],
      "properties": {
        "publisher": "Stackify.LinuxAgent.Extension",
        "type": "StackifyLinuxAgentExtension",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "environment": "myEnvironment"
        },
        "protectedSettings": {
          "activationKey": "myActivationKey"
        }
      }
    }      
```

## <a name="template-deployment"></a>Implementación de plantilla 

Las extensiones de VM de Azure pueden implementarse con plantillas de Azure Resource Manager. El esquema JSON detallado en la sección anterior se puede usar en una plantilla de Azure Resource Manager para ejecutar el agente de Linux para Stackify Retrace durante la implementación de una plantilla de Azure Resource Manager.  

El JSON de una extensión de máquina virtual puede estar anidada en el recurso de máquina virtual, o colocada en la raíz o un nivel superior de una plantilla JSON de Resource Manager. La colocación de la plantilla JSON afecta al valor del nombre y tipo del recurso. Para obtener más información, consulte el artículo sobre cómo establecer el nombre y el tipo de recursos secundarios.

En el siguiente ejemplo se da por supuesto que la extensión de Linux para Stackify Retrace está anidada dentro de los recursos de máquina virtual. Cuando se anidan los recursos de extensión, la plantilla JSON se coloca en el objeto "resources": [] de la máquina virtual.

La extensión requiere `environment` y `activationKey`.

```json
    {
      "type": "extensions",
      "name": "StackifyExtension",
      "apiVersion": "[variables('apiVersion')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Compute/virtualMachines',variables('vmName'))]"
      ],
      "properties": {
        "publisher": "Stackify.LinuxAgent.Extension",
        "type": "StackifyLinuxAgentExtension",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "environment": "myEnvironment"
        },
        "protectedSettings": {
          "activationKey": "myActivationKey"
        }
      }
    }      
```

Al colocar la plantilla JSON de la extensión en la raíz de la plantilla, el nombre de recurso incluye una referencia a la máquina virtual principal, y el tipo refleja la configuración anidada.

```json
    {
        "type": "Microsoft.Compute/virtualMachines/extensions",
        "name": "<parentVmResource>/StackifyExtension",
        "apiVersion": "[variables('apiVersion')]",
        "location": "[resourceGroup().location]",
        "dependsOn": [
            "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
        ],
        "properties": {
            "publisher": "Stackify.LinuxAgent.Extension",
            "type": "StackifyLinuxAgentExtension",
            "typeHandlerVersion": "1.0",
            "autoUpgradeMinorVersion": true,
            "settings": {
              "environment": "myEnvironment"
            },
            "protectedSettings": {
              "activationKey": "myActivationKey"
            }
        }
    }
```


## <a name="powershell-deployment"></a>Implementación de PowerShell

El comando `Set-AzVMExtension` puede utilizarse para implementar la extensión de máquina virtual del agente de Linux para Stackify Retrace en una máquina virtual existente. Antes de ejecutar el comando, las configuraciones públicas y privadas deben estar almacenadas en una tabla hash de PowerShell.

La extensión requiere `environment` y `activationKey`.

```powershell
$PublicSettings = @{"environment" = "myEnvironment"}
$ProtectedSettings = @{"activationKey" = "myActivationKey"}

Set-AzVMExtension -ExtensionName "Stackify.LinuxAgent.Extension" `
    -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" `
    -Publisher "Stackify.LinuxAgent.Extension" `
    -ExtensionType "StackifyLinuxAgentExtension" `
    -TypeHandlerVersion 1.0 `
    -Settings $PublicSettings `
    -ProtectedSettings $ProtectedSettings `
    -Location WestUS `
```

## <a name="azure-cli-deployment"></a>Implementación de la CLI de Azure 

La herramienta de la CLI de Azure puede utilizarse para implementar la extensión de máquina virtual del agente de Linux para Stackify Retrace en una máquina virtual existente.  

La extensión requiere `environment` y `activationKey`.

```azurecli
az vm extension set --publisher 'Stackify.LinuxAgent.Extension' --version 1.0 --name 'StackifyLinuxAgentExtension' --protected-settings '{"activationKey":"myActivationKey"}' --settings '{"environment":"myEnvironment"}'  --resource-group 'myResourceGroup' --vm-name 'myVmName'
```

## <a name="troubleshoot-and-support"></a>Solución de problemas y asistencia

### <a name="error-codes"></a>Códigos de error

| Código de error | Significado | Acción posible |
| :---: | --- | --- |
| 10 | Error de instalación | wget es obligatorio |
| 20 | Error de instalación | python es obligatorio |
| 30 | Error de instalación | sudo es obligatorio |
| 40 | Error de instalación | activationKey es obligatorio |
| 51 | Error de instalación | La distribución del sistema operativo no se admite |
| 60 | Error de instalación | environment es obligatorio |
| 70 | Error de instalación | Desconocido |
| 80 | Error de habilitación | No se pudo configurar el servicio |
| 90 | Error de habilitación | No se pudo iniciar el servicio |
| 100 | Error de deshabilitación | No se pudo detener el servicio |
| 110 | Error de deshabilitación | No se pudo quitar el servicio |
| 120 | Error de desinstalación | No se pudo detener el servicio |

Si necesita más ayuda, puede ponerse en contacto con el soporte técnico de Stackify en https://support.stackify.com.