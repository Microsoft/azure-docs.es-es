---
title: Implementación de extensiones de máquina virtual con una plantilla
description: Aprenda a implementar extensiones de máquina virtual con plantillas de Azure Resource Manager.
author: mumian
ms.date: 04/23/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: f82e0eb45f4bc7c3260554b1b1120025029336bc
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89073649"
---
# <a name="tutorial-deploy-virtual-machine-extensions-with-arm-templates"></a>Tutorial: Implementación de extensiones de máquina virtual con plantillas de Resource Manager

Aprenda a usar [extensiones de máquina virtual de Azure](../../virtual-machines/extensions/features-windows.md) para realizar tareas de automatización y configuración posteriores a la implementación en máquinas virtuales de Azure. Hay muchas extensiones de máquina virtual diferentes disponibles para su uso con máquinas virtuales de Azure. En este tutorial se implementa una extensión de Custom Script desde una plantilla de Azure Resource Manager para ejecutar un script de PowerShell en una máquina virtual de Windows.  El script instala un servidor web en la máquina virtual.

En este tutorial se describen las tareas siguientes:

> [!div class="checklist"]
> * Preparación de un script de PowerShell.
> * Apertura de una plantilla de inicio rápido
> * Edición de la plantilla
> * Implementación de la plantilla

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para completar este artículo, necesitará lo siguiente:

* Visual Studio Code con la extensión Resource Manager Tools. Consulte [Quickstart: Creación de plantillas de Azure Resource Manager con Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).
* Para aumentar la seguridad, utilice una contraseña generada para la cuenta de administrador de máquina virtual. Este es un ejemplo para generar una contraseña:

    ```console
    openssl rand -base64 32
    ```

    Azure Key Vault está diseñado para proteger las claves criptográficas y otros secretos. Para más información, consulte el [Tutorial: Integración de Azure Key Vault en la implementación de la plantilla de Resource Manager](./template-tutorial-use-key-vault.md). También se recomienda actualizar la contraseña cada tres meses.

## <a name="prepare-a-powershell-script"></a>Preparación de un script de PowerShell.

Puede usar un script de PowerShell en línea o un archivo de script.  En este tutorial se muestra cómo usar un archivo de script. Un script de PowerShell con el siguiente contenido se comparte desde [GitHub](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorial-vm-extension/installWebServer.ps1):

```azurepowershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

Si elige publicar el archivo en su propia ubicación, actualice el elemento `fileUri` de la plantilla más adelante en el tutorial.

## <a name="open-a-quickstart-template"></a>Apertura de una plantilla de inicio rápido

Plantillas de inicio rápido de Azure es un repositorio de plantillas de Azure Resource Manager. En lugar de crear una plantilla desde cero, puede buscar una plantilla de ejemplo y personalizarla. La plantilla que se usa en este tutorial se denomina [Deploy a simple Windows VM](https://azure.microsoft.com/resources/templates/101-vm-simple-windows/).

1. En Visual Studio Code, seleccione **Archivo** > **Abrir archivo**.
1. En el cuadro **Nombre de archivo**, pegue la siguiente dirección URL:

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json
    ```

1. Para abrir el archivo, seleccione **Abrir**.
    La plantilla define cinco recursos:

   * [**Microsoft.Storage/storageAccounts**](/azure/templates/Microsoft.Storage/storageAccounts).
   * [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses).
   * [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups).
   * [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks).
   * [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces).
   * [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines).

     Resulta útil obtener cierta información básica de la plantilla antes de personalizarla.

1. Guarde una copia del archivo en su equipo local con el nombre *azuredeploy.json*; para ello, seleccione **Archivo** > **Guardar como**.

## <a name="edit-the-template"></a>Edición de la plantilla

Agregue un recurso de extensión de máquina virtual a la plantilla existente con el siguiente contenido:

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "apiVersion": "2019-12-01",
  "name": "[concat(variables('vmName'),'/', 'InstallWebServer')]",
  "location": "[parameters('location')]",
  "dependsOn": [
      "[concat('Microsoft.Compute/virtualMachines/',variables('vmName'))]"
  ],
  "properties": {
      "publisher": "Microsoft.Compute",
      "type": "CustomScriptExtension",
      "typeHandlerVersion": "1.7",
      "autoUpgradeMinorVersion":true,
      "settings": {
        "fileUris": [
          "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorial-vm-extension/installWebServer.ps1"
        ],
        "commandToExecute": "powershell.exe -ExecutionPolicy Unrestricted -File installWebServer.ps1"
      }
  }
}
```

Si necesita más información acerca de la definición de este recurso, consulte la [referencia de la extensión](/azure/templates/microsoft.compute/virtualmachines/extensions). Estos son algunos elementos importantes:

* **name**: Dado que el recurso de extensión es un recurso secundario del objeto de máquina virtual, el nombre debe tener el prefijo del nombre de máquina virtual. Consulte [Establecimiento del nombre y el tipo de recursos secundarios](child-resource-name-type.md).
* **dependsOn**: Cree el recurso de extensión después de haber creado la máquina virtual.
* **fileUris**: son las ubicaciones donde se almacenan los archivos de script. Si elige no utilizar la ubicación que se proporciona, deberá actualizar los valores.
* **commandToExecute**: Este comando invoca el script.

Para usar el script en línea, quite **fileUris** y actualice **commandToExecute** a:

```powershell
powershell.exe Install-WindowsFeature -name Web-Server -IncludeManagementTools && powershell.exe remove-item 'C:\\inetpub\\wwwroot\\iisstart.htm' && powershell.exe Add-Content -Path 'C:\\inetpub\\wwwroot\\iisstart.htm' -Value $('Hello World from ' + $env:computername)
```

Este script en línea también actualiza el contenido de Iisstart.html.

También debe abrir el puerto HTTP para poder acceder al servidor web.

1. Busque **securityRules** en la plantilla.
1. Agregue la siguiente regla junto a **default-allow-3389**.

    ```json
    {
      "name": "AllowHTTPInBound",
      "properties": {
        "priority": 1010,
        "access": "Allow",
        "direction": "Inbound",
        "destinationPortRange": "80",
        "protocol": "Tcp",
        "sourcePortRange": "*",
        "sourceAddressPrefix": "*",
        "destinationAddressPrefix": "*"
      }
    }
    ```

## <a name="deploy-the-template"></a>Implementación de la plantilla

Para conocer el procedimiento de implementación, consulte la sección "Implementación de la plantilla" de [Tutorial: Creación de plantillas de Resource Manager con recursos dependientes](./template-tutorial-create-templates-with-dependent-resources.md#deploy-the-template). Se recomienda usar una contraseña generada para la cuenta de administrador de la máquina virtual. Consulte la sección [Requisitos previos](#prerequisites) de este artículo.

En Cloud Shell, ejecute el siguiente comando para recuperar la dirección IP pública de la máquina virtual:

```azurepowershell
(Get-AzPublicIpAddress -ResourceGroupName $resourceGroupName).IpAddress
```

Pegue la dirección IP en un explorador web. Se abre la página de bienvenida predeterminada de Internet Information Services (IIS):

![Página de bienvenida de Internet Information Services](./media/template-tutorial-deploy-vm-extensions/resource-manager-template-deploy-extensions-customer-script-web-server.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos de Azure que implementó, elimine el grupo de recursos para limpiarlos.

1. En Azure Portal, en el panel de la izquierda, seleccione **Grupo de recursos**.
2. En el campo **Filtrar por nombre**, escriba el nombre del grupo de recursos.
3. Seleccione el nombre del grupo de recursos.
    Se muestran seis recursos en el grupo de recursos.
4. En el menú superior, seleccione **Eliminar grupo de recursos**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha implementado una máquina virtual y una extensión de virtual machine. La extensión ha instalado al servidor web de IIS en la máquina virtual. Para aprender a usar la extensión de Azure SQL Database para importar un archivo BACPAC, consulte:

> [!div class="nextstepaction"]
> [Implementación de extensiones de SQL](./template-tutorial-deploy-sql-extensions-bacpac.md)
